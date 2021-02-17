---
layout: post 
title: "Node 的多进程/多线程/线程通信/监控内存" 
author: "fedono"
---

- 如何开启多进程
- 多进程之间如何通信
- 如何监控内存


Node 在选型时决定在 V8 引擎之上构建，也就意味着它的模型与浏览器类似。我们的 JavaScript 将会运行在单个进程的单个线程上。它带来的好处是：程序状态是单一的，在没有多线程的情况下没有锁、线程同步问题，操作系统在调度时也因为较少上下文的切换，可以很好地提高 CPU 的使用率。

单进程单线程并非完美的结构，如今 CPU 基本均是多核的，真正的服务器往往还有多个 CPU。一个 Node 进程只能利用一个核，这将抛出 Node 实际应用的第一个问题：**如何充分利用多核 CPU 服务器。** 

另外，由于 Node 执行在单线程上，一旦单线程上抛出的异常没有被捕获，将会引起整个进程的崩溃。这给 Node 的实际应用抛出了第二个问题：**如何保证进程的健壮性和稳定性**



`Apache ` 就是采用多线程/多进程模型实现的，当并发增长到上万时，内存耗用的问题将会暴露出来，这即是注明的 C10K 问题

为了解决高并发问题，基于事件驱动的服务模型出现了，像Node 和 NGINX 均是基于事件驱动的方式实现的，采用单线程避免了不必要的内存开销和上下文切换开销。

基于事件的服务模型存在的问题即是：CPU的利用率和进程的健壮性。对于 Node 来说，所有的请求的上下文都是统一的，它的稳定性是亟需解决的问题。

由于所有的处理都在单线程上进行，影响事件驱动服务模型性能的点在于CPU的计算能力，它的上线决定这类服务模型的性能上限，但它不受多进程或多线程模式中资源上线的影响，可伸缩性远比前两者高。如果解决掉多核CPU的利用问题，带来的性能上提升是可观的。

### 多进程架构

面对单线程单进程多核使用不足的问题，前人的经验是启动多进程即可。理想状态下每个进程各自利用一个CPU，以此实现多核CPU 的利用。所幸，Node 提供了 child_process 模块，并且也提供了 `child_process.fork()` 函数为我们实现进程的复制。

经典示例如下

```js
// worker.js 侦听1000到2000之间的一个随机端口
const http = require('http');
http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
}).listen(Math.round((1 + Math.random()) * 1000), '127.0.0.1');

// master.js 根据当前cpu数量复制出对应Node进程数。在*nix系统下可以通过ps aux | grep worker.js 查看到进程的数量
const fork = require('child_process').fork;
const cpus = require('os').cpus();
for (let i = 0; i < cpus.length; i++) {
    fork('./worker.js');
}
```

下图为著名的`Master-Worker` 模式，又称主从模式。图中的进程分为两种：主进程和工作进程。这是典型的分布式架构中用于并行处理业务的模式，具备较好的可伸缩性和稳定性。主进程不负责具体的业务处理，而是负责调度或管理工作进程，它是趋于稳定的。工作进程负责具体的业务处理，因为业务的多种多样，其中一项业务由多人开发完成，所以工作进程的稳定性值得开发者关注。

![Master-worker](../assets/imgs/process/Master-Worker.png)

> 通过 fork() 复制的进程都是一个独立的进程，这个进程中有着独立而全新的 V8 实例。它需要至少 30毫秒的启动时间和至少10MB的内存。尽管Node 提供了 fork() 供我们复制进程使每个 CPU内核都使用上，但是依然要切记 fork() 进程使昂贵的。好在 Node 通过事件驱动的方式在单线程上解决了大并发的问题，这里启动多个进程只是为了充分将 CPU 资源利用起来，而不是为了解决并发问题。

`child_process` 提供了四种方法用于创建子进程

1. `spawn()` 启动一个子进程来执行命令
2. `exec()` 启动一个子进程来执行命令，与 `spawn() ` 不同的是其接口不同，他有一个回调函数来获知子进程的状况
3. `execFile()` 启动一个子进程来执行可执行的文件
4. `fork()` 与`spawn()` 类似，不同的点在于它创建Node 的子进程只需指定要执行的JavaScript 文件模块即可

`spawn()` 与`exec()`、`execFile()` 不同的是，后两者创建时可以指定`timeout` 属性设置超时时间，一旦创建的进程运行超过设定的时间将会被杀死。

