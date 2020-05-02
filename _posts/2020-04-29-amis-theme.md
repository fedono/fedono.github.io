---
layout: post 
title: "AMis 主题方案" 
author: "fedono"
---

代码在 `src/theme.tsx` 中，直接看 `themeable` 函数

```js
// 创建 theme 的 context
export const ThemeContext = React.createContext('theme');

// themeable 函数
// 设定当前的 context 类型
static contextType = ThemeContext;

// 获取到当前选定的 theme 
const theme: string = this.props.theme || this.context || defaultTheme;
const config = hasTheme(theme) ? getTheme(theme) : getTheme(defaultTheme);
const injectedProps = {
  classPrefix: config.classPrefix as string,
  classnames: config.classnames
};

// 通过 context 传递给组件
<ThemeContext.Provider value={theme}>
  <ComposedComponent
    {
      ...this.props as any
    }
    {...injectedProps}
  />
</ThemeContext.Provider>
);
```

如何在组件中设置主题呢

```js
// 在组件中的使用，如 src/components/Alert.tsx 
const {classnames: cx} = this.props;

<div className={cx('Modal-header')}>
     <div className={cx('Modal-title')}>{this.state.title || title}</div>
</div>

// 导出的时候，使用 themeable 来传递 theme 给 Alert 组件
export default themeable(Alert);
```

这里的 `classNames` 是`themeable`封装 提供给组件增加前缀的函数，这样在加载样式文件的时候，就能根据前缀匹配到特定组件的样式了。

在默认主题文件`src/themes/default.ts` 中，可以看到 

```js
import {theme, ClassNamesFn, makeClassnames} from '../theme';
export const classPrefix: string = 'a-';
export const classnames: ClassNamesFn = makeClassnames(classPrefix);

theme('default', {
  classPrefix,
  classnames
});
```

也就是默认主题的前缀为 `a-` ，我们看下 `makeClassnames` 函数

```js
export function makeClassnames(ns?: string) {
  if (ns && fns[ns]) {
    return fns[ns];
  }

  const fn = (...classes: ClassValue[]) => {
    // 这里的 cx 为 classnames 库
    const str = cx(...(classes as any));
    return str && ns
      ? str
          .replace(/(^|\s)([A-Z])/g, '$1' + ns + '$2')
          .replace(/(^|\s)\:/g, '$1')
      : str || '';
  };

  ns && (fns[ns] = fn);
  return fn;
}
```

该函数就是给内部的参数加上主题的前缀，所以在上面的组件`Alert` 中，通过 `className={cx('Modal-header')}` 设置后，该 `className` 就转换为 `className={'a-Modal-header'}` 了

AMis 中有两种情况需要处理，一种就是上述的组件，还有一种就是渲染器，渲染器是调用组件来使用的，部分渲染器可以直接实现，可以不调用组件，那么在渲染器中如何使用呢？

在 `factory` 中`RootRenderer`  设置如下，传递给要渲染的渲染器后，在渲染器中就可以使用 `classnames` 来设置元素的 `className` 了

```js
<RootStoreContext.Provider value={rootStore}>
  <ThemeContext.Provider value={this.props.theme || 'default'}>
    <ImageGallery>
    {
      renderChild(
        pathPrefix || '',
         isPlainObject(schema)
        ? {
          type: 'page',
          ...(schema as Schema)
        }
        : schema,
        {
        ...rest,
        resolveDefinitions: this.resolveDefinitions,
        location: location,
        data: finalData,
        env,
        classnames: theme.classnames,
        classPrefix: theme.classPrefix
      }
    ) as JSX.Element
  }
  </ImageGallery>
</ThemeContext.Provider>
</RootStoreContext.Provider>
```



- 一个不相关的疑问，为啥要把 `context` 设置成 `store` ，作用在哪

  ```js
  // StoreFactory 函数中 
  const rootStore = this.context;
  
  const store = (this.store = rootStore.addStore({
    id: guid(),
    path: this.props.$path,
    storeType: renderer.storeType,
    parentId: this.props.store ? this.props.store.id : ''
  } as any));
  ```

  在 `renderChild` 里面会把这个 `store` 传给渲染器想要使用 `this.props.render`  的时候来渲染 `schema` 的时候可以使用到顶层的 `store` 

  ```js
  renderChild(
    region: string,
    node: SchemaNode,
    subProps: {
    data?: object;
    [propName: string]: any;
  } = {}
  ) {
    let {render} = this.props;
  
    return render(region, node, {
      data: this.store.data,
      dataUpdatedAt: this.store.updatedAt,
      ...subProps,
      scope: this.store.data,
      store: this.store
    });
  }
  ```

  **这样做当然是好，但是我的问题是，为什么要把他放到 `context` 里？**

  > 回答这个疑问，这里不是把他放在 `context` 里，这里是 `StoreFactory `是个 `context `的`` consumer `，也就是在 `RootRenderer` 中使用了 `<RootStoreContext.Provider value={rootStore}></RootStoreContext.Provider>` 来把 `rootStore` 传下来，那么现在在 `StoreFactory` 中，我们就需要拿到这个 `store` 来使用了  

  你看在 `function render` 函数里，这里的 `store` 是直接把 `store` 就是直接引入的 `RendererStore` 文件，然后创建的，然后传入到 `ScopedRootRenderer` 中

  ```js
  <ScopedRootRenderer
    {...props}
    schema={schema}
    pathPrefix={pathPrefix}
    rootStore={store}
    env={env}
    theme={theme}
  />
  ```

  

