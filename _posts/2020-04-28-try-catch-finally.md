记个点

1. `catch` 和 `finally`  中都有 `return ` 的话，那么最终会返回 `finally` 的 `return` 
2. `catch` 和 `finally` 中的 `return` 之后的语句都不会执行
3. `catch` 中有 `return` ，但是 `finally` 中没有，会执行完 `finally` 中的语句，再返回 `catch` 中的 `return` 

```js
var count = 0;
function countUp() {
  try {
    return count;
  } finally {
    count++;
  }
}

countUp()
// 0
count
// 1
```

上面代码说明，`return`语句里面的`count`的值，是在`finally`代码块运行之前就获取了。

4. 执行顺序

   ```js
   function f() {
     try {
       console.log(0);
       throw 'bug';
     } catch(e) {
       console.log(1);
       return true; // 这句原本会延迟到 finally 代码块结束再执行
       console.log(2); // 不会运行
     } finally {
       console.log(3);
       return false; // 这句会覆盖掉前面那句 return
       console.log(4); // 不会运行
     }
   
     console.log(5); // 不会运行
   }
   
   var result = f();
   // 0
   // 1
   // 3
   
   result
   // false
   ```



## 参考

1. [错误处理机制](https://wangdoc.com/javascript/features/error.html)
2. [winter-重学前端 - 19-JavaScript执行（四）：try里面放return，finally还会执行吗？]()