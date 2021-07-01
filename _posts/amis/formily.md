- form-render 是没有引入状态管理的，为什么 formily 要引入？
- formily 的文章要多看几遍，
  - formily 其实一直有强调，就是form的渲染性能问题，那么究竟 formily 是怎么解决的
  - 为什么要引入 rxjs
  - 为什么要设置 path ，为什么 form-render 不设置也能用
  - formily 究竟比 react-final-form 好在哪里
  - react-hooks-form 为什么能横空出世

- 如何本地调试 formily 
  - 使用 npm link 试试 





```
"start": "dumi dev",

"start": "dumi dev",
```

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { createForm } from '@formily/core'
import { FormProvider, FormConsumer, Field } from '@formily/react'
import {
    FormItem,
    FormLayout,
    Input,
    FormButtonGroup,
    Submit,
} from '@formily/antd';

const form = createForm();

function App() {

    return (
        <FormProvider form={form}>
            <FormLayout layout="layout">
                <Field
                    name="input"
                    title="输入框"
                    required
                    initialValue="Hello world"
                    decorator={[FormItem]}
                    component={[Input]}
                />
            </FormLayout>
            <FormConsumer>
                {() => (
                    <div
                        style={{
                            marginBottom: 20,
                            padding: 5,
                            border: '1px dashed #666'
                        }}
                    >
                        实时响应: {form.values.input}
                    </div>
                )}
            </FormConsumer>
            <FormButtonGroup>
                <Submit onSubmit={console.log}>提交</Submit>
            </FormButtonGroup>
        </FormProvider>
    )
}

ReactDOM.render(
    <App/>,
    document.getElementById('root')
)
```

