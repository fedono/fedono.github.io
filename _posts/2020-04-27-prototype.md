为什么不能直接在 string 上加上 prototype 的方法

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





## 参考

- [详解prototype与__proto__](http://louiszhai.github.io/2015/12/17/prototype/) 

