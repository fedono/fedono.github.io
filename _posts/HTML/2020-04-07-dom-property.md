---
layout: post
title:  "DOM Property"
author: "fedono"
---

# DOM 的属性

- clientLeft 就是 borderLeft 的值
- offsetLeft 就是距离父元素左边的距离

- pageXOffset 是  `scrollX ` 的别名

- scrollHeight 是包括滚动条的所有内容，没有滚动条时，scrollHeight值与元素 的 [clientHeight](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/clientHeight)相同。包括元素的padding，但不包括元素的border和margin。scrollHeight也包括 [`::before`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before) 和 [`::after`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::after)这样的伪元素。

  - 如果元素滚动到底，下面等式返回true，没有则返回false.

    ```js
    element.scrollHeight - element.scrollTop === element.clientHeight
    ```

    这个可以作用于当用户阅读协议，滚动到最后时，才允许勾选『已阅读』的选项

