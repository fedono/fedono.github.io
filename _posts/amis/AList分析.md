- effects 里面都是可以用来消费这些生命周期执行副作用

  ```js
  import { List, createListActions, ListLifeCycleTypes } from '@alist/antd'
  const App = () => {
    const actions = createListActions()
    return <List
        effects={($, actions) => {
          $(ListLifeCycleTypes.ON_LIST_WILL_INIT).subscribe(() => {
            console.log('list will init')
          })
          $(ListLifeCycleTypes.ON_LIST_INIT).subscribe(() => {
            console.log('list init')
          })
          $(ListLifeCycleTypes.ON_LIST_BEFORE_QUERY).subscribe((queryData) => {
            console.log('list before query', queryData)
          })
  
          $(ListLifeCycleTypes.ON_LIST_AFTER_QUERY).subscribe((resp) => {
            console.log('list after query', resp)
          })
        }}
        actions={actions}
      >
          ...
      </List>
  }
  ```

- Actions 

  `AList` 将所有列表相关的操作都抽象到 `ListActions`，我们可以通过 `createListActions` 获取它。

  通过使用它的API实现各种列表操作，如刷新，重置，清空等操作, 具体可以查看 [API列表](https://alist.wiki/iframe.html?path=/home/runner/work/alist/alist/docs/interface/actions.md#API列表)。

