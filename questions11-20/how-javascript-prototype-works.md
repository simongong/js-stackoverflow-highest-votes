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

##Object prototype
我们由前面的定义可知道：所有JavaScript对象都有prtotype属性。

而[JavaScript Standard Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)包含以下几类：
* Value properties
  * Infinity, Nan, undefined, null
* Function properties
  * eval(), isNaN(), parseInt()...
* Fundamental objects
  * Object, Function, Boolean, Error...

_...(更多详细请参考链接文档)_

在这些对象中，只有Function的prototype是可写的。因此，在谈到Object prototype的时候，一般就是指Function prototype。

##Function prototype
对于这个主题，有一个很好很精简的PPT讲解：[John Resig's JavaScript Ninja slide](http://ejohn.org/apps/learn/#64)。

其内容跟我上面列举的Object.prototype的特点3/4/5是一致的。这里也以PPT中的代码实例来说明。

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

##后记
在本文章中，如果你够细致，会发现还遗留了这几个问题：
* JavaScript Object到底是怎样一个结构
* 创建一个JavaScript对象一定要用`new`关键字吗

由于这几个问题会单独开一个主题写，因此在那部分完成之后，我再链接过来。