---
layout: post 
title: "redux/thunk/saga 概念理清" 
author: "fedono"
---

## 概念

### Redux

使用方法

```js
// action 是一个函数，返回一个对象
const set = (value) => ({
  type: 'SET_COUNTER',
  payload: value
})

// reduce 是通过将 action 的返回的对象进行处理后返回
// 也就是在 container 组件中，通过 dispatch，比如 dispatch(set('addNumber')); 触发 reduce 的操作
const counter = (state = 0, action) => {
  switch (action.type) {
    case SET_COUNTER:
      return action.payload
    default:
      return state
  }
}

// container 组件
// container 组件会通过connect函数传给组件一个 dispatch，然后通过 dispatch 调用 action 中引入的函数
import {set} from '../actions';

class App extends React.Component {
  
  // 组件内的操作
  const { dispatch } = this.props
	dispatch(set('addNumber'))
}

const mapStateToProps = state => {
  const { selectedSubreddit, postsBySubreddit } = state

  return {
    selectedSubreddit,
    posts,
    isFetching,
    lastUpdated
  }
}

export default connect(mapStateToProps)(App)

```

以上三步是一个最基础，也是`redux` 的核心三步。



### Thunk

redux 中的 `dispatch` 函数中的参数是一个对象，也就是`dispatch(action)`，`thunk` 支持的方式为`dispatch(function)` 

