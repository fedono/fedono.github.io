检测资源加载的错误你知道有哪些吗？

- JS运行时候的错误
  - 使用 window.onerror 
  - 使用 try catch
- 资源加载的错误
  - 每个 DOM 结构都有 onerror 事件，但是肯定是监听的是全局的事件
  - 使用 performance 的 entries，这里面会获取到所有的资源加载，用来检测资源是否加载出错

