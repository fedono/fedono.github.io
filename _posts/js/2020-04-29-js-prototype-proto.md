---
layout: post 
title: "prototype 与 proto" 
author: "fedono"
---

`instanceof`运算符的一个用处，是判断值的类型。

```js
var x = [1, 2, 3];
var y = {};
x instanceof Array // true
y instanceof Object // true
```

上面代码中，`instanceof`运算符判断，变量`x`是数组，变量`y`是对象。

注意，`instanceof`运算符只能用于对象，不适用原始类型的值。

```js
var s = 'hello';
s instanceof String // false
```

上面代码中，字符串不是`String`对象的实例（因为字符串不是对象），所以返回`false`。

此外，对于`undefined`和`null`，`instanceof`运算符总是返回`false`。

```js
undefined instanceof Object // false
null instanceof Object // false
```

利用`instanceof`运算符，还可以巧妙地解决，调用构造函数时，忘了加`new`命令的问题。

```js
function Fubar (foo, bar) {
  if (this instanceof Fubar) {
    this._foo = foo;
    this._bar = bar;
  } else {
    return new Fubar(foo, bar);
  }
}
```

上面代码使用`instanceof`运算符，在函数体内部判断`this`关键字是否为构造函数`Fubar`的实例。如果不是，就表明忘了加`new`命令。

### __proto__

刚才留下了一个问题: Person.__proto__ 指向的是一个空函数, 下面我们来看看这个空函数究竟是什么.

```js
console.log(Person.__proto__ === Function.prototype);//true
```

Person 是构造器也是函数(function), Person的__proto__ 属性自然就指向 函数(function)的原型, 即 Function.prototype.

这说明了什么呢?

我们由 “特殊” 联想到 “通用” , 由Person构造器联想一般的构造器.

这说明 **所有的构造器都继承于Function.prototype** (此处我们只是由特殊总结出了普适规律, 并没有给出证明, 请耐心看到后面) , 甚至包括根构造器Object及Function自身。所有构造器都继承了Function.prototype的属性及方法。如length、call、apply、bind（ES5）等. 如下:

```js
console.log(Number.__proto__   === Function.prototype); // true
console.log(Boolean.__proto__  === Function.prototype); // true
console.log(String.__proto__   === Function.prototype); // true
console.log(Object.__proto__   === Function.prototype); // true
console.log(Function.__proto__ === Function.prototype); // true
console.log(Array.__proto__    === Function.prototype); // true
console.log(RegExp.__proto__   === Function.prototype); // true
console.log(Error.__proto__    === Function.prototype); // true
console.log(Date.__proto__     === Function.prototype); // true
```

