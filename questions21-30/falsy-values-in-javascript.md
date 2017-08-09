原问题：[Why is null an object and what's the difference between null and undefined?](http://stackoverflow.com/questions/801032/why-is-null-an-object-and-whats-the-difference-between-null-and-undefined)

提问者主要是问`null`，以及`null`与`undefined`的区别。延伸一下，可以一起说说JavaScript中的**falsy value**。


## Falsy Value
《JS Definitive Guide》和《JS The Good Parts》中都提到了*falsy value*的概念。falsy value指的是逻辑表达式的结果为假的情况。以下值会被当作falsy value：

* false
* null
* undefined
* ''
* 0
* NaN

所有这些值的`if ( !x ) { }`都会通过。除此以外其他的值都是true value真值，`if ( x ) { }`都会通过。

但有种情况需要注意：

> 对于一个window对象的属性（全局变量），比如window.bar。如果bar未定义在window中，那么使用`if (bar ) { }`会报错。这时候需要使用`if ( typeof bar === 'undefined' )`或者`if ( 'bar' in window )`。

## 具体的falsy value
首先要更正原问题提出者的一个错误：**`null`不是一个对象。** 尽管使用`typeof null`会返回`object`，但这是typeof的bug。`null`是一个*primitive value*，它没有其他属性和方法，你也无法给它添加属性和方法。它不是一个对象。

接下来对上述各个falsy value，借用排名第一的答案来做说明。

**背景：**

我需要引用一个叫name的变量的值，因此我问JavaScript解释器：告诉我name的值是多少。

**场景一：** *（name 为 undefined）*

* name? 什么是name? 我不知道你在说什么，之前也从没人提过它。

**场景二：** *（name = null）*

* 我知道有name这个变量存在，但我不知道它的值是多少。

**场景三：** *（name = false）*

* name的值为布尔值false

**场景四：** *（name = ''）*

* name的值是一个空字符串

**场景五：** *（name = NaN）*

* name的值是一个无穷大的数字

## 垃圾回收
由`falsy value`和`null`，你也许会想到或者曾经听到过，它跟垃圾回收相关。我们就来看看JavaScript的垃圾回收。

跟其他很多种高级语言一样，JavaScript的垃圾回收机制也是采用的[引用计数](http://en.wikipedia.org/wiki/Reference_counting)技术。

引用计数是用图的结构来表示当前每块内存的引用情况，当图中出现**无法到达的**_(unreachable )_节点的时候，这块节点的内存就是可以被回收的对象。示例：

![reference-counting-GC](https://farm8.staticflickr.com/7511/15833370392_80ac7c3ce3_o.gif)

而对于引用计数中的具体的**无法到达**的判定问题，使用的是[Mark-and-sweep算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#Mark-and-sweep_algorithm)。

大致过程如下：（摘取自[MSDN - Mark and Sweep Garbage Collection](http://blogs.msdn.com/b/abhinaba/archive/2009/01/30/back-to-basics-mark-and-sweep-garbage-collection.aspx)）

```
void GC()
{
    HaltAllProcessing();    // 阻塞正常程序的执行
    ObjectCollection roots = GetRoots();
    for(int i = 0; i < roots.Count(); ++i)
        Mark(roots[i]); //标记
    Sweep();    //回收
}
```

JavaScript引擎会定期扫一遍这个图，标记无法到达的节点，然后回收这些节点的内存。GC过程会阻塞正常程序的执行。

### falsy value与垃圾回收
`null && undefined`与垃圾回收有点关系：它销毁目标变量对当前内存的引用。

比如：`var a = new Obj(); var b = a; a = null;`，之后，a变量就不再指向obj那块内存的引用了。但b仍然指向obj那块内存。

因此，`a = null`能促进垃圾回收，但不会直接引发垃圾回收。

另外，`delete`运算符能删除一个对象的某个属性，删除的结果就是`obj.attr = undefined`。因此，它也能促进垃圾回收。（但之前在[如何移除一个对象的某个属性](https://github.com/simongong/js-stackoverflow-highest-votes/blob/master/questions1-10/remove-property-from-javascript-object.md)中已经提到过，delete效率低，少用）