`exec()` 与`execFile()` 不同的是，`exec()` 适合执行已有的命令，`execFile()` 适合执行文件。这里我们以一个寻常命令为例，`node worker.js` 分别用上述四种方法来实现，如下所示：

```js
const cp = require('child_process');
cp.spawn('node', ['worker.js']);
cp.exec('node worker.js', function(err, stdout, stderr) {
    // some code
});
cp.execFile('worker.js', function(err, stdout, stderr) {
    // some code
});
cp.fork('./worker.js');
```

以上四个方法在创建子进程之后均会返回子进程对象。他们的差别如下

![diff-of-method](../assets/imgs/process/diff-of-method.png) 

这里的可执行文件是指可以直接执行的文件，如果是`JavaScript` 文件通过`execFile()` 运行，它的首行内容必须添加如下代码

```js
#!/usr/bin/env node
```

尽管4种创建子进程的方式有些差异，但事实上后面三种方法都是`spawn()` 的延伸应用。

### 进程间通信

> 这个是重点中的重点

在 `Master-Worker` 模式中，要实现主进程管理和调度工作进程的功能，需要主进程和工作进程之间的通信。

在前端浏览器中，`JavaScript` 主线程与UI渲染共用一个线程。执行JavaScript的时候UI渲染时停滞的，渲染UI时，JavaScript是停滞的，两者互相阻塞。长时间执行JavaScript将会造成UI停顿不响应。为了解决这个问题，HTML5提出了`WebWorker` API，`WebWorker` 运行创建工作线程并在后台运行，使得一些阻塞较为严重的计算不影响主线程的UI渲染。API 示例如下

```js
// worke.html
const worker = new Worker('./worker.js');
worker.onmessage = function(event) {
  document.getElementById('result').textContent = event.data;
}
// worker.js
let n = 1;
search: while(true) {
    n += 1;
    for (let i = 2; i <= Math.sqrt(n); i += 1) {
        if (n % 1 == 0) {
            continue search;
        }
    }
    postMessage(n);
}
```

主线程与工作线程之间通过`onmessage()` 和 `postMessage()` 进行通信，子进程对象则由`send()`  方法实现主进程向子进程发送数据，`message` 事件实现收听子进程发来的数据，与 `API` 在一定程度上相似。通过消息传递内容，而不是共享或直接操作相关资源，这是较为轻量和无依赖的做法。

```js
// parent.js
const cp = require('child_process');
const n = cp.fork(__dirname + '/sub/js');

n.on('message', function(m) {
    console.log('PARENT got message:', m);
});

n.send({hello: 'world'});

// sub.js
process.on('message', function(m) {
    console.log('CHILD got message:', m);
});

process.send({foo: 'bar'});
```

执行`node parent.js`

获得以下输出

```js
CHILD got message: { hello: 'world' }
PARENT got message: { foo: 'bar' }
```

通过`fork()` 或者其他API，创建子进程之后，为了实现父子进程之前的通信，父进程与子进程之间将会创建 IPC通道。通过 IPC通道，父子进程之间才能通过 `message` 和 `send` 传递消息。

#### 进程间通信原理

IPC 的全称是`Inter-Process Communication` ，即进程间通信。进程间通信的目的是为了让不同的进程能够互相访问资源并进行协调工作。实现进程间通信的技术有很多，如命名管道、匿名管道、socket、信号量、共享内存、消息队列、Domain Socket 等。Node中实现IPC通道的事管道(pipe) 技术。但此管道非彼管道，在 Node 中管道是个抽象层面的称呼，具体细节由libuv提供，在`windows` 下由命名管道(`named pipe`) 实现，*nix 系统则采用`Unix Domain Socket` 实现。表现在应用层上的进程间通信只有简单的 `message` 事件和`send()` 方法，接口十分简洁和消息化。

下图为IPC 创建和实现的示意图

![](../assets/imgs/process/create-ipc.png)

父进程在实际创建子进程之前，会创建IPC通道并监听它，然后才真正创建出子进程，并通过环境变量(NODE_CHANNEL_FD) 告诉子进程这个IPC通道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个已存在的 IPC通道，从而完成父子进程之间的连接。下图为创建IPC管道的步骤示意图。

![](../assets/imgs/process/create-ipc-step.png)



建立连接之后的父子进程就可以自由地通信了。由于IPC 通道是用命名管道或Domain Socket 创建的，它们与网络socket 的行为比较类似，属于双向通信。不同的是它们在系统内核中就完成了进程间的通信，而不用进过实际的网络层，非常高效。在 Node 中，IPC通道被抽象为Stream对象，在调用`send()` 时发送数据（类似于write()），接收到的消息会通过 `message` 事件（类似于`data`）触发给应用层

