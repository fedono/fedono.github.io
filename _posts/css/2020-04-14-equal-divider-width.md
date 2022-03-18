---
layout: post
title:  "等分布局"
author: "fedono"
---

横向等分布局是一个很常见的需求，按照一般的思路，我们可以使用百分比宽度来解决，我们参考以下代码：

```html
<div class="outer">
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
</div>
.inner {
    width:33.33%;
    height:300px;
    display:inline-block;
    outline:solid 1px blue; // outline 轮廓不占据空间，它们被描绘于内容之上
}
```

在这段HTML代码中，我们放了三个div，用CSS给它们指定了百分比宽度，并且指定为inline-block。

但是这段代码执行之后，效果跟我们预期不同，我们可以发现，每个div并非紧挨，中间有空白，这是因为我们为了代码格式加入的换行和空格被HTML当作空格文本，跟inline盒混排了的缘故。

解决方案是修改HTML代码，去掉空格和换行：

```html
<div class="outer"><div class="inner"></div><div class="inner"></div><div class="inner"></div></div>
```

但是这样做影响了源代码的可读性，一个变通的方案是，改变outer中的字号为0。

```html
.inner {
    width:33.33%;
    height:300px;
    display:inline-block;
    outline:solid 1px blue;
    font-size:30px;
}
.outer {
    font-size:0;
}
```

除了使用inline-block，float也可以实现类似的效果，但是float元素只能做顶对齐，不如inline-block灵活。



附：以前一直不知道 display:inline-block 布局会有换行符空格引起的间隙问题 

 - [浮动布局-基于display:inline-block的列表布局]([https://www.zhangxinxu.com/wordpress/2010/11/%E6%8B%9C%E6%8B%9C%E4%BA%86%E6%B5%AE%E5%8A%A8%E5%B8%83%E5%B1%80-%E5%9F%BA%E4%BA%8Edisplayinline-block%E7%9A%84%E5%88%97%E8%A1%A8%E5%B8%83%E5%B1%80/](https://www.zhangxinxu.com/wordpress/2010/11/拜拜了浮动布局-基于displayinline-block的列表布局/)) 
 - [CSS常见布局](https://juejin.im/post/5cc851eb5188252dd765eec4) 

