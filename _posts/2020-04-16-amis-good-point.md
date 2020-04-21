---
layout: post
title:  "AMis 的亮点”
author: "fedono"
---

## 谈谈 AMis 设计中的亮点

1. 一个是 `render` 方法，这个`render` 方法不是 `component` 组件中自带的 `render` 的方法，而是 `AMis` 中一个在 `factory ` 中的方法，这个方法是可以获取到 `schema` 来渲染组件的方法，也就是在任意组件中，可以封装 `schema `，然后调用从 `factory` 中使用 `props` 传下来的 `render` 方法，就可以渲染出其他的组件，完全不需要在组件的顶部使用 `import ` 来调用其他的组件进来实现渲染，非常的方便

2. 渲染组件的方式，也就是组织组件的方式

   普通的组织组件的方式就是通过 `import` 来导入，这是非常方便也非常直观的方式，`AMis ` 使用 `path` 来获取到对应组件的方式是一大亮点，通过在页面中`schema` 定义组件的 `path` ，来找到当前组件进行渲染，而在每个组件中使用 `test` 配置来定义当前组件会在什么情况下会被匹配到
   
3. Scope 能够数据传输到对方组件的方式，这个确实是个亮点，能够执行 target 和 reload 

## 谈谈 AMis 设计中的要点

1. 执行验证的方式（感觉从4开始，就是要点，而不是亮点
2. 能够使用 `name` 就能够直接渲染到相关组件中
   1. 这个应该就是 `mst` 的好处，所有的数据汇总成`tree` 的结构，话说其他的数据管理不都是这样吗？
3. service 的后端获取 schema 动态渲染
4. crud 的多种形式 
5. form的组织方式，包括formItem/control 及 options 公共所有需要获取选项的组件的方式
6. form的布局