> 只有启动的子进程是`Node` 进程使，子进程才会根据环境变量去连接IPC通道，对于其他类型的子进程则无法实现进程间通信，除非其他进程也按约定去连接这个已经创建好的IPC通道。

#### 句柄传递

建立好进程之间的IPC后，如果仅仅只用来发送一些简单的数据，显然不够我们的实际应用使用。

多个进程不能监听一个端口，要解决这个问题，通常的做法是让每个进程监听不同的端口，其中主进程监听主端口（如80），主进程对外接收所有的网络请求，再将这些请求分别代理到不同的端口的进程上。示意图如下

![](../assets/imgs/process/process-multiple-port.png) 

通过代理，可以避免端口不能重复监听的问题，甚至可以在代理进程上做适当的负载均衡，使得每个子进程可以较为均衡地执行任务。由于进程每接收到一个连接，将会用掉一个文件描述符，因此代理方案中客户端连接到代理进程，代理进程连接到工作进程的过程需要用掉两个文件描述符。操作系统的文件描述符是有限的，代理方案浪费掉一倍数量的文件描述符的做法影响了系统的扩展能力。

为了解决这个问题，Node在版本v0.5.9引入了进程间发送句柄的功能。`send()` 方法除了能通过IPC发送数据后，还能发送句柄，第二个可选参数就是句柄。

```js
child.send(message, [sendHandler]);
```

**句柄** 是一种可以用来标识资源的引用，它的内部包含了指向对象的文件描述符。比如句柄可以用来标识一个服务器端socket 对象。一个客户端socket 对象。一个UDP套接字。一个管道等。

发送句柄意味着，我们可以去掉代理这种方法，使主进程接收到socket请求后，将这个socket直接发送的工作进程，而不是重新建立新的socket 连接来转发数据。文件描述符浪费的问题可以通过这样的方式轻松解决。

示例代码如下

```js
// parent.js
const child = require('child_process').fork('./child.js');

// Open up the server object and send the handle
const server = require('net').createServer();
server.on('connection', function(socket) {
    socket.end('handled by parent\n');
});
server.listen(1337, function() {
    child.send('server', server);
});

// child.js
process.on('message', function(m, server) {
    if (m === 'server') {
        server.on('connection', function(socket){
            socket.end('handled by child\n');
        })
    }
})
```

这个示例中直接将一个TCP服务器发送给了子进程，我们测试一下

```js
node parent.js
// 新开一个命令窗口，用 curl 工具测试
curl 'http://127.0.0.1:1337/' // handled by child
curl 'http://127.0.0.1:1337/' // handled by child
curl 'http://127.0.0.1:1337/' // handled by parent
curl 'http://127.0.0.1:1337/' // handled by child
```

命令行中响应结果可以看出，这里子进程和父进程都有可能处理我们客户端发起的请求。

试试将服务发给多个子进程

```js
// parent.js
const cp = require('child_process');
const child1 = cp.fork('child.js');
const child2 = cp.fork('child.js');

// Open up the server object and send the handle
const server = require('net').createServer();
server.on('connection', function(socket) {
    socket.end('handled by parent\n');
});
server.listen(1337, function() {
    child1.send('server', server);
    child2.send('server', server);
});

// child.js
process.on('message', function(m, server) {
    if (m === 'server') {
        server.on('connection', function(socket) {
            socket.end('handled by child, pid is ' + process.pid + '\n');
        })
    }
})
```

再用 `curl` 测试我们的服务，如下所有

```js
curl 'http://127.0.0.1:1337/' //handled by child, pid is 74734
curl 'http://127.0.0.1:1337/' //handled by child, pid is 74734
curl 'http://127.0.0.1:1337/' //handled by child, pid is 74735
```

测试的结果是每次出现的结果都可能不同，结果可能被父进程处理，也可能被不同的子进程处理。并且这是在 TCP 层面上完成的事情，我们尝试将其转化到HTTP层面来试试。

对于主进程而言，我们可以让它更轻量一点，将服务器句柄发送给子进程之后，就可以关掉服务器的监听，让子进程来处理请求。

对上述的代码进行改造

