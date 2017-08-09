原问题：[How does JavaScript .prototype work?](http://stackoverflow.com/questions/572897/how-does-javascript-prototype-work)

JavaScript的prototype，准确地说，是[Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)。

Object.prototype有以下特点：
* `{writable: false, enumerable: false, configurable: false}`
* 本身是一个对象
* 所有JavaScript对象都继承它，拥有它的属性可方法
* prototype对象的方法可以被其他对象的constructor覆盖
* 对于一个prototype属性的修改，会影响所有该对象实例

暂时不深入这些细节，我们先来关注一个一直被忽略的问题。

我看到几乎所有人在讲Object prototype的时候，上来就讲Function prototype，举的例子也全都是函数的prototype属性。他们似乎遗漏了对一个前提的说明：
> Why things about Object prototype come to Function prototype?

## Object prototype
我们由前面的定义可知道：所有JavaScript对象都有prtotype属性。

而[JavaScript Standard Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)中，只有Function有一个可写的prototype属性。因此，在谈到Object prototype的时候，很多时候是指Function prototype。

但其实，在prototype的世界里，Function是JavaScript中的一个例外。如果你是因为习惯用
```
var foo = function(){};
foo.prototype.bar = function(){};
```
的方式来定义一个函数，以为Object.prototype就是Function.prototype。那应该看下面。

其实不然。

这个属性的说明请查看[Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)。它是一个对象，有两个属性：
- [Object.prototype.__proto__](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)
  - 与这个属性相关的，有两个方法：[Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create), [Object.getPrototypeOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf)
- [Object.prototype.constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor)
  - 如果该对象是由函数创建的，那么该属性是对函数本身的一个引用。如果不是由函数创建的，那么它是指向Object的一个引用。注意它不是一个字符串。

关于Object.prototype，做以下几点总结：

1. 在浏览器中的实现为__proto__属性，表示为Object.__proto__。它是只读的，你不能修改它；
2. 所有对象都包含这个属性；
3. 可以使用obj.property直接访问Object.__proto__.property；
4. `Object.create(obj)`方法返回一个函数对象，这个函数对象的__proto__属性是obj.__proto__的一份复制，同时该函数对象的prototype属性为null。
```
if (typeof Object.create != 'function') {
  Object.create = (function() {
    var Temp = function() {};
    return function (prototype) {
      if (arguments.length > 1) {
        throw Error('Second argument not supported');
      }
      if (typeof prototype != 'object') {
        throw TypeError('Argument must be an object');
      }
      Temp.prototype = prototype;
      var result = new Temp();
      Temp.prototype = null;
      return result;
    };
  })();
}
```

对于第四点中的**同时该函数对象的prototype属性为null**不理解的话，继续看下面。

来看一张对于JavaScript prototype的集大成的图（可以看完本文后续部分之后再回来看这张图）：
![prototype in javascript](https://cloud.githubusercontent.com/assets/729479/6819612/4cde299a-d307-11e4-9d00-2234e42eaa57.png)

## Function prototype
要说Function prototype，首先得明确：

一个Function本身也是一个Object，因此它有__proto__属性。同时，它还有一个prototype属性，这是函数对象特有的。

来看一张代码图示：![Structure of Function and Object in JavaScript](https://cloud.githubusercontent.com/assets/729479/6819476/f6e52158-d304-11e4-8ebf-676c31fdace7.png)

看图上标注的1/2/3/4，体会一下，一个函数对象包含的prototype和__proto__，这两个属性的异同。

对上图再分析其中的`defFunc.prototype`，看一下它与`defFunc.__proto__`的异同。
![Function.prototype](https://cloud.githubusercontent.com/assets/729479/6819479/f7a07e1c-d304-11e4-942e-efb6fceba284.png)

顺便说一句，JavaScript中的Function与Object另两点不同就是：

* 它使用`new`关键字来创建对象，而Object则只能使用`Object.create()`方法来创建对象
* [instanceof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof)运算符只作用与函数对象，不作用于一般Object对象

最后来看看，使用`new`关键字创建出来的函数对象，跟`Object.create()`方法创建出来的对象有什么异同。
![Function object and normal object](https://cloud.githubusercontent.com/assets/729479/6819795/2c38339a-d30a-11e4-9f72-7d137cb2944e.png)

[John Resig's JavaScript Ninja slide](http://ejohn.org/apps/learn/#64)中，对Function.prototype做了一个总结。

* 所有JavaScript对象都继承它，拥有它的属性可方法 (只有对象才有prototype)
```
function Ninja(){}

Ninja.prototype.swingSword = function(){
  return true;
};

var ninjaA = Ninja();
assert( !ninjaA, "Is undefined, not an instance of Ninja." );

var ninjaB = new Ninja();
assert( ninjaB.swingSword(), "Method exists and is callable." );
```
* prototype对象的方法可以被其他对象的constructor覆盖
```
function Ninja(){
  this.swingSword = function(){
    return true;
  };
}

// Should return false, but will be overridden
Ninja.prototype.swingSword = function(){
  return false;
};

var ninja = new Ninja();
assert( ninja.swingSword(), "Calling the instance method, not the prototype method." );
```
* 对于一个prototype属性的修改，会影响所有该对象实例
```
function Ninja(){
  this.swung = true;
}

var ninjaA = new Ninja();
var ninjaB = new Ninja();

Ninja.prototype.swingSword = function(){
  return this.swung;
};

assert( ninjaA.swingSword(), "Method exists, even out of order." );
assert( ninjaB.swingSword(), "and on all instantiated objects." );
```

## 后记
在本文章的第一版中，还遗留了这两个问题：
* JavaScript Object到底是怎样一个结构
  * 看文中的插图，应该就明白了
* 创建一个JavaScript对象一定要用`new`关键字吗
  * 用`{}`声明的变量，就是创建了对象；用`function(){}`声明的变量，需要使用`new`关键字