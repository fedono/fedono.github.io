---
layout: post 
title: "React Hooks 学习" 
author: "fedono"
---

Why React Hooks

> With Hooks, you can extract stateful logic from a component so it can be tested independently and reused. Hooks allow you to reuse stateful logic without changing your component hierarchy. This makes it easy to share Hooks among many components or with the community.
> They let you use state and other React features without writing a class.

官方文档上专门用了一个版块来介绍[Hook](https://zh-hans.reactjs.org/docs/hooks-intro.html), 这里摘抄了几个比较关心的问题(其他`FAQ`请移步官网):

1. [引入`Hook`的动机?](https://zh-hans.reactjs.org/docs/hooks-intro.html#motivation)

   - 在组件之间复用状态逻辑很难; 复杂组件变得难以理解; 难以理解的 class. 为了解决这些实际开发痛点, 引入了`Hook`.

2. [`Hook` 是什么? 什么时候会用 `Hook`?](https://zh-hans.reactjs.org/docs/hooks-state.html#whats-a-hook)

   - `Hook` 是一个特殊的函数, 它可以让你“钩入” `React` 的特性. 如, `useState` 是允许你在 `React` 函数组件中添加 `state` 的 `Hook`.
   - 如果你在编写函数组件并意识到需要向其添加一些 `state`, 以前的做法是必须将其转化为 `class`. 现在你可以在现有的函数组件中使用 `Hook`.

3. [`Hook` 会因为在渲染时创建函数而变慢吗?](https://zh-hans.reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)

   - 不会. 在现代浏览器中,闭包和类的原始性能只有在极端场景下才会有明显的差别. 除此之外,可以认为Hook

     的设计在某些方面更加高效:

     - `Hook` 避免了 `class` 需要的额外开支,像是创建类实例和在构造函数中绑定事件处理器的成本.
     - 符合语言习惯的代码在使用 `Hook` 时不需要很深的组件树嵌套. 这个现象在使用`高阶组件`、`render props`、和 `context` 的代码库中非常普遍. 组件树小了, `React` 的工作量也随之减少.

所以`Hook`是`React`团队在大量实践后的产物, 更优雅的代替`class`, 且性能更高. 故从开发使用者的角度来讲, 应该拥抱`Hook`所带来的便利.



Hooks 优势

> 1. 函数组件不能使用state，遇到交互更改状态等复杂逻辑时不能更好地支持，hooks让函数组件更靠近class组件，拥抱函数式编程。
> 2. 解决副作⽤问题，hooks出现可以处理数据获取、订阅、定时执行任务、手动修改 ReactDOM这些⾏为副作用，进行副作用逻辑。比如useEffect。
> 3. 更好写出有状态的逻辑重用组件。
> 4. 让复杂逻辑简单化，比如状态管理：useReducer、useContext。
> 5. 函数式组件比class组件简洁，开发的体验更好，效率更⾼，性能更好。
> 6. 更容易发现无用的状态和函数。
>    

使用`useContext`

```js
import React, {useState, useContext} from 'react';

const themes = {
    light: {
        foreground: '#000',
        background: '#fff'
    },
    dark: {
        foreground: '#fff',
        background: '#222'
    }
};

// 还是使用 createContext 来定义 context，但是使用 useContext 来使用
const ThemeContext = React.createContext({
    theme: themes.light,
    toggle: () => {}
})

export default () => {
    const [theme, setTheme] = useState(themes.light);
    return <ThemeContext.Provider value={{
        theme,
        // 为什么这里有两个 setTheme，第二个是 useState 里的，那第一个呢？
        toggle: () => {
            setTheme(theme => {
                setTheme(theme === themes.light ? themes.dark : themes.light)
            });
        }
    }}>
        <Toolbar />
    </ThemeContext.Provider>
}

const Toolbar = () => {
    return <ThemedButton />
}

const ThemedButton = () => {
    const context = useContext(ThemeContext);
    return (
        <button
            style={{
                color: context.theme.foreground,
                backgroundColor: context.theme.background
            }}
            onClick={() => {
                context.toggle();
            }}
        >
            Click Me!
        </button>
    )
}


```

使用 `reduce`

```js
import React, {useState, useRef, useReducer} from 'react';


function reduce(state, action) {
    switch (action.type) {
        case 'add': return state + 1;
        case 'decrease': return state - 1;
        default: return 1;
    }
}

export default () => {
    const [state, dispatch] = useReducer(reduce, 0);
    return (
        <div>
            {state}
            <button onClick={() => {
                dispatch({type: 'add'})
            }}>+</button>
            <button onClick={() => {
                dispatch({type: 'decrease'})
            }}>-</button>
        </div>
    )
}


```

使用 `useRef`

可以使用 ref 来缓存一个值

```js
import React, {useState, useRef, useReducer} from 'react';
export default () => {
    const [counter, setCounter] = useState(0);
    const prev = useRef(null); 
    return (
        <div>
            <p>当前值：{counter}</p>
            <p>之前的值：{prev.current}</p>
            <button onClick={() => {
                prev.current = counter;
                setCounter(x => x + 1)
            }}>+</button>
            <button onClick={() => {
                prev.current = counter;
                setCounter(x => x - 1);
            }}>-</button>
        </div>
    )
}
```

通过也可以使用 ref 来存一个元素，如 

```js
const refInput = useRef(null);
<input ref={refInput} />
  
 // 然后就可以使用 refInput.focus() 来让这个元素聚焦 
```

使用 `useEffect`

```js
export default () => {
    const [count, setCount] = useState(0);

    function myEffect() {
        const I = setInterval(() => {
            console.log(count);
          	// 这里面需要使用 count => count + 1 而不是 count + 1,
          	// 上面的 console.log 始终打印的都是 0，所以这里面的参数 count 会同步传进去，
            // 类似的问题如 react 的 setState({count: count + 1}) 与 setState(state => {return state + }) 的问题一样，前者如果有多个，则只会执行前一个，后者如果有多个，则多个会累加，也就是每次 state => state + 1这种的参数 state 会同步当前环境中的 state 
            setCount(count => count + 1);
        }, 1000);
        return () => clearInterval(I);
    }

    useEffect(myEffect, []);
    return <div>{count}</div>
}
```

- 如何来封装自己的 Hooks

  ```js
  import React, {useState, useRef, useReducer, useEffect} from 'react';
  
  function sleep(time) {
      return new Promise(resolve => {
          setTimeout(() => resolve(), time)
      });
  }
  
  async function getPerson() {
      await sleep(200);
      return ['a', 'b', 'c']
  }
  
  function usePerson() {
      const [list, setList] = useState(null);
  
      async function request() {
          const list = await getPerson();
          setList(list);
      }
  
      useEffect(request, []);
      return list;
  }
  
  export default () => {
      const list = usePerson();
      if (list === null) {
          return <div>Loading...</div>
      }
      return (
          <div>
              {list.map(name => {
                  return <li>{name}</li>
              })}
          </div>
      )
  }
  
  ```



## 参考

- [React Hooks 原理](https://github.com/brickspert/blog/issues/26)