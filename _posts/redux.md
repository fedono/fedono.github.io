- Sagas 负责策划统筹合成和异步操作。

Sagas 使用Generator functions（生成器函数）创建。

> 这个中间件不只是处理异步流。如果需要简化异步控制流，可以容易的使用一些promise中间件的async/await 函数。

这个中间件的目的?

- 一个目的是抽象出 **Effect** （影响）: 等待一个action，触发State更新 (使用分配action给store), 调用远程服务，这些都是不同形式的Effect。Saga使用常见的控制流程(if, while, for, try/catch)去组合这些Effect。
- Saga本身就是Effect。它可以通过选择器和其他Effect组合。它也可以被内部的其他Saga调用，提供丰富功能的子程序和 [结构化程序设计](https://en.wikipedia.org/wiki/Structured_programming)
- Effect可以迭代声明。你可以迭代声明一个被中间件执行的Effect。这使得你在生成器中的运算逻辑完全可测试的。
- 你可以实现复杂的业务逻辑，跨越多个操作(比如用户培训、向导对话框、复杂的游戏规则...)，这不是简单的使用其他Effect中间件表达。



虽然`saga` 的`put` 是同步的代码，但是还是使用了 `yield` ，也就是 `saga` 在控制步骤 

使用`redux` 的 `applyMiddleware` ，中间件的格式都是统一的，所以`saga` 的是

``` js
function sagaMiddleware({ getState, dispatch }) {
    boundRunSaga = runSaga.bind(null, {
      ...options,
      context,
      channel,
      dispatch,
      getState,
      sagaMonitor,
    })

    return next => action => {
      if (sagaMonitor && sagaMonitor.actionDispatched) {
        sagaMonitor.actionDispatched(action)
      }
      const result = next(action) // hit reducers
      channel.put(action)
      return result
    }
  }
```

使用 `saga`  一般都如下这样使用

```js
const sagaMiddleware = createSagaMiddleware()
const store = createStore(rootReducer, applyMiddleware(sagaMiddleware))
sagaMiddleware.run(rootSaga)
```

比较奇怪的是，为什么要执行一个 `sagaMiddleware.run` 这个方法，为什么就不能在 `createSagaMiddleware` 中就直接执行 `run` 方法呢，还要多一步

看到一个 `connect` 的组件方式是这样写

```js
export default connect(
  state => ({
    products: getCartProducts(state),
    total: getTotal(state),
    error: getCheckoutError(state),
    checkoutPending: isCheckoutPending(state),
  }),
  { checkout, removeFromCart },
)(Cart)

// checkout 为 reduce 
export function checkout() {
  return {
    type: CHECKOUT_REQUEST,
  }
}
```

这样写高阶组件也太简洁了吧

- 不太明白为什么所有的`sagas` 在 `sagaMiddleware.run(rootSaga)` 加载完后，就没有在其他文件中看到有用到`sagas` 中的函数，

- 以前好像一直没有明白在使用`redux` 的时候，其中的 `action` 的意思是什么

  一般的用法都是这样 

  ```js
  export const addTodo = text => ({
    type: 'ADD_TODO',
    id: nextTodoId++,
    text
  })
  ```

  也就是 `action` 中配置 `type` 和相应的数据，然后通过 `dispatch` 后，就能到`reduces` 中找到该 `type` 类型，然后就可以将`reduces` 中的数据返回回来

  但其实 `action` 的作用就是把去获取数据，然后`dispatch` 就可以，所以只要遵循了这两点，就可以随便发挥了。

  所以我们可以封装公用的 `fetch` 方法，然后方法调用这个 `fetch` ，然后把 `type` 传给 `fetch` ，在 `fetch` 的成功回调中，`dispatch` 这个 `type` 类型就可以了。



## Redux

`connectAdvanced` 最重要的作用还是`memo` ，也就是提供缓存的作用

redux  是做数据管理的，react-redux 是做数据更新的，那么如何更新呢，就需要监听 `props/state` 的变化来更新组件



```js
function handleFirstCall(firstState, firstOwnProps) {
    state = firstState
    ownProps = firstOwnProps
    stateProps = mapStateToProps(state, ownProps)
    dispatchProps = mapDispatchToProps(dispatch, ownProps)
    mergedProps = mergeProps(stateProps, dispatchProps, ownProps)
    hasRunAtLeastOnce = true
    return mergedProps
  }

const selectorFactoryOptions = {
      ...connectOptions,
      getDisplayName,
      methodName,
      renderCountProp,
      shouldHandleStateChanges,
      storeKey,
      displayName,
      wrappedComponentName,
      WrappedComponent
    }
```

## React-Redux

最重要的就是 `Provider` 和 `connect` 了

### `Provider`

`provider` 是提供一个 `context` ，然后把`store` 传给子元素

```js
return <Context.Provider value={contextValue}>{children}</Context.Provider>
```

每次`store` 和`state` 有变化的时候，就会触发一次更新，同时会触发一次`subscription` 中的`listeners` 执行 `notify `

### `Connect` 

`connect` 是调用 `connectAdvanced` ，所以重点在 `connectAdvanced` 中，`connectAdvanced` 中又是调用 `selectorFactory` 

`mobx` 中也有这种模式

`wrapMapToPropsFunc` 这个函数我是一直都没弄懂

```js
const proxy = function mapToPropsProxy(stateOrDispatch, ownProps) {
      return proxy.dependsOnOwnProps
        ? proxy.mapToProps(stateOrDispatch, ownProps)
        : proxy.mapToProps(stateOrDispatch)
    }
```

这里定义了 `proxy` 方法，然后调用`proxy.mapToProps` 方法

下面第一 `proxy.mapToProps` 方法的时候，竟然内部还调用`proxy` 方法，这不是一直在调用自己吗？

```js
proxy.mapToProps = function detectFactoryAndVerify(
	stateOrDispatch,
 	ownProps
) {
  proxy.mapToProps = mapToProps
  proxy.dependsOnOwnProps = getDependsOnOwnProps(mapToProps)
  let props = proxy(stateOrDispatch, ownProps)

  if (typeof props === 'function') {
    proxy.mapToProps = props
    proxy.dependsOnOwnProps = getDependsOnOwnProps(props)
    props = proxy(stateOrDispatch, ownProps)
  }

  return props
}
```

现在的问题是，`connect` 到底做了什么？`provider` 提供了 `store` ，那么 `connect ` 是把`mapDispatchToProps` 和 `mapStateToProps` 都执行一次，也就是在`dispatch` 执行的 `action` 中产生的数据，放到 `props` 中，那么这个过程是怎样的