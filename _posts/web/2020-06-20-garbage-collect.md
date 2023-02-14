---
layout: post 
title: "内存泄漏与垃圾回收" 
author: "fedono"
---

内存泄漏通常和垃圾回收是放在一起的，没有回收的内存，也就是垃圾回收没有回收到，就导致内存泄漏了。

## 内存处理

### 内存的生命周期

- 内存分配：声明变量、函数、对象的时候，JS 会自动分配内存

  ```js
  cosnt n = 123; // 给数值变量分配内存
  const s = 'jack'; // 给字符串分配内存
  
  const o = {
    a: 1, 
    b: null
  }; // 给对象及包含的值分配内存
  ```

- 内存使用：即读写内存，也就是使用变量、函数等

  使用值的过程实际上是对分配内存进行读取与写入的操作。读取与写入可能是写入一个变量或者一个对象的属性值，甚至传递函数的参数。

  ```js
  var a = 10; // 分配内存
  console.log(a); // 对内存的使用
  ```

- 内存回收：使用完毕，由垃圾回收机制自动回收不再使用的内存

  垃圾回收算法主要依赖于引用的概念。

  在内存管理的环境中，一个对象如果有访问另一个对象的权限（隐式或者显式），叫做一个对象引用另一个对象。

  例如，一个 JavaScript 对象具有对它原型的引用（隐式引用）和对它属性的引用（显式引用）。

  在这里，“对象”的概念不仅特指 JavaScript 对象，还包括函数作用域（或者全局词法作用域）

  

  **1. 引用计数垃圾回收**

  引用计数算法定义“内存不再使用”的标准很简单，就是看一个对象是否有指向它的引用。如果没有其他对象指向它了，就说明对象已经不再需要了。

  但它存在一个致命的问题：循环引用。

  如果两个对象相互引用，尽管它们已经不再使用，垃圾回收不会进行回收，导致内存泄漏。

  

  **2. 标记清除算法**

  标记清楚算法将“不再使用的对象”定义为“无法达到的对象”。简单来说，就是从根部（在JS中就是全局对象）出发定时扫描内存中的对象。凡是能从根部到达的对象，都是还需要使用的。

  那些无法由根部出发触及到的对象被标记为不再使用，稍后进行回收。

  - 垃圾收集器在运行的释藏会给存储在内存中的所有变量加上标记
  - 从根部出发将能触及到的对象标记清除
  - 那些还存在标记的变量被视为准备删除的变量
  - 最后垃圾收集器会执行最后一步内存清除的工作，销毁那些带标记的值并回收它们所占用的内存空间

  

### 有哪些常见的内存泄漏

- 全局变量

  ```js
  function foo() {
    bar1 = 'some text'; // 没有声明变量，实际上是全局变量 => window.bar1
    
    this.bar2 = 'some text 2'; // 全局变量 => window.bar2
  }
  
  foo()
  ```

- 未被清除的定时器和回调

  如果后续 renderer 元素被移除，整个定时器实际上没有任何作用。但如果你没有回收定时器，整个定时器依然有效，不但定时器无法被内存回收，定时器函数中的依赖也无法回收。在这里案例中的 sercerData 也无法被回收

  ```js
  var serverData = loadData();
  
  setInterval(function() {
    var renderer = document.documentElementById('renderer');
    
    if (renderer) {
      renderer.innerHTML = JSON.stringify(serverData);
    }
  }, 5000);
  ```

- 闭包

  在 JS 开发中，我们经常会使用到闭包，一个内部函数，有权访问包含其的外部函数中的变量，下面这种情况，闭包也会造成内存泄漏。
  
  ```js
var theThing = null;
  
  var replaceThing = function() {
    var originalThing = theThing;
    var unused = function() {
      if (originalThing) { // 对于 originalThing 的引用
        console.log("hi"); 
      }
    };
    
    theThing = {
      longStr: new Array(1000000).join('*'),
      someMethod: function() {
        console.log('message');
      }
    }
  }
  
  setInterval(replaceThink, 1000);
  ```
  
  这段代码，每次调用 replaceThing 时，theThing 获得了包含一个巨大的数组和一个对于新闭包 someMethod 的对象。同时 unused 是一个引用了 originalThing 的闭包
  
  这个范例的关键在于，闭包之间是共享作用域的，尽管 unused 可能一直没有被调用，但是 someMethod 可能会被调用，就会导致无法对其内存进行回收。当这段代码被反复执行时，内存会持续增长。
  
