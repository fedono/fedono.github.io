

日常使用的 react-router ，不包括以下的 `react-router-native` ，我们逐个分析

| Package                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`react-router`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router) | The core of React Router                                     |
| [`react-router-dom`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom) | DOM bindings for React Router                                |
| [`react-router-native`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-native) | [React Native](https://facebook.github.io/react-native/) bindings for React Router |
| [`react-router-config`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-config) 5.1.1 | Static route config helpers                                  |

使用较多的都是 `react-router-dom` 和 `react-router-config` 

先从简单的来 `react-router-config` 只有两个函数

- `matchRoutes(routes, pathname)` 用来在所有 `routes`  匹配到传进来的 `pathname` 的 `route` 
- `renderRoutes(routes, extraProps = {}, switchProps = {})`  用来对`routers` 进行一次分配，没明白啥意思是不，继续往下看

`react-router-config` 就是一个 `react-router` 的应用库而已，直接上源码好了，代码就几行

```js
// matchRoutes 在所有的 routes 找到匹配到传入的 pathname 的route的数组，然后返回
function matchRoutes(routes, pathname, /*not public API*/ branch = []) {
  routes.some(route => {
    const match = route.path
      ? matchPath(pathname, route)
      : branch.length
      ? branch[branch.length - 1].match // use parent match
      : Router.computeRootMatch(pathname); // use default "root" match

    if (match) {
      branch.push({ route, match });

      if (route.routes) {
        matchRoutes(route.routes, pathname, branch);
      }
    }

    return match;
  });

  return branch;
}

// renderRoutes 就是对所有的 routes 进行一次 switch 操作，从而简化用户的代码编写就是
function renderRoutes(routes, extraProps = {}, switchProps = {}) {
  return routes ? (
    <Switch {...switchProps}>
      {routes.map((route, i) => (
        <Route
          key={route.key || i}
          path={route.path}
          exact={route.exact}
          strict={route.strict}
          render={props =>
            route.render ? (
              route.render({ ...props, ...extraProps, route: route })
            ) : (
              <route.component {...props} {...extraProps} route={route} />
            )
          }
        />
      ))}
    </Switch>
  ) : null;
}
```

demo 查看[官方文档](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-config)

继续研究 `react-router-dom` ，有 `dom` 也就是与浏览器有关，所以这里需要使用到 [history](https://www.npmjs.com/package/history) 这个库，这个库先不管，我们看 `react-router-dom` 有啥功能

- BrowserRouter  的代码直接放上来

  ```js
  /**
   * The public API for a <Router> that uses HTML5 history.
   */
  class BrowserRouter extends React.Component {
    history = createBrowserHistory(this.props);
  
    render() {
      return <Router history={this.history} children={this.props.children} />;
    }
  }
  ```

- HashRouter 的代码也直接放上来

  ```js
  /**
   * The public API for a <Router> that uses window.location.hash.
   */
  class HashRouter extends React.Component {
    history = createHashHistory(this.props);
  
    render() {
      return <Router history={this.history} children={this.props.children} />;
    }
  }
  ```

- Link 就是一个 a 标签，为了能够灵活一点，就可以让用户添加 `object` 的形式，然后用 `history` 库的方式再组合成一个 `http` 的正常的 `url` 形式

  ```js
  // 两个处理用户传入的 location 的方法， createLocation 是 history 库的方法
  export const resolveToLocation = (to, currentLocation) =>
    typeof to === "function" ? to(currentLocation) : to;
  
  export const normalizeToLocation = (to, currentLocation) => {
    return typeof to === "string"
      ? createLocation(to, null, null, currentLocation)
      : to;
  };
  
  // 最终的返回形式
  return <a {...props} />;
  ```

- NavLink 顾名思义就是导航的链接，加了个 `isActive` ，当前页面的链接等同于路由的链接的时候，如果 `isActive` 有属性的话，就会高亮当前调用的 `Link` 

  ```JS
  const props = {
    "aria-current": (isActive && ariaCurrent) || null,
    className,
    style,
    to: toLocation,
    ...rest
  };
  
  return <Link {...props} />;
  ```


回到重点的 `react-router`

  - StaticRouter 不能改变当前的 location，只是用来记录 location 的变化，通常用于 测试或者是服务端渲染

  - Switch 用来渲染匹配到的 `route`

    ```js
    React.Children.forEach(this.props.children, child => {
      if (match == null && React.isValidElement(child)) {
        element = child;
    
        const path = child.props.path || child.props.from;
    
        match = path
          ? matchPath(location.pathname, { ...child.props, path })
        : context.match;
      }
    });
    
    return match
      ? React.cloneElement(element, { location, computedMatch: match })
    : null;
    ```

- Redirect 跳转，本以为就是判断一下当前的 location和目的的 location，然后改变 location 就好，没想居然还看到一个亮点，Lifecycle 组件，这个真的可以写个单独的技术文档，而且肯定可以扩充看看，使用到其他的场景里面

  ```js
  <Lifecycle
    onMount={() => {
      method(location);
    }}
    onUpdate={(self, prevProps) => {
      const prevLocation = createLocation(prevProps.to);
      if (
        !locationsAreEqual(prevLocation, {
          ...location,
          key: prevLocation.key
        })
      ) {
        method(location);
      }
    }}
    to={to}
  />
      
  class Lifecycle extends React.Component {
    componentDidMount() {
      if (this.props.onMount) this.props.onMount.call(this, this);
    }
  
    componentDidUpdate(prevProps) {
      if (this.props.onUpdate) this.props.onUpdate.call(this, this, prevProps);
    }
  
    componentWillUnmount() {
      if (this.props.onUnmount) this.props.onUnmount.call(this, this);
    }
  
    render() {
      return null;
    }
  }    
  ```

- Prompt 是调用的浏览器的 prompt 方法，这个是在 history 库中使用的 block 方法

  - 但是没有明白这里的 `release` 方法是什么，看来需要调试下看看是什么了



再往下需要研究的就是两点了，一个是在使用的项目中，是如何最大化的使用 `react-router`这些库的，然后再结合文档仔细看下用法是什么，自己写写demo，调试下输出的结果

第二个就是需要研究 history 这个库了，感觉这个库封装的挺多的方法都可以在以后的 工作中可以用得到，比如建立一个location的基本的元素是什么，如何建立，等等所有的边界情况考虑，这些搞清楚了，浏览器的 location 和 history 就真的搞清楚了

另外就是 `createBrowserHistory` 和 `createHashHistory` 还可以在 浏览器中的 `history` 中有对应的概念，但是 `createMemoryHistory` 是什么意思？不在浏览器中建立，而是就是使用变量来存储路由吗，也就是所有的内存中存储路径

- 拓展点就是，如何来写一个路由，现在看这几个库，都没有 使用到 监听 `hashchange` 这类相关的代码都没有，那么在哪会有呢，现在只能寄希望于 `history` 这个库能找得到了。
- 结合 `history` 里面的 location ，和 amis  里的 normal location 看下，