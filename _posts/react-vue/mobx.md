# Mobx

- autorun 是怎么知道函数里面哪个属性是被监听的？

  > autorun 对它函数中使用的任何东西作出反应

​    mobx 中的 autoRun 是怎么知道里面有哪些 observer 的数据的

- 给每个数据被调用的时候，加上 autoRun 的方法，这样就能够在当前数据被调用的时候，执行了
  - 其实也只能使用这种方式了对不对，就看作者是怎么写的了 
- Mobx 的 derivation 到底是个什么东西？ 和 observer 是什么关系？
  - observer 是数据变化后需要触发的，那derivation 呢



理解一下 autorun 函数| [文档](https://cn.mobx.js.org/refguide/autorun.html)

```js
var numbers = observable([1,2,3]);
var sum = computed(() => numbers.reduce((a, b) => a + b, 0));

var disposer = autorun(() => console.log(sum.get()));
// 输出 '6'
numbers.push(4);
// 输出 '10'
```

从上面的代码中，知道 autorun 在函数运行的时候就会被调用，也就是在 `numbers.push ` 的时候又会调用一次。

我的理解如下

1. 首先函数运行到 `autorun(() => console.log(sum.get()))` 时，会自动执行 autorun 里面的函数，这时候肯定会执行 `sum.get()` 方法的，所以这时候输出 6 是没什么疑问的

   疑问就是，那么在下一次 `numbers.push(4)` 的时候，为什么 autorun 里面的函数还能执行

   - 第一次执行 autorun 的时候，这个时候我们标记一下，现在是进入了 autorun 函数
   - 这时候执行 sum.get 的时候，因为已经对 numbers 做了 observable了，也就是在 push 这种操作的时候，我们将当前 autorun 函数标记一下，也就是知道我们在对 numbers 操作的时候，是要触发 autorun 这个函数的
   - 所以重点就是，不是我们在还没运行 autorun 的时候，我们就知道 numbers 与 autorun 函数进行绑定，而是我们运行一次 autorun 函数之后，我们触发了 numbers 函数，并且知道当前是在 autorun 函数中触发的，所以下一次我们继续修改 numbers 的时候，我们就知道要调用一次 autorun 的函数

2. 具体的 mobx 的 autorun 实现，参考[autorun 实现](https://github.com/fedono/state-manage/tree/master/mobx/autorun)

   > 和我上面的想法是一致的



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

- dependency injection 为在建立 store 的时候，第三个参数为建立全局的参数可以在全局引用
  - 参考 [官方文档 - Dependency Injection](https://mobx-state-tree.js.org/concepts/dependency-injection)



## 参考

- [mobx源码分析（二） 订阅响应式数据](https://zhuanlan.zhihu.com/p/42225597)

- [mobx源码解析](https://github.com/dxmz/Mobx-Chinese-Interpretation)

- [深入理解 Mobx Computed](https://luncher.github.io/2020/05/18/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3-Mobx-Computed/)

  > 这个博客下面的 `mobx ` tag 下的几个 mobx 的文章还是挺好的，虽然不是讲源码，但是文章写的思路清晰，其他文章也可以看看
  >
  > 这特么才是真正适合在大厂混的，会写文档太重要了！！！

- [Mobx真的好用吗？它有什么优缺点？主要适用于什么场景？](https://www.zhihu.com/question/328612405) 

- [mobx-state-tree philosophy](https://mobx-state-tree.js.org/intro/philosophy)

- [mobx 概念与原则](https://cn.mobx.js.org/intro/concepts.html) 

