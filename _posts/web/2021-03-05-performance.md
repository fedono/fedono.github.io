---
layout: post 
title: "前端性能优化" 
author: "fedono"
---



## 页面性能优化的所有指标

- FP

  > First Paing 首次绘制 

- FCP

  > First Contentful Paint 首次内容绘制

- FMP

  > First Meaningful Paing 首次

- TTI

  > Time To Interactive 开始与用户进行交互的时间

- DOMContentLoaded

- LCP

  > Largest Contentful Paint: measures loading performance. To provide a good user experience, LCP should occur within 2.5 seconds of when the page first starts loading.

- FID

  > First Input Delay: measures interactivity. To provide a good user experience, pages should have a FID of less then milliseconds.

- CLS

  > Cumulative Layout Shift: measures visual stability. To provide a good user experience, pages should maintain a CLS of less then 0.1.



## 如何性能优化

- 常见的优化手段

  - 做优化首先要有目的，即你做优化是为了什么，是把某个关键的指标提高吗？
  - 使用页面性能检测工具 insights

- 只请求当前需要的资源

  - 异步加载
  - 懒加载
  - polyfill 的优化 https://polyfill.io/v3/url-builder/ 不用在使用 webpack 编译时加载所有的 polyfill，而是选择性的加载需要的 polyfill

- 缩减资源体积

  - 打包压缩
  - 使用 gzip
  - 图片格式优化、压缩、根据屏幕分辨率展示不同分辨率的图片
  - 尽量控制 cookie 的大小

- 时序优化

  - JS 中的 Promise.all

  - 使用服务端渲染 SSR 

  - 使用 prefetch/prerender/preload

    ```html
    <link rel="dns-prefetch" href="xxx" />
    <link rel="preconnect" href="xxx" />
    <link rel="preload" as="image" href="xxx" />
    ```

- 合理使用缓存

  - 使用 CDN
  - HTTP 缓存
  - localStorage, sessionStorage

- 雅虎的35条军规

  ![img](../assets/imgs/web-performance/yahoo.png)



## 问题

- 什么是 CRP，即关键渲染路径(Critical Rendering Path) ? 如何优化 ?

  关键渲染路径是浏览器将 HTML CSS JavaScript 转换为在屏幕上呈现的像素内容所经历的一系列步骤。也就是我们上面说的浏览器渲染流程。

  为尽快完成首次渲染,我们需要最大限度减小以下三种可变因素:

  - 关键资源的数量: 可能阻止网页首次渲染的资源。
  - 关键路径长度: 获取所有关键资源所需的往返次数或总时间。
  - 关键字节: 实现网页首次渲染所需的总字节数,等同于所有关键资源传送文件大小的总和。



通过 navigation 中的参数来计算当前渲染页面的时间 | [来源](https://www.w3.org/TR/navigation-timing/)

![img](../assets/imgs/web-performance/navigation.png)

有个[岳鹰全景监控](https://yueying-docs.effirst.com/) 的监控工具，监控了前端的一些性能，是根据上图来定义的性能指标，或许有一点参考性吧

> 其实看看这种监控工具的重点参数确实也是很重要
>
> 因为来做这些监控工具的，至少是专业的，来分析某些要点是需要监控的
>
> 那时候字节也是问你，在这些监控工具中，你关心哪些性能
>
> 参考 [岳鹰全景监控 - 页面性能](https://yueying-docs.effirst.com/guide-perf.html) 也可以看看其他竞品监控的性能

### 指标说明 - [来源](https://yueying-docs.effirst.com/faq-perf.html)

| 指标名称     | 描述                               | 计算方式                                  |
| ------------ | ---------------------------------- | ----------------------------------------- |
| 首字节       | 收到首字节的时间                   | responseStart - fetchStart                |
| DOM Ready    | HTML加载完成时间                   | domContentLoadedEventEnd - fetchStart     |
| 页面完全加载 | 页面完全加载时间                   | loadEventStart - fetchStart               |
| DNS查询      | DNS解析耗时                        | domainLookupEnd - domainLookupStart       |
| TCP连接      | TCP链接耗时                        | connectEnd - connectStart                 |
| 请求响应     | Time to First Byte(TTFB)           | responseStart - requestStart              |
| 内容传输     | 数据传输耗时                       | responseEnd - responseStart               |
| DOM解析      | DOM 解析耗时                       | domInteractive - responseEnd              |
| 资源加载     | 资源加载耗时(页面中同步加载的资源) | loadEventStart - domContentLoadedEventEnd |



## 参考

- [从web-vitals到PerformanceObserver](https://juejin.im/post/6844904158076600334) 

- [First_paint](https://developer.mozilla.org/zh-CN/docs/Glossary/First_paint)

- [从 8 道面试题看浏览器渲染过程与性能优化](https://juejin.im/post/6844904040346681358)

- [Performance audits](https://web.dev/lighthouse-performance/) 

  > 使用控制台中的 lighthouse 来测试网站的性能，其中测试的 FCP/TTI 等指标的性能解析
  >
  > 还有一个是 [insights](https://developers.google.com/speed/pagespeed/insights/) 可以查看PC和mobile端各自的得分

- [Web Performance](https://developer.mozilla.org/en-US/docs/Web/Performance) 

  > MDN  中介绍关于 web performance 相关指标与解析
  
- [前端性能优化之雅虎35条军规](https://juejin.cn/post/6844903657318645767) | [雅虎原版](https://developer.yahoo.com/performance/rules.html?guccounter=1)

