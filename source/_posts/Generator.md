---
title: 聊一下ES6里面的Generator函数
date: 2017-05-20 19:21:39
tags: [Javascript, 随手记]
categories: Javascript
---



### Generator函数介绍

首先让我们定义一个Generator函数
{% codeblock lang:Javascript %}
const gen =  function* (a) {
  const b = yield a;
  return b;
}
{% endcodeblock %}

ok，定义完成。

<!--more-->

我们开始运行吧。
{% codeblock lang:Javascript %}
    // 就用上面的那个函数
    const n = gen(1);   // 这里控制台无输出
    console.log(n.next())  // 输出 { value: 1, done: false }
    console.log(n.next(20)) // 输出了{ value: 20, done: true }
{% endcodeblock %}

最开始我们调用gen()的时候返回了一个指针(即遍历器)

当我们第一次调用n.next()方法时候，才正式开始运行，因此这里输出了1。 然后运行到  yield a 那里就停止了，a的值为1。 

因此输出了{ value: 1, done: false } 

value表示的是yield后面的表达式返回的值，所以这里是1。 done表示这个Generator函数是否执行结束。这里表示还未执行结束。

好的我们继续。

第二次调用了n.next(20) 输出了{ value: 20, done: true }。

done: true 表示执行结束

这里的value表示的是这个函数的返回值。 那么问题来了为什么这里是20呢？？？？

因为我们第二次执行的时候用的是 n.next(20)。  next()方法接受参数，这个参数用来替换上个任务的结果。本来上个任务的结果应该是1（因为a = 1）。但是这里替换掉了，并且赋值给了b，因此这里b是2.

好的。关于Generator的函数简单介绍就在这里。下面是进阶玩法。

### Generator函数进阶玩法

#### CO模块原理

先来看一段使用到了co模块的代码
{% codeblock lang:Javascript %}
    const co = require('co');
    
    const gen = function* (){
      const f1 = yield readFile('A');
      const f2 = yield readFile('B');
      console.log(f1.toString());
      console.log(f2.toString());
    };
    
    co(gen)
{% endcodeblock %}

对于Javascript有一定了解的同学应该都知道，Javascript有一个特色就是异步回调。这种异步回调的问题就是代码会难看，然后也有回调地狱这种坑。

co模块的作用就是让Generator函数里的代码同步执行。

使用co模块。Generator函数的yield命令后面，只能是Thunk函数或Promise对象。

我这里就用Thunk函数举一个例子来探讨一下co的原理。Promise对象实现的方法，大家就自己去尝试。

首先，我先来自己实现一个这个readFile函数
{% codeblock lang:Javascript %}
    const fs = require('fs');
    const readFile = (...args) => {
      return function (callback) {
        fs.readFile(...args, callback); 
      }
    };
    
    const gen = function* (){
      const f1 = yield readFile('A');
      const f2 = yield readFile('B');
      console.log(f1.toString());
      console.log(f2.toString());
    };
{% endcodeblock %}

ok,  就这样。

然后，让我们跟着这个流程走一遍。

首先
{% codeblock lang:Javascript %}
    const n = gen()
    const result1 = n.next() // 这里运行到readFile('A')停止，返回了一个函数.
    result1.value((err, data) => {
      // 这里自己处理错误啊，我就不处理了。
      n.next(data); // 就这样，我们读取到的文件A的值给传给了上面的f1。
      // 剩下的步骤我这里就不写。
    })
{% endcodeblock %}

那么问题来了，这样是不是一个任务一个任务写，是不是太麻烦了。

好好好，我们继续改进。

我们可以写一个run函数

{% codeblock lang:Javascript %}
    const run = (func) => { // 接受一个Generator函数
      const gen = func()
      
      function next(err, data) {
        //这里自己处理错误啊= =
        const result = gen.next(data);
        if (result.done) return;
        result.value(next);
      }
      
      next()
    }
    
    run(gen); // ok 这里我们用run函数跑一下。
{% endcodeblock %}
ok， 就到这儿啦。


