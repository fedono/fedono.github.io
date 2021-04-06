---
layout: post 
title: "Webpack 相关" 
author: "fedono"
---

## 知识点

webpack 的关键点是怎么做的，这些都是重点，如何做到的，也就是为什么要使用 webpack 的原因

- 如何热更新的
- 如何 tree Shaking
- 如何提取公共代码
- 如何按需加载
- 如何 Prepack
- 如何 Scope Hositing 

https://github.com/Advanced-Frontend/Daily-Interview-Question

## 问题

1. webpack是如何实现动态导入的
   1. webpack 提供的 `import()` 和 `require.ensure` 两个 API 来使用该功能  [文档](https://webpack.docschina.org/api/module-methods/#import)
2. 假如有2个团队，一个团队想用另一个团队的一个类库，并且还是想在用到的时候才加进来，怎么办
   1. webpack的externals ，[文档](https://webpack.docschina.org/configuration/externals/#src/components/Sidebar/Sidebar.jsx) 
3. webpack[jsonp] 是怎么做的

## 参考

- [webpack是如何实现动态导入的](https://juejin.im/post/5d26e7d1518825290726f67a) 
- [从 Bundle 文件看 Webpack 模块机制](https://zhuanlan.zhihu.com/p/25954788)