```js
// parent.js
const cp = require('child_process');
const child1 = cp.fork('child.js');
const child2 = cp.fork('child.js');

// Open up the server object and send the handle
const server = require('net').createServer();
server.listen(1337, function() {
    child1.send('server', server);
    child2.send('server', server);
    // 关掉
    server.close();
});

// child.js
const http = require('http');
const server = http.createServer(function(req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('handled by child, pid is ' + process.pid + '\n');
})

process.on('message', function(m, tcp) {
    if (m === 'server') {
        tcp.on('connection', function(socket) {
            server.emit('connection', socket);
        })
    }
})
```

这样一来，所有的请求都是由子进程来处理了。整个过程中，服务的过程发生了一次改变

![](../assets/imgs/process/master-slave-1.png)

主进程发送完句柄并关闭监听之后，成为了如下图的结构

![](../assets/imgs/process/master-slave-2.png)

这样，通过代理，多个子进程就可以同时监听相同的端口，再也没有不通过代理，同时开始多个子进程监听相同端口出现的 `EADDRINUSE` 异常发生了。

#### 句柄发送与还原

1. 句柄发送和我们直接将服务器对象发给子进程有什么区别
2. 是否是真的将服务器对象发送给了子进程
3. 为什么它可以发送到多个子进程中
4. 发送给子进程为什么父进程还存在这个对象

`send()` 方法在将消息发送到IPC管道前，将消息组装成两个对象，一个参数是handle，另一个是 `message` ，`message` 参数如下所示

```
{
	cmd: 'NODE_HANDLE',
	type: 'net.Server',
	msg: message
}
```

发送到IPC管道中的实际上时我们要发送的句柄文件描述符，文件描述符实际上时一个整数值。这个`message` 对象在写入到 IPC管道时也会通过`JSON.stringify()` 进行序列化。所以最终发送到IPC通道中的信息都是字符串，`send()` 方法能发送消息和句柄并不意味着它能发送任意对象。

连接了IPC通道的子进程可以读取到父进程发来的消息，将字符串通过`JSON.parse()` 解析还原对象后，才触发`message` 事件将消息体传递给应用层使用。在这个过程中，消息对象还要被进行过滤处理，`message.cmd` 的值如果以`NODE_` 为前缀，它将响应一个内部事件`internalMessage` ，如果`message.cmd` 的值为`NODE_HANDLE`，它将取出`message.type` 的值和得到的文件描述符一起还原出一个对应的对象。

整个过程示例如下

![](../assets/imgs/process/handle-send.png) 

以发送的TCP服务器句柄为例，子进程收到消息后的还原过程如下所示：

```js
function(message, handle, emit) {
  var self = this;
  
  var server = new net.Server();
  server.listen(handle, function() {
    emit(server);
  })
}
```

上面的代码中，子进程根据`message.type` 创建对应TCP服务器对象，然后监听到文件描述符上。由于底层细节不被应用层感知，所以在子进程中，开发者会有一种服务器就是从父进程中直接传递过来的错觉。值得注意的是，Node 进程之间只有消息传递，不会真正传递对象，这种错觉是抽象封装的结果。

#### 端口共同监听

为何通过发送句柄后，多个进程可以监听到相同的端口而不引起`EADDRINUSE` 异常。答案就是，我们在独立启动的进程中，TCP服务器端`socket` 套接字的文件描述符并不相同，导致监听到相同的端口会抛出异常。

Node 底层对每个端口监听都设置了`SO_REUSERADDR` 选项，这个选项的涵义是不同进程可以就相同的网卡和端口进行监听，这个服务器端套接字可以被不同的进程复用。

由于独立启动的进程互相之间并不知道文件描述符，所以监听相同的端口时就会失败。但是对于`send()` 发送的句柄还原出来的服务而言，他们的文件描述符是相同的，所以监听相同端口不会引起异常。

多个应用监听相同端口时，文件描述符同一时间只能被某个进程所用。换言之就是网络请求向服务器端发送时，只有一个幸运的线程能够抢到连接，也就是说只有他能为这个请求进行服务。这些进程服务是**抢占式** 的

## 集群稳定

使用`child_process` 模块在单机上搭建Node集群，能够在多核CPU的环境下，让 Node 进程能够充分利用资源。

但是还有一些细节需要考虑

1. 性能问题
2. 多个工作进程的存活状态管理
3. 工作进程的平滑重启
4. 配置或者静态数据的动态重新载入

虽然我们创建了很多工作进程，但每个工作进程依然是在单线程上执行的，它的稳定性还不能得到完全的保障。



## 参考

- [深入浅出NodeJS 第九章 - 玩转进程]() 
- [cluster模块概览](https://github.com/chyingp/nodejs-learning-guide/blob/master/模块/cluster.md)

