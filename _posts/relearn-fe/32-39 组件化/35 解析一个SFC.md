VUE 中单个文件是 single file component 也就是 SFC，里面含有 template / script / css，那么这节课就是实现一个 loader 在解析.view文件，通过获取到 templete / script  / css ，然后可以实现组件的渲染

之前的课程有做解析HTML的标签，这节课再加上解析 script 的过程，然后就可以获取到里面的内容了

因为 vue 中在写template 都是以 `<template></template>`  标签闭合的，那么就可以解析到这个标签内的左右子内容后，渲染

那么`<script></script>` 也是解析到这个闭合标签内的所有内容，然后进行解析里面的 script 的

因为在上一节课中实现了一个jsx 解析的方式，这时候就可以在获取到 template 的时候，可以渲染对应的组件

>  这节课确实有助于理解 vue 是怎么做的文件解析的工作的

>  还是得在 github 中搜一下重学前端的学员有没有写出这些代码，然后对照着看一下，跑一下，有一个地方没跟着，后续就真的不好跟上了