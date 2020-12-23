---
layout: post 
title: "react-jsonschema-form 分析" 
author: "fedono"
---

[react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form) 是一个用户编写`json-schema ` 就能够解析成为`form` 的库，支持自定义主题，并且提供用户注册组件、主题和模板。



## 用法:

```js
const Form = JSONSchemaForm.default;
const schema = {
  type: "string"
};

ReactDOM.render((
  <Form schema={schema} />
), document.getElementById("app"));
```

就可以渲染出一个input输入框和提交按钮。

## 渲染逻辑

内部渲染逻辑如下：

```js
// packages/core/src/components/Form.js:448
<FormTag // FormTag 默认为  <form 
  className={className ? className : "rjsf"}
  ...
  onSubmit={this.onSubmit}
  >
  {this.renderErrors()}
  <_SchemaField // SchemaField 组件
    schema={schema}
    ...
    disabled={disabled}
  />
  {children ? (
    children
  ) : (
    <div>
      <button type="submit" className="btn btn-info">
        Submit
      </button>
    </div>
  )}
</FormTag>
```

通过将 schema 传给 `SchemaField` 来渲染`schema` 

rjsf 内部的组件统一为`widget` ，通过`field ` 来封装`widget` ，官方的解释为

> - A *widget* represents a HTML tag for the user to enter data, eg. `input`, `select`, etc.
> - A *field* usually wraps one or more widgets and most often handles internal field state; think of a field as a form row, including the labels.

在 `widgets` 与`fields` 文件夹中都有`index` 文件将所有的`widget/field`  import 然后 export 出来

在`SchemaField` 中就通过用户输入的`type` 来在所有的`fields` 找到`filed` 然后将 `schema ` 传给 `field` ，`field` 中找到对应的`widget` 然后渲染。

`SchemaField` 中也会把`label/description/disabled/readonly/classNames` 这些处理，然后一起渲染，相当于处理了`formItem ` 中的逻辑。

## 数据验证

rjsf 通过引入[ajv](https://www.npmjs.com/package/ajv) (Ajv: Another JSON Schema Validator)来进行验证， 验证失败后会将错误信息显示在 form 的最顶部。



## 总结

- rjsf 总的来说是一个实现了通过`json-schema` 来渲染表单的库，调查了业内一圈号称自己使用了`json-schema ` 来渲染页面的库，只有`rjsf` 是真的按照 `json-schema` 的协议来实现，无论是`formily` 还是`amis` 都是在`json-schema` 的基础上做改造，并未实现`allOf/oneOf` 这类，基本上都是通过`json-schema` 来做组件的描述。
- rjsf 代码并不复杂，当前实现的功能也并不多，核心就是能够实现`json-schema` 来渲染出表单。缺点是组件库少，没有数据管理，没有表单联动，没有复杂的布局，没有列表，没有自增，没有递归渲染等等，相比于`formily` 和`amis` 的确还是不能够覆盖大部分的场景。只能是作为一个从`json-schame`概念上实现的比较好的库。



