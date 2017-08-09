原问题：[Commonly accepted best practices around code organization in JavaScript](http://stackoverflow.com/questions/247209/commonly-accepted-best-practices-around-code-organization-in-javascript)

提问者的问题是：如何组织日益庞大的JavaScript代码，使其可维护性更高？

## JavaScript可维护性
要提高可维护性，首先要做到：抽象，封装，解耦。

想要实现这个目的，有两大基础：

* Modular Pattern - 封装
  * 普通业务逻辑
* Publisher/Subscriber Pattern - 监听
  * 事件管理（DOM事件、timer事件、Ajax异步事件）

目前标准的JavaScript结构是MVC的，像Backbone, Angular等，都是很好的参考示例。

## 封装
封装对于JavaScript来说，尤其重要，因为可以解决【全局变量污染】的问题。

### IIFE
JavaScript的封装，主要就是利用JavaScript函数作用域的特性，使用[Immediately Invoked Function Expression (IIFE)](http://stackoverflow.com/questions/2421911/what-is-the-purpose-of-wrapping-whole-javascript-files-in-anonymous-functions-li/26738969#26738969)的形式来实现。

```
(function() {
      // Your code goes here
}());
```

由于函数立即执行，结束之后函数的context就自动销毁了。因此函数内部定义的变量，在函数外部是看不到的。这就实现了封装效果。

### NameSpace
JavaScript其实是有对象的，JSON就是对象。因此，NameSpace很简单，一个JSON对象就自带NameSpace。

```
myAppNamespace = {
    attr1: "val1",
    attr2: "val2"
};
```

只能通过myAppNamespace才能访问到attr1和attr2，这就有了NameSpace效果。

## Modular Pattern
在以上两点的基础上，再利用一下closure特性，从而能实现真正的对象封装的效果。

```
var modularpattern = (function() {
    // your module code goes here
    var sum = 0 ;

    return {
        add:function() {
            sum = sum + 1;
            return sum;
        },
        reset:function() {
            return sum = 0;
        }
    }
}());
alert(modularpattern.add());    // alerts: 1
alert(modularpattern.add());    // alerts: 2
alert(modularpattern.reset());  // alerts: 0
```

## Publisher/Subscriber Pattern
无论是DOM事件监听，还是浏览器的timer监听，还是Ajax异步监听，都是这种模式。因为JavaScript天生就是异步的，它的最初目的是响应用户操作页面时对应触发的DOM事件。

### 对象间通信
监听模式的本质其实是一种对象间通信的实现方式。

对于Java这种正统OOP语言，一般是使用接口实现类之间的通信。但这种方式依赖于多态，对javascript不适用。因此只能考虑传统的方式：保留对象引用。 但是如果直接保留类对象的引用，未免太重了。一是增加了代码耦合度，维护不方便；二是影响内存回收，进而影响程序性能。

我们分析一下通信这个问题，其实包含两方面：

1. 通信时机；
2. 数据传递；

我们可以把这部分内容提取出来，定义一个中间代理，专门负责类与类之间的通信。具体实现上就是把通信封装成事件，然后定义一个全局的事件管理类，采用Observer模式，使用队列来注册/触发事件。

在类实例这边，可以保存这个全局的队列的事件对象的引用（或者直接继承事件管理类）。这样就可以在类实例中注册或者响应事件，从而达到类与类之间通信的目的。

基本结构如图所示：

![communication-between-objects](https://farm4.staticflickr.com/3859/14708403220_5923af3ba5.jpg)

### 监听模式的实现
Backbone.Events就是一个很好的示例。

它包含了this._events数组，用作事件集合，也定义了on()/off()/trigger()等事件操作函数。Model/Collection/View都继承了Events。因此在Model/Collection/View上都能直接调用on()/off()/trigger()等方法，实现对象上对数据的异步通信。

Backbone.Model的定义

```
_.extend(Model.prototype, Events, {

// A hash of attributes whose current and previous value differ.
changed: null,

// The value returned during the last failed validation.
validationError: null,

// The default name for the JSON `id` attribute is `"id"`. MongoDB and
// CouchDB users may want to set this to `"_id"`.
idAttribute: 'id',

...
}
```

## 更多
实现了以上两点，JavaScript基本就是模块化、低耦合的了。

再进一步的结构优化，就是与具体的业务逻辑相关的，主要是：

* 抽离公共代码，DRY
* 增强对event handler的管理
  * 给event handler命名
  * 退出作用域的时候，detach event handler

目前为止，我大致就是认识到这个地步。以后有新的体会和认识的时候，再来更新。