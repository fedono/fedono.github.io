38 

style > id > class > tagName

flex 里面每个 item 都是 BFC 



下面这种会有问题，因为操作了 children

![image-20210507110433182](/Users/mac/Library/Application Support/typora-user-images/image-20210507110433182.png)





所以需要使用下面这种

![image-20210507110505993](/Users/mac/Library/Application Support/typora-user-images/image-20210507110505993.png)



正常操作都是操作 Node

那么在需要操作Element的时候，也同样提供了对应的方法如`parent, children, firstElementChild` 等 

> 在获取到 childNodes 的时候，如果没有转成数组，那么即使是复制了一份操作后，还是会影响到原来的 childNodes 

![image-20210507091831232](/Users/mac/Library/Application Support/typora-user-images/image-20210507091831232.png)

![image-20210507091840599](/Users/mac/Library/Application Support/typora-user-images/image-20210507091840599.png)



![image-20210507092108955](/Users/mac/Library/Application Support/typora-user-images/image-20210507092108955.png)

在 CSS 动画这一节中，为什么这里还有个 setTimeout 里面设置 style.left/style.top，使用style.transition 不是本身就会移动吗？

![image-20210507080338306](/Users/mac/Library/Application Support/typora-user-images/image-20210507080338306.png)



重排一定会触发重绘

CSS 一旦进行重新计算，那么就一定会触发重排

![image-20210506190606564](/Users/mac/Library/Application Support/typora-user-images/image-20210506190606564.png)

![image-20210506191819538](/Users/mac/Library/Application Support/typora-user-images/image-20210506191819538.png)