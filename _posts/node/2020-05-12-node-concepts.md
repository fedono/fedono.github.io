一个有趣的问题是，为什么 Node.js 约定，回调函数的第一个参数，必须是错误对象err（如果没有错误，该参数就是 null）？原因是执行分成两段，在这两段之间抛出的错误，程序无法捕捉，只能当作参数，传入第二段。





## 参考

-  [Thunk 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/04/generator.html)

