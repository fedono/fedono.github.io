- 组件间的通信 

  只有设定了`isolateScope: true` 的组件才会注册到`Scoped` 中去，也就是可以使用到 `reload/receive` 通信 的组件，但是全局查找，只有`drawer/dialog/page/form` 才有这几个属性

  在`Scoped ` 中会通过`Provider` 将`scoped` 传给子组件，也就是这些子组件可以通过`consumer` 使用到，但是什么时候使用到啊，没见着啊

	在 `Control` 中有用到，也就是所有的`formItem` 组件其实都有通过这个来注册进去。每个注册进去的都是实例，也就是 `ref` 

  ```js
  // src/renderers/Form/Control.tsx:306
  controlRef(control: any) {
    const {addHook, removeHook, formStore: form} = this.props;
  
    // 因为 control 有可能被 n 层 hoc 包裹。
    while (control && control.getWrappedInstance) {
      control = control.getWrappedInstance();
    }
  
    this.control = control;
  }
  
  scoped.registerComponent(this.control);
  ```
  
  
  
- 验证

  `Control` 组件中的 ` controlRef` 函数中当获取到`formItem` 的时候，通过将组件的 `validate` 函数提取出来，然后`addHook` ，这样就将当前渲染的所有组件的验证函数收集到了，那么在点击提交的时候，就可以从这里的`hooks ` 中的所有验证函数执行一遍。

  ```js
  // src/renderers/Form/Control.tsx:314
  if (control && control.validate && this.model) {
    const formItem = this.model as IFormItemStore;
    let validate = promisify(control.validate.bind(control));
    this.hook = function () {
      formItem.clearError('component:valdiate');
  
      return validate(form.data, formItem.value, formItem.name).then(ret => {
        if ((typeof ret === 'string' || Array.isArray(ret)) && ret) {
          formItem.setError(ret, 'component:valdiate');
        }
      });
    };
    addHook(this.hook);
  }
  ```

  `Control` 本身中含有`validate` 函数，在拿到 `this.hook` 的时候，将这个函数放到当前的`formItem` 中的 `validate` 函数中进行验证 

  ```js
  //  src/renderers/Form/Control.tsx:348
  this.model.validate(this.hook);
  ```

