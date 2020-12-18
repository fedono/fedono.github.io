---
layout: post 
title: "你不知道的JS-笔记-异步" 
author: "fedono"
---

## Promise 

1. 

```js
var p3 = new Promise( function(resolve,reject){
    resolve( "B" );
} );
var p1 = new Promise( function(resolve,reject){
    resolve( p3 );
} );
p2 = new Promise( function(resolve,reject){
    resolve( "A" );
} );
p1.then( function(v){
    console.log( v );
} );

p2.then( function(v){
    console.log( v );
} );
```

p1 不是用立即值而是用另一个 promise p3 决 议，后者本身决议为值 "B"。规定的行为是把 p3 展开到 p1，但是是异步地展开。所以，在 异步任务队列中，p1 的回调排在 p2 的回调之后(参见 1.5 节)。

要避免这样的细微区别带来的噩梦，你永远都不应该依赖于不同 Promise 间回调的顺序和 调度。实际上，好的编码实践方案根本不会让多个回调的顺序有丝毫影响，可能的话就要 避免。

> 对于任务队列最好的理解方式就是，它是挂在事件循环队列的每个 tick 之后 的一个队列。在事件循环的每个 tick 中，可能出现的异步动作不会导致一个完整的新事件 添加到事件循环队列中，而会在当前 tick 的任务队列末尾添加一个项目(一个任务)。

2. 

不过在这里还有更多的细节需要研究。如果在 Promise 的创建过程中或在查看其决议 结果过程中的任何时间点上出现了一个 JavaScript 异常错误，比如一个 TypeError 或 ReferenceError，那这个异常就会被捕捉，并且会使这个 Promise 被拒绝。

```js
var p = new Promise( function(resolve,reject) {
    foo.bar(); // foo未定义，所以会出错!
    resolve( 42 ); // 永远不会到达这里 :(
} );
p.then(
    function fulfilled(data){
        console.log()
// 永远不会到达这里 :(
    },
    function rejected(err){
        console.log(err)
// err将会是一个TypeError异常对象来自foo.bar()这一行
    })
```

3. 

如果 Promise 完成后在查看结果时(then(..) 注册的回调中)出现了 JavaScript 异 常错误会怎样呢?即使这些异常不会被丢弃，但你会发现，对它们的处理方式还是有点出 乎意料，需要进行一些深入研究才能理解:

```js
var p = new Promise( function(resolve,reject){
    resolve( 42 );
} );
p.then(
    function fulfilled(msg){
        foo.bar();
        console.log( msg ); // 永远不会到达这里 :(
    },
    function rejected(err) {
        // 永远也不会到达这里 :(
        console.log(err)
    })
```

等一下，这看起来像是 foo.bar() 产生的异常真的被吞掉了。别担心，实际上并不是这样。 但是这里有一个深藏的问题，就是我们没有侦听到它。p.then(..) 调用本身返回了另外一 个 promise，正是这个 promise 将会因 TypeError 异常而被拒绝。

为什么它不是简单地调用我们定义的错误处理函数呢?表面上的逻辑应该是这样啊。如果 这样的话就违背了 Promise 的一条基本原则，即 Promise 一旦决议就不可再变。p 已经完成 为值 42，所以之后查看 p 的决议时，并不能因为出错就把 p 再变为一个拒绝。

除了违背原则之外，这样的行为也会造成严重的损害。因为假如这个 promise p 有多个 then(..) 注册的回调的话，有些回调会被调用，而有些则不会，情况会非常不透明，难以 解释。

p.then 中的 `fulfilled` 中出现了错误之后，就可以在`p.then().then()` 中的后一个`then` 中捕获错误了。

```js
var p = new Promise( function(resolve,reject){
    resolve( 42 );
} );
p.then(
    function fulfilled(msg){
        foo.bar();
        console.log( msg ); // 永远不会到达这里 :(
    },
    function rejected(err) {
        // 永远也不会到达这里 :(
        console.log(err)
    })
    .then(
        function fulfilled(msg){
            console.log( msg );  
        },
        function rejected(err) {
            console.log(1); // 会输出1 
            console.log(err); // ReferenceError: foo is not defined
        })
```

