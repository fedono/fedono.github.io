---
layout: post
title:  "AMis 如何寻找组件"
author: "fedono"
---

在 `src/factory.tsx` 中寻找组件的方法如下 

```react
export function resolveRenderer(
  path: string,
  schema?: Schema,
  props?: any
): null | RendererConfig {
  if (cache[path]) {
    return cache[path];
  } else if (path && path.length > 1024) {
    throw new Error('Path太长是不是死循环了？');
  }

  let renderer: null | RendererConfig = null;

  renderers.some(item => {
    let matched = false;

    // 根据 path 来命中组件
    if (typeof item.test === 'function') {
      matched = item.test(path, schema, resolveRenderer);
    } else if (item.test instanceof RegExp) {
      matched = item.test.test(path);
    }

    if (matched) {
      renderer = item;
    }

    return matched;
  });

  // 只能缓存纯正则表达式的后者方法中没有用到第二个参数的，因为自定义 test 函数的有可能依赖 schema 的结果
  if (
    renderer !== null &&
    ((renderer as RendererConfig).test instanceof RegExp ||
      (typeof (renderer as RendererConfig).test === 'function' &&
        ((renderer as RendererConfig).test as Function).length < 2))
  ) {
    cache[path] = renderer;
  }

  return renderer;
}
```

所有的寻找方法其实就是根据正则来命中组件的路径

```react
// 判断组件中的 test 来判断
if (typeof item.test === 'function') {
      matched = item.test(path, schema, resolveRenderer);
    } else if (item.test instanceof RegExp) {
      matched = item.test.test(path);
    }
```

那么在组件中的 `test` 该如何写呢？我们看下 `src/renderers/Table.tsx`  这个文件

table 组件中有三个 `test` ，如`table` 的 `test`  如下

```react
@Renderer({
  test: (path: string) =>
    /(^|\/)table$/.test(path),
  storeType: TableStore.name,
  name: 'table'
})
```

这里的 `test` 是个 `function` ，需要通过 `path` 来判断（其实这里写成正则的字符串形式也是可以的），

我们看下 `table-cell` 的判定规则

```react
@Renderer({
  test: /(^|\/)table\/(?:.*\/)?cell$/,
  name: 'table-cell'
})
```

`table-cell` 需要判断 `table` 以及 `cell` 同时存在，所以单独的 `table` 的规则中有 `$` 符号，来说明判断`table` 时 `table` 是`path` 中最后的位置了。



form 下的组件因为会通过 `src/renderers/Form/Item.tsx` 来 统一封装公共的方法，所以所有的 `test` 也是在这里统一设定，form 下的组件通过设定自身组件的 `type` 字段为自身的名称，然后在 `Item` 组件中统一设定 `test` 属性

```react
return registerRenderer({
    ...config,
    name: config.name || `${config.type}-control`,
    weight: typeof config.weight !== 'undefined' ? config.weight : -100, // 优先级高点
    test:
      config.test ||
      new RegExp(
        `(^|\/)form(?:\/.+)?\/control\/(?:\d+\/)?${config.type}$`,
        'i'
      ),
    component: FormItemRenderer,
    isFormItem: true
  });
```

