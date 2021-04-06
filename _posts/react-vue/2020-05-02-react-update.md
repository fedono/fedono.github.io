React 的三种更新方式

- `ReactDOM.render` || `hydrate`
- setState
- forceUpdate



第三章主要讲创建完更新，到调度的过程，至于怎么调度的，还需要后面再讲

什么事 FiberRoot

- 整个应用起点
- 包含应用挂载的目标节点
- 记录整个应用更新过程的各种信息



## Fiber

Component 本身记录的状态都是在 Fiber 里面，在 Fiber 里面更新完后，Component 才更新，这样就把组件和更新分离了，这样也就能够把 function 的 hooks 加进来了

每一个 ReactElement 对应一个 Fiber 对象

记录节点的各种状态

串联整个应用形成树结构

## Update 

用于记录组件状态的改变

- 存放于 UpdateQueue 中
  - 多个 setState 会存在 UpdateQueue 中，然后批量更新

多个 Update 可以同时存在

数据结构在 `react-16.6.0/packages/react-reconciler/src/ReactUpdateQueue.js` 

```js
export type Update<State> = {
  expirationTime: ExpirationTime,

  // export const UpdateState = 0;
	// export const ReplaceState = 1;
	// export const ForceUpdate = 2;
	// export const CaptureUpdate = 3; 处理异常的时候，react 的 error Boundaries 会触发这个更新来渲染错误信息到页面
  tag: 0 | 1 | 2 | 3,
  // 要渲染的内容
  payload: any, 
  callback: (() => mixed) | null,

  // 下一个要更新的队列
  next: Update<State> | null,
  nextEffect: Update<State> | null,
};
```

所有的 `Update` 都是通过 `UpdateQueue` 来管理的，所以这里有 `firstUpdate` 来记录第一个需要更新的 `Update`， 根据 `Update` 中的 `next` 找到下一个要更新的 `Update` ，然后就可以一直更新下去了

```js
export type UpdateQueue<State> = {
  // 每次计算一次 `Update` 时更新这个baseState， 这样下次的`Update` 就使用这个baseState 作为state 来计算，而不是每次都是根据第一次的 `Update` 中的 state 来计算
  baseState: State,

  firstUpdate: Update<State> | null,
  lastUpdate: Update<State> | null,

  // 有错误捕获的时候的 Update
  firstCapturedUpdate: Update<State> | null,
  lastCapturedUpdate: Update<State> | null,

  firstEffect: Update<State> | null,
  lastEffect: Update<State> | null,

  firstCapturedEffect: Update<State> | null,
  lastCapturedEffect: Update<State> | null,
};
```

## expirationTime

如何计算这个时间值，来判断是否要更新组件的，包括了同步更新时候的这个时间值和异步更新的时候，这个时间值要怎么计算

在 `react-16.6.0/packages/react-reconciler/src/ReactFiberReconciler.js` 中有使用到非常多的 `ExpirationTime` ，所有的 `ExpirationTime` 都在 `react-16.6.0/packages/react-reconciler/src/ReactFiberExpirationTime.js` 

- `computeExpirationBucket` 计算多次更新 `setState` 时会采取的计算的方式

  ```js
  (
    MAGIC_NUMBER_OFFSET +
    ceiling(
      currentTime - MAGIC_NUMBER_OFFSET + expirationInMs / UNIT_SIZE,
      bucketSizeMs / UNIT_SIZE,
    )
  )
  
  // 把常量替换过来就是
  ((((currentTime - 2 + 5000 / 10) / 25) | 0) + 1) * 25
  ```

  `| 0`的作用是取整数 也就是  `((currentTime - 2 + 5000 / 10) / 25)` 计算出有多少个 25，把不够 25 的余数去掉，然后 + 1，再 * 25 ，就相当于是 `ceil` 的运算了，不够一个 25 的，需要计算成 一个 25

  

  >  React 这么设计抹相当于抹平了`25ms`内计算过期时间的误差，这么做也许是为了让非常相近的两次更新得到相同的`expirationTime`，然后在一次更新中完成，相当于一个自动的`batchedUpdates`。



## setState & forceUpdate

