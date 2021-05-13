学习知识的时候还是要告诉自己慢下来，搞懂才是重点。自己本来就求快，看书划过去就是，这样导致很多概念自己都只是浅尝辄止。



- 还是要自己去学会如何看源码的过程，养成自己的方法论

  >  多看下别人的方法

  - 带着问题去看

  - 从一个小点入口，把一条线缕清

  - 刚开始看肯定很多陌生的代码，先不用管细节，先把大致的逻辑捋一捋，然后整体的思路自己知道的差不多就行，然后看别人是如何分析的

  - 看不懂就看不懂，没多大事，把自己看懂的部分先记录下来，然后后续再接着看的时候，就能知道上次看的时候是什么思路了，这样就不用说是下次还得重新看

    > 记录是很重要的，光靠记忆是很容易忘记的



- bfc and dfc
- formily 的使用与原理，可视化编辑器的原理
- mobx 的原理
- webpack 的打包流程
  - 如何实现热更新



tayasui sketches



复杂度分析

时间复杂度：O(mn)，其中 m 和 n 分别是字符串text1 和  text2 的长度。二维数组 dp 有 m + 1 行和 n + 1 列，需要对 dp 中的每个元素进行计算
空间复杂度：O(mn)，其中 m 和 n 分别是字符串 text1 和 text2 的长度，创建了 m + 1 行和 n + 1 列的二维数组 dp 



给你很多形如 `[start, end]` 的闭区间，请你设计一个算法，**算出这些区间中最多有几个互不相交的区间**。

```java
int intervalSchedule(int[][] intvs) {}
```

举个例子，`intvs = [[1,3], [2,4], [3,6]]`，这些区间最多有 2 个区间互不相交，即 `[[1,3], [3,6]]`，你的算法应该返回 2。注意边界相同并不算相交。

这个问题在生活中的应用广泛，比如你今天有好几个活动，每个活动都可以用区间 `[start, end]` 表示开始和结束的时间，请问你今天**最多能参加几个活动呢？**显然你一个人不能同时参加两个活动，所以说这个问题就是求这些时间区间的最大不相交子集。



---

bilibili

- react-router 源码
- 拉勾的三百分钟的算法 
- 前端进阶
  - 
- 瓜娃
  - 





![image-20210320120056200](/Users/zengdeping/Library/Application Support/typora-user-images/image-20210320120056200.png)



![image-20210320122242790](/Users/zengdeping/Library/Application Support/typora-user-images/image-20210320122242790.png)



![image-20210320122305649](/Users/zengdeping/Library/Application Support/typora-user-images/image-20210320122305649.png)

![image-20210320122323275](/Users/zengdeping/Library/Application Support/typora-user-images/image-20210320122323275.png)

![image-20210320122520035](/Users/zengdeping/Library/Application Support/typora-user-images/image-20210320122520035.png)