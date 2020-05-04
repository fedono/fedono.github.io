Scheduler 的整体流程概览

调度过程中的各种全局变量一览

构建任务调度的概念



scheduleWork

- 找到更新对应的 FiberRoot 节点
- 如果符合条件重置 stack
- 如果符合条件就请求工作调度



requestWork

- 加入到 root 调度队列
- 判断是否批量更新
- 根据 expirationTime 判断调度类型



## batchUpdates

是在 `react-dom` 包中

有个按钮直接点击更新 count + 1，函数为 handleClick

```js
function handleClick() {
  this.setState({
    count: this.state.count++
  }) ;
  
  
}

function numberAdd() {
  this.setState({
    count: this.state.count++
  }) ;
  console.log(this.state.count);
  this.setState({
    count: this.state.count++
  }) ;
  console.log(this.state.count);
  this.setState({
    count: this.state.count++
  }) ;
}
```



## performance

- 是否有 deadline 的区分
- 循环渲染 root 的条件
- 超过时间片的处理

