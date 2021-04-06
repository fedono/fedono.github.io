---
layout: post 
title: "详解 JS 的 reduce" 
author: "fedono"
---



终于明白了，或者说是懂了 JS 中 `array` 中的 `reduce` 了，以前好像是一直没有懂，或者说是似懂非懂，`reduce` 要的就是返回函数中的第一个参数

```js
var arr = [1, 2, 3, 4];

var sum = function (a, b) {
  console.log(a, b);
  return a + b;
};

arr.reduce(sum) // => 10
// 1 2
// 3 3
// 6 4
```

上面的 `sum` 函数，也就是`reduce` 返回的就是 `a` ，所以在 `sum` 函数中怎么操作都行，反正最后就是要返回`a` ，想起那时候在头条面试的时候，问起来的，怎么使用 `reduce` 来实现一个 `map`  ，看下下面这个操作就非常明显了

```js
var arr = [1, 2, 3, 4];

var handler = function (newArr, x) {
  newArr.push(x + 1);
  return newArr;
};

arr.reduce(handler, [])
// [2, 3, 4, 5]
```

自己实现一个通过`reduce` 来实现 `map` 的方法

```js
Array.prototype._map = function(func) {
    return Array.prototype.reduce((a, b, index, array) => {
        a.push(func(b, index, array));
        return a;
    }, [])
}
```

测试一次

```js
let res1 = [1, 2, 3].map((v, i, arr, context) => {
    return v + 1;
})
console.log(res1); // [ 2, 3, 4 ]

let res2 = [1, 2, 3]._map((v, i, arr, context) => {
    return v + 1;
})

console.log(res2); // []
```

结果出错了，问题出在哪里？

打印一下

```js
Array.prototype._map = function(func) {
    console.log(1)
    return Array.prototype.reduce((a, b, index, array) => {
        a.push(func(b, index, array));
        console.log(a, b);
        return a;
    }, [])
}
```

发现只打印了 `1` ，而在 `reduce` 中，就没有执行 ，也就是根本就没有执行 `console.log(a, b)` 

翻了下以前做的笔记，把 `Array.prototype.reduce` 修改成了 `this.reduce` 就可以了，最终结果如下

```js
Array.prototype._map = function(func) {
    return this.reduce((a, b, index, array) => {
        a.push(func(b, index, array));
        return a;
    }, []);
}
```

这时候再进行测试，结果正常了

```js
let res2 = [1, 2, 3]._map((v, i, arr, context) => {
    return v + 1;
})

console.log(res2); // [ 2, 3, 4 ]
```

之前也有疑惑，使用 `Array.prototype.reduce` 会不会不合适，这时候肯定不会被调用，因为这是个加在 `Array` 原型上的方法，但是使用 `this` ，这时候不是在 `function` 中吗，`this` 到底是个啥也不清楚，打印一下

```js
Array.prototype._map = function(func) {
    console.log(this); // [ 1, 2, 3 ]
    return this.reduce((a, b, index, array) => {
        a.push(func(b, index, array));
        return a;
    }, []);
}
```

结果是 `[ 1, 2, 3 ]` 就是当前数组（这里个疑惑，为什么打印出来的 `this` 是当前数组，答案见下方

继续对 `_map` 函数做个优化

```js
Array.prototype._map = function(fn) {
    return this.reduce((result, v, i, arr) => result.push(fn(v, i, arr)), []);
}
// 这样写的问题在于 result.push 返回的是数组的长度，也就是下一次的reduce 的运行时，会出现 result.push is not a function，因为 Number 没有push 这个方法

// 优化后的方法如下
Array.prototype._map = function(fn) {
    return this.reduce((result, v, i, arr) => [...result, fn(v, i, arr)], []);
}

// 因为之前在_map 中使用 this 打印出了 [1, 2, 3]也就是当前数组，然后在使用的时候，就写成了下面这样
Array.prototype._map = function(fn) {
    return this.reduce((result, v, i, this) => [...result, fn(v, i, this)], []);
}
// 也就是把 reduce 中的参数 arr 换成了 this，结果报错了 `SyntaxError: Unexpected token this` ，才反应过来，reduce 中的参数 `arr` 是个形参，也就是 `reduce` 会把数组放到这个参数中取，如果自己放 this 就会导致出错了
```



### 注意

`reduce`为数组中的每一个元素依次执行`callback`函数，不包括数组中被删除或从未被赋值的元素，接受四个参数：

- `accumulator 累计器`
- `currentValue 当前值`
- `currentIndex 当前索引`
- `array 数组`

回调函数第一次执行时，`accumulator` 和`currentValue`的取值有两种情况：如果调用`reduce()`时提供了`initialValue`，`accumulator`取值为`initialValue`，`currentValue`取数组中的第一个值；如果没有提供 `initialValue`，那么`accumulator`取数组中的第一个值，`currentValue`取数组中的第二个值。

**注意：**如果没有提供`initialValue`，reduce 会从索引1的地方开始执行 callback 方法，跳过第一个索引。如果提供`initialValue`，从索引0开始。

如果数组为空且没有提供`initialValue`，会抛出[`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 。如果数组仅有一个元素（无论位置如何）并且没有提供`initialValue`， 或者有提供`initialValue`但是数组为空，那么此唯一值将被返回并且`callback`不会被执行。

```js
let res = [1, 2, 3].reduce((res, v) => {
    return res + (v * 2)
}); // 11, 没有设置初始值的时候，第一次计算时 1 + (2 * 2) ，也就是 1 没有 * 2 ，所以结果会是 11

let res = [1, 2, 3].reduce((res, v) => {
    return res + (v * 2)
}, 0); // 12
```

### 疑惑

为什么 `this` 是当前数组，我还是没有搞懂

> 因为 this 就是当前对象

```js
// 案例1 打印对象
let obj = {
    a: 1,
    sayName: function() {
        console.log(this);  // { a: 1, sayName: [Function: sayName] }
        return this.a;
    }
}

obj.sayName(); 

// 案例2 打印函数中的 this
function getName() {
    console.log(this);
    this.name = 'jack';
}

getName();

// 输出结果，因为是在 webstorm 中使用 node 作为环境来执行的，
/*
Object [global] {
    DTRACE_NET_SERVER_CONNECTION: [Function],
    ...
    features:
    { debug: false,
      uv: true,
      ipv6: true,
      tls_alpn: true,
      tls_sni: true,
      tls_ocsp: true,
      tls: true },
   ppid: 47817,
       ...
  setImmediate:
   { [Function: setImmediate] [Symbol(util.promisify.custom)]: [Function] },
  setInterval: [Function: setInterval],
  setTimeout:
   { [Function: setTimeout] [Symbol(util.promisify.custom)]: [Function] } }  
*/

// 放在浏览器的控制台中打印，结果如下
//Window {parent: Window, opener: null, top: Window, length: 0, frames: Window, …}
```





## 参考

- [Reduce 和 Transduce 的含义](http://www.ruanyifeng.com/blog/2017/03/reduce_transduce.html)
- [Reduce相关操作](https://gist.github.com/catchonme/3d3cbd6387bbf633c41a3b6d26a27ec9)
  - reduce 的相关应用
  - reduce 实现数组中的各种原生函数
  - 使用 function 实现 reduce  
- [mdn reduce](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
- [mdn flat](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) 
  - ECMAScript 2019 已经实现了数组无限扁平化了
  - 里面有其他各种方式来实现扁平的操作