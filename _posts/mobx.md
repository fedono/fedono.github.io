# Mobx

- autorun 是怎么知道函数里面哪个属性是被监听的？

  > autorun 对它函数中使用的任何东西作出反应

​    mobx 中的 autoRun 是怎么知道里面有哪些 observer 的数据的

- 给每个数据被调用的时候，加上 autoRun 的方法，这样就能够在当前数据被调用的时候，执行了
  - 其实也只能使用这种方式了对不对，就看作者是怎么写的了 
- Mobx 的 derivation 到底是个什么东西？ 和 observer 是什么关系？
  - observer 是数据变化后需要触发的，那derivation 呢



还是找找文章看看，自己这样看，没那么容易理清思路。自己看主要是问题不够多，所以要根据哪些方面来解析 mobx 还不够清晰。



## Mobx-state-tree

mst 是要解决 mobx 什么问题，或者是提供一种什么范式，为什么不能直接使用 mobx 来实现状态的管理，mst 到底好在哪里

官方文档中的对比

> - mobx
>
>   MobX is **unopinionated** and allows you to manage your application state outside of any UI framework. This makes your code decoupled, portable, and above all, easily testable.
>
> - mobx-state-tree
>
>   Unlike MobX itself, MST is very **opinionated** about how data should be structured and updated. This makes it possible to solve many common problems out of the box.

对比一下`mobx` 和`mst` 的写法

```js
// mobx
export class ArticlesStore {
  @observable articlesRegistry = observable.map();

  @computed get articles() {
    return this.articlesRegistry.values();
  };

  @action setPage(page) {
    this.page = page;
  }
}

// mobx-state-tree
export const ShopStore = types
    .model("ShopStore", {
        bookStore: types.optional(BookStore, {
            books: {}
        })
    })
    .views((self) => ({
        get books() {
            return self.bookStore.books
        }
    }))
    .actions((self) => ({
        afterCreate() {
            self.bookStore.loadBooks()
        }
    }))
```

关于 **unopinionated** 和 **opinionated ** 的对比，应该是在`mst` 中就对所有的数据都定好了，整个结构非常清晰。

在介绍的文档中有写到

> Simply put, MST tries to combine the best features of both immutability (transactionality, traceability and composition) and mutability (discoverability, co-location and encapsulation) based approaches to state management; 

这些`immutability` 和 `mutability` 的特性是如何在 mst 中体现的？

> Finally, MST has built-in support for references, identifiers, dependency injection, change recording and circular type definitions (even across files).

这里的 `dependency injection/ change recording` 又是在代码中如何体现的。





## 参考

- [mobx源码分析（二） 订阅响应式数据](https://zhuanlan.zhihu.com/p/42225597)

- [mobx源码解析](https://github.com/dxmz/Mobx-Chinese-Interpretation)

- [深入理解 Mobx Computed](https://luncher.github.io/2020/05/18/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3-Mobx-Computed/)

  > 这个博客下面的 `mobx ` tag 下的几个 mobx 的文章还是挺好的，虽然不是讲源码，但是文章写的思路清晰，其他文章也可以看看

- [Mobx真的好用吗？它有什么优缺点？主要适用于什么场景？](https://www.zhihu.com/question/328612405) 

- [mobx-state-tree philosophy](https://mobx-state-tree.js.org/intro/philosophy)

- [mobx 概念与原则](https://cn.mobx.js.org/intro/concepts.html) 

