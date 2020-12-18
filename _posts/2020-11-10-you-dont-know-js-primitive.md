---
layout: post 
title: "你不知道的JS-笔记-基础类型" 
author: "fedono"
---

# JS  要点

## isNaN

isNaN(..) 有一个严重的缺陷，它的检查方式过于死板，就是“检查参数是否不是 NaN，也不是数字”。

```js
var a = 2 / "foo"; 
var b = "foo";
a; // NaN 
b; "foo"
window.isNaN( a );  // true
window.isNaN( b ); // true——晕!
```

很明显 "foo" 不是一个数字，但是它也不是 NaN。这个 bug 自 JavaScript 问世以来就一直存 在，至今已超过 19 年。

从 ES6 开始我们可以使用工具函数 Number.isNaN(..)。ES6 之前的浏览器的 polyfill 如下

```js
if (!Number.isNaN) { 
  Number.isNaN = function(n) {
    return (
      typeof n === "number" && window.isNaN( n )
    }
  ); 
};
var a = 2 / "foo"; 
var b = "foo";
Number.isNaN( a );  // true
Number.isNaN( b ); // false——好!
```

实际上还有一个更简单的方法，即利用 NaN 不等于自身这个特点。NaN 是 JavaScript 中唯 一一个不等于自身的值。

于是我们可以这样:

```js
if (!Number.isNaN) { 
  Number.isNaN = function(n) {
    return n !== n;
  };
}
```

```js
var a = 42;
var b = "foo";
a < b; // false
a > b; // false
a == b;  // false
```

为什么这三个比较结果都为假呢?这是因为 < 和 > 比较中的值 b 都被类型转换为了“无效 数字值”NaN，规范设定 NaN 既不大于也不小于任何其他值。

== 比较的结果为假的原因则不同。不论解释为 42 == NaN 还是 "42" == "foo"，都会使得 a == b 结果为假——我们已经在前文中介绍过，这种情况属于前者。

## Array

```js
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]
```

构造函数 Array(..) 不要求必须带 new 关键字。不带时，它会被自动补上。 因此 Array(1,2,3) 和 new Array(1,2,3) 的效果是一样的。

Array 构造函数只带一个数字参数的时候，该参数会被作为数组的预设长度(length)，而 非只充当数组中的一个元素。

```js
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];

a.join( "-" ); // "--"
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ] 
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

a.map(..) 之所以执行失败，是因为数组中并不存在任何单元，所以 map(..) 无从遍历。而 join(..) 却不一样，它的具体实现可参考下面的代码:

```js
function fakeJoin(arr,connector) { 
  var str = "";
	for (var i = 0; i < arr.length; i++) { 
    if (i > 0) {
      str += connector;
    }
    if (arr[i] !== undefined) {
      str += arr[i];
    }
  }
  return str; 
}
var a = new Array( 3 ); fakeJoin( a, "-" ); // "--"
```

从中可以看出，join(..) 首先假定数组不为空，然后通过 length 属性值来遍历其中的元 素。而 map(..) 并不做这样的假定，因此结果也往往在预期之外，并可能导致失败。

我们可以通过下述方式来创建包含 undefined 单元(而非“空单元”)的数组: 

```js
var a = Array.apply( null, { length: 3 } ); 
a; // [ undefined, undefined, undefined ]
```

上述代码或许会引起困惑，下面大致解释一下。

apply(..) 是一个工具函数，适用于所有函数对象，它会以一种特殊的方式来调用传递给 它的函数。

第一个参数是 this 对象(《你不知道的 JavaScript(上卷)》的“this 和对象原型”部分中 有相关介绍)，这里不用太过费心，暂将它设为 null。第二个参数则必须是一个数组(或 者类似数组的值，也叫作类数组对象，array-like object)，其中的值被用作函数的参数。

于是 Array.apply(..) 调用 Array(..) 函数，并且将 { length: 3 } 作为函数的参数。 我们可以设想 apply(..) 内部有一个 for 循环(与上述 join(..) 类似)，从 0 开始循环到

length(即循环到 2，不包括 3)。

假设在 apply(..) 内部该数组参数名为 arr，for 循环就会这样来遍历数组:arr[0]、 arr[1]、arr[2]。然而，由于{ length: 3 }中并不存在这些属性，所以返回值为 undefined。

换句话说，我们执行的实际上是 Array(undefined, undefined, undefined)，所以结果是单 元值为 undefined 的数组，而非空单元数组。

虽然Array.apply( null, { length: 3 } )在创建undefined值的数组时有些奇怪和繁琐， 但是其结果远比 Array(3) 更准确可靠。

总之，永远不要创建和使用空单元数组。

## 对象

有些原生原型(native prototype)并非普通对象那么简单:

```js
typeof Function.prototype;  // "function"
Function.prototype();  // 空函数!