- DOM 引用

  很多时候，我们对 DOM 的操作，会把 DOM 的引用保存在一个数组或者 Map 中

  ```js
  var elements = {
    imgage: document.getElementById('image')
  };
  
  function doStuff() {
    elements.image.src = 'http://example.com/image_name.png';
  }
  
  function removeImage() {
    document.body.removeChild(document.getElementById('image'));
    // 这个时候我们对于 #image 仍然有一个引用， Image 元素，仍然无法被内存回收
  }
  ```

  上述案例中，即使我们对于 image 元素进行了移除，但是仍然有对 image 元素的引用，依然无法对其进行内存回收。

  所以需要在 removeImage() 函数内设置

  ```js
  elements.image = null; 
  ```

### 如何避免内存泄漏

- 减少不必要的全局变量，使用严格模式避免意外创建全局变量
- 在你使用完数据后，及时解除引用（闭包中的变量，DOM 引用，定时器清除）
- 组织好你的逻辑，避免死循环等造成浏览器卡顿，崩溃的问题



**拓展 **

1. 既然对内存这么了解，那么来实现一个函数，用来获取传入的对象所占的Bytes数值

> 前置知识：一个 number 占据 8个 Byte，一个Boolean 占据4个Byte，一个String 占据2个Byte

```js
var sizeof = sizeof || {};
sizeof.sizeof = function(object, pretty) {
    var objectList = [];
    var stack = [object];
    var bytes = 0;

    while (stack.length) {
        var value = stack.pop();

        if(typeof value === 'boolean'){
            bytes += 4;
        }
      	else if(typeof value === 'string'){
            bytes += value.length * 2;
        }
      	else if(typeof value === 'number'){
            bytes += 8;
        }
      	// 在对象中，如果 value 指向同一个对象，则只计算一次，所以使用 objectList 来判断
      	else if(typeof value === 'object' && objectList.indexOf( value ) === -1){
          	// objectList 来存储对象
            objectList.push(value);
          	
          	// 在对象中，所有的 key 都是字符串，所以 key 需要单独计算
            // if the object is not an array, add the sizes of the keys
            if (Object.prototype.toString.call(value) != '[object Array]'){
                for(var key in value) {
                  bytes += 2 * key.length;
                }
            }
            for(var key in value) stack.push(value[key]);
        }
    }
    return pretty ? sizeof.format(bytes) : bytes;
}
sizeof.format = function(bytes){
    if(bytes < 1024) return bytes + "B";
    else if(bytes < 1048576) return(bytes / 1024).toFixed(3) + "K";
    else if(bytes < 1073741824) return(bytes / 1048576).toFixed(3) + "M";
    else return(bytes / 1073741824).toFixed(3) + "G";
}
exports = module.exports = sizeof;
```

