原问题：[Loop through array in JavaScript](http://stackoverflow.com/questions/3010840/loop-through-array-in-javascript)

在Java中，可以使用改进的`for()`循环来遍历一个数组：
```
String[] myStringArray = {"Hello","World"};
for(String s : myStringArray)
{
    //Do something
}
```

在JavaScript中对数组的遍历也能这样简洁方便么？

* [JavaScript中的for语句](#for-statement)
* [JavaScript的对象及其属性](#object-and-properties)

##for Statement
JavaScript中的`for`语句有两种形式：
```
    // for
    for (initialization; condition; update) {
        statements
    }

    // for...in
    for (variable in object) {
        if (filter) {
            statements
        }
    }
```

第一种for语句跟Java中一样，它是遍历 _(Iteration)_。第二种为枚举 _(Enumeration)_，我们称其为`for...in`语句，它有点特别。

关于Iteration和Enumeration的区别，请参考：[Enumeration VS Iteration](http://web.archive.org/web/20101213150231/http://dhtmlkitchen.com/?category=/JavaScript/&date=2007/10/21/&entry=Iteration-Enumeration-Primitives-and-Objects)

从形式上可以看出，`for...in`是用来*遍历一个对象的所有属性(Properties)*的。

##Object and Properties
* JavaScript中，除了基本值类型变量之外，其他变量都代表一个对象。
* 一个JavaScript的对象就是一个JSON实例，是一组属性的集合。
* 一个JavaScript对象的属性也是变量

因此这是一个递归定义。

###[Object Properties](http://www.w3schools.com/js/js_properties.asp)
对于一个对象的属性*(Object Properties)*，我们从其性质和来源两方面来分析。

**性质**：
* 属性(Property)本身是一个对象，有一个名字
* 属性(Property)包含若干个attributes，包括value, enumerable, configurable, writable。这些attributes定义了存取这个属性的权限

**来源**：

由于[Prototype Chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)机制，一个JavaScript对象会自动地*拥有*其Prototype Chain上游对象的属性。

* 继承属性 _(inherited property)_
* 自定义属性 _(own defined property)_

这两类属性的性质是不同的，具有不同的存取权限。

为了区分一个对象的属性的来源，JavaScript为Object对象提供了`hasOwnProperty(key)`方法。

因此，如果你使用`for...in`且只想遍历你的自定义对象的自定义属性，那么需要使用`hasOwnProperty(key)`方法。
```
    for (variable in object) {
        if (object.hasOwnProperty(variable)) {
            statements
        }
    }
```

