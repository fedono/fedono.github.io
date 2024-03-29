在了解操作系统前，你需要先知道一下什么是计算机系统：现代计算机系统由**一个或多个处理器、主存、打印机、键盘、鼠标、显示器、网络接口以及各种输入/输出设备构成的系统**。这些都属于`硬件`的范畴。我们程序员不会直接和这些硬件打交道，并且每位程序员不可能会掌握所有计算机系统的细节。

所以计算机科学家在硬件的基础之上，安装了一层软件，这层软件能够根据用户输入的指令达到控制硬件的效果，从而满足用户的需求，这样的软件称为 `操作系统`，它的任务就是为用户程序提供一个更好、更简单、更清晰的计算机模型。也就是说，操作系统相当于是一个中间层，为用户层和硬件提供各自的借口，屏蔽了不同应用和硬件之间的差异，达到统一标准的作用。



计算机组成指的是系统结构的逻辑实现，包括机器机内的数据流和控制流的组成及逻辑设计等。 

主要分为五个部分：控制器，运算器，存储器，输入设备，输出设备。





- 操作系统也是一种软件，但是操作系统是一种非常复杂的软件。操作系统提供了几种抽象模型
  - 文件：对 I/O 设备的抽象
  - 虚拟内存：对程序存储器的抽象
  - 进程：对一个正在运行程序的抽象
  - 虚拟机：对整个操作系统的抽象



看一下《计算机操作系统》第四版的目录，就能够总结出来，操作系统包含哪些部分

- 进程（包含进程调度
- 存储器（包含了虚拟存储
- 输出输出（I/O 设备
- 文件管理 
- 磁盘存储
- 操作系统接口（就是 shell 这些
- 多媒体文件操作、管理、调度等 

## 参考

- [5万字、97 张图总结操作系统核心知识点](https://www.cnblogs.com/cxuanBlog/p/13297199.html)

- 怎样学习和理解计算机组成原理？ - 小林coding的回答 - 知乎 https://www.zhihu.com/question/20706264/answer/2020100029

- [计算机速成课 Crash Course Computer Science](https://www.bilibili.com/video/BV1iK411g7iK/?spm_id_from=333.788.recommend_more_video.0)
