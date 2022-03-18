---
layout: post 
title: "CSS 两列/三列 布局" 
author: "fedono"
---

## 两列布局

左边固定，右边自适应

```html
<div class="container">
  <div class="float">
    浮动元素
  </div>
  <div class="main">
    自适应
  </div>
</div>
```

### 使用 flex

```css
contanier{
  width:500px;
  display:flex;
}
.float{
  height:200px;
  background:red;
}
.main{
  background:green;
  height:200px;
  flex-grow:1; // 使用 flex-glow 来占据剩余的空间
}
```

### 使用 float

```css
.float{
  height:100px;
  float:left;
  background:red;
  width:200px;
}
.main{
  background:green;
  height:100px;
  margin-left: 200px;
}
.container{
  width:500px;
}
```

### 使用 BFC

```css
.container{
  width:500px;
}
.float{
  height:200px;
  background:red;
  float:left;
}
.main{
  background:green;
  height:200px;
  overflow:hidden;//触发BFC
}
```

## 三列布局

左右固定宽度，中间自适应

### 使用 flex

```html
<div class="container">
    <div class="left">left</div>
    <div class="main">main</div>
    <div class="right">right</div>
</div>
.container{
    display: flex;
}
.left{
    flex-basis:200px;
    background: green;
}
.main{
    flex: 1;
    background: red;
}
.right{
    flex-basis:200px;
    background: green;
}
```

### **第二种：position + margin**

> 最重要的是之前一直不知道 div 是会默认占据父级元素的全部宽度的，div 是 block 啊，真是突然醒悟。

```html
<div class="container">
    <div class="left">left</div>
    <div class="right">right</div>
    <div class="main">main</div>
</div>
body,html{
    padding: 0;
    margin: 0;
}
.left,.right{
    position: absolute;
    top: 0;
    background: red;
}
.left{
    left: 0;
    width: 200px;
}
.right{
    right: 0;
    width: 200px;
}
.main{
    margin: 0 200px ;
    background: green;
}
```

### **第三种：float + margin**

```html
<div class="container">
    <div class="left">left</div>
    <div class="right">right</div>
    <div class="main">main</div>
</div>
body,html{
    padding:0;
    margin: 0;
}
.left{
    float:left;
    width:200px;
    background:red;
}
.main{
    margin:0 200px;
    background: green;
}
.right{
    float:right;
    width:200px;
    background:red;
}
```



## 参考

- 《深入解析CSS》第二部分 精通布局

