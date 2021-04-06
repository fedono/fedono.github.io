amis 的验证

在 formStore 和 formItemStore 中都有 validate 验证的函数，需要理清这里面

formStore 中有两个验证函数，一个是validate，另一个是 validateFields

- validate 会验证当前 form 下所有的 item 的 validate 函数，并且将传入的 hooks 中的函数也执行一遍

```js
for (let i = 0, len = items.length; i < len; i++) {
  let item = items[i] as IFormItemStore;

  // 验证过，或者是 unique 的表单项，或者强制验证
  if (!item.validated || item.unique || forceValidate) {
    yield item.validate();
  }
}

if (hooks && hooks.length) {
  for (let i = 0, len = hooks.length; i < len; i++) {
    yield hooks[i]();
  }
}
```

​	validate 函数在 `src/renderers/Form/index.tsx:450` 中执行

```js
validate(forceValidate?: boolean): Promise<boolean> {
  const {store} = this.props;

	this.flush();
	return store.validate(this.hooks['validate'] || [], forceValidate);
}
```

​	这里的 `this.hooks['validate']` 是整个表单验证非常核心的一部分，整个表单的验证分两个方面，一个是在 `formItemStore`  中的验证函数，另外一个就是这个在formItem 组件中通过 添加函数到`this.hooks['validate']` 中，然后执行表单的验证时，将这个传给 `formStore` 的 `validate` 函数进行执行验证。

在`src/renderers/Form/index.tsx` 中会在渲染`formItem` 的时候，将`addHook` 也就是往`this.hooks['validate']` 添加验证的函数传给每个 `formItem` 组件，这样就能够收集到每个 `formItem ` 需要处理的验证函数 

`formItem` 可以添加验证的有两个地方

1. 在`formItem` 组件中使用 `form` 传下来的 `addHook`  往`this.hooks['validate']` 中添加函数，如

   ```js
   // src/renderers/Form/Control.tsx:139
   componentDidMount() {
     const {
       store,
       formStore: form,
       control: {name, validate},
       addHook
     } = this.props;
   
     const formItem = this.model as IFormItemStore;
     if (formItem && validate) {
       let finalValidate = promisify(validate.bind(formItem));
       this.hook2 = function () {
         formItem.clearError('control:validate');
         return finalValidate(form.data, formItem.value, formItem.name).then(
           (ret: any) => {
             if ((typeof ret === 'string' || Array.isArray(ret)) && ret) {
               formItem.addError(ret, 'control:validate');
             }
           }
         );
       };
       addHook(this.hook2);
     }
   }
   ```

2. 在 `formItem` 中添加`rules` ，在`formItemStore` 的 validate 函数，会验证当前`formItemStore` 中的`rules` 

   ```js
   doValidate(
     self.value,
     self.form.data,
     self.rules,
     self.messages,
     self.__
   )
   ```

   这里的`doValidate` 就是 `src/utils/validations.ts`  验证所有 amis 内置的规则的函数`validate` ，如`isEmail/isUrl` 等验证规则，所有的规则在 `src/utils/validations.ts:26`  中的`validations `

- validateFields 则是将传入的需要验证的 formItem 的 validate 执行一遍

  ```js
  for (let i = 0, len = items.length; i < len; i++) {
    let item = items[i] as IFormItemStore;
  
    // 传进来的 fields 如果在 items 中就验证
    if (~fields.indexOf(item.name)) {
      result.push(yield item.validate());
    }
  }
  ```

  validateFields 则是在 `src/renderers/Form/index.tsx:552` 中执行。也就是当前的某项操作需要某几个  formItem 验证通过才行，所以这里的 action.required 是 formItem 的 name 的集合。

  ```js
  handleAction(
      e: React.UIEvent<any> | void,
      action: Action,
      data: object,
      throwErrors: boolean = false,
      delegate?: IScopedContext
    ): any {
    
    	if (Array.isArray(action.required) && action.required.length) {
        return store.validateFields(action.required).then(result => {
          ...
        });
      }
   
  }
  ```

  

