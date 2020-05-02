## context 

react  的 context 分为 `provider`  和 `consumer` ，`provider` 来传递，也就是使用 `provider` 来包裹组件，`consumer` 来使用数据，具体的 `context` 用法如下，官网文档说的很清楚了，这里只是自己不记得，做个记录。

1. 使用`createContext` 来创建 context

   ```js
   const MyContext = React.createContext(defaultValue);
   ```

2. 使用 `provider` 来传递值

   ```js
   <MyContext.Provider value={/* 某个值 */}>
   ```

3. 使用`context` 其实有两种方式，这个在[官方文档](https://reactjs.org/docs/context.html) 中也有说明

   1. 通过设定组件的 `contextType` 为需要的`context` ，然后使用 `this.context` 来使用

      ```js
      class MyClass extends React.Component {
        componentDidMount() {
          let value = this.context;
          /* perform a side-effect at mount using the value of MyContext */
        }
        componentDidUpdate() {
          let value = this.context;
          /* ... */
        }
        componentWillUnmount() {
          let value = this.context;
          /* ... */
        }
        render() {
          let value = this.context;
          /* render something based on the value of MyContext */
        }
      }
      MyClass.contextType = MyContext;
      ```

   2. 就是使用 `consumer` 来使用

      ```js
      <MyContext.Consumer>
        {value => /* render something based on the context value */}
      </MyContext.Consumer>
      ```

      

## forwordRef

forwordRef 的作用就是让调用子组件的父级组件，可以通过 ref 来调用子组件内部的元素，如子组件中有`input` ，那就父级组件就可以使用 `this.inputRef.focus()` 来使得子组件执行 `focus` 事件

需要注意的是，子组件是以 `function` 这种形式来传递 `ref` 的，示例如下

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

也就是`React.forwordRef` 内部就只能使用 `function` 的形式，然后再第二个参数中使用 `ref` ，这样才能把上一级的`ref` 传递下来，那如果要使用  `class` 呢？那就需要在 `function` 内部来调用 `class` 的组件，然后指定`ref` 下传就可以了

```js
const FancyButton = React.forwardRef((props, ref) => (
  <RefButton {...props} forwardedRef={ref} />
));

// 在 class 中把 ref 放到组件中
class RefButton extends React.Component {
  render() {
    const {forwardedRef, ...rest} = this.props;
    
    return (
    	<button
      	{...rest}
      	ref={forwardedRef}
			></button>
    )
  }
}
```