所以可以看一下 MDN 中的 Promise 的解释，也就是前一个 promise 会影响到后一个 promise 的状态

4. 

如果向 Promise.resolve(..) 传递一个非 Promise、非 thenable 的立即值，就会得到一个用 这个值填充的 promise。下面这种情况下，promise p1 和 promise p2 的行为是完全一样的:

```js
var p1 = new Promise( function(resolve,reject) {
    resolve( 42 );
} );
var p2 = Promise.resolve( 42 );
```

而如果向 Promise.resolve(..) 传递一个真正的 Promise，就只会返回同一个 promise:

```js
var p1 = Promise.resolve( 42 );
var p2 = Promise.resolve( p1 );
p1 === p2; // true
```

更重要的是，如果向 Promise.resolve(..) 传递了一个非 Promise 的 thenable 值，前者就会

试图展开这个值，而且展开过程会持续到提取出一个具体的非类 Promise 的最终值。

```js
var p = {
    then: function(cb) {
        cb( 42 ); }
};

p.then(
    function fulfilled(val){ 
        console.log( val ); // 42
    },
    function rejected(err){
// 永远不会到达这里
    });
```

这个 p 是一个 thenable，但并不是一个真正的 Promise。幸运的是，和绝大多数值一样，它

是可追踪的。但是，如果得到的是如下这样的值又会怎样呢:

```js
var p = {
    then: function(cb,errcb) {
        cb( 42 );
        errcb( "evil laugh" ); }
};
p .then(
    function fulfilled(val){
        console.log( val ); // 42
    },
    function rejected(err){
				// 啊，不应该运行!
        console.log( err ); // evil laugh
    } );
```

我们还是都可以把这些版本的 p 传给 Promise.resolve(..)，然后就会得到期望中的规范化后的安全结果:

```js
Promise.resolve( p ) .then(
    function fulfilled(val){ 
      console.log( val ); // 42
    },
    function rejected(err){
// 永远不会到达这里
 });
```

Promise.resolve(..) 可以接受任何 thenable，将其解封为它的非 thenable 值。从 Promise. resolve(..) 得到的是一个真正的 Promise，是一个可以信任的值。如果你传入的已经是真 正的 Promise，那么你得到的就是它本身，所以通过 Promise.resolve(..) 过滤来获得可信 任性完全没有坏处。

假设我们要调用一个工具 foo(..)，且并不确定得到的返回值是否是一个可信任的行为良 好的 Promise，但我们可以知道它至少是一个 thenable。Promise.resolve(..) 提供了可信 任的 Promise 封装工具，可以链接使用:

```js
// 不要只是这么做: 
foo( 42 )
.then( function(v){
    console.log( v ); 
} );
// 而要这么做: 
Promise.resolve( foo( 42 ) ) 
    .then( function(v) {
    console.log( v ); 
} );
```

对于用 Promise.resolve(..) 为所有函数的返回值(不管是不是 thenable) 都封装一层。另一个好处是，这样做很容易把函数调用规范为定义良好的异 步任务。如果 foo(42) 有时会返回一个立即值，有时会返回 Promise，那么 Promise.resolve( foo(42) ) 就能够保证总会返回一个 Promise 结果。

- 看一下MDN 的 promise 文档，那个前一个 promise 会影响后一个 promise 的问题
- 3.4 链式流中的  

5. 

对多数开发者来说，错误处理最自然的形式就是同步的 try..catch 结构。遗憾的是，它只 能是同步的，无法用于异步代码模式:

```js
function foo() {
    setTimeout( function(){
        baz.bar(); }, 100 );
}

try {
    foo();
// 后面从 `baz.bar()` 抛出全局错误 
} catch (err) {
// 永远不会到达这里
}

// 改成
function foo(cb) {
    setTimeout( function(){
        try {
            var x = baz.bar();
            cb( null, x ); // 成功!
        }
        catch (err) {
            cb( err ); }
    }, 100 ); 
}
```

6. all([ .. ]) 和 race([ .. ]) 的变体

虽然原生ES6Promise中提供了内建的Promise.all([ .. ])和Promise.race([ .. ])，但这些语义还有其他几个常用的变体模式。

