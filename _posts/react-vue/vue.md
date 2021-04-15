- 在那个生命周期中做了什么事

  - 在 created 中数据是否是响应式的
- 在 vue-runtime-compiler 与 vue-runtime-only 的区别
  
- computed 和 watch 的区别是什么

  > 为什么会有这个问题，因为 computed 和 watch 有相似的地方，也有不同的地方，所以才会有这种问题

  - 如果只是监听一个属性，然后计算后返回另一个属性，那么这两个都可以使用，那么就需要知道差异是什么
  - computed 不能执行异步操作
  - watch 更适合在监听一个属性后，可以继续做请求等一些异步的操作卡，或者是做一些业务操作，这个操作是可以不用返回一个数据的，但是 computed 是必须要返回一个计算后的属性的
  - 这时候我们就可以去看看源码怎么说的了 [计算属性 VS 侦听属性](https://ustbhuangyi.github.io/vue-analysis/v2/reactive/computed-watcher.html#computed) 

- keep-alive 是如何做的

- 说说 vue 父子组件加载顺序

  这我知道答案

  1. 父 beforeCreate
  2. 父 created
  3. 父 beforeMount
  4. 子 beforeCreate
  5. 子 created
  6. 子 beforeMount
  7. 子 mounted
  8. 父 mounted

  子组件若有 props 的话更新顺序是四步，若无的话两步不触发父亲的钩子。

  1. 父 beforeUpdate
  2. 子 beforeUpdate
  3. 子 updated
  4. 父 updated

  父组件更新顺序是

  1. 父 beforeUpdate
  2. 子 deactivated
  3. 父 updated

  销毁过程是

  1. 父 beforeDestroy
  2. 子 beforeDestroy
  3. 子 destroyed
  4. 父 destroyed





vue 的一些可看的文章

> 有一些会收藏到自己的收藏夹中，就不在下面列出了 

## 源码解析

- [http://hcysun.me/vue-design/zh/](http://hcysun.me/vue-design/zh/)
  - https://github.com/HcySunYang/vue-design
- https://ustbhuangyi.github.io/vue-analysis/v2/prepare/
  - vue 2 的解析视频，视频很好，但不一定每个概念都讲清楚了，比如 computed vs watch这两个概念，虽然讲了源码，但是在文字上还是没有讲明白
- [ Vue.js 3.0 核心源码解析](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=326#/content) 
  - 拉勾的
- https://vue3.w2deep.com/source-code/runtime/renderer.html

## 用法解析

- 直接通过组件能够解析出文档 [vuese](https://github.com/vuese/vuese)

