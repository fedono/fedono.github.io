## 理解 HTTP

http 是应用层的通信，也就是我们日常在浏览器中调用的http请求，这时候可以看到请求的头和数据，以及返回的头和数据

为了建立更好的性能，所以做了缓存

为了

## 理解 TCP / UDP

首先要理解概念，就是 TCP 做了什么事情，TCP 包含了哪些东西，所以需要完善一些概念上的东西

如何传输数据，如何达到性能最优，所以有了滑动窗口，超时重传，流量控制和拥塞控制这些来保证 TCP 的可靠性交付

TCP 报文 (Segment)，包括首部和数据部分。

![img](https://bluesun-1252625244.cos.ap-guangzhou.myqcloud.com/post/understand-tcp-udp/3A60080FBC8767DB575C2D2919097613.png)

​																					报文结构

而 TCP 的全部功能都体现在它首部中各字段的作用，只有弄清 TCP 首部各字段的作用才能掌握 TCP 的工作原理。 TCP 报文段首部的前20个字节是固定的，后面有 4N 字节是根据需要而增加的。 下图是把 TCP 报文中的首部放大来看。

![img](https://bluesun-1252625244.cos.ap-guangzhou.myqcloud.com/post/understand-tcp-udp/CFC6314E4B2FD039C450821D946E93E2.png)

​																				报文首部结构.png

TCP 的首部包括以下内容：

1. 源端口 source port    
2. 目的端口 destination port    
3. 序号 sequence number    
4. 确认号 acknowledgment number    
5. 数据偏移 offset    
6. 保留 reserved    
7. 标志位 tcp flags    
8. 窗口大小 window size    
9. 检验和 checksum    
10. 紧急指针 urgent pointer    
11. 选项 tcp options    

>[各个字段的意义和作用](https://jerryc8080.gitbook.io/understand-tcp-and-udp/chapter2)

- [理解 TCP 和 UDP](https://jerryc8080.gitbook.io/understand-tcp-and-udp/chapter7)

## 理解 IP

最常见的就是 IP 地址了，也就是在这里定义了地址，使用 TCP 传输的时候，头部是需要有 IP 地址的

也就是需要在这里搞清楚 IP 地址和物理地区的区别







## 参考

- [计算机网络 第七版](https://book.douban.com/subject/26960678/) 

  这一班介绍了各个层的功能，很详细