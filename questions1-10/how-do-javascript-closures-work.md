原问题：[How do JavaScript closures work?](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)

> If you can't explain it to a six-year old, you really don't understand it yourself.  
*-- Albert Einstein*

如何简洁生动的说明JavaScript中的**closures**是什么。

## [给6岁孩子讲故事](http://stackoverflow.com/a/7285658/1295057)
从前，有一位公主
```
function princess(){
```
生活在无聊的世界里，周围都是年长者。她喜欢冒险。有一次不小心走到一个新的世界，在那里有会说话的动物，有独角兽，还遇见了她的白马王子。公主过得很开心，见到听到很多有趣的事。
```
    var adventures = [];
    function princeCharming() { /* ... */ }
    var unicorn = { /* ... */ },
        dragons = [ /* ... */ ],
        squirrel = "Hello!";
```
但她仍然不得不回到原来的世界里去，那里有她的家人和朋友们。
```
    return {
```
回去的时候，公主把在新世界认识的朋友和新动物都忘记了，只有那些有趣的事仍在记忆中。回去之后，公主会给家人和朋友们讲她在那个新世界里有趣的事。
```
        story: function() {
            return adventures[adventures.length - 1];
        }
    };
}
```
但家人和朋友们毕竟没有自己去过，他们所见到的，只是一个小女孩
```
var littleGirl = princess();
```
在讲一些有意思的故事。
```
littleGirl.story();
```
他们能够知道那些故事，也能够确认小女孩的存在。但他们并不确认那些故事是否真的存在，因为他们看不到。

这个小公主就是一个闭包。
而非闭包的一般函数，跟小公主唯一的区别就是：回到原来的世界时，会把在新世界发生的时都忘记。

## [给程序员讲理论](http://stackoverflow.com/a/111111/1295057)
首先先简要总结闭包特性：

* 函数的局部变量在函数返回之后仍然可用
* 栈上的内存空间在函数返回之后仍在存在，不被回收

给个例子。下面这段代码会返回一个函数的引用：
```
function sayHello2(name) {
    var text = 'Hello ' + name; // Local variable
    var sayAlert = function() { alert(text); }
    return sayAlert;
}
say2 = sayHello2('Bob');
say2(); // alerts "Hello Bob"
```
对于这段代码，C程序员可能会认为`sayAlert`和`say2`一样，都是指向一个函数的指针。但实际上它俩有一个重要区别： 在JavaScript中，你可以认为一个函数的指针变量同时拥有两个指针。一个指向这个函数，另一个隐藏的指针指向一个闭包。

> 重点在于你的函数内是否引用的外部变量。

在JavaScript中，如果你在一个函数内定义一个新的函数，那么这个新的函数就是一个闭包。
对于C或者其他高级语言，函数执行结束并返回之后，它所占用的栈空间将被释放回收。函数内定义的局部变量将不再可用。但在JavaScript中，并不这样。如上所示，函数执行结束后，它所占用的栈空间并不会被全部回收。

上面是基本理论。更进一步，再来一个例子：
```
function say667() {
    // Local variable that ends up within closure
    var num = 666;
    var sayAlert = function() { alert(num); }
    num++;
    return sayAlert;
}
var sayNumber = say667();
sayNumber(); // alerts 667
```
这个例子说明：闭包中使用的函数局部变量并非是值拷贝，而是引用。`say667()`执行结束之后number所在的那块内存的值为667，而`sayNumber()`是在`say667()`执行结束之后才执行，当它访问number所在的内存时，结果自然也是667。

再进一步，看看用closure时易发生的错误的例子：
```
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + list[i];
        result.push( function() {alert(item + ' ' + list[i])} );
    }
    return result;
}

function testList() {
    var fnlist = buildList([1,2,3]);
    // Using j only to help prevent confusion -- could use i.
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}
```

> 时刻保持清醒：变量是在内存里的，闭包使用的是内存的引用而不是那块内存的值拷贝。

当你在循环中定义函数（闭包）的时候得小心，它可能并不像你最开始想的那样工作。关键有两个：

- 子函数使用的是外部函数的局部变量的引用。
- 循环内只是**定义**了子函数，并没有**执行**这个字函数。

最后，来一个最抽象的例子：
```
function newClosure(someNum, someRef) {
    // Local variables that end up within closure
    var num = someNum;
    var anArray = [1,2,3];
    var ref = someRef;
    return function(x) {
        num += x;
        anArray.push(num);
        alert('num: ' + num +
            '\nanArray ' + anArray.toString() +
            '\nref.someVar ' + ref.someVar);
      }
}
obj = {someVar: 4};
fn1 = newClosure(4, obj);
fn2 = newClosure(5, obj);
fn1(1); // num: 5; anArray: 1,2,3,5; ref.someVar: 4;
fn2(1); // num: 6; anArray: 1,2,3,6; ref.someVar: 4;
obj.someVar++;
fn1(2); // num: 7; anArray: 1,2,3,5,7; ref.someVar: 5;
fn2(2); // num: 8; anArray: 1,2,3,6,8; ref.someVar: 5;
```
这个例子说明，闭包的创建时机是在函数被调用的时候。每次函数调用都会生成一个新的闭包，也就是一块新的内存区域。因为函数每次调用都会新分配一块栈内存，这是一回事。

最后我自己来总结一下闭包：

- 函数的局部变量在其他地方被引用
- 闭包有两种基本情况：闭包的返回值是一个函数，它其中使用了该闭包的局部变量；闭包内定义了内部函数，内部函数引用了闭包的局部变量
- 每次函数调用，都会生成一个新的闭包，分配新的内存
