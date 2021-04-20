node 在面试中会问到的点

- Node 中引入模块的步骤

  - 路径分析
  - 文件定位
  - 代码编译
  - 加入内存

- JS 的事件循环和 Node 的事件循环，是在Node的哪个版本进行抹平的

- JS 的垃圾回收和 Node 的垃圾回收  

- Buffer 

- stream 

  - 有哪些流的类型

    - readable 如 fs.createReadStream()
  - writeable  如 fs.createWriteStream()
    
    - duplex 可读又可写的流  如 net.Socket
  - transform 在读写过程中可以修改或转换数据的 `Duplex` 流，例如 [`zlib.createDeflate()`](http://nodejs.cn/api/zlib.html#zlib_zlib_createdeflate_options) 
    
  - stream 有哪些事件类型，具体用在什么场景下

    - close：当流或其底层资源（比如文件描述符）被关闭时触发 `'close'` 事件

    - ##### data：当流将数据块传送给消费者后触发

    - end: 当流中没有数据可供消费时触发。

    - error: 可能随时由 `Readable` 实现触发。 通常，如果底层的流由于底层内部的故障而无法生成数据，或者流的实现尝试推送无效的数据块，则可能会发生这种情况。监听器回调将会传入一个 `Error` 对象

    - pause: 当调用 [`stream.pause()`](http://nodejs.cn/api/stream.html#stream_readable_pause) 并且 `readsFlowing` 不为 `false` 时，就会触发 `'pause'` 事件。
    - readable: 当有数据可从流中读取时，就会触发 `'readable'` 事件

- 进程与线程（child_process / cluster)

  - 如何监控某个进程的内存和 CPU

  - 讲解下 node 的线程模型

    要理解 node 的线程模型，首先要了解 node 的架构，node 的底层是 `libuv`，`v8` 两大支架及一些较小的依赖如 `node12` 中引进来更快的 `llhttp`，`c-ares`，`icu-data`，`zlib`等。因此要学好 `node`，则要了解 `libuv` 以及 `v8` 两大块，其一 `libuv` 与事件循环息息相关，无时无刻体现在你的代码中，而 `v8` 与 cpu/memory/gc 相关，无时无刻体现在你的线上问题排查中。关于线程更多的问题，可以参考 [浅析 Node 进程与线程](https://mp.weixin.qq.com/s?__biz=MjM5NTk4MDA1MA==&mid=2458073467&idx=1&sn=e23e2bccaf65db1a07486ec66878b170&chksm=b187af0686f026100b46f1dec7aef21e4c3b010a7fb53ebca9be30363fca11bff2c6e0f22569#rd)

  - 进程间的通信

    - 《深入浅出Node.js》中第9.2节 多进程架构 
    - [Nodejs进程间通信](http://www.ayqy.net/blog/nodejs%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1/)
      - 使用 child_process 创建的父子进程，可以使用 send / on('message') 事件进行发送数据和接收数据，这是借助内置的 IPC 机制通信
      - 

  - 什么是IO多路复用

    - IO 多路复用是什么意思？ - 罗志宇的回答 - 知乎 https://www.zhihu.com/question/32163005/answer/55772739

  - libuv 是干什么的 

    > the C library that implements the Node.js event loop and all of the asynchronous behaviors of the platform
    >

- 你们的 node 的服务端应用如何部署

- Node 中的接口如何优化

  - 

- 部署 node 时如何充分利用服务器的多核

  - 比如用node 的 cluster，用 k8s 也能部分利用多核性能

- 你们有没有对服务端的异常进行监控

  - 比如用 sentry 监控异常，elk 打日志，prometheus 监控性能并用 alertmanager 报警，再写一个webhook到钉钉

- 那你们在线上出现问题时如何在应用层面监控 cpu 和 memory 的信息

  - 虽然线上出现过问题，但这个确实不清楚。`cpu` 和 `heapdump`

- 如何查看一个 node 的服务端应用的内存和CPU

  - ps / pidstat

- 当服务端的内存发生了 OOM 问题如何排查

  - 比如看 promethues，查看监控的突然高峰，看日志那段时候发生了什么，看有没有提交代码