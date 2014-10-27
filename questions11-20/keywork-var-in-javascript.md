原问题：[What is the function of the var keyword and when to use it (or omit it)?](http://stackoverflow.com/questions/1470488/what-is-the-function-of-the-var-keyword-and-when-to-use-it-or-omit-it)

`var`关键字是个很蛋疼的点，当初我看到这个问题的时候准备略过的。我不想写这么细的点，觉得价值不大浪费时间，认为只要记住——**永远用`var`**——就行。

但认真看了这个问题和众人的解答之后，觉得还是能理出一些JavaScript重要的知识点，同时还不用落入繁杂细节。

首先对`var`来个总结：

* `var a`在当前scope定义一个变量a
* global scope是`window`
* 不使用`var`的话，a将默认被定义在global scope内
* `a`被使用时，JavaScript解释器会**由小到大**的范围内寻找`a`的定义

从这几条总结中，再提炼一下涉及到的JavaScript知识点。

##Varaiable Scope
ECMAScript在对[Variable Statement](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-variable-statement)的定义中提到：
> A var statement declares variables that are scoped to the running execution context's VariableEnvironment.  
Var variables are created when their containing Lexical Environment is instantiated and are initialized to undefined when created.

因此，`var`定义的对象是**有范围**的，这个范围是**当前运行的执行上下文的变量环境**_(running execution context's VariableEnvironment)_。

顺藤摸瓜，我们来看看什么是*running execution context*和*VariableEnvironment*。

###Running Exucution Contexts
首先来看*[Exucution Context](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-execution-contexts)*：
> 执行上下文，是由ECMAScript的实现*（比如V8）*提供的一个对象，用来追踪代码的执行进度和相关的运行时的值。在代码被执行的任一时刻，都只有一个执行上下文。

打开一个页面，调用一个函数，都会创建一个新的执行上下文。

一个运行上下文包含以下组件：

* Code evaluation state
* Function - 当前的上下文是否追踪的是一个函数的执行，否则为`null`
* Realm
* LexicalEnvironment - 词法环境，解析该上下文中的标识符
* VariableEnvironment - 标识该上下文中由*VariableStatements*创建的变量

JavaScript引擎维护一个*execution context stack*，当前的*running execution context*始终处于这个栈的顶端。

###VariableEnvironment
初始创建执行上下文的时候，`LexicalEnvironment`和`VariableEnvironment`的值是一样的。在代码被执行的过程中，VariableEnvironment始终不变，而LexicalEnvironment会改变。

VariableEnvironment包含以下两部分：

* 外部变量
  * global variant
  * closure variant
* 内部`var`定义的变量

因此，*Variable Scope*就是指当前的*running execution context*中的*VariableEnvironment*。

##With Statement
答案中有人说，浏览器中的JavaScript代码是这样被执行的：
```
with (window) {
    //Your code
}
```

我考证了下，没找到根据。不过根据实际经验，这个是成立的。如果这个是真的，那么上面的总结中，第二条和第三条都是成立的。

因为根据[ECMAScript中对`With`的定义](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-with-statement)：
> The with statement adds an object environment record for a computed object to the lexical environment of the running execution context. It then executes a statement using this augmented lexical environment. Finally, it restores the original lexical environment.

所以，`window`就成了浏览器中JavaScript代码的*original lexical environment*，也就是*VariableEnvironment*

##定位变量
当JavaScript引擎解析到一个语句里的`a`变量时，如何确定它的值呢？

答案就是在*VariableEnvironment*中查找。

根据前面总结的第四条：
> `a`被使用时，JavaScript解释器会**由小到大**的范围内寻找最近的`a`的定义，寻找失败则当`a`是一个值为`undefined`的global variant。

因此：**VariableEnvironment中保存的变量及其运行时的值，是带scope的**。

来看一个例子：
```
/* global scope */
var local = true;
var global = true;

function outer() {
    /* local scope */
    var local = true;
    var global = false;

    /* nearest scope = outer */
    local = !global;

    function inner() {
        /* nearest scope = outer */
        local = false;
        global = false;

        /* nearest scope = undefined */
        /* defaults to defining a global */
        public = global;
    }
}
```


