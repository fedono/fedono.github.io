要写一个 `collapse` 组件，

- 关键一个是在于更新`collapse` 内部的内容的高度
  - 整个项目中，`height` 共有三个值，`auto/ 0 / this.innerRef.scrollHeight` ，也就是根本就不用去计算，正常的设置高度应该是 `offsetHeight + marginTop + marginBottom` 
- 另一个是如何实现动画的效果
  - 使用原生的 `transition` 动画，设置属性为 `height` ，时间为用户设置的时间，默认的动画效果为`linear` ，动画效果可以提供给用户设置
- 其他的附加效果
  - 开始展开动画时候的事件
    - 在打开的时候，调用 `onOpening` 事件
  - 开始结束展开动画时候的事件
    - 在收起的时候，调用 `onClosing` 事件
  - 已展开的事件
    - 使用的元素上的 `onTransitionEnd` 事件，这时候判断当前状态是展开还是收起了，如果展开那就调用 `onOpen` ，如果结束就调用 `onClose` 事件

已写完，代码地址[react-collapsible](https://github.com/fedono/react-collapsible)