来源 [sizeof](https://github.com/lyroyce/sizeof/blob/master/lib/sizeof.js) 库

2. 如何来使用 weakMap/weakSet 来达到节省内存的效果





## 垃圾回收

垃圾回收算法主要依赖于引用的概念。在内存管理的环境中，一个对象如果有访问另一个对象的权限（隐式或者显式），叫做一个对象引用另一个对象。例如，一个Javascript对象具有对它[原型](https://developer.mozilla.org/en/JavaScript/Guide/Inheritance_and_the_prototype_chain)的引用（隐式引用）和对它属性的引用（显式引用）。

在这里，“对象”的概念不仅特指 JavaScript 对象，还包括函数作用域（或者全局词法作用域）。

**重点**

- 垃圾回收有哪些方式

- 浏览器和node的垃圾回收方式

- 闭包如何实现释放内存

  > 闭包内引用的外部变量，设置为 null



### 引用计数

这是最初级的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

#### 限制：循环引用

该算法有个限制：无法处理循环引用的事例。在下面的例子中，两个对象被创建，并互相引用，形成了一个循环。它们被调用之后会离开函数作用域，所以它们已经没有用了，可以被回收了。然而，引用计数算法考虑到它们互相都有至少一次引用，所以它们不会被回收。



### 标记-清除

这个算法把“对象是否不再需要”简化定义为“对象是否可以获得”。

这个算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。垃圾回收器将定期从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和收集所有不能获得的对象。

这个算法比前一个要好，因为“有零引用的对象”总是不可获得的，但是相反却不一定，参考“循环引用”。

从2012年起，所有现代浏览器都使用了标记-清除垃圾回收算法。所有对JavaScript垃圾回收算法的改进都是基于标记-清除算法的改进，并没有改进标记-清除算法本身和它对“对象是否不再需要”的简化定义。

#### 循环引用不再是问题了

在上面的示例中，函数调用返回之后，两个对象从全局对象出发无法获取。因此，他们将会被垃圾回收器回收。第二个示例同样，一旦 div 和其事件处理无法从根获取到，他们将会被垃圾回收器回收。

#### 限制: 那些无法从根对象查询到的对象都将被清除

尽管这是一个限制，但实践中我们很少会碰到类似的情况，所以开发者不太会去关心垃圾回收机制。



短时间执行的场景，如网页应用，命令行工具，这类场景由于运行时间短，且运行在用户的机器上，即使内存使用过多或者内存泄漏，也只会影响到终端用户。由于运行时间短，随着进程的退出，内存会释放，几乎没有内存管理的必要。

基于无阻塞、事件驱动建立的 Node 服务，具有内存消耗低的优点，非常适合处理海量的网络请求。在海量请求的前提下，开发者就需要考虑一些平常不会形成影响的问题。

Node  提供了 V8 中内存使用量的查看方法`process.memoryUsage()`

```javascript
> process.memoryUsage();
{ rss: 28078080,
  heapTotal: 10207232,
  heapUsed: 6020400,
  external: 8982 }
```

`heapTotal` 和 `heapUsed` 是 `V8` 的堆内存使用情况，前者是已申请到的堆内存，后者是当前的使用量。

当我们在代码中声明变量并赋值时，所使用的对象的内存就分配在堆中。如果已申请的堆空闲内存不够分配新的对象，将继续申请堆内存，直到堆的大小超过V8的限制为止。

> 1. V8 为何要限制堆的大小
>
>    表层原因是V8最初为浏览器而设计，不太可能遇到用大量内存的场景。
>
>    深层原因是V8的垃圾回收机制的限制。按官方的说法，以1.5GB的垃圾回收堆内存为例，V8做一次小的垃圾回收需要50毫秒以上，做一次非增量式的垃圾回收甚至要1秒以上。这是垃圾回收中引起JavaScript线程暂停执行的时间，在这样的时间花销下，应用的性能和响应能力都会直接下降。这样的情况不仅仅后端服务无法接受，前端浏览器也无法接受。因此，在当去处的考虑下直接限制堆内存是一个好的选择。
>
> 2. 打开限制
>
>    V8开启了选项可以使用更多的内存。Node 在启动的时候，传递`--max-old-space-size`  或`--max-new-space-size` 来调整内存限制的大小。示例如下
>
>    ```js
>    node --max-old-space-size=1700 test.js // 单位为 MB
>    // 或者
>    node --max-new-space-size=1024 test.js // 单位为 KB
>    ```
>
>    上述参数在V8初始化时生效，一旦生效就不能再动态改变。

## V8 的垃圾回收机制

V8 的垃圾回收策略主要基于分代式垃圾回收机制。按对象的存活时间将内存的垃圾回收进行不同的分代，然后对不同分代的内存施以更高效的算法。

在V8中，主要将内存分为新生代和老生代两代。新生代中的对象为存活时间较短的对象，老生代中的对象为存活时间较长或常驻内存的对象。

V8 的整体大小就是新生代所用内存空间加上老生代的内存空间。 `--max-old-space-size` 命令行参数可以用于设置老生代内存空间的最大值， `--max-new-space-size` 命令行参数则用于设置新生代内存空间的大小。

> 这两个最大值需要在启动时就指定。这意味着V8使用的内存没有办法根据使用情况自动扩充，当内存分配过程中超过极限值时，就会引起进程出错。
>
> 在默认设置下，如果一直分配内存，在64位系统和32位系统下会分别只能使用约为1.4GB和约0.7GB的大小，老生代的设置在64位系统下为1400MB，在32位系统下为700MB，新生代内存在64位系统和32位系统上最大值分别为32MB和16MB。因此V8堆内存的最大值在64位系统上为1464MB，在32位系统上为732MB。

### 新生代对象的回收方式

在分代的基础上，新生代中的对象主要通过`Scavenge` 算法进行垃圾回收。在`Scavenge` 的具体实现中，主要采用了`Cheney` 算法。

`Cheney` 算法是一种采用复制的方式实现的垃圾回收算法，它将堆内存一分为二，每一部分空间称为 `semispace` ，在这两个 `semispace` 空间中，只有一个处于使用中，另一个处于闲置状态。处于使用状态的 `semispace` 空间称为 `From` 空间，处于闲置状态的空间称为 `To` 空间。当我们分配对象时，先是在`From` 空间中进行分配。当开始进行垃圾回收时，会检查`From` 空间中的存活对象，这些存活动向将被复制到`To` 空间中，而非存活对象占用的空间将会被释放。完成复制后，`From` 空间和`To` 空间的角色发生对象。简而言之，在垃圾回收的过程中，就是通过将存活对象在两个 `semispace` 空间之间进行复制。

`Scavenge` 算法的缺点是只能使用堆内存中的一半，这是由划分空间和复制机制所决定的。但是`Scavenge` 由于只复制存活的对象，并且对于生命周期短的场景存活对象只占少部分，所以它在时间效率上有优异的表现。

> 由于 `Scavenge` 是典型的牺牲空间换取时间的算法，所以无法大规模地应用到所有的垃圾回收中。但是可以发现，`Scavenge` 非常适合应用在新生代中，因为新生代中对象的生命周期较短，恰恰适合这个算法。

**实际使用的堆内存是新生代中的两个`semispace` 空间大小和老生代所用内存大小之和。**

当一个对象经过多次复制依然存活时，它将会被认为是声明周期较长的对象。这种较长声明周期的对象随后会被移动到老生代中，采用新的算法进行管理。对象从新生代中移动到老生代中的过程称为晋升。

在单纯的`Scavenge` 过程中，`From` 空间中的存活对象会被复制到`To` 空间中，然后对 `From` 空间和`To` 空间进行角色兑对换。但在分代式垃圾回收的前提下，`From` 空间中的存活对象复制到`To` 空间之前需要进行检查。在一定条件下，需要将存活周期长的对象移动到老生代中，也就是完成对象晋升。

对象晋升的条件主要有两个，一个是对象是否经历过`Scavenge` 回收，一个是 `To` 空间的内存占用比超过限制。

> 1. 在默认情况下，V8的对象分配主要集中在`From` 空间中，对象才`From` 空间中复制到`To` 空间时，会检查它的内存地址来判断这个对象是否已经经历过一次 `Scavenge` 回收。如果已经经历过了，会将该对象从 `From` 空间复制到老生代空间中，如果没有，则复制到`To` 空间中。
> 2. 另一个判断条件是 `To` 空间的内存占用比。当要从`From` 空间复制一个对象到`To` 空间时，如果 `To` 空间已经使用了超过`25%` ，则这个对象直接晋升到老生代空间中。设置 `25%` 这个限制值的原因是当这次 `Scavenge` 回收完成后，这个`To` 空间将变成`From` 空间，接下来的内存分配将在这个空间中进行。如果占比过高，会影响后续的内存分配。
> 3. 对象晋升后，将会在老生代空间中作为回收周期较长的对象来对待，接收新的回收算法处理。

### 老生代对象的回收方式

对于老生代中的对象，由于存活对象占较大的比重，再采用`Scavenge` 的方式会有两个问题：一个是存活对象较多，复制存活对象的效率会很低；另一个问题依然是浪费一般空间的问题。这两个问题导致应对生命周期较长的对象时，`Scavenge` 会显得捉襟见肘。为此，V8在老生代中主要采用了`Mark-Sweep` 和 `Mark-Compact` 相结合的方式进行垃圾回收。

#### `Mark-Sweep` 

`Mark-Sweep` 是标记清除的意思，它分为标记和清除两个阶段。与 `Scavenge` 相比，`Mark-Sweep` 并不将内存空间划分为两半，所以不存在浪费一半空间的行为。与 `Scavenge` 复制活着的对象不同，`Mark-Sweep` 在标记阶段遍历堆中的所有对象，并标记活着的对象，在随后的清除阶段中，只清除没有被标记的对象。可以看出，`Scavenge` 中只复制活着的对象，而 `Mark-Sweep` 只清理死亡的对象。活对象在新生代中只占较小部分，死对象在老生代中只占较小部分，这是两种回收方式能高效处理的原因。

`Mark-Sweep` 最大的问题是在进行一次标记清除回收后，内存空间会出现不连续的状态。这种内存碎片会对后续的内存分配造成问题，因为很可能出现需要分配一个大对象的情况，这时所有的碎片空间都无法完成此次分配，就会提前触发垃圾回收，而这次垃圾回收时不必要的。

#### `Mark-Compact` 

为了解决`Mark-Sweep` 的内存碎片问题，`Mark-Compact` 被提出来。`Mark-Compact` 是标记整理的意思，是在 `Mark-Sweep` 的基础上演变而来的。他们的差别在于对象在标记为死亡后，在整理的过程中，将活着的对象往一端移动，移动完成后，直接清理边界外的内存。

#### `Incremental Marking` 

为了避免出现`JavaScript` 应用逻辑与垃圾回收器看到的不一致的情况，垃圾回收的3种基本算法都需要将应用逻辑暂停下来，待执行完垃圾回收后再恢复执行应用逻辑，这种行为被称为『全停顿』(stop-the-world) 。在`V8` 的分布式垃圾回收中，一次小垃圾回收只收集新生代，由于新生代默认配置得很小，且其中存活对象通常较少，所以即便是全停顿的影响也不大。但V8的老生代通常配置得很大，且存活对象较多，全堆垃圾回收(full 垃圾回收) 的标记、清理、整理等动作造成的停顿就会比较可怕，需要设法改善。

为了降低全堆垃圾回收带来的停顿时间，V8先从标记阶段入手，将原来要一口气停顿完成的动作改为增量标记(incremental marking) ，也就是拆分为许多小"步进"，没做完一"步进"，就让`JavaScript` 应用逻辑执行一小会儿，垃圾回收与应用逻辑交替执行直到标记阶段完成。

V8在经过增量标记的改进后，垃圾回收的最大停顿时间可以减少到原本的1/6左右。

V8后续还引入了延迟清理(Lazy sweeping)与增量式整理(incremental compaction) ，让清理与整理动作也变成增量式。同时还计划引入并行标记于并行整理，进一步利用多核性能降低每次停顿的时间。

## 高效实用内存

### 变量的主动释放

如果变量是全局变量(不通过 var 声明或定义在 global 变量上)，由于全局作用域需要直到进程退出才能释放，此时将导致引用的对象常驻内存(常驻在老生代中)。如果需要释放常驻内存的对象，可以通过`delete` 操作来删除引用关系。或者将变量重新赋值，让旧的对象脱离引用关系。在接下来的老生代内存清除和整理的过程中，会被回收释放。

同样，如果在非全局作用域中，想主动释放变量引用的对象，也可以通过这样的方式。

#### 查看系统的内存占用

与 `process.memoryUsage()` 不同的是，OS模块中的 `totalmem() ` 和 `freemem()` 这两个方法用于查看操作系统的内存使用情况，它们分别返回系统的总内存和闲置内存，以字节为单位

```js
os.totalmem();
os.freemem();
```

通过 `process.memoryUsage()` 的结果可以看到，堆中的内存用量总是小于进程的常驻内存用量，这意味着Node中的内存使用并非都是通过 V8 进行分配的。我们将那些不是通过 V8 分配的内存称为堆外内存。

> Buffer 对象不同于其他对象，它不经过V8的内存分配机制，所以也不会有堆内存的大小限制。
>
> 这意味着利用堆外内存可以突破内存限制的问题。
>
> 为何 Buffer 对象并非通过 V8 分配？这在于 Node 并不同于浏览器的应用场景。在浏览器中，JavaScript 直接处理字符串即可满足绝大多数的业务需求，而 Node 则需要处理网络流和文件I/O流，操作字符串远远不能满足传输的性能需求。

Node 的内存构成主要由通过 V8 进行分配的部分和 Node 自行分配的部分，受V8 的垃圾回收限制的主要是 V8 的堆内存。

## 内存泄漏

通常，造成内存泄漏的原因有如下几个

1. 缓存

   > 如果一个对象被当做缓存来使用，那就意味着它将会常驻在老生代。缓存中存储的键越多，长期存活的对象也就越多，这将导致垃圾回收在进行扫描和整理时，对这些对象做无用功
   >
   > **解决办法**：LRU 算法，

2. 队列消费不及时

   > 有的应用收集日志存储在数据库中，数据库构建在文件系统之上，写入效率远远低于文件直接写入，于是会形成数据库写入操作的堆积，而 JavaScript 中相关的作用域也不会得到释放，内存占用不会回落，从而出现内存写入 。
   >
   > **解决办法**：
   >
   > 1. 表层的解决方案是换用消费速度更高的技术，在日志收集的案例中，换用文件写入日志的方式会更高效。需要注意的是，如果生成速度因为某些原因突然激增，或者消费速度因为突然的系统故障降低，内存泄漏还是可能出现的
   > 2. 深度的解决方案应该是监控队列的长度，一旦堆积，应当通过监控系统产生报警并通知相关人员。另一个解决方案是任意调用异步调用都应该包含超时机制，一旦在限定的时间内未完成响应，通过回调函数传递超时异常，使得任意异步调用的回调都具备可控的响应时间，给消费速度一个下限值。

3. 作用域未释放

### 内存泄漏排查

1. 使用 `node-heapdump`

   抓拍堆内存的快照

2. 使用 `node-memwatch` 

   提供抓取和比较快照的功能，它能够比较堆上对象的名称和分配数据，从而找到导致内存泄漏的元凶。

### 大内存应用

在 Node  中，不可避免地还是会存在操作大文件的场景。由于 Node 的内存限制，操作大文件也需要小心，好在 Node 提供了 stream 模块用于处理大文件。

stream 模块是 Node 的原生模块，直接引用即可。stream 继承自 EventEmitter，具备基本的自定义功能，同时抽象出标准的事件和方法。它分可读和可写两种。Node 中的大多数模块都有 stream 的应用，比如 fs 的 `createReadStream` 和 `createWriteStream` 方法可以分别用于创建文件的可读流和可写流， `process` 模块中的 `stdin` 和 `stdout` 则分别是可读流和可写流的示例。

由于 `V8` 的内存限制，我们无法通过`fs.readFile()` 和 `fs.writeFile()` 直接进行大文件的操作，而改用`fs.createReadStream` 和 `fs.createWriteStream` 方法通过流的方式实现对大文件的操作。

```js
let reader = fs.createReadStream('in.txt');
let writer = fs.createWriteStream('out.txt');
reader.pipe(writer);
```

可读流提供了管道方法 `pipe()` ，封装了`data` 事件和写入操作，通过流的方式，上述代码不会受到 V8 内存限制的影响，有效提供了程序的健壮性。

如果不需要进行字符串层面的操作，则不需要借助 V8 来处理，可以尝试进行纯粹的 Buffer 操作，这不会受到 V8 堆内存的限制。但是这种大片使用内存的情况依然要小心，即使 V8 不限制堆内存的大小，物理内存依然有限制。



##  参考

- [V8 之旅： 垃圾回收器](http://newhtml.net/v8-garbage-collection/) 
- [垃圾回收](https://zh.javascript.info/garbage-collection) 
- 《深入浅出Node.js》 第五章-内存控制。

- [mdn-内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management) 

- [内存管理](http://js.pingan8787.com/)

  - JavaScript基础（高级） -> 内存管理

- [JavaScript 内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)

- [尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)

  尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用记录，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用记录，取代外层函数的调用记录就可以了。

  > ```javascript
  > function f() {
  > let m = 1;
  > let n = 2;
  > return g(m + n);
  > }
  > f();
  > 
  > // 等同于
  > function f() {
  > return g(3);
  > }
  > f();
  > 
  > // 等同于
  > g(3);
  > ```

  上面代码中，如果函数g不是尾调用，函数f就需要保存内部变量m和n的值、g的调用位置等信息。但由于调用g之后，函数f就结束了，所以执行到最后一步，完全可以删除 f() 的调用记录，只保留 g(3) 的调用记录。

  这就叫做"尾调用优化"（Tail call optimization），即只保留内层函数的调用记录。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用记录只有一项，这将大大节省内存。这就是"尾调用优化"的意义。