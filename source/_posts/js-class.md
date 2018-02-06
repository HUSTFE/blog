---
title: Javascript 之中的 class/构造函数/工厂函数
date: 2018-02-07 03:17:48
author: Dominic Ming
email: mxz96102@qq.com
tags:
    - JavaScript
---

到了ES6时代，我们创建对象的手段又增加了，在不同的场景下我们可以选择不同的方法来建立。现在就主要有三种方法来构建对象，class关键字，构造函数，工厂函数。他们都是创建对象的手段，但是却又有不同的地方，平时开发时，也需要针对这不同来选择。

<!-- more -->

首先我们来看一下，这三种方法是怎样的

```Javascript
// class 关键字，ES6新特性
class ClassCar {
  drive () {
    console.log('Vroom!');
  }
}

const car1 = new ClassCar();
console.log(car1.drive());


// 构造函数
function ConstructorCar () {}

ConstructorCar.prototype.drive = function () {
  console.log('Vroom!');
};

const car2 = new ConstructorCar();
console.log(car2.drive());


// 工厂函数
const proto = {
  drive () {
    console.log('Vroom!');
  }
};

function factoryCar () {
  return Object.create(proto);
}

const car3 = factoryCar();
console.log(car3.drive());
```

这些方法都是基于原型的创建，而且都支持在构造时函数中私有变量的实现。换句话来说，这些函数拥有着大部分相同的特性，甚至在很多场景下，他们是等价的。

> 在 Javascript 中，每一个函数都能返回一个新的对象。当它不是构造函数或者类的时候，它就被称作工厂函数。

ES6的类其实是构造函数的语法糖（至少现阶段是这样实行的），所以接下来讨论的所有内容都适用于构造函数的也适用于ES6类：

```Javascript
class Foo {}
console.log(typeof Foo); // function
```

### 构造函数和ES6类的好处

- 大部分的书会教你去用类和构造函数
- ‘`this`’ 是指向新的这个对象的。
- 一些人喜欢 `new` 关键字的可读性
- 也许还会有一些很小的细节方面的差别，但是如果在开发过程中没有问题的话，也不用太担心。

### 构造函数和ES6类的坏处

#### 1. 你需要 new 关键字

到了ES6，构造函数和类都需要带 `new` 关键字。

```
function Foo() {
  if (!(this instanceof Foo)) { return new Foo(); }
}
```

在ES6中，如果你尝试调用类函数没有 `new` 关键字就会抛出一个任务。如果你要个不用 `new` 关键字的话，就只能使用工厂函数把它包起来。

#### 2. 实例化过程中的细节暴露给了外界API

所有的调用都紧紧的关联到了构造器的实现上，如果你需要自己在构造过程中动一些手脚，那就是一个非常麻烦的事情了。

#### 3. 构造器没有遵守 Open / Closed 法则

因为`new`关键字的细节处理，构造器违反 Open / Closed 法则：API应该开放拓展，避免修改。

我曾经质疑过，类和工厂函数是那么的相似，把类函数升级为一个工厂函数也不会有什么影响，不过在JavaScript里面，的确有影响。

如果你开始写着构造函数或者类，但是写着写着，你发现需要工厂函数的灵活性，这个时候你不能简单的就改改简单改改函数一走了之。

不幸的是，你是个JavaScript程序员，构造器改造成工厂函数是一个大手术：

```Javascript
// 原来的实现：

// class Car {
//   drive () {
//     console.log('Vroom!');
//   }
// }

// const AutoMaker = { Car };

// 工厂函数改变的实现：
const AutoMaker = {
  Car (bundle) {
    return Object.create(this.bundle[bundle]);
  },

  bundle: {
    premium: {
      drive () {
        console.log('Vrooom!');
      },
      getOptions: function () {
        return ['leather', 'wood', 'pearl'];
      }
    }
  }
};

// 期望中的用法是：
const newCar = AutoMaker.Car('premium');
newCar.drive(); // 'Vrooom!'

// 但是因为他是一个库
// 许多地方依然这样用:
const oldCar = new AutoMaker.Car();

// 如此就会导致:
// TypeError: Cannot read property 'undefined' of
// undefined at new AutoMaker.Car
```

在上面例子里面，我们从一个类开始，最后把它改成来一个可以根据特定的原型来创建对象的工厂函数，这样的函数可以广泛应用于接口抽象和特殊需求定制。

