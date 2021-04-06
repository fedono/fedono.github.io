amis 中的 service 有两个作用

1. 可以调用接口
2. 可以从接口获取到 schema ，然后渲染到页面中

amis  中有两个 service，还有一个是在 `form` 下的 service，

在`src/renderers/Service.tsx`  中的 service 是直接渲染在内部的 `schema` 

在`src/renderers/Form/Service.tsx` 中的 service 继承了 `src/renderers/Service.tsx`   中的 service并且在form渲染的时候，将 

`renderFormItems ` 通过 `props` 传给了 `formItem` 的每个组件，这样在 service 中就可以调用这个来渲染表单中的组件，也就是form下的 service 只能渲染表单中的组件。

