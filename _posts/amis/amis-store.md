amis 在 store  中定义的数据为

```
data: types.optional(types.frozen(), {})
```

那么要获取到 data 然后再往里面添加数据的时候需要注意，这里的 data 通过 self.data 获取到之后，不能直接添加，这样会有问题的，

```
Uncaught TypeError: Cannot add property 3, object is not extensible
```

也就是这种的 data 要先要复制一遍，然后添加值，然后再设置到 self.data 中

   

```
"initApi": "http://127.0.0.1:3002/detail/${id}",
"api": "post:http://127.0.0.1:3002/edit/${id}",
```