RegExp.prototype.toString();  // "/(?:)/"——空正则表达式 
"abc".match( RegExp.prototype ); // [""]
```

更糟糕的是，我们甚至可以修改它们(而不仅仅是添加属性):

```js
Array.isArray( Array.prototype );  // true
Array.prototype.push( 1, 2, 3 );  // 3
Array.prototype; // [1,2,3]

// 需要将Array.prototype设置回空，否则会导致问题!
 Array.prototype.length = 0;
```

这里，Function.prototype 是一个函数，RegExp.prototype 是一个正则表达式，而 Array. prototype 是一个数组。是不是很有意思?

**将原型作为默认值** 

Function.prototype 是一个空函数，RegExp.prototype 是一个“空”的正则表达式(无 任何匹配)，而 Array.prototype 是一个空数组。对未赋值的变量来说，它们是很好的默 认值。

**Object.assign** 

ES6 新增了 Object.assign(..)，这是这些算法的简化版本。第一个参数是 target，其他 传入的参数都是源，它们将按照列出的顺序依次被处理。对于每个源来说，它的可枚举 和自己拥有的(也就是不是“继承来的”)键值，包括符号都会通过简单 = 赋值被复制。 Object.assign(..) 返回目标对象。

也就是如果 `enumerable: false ` 则不会将当前属性复制到目标属性中。

### 属性排序 

在 ES6 之前，一个对象键 / 属性的列出顺序是依赖于具体实现，并未在规范中定义。一般 来说，多数引擎按照创建的顺序进行枚举，虽然开发者们一直被强烈建议不要依赖于这个 顺序。

对于 ES6 来说，拥有属性的列出顺序是由 [[OwnPropertyKeys]] 算法定义的(ES6 规范， 9.1.12 节)，这个算法产生所有拥有的属性(字符串或符号)，不管是否可枚举。这个顺 序 只 对 `Reflect.ownKeys(..)` ( 以 及 扩 展 的 `Object.getOwnPropertyNames(..)` 和 `Object. getOwnPropertySymbols(..)`)有保证。

其顺序为:

- 首先，按照数字上升排序，枚举所有整数索引拥有的属性; 

- 然后，按照创建顺序枚举其余的拥有的字符串属性名;
- 最后，按照创建顺序枚举拥有的符号属性。

```js
var o = {}; 
o[Symbol("c")] = "yay";
o[2] = true; 
o[1] = true;
o.b = "awesome"; 
o.a = "cool";

Reflect.ownKeys( o ); // [1, 2, "b", "a", Symbol(c)]
Object.getOwnPropertyNames( o );  // [1, 2, "b", "a"]
Object.getOwnPropertySymbols( o ); // [Symbol(c)]
```



## JSON 字符串化

工具函数 JSON.stringify(..) 在将 JSON 对象序列化为字符串时也用到了 ToString。 请注意，JSON 字符串化并非严格意义上的强制类型转换，

对大多数简单值来说，JSON 字符串化和 toString() 的效果基本相同，只不过序列化的结 果总是字符串:

```js
JSON.stringify( 42 );  // "42"
JSON.stringify( "42" ); // ""42""(含有双引号的字符串) 
JSON.stringify( null );  // "null"
JSON.stringify( true ); // "true"
```

所有安全的 JSON 值(JSON-safe)都可以使用 JSON.stringify(..) 字符串化。安全的 JSON 值是指能够呈现为有效 JSON 格式的值。

为了简单起见，我们来看看什么是不安全的 JSON 值。undefined、function、symbol (ES6+)和包含循环引用(对象之间相互引用，形成一个无限循环)的对象都不符合 JSON

结构标准，支持 JSON 的语言无法处理它们。
 JSON.stringify(..) 在对象中遇到 undefined、function 和 symbol 时会自动将其忽略，在

数组中则会返回 null(以保证单元位置不变)。

```js
JSON.stringify( undefined );   // undefined
JSON.stringify( function(){} ); // undefined
JSON.stringify( [1,undefined,function(){},4]);  // "[1,null,null,4]"
JSON.stringify(
	{ a:2, b:function(){} }
  ); // "{"a": 2}"
