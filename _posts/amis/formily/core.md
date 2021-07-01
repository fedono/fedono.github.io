感觉 core 最大的要点就是建立了各种 models

```JS
// 建立 LifeCycle，相当于一个set类型，有 add/remove/has 这些方法
export * from './Heart'
// 就是一个 event listeners 啊，建立每个 type 的 listeners，然后调用就是
export * from './LifeCycle'
/* 管理 整个 form 中 每个 field 的 state */
export * from './Graph'
/* query 不知道是要干啥的 */
export * from './Query'
/* form 的所有方法，初始化、验证、获取所有 fields 的 values */
export * from './Form'
/* 这个是单个的 formItem 的所有的方法 */
export * from './Field'
/* 多个 field 的 push/pop/remove */
export * from './ArrayField'
export * from './ObjectField'
export * from './VoidField'
```

Json-schema 里面到底是要返回啥啊，怎感觉都是细枝末节啊

react 这里是真正的渲染各个节点的

- 通过递归的调用，来达到实现 json-schema 的渲染的作用



老实说 formily 2 的代码组织结构要比 formily 好很多，现在结构比较清晰明了了

需要好好看看 formily 现在做了哪些方面的改进，感觉这些代码都比较平常啊，没看到有多少的亮点

