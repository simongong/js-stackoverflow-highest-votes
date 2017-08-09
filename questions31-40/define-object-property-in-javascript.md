原问题：[How to use javascript Object.defineProperty](http://stackoverflow.com/questions/18524652/how-to-use-javascript-object-defineproperty)

提问者说，他对这一段代码感到不解：
```
Object.defineProperty(player,"health",{get:function() {
     return 10 + ( player.level * 15 );
}});
```
不解的原因是，他本来就可以直接访问`player.level`，为什么要使用`Object.defineProperty`定义一个属性来做这样的事。他觉得这是多此一举，希望有人给他讲讲这样做有什么理由。

## Object Property

对于JavaScript变量来说，正常的属性添加通过赋值来创建并显示在属性枚举中（for...in 循环 或 Object.keys 方法）。这种方式添加的属性值可能被改变，也可能会被 删除。该方法允许改变这些额外细节的默认设置。
```
var obj = {}
obj.a = 1;
obj.b = 2;
var keys = Object.keys(obj);
console.log(keys);

delete obj.b;
for(var key in obj){
    console.log(key + ':' + obj[key]);
}
```

### defineProperty
我们先来看这个方法的定义：[Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。

```
Object.defineProperty(obj, prop, descriptor)
```
来看这两个参数：
- prop
  - 需被定义或修改的属性名。
- descriptor
  - 需被定义或修改的属性的描述符。

对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。

数据描述符也就是我们常见的属性的值。它是一个拥有可写或不可写值的属性。存取描述符是由一对 getter-setter 函数功能来描述的属性。描述符必须是两种形式之一；不能同时是两者。也就是说，要么为prop指定数据描述符——赋值，要么为其指定存取描述符——指定getter/setter。

MDN链接页面里给出了很多具体的使用实例，请认真看。

总之，就是你可以使用`Object.defineProperty(obj, prop, descriptor)`方法来为obj定义一个属性的值，以及这个属性的描述符。

实际使用中，我一般是用它来设置一个类的只读属性和可读写属性。

```
// Read-Only properties
Object.defineProperty(this, 'data', { get: function() { return _data; } });

// Read-Write properties
Object.defineProperty(this, 'data', {
    get: function() { return _data; },
    set: function(value) { _data = value; }
});
```