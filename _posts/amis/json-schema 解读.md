JSON 渲染的前提就是需要了解到 [json-schema](https://json-schema.org/learn/getting-started-step-by-step) 

- 是用来干什么的
- 为什么需要 json-schema
- 为什么还需要有标准来迭代，每个标准是经历了什么，为什么这种 oneOf 这种还需要定义标准呢？

其实文档中的标准已经说得很清楚了

> Let’s pretend we’re interacting with a JSON based product catalog. This catalog has a product which has:
>
> - An identifier: `productId`
> - A product name: `productName`
> - A selling cost for the consumer: `price`
> - An optional set of tags: `tags`.
>
> For example:
>
> While generally straightforward, the example leaves some open questions. Here are just a few of them:
>
> - What is `productId`?
> - Is `productName` required?
> - Can the `price` be zero (0)?
> - Are all of the `tags` string values?
>
> When you’re talking about a data format, you want to have metadata about what keys mean, including the valid inputs for those keys. **JSON Schema** is a proposed IETF standard how to answer those questions for data.

也就是 JSON SCHEMA 是用来定义以及规范数据类型的

那为什么 JSON 渲染的工具需要使用 json-schema呢

- JSON 中使用的组件是有限的
- JSON中定义的组件的属性是规定好了的

那这既然是一个定义，那我们在开发 json 渲染工具的时候，json-schema 能用来干什么？

- 在用户编写 JSON 的时候做提示

- 在用户编写 JSON 的时候做类型定义，如果出错会提示出错 

- 在编写 JSON 的时候，我们会设定一条数据

  ```
  $schema: 'https://xxx.com/abc'
  ```

  > The [`$schema`](https://json-schema.org/draft/2020-12/json-schema-core.html#rfc.section.8.1.1) keyword states that this schema is written according to a specific draft of the standard and used for a variety of reasons, primarily version control.



---

基本上了解上面就差不多了，再了解一下 [JSON Schema Validation](https://json-schema.org/draft/2020-12/json-schema-validation.html)  就可以