### `React.PureComponent`

`React.PureComponent` 与 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 很相似。两者的区别在于 [`React.Component`](https://zh-hans.reactjs.org/docs/react-api.html#reactcomponent) 并未实现 [`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)，而 `React.PureComponent` 中以浅层对比 prop 和 state 的方式来实现了该函数。

如果赋予 React 组件相同的 props 和 state，`render()` 函数会渲染相同的内容，那么在某些情况下使用 `React.PureComponent` 可提高性能。

### `React.memo`

```
const MyComponent = React.memo(function MyComponent(props) {
  /* 使用 props 渲染 */
});
```

`React.memo` 为[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)。它与 [`React.PureComponent`](https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent) 非常相似，但只适用于函数组件，而不适用 class 组件。

如果你的函数组件在给定相同 props 的情况下渲染相同的结果，那么你可以通过将其包装在 `React.memo` 中调用，以此通过记忆组件渲染结果的方式来提高组件的性能表现。这意味着在这种情况下，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果。

`React.memo` 仅检查 props 变更。如果函数组件被 `React.memo` 包裹，且其实现中拥有 [`useState`](https://zh-hans.reactjs.org/docs/hooks-state.html) 或 [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext) 的 Hook，当 context 发生变化时，它仍会重新渲染。

默认情况下其只会对复杂对象做浅层对比，如果你想要控制对比过程，那么请将自定义的比较函数通过第二个参数传入来实现。

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

- Hooks

  - [Hooks FAQ](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-memoize-calculations) 

    - 关于 `Hooks` 需要知道的

- [React Hooks的体系设计之一 - 分层](https://zhuanlan.zhihu.com/p/106665408)

  - 其他三篇可以一起看一下

- [你可能不需要使用派生 state](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization) 

  - 官方的博客的质量真的很好

- []