#### 4. 使用构造器让 `instanceof` 有可乘之机

构造函数和工厂函数的不同就是`instanceof`操作符，很多人使用`instanceof`来确保自己代码的正确性。但是说实话，这是有很大问题的，建议避免`instanceof`的使用。

> `instanceof` 会说谎。

```
// instanceof 是一个原型链检查
// 不是一个类型检查

// 这意味着这个检查是取决于执行上下文的,
// 当原型被动态的重新关联,
// 你就会得到这样令人费解的情况

function foo() {}
const bar = { a: 'a'};

foo.prototype = bar;

// bar是一个foo的实例吗，显示不是
console.log(bar instanceof foo); // false

// 上面我们看到了，他的确不是一个foo实例
// baz 显然也不是一个foo的实例，对吧?
const baz = Object.create(bar);

// ...不对.
console.log(baz instanceof foo); // true. oops.
```

`instanceof`并不会像其他强类型语言那样做检查，他只是检查了原型链上的对象。

在一些执行上下文中，他就会失效，比如你改变了`Constructor.prototype` 的时候。

又比如你开始些的是一个构造函数或者类，之后你又将它拓展为一个另一个对象，就像上面改写成工厂函数的情况。这时候`instanceof`也会有问题。

总而言之，`instanceof`是另一个构造函数和工厂函数呼唤的大改变。

### 用类的好处

- 一个方便的，自包含的关键字
- 一个唯一的权威性方法在JavaScript来实现类。
- 对于其他有class的语言开发经验的开发者有很好的体验。

### 用类的坏处

构造器所有的坏处, 加上:

- 使用 `extends` 关键字创建一个有问题的类，对于用户是一个很大的诱惑。

类的层级继承会造成很多有名的问题，包括 fragile base class（基础类会因为继承被破坏），gorilla banana problem（对象混杂着复杂的上下文环境），duplication by necessity（类在继承多样化时需要时时修改）等等。

虽然其他两种方法也有可能让你陷入这些问题，但是在使用 `extend` 关键字的时候，环境使然，就会把你引导上这条路。换句话说，他引导你向着一个不灵活的关系编写代码，而不是更有复用性的代码。

### 使用工厂函数的好处

工厂函数比起类和构造函数都更加的灵活，也不会把人引向错误的道路。也不会让你陷入深深的继承链中。你可以使用很多手段来模拟继承

#### 1. 用任意的原型返回任意的对象

举个例子，你可以通过同一个实现来创建不同的实例，一个媒体播放器可以针对不同的媒体格式来创建实例，使用不同的API，或者一个事件库可以是针对DOM时间的或者ws事件。

工厂函数也可以通过执行上下文来实例化对象，可以从对象池中得到好处，也可以更加灵活的继承模型。

#### 2. 没有复杂重构的担忧

你永远不会有把工厂函数转换成构造函数这样的需求，所以重构也没必要。

#### 3. 没有 `new`

你不用new关键字来新建对象，自己可以掌握这个过程。

#### 4. 标准的`this` 行为

this 就是你熟悉的哪个this，你可以用它来获取父对象。举个例子来说，在`player.create()` 中，this指向的是player，也可以通过call和apply来绑定其他this。

#### 5. 没有 `instanceof`  的烦恼

#### 6. 有些人喜欢直接不带new的写法的可读直观性。

### 工厂函数的坏处

- 并没有自动的处理原型，工厂函数原型不会波及原型链。
- `this` 并没有自动指向工厂函数里的新对象。
- 也许还会有一些很小的细节方面的差别，但是如果在开发过程中没有问题的话，也不用太担心。

### 结论

在我看来，类也许是一个方便的关键字，但是也不能掩饰他会把毫无防备的用户引向继承深坑。另一个风险在于未来的你想要使用工厂函数的可能性，你要做非常大的改变。

如果你是在一个比较大的团队协作里面，如果要修改一个公共的API，你可能干扰到你并不能接触到的代码，所以你不能对改装函数的影响视而不见。

工厂模式很棒的一个地方在于，他不仅仅更加强大，更加灵活，也可以鼓励整个队伍来让API更加简单，安全，轻便。

> 翻译自：<https://medium.com/javascript-scene/javascript-factory-functions-vs-constructor-functions-vs-classes-2f22ceddf33e>
>
> 作者：[Eric Elliott](https://medium.com/@_ericelliott?source=post_header_lockup)
>
> 翻译：Dominic Ming
