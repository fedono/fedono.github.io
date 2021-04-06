# React 和 Vue 的不同

1. 首先最大的对于开发者来说就是写法上的不同，vue 把 js 和 css 还有 html 都放在一个文件中了，而 React 是分开的，也就是在 React 中会更强调与把CSS 和 JS 分开

2. 设计理念上的不同，React 会把很多的控制权交给开发者，如是否更新，所以`react` 增加了`pureComponent` ，以及在 `componentShouldUpdate` 来精确控制我们是否需要更新，而 vue 本身就使用了 getter/setter 以及一些函数的劫持，能精确知道数据变化，不需要特别的优化就能达到很好的性能

   > 为什么 React 不精确监听数据变化呢？这是因为 Vue 和 React 设计理念上的区别，Vue 使用的是可变数据，而React更强调数据的不可变。

3. vue 中使用模板来生成 html ，`React` 使用了 `jsx` 的形式，个人感觉还是写 `jsx` 的形式会好一点



## 参考

1. [Vue和React区别](https://juejin.im/post/5b8b56e3f265da434c1f5f76) 

