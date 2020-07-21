---
layout: post 
title: "Event Loop" 
author: "fedono"
---

## 描述

首先要你介绍什么是JS 的事件循环

1. 所有的同步任务都在主线程上执行，形成一个执行栈
2. 主线程之外，还存在一个任务队列，只要有异步任务，就在任务队列中放置一个事件
3. 一旦执行栈中的所有同步任务执行完毕后，系统就会读取任务队列，将队列中的事件放到执行栈中一次执行
4. 主线程从任务队列中读取事件，这个过程是循环不断的



**具体的理解**

先看下这段代码

```javascript
console.log('start')

setTimeout(function() {
  console.log('setTimeout')
}, 0)

Promise.resolve().then(function() {
  console.log('promise1')
}).then(function() {
  console.log('promise2')
})

console.log('end')
```

执行的过程为

1. 执行整个代码，整个代码也是 `macroTask` 

2. 执行`console.log('start')` 输出 `start`

3. 执行到 `setTimeout` ，因为`setTimeout` 是 `macroTask` ，加入到`macroTask`队列中

4. 执行到 `Promise` ，加入到`microTask` 的队列中

5. 执行`console.log('end')` 输出 `end` 

6. 因为 `macroTask` 执行完会执行`microTask` 中队列的所有任务，所以执行 

   ```js
   Promise.resolve().then(function() {
     console.log('promise1')
   }).then(function() {
     console.log('promise2')
   })
   ```

   这时候由`Promise` 产生的 `then` 也添加到`microTask` 中，也继续执行完成

7. 然后再执行下一个`macroTask` ，也就是`setTimeout` ，所以输出`setTimeout` 



## 浏览器和NODE 执行Event Loop 的差异 

关于 eventLoop ，在 浏览器和在 Node 中执行是不一样的

```js
setTimeout(()=>{
    console.log('timer1')

    Promise.resolve().then(function() {
        console.log('promise1')
    })
}, 0)

setTimeout(()=>{
    console.log('timer2')

    Promise.resolve().then(function() {
        console.log('promise2')
    })
}, 0)
```

在浏览器中是

```
timer1
promise1
timer2
promise2
```

但是在 `Node` 中是

```
timer1
timer2
promise1
promise2
```

**解释**

对于浏览器来说，解释为执行整个代码时将两个`setTimeout` 都加入到`macroTask` 中，然后执行`macroTask` 时，产生的 `microTask` 也会一并执行 ，也就是每次执行一次 `macroTask` 都会去检查`microTask` 中的任务。

>
>    1. 检查macrotask队列是否为空，非空则到2，为空则到3
>    2. 执行macrotask中的一个任务
>    3. 继续检查microtask队列是否为空，若有则到4，否则到5
>    4. 取出microtask中的任务执行，执行完成返回到步骤3
>    5. 执行视图更新
>

对于Node来说

> - 当 event loop 到达某个阶段时，将执行该阶段的任务队列，直到队列清空或执行的回调达到系统上限后，才会转入下一个阶段
> - 当所有阶段被顺序执行一次后，称 event loop 完成了一个 tick



**注意要点** 

- 浏览器和Node 的区分



### setTimeou第二个参数仅仅表示最少延迟时间，而非确切的等待时间

函数 `setTimeout` 接受两个参数：待加入队列的消息和一个时间值（可选，默认为 0）。这个时间值代表了消息被实际加入到队列的最小延迟时间。如果队列中没有其它消息并且栈为空，在这段延迟时间过去之后，消息会被马上处理。但是，如果有其它消息，`setTimeout` 消息必须等待其它消息处理完。因此第二个参数仅仅表示最少延迟时间，而非确切的等待时间。

### setImmediate/setTimeout/process.nextTick

`process.nextTick` 会在执行完当前的任务后立即执行，会比 `setImmediate/setTimeout` 都要快

`setImmediate` 和 `setTimeout(fn, 0)` 差不多

参考[官方文档-了解 setImmediate()](http://nodejs.cn/learn/understanding-setimmediate)



## 参考

- [深入理解js事件循环机制（浏览器篇）](http://lynnelv.github.io/js-event-loop-browser)
- [深入理解js事件循环机制（Node.js篇）](http://lynnelv.github.io/js-event-loop-nodejs) 

- [阮一峰-JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

- [【朴灵评注】JavaScript 运行机制详解：再谈Event Loop](https://blog.csdn.net/lin_credible/article/details/40143961)

- [mdn-EventLoop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop) 