---
layout: post
title:  "JavaScript 好问题"
author: "fedono"
---

1. [getName 题](https://github.com/Wscats/articles/issues/85)

   1. 变量提升与函数提升的区别	
      1. 变量定义的函数与全局中函数，执行时间上的区别
   2. 函数内部定义变量与原型上的变量的优先级
      1. 内部变量的优先级比原型上的优先级高
   3. 操作符的优先级，如 new 与 静态属性 的优先级
      1. 参考 [运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)
   4. new 操作执行了哪些操作 
      1. 以构造器的 prototype 属性（注意与私有字段[[prototype]]的区分）为原型，创建新对象；
      2. 将 this 和调用参数传给构造器，执行；
      3. 如果构造器返回的是对象，则返回，否则返回第一步创建的对象。

   5. new foo 与 new foo() 的区别
      1. 没有区别，这道题中还是在考察优先级时，顺带考察了这个
   
2.  [为什么 0.1 + 0.2 = 0.300000004](https://mp.weixin.qq.com/s/M2m3haos2OebXyXFzTc_6A)

   1. 这不是因为在计算时出现了错误，而是因为浮点数计算标准的要求。

   2. 编程中的浮点数的精度往往都是有限的，单精度的浮点数使用 32 位表示，而双精度的浮点数使用 64 位表示；

   3. 在使用单精度和双精度浮点数时也应该牢记它们只有 7 位和 15 位的有效位数。单精度和双精度的浮点数只包括 7 位或者 15 位的有效小数位，存储需要无限位表示的小数时只能存储近似值；单精度浮点数、双精度浮点数的位数决定了它们能够表示的精度上限

   4. 二进制无法在有限的长度中精确地表示十进制中 0.1 和 0.2；

   5. 解决办法

      ```js
      // 使用 toFixed
      +(x + y).toFixed(1)
      ```

      

