一直对元素的 clientHeight / offsetHeight / scrollHeight / innerHeight 这些搞不懂，有次滴滴面试的时候也没有写出来如何获取屏幕的宽高啊，一直使用的 `document.body.clientHeight` 导致一直获取的是0



为什么在页面没有元素的时候，使用 `document.body.clientHeight` 获取的高度是0，但是使用`document.documentElement.clientHeight` 确实屏幕的高度？

而且还可以使用 `window.innerHeight` 来获取屏幕的高度

document.body 获取到的是html 中的 body

document.documentElement 获取到的是 html

在页面没有元素的时候，切换到Chrome调试工具中的 Elements 面板，选中 html 和 body ，在页面区域显示都是同一块区域啊，而且都很小啊，所以还是为什么使用 document.documentElement.clientHeight 能够获取到屏幕的宽度啊

还是在 Elements 面板，选中后，在页面区域显示的，其实不是 clientHeight 的高度？

哦确实不是，是他们的 offsetHeight，打印出他们的 offsetHeight 就能看出来了

卧槽怎么回事，offsetHeight 难道不是 clientHeight + border 的宽度吗？咋回事啊？

为啥 document.documentElement.offsetHeight 都是8，但是 document.documentElement.clientHeight 却是 1001，能够获取到屏幕的高度？？？

在mdn 中有一篇文档[Determining the dimensions of elements](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model/Determining_the_dimensions_of_elements#how_much_room_does_it_use_up.3f)中写了  

- offsetHeight 是 `including the width of the visible content, scrollbars (if any), padding, and border`  也就是元素实际占据的大小

  How much room does it use up

- clientHeight 是 ` including padding but not including the border, margins, or scrollbars` 

  how much space the actual displayed content takes up

那么，如果需要日常在元素内部有滚动的时候，获取到元素的宽高为 `scrollWidth / scrollHeight` ，计算到 `scrollTop` 就是滚动的距离

```js
element.scrollHeight - element.scrollTop === element.clientHeight
```





- `scrollHeight `的值等于该元素在不使用滚动条的情况下为了适应视口中所用内容所需的最小高度。 没有垂直滚动条的情况下，scrollHeight值与元素视图填充所有内容所需要的最小值[`clientHeight`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/clientHeight)相同。包括元素的padding

  也就是 scrollHeight 是包含了当前

- `offsetHeight`  元素的offsetHeight是一种元素CSS高度的衡量标准，包括元素的边框、内边距和元素的水平滚动条（如果存在且渲染的话），不包含:before或:after等伪类元素的高度。

  如果元素被隐藏（例如 元素或者元素的祖先之一的元素的style.display被设置为none），则返回0

- 

![Image:Dimensions-client.png](https://developer.mozilla.org/@api/deki/files/185/=Dimensions-client.png)

![Image:Dimensions-offset.png](https://developer.mozilla.org/@api/deki/files/186/=Dimensions-offset.png)



关于 window.innerWidth 和 window.outerWidth

比如在没有开启控制台的时候，包含的 innerWidth 是和 outerWidth 是一致的，开启来控制台之后，outerWidth 就包含了控制台的宽度了，因为控制台也在 window 中嘛