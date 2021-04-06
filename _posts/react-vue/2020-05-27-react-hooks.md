



- useState 其实很好理解
	
	```js
	// 之后再 setCount(x)中，x 就是要设置 count 的值，这个 x 可以随便怎么变
	const [count, setCount] = useState(0);
	
	```
	
- useRef 是什么东西，下面这个示例，我以为是加一个 `dom` 到 `useRef` 的 current 中，但居然是可以随便加？不只是 `dom` 也可以 

  ```js
  function Counter() {
    const [count, setCount] = useState(0);
    const prevCount = usePrevious(count);
    return <h1>Now: {count}, before: {prevCount}</h1>;
  }
  
  function usePrevious(value) {
    const ref = useRef();
    useEffect(() => {
      ref.current = value;
    });
    return ref.current;
  }
  ```

  