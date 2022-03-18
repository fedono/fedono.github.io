

  > - 什么场景会导致这种攻击
  > - 如何做验证是个关键点，有什么方案，有什么利弊

  

## 防护策略

CSRF通常从第三方网站发起，被攻击的网站无法防止攻击发生，只能通过增强自己网站针对CSRF的防护能力来提升安全性。

上文中讲了CSRF的两个特点：

- CSRF（通常）发生在第三方域名。
- CSRF攻击者不能获取到Cookie等信息，只是使用。

针对这两点，我们可以专门制定防护策略，如下：

- 阻止不明外域的访问
  - 同源检测
  - Samesite Cookie
- 提交时要求附加本域才能获取的信息
  - CSRF Token
  - 双重Cookie验证

以下我们对各种防护方法做详细说明。

- 同源检测
  - 检测 origin header 和 referer
- CSRF Token
  - 在提交表单中创建token 与 cookie 中的 token 作对比
- 双重cookie 
  - 请求的时候从 cookie 中获取一个唯一值到请求的链接中，服务端将这个值与cookie 的值进行对比
- 设置 Samesite Cookie属性 
  - csrf  本质上还是第三方获取到网站的cookie 然后发起请求，这时候设置了 `Samesite=Strict` 后，第三方网站就无法获取到当前网站的 cookie ，这样就不会产生 CSRF 的问题



## 参考
- [如何防止CSRF攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)
- [如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)
