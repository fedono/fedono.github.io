内存泄漏

典型的就是定义了变量，但是没有消除，导致内存常驻，最终导致内存占用越来越高，影响系统性能





## 参考

- [mdn-内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management) 

- [内存管理](http://js.pingan8787.com/)

  - JavaScript基础（高级） -> 内存管理

- [JavaScript 内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)

- [尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)

  尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用记录，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用记录，取代外层函数的调用记录就可以了。

  > ```javascript
  > function f() {
  >   let m = 1;
  >   let n = 2;
  >   return g(m + n);
  > }
  > f();
  > 
  > // 等同于
  > function f() {
  >   return g(3);
  > }
  > f();
  > 
  > // 等同于
  > g(3);
  > ```

  上面代码中，如果函数g不是尾调用，函数f就需要保存内部变量m和n的值、g的调用位置等信息。但由于调用g之后，函数f就结束了，所以执行到最后一步，完全可以删除 f() 的调用记录，只保留 g(3) 的调用记录。

  这就叫做"尾调用优化"（Tail call optimization），即只保留内层函数的调用记录。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用记录只有一项，这将大大节省内存。这就是"尾调用优化"的意义。