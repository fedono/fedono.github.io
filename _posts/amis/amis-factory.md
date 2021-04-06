factory 的处理逻辑



## 渲染的过程 

1. `render` 函数开头

   ```js
   function render(
     schema: SchemaNode,
     props: RootRenderProps = {},
     options: RenderOptions = {},
     pathPrefix: string = ''
   ) {
     ...
     return (
       <ScopedRootRenderer
         {...props}
         schema={schema}
         pathPrefix={pathPrefix}
         rootStore={store}
         env={env}
         theme={theme}
         locale={locale}
         translate={translate}
       />
     );
   } 
   ```

2. 调用 Scoped 来封装 RootRenderer

   ```JS
   export const ScopedRootRenderer = Scoped(RootRenderer);
   ```

3. `RootRenderer` 将各种 `context` 以及处理的`definitions` 的方法下传

   ```js
   export class RootRenderer extends React.Component<RootRendererProps> {
     
     ...
     render() {
     	<RootStoreContext.Provider value={rootStore}>
           <ThemeContext.Provider value={this.props.theme || 'default'}>
             <LocaleContext.Provider value={this.props.locale!}>
               <ImageGallery modalContainer={env.getModalContainer}>
                 {
                   renderChild(
                     pathPrefix || '',
                     isPlainObject(schema)
                       ? {
                           type: 'page',
                           ...(schema as any)
                         }
                       : schema,
                     {
                       ...rest,
                       resolveDefinitions: this.resolveDefinitions,
                       location: location,
                       data: finalData,
                       env,
                       classnames: theme.classnames,
                       classPrefix: theme.classPrefix,
                       locale,
                       translate
                     }
                   ) as JSX.Element
                 }
               </ImageGallery>
             </LocaleContext.Provider>
           </ThemeContext.Provider>
         </RootStoreContext.Provider>
   	}	
   }
   ```

4. `RootRenderer` 的核心还是 `renderChild` 

   ```js
   export function renderChild(
     prefix: string,
     node: SchemaNode,
     props: renderChildProps
   ): ReactElement {
     if (Array.isArray(node)) {
       // renderChildren 来处理循环，如果是单个，就调用 renderChild 来处理
       return renderChildren(prefix, node, props);
     }
     
     return (
       <SchemaRenderer
         {...props}
         {...exprProps}
         schema={schema}
         $path={`${prefix ? `${prefix}/` : ''}${(schema && schema.type) || ''}`}
       />
     );
   }
   ```

5. `SchemaRenderer` 来处理匹配到路径的组件，然后开始渲染

   ```js
   class SchemaRenderer extends React.Component<SchemaRendererProps, any> {
     ...
     render() {
       return (
         <Component
           {...theme.getRendererConfig(renderer.name)}
           {...restSchema}
           {...chainEvents(rest, restSchema)}
           defaultData={defaultData}
           $path={$path}
           ref={this.refFn}
           render={this.renderChild}
         />
       );
     }
   }
   ```

   

## 注册

1. 提供注册方法（提供一个 curry 的函数）

   ```js
   export function Renderer(config: RendererBasicConfig) {
     return function <T extends RendererComponent>(component: T): T {
       const renderer = registerRenderer({
         ...config,
         component: component
       });
       return renderer.component as T;
     };
   }
   ```

2. 使用`registerRenderer` 将注册的组件放到一个全局的 数组中（后续根据组件名称来在这里查找） 

   ```js
   export function registerRenderer(config: RendererConfig): RendererConfig {
     rendererNames.push(config.name);
   }
   ```

   - 注册的组件有特殊的参数，需要针对性的处理

     ```js
     // 有自己的 store 的组件，需要给组件的 props 中加上对应的 storeType 的 store 
     if (config.storeType && config.component) {
       config.component = HocStoreFactory({
         storeType: config.storeType,
         extendsData: config.storeExtendsData
       })(observer(config.component));
     }
     
     // 针对当前的组件，设置单独的作用域
     if (config.isolateScope) {
       config.component = Scoped(config.component);
     }
     ```

     - `HocStoreFactory` 将该 store 加到对应的组件的 `props` 中，然后把 `store` 中的 data 也要放到 组件的 props 中

     ```js
     export function HocStoreFactory(renderer: {
       storeType: string;
       extendsData?: boolean;
     }) {
       return function() {
         @observer
         class StoreFactory extends React.Component<Props> {
           
           componentWillMount() {
             // 根据 storeType 来获取到 mst 中写好的 store 
             const store = (this.store = rootStore.addStore({
               id: guid(),
               path: this.props.$path,
               storeType: renderer.storeType,
               parentId: this.props.store ? this.props.store.id : ''
             } as any));
           }
     
           render() {
     				return (
               <Component
                 {
                   ...(rest as any) 
                 }
                 {...exprProps}
                 ref={this.refFn}
                 data={this.store.data}
                 dataUpdatedAt={this.store.updatedAt}
                 store={this.store}
                 scope={this.store.data} // 有点不明白为什么要把 store.data 放到 scope 里面来？
                 render={this.renderChild} // 这里把renderChild给每个组件，这样组件就可以使用 this.props.render 来渲染内部的 schema 了
               />
             );
           }
         }
         
         return StoreFactory
       }
     }
     ```

     - `Scoped` 则是将当前组件设置一个单独的作用域，这样就可以使用`reload/target` 这些方式让这些组件自动更新了

       ```js
       // Scoped 就是 HocScoped 这个 curry 函数
       config.component = Scoped(config.component);
       
       
       export function HocScoped(
         ComposedComponent: React.ComponentType<T>
       ) {
         class ScopedComponent extends React.Component {
           // 通过 createScopedTools 来建立当前组件注册到 Scope 中，Scope中的组件可以互相使用 reload/send 通信
           scoped = createScopedTools(this.props.$path, this.context, this.props.env);
       
           render() {
             const {scopeRef, ...rest} = this.props;
       
             return (
               <ScopedContext.Provider value={this.scoped}>
                 <ComposedComponent
                   {
                     ...(rest as any) 
                   }
                   ref={this.childRef}
                 />
             	</ScopedContext.Provider>
           );
         }
       }
       
       return ScopedComponent;
       }
         
       
       // createScopedTools 这个注册当前的组件到 components 中，然后后续可以从 components 中获取到这个 component，调用 component 的 reload / send 等方式完成组件的更新  
       function createScopedTools(
         path?: string,
         parent?: AlisIScopedContext,
         env?: RendererEnv
       ): IScopedContext {
         const components: Array<ScopedComponentType> = [];
       
         return {
           parent,
           registerComponent(component: ScopedComponentType) {},
       
           unRegisterComponent(component: ScopedComponentType) {},
       
           getComponentByName(name: string) {},
       
           getComponents() {},
       
           reload(target: string, ctx: any) {},
       
           send(receive: string, values: object) {},
       
           close(target: string | boolean) {}
         };
       }  
       ```

       

