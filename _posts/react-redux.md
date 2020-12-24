---
layout: post 
title: "react-redux/mobx-react 做了什么" 
author: "fedono"
---

 `react-redux/mobx-react` 都是用来把数据管理和`react` 联合起来，那是怎么联合的，究竟做了什么。

 `react-redux/mobx-react`  都提供了一个`provider`，`provider ` 用来在顶层使用`React.context`把`store`数据传递给内部的组件，这样在内部就可以全部使用

```react
// Provider 函数的返回
return <Context.Provider value={contextValue}>{children}</Context.Provider>

// 使用时
<Provider store={store}>
  <App />
</Provider>
```

## React-Redux

react-redux 中有个 `connect` 用来连接组件和store。

> React Redux provides a `connect` function for you to connect your component to the store.

正常使用`connect` 的示例如下

```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
  setFilter: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

// action 中的 setVisibilityFilter 函数
export const setVisibilityFilter = filter => ({ type: types.SET_VISIBILITY_FILTER, filter})
```

整个分析的过程看 redux 那篇文档

## mobx-react 

mobx-react  中使用 `observer` 来监听组件，那么则组件的 `props/state` 变化的时候，会触发组件的重新渲染

```js
// observer 中对组件进行重新封装
return makeClassComponentObserver(component as React.ComponentClass<any, any>) as T

// makeClassComponentObserver 中将 props/state 监听
makeObservableProp(target, "props")
makeObservableProp(target, "state")

const baseRender = target.render
target.render = function() {
  return makeComponentReactive.call(this, baseRender)
}

// makeComponentReactive 中使用 mobx 来对 render 进行重新渲染
const baseRender = render.bind(this)
this.render = reactiveRender

function reactiveRender() {
  isRenderingPending = false
  let exception = undefined
  let rendering = undefined
  // track 用来更新视图 （话说 mobx 的 reaction.track 的）
  reaction.track(() => {
    try {
      rendering = _allowStateChanges(false, baseRender)
    } catch (e) {
      exception = e
    }
  })
  if (exception) {
    throw exception
  }
  return rendering
}

return reactiveRender.call(this)
```

