## vue 

- keep-alive 的实现原理是什么，使用了什么算法

   `<keep-alive>` 组件是一个抽象组件，它的实现通过自定义 `render` 函数并且利用了插槽，并且知道了 `<keep-alive>` 缓存 `vnode`，了解组件包裹的子元素——也就是插槽是如何做更新的。且在 `patch` 过程中对于已缓存的组件不会执行 `mounted`，所以不会有一般的组件的生命周期函数但是又提供了 `activated` 和 `deactivated` 钩子函数。另外我们还知道了 `<keep-alive>` 的 `props` 除了 `include` 和 `exclude` 还有文档中没有提到的 `max`，它能控制我们缓存的个数。

- react 中的 useEffect / useLayerEffect 的区别是什么
  - 生命周期会有不一致
  - 哪个的生命周期更前
  - useEffect 实在渲染器，是异步的，可能会出现闪动的情况
  - useLayerEffect 这里相当于 didMount，是同步的，所以日常如果使用useEffect 出现了闪动，就用这个
- 合成事件
  - 先冒泡再捕获
  - 浏览器的是先捕获，然后再冒泡

- 移动端的设置

  ```HTML
   <meta name="viewport" content="width=device-width">
  ```





