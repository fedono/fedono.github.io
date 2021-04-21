关于可视化编辑器

- 页面结构划分
  - 中间的编辑区域
  - 左边的可选组件、大纲、和JSON配置
  - 右边是对应的组件配置

- 如何实现的拖拽功能
  - 





> 先通过一个组件的配置来理清楚整个流程，然后再去看其他的组件的配置

```js
data-editor-id
```

1. 在 `JSONPipeIn` 函数中，会给当前的配置添加上`$$id` 

   ```js
   toUpdate.$$id = guid();
   ```

   然后后续会通过 id 来找到对应组件的配置

2. 所有的组件都会通过 `@Editor` 来进行配置

   - 然后会渲染器中开放的接口`RendererComponent` 来通过`name` 来找到对应的组件

   - 也就是编辑器中，全局的组件配置中，会把单个组件的配置与组件放到一起，后续应该会拿到这个组件来进行渲染，需要看一下 preview 这个组件是怎么做的
   - 渲染器既然把`renderSchema` 的方法开放出来了，那么就肯定能使用路径来匹配到组件然后渲染，为啥还要在组件的配置到全局的时候，还把组件添加进来呢 

3. 通过`Preview` 组件来进行渲染，`Preview` 组件中有`onDrop` 事件，但是在`Editor` 组件中调用 `Preview` 的时候，没有看到`onDrop` 事件传入，只能在调用 `Editor` 的时候，看下有没有这个熟悉传入了

   - 从左边的组件选择区域拖拽到编辑区域，本来就是在编辑区域监听`onDrop` 事件的呀，为啥会在`Preview` 中没有这个事件呢？
   - 在`miniEditor` 中，有用到`Preview` 组件，在这里传入了`onDrop` 事件，可是`miniEditor` 又是做什么的？也没有哪里有使用`MiniEditor` 的，而且也没导出这个`MiniEditor`  

4. 之前的编辑器版本是不支持拖拽的啊，我草

   1. 就是选中当前编辑区，然后如果当前编辑区域是可以添加子组件的，那么会出现一个加号，然后通过这个加号，弹出可以添加的组件，这时候点击之后，就会添加组件到当前的页面中来
   2. 点击这个加号的时候，就可以获取到当前组件的path，然后通过这个 path 去匹配到所有的可以使用的组件，然后提供用户进行筛选

5. `BasicEditor` 有个 `active` 的函数，这个就是在编辑区域中，点击组件后，会触发当前函数，然后获取到当前选中组件的配置，然后在左侧渲染出当前组件的所有配置

6. 有一个问题，在`Preview` 中是根据`schema` 来渲染页面的，那么在编辑的时候，是怎么在元素上添加的高亮的区域的

   1. 因为渲染 schema ，和 `id` 没啥关系呀，这里也没有用到 `BasicEditor` 的 
2. 所有的组件的配置都是继承 `BasicEditor` 的 
   3. 只是所有的组件配置是继承`BasicEditor` 的，渲染的时候还是获取到全局的 json-schema 来配置的，这时候`BasicEditor` 中的 `render` 渲染含有用什么用呢？又不会去执行`BasicEditor` 
4. 所有的组件注册到全局的 `Editors` 的时候，会设置一个 `component` ，应该会从这里获取到组件的配置，然后渲染，要渲染什么呢？就是渲染当前组件的配置对不，虽然在`BasicEditor` 中还有`HighLight` 组件，但是 `HightLight` 组件是通过`Portal` 定位到组件那里去，所以能做到这个的拆分
   5. 是如何做到在`Preview` 渲染页面的时候，在元素上加上`editor-data-id` 这个属性的

      1. 我们是渲染`schema` 啊，如何来做到这个注入的
6. 通过`Editors` 注册了组件的配置，然后呢？没见着哪里有用到啊
      1. 一个是 `Preview` 里面渲染了当前页面的`schema` ，可是和 `BasicEditor` 有什么关系呢
   2. 什么时候在`Preview` 里面，能够获取到
   7. 加载编辑器的时候，会加载所有的组件配置，这时候会自动注册到全局里去，可是在什么时候调用到这些组件的配置啊？
   8. 在 Editor 这个主文件中，会调用两个组件，一个是左边的设置，一个是中间的编辑区，编辑区域是通过渲染 schema 来生成，配置区则是通过点击编辑区域中的元素，然后获取到 id ，然后匹配到对应的组件配置，在左边的配置区域中渲染
      1. 编辑区中的元素，是如何加上 `data-editor-id` 这种属性的 ，现在最多的的疑问就是这里了
      2. 所有`BasicEditor` 中都有一个 `highlightBox` 组件包裹，然后添加 `data-editor-id` ，所有的容器元素，都会通过`Region` 组件包裹，然后添加上 `data-host-id` ，但是所有继承`BasicEditor` 的组件，不都是配置组件吗？和渲染的组件有什么关系
      3. 等一等？这里通过注册组件，将所有的渲染器中组件加载进来了，然后会包裹一成，也就是在这里获取到的组件不再是渲染器中的组件，而是在编辑器这里对渲染器中的组件封装了一层，然后再去渲染的时候，都是通过获取到编辑器中的组件来渲染的？所以在编辑器中的组件渲染的时候，添加了一层 `data-editor-id` 和 `data-host-id` 的属性 ，感觉应该是这样
      4. 也就是渲染器应该会把这个入口放开，然后在编辑器中将所有的组件封装了配置进去，就这样来完成
      5. 卧槽就是，在`Preview` 里面有个`rendererResolver` 就是从注册的全局组件配置里来找组件进行渲染
         1. 在渲染中就有通过 `env` 中是否有`rendererResolver` 来查找组件的配置，终于理顺了，沃日啊，真会玩
   



## 参考

- [页面可视化搭建工具前生今世](https://github.com/CntChen/cntchen.github.io/issues/15) 

github 上的相关代码

- [hetu](https://github.com/LianjiaTech/hetu)

-  [react-visual-editor](https://github.com/brick-design/react-visual-editor)



更多

https://github.com/topics/low-code?l=typescript

