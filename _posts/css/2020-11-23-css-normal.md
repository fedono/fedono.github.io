---
layout: post 
title: "通常的CSS面试题" 
author: "fedono"
---

> 感觉这种还是放到 [fe-questions](https://github.com/fedono/fe-questions/issues/2) 中把，这个博客还是主要放一些知识点的总结，fe-questions 那里放一些题目以及解答

## 垂直居中

垂直居中的方法，如果全写出来，有10多种。面试的时候一般都会说比较常用的几种。`flex`、`position + transform`、`position + 负margin`是最常见的三种情况。

```html
<div class="outer">
    <div class="inner"></div>
</div>
```

- **flex**

  ```css
  .outer{
      display: flex;
      justify-content: center;
      align-items: center
  }
  ```

- **position + transform, inner宽高未知**

  ```css
  .outer{
      position:relative;
  }
  .inner{
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%,-50%);
  }
  ```

- **position + 负margin, inner宽高已知**

  ```css
  .outer{
      position: relative;
  }
  .inner{
      width: 100px;
      height: 100px;
      position: absolute;
      left: 50%;
      top: 50%;
      margin-left: -50px;
      margin-top: -50px;
  }
  ```

  

## 清除浮动

- BFC 
- clear: both 

## 实现一个自适应的正方形

- **利用margin或者padding的百分比计算是参照父元素的width属性**

  ```css
  .square {
      width: 10%;
      padding-bottom: 10%; 
      height: 0; // 防止内容撑开多余的高度
      background: red;
  }
  ```

- `vw` 会把视口的宽度平均分为100份

  ```css
  .square {
      width: 10vw;
      height: 10vw;
      background: red;
  }
  ```

## 实现一个三角形

- **利用border属性**

  利用盒模型的 `border` 属性上下左右边框交界处会呈现出平滑的斜线这个特点，通过设置不同的上下左右边框宽度或者颜色即可得到三角形或者梯形。

  ```css
  .triangle {
      height:0;
      width:0;
      border-color:red blue green pink;
      border-style:solid;
      border-width:30px;
  }
  ```

  如果想实现其中的任一个三角形，把其他方向上的 `border-color` 都设置成透明即可。

  ```css
  .triangle {
      height:0;
      width:0;
      border-color:red transparent transparent transparent;
      border-style:solid;
      border-width:30px;
  }
  ```

- **利用CSS3的clip-path属性**

  不了解 `clip-path` 属性的可以先看看 `MDN` 上的介绍：[chip-path](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clip-path)

  ```css
  .triangle {
      width: 30px;
      height: 30px;
      background: red;
      clip-path: polygon(0px 0px, 0px 30px, 30px 0px); // 将坐标(0,0),(0,30),(30,0)连成一个三角形
      transform: rotate(225deg); // 旋转225，变成下三角
  }
  ```




## 参考

- [面试总结 CSS ](https://juejin.cn/post/6844903967843942413)

