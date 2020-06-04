## 第一种方法，使用 `while`

```js
function fill(arr, v, s, e) {
    let n = e - s;
    let fillArr = [];
    while (n) {
        fillArr.push(v);
        n--
    }
    arr.splice(s, e - s, ...fillArr);
    return arr;
}

let res = fill([1, 2, 3, 4], 5, 1, 3);
console.log(res); // [ 1, 5, 5, 4 ]
```

但一般出这种题都是为了考察递归，所以还是得遵从面试官的思路（一般都会让你不要用循环），把 `while` 换成递归

## 第二种方法，使用递归

```js
function fill(arr, v, s, e) {
    function add(n, m) {
        n--;
        if (n) {
            return [m].concat(add(n, m))
        } else {
            return [m];
        }
    }
    let len = e - s;
    arr.splice(s, len, ...add(len, v))
    return arr;
}
```

## 第三种，使用原生的`Array.from` 方法

```js
Array.prototype._fill = function (pad, s, e) {
    return Array.from({length: this.length}, (val, index) => {
        if (index >= s && index < e) {
            return pad;
        }
        return this[index];
    });
}
```

## 注意

- fill 是会改变当前的 数组的，而不是产生一个新的数组

## 参考

- [如何使用递归写一个函数 实现js的Array.prototype.fill()方法的效果](https://segmentfault.com/q/1010000015155983)

