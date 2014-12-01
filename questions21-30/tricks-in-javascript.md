每种编程语言里都有一些小伎俩，有些确实能带来方便，有些则只是纯蛋疼。

这里收集一些常见的JavaScript伎俩，按需自取。

## !function foo(){}()
原问题：[What does the exclamation mark do before the function?](http://stackoverflow.com/questions/3755606/what-does-the-exclamation-mark-do-before-the-function)

对于`!function foo(){}()`，有以下几点：

* 运算符`()`比`!`的优先级高，所以它等价于`!(function foo(){}());`
* `!(statement);`会触发JavaScript引擎去解析statement，然后取其结果的布尔反值
* 该语句的目的就是执行这个函数，它等价于`(function foo(){})();`

直接使用`function foo(){}()`会导致语法错误，因为

```
function foo() {}
```

是一个函数声明语句，末尾没有`;`号，对它的解析到`}`号为止，你不能在后面接`(`。想要调用`foo`，只能使用`foo();`。

因此，这个伎俩的目的只是为了省一个字符：`!function foo(){}()` VS `(function foo(){})();`

## !!variant
原问题：[What is the !! (not not) operator in JavaScript?](http://stackoverflow.com/questions/784929/what-is-the-not-not-operator-in-javascript)

JavaScript中没有`!!`运算符，它实际上是两次`!`逻辑运算，结果是把任意一个JavaScript对象转化为一个布尔值。

这其中，如果variant本身一个falsy value，那么`!!variant`是布尔假。否则为真。

```
console.log(!!1);   // true
Boolean(1) === !!1;  // true
console.log(!!"");   // false
Boolean("") === !!"";    //true
```

可以把它看作是布尔值转换运算。但如果你是为了在`if`语句中使用，大可不必这样做。因为JavaScript引擎会对`if(statemen){}`中的`statement`自动执行这个布尔值转换操作。比如，以下代码会把`bar`当作布尔真值来执行。

```
var bar = {a: 1};
if(bar){
    console.log(true);
}else{
    console.log(fasle);
}
```

## $variant
原问题：[Why would a JavaScript variable start with a dollar sign?](http://stackoverflow.com/questions/205853/why-would-a-javascript-variable-start-with-a-dollar-sign)

这只是一个习惯。使用`$`作为变量名的前缀，是来源于jQuery。jQuery变量的别名是`$`，所以在使用jQuery的JavaScript代码中，一般把jQuery对象的变量名都加一个`$`前缀，表明这是一个jQuery对象，不是一个DOM对象或者普通的Object。

后来在Backbone以及相关的前端框架中也沿用了这一习惯。

反过来看，有这种风格的框架一般都依赖于jQuery。

## 函数参数的缺省值
原问题：[Set a default parameter value for a JavaScript function](http://stackoverflow.com/questions/894860/set-a-default-parameter-value-for-a-javascript-function)

在Java/C++/Python中定义方法的时候都能为参数指定一个默认值，但JavaScript语言没有这个机制。因此我们只能仿造。

一般是这样来做的：

```
function myFunc(requiredArg, optionalArg){
  optionalArg = (typeof optionalArg === "undefined") ? "defaultValue" : optionalArg;

  //do stuff
}
```

如果你确定这个参数是一个非falsy value的对象，还可以简化（这是我用得最多的方式）：

```
function myFunc(requiredArg, optionalArg){
  optionalArg = optionalArg || defaultValue;

  //do stuff
}
```

注意：

* 跟Python一样，你只能把带默认值的参数放在参数列表的最后。
* ECMAScript6中的[Function](http://people.mozilla.org/~jorendorff/es6-draft.html)已经加入该特性，Firefox中已经支持：[Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)

## `+`运算符
原问题：[What is the explanation for these bizarre JavaScript behaviours mentioned in the 'Wat' talk for CodeMash 2012?](http://stackoverflow.com/questions/9032856/what-is-the-explanation-for-these-bizarre-javascript-behaviours-mentioned-in-the)

该问题来源于：[Wat - A lightning talk by Gary Bernhardt from CodeMash 2012 ](https://www.destroyallsoftware.com/talks/wat)

这个问题中有很多蛋疼的例子，在这里说它的目的是提醒大家：严谨地使用`+`运算符。

以下是使用`+`的蛋疼示例，运行示例请参考[jsfidde](http://jsfiddle.net/fe479/9/)

* [] + []
  * // an empty string
* [] + {}
  * [object Object]
* {} + []
  * 0
* {} + {}
  * NaN
* Array(16).join("wat" + 1)
  * wat1wat1wat1wat1wat1wat1wat1wat1wat1wat1wat1wat1wat1wat1wat1
* Array(16).join("wat" - 1)
  * NaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaNNaN

实际上以上结果都是遵循[ECMAScript5](http://www.ecma-international.org/publications/standards/Ecma-262.htm)的执行结果。对于`+`运算符的规定为 [The Addition operator ( + )](http://www.ecma-international.org/ecma-262/5.1/#sec-11.6.1)：

* 不限制运算数必须为primitive
* 首先将两个操作数转化为primitive - [ToPrimitive](http://www.ecma-international.org/ecma-262/5.1/#sec-9.1)
* 将一个object转化为primitive就是返回其`default value`，如果该object有`toString()`方法，将返回该方法的值
* 对于Array来说，就是调用其`.join()`方法
* 对于object来说，调用`toString()`将返回`"[object Object]"`

以上规范能解释大部分使用`+`的场合，但还有两个示例需要进一步的解释：

* {} + []
  * `{}`会被解析为一个[block](http://www.ecma-international.org/ecma-262/5.1/#sec-12.1)，而不是一个对象（除非你使用`({})`使它成为一个表达式）
  * 因此`{} + []`等价于`+[]`
  * `+operand`将返回`toNumber(ToPrimitive(operand))`
  * `+[]`将返回`toNumber("")`，也就是0
* {} + {}
  * 由上可知它将返回`toNumber("[object Object]")`
  * 根据[toNumber()的定义](http://www.ecma-international.org/ecma-262/5.1/#sec-9.3.1)，这无法转成number，因此返回NaN

太复杂。实际使用中，最好保证`+`的两个操作数均为primitive类型，不能是object类型。
