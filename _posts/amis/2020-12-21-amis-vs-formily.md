---
layout: post 
title: "AMis 与 formily 的比较" 
author: "fedono"
---

- todo 
  - 各自的 formItem 都做了什么
  - amis 中增加的hook，以及使用了 store 来管理数据
  - 针对递归渲染，他们的区别是什么 



- 整个 formily 中，数据是如何处理的，没有 store，那么在 submit 的时候，是如何获取到数据然后提交的呢？
- 在数据更新的时候，使用到了 setFieldState 来更新单个数据，这个又是如何才能通过路径对应到 formItem 中的呢？
- formily 是如何与 rxjs 进行绑定的 
  - 在 react-action 中处理了 effects 和 actions 
- formily 中的 [性能优化实践](https://formilyjs.org/#/xbS7SW/yAtktNs3hx) 中提到了 formily 可以只更新单个 formItem，这个是如何来做到的
  - 每个formItem 都会创建一个 model，然后里面会用 rxjs 来更新对应 formItem 的值，但是在 amis 中不是也是每个 formItem 都会创建一个 Model吗，为什么使用 rxjs 就能够更新对应的 formItem 的值而不让整个form渲染，但是在 amis 中就会整个form 都渲染呢？
  - 是 rxjs 有做什么？还是两个系统的数据管理的方式会有这种问题的
  - amis 解决数据量大的措施是使用可视区域内的才显示 formItem这样就会减少渲染多次

- 







---





- amis 和 formily 的相同与区别

  - formily 主要的还是在处理 form 的数据提交，表单联动

    - formily 集成了 next/antd/meet 两个组件库，文档还是倾向于开发者能够快速的实现表单功能

    - **数据联动**：有实现方式的差异，比如表单的联动，amis 都是完全通过在json 配置中来对其他的表单的显隐来做操作，formily 需要使用 observer/subscribe 来操作对应组件数据的

      - amis 可以支持显隐联动，这点可以通过 visibleOn 来支持
    - 手动更新，如指定组件的名称来实现 reload
      
  - 自动监听，依赖变化，如在 componentWillReceiveProps 中监听 url 中的参数来达到自动触发更新
      
  - formily 划分为自己定义的核心功能，以及对于相关组件库的改造的，amis 划分为渲染器和可视化编辑器以及平台功能（formily 也有 editor 啊
  
  - formily 支持使用jsx 和 json-schema 的形式来编写组件，amis 则是使用 json-schema 的形式以及拖拽的形式
  
    - **递归渲染**  
  
    - **表单动态化**：formily 中也可以通过 动态化的 json-schema 来支持渲染，amis 中也可以通过从服务端获取到 json 后，然后渲染json，同时还能够将表单的数据返回，渲染完成 json-schema 后，会将当前的数据也填充到表单中
  
      > ### **后端数据驱动问题**
    >
      > 有一些场景，我们的表单页面是非常动态化的，某一些字段是否存在，前端完全感知不到，是由后端建表，由不同职业属性的用户手工录入的字段信息，比如电商购物车的表单页面，交易下单页面，系统需要千人千面的能力，这样就需要前端拥有动态化渲染表单的能力了，不管是Antd Form还是Fusion Next Form都没有原生就支持这样的动态化渲染的能力，你只能在上层在封装一层动态化渲染机制，这样就得基于某个JSON 协议来驱动渲染，
      >
      > --- 来源 [面向复杂场景的高性能表单解决方案(背景篇)](https://zhuanlan.zhihu.com/p/62927004) 
  
    - **路径**：formily 和 amis 中都有路径的概念，但是两个库的路径是完全不同的
      
      - formily 是基于所有的 name 值，能够找到对应组件，核心原理是 cool-path 库，与rxJS 一起用来处理数据联动
      - amis 中的路径是组件的名称在当前页面中的路径，是用来通过在 json-schema 中找到当前的组件的，如`page/body/form/controls/0/text` 用来匹配到form 下的 `text` 组件
      
    - 代码的组织方式
    - amis 通过将组件注册进全局（注册的同时将当前组件会被匹配到的正则路径也同时进行注册），然后在渲染的时候，将匹配组件的方法通过 props 可以顶层传给每个组件，这样在每个组件中，就可以直接调用该方法，将在当前组件的子组件的 json-schema 传给该函数进行渲染
      - formily 在 jsx 中是通过组件的调用（在使用 json-schema 的时候，是怎么获取到组件的 
  
  - formily 有声明周期的概念，可以在表单的不同时期做不同的处理 
  
  - formily 中，`json-schema ` 只是使用 `formily` 的一种方式，是提供给后端人工渲染动态表单的方式，前端人员还是使用 `jsx` 的形式来写。
  
  - amis 支持跨表单的数据传递，formily 支持吗？
  
    - [react-eva](https://link.zhihu.com/?target=https%3A//github.com/janrywang/react-eva) 解决方案，借助它，我们可以非常轻松的处理外部联动问题，同时还不会触发全局整体渲染。
  
  - formily 中使用的 schema 是不具备逻辑操作的，只是单纯的渲染功能，所有的需要执行的数据提交，表单之前的数据关联等操作都是无法做到的。但是 amis 是可以的，也就是在amis 中是全部可以使用 json 来处理这些操作，所有的逻辑都可以通过配置 json 来完成的。
  
  - formily 是没有嵌套组件的，也就是所有的组件都是平级的，也就是 formily 是不支持 tree 类型这种组件来实现自定义添加子层级的 
  
  - 
  
  - **公共配置**：
  
    - amis  中可以通过建立公共配置，那么在能够复用相同的页面显示的时候，就可以更节省开发量
  
  - 适配其他的组件库
  
    - formily 封装了 antd/next 的组件库，只要能够输出 `value, onchange` 事件就能够封装
  
    - amis 自定义自己的组件库，用来实现非常多的各种逻辑
  
        - 而且还可以实现在其他平台如 vue
  
          1. 实例化vue，将 props 中数据放到 vue 的 data 中，然后将props 中的function 通过实例化的的`vue` 通过 `$on` 进行绑定，创建一个 div 元素，将实例化的 `vue` 也就是`this.vm.$mount().$el`放到创建的 div 元素内。然后将当前的配置注册到 amis 开放的接口 `FormItem` 中，`FormItem` 是用来注册 `formItem` 组件的。
  
          2. 使用的过程中可以引入vue 的组件库，通过`Vue.use(ElementUI)` 来实现
  
  - amis 侧重于整个页面的json实现，为了实现能够用户能做快速的实现页面，做了非常多的处理
  
    > amis 更多的方向是倾注于低代码的，也就是用户可以不用懂如何写代码，就可以创造页面 
  >
    > amis 搭建了平台是可以更加的节省业务线的成本，集成了 用户认证，页面管理，可视化编辑器，菜单定制，权限地址，用户/角色管理，文件图片上传云端，API Mock，操作日志，页面统计，自定义组件和 Serverless 等更多公共服务。
  
    - amis 其实更倾向于是非开发者直接使用平台来生成页面，做了可视化的工具来使用用户拖拽组件，配置组件配置，即可生成页面的功能
    - 比如数据的转换，根据不同的平台的接口返回格式的不一样，在接口的发送与接口数据格式上做适配器，如http发送数据的格式是content-Type/json 还是www-form-urlencode
    - 以及在用户操作 amis 的时候，内部实现了多种数据的转换以及获取，比如转换成
    - 快速生成页面，直连用户的数据库，获取数据表的字段，根据用户选择生成页面的类型，直接生成页面和后端的接口
    - 针对链接是否跨域做了代理操作，将用户的URL替换成平台的URL，然后在后端机器上还原回真正的URL进行请求
    - 实现在平台上编写Node代码用来实现接口的功能，用户只需要提供数据库登录的相关字段
    - 能够实现用户自定义页面的样式
    - 调试页面的Mock API 
    - 用户，角色，权限管理
    - 多个操作环境的变量定义
    - 消息提醒 

---

附：

1. 在 [面向复杂场景的高性能表单解决方案(背景篇)](https://zhuanlan.zhihu.com/p/62927004) 中写到了 formily 为什么做的原因，其中有一条『某小A』的评论中有提到

> 1. 选用 json schema 往往是要做表单动态化，json schema 只适用于给机器阅读，表单方案，动态性应该只是其核心能力，并不应该是其核心架构设计主线。一堆前端开发人员居然要写 json schema 来做表单，简直反人类
>   
>    - 这里说了为啥要使用 json-schema ，但是不应该把 json-schema 作为主力
> 2. 在万恶之源的副作用管理核心设计上，redux saga 将 saga pattern 带到了前端领域已是造福之举，现在社区有 callbag 规范可以让我们发挥所能，这方面看到 uform 也果断直接引用了 rxjs，是一个简单直接暴力，但却最有效的做法
>    - saga pattern 是啥意思？
>    - callbag 规范 又是啥
> 3. 高性能的核心设计值得称赞，从单向数据流变成发布订阅模式，这样就可以在广播前做很多务实的事情，比如按需更新。
>
>    - amis 在这方面有什么优势，amis 有按需更新吗？这篇文章的最后两个图有介绍formily 的按需更新的动图，可以参照来看下 amis 
>
>      (打开方式，安装好 react-develop-tools后，打开控制台，会新增 components, Profiler 两个tab 栏，打开 components，有一个设置，勾选 Highlight updates when components render，然后在当前amis 页面建立一个表单，发现修改一个 formItem 的数据时，整个表单都会进行更新，当然不是表单以及与当前表单同级的其他表单就不会更新了，说明 amis 没有按需更新)
>
>      ![image-20201221152253083](/Users/zengdeping/www/fedono.github.io/assets/imgs/amis-vs-formily/components-render.png)

2. formily 是强参考了  **[react-final-form](https://github.com/final-form/react-final-form)** ，**[redux-form](https://github.com/redux-form/redux-form)** 来做的代码的实现，并且自己加上了`json-schema ` 来实现表单动态化

   react-final-form 的缺点在附录1 中的文章也有总结 

   > 优缺点
   >
   > **优点：** 
   >
   > 1. 该方案的思路非常明确，就是每个字段自己管理状态，自己做渲染更新，分布式状态管理，完全与redux-form的单向数据流理念背道而驰，但是，收益一下子就上来了，表单的整体渲染次数大大降低，React的CPU渲染压力也大大降低，所以，final-form就像它的名字一样，终结了表单的性能问题。
   >
   > 2. 同时，对于代码可维护性而言，final-form也有自己的亮点，就是它将表单字段的联动逻辑做了抽离，在一个独立的`calculate` 里处理，这样就不会使得业务组件变得非常臃肿。而且，作者对final-form的可扩展设计也是非常清晰的，还有，final-form是一个开箱即用的解决方案，它就是一个壳，通过render props的形式可以组合使用各种组件库，总之，final-form解决方案解决了表单领域的大部分问题。
   >
   > **缺点**：
   >
   > 1. 联动不能一处编写，单纯calculator不能处理状态的联动，比如字段A的值变化会控制字段B的disabled状态，必须结合jsx层的Field subscription才能做状态联动，用户需要不停的切换写法，开发体验较差，比如：[https://codesandbox.io/s/jj94wojl95](https://link.zhihu.com/?target=https%3A//codesandbox.io/s/jj94wojl95)
   > 1. 嵌套数据结构需要手动拼接字段路径，比如 [https://codesandbox.io/s/8z5jm6x80](https://link.zhihu.com/?target=https%3A//codesandbox.io/s/8z5jm6x80)
   > 2. 组件内外通讯机制过于Hack，比如在外部调用Submit函数 [https://codesandbox.io/s/1y7noyrlmq](https://link.zhihu.com/?target=https%3A//codesandbox.io/s/1y7noyrlmq)
   > 3. 组件外部不能精确控制表单内部的某个字段的状态更新，除非使用全局rerender的单向数据流机制。
   > 4. 不支持动态化表单渲染，还是需要在上层建立一个动态渲染引擎


3. 开源届也有很多的基于 json-schema来做的方案，如[react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form)，[surveyjs](https://surveyjs.io/)

   - react-jsonschema-form  
     - [react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form) 这个库几乎是完整的按照 `json-schema` 的协议来写的，非常多的基于`json-schema` 来做的页面搭建几乎都没有完整的实现`json-schame` 的语法，只有这个实现了。
     - 这个库有验证功能，可自定义插件和主题，但是高级一点的功能没有，比如表单的联动
     - rjsf 的设计其实也非常好，内部每一个组件都是 `Widget` 的形式来注册，这样在拓展外部组件的时候，也是同样的方式
     - 核心组成其实就是两个，一个是 `widget` 还有一个就是`field` 
       - 现在还没搞懂 `field` 到底是要干什么，内部也是通过调用`Widget` 的形式，然后把所有的参数传给`widget` 
     - 基础的组件比较少，也没有数据管理，是一个

