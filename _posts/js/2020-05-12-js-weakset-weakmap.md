及时清除引用非常重要。但是，你不可能记得那么多，有时候一疏忽就忘了，所以才有那么多内存泄漏。

最好能有一种方法，在新建引用的时候就声明，哪些引用必须手动清除，哪些引用可以忽略不计，当其他引用消失以后，垃圾回收机制就可以释放内存。这样就能大大减轻程序员的负担，你只要清除主要引用就可以了。

ES6 考虑到了这一点，推出了两种新的数据结构：[WeakSet](http://es6.ruanyifeng.com/#docs/set-map#WeakSet) 和 [WeakMap](http://es6.ruanyifeng.com/#docs/set-map#WeakMap)。它们对于值的引用都是不计入垃圾回收机制的，所以名字里面才会有一个"Weak"，表示这是弱引用。





## 参考文档

- [JavaScript 内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)

