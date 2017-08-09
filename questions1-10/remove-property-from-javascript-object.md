原问题：[Remove property from JavaScript object](http://stackoverflow.com/questions/208105/remove-property-from-javascript-object)

如何移除一个JavaScript对象？

首先分析问题：

* JavaScript对象的结构都是JSON
* 问题转化成移除一个JSON对象的属性


再由具体到抽象，尝试三个层面来解决这个问题：
  * JSON Object Property，查ECMAScript JSON规范 ->
  * Object Property：查ECMAScript Object规范 ->
  * Property：因为一个对象的属性仍然是对象，所以仍然查ECMAScript Object规范

根据ECMAScript中对[JSON对象](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-json-object)的定义，它只有两个方法：`parse() && stringify()`，放弃。

根据ECMAScript的[Working with Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects)，文档倒数第二小节正好讲得就是[Deleting properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Deleting_properties)：
> 可以使用`delete`运算符来删除一个对象的**非继承**属性。

因此，我们就从`delete`运算符入手。
* delete运算符
  * 语法
  * 运算结果
* 谨慎使用delete
  * 为什么delete很慢
  * delete的替代方案

## delete
来看看[`delete`运算符的定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators#delete)：
> `delete`运算符用来删除一个对象、一个对象属性或者数组中的某个元素。

#### delete语法
示例：
```
delete objectName;
delete objectName.property;
delete objectName[index];
delete property; // legal only within a with statement
```

> `delete`还能用来删除一个全局变量，如果该变量不是使用`var`声明的。（区分*variable declaration*和*property assignment*）

示例：
```
var GLOBAL_OBJECT = this;

/* create global property via variable declaration; property has DontDelete */
var foo = 1;

/* create global property via undeclared assignment; property has no DontDelete */
bar = 2;

console.log(foo);
delete foo; // false
console.log(typeof foo); // "number"

console.log(bar);
delete bar; // true
console.log(typeof bar); // "undefined"
```

#### delete结果
如果`delete`操作成功，它将会把操作数的值置为`undefined`，然后返回`true`。否则返回`false`。

注意，使用`delete`删除一个数组array的元素i后，array[i]元素就不存在了。
```
var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
delete trees[3];
if (trees[3] || 3 in trees) { // undefined || false
  // this does not get executed
}
```

因此，如果你只是想删除array[i]的值，同时保持i元素在array中的位置，就不能使用`delete`，而应该直接对array[i]赋值。
```
var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
trees[3] = undefined;
if (trees[3] || 3 in trees) { // undefined || true
  // this gets executed
}
```

## 谨慎使用delete
因为`delete`操作的效率很低。

#### 为什么`delete`很慢
因为JavaScript语言的动态属性：
> 可以随时随地向对象添加、删除属性。

因此大多数JavaScript引擎对Object的实现时，采用的是*dictionary-like data structure*，比如`hashmap`。采用这种数据结构的一大弊端就是*dictionary lookup*是非常低效的，即读取速度慢。

为了提高读取对象属性的速度，V8引擎使用了*hidden class*技术，其基本思想就是：空间换时间。
* 每个对象都对应生成一个hidden class
* 生成属性表，使用Java/Smalltalk等静态语言对于对象属性的编译方式：fixed offset
* 每次增加或删除对象属性的时候，更新其hidden class，这称作*transition*

V8 hidden class的具体细节，请参考：[Fast Property Access - V8 Design](https://developers.google.com/v8/design#prop_access)

注意第三条，每次增加或删除属性的操作，都会引发hidden class transition。如果一个对象的属性很多，那么这个transition操作将会很耗资源。

而JavaScript的prototype继承机制，使得对象属性的数量很容易就达到较大的数字。因此，尽量避免使用`delete`。

#### delete的替代方案
与`delete`类似的方案是`undefined/null`的赋值操作。

根据它们的[性能对比测试](http://jsperf.com/delete-vs-undefined-vs-null/30)可以看出，使用`undefined/null`的赋值操作的效率，是`delete`操作的十倍以上。对于一些计算密集的算法，尤其需要注意。



