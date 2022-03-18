学习 typescript 

通过项目来学习会好一点

```
/Users/zengdeping/is-contract-life-manage-fe/node_modules/@types/react
首先通过这个 react 中的所有的定义来一个一个分析，来解决日常编写 React 相关代码所使用的 TypeScript 问题
```

- 在 @types/react/index.d.ts 中，使用 webstorm 打开 structure，会根据二中情况来分类，这样就好很好了

  选中 structure 上面导航栏中的第二个选项「group methods by defining type」

- 在 antd/lib/form/FormItem.d.ts 中有一个从枚举中取值的一个方式

  ```typescript
  declare const ValidateStatuses: ['success', 'warning', 'error', 'validating', ''];
  export declare type ValidateStatus = typeof ValidateStatuses[number]
  ```

  

