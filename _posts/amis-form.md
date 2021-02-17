## 表单的渲染逻辑

文件地址`src/renderers/Form/index.tsx` 

## 注册

首先看注册的配置，从下面的注册的配置看，form会有自己的 store 为 FormStore，也会有自己独立的作用域，也就是设置了`isolateScope` 那么在内部的组件就可以使用 `target/reload/send` 这些方法 

```js
@Renderer({
  test: (path: string) =>
    /(^|\/)form$/.test(path) &&
    !/(^|\/)form(?:\/.+)?\/control\/form$/.test(path),
  storeType: FormStore.name,
  name: 'form',
  isolateScope: true
})
```

## 渲染逻辑

- 首先看 render

1. `renderBody` 会渲染form的所有的内部的表单项
2. 如果form 需要使用 panel 包裹，则会加上一些配置，然后将 renderBody 包裹住
3. `lazyLoad` 一般是因为表单内部太多了，渲染导致卡顿的话，就需要配置成在可视区域内的表单项才渲染，这样就可以解决卡顿的问题

```js
render() {
    let body: JSX.Element = this.renderBody();

    if (wrapWithPanel) {
      body = render(
        'body',
        {
          type: 'panel',
          title: __(title)
        },
        {
          ...
          children: body,
        }
      ) as JSX.Element;
    }

    if (lazyLoad) {
      body = <LazyComponent>{body}</LazyComponent>;
    }

    return body;
  }
```

### renderBody 如何处理内部的渲染

1. body 中一般都是只有 `fields/tabs/controls` ，所以渲染这三个就可

   ```js
   renderBody() {
     return (
       {this.renderFormItems({
          tabs,
          fieldSet,
          controls
       })}
   	)
   }
   ```

2. renderFormItems 中处理 所有的内部schema

   ```js
   renderFormItems(
       schema: FormSchema,
       region: string = '',
       otherProps: Partial<FormProps> = {}
     ): React.ReactNode {
       
       return this.renderControls(
         schema.controls as SchemaNode,
         region,
         otherProps
       );
       
     }
   ```

3. renderControls 循环处理所有单个的 control

   ```typescript
   renderControls(
     controls: SchemaNode,
     region: string,
     otherProps: Partial<FormProps> = {}
   ): React.ReactNode {
     controls = controls || [];
   
     if (!Array.isArray(controls)) {
       controls = [controls];
     }
   
     return controls.map((control, key) =>
        this.renderControl(control, key, otherProps, region)
     );
   }
   ```

4. `renderControl` 来处理单个的表单项的渲染，使用从 factory 中一直给每个组件传下来的 `render` 函数来渲染schema

   ```js
   renderControl(
       control: SchemaNode,
       key: any = '',
       otherProps: Partial<FormProps> = {},
       region: string = ''
     ): React.ReactNode {
   
       const subProps = {
         formStore: form,
         data: store.data,
         ...
       };
   
       const subSchema: any =
         (control as Schema).type === 'control'
           ? control
           : {
               type: 'control',
               control
             };
       
       return render(`${region ? `${region}/` : ''}${key}`, subSchema, subProps);
     }
   ```

   





   

   