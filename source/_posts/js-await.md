---
title: JavaScript异步编程大冒险 Async/Await
date: 2018-02-07 03:20:59
author: Dominic Ming
email: mxz96102@qq.com
tags:
  - JavaScript
  - ES6
---

### Async/Await 是什么?

Async/Await 也就是大家知道的异步函数，它是一个用来控制 JavaScript 异步流程的一个记号。而在很多现代浏览器上也曾实现过这样的设想。它的灵感来源于C# 和 F#，现在 Async/Await 在ES2017已经平稳着陆。

<!-- more -->

通常我们认为 `async function` 是一个能返回  `Promise` 的 `function` 。你也可以在 `async function` 使用 `await` 关键字。 `await` 关键字可以放在一个需要返回Promise的表达式前，所得到的值被从Promise里面剥离开，以便能用更直观的同步体验。我们来看一下实际的代码更直观。

```Javascript
// 这是一个简单的返回 Promise 函数
// 功能是在两秒以后 resolve("MESSAGE") .
function getMessage() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve("MESSAGE"), 2000);
  });
}
```

```
async function start() {
  const message = await getMessage();
  return `The message is: ${message}`;
}
```

```
start().then(msg => console.log(msg));
// "The message is: MESSAGE"
```

### 为什么要用 Async/Await?

Async/Await 提供了一个看起来相对同步的方法来执行异步代码。同时也提供了一种简洁而直观的方法来处理异步的错误，因为它实现了`try…catch` 标记，这是JavaScript里面最常见的一种同步模式。

在我们开始冒险之前，我们应该清楚，Async/Await 是建立在 [JavaScript Promises](https://medium.com/@BenDiuguid/asynchronous-adventures-in-javascript-promises-1e0da27a3b4) 上的，而且关于它的知识是很重要的。

### 关于记号

#### Async 函数

要创建一个 `async` 函数，一般就要把 `async` 关键字放在声明函数之前，就像这样：

```Javascript
async function fetchWrapper() {
  return fetch('/api/url/');
}

const fetchWrapper = async () => fetch('/api/url/');
```

```
const obj = {
  async fetchWrapper() {
    // ...
  }
}
```

#### Await 关键字

```Javascript
async function updateBlogPost(postId, modifiedPost) {
  const oldPost = await getPost(postId);
  const updatedPost = { ...oldPost, ...modifiedPost };
  const savedPost = await savePost(updatedPost);
  return savedPost;
}
```

在这里的 `await` 是用在其他返回 promise 的函数前。在第一行，oldPost被赋值为getPost执行resolve后返回的value。在下一行，我们使用了解构赋值来演示怎样把 oldPost 和 modifiedPost 合并。最终我们把 post 储存下来，返回了 savedPost 的结果。

### 示例 / FAQ

>  🖐️“到底怎么处理错误?”

这是一个好问题！当你使用 async/await 的时候，你也可以使用 `try...catch` 。在下面展示了，我们异步的 fetch 了一些东西，返回了某种错误，我们可以在catch里面拿到错误。

```Javascript
async function tryToFetch() {
  try {
    const response = await fetch('/api/data', options);
    return response.json();
  } catch(err) {
    console.log(`An error occured: ${err}`);
    // 比起返回一个错误
    // 我们可以返回一个空的data
    return { data: [] };
  }
}
```

```Javascript
tryToFetch().then(data => console.log(data));
```

>  🖐️ ️“我还是不知道为什么 async/await 比 callbacks/promises 好.”

很高兴你问了这个问题，这里有一个例子可以说明不同。我们这里只是想要异步的 fetch 一些数据，然后得到数据后，简单的返回一些经过处理的data，如果有错误，我们简单的只是想要返回一个对象。

```Javascript
// 我们这里有 fetchSomeDataCB, 和 processSomeDataCB
// NOTE: CB 代表 callback

function doWork(callback) {
  fetchSomeDataCB((err, fetchedData) => {
    if(err) {
      callback(null, [])
    }

    processSomeDataCB(fetchedData, (err2, processedData) => {
      if(err2) {
        callback(null, []);
      }

      // return the processedData outside of doWork
      callback(null, processedData);
    });
  });
}

doWork((err, processedData) => console.log(processedData));
```

```Javascript
// 我们这里有 fetchSomeDataP, 和 processSomeDataP
// NOTE: P 意味着这个函数返回一个 Promise

function doWorkP() {
  return fetchSomeDataP()
    .then(fetchedData => processSomeDataP(fetchedData))
    .catch(err => []);
}

doWorkP().then(processedData => console.log(processedData));
```

```Javascript
async function doWork() {
  try {
    const fetchedData = await fetchSomeDataP();
    return processSomeDataP(fetchedData);
  } catch(err) {
    return [];
  }
}

doWork().then(processedData => console.log(processedData));
```

Callback vs Promise vs Async/Await

> 🖐️“他的并发性如何”

当我们需要有顺序的做一些事情，我们通常用await一个一个声明所有的步骤。在这之前为了理解并发，我们必须使用`Promise.all`。如果我们现在有三个异步动作需要平行执行，在加上await之前，我们需要让所有的Promise先开始。

```Javascript
// 这不是解决方法，他们会逐个执行
async function sequential() {
  const output1 = await task1();
  const output2 = await task2();
  const output3 = await task3();
  return combineEverything(output1, output2, output3);
}
```

因为上述代码只是依次的执行了三个任务，而没有并发的执行，后一个会依赖前一个执行完成。所以我们要改造成`Promise.all` 的方式。

```Javascript
// 这就可以并发的执行
async function parallel() {
  const promises = [
    task1(),
    task2(),
    task3(),
  ];
  const [output1, output2, output 3] = await Promise.all(promises);
);
  return combineEverything(output1, output2, output3);
}
```

在这个例子上，我们首先执行了3个异步的任务，之后把Promise都储存进了一个array。我们使用 `Promise.all` 来完成来全部并发结果的收集。

#### 另外的一些提示

- 你很容易会忘记每次你 `await` 一些代码, 你需要声明这个函数是一个 `async function`。
- 当你使用 `await` 时候，它值暂停了所涉及的 `async function` 。 换句话说，下面的代码会在其他东西log之前log `'wanna race?'` 。

```Javascript
const timeoutP = async (s) => new Promise((resolve, reject) => {
  setTimeout(() => resolve(s*1000), s*1000)
});
```

```Javascript
[1, 2, 3].forEach(async function(time) {
  const ms = await timeoutP(time);
  console.log(`This took ${ms} milliseconds`);
});
```

```
console.log('wanna race?');
```

当你的第一个await 的 promise在主线程上返回执行了结果，在forEach外面的log不会被阻塞。

#### 浏览器支持

看一看这张[浏览器支持表](http://caniuse.com/#feat=async-functions)。

#### Node 支持

 `node 7.6.0` 以及以上版本支持 Async/Await !

> 作者：[Benjamin Diuguid](https://medium.com/@BenDiuguid?source=post_header_lockup)
>
> 原文：[Asynchronous Adventures in JavaScript: Async/Await](https://medium.com/dailyjs/asynchronous-adventures-in-javascript-async-await-bd2e62f37ffd)
>
> 翻译：Dominic Ming