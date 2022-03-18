## 明白了 amis 使用 path 来匹配路径的好处了

- 有哪些地方需要使用 text(input) 这种类型的组件

  - 表单
  - 表格

  那么在用户配置表单的时候，在 `controls` 中添加Input 这种组件是很常见的，因为这种输入型的本来就是在表单中，也就是在 form 下，那么在 table 中，这时候设置 text 是只是展示的，那么在 text 中进行匹配的时候，就可以根据是否匹配到了`controls` 这种路径来确定是否是只展示还是输入框

  这时候就可以通过路径这种方式来解决了，也就是不需要用户来判断，我在表单还是表格下来加配置来解决

  那我如果要在表格中添加输入框怎么，那么就是另外的一层配置了，这时候就是在 table / column 中配置 editable = true 这种方式来进行，同时来配置这种 table 在编辑的时候通过什么方式来展示

## 明白了 amis 是如何给组件创建对于的 store

- 明白了使用 HocStoreFactory 是如何来 form 这些组件加上 store 的

  ```tsx
  // src/factory.tsx:172
  // 是在这里加上的 store
  if (config.storeType && config.component) {
    // 得看下 HocStoreFactory 做了什么，
    // 就是所有有 storeType的，都要在这里创建一个 store，然后传递给组件作为 store 的属性
    // 那么这个 store 是如何 和 mobx 联系上的呢，你看下面的 component 是用 observer 包裹了的
    config.component = HocStoreFactory({
      storeType: config.storeType,
      extendsData: config.storeExtendsData,
      shouldSyncSuperStore: config.shouldSyncSuperStore
      // 这里只是加上了 observe，就能够和 store 能够连上关系吗？
    })(observer(config.component));
    
    
    /*
    * HocStoreFactory 首先是创建了一个 store -> rootStore.addStore 比如 form 创建了一个 formStore
    * 然后将这个 formStore 作为高阶组件使用 props 传递给 form
    * 因为 form 下的所有组件都是在 form 渲染之后，所以可以通过在 form 渲染 formItem 的时候，将 store 通过 props 往下传
    * 
    * observer 的意思是，只要有 state/props 有变化，就触发这个 component 的更新呀
    * 难道每个组件都加上了 observer 吗？应该没有吧，那是如何做到的当 store 中的数据改变了，同步到了 component 呢
    * 因为我们改变了 form 的数据啊，也就是改变了 父级的数据啊，那既然 form 中的 data 会更新，传递给子组件的 props 中的 data 也就更新了，也就会触发子组件的更新啊
    */
  ```

  **问题**  

  1. form是会重新渲染的，每次重新渲染都会重新加上一个新的 store 吗，如果不会
     1. 在 form/index 是没有添加 store 的逻辑的 ，是因为通过高阶组件把 store 从 props 中传下来了，这时候也会同样的传给子组件
     2. 

## 明白了 amis 是如何来做主题的

- 首先在 factory 中获取到 props 的 theme，然后通过将这个 theme 这个字段通过 `ScopedRootRenderer` 就能往下传

- 因为最后要渲染组件，就肯定要使用到 `SchemaRenderer` ，然后 `theme` 通过props 传到这里，这里会找到组件然后渲染，然后就可以把 `theme` 这个字段传给组件了

  >  卧槽，这种通过外部传入属性然后改变所有组件的属性配置的方法简直太方便了，根本就不用什么 context 嘛，

- 还有个问题就是，既然选择了哪种主题，那么加载文件的时候是怎么来选择加载的？

  - fis 自动分析的吗？有用到的才加载进来吗？