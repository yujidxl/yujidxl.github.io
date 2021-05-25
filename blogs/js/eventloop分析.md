---
title: event loop分析
date: 2020-09-04
tags:
 - js
categories: 
 - js
---
## javascript的单线程
1. 首先我们思考一个问题，我们或多或少听说过javascript是单线程的。但是你知道它为什么会被设计成单线程的吗？那么让我们来了解一下为什么javascript会被设计成一门单线程的语言？
>JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？
后来为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。
2. 那么单线程是怎么制定事件的执行的先后顺序的呢？
>我们知道单线程意味着前面一个任务执行完遇到一个需要等待的任务，我们就必须一直等着这个任务执行了，但是就目前而言我们接触的执行步骤等待，基本是不存在的现象，除非出现程序错误。这是因为语言的设计者将需要等待执行的任务和不需要等待即可执行的任务区分开了，在遇到等待任务的时候，会把等待的任务抛在一旁，继续执行下面的任务，直到任务执行完，再回过头来执行等待的任务。
3. 我们前端可以将当前任务分为几种类别呢？
>同步任务和异步任务。说到同步任务和异步任务就不得不提到主线程和任务队列这两个东西了。
4. 什么是同步任务什么是异步任务？
>同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

如下图所示：

![avatar](/img1.jpeg)
<br />释义： 只要主线程空了，就会去读取"任务队列"，这就是JavaScript的运行机制。这个过程会不断重复。
需要注意的是，setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout()指定的时间执行。（但是估计一般我们的业务也碰不上这种情况）

5. 描述：
`event loop是一个执行模型，在不同的地方有不同的实现。浏览器和NodeJS（这个后续看一下，node没怎么使用过）基于不同的技术实现了各自的Event Loop。`

## 常见宏队列和微队列的异步任务
1. `宏队列，macrotask，也叫tasks` 一些异步任务的回调会依次进入macro task queue，等待后续被调用，这些异步任务包括：
- setTimeout
- setInterval
- requestAnimationFrame
- I/O
- UI rendering
2. `微队列，microtask，也叫jobs` 另一些异步任务的回调会依次进入micro task queue，等待后续被调用，这些异步任务包括：
- Promise
- Object.observe
- MutationObserver
::: tip
3. 浏览器的event loop：
:::
![avator](/img2.png)
#### 归纳一下：
- 宏队列macrotask一次只从队列中取一个任务执行，执行完后就去执行微任务队列中的任务；
- 微任务队列中所有的任务都会被依次取出来执行，直到microtask queue为空；
- 图中没有画UI rendering的节点，因为这个是由浏览器自行判断决定的，但是只要执行UI rendering，它的节点是在执行完所有的microtask之后，下一个macrotask之前，紧跟着执行UI render
::: tip
一起来看一下这段代码的执行顺序：
:::
``` js
console.log(1);
setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
    Promise.resolve().then(() => {
      console.log(11);
    });
  });
  Promise.resolve().then(() => {
    console.log(8);
  });
});

new Promise((resolve, reject) => {
  console.log(4);
  resolve(5);
}).then((data) => {
  console.log(data);
});

setTimeout(() => {
  console.log(6);
  Promise.resolve().then(() => {
    console.log(9);
  });
  Promise.resolve().then(() => {
    console.log(10);
  });
});
console.log(7);
```
`执行顺序为:` 1 | 4 | 7 | 5 | 2 | 3 | 8 | 11 | 6 | 9 | 10
::: tip
再看一段代码的执行顺序
:::
``` js
Promise.resolve().then(() => {
  console.log('mm');
  Promise.resolve()
  .then(() => {
    console.log('xx');
  })
  .then(() => {
    console.log('yy');
  });
}).then(() => {
  console.log('nn');
});
```
`执行顺序为：` mm | xx | nn | yy
<br />`结语：` 上述流程我们可以理解为每一个异步的宏任务执行完，都会执行当前宏任务后的微任务队列，直到把当前微任务队列完全执行完，再执行下一个异步宏任务。在执行微队列microtask queue中任务的时候，如果又产生了microtask，那么会继续添加到队列的末尾，也会在这个周期执行，直到microtask queue为空停止。
#### 了解更多 :tada:
[带你彻底理解event loop](https://segmentfault.com/a/1190000016278115)
<br />[阮一峰的再谈event loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
