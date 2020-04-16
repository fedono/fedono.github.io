---
layout: post
title:  "AMis 的亮点”
author: "fedono"
---

谈谈 AMis 设计中的亮点

一个是 `render` 方法，这个`render` 方法不是 `component` 组件中自带的 `render` 的方法，而是 `AMis` 中一个在 `factory ` 中的方法，这个方法是可以获取到 `schema` 来渲染组件的方法，也就是在任意组件中，可以封装 `schema `，然后调用从 `factory` 中使用 `props` 传下来的 `render` 方法，就可以渲染出其他的组件，完全不需要在组件的顶部使用 `import ` 来调用其他的组件进来实现渲染，非常的方便

