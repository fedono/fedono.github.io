- 想要了解 网络层，先了解HTTP吧，毕竟这个是应用层的东西，更切合我们日常的开发，所以肯定更熟悉

  > HTTP 到底是干什么的，Node 的 net 库是实现 http 的吗？
  >
  > 如果 HTTP 是建立连接的，那么三次握手涉及到 tcp/ip 吗？感觉应该是肯定涉及到的，那么又是如何从 http 到 tcp/ip 来建立连接的呢？

  - 这个应该还要看一下计算机网络了，这个得看一下流量是怎么做交互的

- 第二部分是 TCP/IP ，然后看 HTTP 和 TCP / IP 是如何做的交互的，先把这两个部分搞懂吧，至少流程上搞懂一点吧

- 然后再看以下的层

---

- http 从 0.9到3.0，发生了哪些变化
- 前端如何开启 http2，后端呢？
- HTTPS 增加了什么，为什么是安全的
- HTTP三次握手的过程和四次挥手的过程
- 既然 TCP 是安全可靠的，为什么还需要 UDP 
  - 为什么TCP是安全可靠的
- HTTP 缓存
  - 强缓存
    - expire
    - cache-control
  - 协商缓存
    - last-modified/if-modified-since
    - etag/if-none-match
- 服务器推送
  - 除了有  http2
  - 还有 websocket 



## 参考

- [一文读懂一台计算机是如何把数据发送给另一台计算机的](https://mp.weixin.qq.com/s?__biz=Mzg2NzA4MTkxNQ==&mid=2247485105&idx=1&sn=c045b7599c1637776dd89890cac5a9b5&chksm=ce404d65f937c4736257f0795a0243a5cbe3526d88013074277fa2f44fa809e76f0330796bee&scene=21#wechat_redirect)

- [CP 的超时重传时间是如何计算的](https://sanyuan0704.top/blogs/net/tcp/008.html#%E7%BB%8F%E5%85%B8%E6%96%B9%E6%B3%95) 
