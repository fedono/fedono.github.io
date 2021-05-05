- 补充基础
  - https://www.cnblogs.com/kubidemanong/category/1263428.html
- amis/编辑器原理，平台功能
  - 关于编辑器的
    - 把[h5-Dooring](https://github.com/MrXujiang/h5-Dooring) 弄懂，研究一下可视化的编辑器的竞品通常都有哪些方案
    - [可视化拖拽组件库一些技术要点原理分析](https://zhuanlan.zhihu.com/p/350801650) 总结一下这类可视化编辑器的要点
- 类似 amis 这种使用 json-schema 来做的如formily 这些的优缺点比较
- react 数据管理
  - 为什么需要数据管理
  - 有哪些数据管理的库
  - redux/mobx/mst 的原理
- react 路由 
- react
  - 更新逻辑（diff / fiber
  - 为什么需要 hooks（hook 的用法
  - setState 是同步还是异步（什么时候是同步，什么时候是异步

---

- HTTP 
  - 参考 [http all](./2020-06-03-http-all.md) 
  - [http](./2020-03-28-HTTP.md)  

---

- amis 
  - 注册组件以及在 json 渲染组件的机制
  - scope 通信机制
  - 组件间的通信
  - 接口优化，适配器
    - 建立缓存，兼容所有的其他业务的接口适配
  - 多主题
    - 在 context 中建立了主题的样式，那么在使用的时候，好像没有看到使用 context 啊，是怎么做的 
  - 样式是如何设计的，方案是什么
    - 
- 爱速搭
  - 多页面渲染 
- 编辑器 
- 平台
  - 如何搭建 Node 平台，实现用户编写Node代码，测试运行，版本管理，
  - 快速新建页面
    - 代码自动生成
    - 模块代码
    - 获取数据表结构完成当前页面的自定义

react 相关问题

参考 react-inter-v

---

各大 json-schema 的渲染引擎是如何做的，有什么优缺点

编辑器又是如何做的，有什么优缺点

formily 既然是引用的组件，那他的编辑器是如何做的，如何来做到可视化的

---

- 自定义主题
  - 使用 saas-doc 解析到变量然后设置，和之前的SaaS文件一同编译，最后保存到数据库中
- JSON schema 如何做组件的渲染与显示
  - 参考 san-xui，里面通过 asForm/createForm 这些方法
- amis 如何做到兼容 vue 和 element-ui 这些组件嵌入进来

