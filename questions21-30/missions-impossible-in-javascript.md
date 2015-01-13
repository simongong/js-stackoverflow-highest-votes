这里收集一些浏览器环境中的使用JavaScript无法实现的功能或需求（as of 2015/01/13）。

## DOM节点上绑定的event listener
原问题：[How to find event listeners on a DOM node?](http://stackoverflow.com/questions/446892/how-to-find-event-listeners-on-a-dom-node)



## 在新tab里打开一个URL
原问题：[Open a URL in a new tab using JavaScript](http://stackoverflow.com/questions/4907843/open-a-url-in-a-new-tab-using-javascript)

提问者想用JavaScript实现：在浏览器新开一个标签，打开一个URL。

类似地已有的方案可以实现在一个浏览器窗口打开一个URL：

```
window.open(url,'_blank');  // 新开一个窗口
window.open(url);   // 当前窗口
```

注意，这是纯靠JavaScript实现的，不需要用户任何点击鼠标的操作。

实际上，这个需求目前是无法实现的，因为：

* ECMAScript规范中对浏览器动作也没有做任何规定
* window对象的API是由浏览器决定的
* tab的操作，也是由浏览器决定的

而目前主流浏览器的tab实现机制并不一样，因此不会有统一的API来实现这个功能。

