- JavaScript在发布v1.0的时候，仅有结构化语言的特性以及一些基础的面向对象特性（即structured and object based programming）。到v1.3时有了本质性的改观，使得这一语言成为包括动态语言、函数式语言特性在内的特性丰富的混合语言。这一切被ECMA通过标准化的形式确定下来，形成了以JavaScript 1.5和ECMAScript Ed3版本（即JS1.5/ES3.1）为代表的，一直到我们今天仍在使用的JavaScript语言。

- 严格地说，ES5与ES4基本没有什么关系，而是对ES3.1所代表的语言方向的一个补充。换言之，ES5没有改变JavaScript 1.x的语言特性，而ES4则是一门集语言生产商所有创想之大成且又与JavaScript 1.x倡导的语言风格相去甚远的语言。历史证明，我们暂时抛弃了后者—过去10年，我们都未能将JavaScript 1.x推进到v2.0版本。

  由于ES5秉承了JavaScript设计的原始思想，因此基于ES5又展开了新一轮的语言进化的角力（图1-6中的三条虚线试图说明，语言最初的特性设计是这一切的根源）：

  ■ 第一个方向由Microsoft、Adobe等大厂商所主导，沿着ES4—或称为JS2、或ECMAScript Harmony—的方向，向更加丰富的面向对象（OO++）特性发展，主要试图解决大型系统开发中所需要的复杂的对象层次结构、类库以及框架。

  ■ 第二个方向，则由包括Brendan Eich本人在内的语言开发者、研究者主导，他们力图增强JavaScript语言的函数式特性（Functional++），因为这样的语言特性在解决许多问题时来得比结构化的、面向对象的语言更优雅有效，而且从语言角度看来，函数式更为“纯粹”。

  ■ 在第三个方向上，Common JS等开发组意识到JS1.x在应用于浏览器之外的开发场景中，以及在组织大型项目方面显得无力。而将这一问题归结起来，就是“缺乏基础运行框架和运行库”，于是通过参考传统的、大型的、系统级的应用开发语言，尝试性地提出了在JavaScript中的同等解决方案（System++）。

  具体的引擎或框架已经不再是被关注的话题。ES4的失败给整个JavaScript领域带来的思考是：我们究竟需要一种怎样的语言？

- 社区强大而丰富的能量灌注以及TC39孜孜不倦的努力，终于联手打造了ECMAScript史上最大的一次升级：在2015年6月，ES6发布了。这个ECMAScript版本几乎集成了当时其他语言梦寐以求的所有明星特性，并优雅地、不留后患地解决了几乎所有的JavaScript遗留问题—当然，其中那些最大的、最本质的和核心的问题其实都已经在ES5推出时通过“严格模式（strict mode）”解决了。

- ES6提出了四大组件：Promise、类、模块、生成器/迭代器。这事实上是在并行语言、面向对象语言、结构化语言和函数式语言四个方向上的奠基工作。相对于这种重要性来说，其他类似于解构、展开、代理等看起来很炫很实用的特性，反倒是浮在表面的繁华了。

