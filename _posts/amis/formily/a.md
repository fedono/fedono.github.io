```js
import { createForm } from '@formily/core'
```

`@formily/core ` 其实就类似于 `useForm` ，是用来提供 form 的各种方法的

```js
import { FormProvider, FormConsumer, Field } from '@formily/react'
```

`@formily/react` 其实是用来提供表单的渲染相关操作的，我只是很奇怪，为什么这些要放到 react 中去，这本来不是应该是正常 form 的行为吗？不是应该是通用的吗？

[文档](https://v2.formilyjs.org/zh-CN/guide/quick-start) 中是有解释像 `createForm/FormProvider/FormLayout/Field/FormConsumer/FormButtonGroup/Submit` 这种事用来干啥的

既然 mobx 是通过监听组件的 props/state 来实现重新渲染更新的，那肯定也是有一个比对的过程，这个比对的应该也是一个 compare 的方法，看下这个方法是不是根据引用的对比来计算的

以及既然是监听 props/state，那肯定也是使用 proxy 来监听 props/state ，所以需要理解是如何使用的 proxy 的

