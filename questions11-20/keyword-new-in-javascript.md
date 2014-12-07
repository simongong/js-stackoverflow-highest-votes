原问题：[What is the 'new' keyword in JavaScript?](http://stackoverflow.com/questions/1646698/what-is-the-new-keyword-in-javascript)

通常，JavaScript初学者在看过`new`关键字的讲解之后会陷入迷惑之中。因为JavaScript不是一个OOP语言，而传统的`new`关键字是用来创建一个**对象**的。接着，当看到很多书上讲到`prototype`和`prototype.constructor`的时候，更迷惑了：这个`constructor`是几个意思？

现在由`new`关键字为契机，说一下JavaScript中的对象和OOP。

## 内存和new
虽然说JavaScript不是面向对象的编程语言，但程序总归还是在内存里运行的。不管编程语言有没有对象的概念，最终还是要跟内存打交道。

那么问题来了：一个JavaScript程序在内存中运行的时候，内存是怎样一种状况呢？

不走得太深，我们规定场景为：浏览器打开一个web页面，该页面中的JavaScript程序的运行情况。

一般情况下，内存中会是以下场景：

* window对象，全局的
* 栈，函数调用时使用
* 堆，使用`new`关键字申请

那么答案来了：`new`关键字是用来申请内存的。

## 对象和new
又重复一遍：JavaScript不是面向对象的编程语言。这句话中的**面向对象**指的是：封装、继承、多态。实际上这句话是废话，没有任何意义，建议以后忘掉它。

实际上JavaScript中处处都是对象——JSON对象。Object/Attribute, Function/Method的实例通通都是一个JSON对象。

与其他编程语言一样，使用`new`关键字可以创建一个新对象，也就是一个Object的实例，一个JSON实例。

实例展示：[JSFiddle](http://jsfiddle.net/simongong/exjd51f9/1/)

```
function Foo(){
  this.bar = 'foo bar';
}

/** 在内存创建了一个新的JSON对象
*{
*  bar: 'foo bar',
*  __proto__: {
*    constructor: {name: 'Foo', ...},
*    __proto__
*  }
*}
**/
var foo = new Foo();

console.log(JSON.stringify(foo));  // 输出: {"bar":"foo bar"}

// 输出: bar foo bar
for(var propName in foo) {
    propValue = foo[propName]
    console.log(propName,propValue);
}
```

## 深入JavaScript对象
其实没什么好深入的，就是在MCMAScript中对这个JSON对象有几个特别的规定：[ECMAScript- Object](http://www.ecma-international.org/ecma-262/5.1/#sec-15.2)

* prototype属性
* constructor属性
* hasOwnProperty(property)方法
* propertyIsEnumerable(property)方法

这几个特别属性和方法，请参考文档中的说明。其实这几个规定的目的，也是为了完善创建出来的JSON对象的属性列表，使其具有**任意扩展、属性继承**的效果。

使用prototype来任意扩展可访问的JSON属性，看一个例子：[JSFiddle](http://jsfiddle.net/simongong/exjd51f9/2/)

```
function Foo(){
  this.bar = 'foo bar';
}
Foo.prototype.b = 'foo bar b';  // 给Foo.prototype属性赋值

/** 在内存创建了一个新的JSON对象
*{
*  bar: 'foo bar',
*  __proto__: {
*    b: 'foo bar b';
*    constructor: {name: 'Foo', ...},
*    __proto__
*  }
*}
**/
var foo = new Foo();

debugger;

console.log(JSON.stringify(foo)); // 还是输出: {"bar":"foo bar"}

// 输出: bar foo bar
// b foo bar b
for(var propName in foo) {
    propValue = foo[propName]
    console.log(propName,propValue);
}
```

![使用prorotype扩展属性](https://cloud.githubusercontent.com/assets/729479/5331544/d11c4ad2-7e72-11e4-94f4-eee0b5232753.png)

至于使用prototype实现继承效果，不详细解释。这里给出一般使用的`extend()`方法的定义，再来一个使用实例：[JSFiddle](http://jsfiddle.net/simongong/exjd51f9/3/)

```
var extend = function (Base) {
    var Class = function () {
        Base.apply(this, arguments);
    }, F;
    if (Object.create) {
        Class.prototype = Object.create(Base.prototype);
    } else {
        F = function () {};
        F.prototype = Base.prototype;
        Class.prototype = new F();
    }
    Class.prototype.constructor = Class;
    return Class;
};
```
![使用prorotype继承属性](https://cloud.githubusercontent.com/assets/729479/5331546/d70c52a2-7e72-11e4-932f-953682404ee3.png)

## 不使用new呢？
如果不使用`new`关键字，那就是函数调用。本质上就是执行内存中`window`对象的代码。

而函数定义这个操作，本身就是给`window`这个JSON对象添加成员。

比如

```
// 以下函数定义的实质是window.Foo = function{}
var Foo = function(){
  this.A = 1;
  this.B = 2;
};

Foo(); // 将会对`window`对象添加两个属性A和B。因为此时this指向的内存是`window`
```
