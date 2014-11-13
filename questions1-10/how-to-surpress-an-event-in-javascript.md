原问题：[event.preventDefault() vs. return false](http://stackoverflow.com/questions/1357118/event-preventdefault-vs-return-false)

提问者说：在一个事件被触发之后，比如点击按钮事件，会引发注册到该事件的handlers的执行。如果我想在某个handler执行完之后，阻止后续的handlers执行，可以用以下两种方法（基于jQuery）：

* `event.preventDefault()`
```
$('a').click(function (e) {
    // custom handling here
    e.preventDefault();
});
```
* `return false;`
```
$('a').click(function () {
    // custom handling here
    return false;
});
```

那么问题来了，这两种方法有什么区别？哪种更好？

其实除了提问者说的那两种之外，对于这个问题，一般还会提到：

* `event.stopPropagation()`
* `event.stopImmediatePropagation()`

这四种方式在本质上是阻止事件执行，因此我把这篇文章的链接设置为：

#How to surpress an event in javascript?

这个问题不限于jQuery，我自己在前端开发中也为这个问题困扰很久了。

这个问题的根源是**Event**。要解答这个问题，需要弄清楚三个东西：DOM, DOM Event, jQuery Event. 在弄清楚Event的来龙去脉之后，你才会知道这四种操作究竟是干了些啥。

其他参考：

* http://cross-browser.com/talk/event_interface_soup.php
* http://blog.niftysnippets.org/2011/11/story-on-return-false.html
* http://fuelyourcoding.com/jquery-events-stop-misusing-return-false/
* Event _[Doc](https://developer.mozilla.org/en-US/docs/Web/API/Event)_
  * Register Event Listner
  * Event Interface
  * Event Reference _[Doc](https://developer.mozilla.org/en-US/docs/Web/Events)_

Table of Contents:

* DOM
  * DOM Level 1
  * DOM Level 2
  * DOM Level 3
  * 其他DOM
* DOM Event
  * DOM Level 0 Event
  * Flexible Event Model
    * DOM Level 2 Event Model
    * DOM Level 3 Event Model
  * Event Flow
* Surpress an Event
  * by DOM
  * by jQuery

## DOM
DOM是指[Document Object Model](http://www.w3.org/DOM/DOMTR)，其中：

* Document是指XML/HTML文档
* Object是指是把文档封装为一个树形结构的对象，并对外暴露一些对象方法
* Model是指对该树形结构对象的各个子节点都封装为一个模型，并对外暴露一些模型的属性和方法

简单的说，DOM就是对XML/HTML进行封装，然后暴露API。这个API涉及两类实体：

* 浏览器。
  * 根据DOM来展示一个HTML字符串
  * 接收用户操作，触发对应事件
* 程序员。
  * 通过DOM API监听用户动作（事件）
  * 调用DOM API操作DOM

DOM规范是由W3C制定的。尽管DOM通常会使用JavaScript来访问， 但它并不是JavaScript的一部分，它也可以被其他语言使用。

根据这个树形结构的层次，DOM规范中包含Level 1, Level 2, Level 3三个级别。

### DOM Level 1
DOM核心, 提供了一种方式，对基于XML的文档结构的映射，使得文档任何部分的访问和操作都变得很方便； DOM HTML，通过添加HTML专用的对象和方法对DOM核心进行扩展。

### DOM Level 2
DOM Level 2 引入了几种DOM的新方法来处理新类型的接口：

* DOM Views — 描述了跟踪文档的各种视图的接口（就是说，CSS样式之前和之后的文档）
* DOM Events — 描述了事件接口
* DOM Style — 描述了处理基于CSS的样式的接口
* DOM Traversal and Range — 描述了遍历和操作一棵文档树的接口

### DOM Level 3
DOM Level 3 进一步对DOM进行了扩展，引入了使用统一方式加载和保存文档的方法（包含在一个叫做DOM Load and Save的新模块中）。

在Level 3，DOM核心被扩展以支持XML 1.0的所有部分，包括XML Infoset, XPath和XML Base。

### 其他DOM
* Scalable Vector Graphics (SVG)
* Mathematical Markup Language (MathML)
* Synchronized Multimedia Integration Language (SMIL)

本部分参考：[JavaScript Vs DOM Vs BOM, relationship explained](http://vkanakaraj.wordpress.com/2009/12/18/javascript-vs-dom-vs-bom-relationship-explained/)

## DOM Event
DOM Event的规范一直在发展中，现在人们提到DOM Event，通常指的是[DOM Level 3 Events](http://www.w3.org/TR/DOM-Level-3-Events/)。它包含两大部分：

* DOM Event Architecture
* Event Types

DOM Event Architecture中最重要的就是Event Flow，接下来会介绍。Event Types定义了事件类型的集合，具体请查看标准文档，这里不涉及。

### DOM Level 0 Event
[DOM Level 0 Event](http://en.wikipedia.org/wiki/DOM_events#DOM_Level_0)很古老，产生于DOM Level 2 Event Model概念之前，是由浏览器提供的JavaScript接口支持的，没有统一规范。实际上，**Level 0 Event**这个名字也不是正式的，只是依照后来的命名规范取的一个名字，你可以忽略这个名字。

它本质上是DOM节点的一个属性。因此它在类型数量和灵活性上没有后来的 Level 2 Event Model高。定义方式也是通过属性赋值的方式完成。有两种：

* Inline model
```
<a href="http://www.example.com" onclick="triggerAlert('Joe'); return false;">Joe</a>
```
* Traditional model
```
var targetElem = document.getElementById("target");
targetElem.onclick = function() {
  console.log('clicked');
  console.log(this.id); // `this` refers to targetElem itself
};
targetElem.onclick = null;  // Remove a DOM Level 0 Event
```

### Flexible Event Model
为了增加事件模型的灵活性，W3C定义了[DOM Level 2 Event Model](http://www.w3.org/TR/DOM-Level-2-Events/)和[DOM Level 3 Event Model](http://www.w3.org/TR/DOM-Level-3-Events/)。

#### DOM Level 2 Event Model

它有两个目标：

* The first goal is the design of a generic event system which allows registration of event handlers, describes event flow through a tree structure, and provides basic contextual information for each event.
* The second goal of the event model is to provide a common subset of the current event systems used in DOM Level 0 browsers.

DOM Level 2 Event首先是面向DOM提供Event接口，负责DOM事件句柄的注册、事件流的描述和事件上下文的定义。其次是面向浏览器，提供与浏览器的交互能力。其中比较重要的就是我们常用的用于处理注册和删除事件处理程序的操作：addEventListener()和removeEventListener()。

他们的[IDL定义](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-registration)如下：
```
// Introduced in DOM Level 2:
interface EventTarget {
  void  addEventListener(in DOMString type, in EventListener listener, in boolean useCapture);
  void  removeEventListener(in DOMString type, in EventListener listener, in boolean useCapture);
  boolean  dispatchEvent(in Event evt), raises(EventException);
};
```

使用方式如下：
```
var targetElem = document.getElementById("target");
targetElem.addEventListener("click", function() {
  console.log(this.id);
}, false);
```

通过addEventListener()添加的处理程序可以通过removeEventListener()来移除，但需要指明事件处理程序名字。而刚刚那个例子中的处理程序是一个匿名函数，它无法被删除。因此使用addEventListener()的一个好习惯是**给事件处理函数命名**。

```
var targetElem = document.getElementById("target");
var handler = function() {
  console.log(this.id);
};
targetElem.addEventListener("click", handler, false);
targetElem.removeEventListener("click", handler, false);
```

#### DOM Level 3 Event Model
> This specification defines the Document Object Model Events Level 3, a generic platform- and language-neutral event system which allows registration of event handlers, describes event flow through a tree structure, and provides basic contextual information for each event. The Document Object Model Events Level 3 builds on the Document Object Model Events Level 2 [DOM2 Events]

DOM Level 3 Event Model是对Level 2 Event Model的改进和具体化规范。因此重点以它为基础来介绍DOM Event Model。

DOM Event Model主要包括以下关键术语的定义和描述：

* DOM Event Architecture
    * Event Flow
      * Capturing 事件流环节之一。从祖先到EventTarget节点。
      * Hit Target 事件流环节之一，事件到达目标节点。
      * Bubbling 时间流环节之一。由EventTarget节点到祖先。
    * Cancelable
      * 每个DOM Event都有一个默认的处理程序，通常是在注册的处理程序结束之后执行
      * 事件的默认处理程序依赖于DOM实现者（比如浏览器）的具体实现
      * 有些事件的是cancelable的，取消的是DOM对该事件的默认处理
      * 在EventListener中调用`event.preventDefault()`，即可实现取消
* Event Types
  * UI Events 用户操作外围设备而产生的事件。比如鼠标键盘。
  * UI Logical Events
  * Mutation Events 插入删除DOM的时候触发。

这些定义还在[更新中](http://www.w3.org/TR/DOM-Level-3-Events/#changes-DOMEvents2to3Changes)。

### Event Flow
DOM Event Model中最重要的就是[事件流](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-flow)。

* 可以在一个DOM节点上注册任意DOM Event的处理程序 _[Event API](https://developer.mozilla.org/en-US/docs/Web/API/Event)_
* 每个Event对象都有一个EventTarget属性（它指向一个DOM节点）
* 事件被触发后，会被派发到EventTarget，这叫做**event dispatch**
* DOM实现者必须决定派发事件时的传播路径，也就是事件流
* 事件的传播路径有两种选择：Capture和Bubble，这由DOM实现者决定
* DOM实现者应该保证事件能到达EventTarget
* EventTarget节点上对该事件上注册的所有处理程序都会被执行，但它们的执行顺序是不确定的
* 事件流结束条件
  * 所有事件处理程序都执行完毕
  * 事件流中抛出异常

#### Event Capture
定义事件流的走向：事件在到达EventTarget之前，如果其祖先节点上也定义了该事件的处理程序，那么先执行祖先节点的处理程序。每个Event Flow的处理由DOM根节点开始，也就是document。

#### Event Bubbling
与Event Capture相反，该模式的事件流走向：事件先到达EventTarget结点，执行完该结点上的事件处理程序之后，向其祖先结点传递该事件。最后到document结点结束。

假设Event Capture和Event Bubbling都有效，整个事件流的处理分为三个阶段：[capture phase](http://www.w3.org/TR/DOM-Level-3-Events/#capture-phase), [target phase](http://www.w3.org/TR/DOM-Level-3-Events/#target-phase)和[bubble phase](http://www.w3.org/TR/DOM-Level-3-Events/#bubble-phase)。

如下图所示（来自[DOM Event Architecture](http://www.w3.org/TR/DOM-Level-3-Events/#dom-event-architecture)）：
![dom-event-flow](https://farm8.staticflickr.com/7543/15593158310_00fd385447.jpg)

## Surpress an Event
这里指的是终止事件流的执行。

### by DOM
下面我就来看看本文最开始提到的四种方案：

* `return false;`
  * 无效语句，详情见下面的说明。
* `event.preventDefault()` - **Event Context**
  * 是一个Level 2 Events事件，定义在[Level 2 Events-Event](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-Event)
  * 每个DOM Event都有一个默认的处理程序，通常是在注册的处理程序结束之后执行
  * 事件的默认处理程序依赖于DOM实现者（比如浏览器）的具体实现
  * 有些事件的是cancelable的，取消的是DOM对该事件的默认处理
  * 在EventListener中调用`event.preventDefault()`，即可实现取消
  * 默认处理程序只能通过这种方式取消，无法通过下面两种方式取消
* `event.stopPropagation()` - **Event Flow Context**
  * 是一个Level 2 Events事件，定义在[Level 2 Events-Event](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-Event)
  * propagation是指事件传播，stop指的是在某个phase停止
  * 如果在event dispatch之前调用，三个phase都不执行
  * 如果在target phase调用，bubble phase将不执行
* `event.stopImmediatePropagation()` - **Phase Context**
  * 它是由Level 3 Events新添加入Event接口的方法
  * Immediate Propagation是指在事件流的某个阶段中
  * 比如target phase中，注册了多个处理程序，它们按某种顺序执行
  * 如果其中一个处理程序调用了`event.stopImmediatePropagation()`，那么该phase中后续的处理程序将被忽略
  * Spec中没有具体定义该方法的动作，[MDN - Event API](https://developer.mozilla.org/en-US/docs/Web/API/Event.stopImmediatePropagation)中定义了

其中后面三种，是DOM的[Basic Event Interfaces](http://www.w3.org/TR/DOM-Level-3-Events/#event-interfaces)，他们肯定被DOM实现者支持。

对于`return false;`，W3C(Vanilla JavaScript)和jQuery对它的处理是不同的。

* `return false;` by W3C
  * Level 2 Event Model中对[EventListener接口](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-registration)的定义中说明了，注册的事件处理程序`handleEvent`是没有返回值的
  * 使用Vanilla JavaScript在事件处理程序中调用`return false;`将没有任何效果
  * 不排除有些浏览器在实现DOM的时候，没有依照规范来，对`handleEvent`处理了返回值。但这并不可靠，我们在写程序时要避免使用这种方式。

### by jQuery
jQuery基于DOM Level 3 Event对event重新进行了封装，源码[jQuery.Event](https://github.com/jquery/jquery/blob/master/src/event.js#L668)。关于`surpress an event`部分的不同有：

* `event.stopImmediatePropagation()`中会默认调用`event.stopPropagation()`
* 它的handleEvent包含一个返回值[event.result](http://api.jquery.com/event.result/)
* 如果在handleEvent中调用了`return false;`，将会调用`event.preventDefault() && event.stopPropagation()`。参见源码[jQuery - Event Dispatch](https://github.com/jquery/jquery/blob/master/src/event.js#L399)

关于这四种方法在jQuery下的效果和选用，这篇文章以实际例子给出了参考：[jQuery Events: Stop (Mis)Using Return False](http://fuelyourcoding.com/jquery-events-stop-misusing-return-false/)。

结论就是：
* 不要使用jQuery的`return false;`
* 根据你的需要来使用其他三种标准DOM方法，注意jQuery的真实动作
  * `preventDefault()` - 阻止事件默认程序的执行，但不会阻止事件的传播
  * `stopPropagation()` - 阻止事件的传播，但会执行完当前对象上绑定的事件处理程序
  * `stopImmediatePropagation()` - 阻止当前对象上未执行的事件处理程序，同时组织事件的传播
* 如果你想在事件的handleEventA结束后停止任何后续绑定的处理程序，以及DOM默认的处理程序，请手动调用`preventDefault() && stopImmediatePropagation()`
* 把这三种方法的调用放在处理程序的第一行

完结。