```

对包含循环引用的对象执行 JSON.stringify(..) 会出错。

如果对象中定义了 toJSON() 方法，JSON 字符串化时会首先调用该方法，然后用它的返回 值来进行序列化。

如果要对含有非法 JSON 值的对象做字符串化，或者对象中的某些值无法被序列化时，就 需要定义 toJSON() 方法来返回一个安全的 JSON 值。

```js
var o = { };
var a = { b: 42,c: o,
         d: function(){}
     };

// 在a中创建一个循环引用 
o.e = a;
// 循环引用在这里会产生错误 // 
JSON.stringify( a );
// 自定义的JSON序列化 
a.toJSON = function() {
// 序列化仅包含b
	return { b: this.b }; 
};
JSON.stringify( a ); // "{"b":42}"
```

很多人误以为 toJSON() 返回的是 JSON 字符串化后的值，其实不然，除非我们确实想要对 字符串进行字符串化(通常不会!)。toJSON() 返回的应该是一个适当的值，可以是任何 类型，然后再由 JSON.stringify(..) 对其进行字符串化。

也就是说，toJSON() 应该“返回一个能够被字符串化的安全的 JSON 值”，而不是“返回 一个 JSON 字符串”。

例如:

```js
var a = {
  val: [1,2,3],
// 可能是我们想要的结果! 
  toJSON: function(){
	return this.val.slice( 1 ); }
};
var b = {
  val: [1,2,3],
// 可能不是我们想要的结果! toJSON: function(){
return "[" +
	this.val.slice( 1 ).join() +
"]"; }
};
JSON.stringify( a ); // "[2,3]" 
JSON.stringify( b ); // ""[2,3]""
```

这里第二个函数是对 toJSON 返回的字符串做字符串化，而非数组本身。 现在介绍几个不太为人所知但却非常有用的功能。

我们可以向 JSON.stringify(..) 传递一个可选参数 replacer，它可以是数组或者函数，用 来指定对象序列化过程中哪些属性应该被处理，哪些应该被排除，和 toJSON() 很像。

如果 replacer 是一个数组，那么它必须是一个字符串数组，其中包含序列化要处理的对象 的属性名称，除此之外其他的属性则被忽略。

如果 replacer 是一个函数，它会对对象本身调用一次，然后对对象中的每个属性各调用 一次，每次传递两个参数，键和值。如果要忽略某个键就返回 undefined，否则返回指定 的值。

```js
var a = { 
  b: 42,
  c: "42",
  d: [1,2,3] 
};
JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"
JSON.stringify( a, function(k,v){ 
  if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

JSON.string 还有一个可选参数 space，用来指定输出的缩进格式。space 为正整数时是指定 每一级缩进的字符数，它还可以是字符串，此时最前面的十个字符被用于每一级的缩进:

```js
var a = { b: 42,
c: "42",
d: [1,2,3] };
JSON.stringify( a, null, 3 );
JSON.stringify( a, null, "-----" );
```

请记住，JSON.stringify(..) 并不是强制类型转换。在这里介绍是因为它涉及 ToString 强制类型转换，具体表现在以下两点。

(1) 字符串、数字、布尔值和 null 的 JSON.stringify(..) 规则与 ToString 基本相同。
 (2) 如果传递给 JSON.stringify(..) 的对象中定义了 toJSON() 方法，那么该方法会在字符

串化前调用，以便将对象转换为安全的 JSON 值。



有一个坑常常被提到，即 [] + {} 和 {} + []，它们返回不同的结果，分别是 "[object Object]" 和 0。



## Symbol

显式和隐式强制类型转换结果通常是一样的，它们之间的差异仅仅体现在代码可读性方面。但 ES6 中引入了符号类型，它的强制类型转换有一个坑，ES6 允许 从符号到字符串的显式强制类型转换，然而隐式强制类型转换会产生错误，例如 

```js
var s1 = Symbol( "cool" );
String( s1 ); // "Symbol(cool)"

var s2 = Symbol( "not cool" ); 
s2 + ""; // TypeError
```

符号不能够被强制类型转换为数字(显式和隐式都会产生错误)，但可以被强制类型转换 为布尔值(显式和隐式结果都是 true)。

## 宽松相等和严格相等

宽松相等(loose equals)== 和严格相等(strict equals)=== 都用来判断两个值是否“相

等”，但是它们之间有一个很重要的区别，特别是在判断条件上。
 常见的误区是“== 检查值是否相等，=== 检查值和类型是否相等”。听起来蛮有道理，然而

还不够准确。很多 JavaScript 的书籍和博客也是这样来解释的，但是很遗憾他们都错了。 正确的解释是:“== 允许在相等比较中进行强制类型转换，而 === 不允许。”

== 和 === 都会检查操作数的类型。区别在于操作数类型不同时它们的处理方 式不同。

### 字符串和数字之间的相等比较

 我们沿用本章前面字符串和数字的例子来解释 == 中的强制类型转换:

```js
var a = 42; 
var b = "42";
a === b;    // false
a == b;     // true
```

因为没有强制类型转换，所以 a === b 为 false，42 和 "42" 不相等。
而 a == b 是宽松相等，即如果两个值的类型不同，则对其中之一或两者都进行强制类型转换。

具体怎么转换?是 a 从 42 转换为字符串，还是 b 从 "42" 转换为数字?

ES5 规范 11.9.3.4-5 这样定义:

- 如果 Type(x) 是数字，Type(y) 是字符串，则返回 x == ToNumber(y) 的结果。

- 如果 Type(x) 是字符串，Type(y) 是数字，则返回 ToNumber(x) == y 的结果。

### 其他类型和布尔类型之间的相等比较

 `==` 最容易出错的一个地方是 true 和 false 与其他类型之间的相等比较。 例如:

```js
var a = "42"; 
var b = true;
a == b; // false
```

规范 11.9.3.6-7 是这样说的:

- 如果 Type(x) 是布尔类型，则返回 ToNumber(x) == y 的结果;

- 如果 Type(y) 是布尔类型，则返回 x == ToNumber(y) 的结果。

### null 和 undefined 之间的相等比较

null 和 undefined 之间的 == 也涉及隐式强制类型转换。ES5 规范 11.9.3.2-3 规定:

- 如果 x 为 null，y 为 undefined，则结果为 true。

- 如果 x 为 undefined，y 为 null，则结果为 true。

在 == 中 null 和 undefined 相等(它们也与其自身相等)，除此之外其他值都不存在这种情况。

## 对象和非对象之间的相等比较

关于对象(对象 / 函数 / 数组)和标量基本类型(字符串 / 数字 / 布尔值)之间的相等比 较，ES5 规范 11.9.3.8-9 做如下规定:

(1) 如果 Type(x) 是字符串或数字，Type(y) 是对象，则返回 x == ToPrimitive(y) 的结果; 

(2) 如果 Type(x) 是对象，Type(y) 是字符串或数字，则返回 ToPromitive(x) == y 的结果。

> 这里只提到了字符串和数字，没有布尔值。原因是我们之前介绍过 11.9.3.6-7 中规定了布尔值会先被强制类型转换为数字。

```js
var a = 42;
var b = [ 42 ];
a == b; // true
```

[ 42 ] 首先调用 ToPromitive 抽象操作(参见 4.2 节)，返回 "42"，变成 "42" == 42，然后 又变成 42 == 42，最后二者相等。

“拆封”，即“打开”封装对象(如new String("abc"))，返回其 中的基本数据类型值("abc")。== 中的 ToPromitive 强制类型转换也会发生这样的情况:

```js
var a = "abc";
var b = Object( a );
a === b;  // false
a == b; // true 
```

a == b结果为true，因为b通过ToPromitive进行强制类型转换(也称为“拆封”，英文为 unboxed 或者unwrapped)，并返回标量基本类型值 "abc"，与 a 相等。 但有一些值不这样，原因是 == 算法中其他优先级更高的规则。例如:

```js
var a = null;
var b = Object( a ); // 和Object()一样
a == b; // false

var c = undefined;
var d = Object( c ); // 和Object()一样
c == d; // false

var e = NaN;
var f = Object( e ); // 和new Number( e )一样
e == f; // false
```

因为没有对应的封装对象，所以 null 和 undefined 不能够被封装(boxed)，Object(null) 和 Object() 均返回一个常规对象。

NaN能够被封装为数字封装对象，但拆封之后NaN == NaN返回false，因为NaN不等于NaN

### 比较少见的情况

更改内置原生原型会导致哪些奇怪的结果

- 返回其他数字

```js
Number.prototype.valueOf = function() { 
	return 3;
};
new Number( 2 ) == 3;   // true
```

2 == 3不会有这种问题，因为2和3都是数字基本类型值，不会调用 Number.prototype.valueOf() 方法。而 Number(2) 涉及 ToPrimitive 强制类型 转换，因此会调用 valueOf()。

- 假值的相等比较
   == 中的隐式强制类型转换最为人诟病的地方是假值的相等比较。

  下面分别列出了常规和非常规的情况:

  ```js
  "0" == null; // false
  "0" == undefined;  // false
  "0" == false; // true -- 晕!
  "0" == NaN; // false
  "0" == 0; // true
  "0" == ""; // false
  
  false == null;  // false
  false == undefined;  // false
  false == NaN; // false
  false == 0; // true -- 晕! 
  false == ""; // true -- 晕!
  false == []; // true -- 晕! 
  false == {}; // false
  
  "" == null; // false
  "" == undefined;  // false
  "" == NaN; // false
  "" == 0; // true -- 晕!
  "" == []; // true -- 晕!
  "" == {}; // false
  
  0 == null; // false
  0 == undefined; // false
  0 == NaN; // false
  0 == []; // true -- 晕! 
  0 == {}; // false
  ```

  

```js
[] + {}; // "[object Object]" 
{} + []; // 0
```

表面上看 + 运算符根据第一个操作数([] 或 {})的不同会产生不同的结果，实则不然。 第一行代码中，{} 出现在 + 运算符表达式中，因此它被当作一个值(空对象)来处理。第

4 章讲过 [] 会被强制类型转换为 ""，而 {} 会被强制类型转换为 "[object Object]"。

但在第二行代码中，{} 被当作一个独立的空代码块(不执行任何操作)。代码块结尾不需 要分号，所以这里不存在语法上的问题。最后 + [] 将 [] 显式强制类型转换(参见第 4 章) 为 0。



## 作用域

如果使用关键字 var 声明一个变量，那么这个变量就属于当前的函数作用域，如果声明是 发生在任何函数外的顶层声明，那么这个变量则属于全局作用域。

### 块作用域函数

从 ES6 开始，块内声明的函数，其作用域在这个块内。在 ES6 之前，规范并没有要求这一

点，但是许多实现就是这么做的。所以现在是规范与现实保持一致了。 考虑:

```js
{
  foo(); // 可以这么做!
  function foo() { // ..
	} 
}
foo(); // ReferenceError
```

foo()函数声明在{ .. }块内部，ES6支持块作用域。所以在块外不可用。但是还要注意

到它是在块内“提升”了，与 let 声明相反，后者会遇到前面介绍的 TDZ 错误陷阱。

如果编写了和前面类似的代码，并依赖于旧有的非块作用域行为，那么函数声明的块作用

域就可能会引发问题。

```js
if (something) {
  function foo() {
    console.log( "1" ); 
  }
}
else {
  function foo() { 
    console.log( "2" );
	} 
}
foo(); // ??
```

在前 ES6 环境中，不管 something 的值是什么，foo() 都会打印出 "2"，因为两个函数声明都被提升到了块外，第二个总是会胜出。而在 ES6 中，最后一行会抛出一个 ReferenceError。

## 默认参数值

可能 JavaScript 最常见的一个技巧就是关于设定函数参数默认值的。多年以来我们实现这

一点的方式是这样的，看起来应该很熟悉:

```js
function foo(x,y) {
  x = x || 11;
  y = y || 31;
  
  console.log( x + y );
}

foo(); // 42
foo( 5, 6 ); // 11
foo( 5 ); // 36
foo( null, 6 ); // 17
```

当然，如果你之前使用过这个模式，就会知道它很有用，但同时又有点危险，比如，如果 对于一个参数你需要能够传入被认为是 falsy(假)的值。考虑:

```js
foo( 0, 42 ); // 53 <--哎呀，并非42
```

为什么?因为这里 0 为假，所以 x || 11 结果为 11，而不是直接传入的 0。 要修正这个问题，有些人会选择增加更多的检查，就像下面这样

```js
function foo(x,y) {
  x = (x !== undefined) ? x : 11;
  y = (y !== undefined) ? y : 31;
  console.log( x + y ); 
}

foo( 0, 42 );           // 42
foo( undefined, 6 );    // 17
```

了解了所有这些之后，现在我们可以讨论 ES6 新增的一个有用的语法来改进为缺失参数赋 默认值的流程。

```js
function foo(x = 11, y = 31) { 
  console.log( x + y );
}
foo(); // 42
foo( 5, 6 ); // 11
foo( 0, 42 ); // 42

foo( 5 );  // 36
foo( 5, undefined );  // 36 <-- 丢了undefined
foo( 5, null );  // 5  <-- null被强制转换为0
foo( undefined, 6 ); // 17  <-- 丢了undefined
foo( null, 6 ); // 6 <-- null被强制转换为0
```

注意这些结果，以及它们和前面的方法之间的微妙区别和相似之处。在函数声明中的 x = 11 更像是 x !== undefined ? x : 11 而不是常见技巧 x || 11，所以 在把前 ES6 代码转换为 ES6 默认参数值语法的时候要格外小心。


## 解构

重复赋值

对象解构形式允许多次列出同一个源属性(持有值类型任意)。举例来说:

```js
var { a: X, a: Y } = { a: 1 };
X; // 1 
Y; // 1
```

这也意味着可以解构子对象 / 数组属性，同时捕获子对象 / 类的值本身。考虑:

```js
var { a: { x: X, x: Y }, a } = { a: { x: 1 } };
X;  // 1
Y;  // 1
a;  // { x: 1 }
```

## 导入导出

```js
export default function foo(..) { // ..
}
```

尽管这里的`function foo..`部分严格说是一个函数表达式，但由于模块内部 作用域的原因，它被当作函数声明对待，因为 foo 名称和模块的最高级作用 域绑定(通常称为“hoisting”)。`export default class Foo..`也是这样。然 而，虽然你可以`export var foo = ..`，但是现在还不能`export default var foo = ..`(或者 let 或者 const)，这种不一致很令人烦恼。在编写本部分时， 已经有讨论建议为了一致性尽快在后 ES6 时期增加这个功能。

## 类

function Foo是“提升的”， 而class Foo并不是;extends ..语句指定了一个不能被“提升”的表达式。所以，在 实例化一个 class 之前必须先声明它。

### static 

当子类 Bar 从父类 Foo 扩展的时候，我们已经看到 Bar.prototype 是 [[Prototype]] 链接到

Foo.prototype 的。然而，Bar() 也 [[Prototype]] 链接到 Foo()。这一点可能并不显而易见。 但是，在为一个类声明了 static 方法(不只是属性)的情况下这就很有用了，因为这些是

直接添加到这个类的函数对象上的，而不是在这个函数对象的 prototype 对象上。考虑:

```js
class Foo {
  static cool() {
    console.log( "cool" ); 
  }
   wow() { 
     console.log( "wow" )
   }
}

class Bar extends Foo {
    static awesome() {
			super.cool(); 
      console.log( "awesome" );
		}
  
  neat() {
    super.wow();
    console.log( "neat" ); 
  }
  
Foo.cool(); // "cool"
Bar.cool();  // "cool"
Bar.awesome();  // "cool"   "awesome"
var b = new Bar(); 
b.neat(); // "wow"  // "neat"
b.awesome;   ;// undefined
b.cool;  // undefined
```