- 于是，之前被框架技术所引领的前端开始将注意力投放到本地渲染和工程化上。主流的技术方向从此开始进入工程化时代，带来了面向统一用户接触层的前端工程化、平台化和标准化等明显的跨代与跨界特征。试图将应用或产品逻辑前移的所谓“大前端”应运而生。至此，无论是在技术、工程还是组织等各个方面，前端都赢得了开创性的局面。

  随着工程与项目规模的扩大，JavaScript语言的一项早期设计越来越成为一切发展的阻碍—它是一门弱类型、动态类型的语言。[[27\]](opeb://6581862dde6be9bd7977192fd24b4fab/epubbook.xhtml#textpart0011_split_007.htmlACDE6B1E0E6D84D6490BBE17ECC2C944C)于是JavaScript不得不在推广过程中面临如何提供静态类型系统的问题，从而推动了TypeScript和Dart这类以ECMAScript转译器作为卖点的新语言的诞生。类似地，还有Facebook推出的Flow在静态类型检查方面的努力。而在这些语言和辅助工具的背后还有一股更为新生的力量，即打算对类型注解（Type annotations）做出规范的ECMAScript，它们被称为装饰器（Decorator）

- Brendan Eich在这份名为《JavaScript这十年》（*JavaScript at Ten Years*）的演讲稿中，重述了这门语言的早期发展历史：Brendan Eich自1995年4月受聘于网景公司，开始实现一种名为“魔卡（Mocha）”—JavaScript最早的开发代号或名称—的语言；仅两个月之后，为了迎合Netscape的Live战略而更名为LiveScript；到了1995年年末，又为了迎合市场对Java语言的热情，正式地、也是遗憾地更名为JavaScript，并随网景浏览器推出。[[29\]](opeb://6581862dde6be9bd7977192fd24b4fab/epubbook.xhtml#textpart0011_split_007.htmlAE1A1F47CE0EA4F23A83FDD7788D3711D)

  Brendan在这篇演讲稿最末一行写道：“不要让营销决定语言名称（Don't let Marketing name your language）”。

- 同其他语言一样，在JavaScript中，值类型与引用类型也用于表明运算时使用数据的方式：参与运算的是其值还是其引用。因此，在下面的示例中，当两次调用函数func()时，各自传入的数据采用了不同的方式：

  ![image-20220318143628158](../../../../Library/Application Support/typora-user-images/image-20220318143628158.png)

  从语义上来说，由于在示例1中实际只传入了str的值，因此对它的toString属性的修改是无意义的[[7\]](opeb://6581862dde6be9bd7977192fd24b4fab/epubbook.xhtml#textpart0012_split_007.htmlA7F4CE8D7FCE14B5D8F85B619E3F80DCB)；而在示例2中传入了obj的引用，因此对它的toString属性的修改将会影响到后来调用obj.toString()的效果。因此两个示例返回的结果不同。

- 当let声明发生在全局代码块时，它与var声明存在细微的差别。这是因为按照早期JavaScript的约定，在全局代码块使用var声明（和具名函数声明语法）时，相当于在全局对象global上声明了一个属性，进而使所有代码都能将这些声明作为全局变量来访问。而let声明与其他一些较新的语法元素遵从“块级作用域”规则，因此即使出现在全局代码块中，它们也只是声明为“全局作用域”中的标识符，而不作为对象global上的属性。例如

  ```js
  var x = 100;
  let y = 200;
  console.log(Object.getOwnPropertyDescriptor(global, 'x').value); // 100
  console.log(Object.getOwnPropertyDescriptor(global, 'y').value);  // undefined
  ```

  常量声明const、类声明class在块级作用域上的特性与let声明是类似的。

- 你总是可以用一对双引号或一对单引号来表示字符串字面量。早期Netscape的JavaScript中允许出现非Unicode的字符[[18\]](opeb://6581862dde6be9bd7977192fd24b4fab/epubbook.xhtml#textpart0012_split_007.htmlA9BCA2160E13D4A0B8AB31F12D0043E26)，但现在ECMAScript标准统一要求字符串必须是Unicode字符序列。

  转义符主要用于在字符串中包含控制字符，以及当前操作系统语言和字符集中不能直接输入的字符；也可以用于在字符串中嵌套引号（可以在单引号声明的字符串中直接使用双引号，或反过来在双引号中使用单引号）。转义符总是用一个反斜线字符“\”引导

  除了定义转义符之外，当反斜线字符“\”位于一行的末尾（其后立即是代码文本中的换行）时，也用于表示连续的字符串声明。这在声明大段文本块时很有用。例如：

  ```js
  let aTextBlock = "\
  abcdefghijklmnopgrstuvwxyz\
   \
  123456789\
   \
  +-*/";
  console.log(aTextBlock);
  ```

  该示例第3行与第5行中各包括了一个空格，因此输出时第2、4、6行将用一个空格分开。显示为：

  ```
  abcdefghijklmnopgrstuvwxyz 123456789 +-8/
  ```

  在各行中仍然可以使用其他转义符—只要它们出现在文本行最后的这个“\”字符之前即可。不过与某些语言的语法不同的是，不能在这种表示法的文本行末使用注释。

  另外一个需要特别说明的是“\0”，它表示NUL字符。在某些语言中，NUL用于说明一种“以＃0字符结束的字符串”（这也是Windows操作系统直接支持的一种字符串形式），在这种情况下，字符串是不能包括该NUL字符串的。但在JavaScript中，这是允许存在的。这时，NUL字符是一个真实存在于字符序列中的字符。下例说明了NUL字符在JavaScript中的真实性：

  ```js
  str1 = String.fromCharCode(0, 0, 0, 0, 0);
  str2 = "\0\0\0\0\0";
  console.log(str1.length);
  console.log(str2.length);
  ```

  显示字符串长等于 5，说明 NUL 字符在字符串中是真实存在的

  在ES6中增加了“\u{nnnnn}”来表示码点大于0xFFFF的Unicode字符的语法，这意味着它支持直接声明和使用UTF-16字符。所以在这种情况下，在当前字符集使用UTF-8时，一个用“\u{nnnnn}”表达的Unicode字符的长度将是2。例如：

  ```js
  console.log("\u{20BB7}".length);
  ```

  在JavaScript中也可以用一对不包含任意字符的单引号或双引号来表示一个空字符串（Null String），其长度值总是为0。比较容易被忽视的是，空字符串与其他字符串一样也可以用作对象成员名。例如：

  ```js
  obj = {
  	'': 100
  }
  console.log(obj['']); // 100
  ```

- 可以通过下标来获取字符串的某个索引的值，但是不能通过下标来修改字符串索引的值

  ```js
  let str = '123';
  str[1] = 'e';
  // console.log(str) // 123
  ```

  看完了 2.3 节