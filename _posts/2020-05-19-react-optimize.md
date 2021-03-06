---
layout: post 
title: "React 性能优化" 
author: "fedono"
---



React 性能优化主要考虑的点

- React 的优化方向：减少 render 的次数；减少重复计算。
- 由于react中性能主要耗费在于update阶段的diff算法，因此性能优化也主要针对diff算法。
- 如何去找到 React 中导致性能问题的方法，见 useCallback 部分。
- 合理的拆分组件其实也是可以做性能优化的，你这么想，如果你整个页面只有一个大的组件，那么当 props 或者 state 变更之后，需要 reconction 的是整个组件，其实你只是变了一个文字，如果你进行了合理的组件拆分，你就可以控制更小粒度的更新。



## 由于react中性能主要耗费在于update阶段的diff算法，因此性能优化也主要针对diff算法。

## 1.减少diff算法触发次数

减少diff算法触发次数实际上就是减少update流程的次数。
正常进入update流程有三种方式：

### 1.setState

setState机制在正常运行时，由于批更新策略，已经降低了update过程的触发次数。
因此，setState优化主要在于非批更新阶段中(timeout/Promise回调)，减少setState的触发次数。
常见的业务场景即处理接口回调时，无论数据处理多么复杂，保证最后只调用一次setState。

### 2.父组件render

父组件的render必然会触发子组件进入update阶段（无论props是否更新）。此时最常用的优化方案即为shouldComponentUpdate方法。
最常见的方式为进行this.props和this.state的浅比较来判断组件是否需要更新。或者直接使用PureComponent，原理一致。



**避免使用箭头函数形式的事件处理器**, 例如:

```js
<ComplexComponent onClick={evt => onClick(evt.id)} otherProps={values} />
```

假设 ComplexComponent 是一个复杂的 PureComponent, 这里使用箭头函数，其实每次渲染时都会创建一个新的事件处理器，这会导致 ComplexComponent 始终会被重新渲染.

更好的方式是使用实例方法:

```js
class MyComponent extends Component {
  render() {
    <ComplexComponent onClick={this.handleClick} otherProps={values} />;
  }
  handleClick = () => {
    /*...*/
  };
}
```

在 21 个优化技巧那里也说了，即使是第二种使用箭头函数，也是有损耗的，如下

箭头函数好处多多，但也有缺点。

当我们添加箭头函数时，该函数被添加为对象实例，而不是类的原型属性。这意味着如果我们多次复用组件，那么在组件外创建的每个对象中都会有这些函数的多个实例。

每个组件都会有这些函数的一份实例，影响了可复用性。此外因为它是对象属性而不是原型属性，所以这些函数在继承链中不可用。

因此箭头函数确实有其缺点。实现这些函数的最佳方法是在构造函数中绑定函数

## 参考

- [React 函数式组件性能优化指南](https://mp.weixin.qq.com/s/mpL1MxLjBqSO49TRijeyeg)

- [ 21 个 React 性能优化技巧](https://www.infoq.cn/article/KVE8xtRs-uPphptq5LUz)
- [浅谈React性能优化的方向](https://juejin.im/post/5d045350f265da1b695d5bf2#heading-1) 