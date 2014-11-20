原问题：[Modify the URL without reloading the page](http://stackoverflow.com/questions/824349/modify-the-url-without-reloading-the-page)

提问者想要修改浏览器地址栏中显示的URL，同时不发生页面重载。

无论是地址栏的内容，还是是否重载页面，都是浏览器的行为。因此我们需要看浏览器是否支持。

我们知道，浏览器与JavaScript之间的接口是window对象。

来看看window对象的文档：[Window - Gecko](https://developer.mozilla.org/en-US/docs/Web/API/window)

目前情况下，该问题只能通过HTML5提供的history接口来完成。

##window.history
HTML5中为window对象添加了history对象。正好上月底W3C正式发布了HTML5规范，我们就来看看正式文档：[HTML5 - History Interface](http://www.w3.org/TR/2014/REC-html5-20141028/browsers.html#the-history-interface) （HTML5目前仍在持续更新中[HTML5 - Draft](http://www.w3.org/html/wg/drafts/html/master/browsers.html#the-history-interface)）。该对象对外暴露了一些属性和方法，让程序员能存取用户浏览记录。

虽然W3C的这个History接口只是在用户浏览历史数据层面上的规范，浏览器可以在实现这些API的时候附加一些自己的动作，比如`history.pushState()`的时候，直接更新地址栏显示的URL。参见Firefox的MDN文档：[Adding and modifying history entries](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history#Adding_and_modifying_history_entries)

它对`history.pushState()`的说明就是：

> This will cause the URL bar to display http://mozilla.org/bar.html, but won't cause the browser to load bar.html or even check that bar.html exists.

因此，对于提问者的需求，在最新的Chrome/Safari/Firefox/IE下，使用`history.pushState()`方法即可实现。

```
var stateObj = { foo: "bar" };
history.pushState(stateObj, "page 2", "bar.html");
```

##不支持HTML5的浏览器
无法实现。

地址栏显示的URL是`document.location`属性。没有修改地址栏内容的API，只能通过给`document.location`赋值的方式，而该赋值动作会导致页面重载。