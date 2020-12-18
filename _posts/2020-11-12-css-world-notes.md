- ·`img` 元素在有`src` 属性时，设置`::before/::after` 是无效的，所以可以利用该特性来设置在图片没有加载时显示问题如`content:attr(alt)` ，一旦`src` 属性有值后，就不再显示了。
- `<span>`元素默认是 inline 水平，但是一 旦设置 `position:absolute`，其 `display` 计算值就变成了 `block` 

### 两端对齐

我们还可以利用 margin 改变元素尺寸的特性来实现两端对齐布局效
 果。列表是我们 Web 开发中是非常常见的，一般都是通过循环遍历呈现出 来的，也就是实际上每个列表的 HTML 样式都是一致的。现在有这样一个 需求:列表块两端对齐，一行显示 3 个，中间有 2 个 20 像素的间隙。假如我们使用浮动来实现， CSS 代码可能是下面这样:

```css
li {
  float: left;
  width: 100px;
  margin-right: 20px; 
}
```

此时就遇到了一个问题，即最右侧永远有个 20 像素的间隙，无法完美实现两端对齐，如 图 4-51 所示。

![image-20201115101039004](../assets/imgs/css-world/margin-00.png)

如果不考虑 IE8，我们可以使用 CSS3 的 nth-of-type 选择器:

```css
li:nth-of-type(3n) { 
  margin-right: 0;
}
```

但如果需要兼容 IE8 那么 nth-of-type 就无能为力了。要么专门使用 JavaScript 打个补

丁，要么列表 HTML 输出的时候给符合 3n 的<li>标签加个类名。例如.li-third:

```css
.li-third { 
  margin-right: 0;
}
```

然而这种技术选型，需要 HTML 逻辑和 CSS 样式相互配合才能生效，相比纯 CSS 控制而 言后期风险和维护成本提高了一倍，那有没有更好的实现方法呢?

有!我们可以通过给父容器添加 margin 属性，增加容器的可用宽度来实现。

```css
ul {
   margin-right:-20px;
}
ul > li {
  float: left; 
  width: 100px; 
  margin-right: 20px;
}
```

此时<ul>的宽度就相当于 100%+20px，于是，第 3n 的<li>标签的 margin-right: 20px 就多了 20 像素的使用空间，正好列表的右边缘就是父级<ul>容器 100%宽度位置，两端 对齐效果就此实现了



这里使用了 max-width 而不是 width，因此如果视口宽度小于 1080px 的话，内层容器就 能缩小到 1080px 以下。换句话说，在小视口上，内层容器会填满屏幕，在大视口上，它会扩展 到 1080px。这种方式能有效避免在小屏幕上出现水平滚动条。