> 一般情况下，actions 都是符合 [FSA](https://github.com/redux-utilities/flux-standard-action) 标准的（即：a plain javascript object），像下面这样：
>
> ```js
> {
>   type: 'ADD_TODO',
>   payload: {
>     text: 'Do something.'  
>   }
> };
> ```
>
> 它代表的含义是：每次执行 `dispatch(action)` 会通知 reducer 将 action.payload（数据） 以 action.type 的方式（操作）**同步更新**到 本地 store 。
>
> 而一个 丰富多变的 Web 应用，payload 数据往往来自于远端服务器，为了能将 **异步获取数据** 这部分代码跟 UI 解耦，redux-thunk 选择以 middleware 的形式来增强 redux store 的 dispatch 方法（即：支持了 `dispatch(function)`），从而在拥有了 异步获取数据能力 的同时，又可以进一步将 数据获取相关的业务逻辑 从 View 层分离出去。
>
> 来看看以下代码
>
> ```js
> // action.js
> // ---------
> // actionCreator(e.g. fetchData) 返回 function
> // function 中包含了业务数据请求代码逻辑
> // 以回调的方式，分别处理请求成功和请求失败的情况
> export function fetchData(someValue) {
>   return (dispatch, getState) => {
>     myAjaxLib.post("/someEndpoint", { data: someValue })
>       .then(response => dispatch({ type: "REQUEST_SUCCEEDED", payload: response })
>         .catch(error => dispatch({ type: "REQUEST_FAILED", error: error });
>   };
> }
> 
> // component.js
> // ------------
> // View 层 dispatch(fn) 触发异步请求
> // 这里省略部分代码
> this.props.dispatch(fetchData({ hello: 'saga' }));
> ```
>
> -- 来源 [**Redux-Saga 漫谈**](https://www.yuque.com/lovesueee/blog/redux-saga) 

这里改变了`dispatch ` 的调用方法，`fetchData` 中的仍然需要返回一个函数用来将 `dispatch` 函数当做参数传进去，所以增加了`redux-thunk` 后，所有的`action` 函数都必须写成这种形式。

### Saga 

在`redux` 使用`Saga` 而不是用`thunk` 后，`dispatch` 还是和原先一样，参数里还是之前的`action` ，也就是对象的形式。那么`saga` 要做些什么呢？

我们改造一下上面`thunk` 的代码，看看使用`saga` 后实现同样的功能是什么样

```js
// saga.js
// -------
// worker saga
// 它是一个 generator function
// fn 中同样包含了业务数据请求代码逻辑
// 但是代码的执行逻辑：看似同步 (synchronous-looking)
function* fetchData(action) {
  const { payload: { someValue } } = action;
  try {
    const result = yield call(myAjaxLib.post, "/someEndpoint", { data: someValue });
    yield put({ type: "REQUEST_SUCCEEDED", payload: response });
  } catch (error) {
    yield put({ type: "REQUEST_FAILED", error: error });
  }
}

// watcher saga
// 监听每一次 dispatch(action)               
// 如果 action.type === 'REQUEST'，那么执行 fetchData
export function* watchFetchData() {
  yield takeEvery('REQUEST', fetchData);
}


// component.js
// -------
// View 层 dispatch(action) 触发异步请求 
// 这里的 action 依然可以是一个 plain object
this.props.dispatch({
  type: 'REQUEST',
  payload: {
    someValue: { hello: 'saga' }
  }
});
```

至于`put/takeEvery` 这些概念，在[官方文档](https://redux-saga-in-chinese.js.org/) 中查看就好，概念都不难，很简短。

将从上面的代码，与之前的进行对比，可以归纳以下几点：

- 数据获取相关的业务逻辑 被转移到单独 saga.js 中，不再是掺杂在 action.js 或 component.js 中。

- dispatch 的参数依然是一个纯粹的 action (FSA)，而不是充满 “黑魔法” thunk function。

- 每一个 saga  都是 一个 generator function，代码采用 **同步书写** 的方式来处理 **异步逻辑**（No [Callback Hell](http://callbackhell.com/)），代码变得更易读（没错，这很 co~ ）。

- 同样是受益于 generator function 的 saga 实现，代码异常/请求失败 都可以直接通过 try/catch 语法直接捕获处理。



## 分析

### Redux

redux 中重要的函数就两，一个是`createStore` 用来初始化应用的`store`， 一个是`combineReducers` 用来将多个文件的`reduce` 合并成一个 ，虽然重要，但是其实你看下函数名，就知道这两个其实很简单。

`react` 与 `redux` 绑定，主要的作用一个如何将组件与store相关联，这就是`react-redux`  中  `connect` 的作用，一个是`dispatch` 如何触发`action` 这个对象，到使用`reduce` 来更新`state` 的，再通过`mapStateToProps ` 传给组件，触发组件的更新的。

```js
// dispatch 函数
function dispatch(action: A) {
  try {
    isDispatching = true
    // 因为使用combineReduce，这里就是一个总的 switch 语句，根据 action.type 找到对应的分子，然后更改当前的 state 就是 
    // 而 currentState 就是应用初始的 initState
    currentState = currentReducer(currentState, action)
  } finally {
    isDispatching = false
  }

  return action
}
```

所以现在唯一重要的就是 `connect` 函数了，如何在`dispatch` 后，让组件更新的。

`connect` 在接收 `mapStateToProps/ mapDispatchToProps` 后执行的是`connectAdvanced` ，`connectAdvanced` 中调用`wrapWithConnect` 来接收组件。

在 `wrapWithConnect` 中，通过在 ConnectFunction 中

```js
function ConnectFunction(props) {...}
    
return hoistStatics(ConnectFunction, WrappedComponent)
```

我们看下`ConnectFunction` ，也就是在 `connect` 中会给 原有的 component(也就是下面的 WrappedComponent) 再传入一些 props，同时在 `renderedChild` 中再使用 context 传入一些可以使用的 value 。

```react
// render the child component.
const renderedWrappedComponent = useMemo(
  () => (
    <WrappedComponent
      {...actualChildProps} // 这里的 actualChildProps 就包含了 mapStateToProps/ mapDispatchToProps 执行后返回的数据
      ref={reactReduxForwardedRef}
    />
  ),
  [reactReduxForwardedRef, WrappedComponent, actualChildProps]
)

const renderedChild = useMemo(() => {
  if (shouldHandleStateChanges) {
    // If this component is subscribed to store updates, we need to pass its own
    // subscription instance down to our descendants. That means rendering the same
    // Context instance, and putting a different value into the context.
    return (
      <ContextToUse.Provider value={overriddenContextValue}>
        {renderedWrappedComponent}
      </ContextToUse.Provider>
    )
	}

	return renderedWrappedComponent
}, [ContextToUse, renderedWrappedComponent, overriddenContextValue])

return renderedChild
```

上面代码中的 `actualChildProps` 是已经执行完 `mapStateToProps/ mapDispatchToProps` 返回的值。

执行的过程为

1. `connect` 中 将`selectorFactory/mapStateToProps/ mapDispatchToProps` 传入到 `connectAdvanced` 
2. `connectAdvanced` 返回  `wrapWithConnect`  接收组件
3. `wrapWithConnect` 中将 `store.dispatch/mapStateToProps/ mapDispatchToProps` 传给 `selectorFactory` 执行，返回 `actualChildProps` 

在 `renderedWrappedComponent` 中当每次  `actualChildProps` 变化了，则会重新引发渲染的过程。

#### applyMiddleware

在 Redux 中还有一个 applyMiddleware 不得不提，applyMiddleware 其实就是把 Redux 的 dispatch 交出去

```js
const middlewareAPI: MiddlewareAPI = {
  getState: store.getState,
  dispatch: (action, ...args) => dispatch(action, ...args)
}
const chain = middlewares.map(middleware => middleware(middlewareAPI))
dispatch = compose<typeof dispatch>(...chain)(store.dispatch)
```

相当于现在的 dispatch 等于如下的形式，至于有多少个 `a/b/c` 取决于用户传入的在使用`createStore` 中的第三个参数 `enhancer` 了

```js
dispatch = a => b => c => dispatch => dispatch(...); 
```

如 `thunk` 就是包了一层 ，本来中间是没有 `(next) =>` 这一层的 

> 因为 Redux 中的 compose 是 `funcs.reduce((a, b) => (...args: any) => a(b(...args)))` 

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => (next) => (action) => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}
```

###  

### Saga

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

  > 现在明白了，action 的结果就是需要返回一个对象，里面必须函数 type，这样才能被dispatch 

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





## 参考

- [**Redux-Saga 漫谈**](https://www.yuque.com/lovesueee/blog/redux-saga) 

  对于理解 redux/thunk/saga 的演进的理解有帮助

