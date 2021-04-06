当你从外部调用某个模块，require其实是在require什么？
require的时候**NodeJS**会到处去找有没有这个模块，如果有，return的就是`module.exports`里的东东。

```js
// b.js
exports.name = 'jack';

module.export = function sayName() {
  return 'wallen';
}
```

如果使用 `require(b.js)` 的时候，只会去拿 `module.export` 的 `sayName` ，拿不到`name` 属性

- exports 是 module.exports 的一个引用

- module.exports 初始值为一个空对象 {}，所以 exports 初始值也是 {}

- require 引用模块后，返回的是 module.exports 而不是 exports!!!!!

- exports.xxx 相当于在导出对象上挂属性，该属性对调用模块直接可见

- exports = 相当于给 exports 对象重新赋值，调用模块不能访问 exports 对象及其属性

  如果此模块是一个类，就应该直接赋值 module.exports，这样调用者就是一个类构造器，可以直接 new 实例



## 参考

- [export 和 module.export 的区别](https://www.jianshu.com/p/e452203d56c4) 