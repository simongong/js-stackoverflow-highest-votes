> Strict Mode是ECMAScript5引入的一个新特性。使用它能使得你的程序或者函数运行在**严格**的环境中。这个严格的运行环境能预防一些危险操作，并会给出异常信息。  
- 捕获一些常见的编程陷阱
- 阻止一些危险操作的发生（比如存取全局对象）
- 禁用了JavaScript中的一些让你迷惑或易发生错误的特性

###使用Strict Mode
它是一个声明式的语句，语句本身就是`"use strict"`这个字符串。

Strict Mode有其应用范围。

如果你想对整个JavaScript文件应用它，那么就把它放在文件第一行：
```
"use strict";
// ... your code ...
```

你也可以只在某个函数上应用它：
```
// Non-strict code...

function(){
  "use strict";
  // ... your code ...
}

// Non-strict code...
```

为了避免污染全局变量，很多library都以**self-excuting function**的形式来定义。对于这些library，仍然可以应用Strict Mode：
```
// Non-strict code...

(function(){
  "use strict";

  // Define your library strictly...
})();

// Non-strict code...
```

总的来说，`use strict`是JavaScript为开发者提供的一个**代码检查工具**。它能在指定范围内检查你的代码，让你避免犯那些常见的JavaScript错误和陷阱。

*注意：Strict Mode并没有引入新的语法，你仍然可以对以前的代码使用它。*

###哪些地方变严格？
jQuery的作者John Resig对此总结的不错，此部分内容来自他的文章：[ECMAScript 5 Strict Mode, JSON, and More](http://ejohn.org/blog/ecmascript-5-strict-mode-json-and-more/)。

- 变量必须先声明，再使用
```
function test(){
  "use strict";
  foo = 'bar';  // Error
}
```
- 不能对变量执行delete操作
```
var foo = "test";
function test(){}

delete foo; // Error
delete test; // Error

function test2(arg) {
    delete arg; // Error
}
```
- 对象的属性名不能重复
```
{ foo: true, foo: false } // Error
```
- 禁用eval()
- 函数的arguments参数
  - 不能修改arguments
  - 不能在函数内定义arguments变量
  - 不能使用`arugment.caller`和`argument.callee`。因此如果你要引用匿名函数，需要对匿名函数命名。比如：
  ```
  setTimeout(function later(){
    // do stuff...
    setTimeout( later, 1000 );
  }, 1000 );
  ```
  - 禁用with(){}

###浏览器支持
如果浏览器不支持`use strict`，会当它是普通的JavaScript语句：定义一个字符串但没有被引用。因此对其他代码没有任何影响。

当前浏览器对`use strict`的支持情况：  
[ECMAScript 5 Strict Mode Supported by Browsers](http://caniuse.com/#feat=use-strict)

###题外话
实际上，现在大家在项目中大多都使用了JSHint，就不需要在多个地方声明`use strict`了。
如果你还没用过JSHint，请参考[JSHint Documentation](http://www.jshint.com/docs/)。



