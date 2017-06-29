---
title: 聊一下Node模块化
date: 2017-05-20 19:29:03
tags: [Node]
categories: Node
---


### 关于Node的模块化

在编译过程中，Node对javascript文件进行了包装。就像这样

```javascript
(function (exports, require, module, __filename, __dirname) {
  // 这里是你原本文件内容
  exports.a = 'xxxx' //这里我们改变了exports的值
})
```

包装之后的代码将会通过runInThisContext()这个方法执行，得到了一个function

然后传入exports, require, module, __filename, __dirname 给这个函数。这个函数将模块的exports返回。因此可以通过域外来访问到exports上的属性。

也正是因此我们在模块代码内将exports赋值，是无效的方法。直接赋值会改变引用，无法改变作用域外的值。我们可以通过

```javascript
module.exports = {
  a: 'xxxx',
}
```

这种方法，将一个对象赋值给module.exports 来达到效果，并不改变形参的引用。