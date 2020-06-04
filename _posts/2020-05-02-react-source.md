- src 文件是定义结构，具体的都是在 react-dom 中进行具体实现，react-native 则是针对其他平台的实现

- babel 网站中的 try it ，有使用 jsx 转换到 js 

  - 比如 

    ```html
    <div id="name">
      <span>jack</span>
    </div>
    ```

    就会转换成

    ```js
    "use strict";
    
    /*#__PURE__*/
    React.createElement("div", {
      id: "name"
    }, /*#__PURE__*/React.createElement("span", null, "jack"));
    ```

    如果是组件呢，

    ```html
    function Comp () {
      return <a></a>
    }
    
    <Comp>
      <div id="name">
      	<span>jack</span>
      </div>
    </Comp>
    ```

    就会转换成

    ```js
    /*#__PURE__*/
    React.createElement(Comp, null, /*#__PURE__*/React.createElement("div", {
      id: "name"
    }, /*#__PURE__*/React.createElement("span", null, "jack")));
    ```

    如果我们的 `Comp` 写成了小写，就会转换成

    ```js
    /*#__PURE__*/
    React.createElement("comp", null, /*#__PURE__*/React.createElement("div", {
      id: "name"
    }, /*#__PURE__*/React.createElement("span", null, "jack")));
    ```

    也就是这时候的 `comp` 是小写的形式，那么到时候 `react` 在解析的时候，会找不到这个元素而报错，所以需要我们在定义组件的时候，一定需要大写

进入 `packages/react` 中的`index` 可以看到引入了`src/react` ，进入 `react` 文件中，可以看到所有通过 `React` 暴露的对象了，如`Children` 暴露出来的方法和属性

## Children

`React` 中可以在 `props` 中使用 `children` ，这时候可以通过`React.Children.map(props.children, c => [c, c] )` 来使用

```js
Children: {
    map,
    forEach,
    count,
    toArray,
    only,
  },
```

，如果 `callback` 是 `c => [c, [c, c]]` 那么就会返回三个一维数组，会将所有的嵌套数据展开来，而不是 `[c, [c, c]]` 这种格式，可以使用如下的案例测试下

```js
import React from 'react';

function ChildrenDemo(props) {
  console.log(props.children);
  
  console.log(React.Children.map(props.children, c => [c, [c, c]]));
  return props.children;
}

export default () => (
	<ChildrenDemo>
  	<span>1</span>
  	<span>2</span>
  </ChildrenDemo>
)
```

### onlyChild

检测如果传进来的不是单个的元素的话，就抛错

```js
function onlyChild(children) {
  invariant(
    isValidElement(children),
    'React.Children.only expected to receive a single React element child.',
  );
  return children;
}
```

另外四个 `map,forEach,count,toArray` 可以放在一起分析

直接分析 `map` 

```js
function mapChildren(children, func, context) {
  if (children == null) {
    return children;
  }
  const result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, func, context);
  return result;
}
```

`mapChildren` 调用 `mapIntoWithKeyPrefixInternal` , `mapIntoWithKeyPrefixInternal` 则调用 `getPooledTraverseContext` 和 `traverseAllChildren`

```js
// getPooledTraverseContext 返回这些参数，并且多加上一个 count 参数
const traverseContext = getPooledTraverseContext(
    array,
    escapedPrefix,
    func,
    context,
  );
traverseAllChildren(children, mapSingleChildIntoContext, traverseContext);

// traverseAllChildren 检测含有 children 后，则调用 traverseAllChildrenImpl 函数
// traverseAllChildrenImpl 检测传进来的 callback ，也就是 traverseAllChildren 中的第二个参数

// callback 在 map 的时候是 mapSingleChildIntoContext， forEach 的时候是 forEachSingleChild
// forEachSingleChild 只是执行完 callback() 就行，mapSingleChildIntoContext 执行完 callback 后，会将结果再执行 cloneAndReplaceKey 后返回到 mapChildren 中的 result 

// traverseAllChildrenImpl 会每次执行的 count 会加1，这样就能知道 Children.count 是多少了，毕竟 map/forEach 都不需要 traverseAllChildrenImpl 的返回
callback(
  traverseContext,
  children,
  // If it's the only child, treat the name as if it was wrapped in an array
  // so that it's consistent if the number of children grows.
  nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar,
)
```

## ReactElement

我们看引入的`ReactElement` 中引入的几个方法

```js
import {
  createElement,
  createFactory,
  cloneElement,
  isValidElement,
} from './ReactElement';
```

