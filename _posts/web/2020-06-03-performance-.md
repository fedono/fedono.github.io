---
layout: post 
title: "性能优化" 
author: "fedono"
---

如果你真的做过多次的性能优化，能够实践很多的性能优化的尝试后，真正能够有效的性能优化的指标还是比较少的

也就是性能指标会有很多歌，但是真正需要关注的性能指标是很少的

- 页面的加载时间 
- 页面加载到用户可交互的时间



## 常见的性能指标

- **First contentful paint (FCP)**: 从页面开始加载，到页面任意内容渲染在屏幕上的时间。(实验属性，现场数据)
- **Largest contentful paint (LCP)**: 从页面加载开始，到页面上最大的文本块或者图片元素渲染在屏幕上的时间。(实验数据，现场数据)
- **First input delay (FID)**: 用户第一个交互行为，比如点击链接、点击按钮等，到浏览器实际响应这次交互的时间。(现场数据)
- **Time to Interactive (TTI)**: 从页面开始加载，到内容呈现，初始js加载完成，再到可以立刻响应用户交互的时间。(实验数据)
- **Total blocking time (TBT)**: 在FCP和TTI之间的时间，即页面可见到可交互的时间，代表了主线程被阻塞无法响应用户交互。（实验数据）
- **Cumulative layout shift (CLS)**: 从页面开始加载，到页面生命周期变为隐藏之间的所有意外的布局变化的累计分数。(实验数据，现场数据)

以上指标可以检测大部分的与用户相关的性能影响，但也不代表全部。

## 自定义指标

有时候你的站点可能需要一些额外的指标来捕获整个性能图。举个例子，LCP的指标，如果最大的文本块或者图片元素不是你的网站的主要内容，这个指标则失去了原本的意义。

为了应对这些情况，Web性能工作小组定制了一些标准的底层API，帮你可以实施自己的自定义指标:

- [User Timing API](https://link.juejin.cn?target=https%3A%2F%2Fw3c.github.io%2Fuser-timing%2F)
- [Long Tasks API](https://link.juejin.cn?target=https%3A%2F%2Fw3c.github.io%2Flongtasks%2F)
- [Element Timing API](https://link.juejin.cn?target=https%3A%2F%2Fwicg.github.io%2Felement-timing%2F)
- [Navigation Timing API](https://link.juejin.cn?target=https%3A%2F%2Fw3c.github.io%2Fnavigation-timing%2F)
- [Resource Timing API](https://link.juejin.cn?target=https%3A%2F%2Fw3c.github.io%2Fresource-timing%2F)
- [Server timing](https://link.juejin.cn?target=https%3A%2F%2Fw3c.github.io%2Fserver-timing%2F)





- 性能优化常见的指标
  - contentLoaded 

- 针对 H5 有哪些规则
  - 预加载，离线加载等
  - 



## 参考

- [前端性能优化之谈谈常见的性能指标及上报策略](https://cloud.tencent.com/developer/article/1623136)
- [雅虎35条军规](https://juejin.im/post/5b73ef38f265da281e048e51#heading-9) 
- [前端性能优化之谈谈常见的性能指标及上报策略](https://mp.weixin.qq.com/s/wDKKj5R8SYm-_75Zn1y30A) 
- [页面加载性能之常见的性能指标](https://juejin.cn/post/6856397971467010061)

