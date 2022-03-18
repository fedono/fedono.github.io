## 概念

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种。

![img](http://www.ruanyifeng.com/blogimg/asset/2017/bg2017051502.png)

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

> ```markup
> ws://example.com:80/some/path
> ```

## 示例



## 其他

### STOMP 

为什么会有一个叫做 STOMP 的东西？

既然都有了 socket，为什么还会有这种，而且文档中还介绍说是一种这个也是一种文本消息的协议，那 socket 是啥，为啥和 socket 有联系？

> [STOMP](http://stomp.github.com/) is a simple text-orientated messaging protocol

看标题应该就明白了为什么会有 STOMP 这种东西了

> # STOMP Over WebSocket

也就是 STOMP 是更高阶的 websocket，看下面的介绍，也就是 stomp 是一个通信标准，只要实现了这个标准就可以，你可以使用任何语言来实现，只要实现了就能够彼此通信。

> STOMP provides an interoperable wire format so that STOMP clients can communicate with any STOMP message broker to provide easy and widespread messaging interoperability among many languages, platforms and brokers.

[stomp 1.2 标准](https://stomp.github.io/stomp-specification-1.2.html) 

stomp 不仅仅是支持web socket 的，stompjs 的说明如下 

> This library provides a STOMP client for Web browser (using Web Sockets) or node.js applications (either using raw TCP sockets or Web Sockets).



### Sockjs / sockjs-client

有点没明白为什么会有这两个东西，而且看 readme 好像很多还是相同的，这是在搞什么？

好像明白了 sockjs-client 是用在浏览器端的，而 sockjs 其实就是 sockjs-node，是用在服务端的 



## 参考

- [WebSocket 教程](http://www.ruanyifeng.com/blog/2017/05/websocket.html?utm_source=tuicool&utm_medium=referral) 