- none([ .. ])

  这个模式类似于all([ .. ])，不过完成和拒绝的情况互换了。所有的Promise都要被 拒绝，即拒绝转化为完成值，反之亦然。

- any([ .. ])

  这个模式与 all([ .. ]) 类似，但是会忽略拒绝，所以只需要完成一个而不是全部。

- first([ .. ])

  这个模式类似于与any([ .. ])的竞争，即只要第一个Promise完成，它就会忽略后续 的任何拒绝和完成。

- last([ .. ])

  这个模式类似于 first([ .. ])，但却是只有最后一个完成胜出。

比如，可以像这样定义 first([ .. ]):

```js
// polyfill安全的guard检查 
if (!Promise.first) {
	Promise.first = function(prs) {
    return new Promise( function(resolve,reject){
    // 在所有promise上循环 
      prs.forEach( function(pr){
    // 把值规整化
        Promise.resolve( pr )
        // 不管哪个最先完成，就决议主promise 
          .then( resolve );
        } );
      } );
  }; 
}
```

这里是只要哪个完成了，就直接在`then` 里面执行 `resolve` ，这样就实现了 `first` 的功能，那么实现`last` 呢，就在`then` 里面加个函数，执行`count++` ，当 `count === prs.length - 1` 的时候，再执行`resolve` 即可，当前的`promise` 就是 `last` 了。那`none` 就是在`then` 里面写第二个函数(也就是`reject`)，只有第二个函数里面的将`count++` 一直到 `count === prs.length - 1` 的时候，在执行 `return new Promise` 中的 `reject` 函数即可。

所以依照上面的思路，来自行实现`all/race ` 其实也没啥难度

```js
const all = (prs) => {
    let len = prs.length;
    let count = 0;
    let result = [];
    return new Promise((resolve, reject) => {
        // 为什么使用 map 而不是 forEach，因为map 的 i 是有顺序的，
        // 而 forEach 的不会按照 prs 的顺序来执行
        prs.map((pr, i) => {
            Promise.resolve(pr)
                .then(data => {
                    result[i] = data;
                    count++;
                    if (count === len - 1) {
                        resolve(result);
                    }
                });
        });
    });
}

let p1 = Promise.resolve(1);
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);

all([p1, p2, p3]).then(data => {
    console.log(data)
});
```

**当心!** 

若向Promise.all([ .. ])传入空数组，它会立即完成，但Promise. race([ .. ]) 会挂住，且永远不会决议。

Promise.all([ .. ])等待所有都完成(或者第一个拒绝)，而Promise.race([ .. ])等待 第一个完成或者拒绝。

Promise.all([]) 将会立即完成(没有完成值)，Promise.race([]) 将会永远 挂起。这是一个很奇怪的不一致，因此我建议，永远不要用空数组使用这些 方法。

## 生成器

```js
function *foo(x) {
    var y = x * (yield "hello");
    return y;
}

var it = foo();

var res = it.next();
console.log(res.value);

res = it.next(7);
console.log(res.value);
```

首先，传入 6 作为参数 x。然后调用 it.next()，这会启动 *foo(..)。

在*foo(..)内部，开始执行语句var y = x ..，但随后就遇到了一个yield表达式。它 就会在这一点上暂停 *foo(..)(在赋值语句中间!)，并在本质上要求调用代码为 yield 表达式提供一个结果值。接下来

，调用it.next( 7 )，这一句把值7传回作为被暂停的 yield 表达式的结果。

所以，这时赋值语句实际上就是var y = 6 * 7。现在，return y返回值42作为调用 it.next( 7 ) 的结果。

注意，这里有一点非常重要，但即使对于有经验的 JavaScript 开发者也很有迷惑性:根据 你的视角不同，yield 和 next(..) 调用有一个不匹配。一般来说，需要的 next(..) 调用要 比 yield 语句多一个，前面的代码片段有一个 yield 和两个 next(..) 调用。

为什么会有这个不匹配?

因为第一个 next(..) 总是启动一个生成器，并运行到第一个 yield 处。不过，是第二个 next(..) 调用完成第一个被暂停的 yield 表达式，第三个 next(..) 调用完成第二个 yield， 以此类推。

