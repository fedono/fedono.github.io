# Mobx

- autorun 是怎么知道函数里面哪个属性是被监听的？

  > autorun 对它函数中使用的任何东西作出反应

​    mobx 中的 autoRun 是怎么知道里面有哪些observer的数据的

- 给每个数据被调用的时候，加上 autoRun 的方法，这样就能够在当前数据被调用的时候，执行了
  - 其实也只能使用这种方式了对不对，就看作者是怎么写的了 
- Mobx 的 derivation 到底是个什么东西？ 和 observer 是什么关系？
  - observer 是数据变化后需要触发的，那derivation 呢



还是找找文章看看，自己这样看，没那么容易理清思路。

## 参考

- [mobx源码分析（二） 订阅响应式数据](https://zhuanlan.zhihu.com/p/42225597)
- [mobx源码解析](https://github.com/dxmz/Mobx-Chinese-Interpretation)

- [深入理解 Mobx Computed](https://luncher.github.io/2020/05/18/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3-Mobx-Computed/)

  > 这个博客下面的 `mobx ` tag 下的几个 mobx 的文章还是挺好的，虽然不是讲源码，但是文章写的思路清晰，其他文章也可以看看