JavaScript中有内置(build-in)构造器/对象共计13个（[ES5](http://kangax.github.com/es5-compat-table/)中新加了JSON），这里列举了可访问的9个构造器。剩下如Global不能直接访问，Arguments仅在函数调用时由JS引擎创建，Math，JSON是以对象形式存在的，无需new。由于任何对象都拥有 __proto__ 属性指向构造器的原型. 即它们的 __proto__ 指向Object对象的原型(Object.prototype)。如下所示:

```js
console.log(Math.__proto__ === Object.prototype);  // true
console.log(JSON.__proto__ === Object.prototype);  // true
```

如上所述, 既然所有的构造器都来自于Function.prototype, 那么Function.prototype 到底是什么呢?

### Function.prototype

我们借用 typeof 运算符来看看它的类型.

```js
console.log(typeof Function.prototype) // "function"
```

实际上, Function.prototype也是唯一一个typeof XXX.prototype为 “function”的prototype。其它的构造器的prototype都是一个对象。如下:

```js
console.log(typeof Number.prototype)   // object
console.log(typeof Boolean.prototype)  // object
console.log(typeof String.prototype)   // object
console.log(typeof Object.prototype)   // object
console.log(typeof Array.prototype)    // object
console.log(typeof RegExp.prototype)   // object
console.log(typeof Error.prototype)    // object
console.log(typeof Date.prototype)     // object
```

### JS中函数是一等公民

既然Function.prototype 的类型是函数, 那么它会拥有 __proto__ 属性吗, Function.prototype.__proto__ 会指向哪里呢? 会指向对象的原型吗? 请看下方:

```js
console.log(Function.prototype.__proto__ === Object.prototype) // true
```

透过上方代码, 且我们了解到: Function.prototype 的类型是函数, 也就意味着一个函数拥有 __proto__ 属性, 并且该属性指向了对象(Object)构造器的原型. 这意味着啥?

根据我们在 `概念2` 中了解到的: **__proto__** 是对象的内部属性, 它指向构造器的原型.

这意味着 Function.prototype 函数 拥有了一个对象的内部属性, 并且该属性还恰好指向对象构造器的原型. 它是一个对象吗? 是的, 它一定是对象. 它必须是.

实际上, JavaScript的世界观里, 函数也是对象, 函数是一等公民.

这说明所有的构造器既是`函数`也是一个普通JS`对象`，可以给构造器添加/删除属性等。同时它也继承了Object.prototype上的所有方法：toString、valueOf、hasOwnProperty等。

可以看下 [mdn Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) 中，`Function` 的 `Inheritance` 是 `Object` 



最后肯定是要放一张图的

![img](../../assets/imgs/prototype/prototype.jpg)



上面这张图中，我没有明白底下的 `function Function()` 和`function Object()`为什么会有这种东西？ 

> 应该是 JS  中内置的 Function 函数和 Object 函数



好像终于把上面那张图看懂了，这张图的入口点有五个

1. `let foo = new Foo()`  中 `foo` 的原型链

   ```js
   1. Foo.prototype
   2. Object.prototype
   3. null
   ```

2. `function Foo(){}` 的原型链

   ```
   1. Function.prototype
   2. Object.prototype
   3. null
   ```

3. `let obj = new Object()` 中 `obj` 的原型链

   ```
   1. Object.prototype
   2. null
   ```

4. `JS` 内置函数 `function Object() ` 的原型链

   >  为什么`Object` 是个函数，因为可以使用 `new Object() `  

   ```
   1. Function.prototype
   2. Object.prototype
   3. null
   ```

5. `JS` 内置函数 `function Function() ` 的原型链

   ```
   1. Function.prototype
   2. Object.prototype
   3. null
   ```

   

```js
function Foo() {}

var obj = new Foo();
// 写一下 Foo 和 obj 的原型链

let proto = Object.getPrototypeOf(obj);
let protoChain = [proto];
while(proto) { // 原型链的顶端就是 null，
  proto = Object.getProtypeOf(proto);
  protoChain.push(proto);
}

console.log(protoChain) // 这样，就能够拿到 obj 的原型链了
// 输出为 [ Foo {}, {}, null ]

// 但是因为是原型链，所以其实在上述打印出来的数组中的第一个和第二个都应该为 Foo.prototype 和 Object.prototype，第三个就是 null 

// Foo 的原型链使用上述方式输出为 [ [Function], {}, null ]
// 但根据上面那张图，应该为 Function.prototype 和 Object.prototype ，然后再是 null
```



这三个概念还是没有搞懂 

> 现在搞懂了

- `getPrototypeOf` 
  
- 相当于 `__proto__` ，通过`__proto__` 来找原型链的时候，现在可以使用 `getPrototypeOf` 来找了，同时`Object`还有一个 `setPrototypeOf` 方法 
  
- `isPrototypeOf` 

  - `isPrototypeOf()` 方法用于测试一个对象是否存在于另一个对象的原型链上
  - 有了上面的 `getProgotypeOf` ，这个`isPrototypeOf` 就很好理解了

- `instanceof` 

  > `isPrototypeOf()` 与 [`instanceof`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof) 运算符不同。在表达式 "`object instanceof AFunction`"中，`object` 的原型链是针对 `AFunction.prototype` 进行检查的，而不是针对 `AFunction` 本身。
  >
  > 也就是 `prototypeObj.isPrototypeOf(AObject)` 中，括号内针对的是`AObject` 本身 



**为什么不能直接在 string 上加上 prototype 的方法**

日常拓展 String 和 Array 方法的方式为 `Array.prototype.add` ，也就是直接在 `Array` 的的原型上添加，为什么可以这样添加，因为 `Array` 是个 `function`， 可以在浏览器的控制台是输入 `Array` 可以看到返回的是个 `function` ，但是在 `[1, 2]` 上添加肯定是不行的，因为这个会 `return ` 回来的不是个 `function` ，这个应该要看下重学前端系列里，好像有说这个

但是却可以这样给 `obj` 替换掉`toString` 和 `valueOf` 

```js
var x = {
    toString: function () { return "test"; },
    valueOf: function () { return 123; }
};

console.log(x); // test ，现在的浏览器已经返回 { toString: [Function: toString], valueOf: [Function: valueOf] }
console.log("x=" + x); // "x=123"
console.log(x + "=x"); // "123=x"
console.log(x + "1"); // 1231
console.log(x + 1); // 124
console.log(["x=", x].join("")); // "x=test"
```

当 “+” 操作符一边为数字时，对象x趋向于转换为数字，表达式会优先调用 valueOf 方法，如果调用数组的 join 方法，对象x趋向于转换为字符串，表达式会优先调用 toString 方法。



参考

- [详解prototype与__proto__](http://louiszhai.github.io/2015/12/17/prototype/) 

  > 感觉看这篇文章就足够了

  可以使用 `Object.prototype.isPrototypeOf()` 来判断当前对象是否为另一个对象的原型 

- [对象的继承](https://wangdoc.com/javascript/oop/prototype.html) 

- [对象](https://wangdoc.com/javascript/oop/object.html)

- 结合[function](./2020-04-21-js-function.md) 和 [prototype](./2020-04-27-prototype.md) 一起看

- [Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html) 

  这篇需要好好读下，`prototype` 设计之初，就是为了数据共享，使用 `new fn()` 生成不同的实例，在 `fn` 内会有每个实例自己的参数，但是 `prototype` 就是多个实例的数据共享了

- [Javascript面向对象编程（二）：构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html) 

  我一直都没有明白在使用实例来实现继承的时候，下面几行代码的含义，在这篇文章中，终于看懂了，感谢阮一峰老师

  ```js
  var F = function(){};
  F.prototype = Parent.prototype;
  Child.prototype = new F();
  
  // 使用上述三行代码通常都是为了继承，也就是继承父类的 prototype，但是又需要在修改子类的 prototype 的时候，不要影响到父类，这时候通过创建一个中间函数，来实现修改子类的时候，不影响父类的属性改变
  
  function extend(Child, Parent) {
    var F = function(){};
  	F.prototype = Parent.prototype;
  	Child.prototype = new F();
  }
  ```

  

  