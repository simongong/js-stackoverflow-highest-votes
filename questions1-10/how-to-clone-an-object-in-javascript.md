原问题有两个：

* [What is the most efficient way to clone an object?](What is the most efficient way to clone an object?)
* [Most elegant way to clone a JavaScript object](http://stackoverflow.com/questions/728360/most-elegant-way-to-clone-a-javascript-object)

这个问题从提问那会儿开始，经过了6年旷日持久的讨论，jQuery作者John Resig也顺便推销了一下[jQuery.clone()](http://api.jquery.com/clone/)。实际上`jQuery.clone`表现得确实不错，后来的[underscore](http://underscorejs.org/#clone)和[lodash](https://lodash.com/docs#clone)也都提供了类似函数。而JSON本身也能简单的实现拷贝功能。

这里先给一个这四个方案的性能比较：[object clone jquery vs underscore vs lodash](http://jsperf.com/object-clone-jquery-vs-underscore-vs-lodash)

从性能测试结果来看，在实现deep clone的前提下，表现最好的是Lo-Dash。但有两个地方需要注意的是：

1. 这四种方案拷贝出来的结果是不一样的，因此不能直接根据此性能结果来下结论
2. underscore不提供deep clone方法。因为他们觉得无法提供一个完美的能广泛适用的deep clone。对此问题的讨论[github issue](https://github.com/jashkenas/underscore/issues/162)。

以下内容是扩展，真正的答案在最后。

## JSON对象clone
克隆一个JSON对象比较简单：

```
JSON.parse(JSON.stringify(obj));
```

对于一个JavaScript对象，这种方法无法克隆两类属性：prototype和function。

由于JavaScript对象中有以上两类特殊的属性，想要实现一种广泛适用没有bug的克隆一个JavaScript对象的操作，比较困难。

因此，通常说的JavaScript Object Clone中通常存在两个比较棘手的问题：

* function不是对象实例所对应的JSON的属性
* deep clone无法完美处理Object.prototype

对于第一个问题，我们可以不使用`JSON.parse(JSON.stringify(obj));`的方式，而是用遍历属性的方式来解决。

第二个问题就比较麻烦，这里稍微说一下。

## Prototype Chain实现
对于prototype chain机制，其实现也是很直白的。

如果你在chrome的Dev Tools里查看过JavaScript对象，你就会知道每个对象都有一个__proto__属性。该属性有一个名字，它就是prototype.constructor，显示为：`__proto__: constructor`。这个属性在Debug时很有用。

如下图所示：
![__proto__ chain](https://cloud.githubusercontent.com/assets/729479/5331546/d70c52a2-7e72-11e4-932f-953682404ee3.png)

可以看到，`__proto__`属性中还有一个Object.prototype对象，所以它是一个嵌套定义。

## 如何克隆Object.prototype
目前为止这是一个无解问题，根本原因是ECMAScript在这方面还不够完备。

首先是你无法用`for...in`循环来访问它，因为它是一个隐藏属性，你只能指定属性名来访问它，但你又不知道浏览器是用什么属性名来实现它的。虽然目前主流浏览器对它的命名为`__proto__`。

其次是你无法真正拷贝它，因为`non-enumerable`属性。具体请看：

* [defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
* [JavaScript properties: inheritance and enumerability](http://www.2ality.com/2011/07/js-properties.html)

## 结论
首先要明确，你究竟想拷贝一个对象的什么？只是数据？还是包括functions, prototype chain和constructor？

* 如果只是想拷贝数据，`JSON.parse(JSON.stringify(obj));`就足够了
* 如果还想拷贝对象中的function定义，那就用Lo-Dash的`_.cloneDeep(obj)`
* 如果还想拷贝通过`function.prototype.property`方式添加的属性，请使用jQuery的`$.extend(true, {}, obj);`。

示例：[JSFiddle](http://jsfiddle.net/simongong/jmajxyL5/2/)
![clone object](https://cloud.githubusercontent.com/assets/729479/5334796/1b30aefe-7ede-11e4-8e96-cc8cc691c766.png)

其他目的，请自行实现。




