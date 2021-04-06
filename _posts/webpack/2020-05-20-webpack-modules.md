---
layout: post 
title: "如何编写 Webpack 的 modules" 
author: "fedono"
---

`Webpack` 的 `modules` 是最常见的配置，但是细分下来，还是有很多需要注意的点，日常需要打包的配置有 `js/jsx/css/icon/img/font/json` 等等

- JS 的打包，加上 `babel-loader`，并且配置`presets`

  ```js
  {
    test: '/\.js$/',
    loader: ['react-hot-loader/webpack', 'babel-loader?' + JSON.stringify({presets: ['react', 'es2015', 'stage-0']})],
    excludes: /node_modules/
  }
  ```

- JSX 的打包

  ```js
  {
    test: /\.jsx?$/,
    loader: 'babel-loader',
    exclude: /node_modules/,
    query: {
      presets: ['env', 'stage-0', 'react'],
      plugins: ['transform-decorators-legacy', 'add-module-exports']  
    }  
  }
  ```

- 



## 参考

- [Module配置]([https://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-3Module.html](https://webpack.wuhaolin.cn/2配置/2-3Module.html))
- [文档](https://webpack.docschina.org/concepts/modules/)