- 数据变更

  在 formItem 中没有数据变化的函数操作，也就是 onChange 操作，操作在 `Control` 中，但是在 `Control` 中执行的只是 `this.state.value = value` ，也就是并没有更新整个 form 中的数据啊，为什么整个表单会进行更新，到底是进行了什么操作

  答案应该是

  ```js
  // 在 formItem 中有两个地方会触发更新整个 form 数据的改变
  // 一个是 emitChange 还一个是  setPrinstineValue， formItem onChange 的时候会触发emitChange
  
  // src/store/formItem.ts:294 
  function changeValue(value: any, isPrintine: boolean = false) {
    if (typeof value === 'undefined' || value === '__undefined') {
      self.form.deleteValueByName(self.name);
    } else {
      self.form.setValueByName(self.name, value, isPrintine);
    }
  }
  
  // src/store/form.ts:90
  // 这个函数会修改整个 form 的 data，这样就不得不更新整个表单的数据，因为需要进行一次 clone，然后再重新赋给form 
  setValueByName 
  
  ```

  很奇怪的一个点是，在 `Control` 中有两个`onChange`，一个是`form` 的，一个是`Control` 的，这两个有什么区别吗？ 

  

  那现在要搞清楚 formily 中为什么可以单个的formItem 更新了

  从 [我又重构了Formily内核](https://zhuanlan.zhihu.com/p/149981696) 从作者有说

  > 还记得从UForm开始，我们对外宣导的概念就是分布式状态管理，也就是说，我们每个字段状态都是独自管理的，这样就能保证每次频繁输入的时候，只会渲染当前字段，而不需要整体渲染，从而表单就不会因为字段数量增加而变卡，这样就获得了最佳的输入体验。

   [面向复杂场景的高性能表单解决方案(背景篇)](https://zhuanlan.zhihu.com/p/62927004)  这篇文章有几个重点

  1. 如何解决从内部到外部以及从外部到内部的通信问题

     使用 effects 和 actions

  2. 如何实现在formItem 更新的时候，只是更新当前组件，而不是整个表单更新

     借用 react-final-form 的方案，还是没有搞懂，在 final-form 中是如何只更新单个的 formItem 的

     在 submit 提交中，看到 value 的提交是通过 state.

     答案就在这里

     1. 看到 这里有个需要计算的 demo  https://codesandbox.io/s/oq52p6v96y?file=/index.js
     2. 更新某个字段的值是如何影响到其他的字段的
     3. 看到触发其他字段更新是用的 `form.change` 事件 
     4. `form.change` 中调用了`changeValue(state, name, () => value)` 这个函数 

     ```js
     // final-form/src/FinalForm.js:218
     const changeValue: ChangeValue<FormValues> = (state, name, mutate) => {
       const before = getIn(state.formState.values, name)
       const after = mutate(before)
       // 奇怪，这里都更新整个 formState.values，难道不会触发整个 form 的更新吗？
       // setIn 里调用 setInRecursor 最终都会重新生成一个 values 然后返回，居然还不会更新整个form？  
       state.formState.values = setIn(state.formState.values, name, after) || {}
     }
     ```

     react-final-form 中，大量的使用了 Subscription，看一下这里面究竟是要干嘛？

     在 [面向复杂场景的高性能表单解决方案(入门篇)](https://zhuanlan.zhihu.com/p/66852916) 中有说到可能会导致整个 form 重新渲染的原因 

     > value：主要用于受控场景，结合 onChange 使用，不推荐这种写法，应该会导致全局 Field rerender；

     在 formily 中用到了一个[react-eva](https://github.com/janryWang/react-eva) 中有介绍
     
     > Faster than one-way data stream, Because when we manage the state distribution, the entire React tree will not be fully redrawn due to a state change.
     
     也就是其实没有自动更新整个表单的核心是这里 state manage 的方式不一样，脱离了 formItem 本身的 `onChange` 事件就可以了。
     
     
     
     在 amis 中因为是自动更新整个 form 的 values，所以在每个 formItem 中设定好当前数据的变化
     就不用在某个 formItem 的数据变化后手动操作了，比如使用`visibleOn` 。
     
     即使 amis 中有进入可视区域才渲染，但是在渲染完成后，不在可视区域后，会重新注销吗？
     然后进入后再重新渲染？如果是这样，那不是会有多次的渲染后销毁的过程？就为了form 某个数据不导致整个 form 重新渲染导致可能的性能变化才不得不这样吗？这样的性能优化是不是有点过？

​			

- formily 中同样也没有实现递归渲染的问题，不是现在的combo 增加，而是 Tree 中不断增加子类型的这种增加 

---

搞清楚在 formily 中的 effects/actions 中做了什么？为什么要这么做 

effects 是使用的 rxjs的效果，这样他才能够使用 subscribe 的效果，那是如何做到能够匹配到 form.path 的效果呢？

·那 actions 呢？通过 createActions 吗？

在 `packages/react/src/shared.ts:29` 中设定了整个的 `createFormActions` ，然后在effects 中可以如下使用

```js
// 在 form 外 
const actions = createFormActions();
// 在 form 的 effects 中
actions.setFieldState("mobile", state => {
  state.visible = false;
});
```

很奇怪的是，在 `createFormActions` 中只是调用了`react-eva` 的 `creactActions` ，然后将`setFieldState` 这些字符串传入，到 `createActions` 中只是将这些参数放到对象`actions = {}`中，并没有与什么进行绑定啊，为什么这些`setFieldState` 就可以通过类似`mobile` 这些在 `form` 中的路径参数进行获取到对应的 `field` 呢，而且获取到还可以设置对应的数据呢？

## 理清 Effects

如

```js
// 使用案例
const { onFieldValueChange$ } = FormEffectHooks

effects={() => {
  const { setFieldState } = createFormActions()

  onFieldValueChange$('aa').subscribe(({ value }) => {
    setFieldState('*(bb,cc,dd)', state => {
      state.visible = value
    })
  })
}}

// 解析
// 1. onFieldValueChange$  packages/react/src/shared.ts:340
onFieldValueChange$: createEffectHook<IFieldState>(
    LifeCycleTypes.ON_FIELD_VALUE_CHANGE
  )
// 2 packages/react/src/shared.ts:249
export const createEffectHook = <TResult, Props extends Array<any> = any[]>(
  type: string
) => (...args: Props): Observable<TResult> => {
  return env.effectSelector(type, ...args)
}

// 3 . effectSelector packages/react/src/shared.ts:202
env.effectSelector = <T = any>(
  type: string,
  matcher?: string | ((payload: T) => boolean)
) => {
  const observable$: Observable<T> = selector(type)
  return observable$
}

// 4. effectSelector 在 createFormEffects 中赋值，所以需要找到 createFormEffects什么时候会被调用
const { dispatch } = useEva({
   effects: createFormEffects(effects, form)
})

// 5. 看 react-eva 中给 effects 传入了什么（因为 createFormEffects 是个 curry 函数，传入了 selector）
const subscription = () => {
    if (isFn(effects)) {
      // effects 里的这个参数是一个函数，也就是 selector 
      effects((type, $filter) => {
        if (!subscribes[type]) {
          subscribes[type] = new Subject()
        }
        if (isFn($filter)) {
          return subscribes[type].pipe(filter($filter))
        }
        return subscribes[type]
      })
    }
  }
```



