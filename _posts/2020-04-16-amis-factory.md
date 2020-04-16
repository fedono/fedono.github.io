---

---

1. `HocStoreFactory` 究竟有什么用

  说一下 `factory` 中的 `HocStoreFactory` 方法，这个方法只在两个地方用到了，那就在组件进行注册的时候用到的，一个就是在 `factory` 自身中，非 form 组件注册的时候，判断组件的配置中是否有 `storeType` ，如果有，就需要通过 `HocStoreFactory` 来作为高阶组件来封装一层，还有一个就是在 `form/Item` 中，是在 `form` 组件注册的时候，如果组件的配置中有 `storeType` ，那么就有这个高阶组件来封装一层

  那么这个方法究竟是用来干嘛的呢，核心就是一个，就是把整个 `amis` 最顶层的 `store`  通过 `props` 的方式，放到这个组件中去，顺带也把顶层的 `store.data` 这个数据也放到组件中去，因为 `amis` 使用的数据管理是 `mobx-state-tree` ，也就是所有的数据都是一个 `tree` 的结构，通过把顶层`store` 的数据放到组件中，也就是这个组件可以获取到所有的数据了

  ```react
  return (
    <Component
      {
        ...rest as any
      }
      {...exprProps}
      ref={this.refFn}
      data={this.store.data}
      dataUpdatedAt={this.store.updatedAt}
      store={this.store}
      scope={this.store.data}
      render={this.renderChild}
      />
  );
  ```

  需要注意的一个点是，在使用 `HocStoreFactory` 来封装组件的时候，会先使用 `mobx-react` 的 `observer` 来包住组件，这样在当前 `props` 有变化的时候，就能够实时更新了

  ```react
  if (config.storeType && config.component) {
      config.component = HocStoreFactory({
        storeType: config.storeType,
        extendsData: config.storeExtendsData
      })(observer(config.component));
    }
  ```

2. `isolateScope` 究竟是干什么的

  看代码当然知道是干什么的，但是在组件自身设置属性的时候，只有 `page`, `dialog` , `drawer` , `wizard` 和 `form/index` 中设置了 `isolateScope: true` 属性
  
  在 `factory ` 的注册组件方法 `registerRenderer` 中，如果检测到了有这个属性，就会使用 `Scoped` 这个高阶组件来封装一层
  
  ```react
  if (config.isolateScope) {
      config.component = Scoped(config.component);
    }
  ```
  
  `Scoped` 组件能够让封装的组件能够使用 `Scoped` 的方法，里面的方法就是组件能够设置 `reload` ，`target` 这些方法，能够让设置的对应的组件能够重新加载一次，或者把当前组件的数据发送到`target` 的组件。
  
  所以目前的猜测就是，只要组件中设计到需要数据封装的，就需要使用 `Scoped` 来封装一次了，这确实解释得通了，那么为什么 `crud` 和 `service`  这两个会进行数据处理的组件没有设置呢，理由就是 `service` 只是一个获取数据的组件，不会操作数据的组件，他只是有个 `api` 可以请求到数据后，然后可以给子组件进行使用，`crud` 也是同理，`crud` 组件本身是可以操作数据，但是 `crud` 其实也是个数据显示的组件，所以的操作还是需要交给内部的其他组件，如 `button` ，但是`button` 在渲染的时候，是通过`form` 来注册的，也就是在走了一层 `form/item` 的注册组件逻辑后，`button` 是有 `Scoped` 中的功能的
  
   