清除不知道为什么会出现的滚动条

当前元素在页面出现了滚动条，但是总也找不到原因，那就在当前元素的父元素上加上 `overflow:hidden` ，如果还是有，那就再往父元素的父元素上加，一直往上加。就能知道是哪个元素的问题了