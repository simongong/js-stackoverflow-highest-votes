这个主题不是vote最高问题列表中的，是因为我今天碰到了一个数值转换问题，然后意识到这个问题很典型，可以整理写一篇。

随后在StackOverflow上手动搜了一下，有这几个相关问题：
- [How to convert decimal to hex in JavaScript?](http://stackoverflow.com/questions/57803/how-to-convert-decimal-to-hex-in-javascript)
- [What's the best way to convert a number to a string in JavaScript?](http://stackoverflow.com/questions/5765398/whats-the-best-way-to-convert-a-number-to-a-string-in-javascript)
- [How do I convert a float to a whole number in Javascript?](http://stackoverflow.com/questions/596467/how-do-i-convert-a-float-to-a-whole-number-in-javascript)

搜索的关键字有： `convert hex`, `convert float`, `convert number`, `bitwise operator`

## JavaScript中的数字

首先，JavaScript中没有primitive type，全部number都是float型，其类型定义为[Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)。

### 整数判断

判断一个数字是否是整形，是一个常见的需求。以前要靠程序员自己来hack，现在EMCAScript6中加入了[isInteger()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/isInteger)

但目前不是所有浏览器都实现了这个方法，MDN给了一个Polyfill：

```
Number.isInteger = Number.isInteger || function(value) {
    return typeof value === "number" && 
           isFinite(value) && 
           Math.floor(value) === value;
};
```

### 数字解析
正如其他语言的primitive type的包装类一样，ECMAScript6对Number也提供了parse方法，能够完成字符串和数字之间的解析。

包括：
- Number.parseInt(string[, radix])
  - 它与window.parseInt是完全一样的，`Number.parseInt == parseInt; // true`
- Number.parseFloat(string)
- Number.prototype.toString([radix])

## 数值转换
与数字有关的转换包括： float与int之间的转换，各进制之间的转换，数字与字符串之间的转换。

### 浮点数和整数转换
JavaScript中浮点数与整数的转换方式有多种。具体参见： [jsPerf demo](http://jsperf.com/float-to-int-conversion-comparison/19)
- `number | 0;`
- `~~number`
- number >> 0
- Math.floor(number)
- `parseInt(number, 10)`

### 进制转换
用[Number.toString(radix)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toString)方法可以实现将一个数字转换成radix进制。

### 转成字符串
将一个数字转换为一个string，有多种方法。 各种方法的速度比较，参见[jsPerf demo](http://jsperf.com/number-to-string/29)
一般使用这两种：
- `'' + num`
- `num.toString()`

### Bitwise操作
Bitwise是对int型的位操作。

由于JavaScript中没有int型，因此，JavaScript中的bitwise操作需要对操作数先进行float -> int转换，然后进行bitwise操作，因此效率比C语言中的位操作要低。

> In Java, the bitwise operators work with integers. JavaScript doesn't have integers. It only has double precision floating-point numbers. So, the bitwise operators convert their number operands into integers, do their business, and then convert them back. In most languages, these operators are very close to the hardware and very fast. In JavaScript, they are very far from the hardware and very slow. JavaScript is rarely used for doing bit manipulation.

摘取自[JavaScript the good parts](https://www.safaribooksonline.com/library/view/javascript-the-good/9780596517748/apbs08.html)

实际上，JavaScript中的[Bitwise Operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators)依然是可以使用的，并且效率不是那样的糟糕（执行效率依赖于浏览器的实现）。参见本文浮点数转整数中的例子。


综合上面的操作，举个栗子：
```
var num = (new Date().getTime() + Math.random() * 16) % 16; // get a random float between [0, 16]
console.log(num);
console.log(num.isInteger());
num = num | 0;  // convert float to int via bitwise operation
console.log(num);
console.log(num.isInteger());

console.log(num);
var hexString = num.toString(16);   // convert decimal to hex
console.log(num);

var = parseInt(hexString, 16);  // convert hex to decimal
console.log(num);
