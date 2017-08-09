原问题：[How to empty an array in JavaScript?](http://stackoverflow.com/questions/1232040/how-to-empty-an-array-in-javascript)

问题很简洁直白：
> var A = [1, 2, 3, 4]; 如何让A变成一个空数组？

答案也很简洁直白。有四种方法：
* A = [];
* A.length = 0;
* A.splice(0, A.length);
* while(A.length > 0){ A.pop(); }

这几种方法的区别，主要在于**内存**和**引用**。

* A = [];
改变A的指向，指向一块新内存，内容为一个空数组。原数组[1, 2, 3, 4]将继续存在或以后被GC回收。
* A.length = 0;
这个语句会使JavsScript解释器将[1, 2, 3, 4]那块内存的内容变成[].
* A.splice(0, A.length);
对于A来说与2方法的效果一致。但该方法有返回值，返回值是一块新内存，内容是被移除的子数组。该例中会返回[1, 2, 3, 4]。
* while(A.length > 0){ A.pop(); }
与2方法的效果一致，并且比1,2,3的效率都要高。

对这几种方式的性能比较：http://jsperf.com/empty-javascript-array

这几种方法里，我们唯一不太确定其具体动作的，就是`A.length = 0;`。所以我们来更深入地研究下：

* [Array.length属性](#array-length)
* [Array.length = n到底做了些什么](#set-array-length)

## Array Length
ECMAScript6草案中定义：[Array.length](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-properties-of-array-instances-length)

* 永远比array index大
* 作为一个对象的属性，其本身属性为{ [[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false }
* 改变Array.length的值，会导致数组实际大小的改变。

## Set Array Length
同样，ECMAScript6草案中也定义了[ArraySetLength(A, Desc)](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-arraysetlength) 的抽象操作过程。

对于语句
```
A.length = 0;
```
假设`oldLen`是A的原始长度，`newLen`为新长度值0。其核心处理为：

* 如果`newLen >= oldLen`，直接扩充数组长度，返回[OrdinaryDefineOwnProperty](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-ordinarydefineownproperty)
* 如果`newLen < oldLen`，则执行一个`while(newLen < oldLen)`，循环内容为
  * `oldLen -= 1;`
  * 删除A的尾元素 _(这里我忽略删除失败情况的处理)_

因此可以看到，表面上简单直接的`A.length = 0;`，背后的处理仍然是逐一删除元素。而且这个过程中还有多个值类型判断、执行结果判断等，因此它在效率上比第4种方法：
```
while(A.length > 0){ A.pop(); }
```
要低。

## 总结
* `while(A.length > 0){ A.pop(); }` 最直接，效率第一
* `A = [];` 最简洁直观，但浪费GC。效率第二
* `A.length = 0` 也简洁直观，但效率最低
* `A.splice(0, A.length);` 不简洁，效率较低，最浪费内存

