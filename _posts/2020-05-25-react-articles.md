- 不推荐在 componentwillmount 里最获取数据的操作呢

  - [关于React v16.3 新生命周期](https://juejin.im/post/5aca20c96fb9a028d700e1ce)

    有一种错觉，在componentWillMount请求的数据在render就能拿到，但其实render在willMount之后几乎是马上就被调用，根本等不到数据回来，同样需要render一次“加载中”的空数据状态，所以在didMount去取数据几乎不会产生影响。

  - [全面了解 React 新功能: Suspense 和 Hooks](https://segmentfault.com/a/1190000017483690)

    有了Fiber 之后， react 的渲染过程不再是一旦开始就不能终止的模式了， 而是划分成为了两个过程： 第一阶段和第二阶段， 也就是官网所谓的 `render phase` and `commit phase`。

    在 Render phase 中, React Fiber会找出需要更新哪些DOM，这个阶段是可以被打断的， 而到了第二阶段commit phase， 就一鼓作气把DOM更新完，绝不会被打断。

    **两个阶段的分界点** 

    这两个阶段， `分界点`是什么呢？

    其实是 `render 函数`。 而且， `render 函数 也是属于 第一阶段 render phase 的`。

    那这两个 phase 包含的的生命周期函数有哪些呢？

    `render phase`:

    - componentWillMount
    - componentWillReceiveProps
    - shouldComponentUpdate
    - componentWillUpdate

    `commit phase`:

    - componentDidMount
    - componentDidUpdate
    - componentWillUnmount

    综上也就是在 `componentWillMount` 中有可能被打断，导致后续需要重新进入，会进行多次的调用

  

- 在componentDidMount中添加加事件监听
  
  react只能保证componentDidMount-componentWillUnmount成对出现，componentWillMount可以被打断或调用多次，因此无法保证事件监听能在unmount的时候被成功卸载，可能会引起内存泄露
  
  pureComponent 好在哪？ 
  
- - 不应等是 pureComponent 里面会自动对比之前的 prevProps 和 nextProps 吗，然后再决定是否更新，但是在 [PureComponent的作用及一些使用陷阱](https://www.jianshu.com/p/33cda0dc316a) 中，说`shouldComponentUpdate` 会始终返回 `true` ，那还要 `PureComponent` 还有什么区别？



- React 新旧生命周期的不同 

  [关于React v16.3 新生命周期](https://juejin.im/post/5aca20c96fb9a028d700e1ce) 

	> 旧的生命周期十分完整，基本可以捕捉到组件更新的每一个state/props/ref，没有什从逻辑上的毛病。
	>
	> 但是架不住官方自己搞事情，react打算在17版本推出新的Async Rendering，提出一种可被打断的生命周期，而可以被打断的阶段正是实际dom挂载之前的虚拟dom构建阶段，也就是要被去掉的三个生命周期。
	>
	> 生命周期一旦被打断，下次恢复的时候又会再跑一次之前的生命周期，因此componentWillMount，componentWillReceiveProps， componentWillUpdate都不能保证只在挂载/拿到props/状态变化的时候刷新一次了，所以这三个方法被标记为不安全。

- `UNSAFE_componentWillMount`

  为什么说此方法是服务端渲染唯一会调用的声明周期函数

  [文档](https://zh-hans.reactjs.org/docs/react-component.html#unsafe_componentwillmount) 

## 参考

- [全面了解 React 新功能: Suspense 和 Hooks](https://segmentfault.com/a/1190000017483690)

  > 这篇文章写的确实很好，我也是从这里明白了，React Fiber一个更新过程被分为两个阶段： `render phase` and `commit phase`，之前一直不知道有 `commit phase` ，看 `react ` 源码视频时也不知道什么事 `commit` 阶段
  >
  > 还有就是 React 的周期，到底应该怎么用

- [React hooks都是数组，没那么神秘](https://mp.weixin.qq.com/s/KPgGUjMjpTK3fA1Zjw5MNA) 

  为什么不要在 hooks 中使用 if / for ，因为会改变顺序

- react-cache 这个库是用来干啥的？

- [Under-the-hood-ReactJS](https://github.com/xitu/Under-the-hood-ReactJS)

  >  ReactJS 底层揭秘

- [【译】React 是如何区分 Class 和 Function 的 ?](https://zhuanlan.zhihu.com/p/51705609) 

- [漫谈前端性能 突破 React 应用瓶颈](https://zhuanlan.zhihu.com/p/42032897) 