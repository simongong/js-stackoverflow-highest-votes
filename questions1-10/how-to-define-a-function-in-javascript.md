原问题：[var functionName = function() {} vs function functionName() {}](http://stackoverflow.com/questions/336859/var-functionname-function-vs-function-functionname)

*PS：对于这个问题，SO上的各个答案之间对于__JavaScript语法解析和执行过程__有不同意见。其中包含了不少文章和ECMAScript5文档。以下内容也引用了其中一些文档，如果有疑问请先参考那些引用文档的描述。*

定义一个JavaScript函数，有两种方式（[Function Definition - ECMAScript5](http://es5.github.io/#x13)）：

- 赋值语句(**Function Expression**)
```
var functionOne = function() {
    // Some code
};
```
- 函数声明(**Function Declaration**)
```
function functionTwo() {
    // Some code
}
```

##区别 - hoist
尽管平时写代码会随意地把这两种方式混用，但它们在**语义**上还是有区别的。区别的原因就在于JavaScript解释器对程序的解析过程中的一个特性——**hoist**
> Function declarations and function variables are always moved (‘hoisted’) to the top of their JavaScript scope by the JavaScript interpreter.  
-- [JavaScript Scoping and Hoisting](http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html)

也就是说，functionTwo在被hoist之后是这样的：
```
var functinoTwo;
// Some code

function functionTwo() {
    // Some code
}
```
这个**hoist**是在解析过程中的**variable instantiation**进行的（这一步是在执行代码run time之前）：所有`var`定义的变量都被初始化为`undefined`，而function declaration定义的函数将被正常初始化。  
关于解析代码的具体过程的规定，请参考[ECMAScript5 - Executable Code and Execution Contexts](http://es5.github.io/#x10)*（我扫了一遍，与上面那个文章里说的是一致的）。*

有了**hoist**的概念，我们就能理解一个直观的区别——可见性：
- functionOne只对其后面的代码可见
```
xyz(); // UNDEFINED!!! we can't call it here
xyz = function(){}  // now it is defined
xyz(); // we can call it here
```
- 而functionTwo对其所在的scope都是可见的
```
abc(); // works. we can call it here
function abc(){}  // yet it is defined down there
abc(); // works. we can call it again
```

##共同点
尽管在parse time时二者不同，但到了run time它们都是普通的函数引用。
- 最终在run time的时候，它俩都变成global object的一个*non-deleteable*属性
- 都可以被重新指向。比如：`functionOne = functionTwo; functionTwo = null;`

##总结
其实对这两类函数定义方式，我看不出很明显的应用场景的差别。  
只是从实践经验来看，使用**自执行函数**或者**模块化JavaScript**的方式下：

- 使用functionOne定义公开成员
- 使用functionTwo定义私有方法

这部分内容写到这里就差不多了。但我对hoist还是有些疑问。举个例子：
```
var foo = 1;
function bar() {
  if (!foo) {
    var foo = 10 
  }
  return foo; 
}
bar(); // 输出10
```
`foo`被hoisted，初始化为`undefined`。所以`!foo`为真，然后执行`foo = 10;`。等于说第一行定义的`var foo = 1;`压根没起到任何作用。

好，那么问题来了，我们为什么需要**hoist**？
*(难道是因为当初为了增加JavaScript的灵活性，不用拘泥于__先定义后使用__，而弄出一个hoist?)*



