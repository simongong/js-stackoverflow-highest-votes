原问题：[What is DOM Event delegation?](http://stackoverflow.com/questions/1687296/what-is-dom-event-delegation)

这个问题是我偶然碰到的，不是投票排名很高的问题，但我觉得还蛮重要的。

## 基础

### Event Bubbling
要理解JavaScript中的Event Delegation，首先需要了解Event Bubbling。

之前在[event.preventDefault() vs. return false](https://github.com/simongong/js-stackoverflow-highest-votes/blob/master/questions1-10/how-to-surpress-an-event-in-javascript.md)这个问题中已经对JavaScript事件模型说得比较细了。其中的Event Flow有三个部分：

* Capturing。事件从祖先到EventTarget节点。
* Hit Target。事件到达目标节点。
* Bubbling。事件由EventTarget节点到祖先。

注意这个[Bubbling](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-flow-bubbling)，它就是JavaScript Event Delegation的关键。即：发生在DOM节点上的事件会向上传递到祖先节点。

### Event Delegation
Event Bubbling为Event Delegation提供了基础，它使得你可以：

> 把若干个节点上的相同事件的处理函数event listener绑定到它的父节点上去，在父节点上统一处理，减轻对event listener的管理负担。

这就是事件委托。通常用在一组元素上。比如`<ul>`节点下的一组`<li>`元素。

## 应用

举个栗子。

一个博客的首页，展示前6篇博客的列表。代码结构如下：

```language-html
<ul id="parent-list">
    <li id="post-1">Post 1</li>
    <li id="post-2">Post 2</li>
    <li id="post-3">Post 3</li>
    <li id="post-4">Post 4</li>
    <li id="post-5">Post 5</li>
    <li id="post-6">Post 6</li>
</ul>
```

假如用户在点击列表中每篇文章的时候我们需要用JavaScript做点儿什么，比如展示一个什么效果，发起一个ajax请求等等。有两种做法：

* 在每个`<li>`元素上绑定click事件
* 在`<ul>`元素上绑定click事件

如果你使用第一种，想象一下，万一将来你要给这个首页列表添加**lazy loading**功能，你还要逐个去处理新添加元素的event listener。你知道，添加和移除event listener的工作是不好管理的，很痛苦的。

更好的方案是使用第二种：把event listener放在父元素上。

### 为什么能这么做

* 前面提到的Event Bubbling保证了`<ul>`元素上的事件会传递到`<ul>`元素
* event listener中传入的`event`对象中包含了事件发生的真实节点——`<li>`的引用，你可以通过`event`对象访问到`<li>`节点的所有信息

### 如何实现

在`<ul>`节点上添加event listener：

```language-javascript
// Get the element, add a click listener...
document.getElementById("parent-list").addEventListener("click", function(e) {
    // e.target is the clicked element
    // If it was a list item
    if(e.target && e.target.nodeName == "LI") {
        // List item found.  Do whatever you want.
        console.log("List item ", e.target.id.replace("post-"), " was clicked!");
    }
});
```
通过这种方式，`<li>`节点可以任意添加，也不用担心event listener的添加和移除问题了。

### 更广泛的应用

上面的例子是简单场景下的示范：子元素都是`<li>`类型。

其实这个**Group**的概念可以是不同元素的集合。比如：

  * 一个`<div>`下有不同类型的子元素
  * 各个子元素的`class`属性会根据场景而改变
  * 我们要处理的对象是`<tag class="classA">`的点击事件，即带有`classA`样式的节点的点击事件

你无法对某个具体的节点去绑定click event listener，因为你不知道到底哪个节点会满足条件。当然你还可以对所有元素绑定，但你知道这很笨。*（看到重点了没：一组元素都绑定类似的event listener。这时候，就可以考虑使用Event Delegation）*

这种情况下，使用Event Delegation更方便。

```language-javascript
// Get the parent DIV, add click listener...
document.getElementById("myDiv").addEventListener("click",function(e) {
    // Get the CSS classes, e.target was the clicked element
    var classes = e.target.className.split(" ");
    if(classes) {
        for(var x = 0; x < classes.length; x++) {
            if(classes[x] == "classA") {
                // Bingo!
                console.log("Target element clicked!");
            }
        }
    }
});
```

以上。
