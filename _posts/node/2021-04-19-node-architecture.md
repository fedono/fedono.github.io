关于Node 的架构

- 框架
  - Koa/Express/Egg
- 自启动以及启动日志
  - Pm2 / nodemon
    - 可以对 pm2 进行二次封装
- 日志记录
  - logger4js
    - 需要对哪些来做日志
      - 请求前的参数记录
      - 请求后的响应参数记录
      - 项目全局的错误
- 请求
  - node-fetch
  - 
- 会做哪些中间件
  - 权限
  - 日志