首先看`function createElement(type, config, children)` 这里有三个参数，这三个参数就是我们上面在 `babeljs.io` 中尝试的从`jsx` 转到 `js` 的格式，`type` 表示当前元素是什么，`config` 就是配置的属性等，`children` 就是当前元素下的还有哪些元素

- `createElement` 中首先会对所有传入的 `config` 做处理，什么是有效的配置，什么是内建的配置，如`ref/ key` 等，这些就会先被排除掉
- 因为`children` 会有多个，所以设置`arguments.length - 2` 以后的都是 `children` ，
- 组件内部也会设置 `defaultProps` ，需要把这些属性也一起合并进来
- 最会把所有的参数传给`ReactElement` 函数，这个函数就是加了一个 `$$typeof: REACT_ELEMENT_TYPE` 参数，这个参数是在后面`react-dom` 渲染的时候会用到

## Component/PureComponent

Component 引入三个参数，第三个参数`updater` 是更新算法，他不会在`react` 这个文件夹中去讲，而且`updater` 也是通过参数引入的，然后再调用这个参数的一系列方法，比如组件的声明周期，都不在 `Component` 文件中

```js
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}
```

`Component` 的`setState` 和`forceUpdate`  也都是调用这个方法

```js
Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};
```

`PureComponent` 则是完全继承了`Component`，然后给自己加了个`isPureComponent` 的标签

```js
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

/**
 * Convenience component with default shallow equality check for sCU.
 */
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;
// Avoid an extra prototype jump for these methods.
Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true;
```

## createRef

`React` 从建立初到现在有三种可以建立 `ref`的方式

```html
//string
<p ref="stringRef">span1</p> 
// method
<p ref={ele => (this.methodRef = ele)}>span3</p> 
// createRef
<p ref={this.objRef}>span3</p> 
```

只有第三种是需要在建立初的时候，通过在 `React` 上的 `API`  `createRef` 建立的

```js
this.objRef = React.createRef();
```

在使用的时候，这三种 `ref` 的使用方式为

```js
this.refs.stringRef.textContent = 'string ref got'
this.methodRef.textContent = 'method ref got'
this.objRef.current.textContent = 'obj ref got'
```

也就是第三种 `ref` 中会有个 `current` 的属性，我们看下`react-16.6.0/packages/react/src/ReactCreateRef.js` 

```js
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  
  return refObject;
}
```

也就是 `createRef` 会直接建立 一个含有 `current` 的对象，之后如何在这三种 `ref` 中把节点挂载上去呢，这部分是在 `react` 的更新机制中

## forwardRef

关于 `forwardRef` 的使用，可以看 [这里](./2020-04-26-react-context-forwardRef.md) ，也就是 `forwardRef` 是一个可以把`ref` 下传的功能，他的源码也很简单，这里也是加了个 `$$typeof` 

```js
export default function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
 
  return {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };
}
```

## Context

可以有两种形式来添加

- childContextType
  - 即将在 17 的版本中废弃掉
- createContext
  - 关于` createContext` 的使用，可以看 [这里](./2020-04-26-react-context-forwardRef.md)

>  为什么要改呢，是说因为每次都要传的数据量太大了，具体过程会在后续`react`更新中中提到 

我们可以看到 `context` 的 `provider` 和 `consumer` ，都是引用自己的

```js
context.Provider = {
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context,
  };

context.Consumer = context;
```

# ConcurrentMode

ConcurrentMode 是一个执行调度的处理，从最高优先级到处理最低的优先级

react-dom 有个 `flushSync` 的方法，会让当前任务以最高优先级的方式来执行

在入口文件中的 `React` 中，`unstable_ConcurrentMode` 只是一个标志位

> 在  16.13.1 的版本中，已经去掉了这个了 ，下载 16.13.1 的版本，发现在 `src/react.js` 中 `ConcurrentMode` 被注释掉了

### 参考文档

- [React Fiber初探](https://zhuanlan.zhihu.com/p/31634312)
- [深入剖析 React Concurrent](https://zhuanlan.zhihu.com/p/60307571)



## Suspense

- throw promise 能获取到什么

suspense 会在内部所有的组件都加载完成后，才会隐藏掉 fallback 的显示

在 `src/React.js` 中 `suspense` 只是一个标志 `Suspense: REACT_SUSPENSE_TYPE,` 

## Hooks





## 参考

- [React.createElement 和 ReactDOM.render 的简易实现](https://juejin.im/post/5e9b0a51e51d4546de47e6cb) 