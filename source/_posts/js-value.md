---
author: Dominc Ming
email: mxz96102@qq.com
aliasUrl: https://github.com/mxz96102
aliasAvatar: https://avatars3.githubusercontent.com/u/15213473?v=4
title: 回归本源：JavaScript 之中的值和引用
date: 2018-02-07 03:33:30
tags:
  - JavaScript
---

> 阅读花费时间：2分钟 
>
> 这是一个非常简短的值和引用的解释。

首先，对于每一个JavaScript开发者来说，值(value)和引用(reference)的定义，一般是从一些bug被引出的，而且在面试中也经常会被问到。这篇文章中也将简单的涵盖这些基本概念。

<!-- more -->

别先急着往下滑，你知道下面这些代码会有什么结果吗？

```javascript
console.log([10] === [10]);
```

```javascript
var oldArray = [];
var object = {};

object.newArray = oldArray;
oldArray.push(10);

console.log(object.newArray === oldArray);
```

第一个例子是 false 而第二个例子是 true。你答对了么，我们来看看这是为什么：



在 JavaScript 中，有一些类型和值是直接复制了引用，分别是下面的这些：

**原始值 (复制值)**

- null
- undefined
- Number
- String
- Boolean

**对象 (复制引用)**

- Object
- Array
- Function

### 原始值

```javascript
var a = 5;

var b = a;

a = 10;

console.log(a); // 10
console.log(b); // 5

// 这也同样适用于 string, boolean, null, undefined
```

当我们把这些初始值赋给了变量的时候， 我们 **复制了值**.

### 对象

现在就是比较迷惑人的部分了

```javascript
var a = {};
var b = a;

a.a = 1;

console.log(a); // {a: 1}
console.log(b); // {a: 1}
```

对于 **数组** 也是一样的

```javascript
var a = [];
var b = a;

a.push(1);

console.log(a); // [1]
console.log(b); // [1]
console.log(a === b); // true
```

当我们把没有初始值的对象赋给变量时，我们只是复制了他的引用。如此可以想象，声明变量`a`时，我们在内存里面创造了一个新的地址，在声明`b`的时候，`b`就直接指向了那个地址，所以我们更新这个地址的内容的时候，`a` 和 `b`有着相同的值。

```
var a = [];     # 地址 #001 -> []
                # 变量 a -> #001

var b = a;      # 变量 b -> #001

a.push(1);      # 地址 #001 -> [1]

变量		| 地址 	| 值
a 		 | #011    | [1]
b        | #011    | [1]
```



#### 关于 [10] === [10] 的例子

当我们比较对象的时候，相等运算符（===）会检查他们是否指向相同的地址。所以如果`[10]`和`[10]`是两个不同的数组，结果就会返回`false`。当你想要对比两个对象或者数组是不是相同的方法很简单，但是这样的方法也很有限

```javascript
JSON.stringify(a) === JSON.stringify(b)
```

尽管这样的方法在数组和对象内部顺序不一样的时候，还是会出错。如果你想要更健壮的解决方法的话，参考[lodash _.isEqual() method](https://lodash.com/docs/4.17.4#isEqual)，或者通过这个[stackoverflow 回答](https://stackoverflow.com/questions/1068834/object-comparison-in-javascript) 来自己实现一个解决方案。

> 作者：[Miro Koczka](https://medium.com/@mirokoczka)
>
> 原文：[Back to roots: JavaScript Value vs Reference](https://medium.com/dailyjs/back-to-roots-javascript-value-vs-reference-8fb69d587a18)
>
> 翻译：Dominic Ming
