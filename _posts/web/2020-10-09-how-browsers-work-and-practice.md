---
layout: post 
title: "浏览器工作原理与实践 笔记" 
author: "fedono"
---

- 进程和线程之间的关系有以下 4 个特点

  - **进程中的任意一线程执行出错，都会导致整个进程的崩溃**

  - **线程之间共享进程中的数据。**

  - **当一个进程关闭之后，操作系统会回收进程所占用的内存**

  - **进程之间的内容相互隔离**

    进程之间的数据是严格隔离的，所以一个进程如果崩溃了，或者挂起了，是不会影响到其他进程的。如果进程之间需要进行数据的通信，这时候，就需要使用用于进程间通信（IPC）的机制了。

- head头部引入的js文件，也是先编译的吗

我先来解释下页面在含有JavaScript的情况下DOM解析流程，然后再来解释你这个问题。

当从服务器接收HTML页面的第一批数据时，DOM解析器就开始工作了，在解析过程中，如果遇到了JS脚本，如下所示：

```html
<html>
    <body>
        极客时间

        <script>
        document.write("--foo")
        </script>

​    </body>
</html>
```

那么DOM解析器会先执行JavaScript脚本，执行完成之后，再继续往下解析。

那么第二种情况复杂点了，我们内联的脚本替换成js外部文件，如下所示：

```html
<html>
    <body>
        极客时间
        <script type="text/javascript" src="foo.js"></script>
    </body>
</html>
```

这种情况下，当解析到JavaScript的时候，会先暂停DOM解析，并下载foo.js文件，下载完成之后执行该段JS文件，然后再继续往下解析DOM。这就是JavaScript文件为什么会阻塞DOM渲染。

我们再看第三种情况，还是看下面代码：

```html
<html>
    <head>
        <style type="text/css" src = "theme.css" />
    </head>
    <body>
        <p>极客时间</p>
        <script>
            let e = document.getElementsByTagName('p')[0]
            e.style.color = 'blue'
        </script>
    </body>
</html>
```

当我在JavaScript中访问了某个元素的样式，那么这时候就需要等待这个样式被下载完成才能继续往下执行，所以在这种情况下，CSS也会阻塞DOM的解析。

所以这时候如果头部包含了js文件，那么同样也会暂停DOM解析，等带该JavaScript文件下载后，便开始编译执行该文件，执行结束之后，才开始继续DOM解析。

- 关于同名变量和函数的两点处理原则：

  1:如果是同名的函数，JavaScript编译阶段会选择最后声明的那个。

  2:如果变量和函数同名，那么在编译阶段，变量的声明会被忽略