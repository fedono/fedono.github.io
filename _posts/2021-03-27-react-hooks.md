---
layout: post 
title: "React Hooks 学习" 
author: "fedono"
---

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
        toggle: () => {
            // 为什么这里有两个 setTheme，第二个是 useState 里的，那第一个呢？
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