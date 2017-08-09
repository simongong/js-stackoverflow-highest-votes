原问题： [window.onload vs $(document).ready()](http://stackoverflow.com/questions/3698200/window-onload-vs-document-ready)

这两个在本质上都是DOM document渲染过程中JavaScript的执行时机问题，`$(document).ready()`是jQuery提供的一个简单JavaScript编程接口。

## 客户端JavaScript时间线
《JS Definitive Guide》的13.3.4小节（[链接1](books.google.com/books?id=6TAODdEIxrgC&pg=PA323&lpg=PA323&dq=javascript+definitive+guide+13.3.4&source=bl&ots=oc8mIRKvRE&sig=aU-EfUoHiTZs2ZKoQMHrH97Ivbc&hl=zh-CN&sa=X&ei=YLFuVLO6Gue4mAXfu4DgCw&redir_esc=y#v=onepage&q=javascript definitive guide 13.3.4&f=false)，[链接2](http://yishouce.com/book/1/27899.html)）已经对这部分内容做了详细的说明了。这里的内容参考自此书内容。

在浏览器解析并渲染DOM的过程中，document对象的readyState属性也随之变化.变化过程：

* 浏览器创建document对象，开始解析HTML元素
  * document.readyState = loading
* DOM解析完成，用户看到东西
  document.readyState = interactive
* 所有defer/async JavaScript脚本执行完毕，图片加载完毕
  * document.readyState = complete
* 浏览器开始调用异步事件，以异步响应用户的输入事件、网络事件、计时器过期。

## window.onload
`window.onload`是浏览器提供的事件，它要在网页内所有的元素全部加载完毕后才触发，包括图片和flash等。

对应到上面的客户端JavaScript时间线可知，在`document.readyState = complete`之后会触发该事件。

现在的JavaScript框架大多提供了DOMReady事件代替window.onload事件，但DOMReady并不是window原生事件，而是JavaScript对readyState和window事件进行包装之后提供的简便接口。

比如在jQuery中就是:
```
jQuery(document).ready(callback);
```

`jQuery.fn.ready`的实现本质上也是读取`document.readyState`的状态，当它的值为`complete`的时候就立即执行回调，否则就将回调注册为DOM事件的处理函数。详细请参考源码：
https://github.com/jquery/jquery/blob/10399ddcf8a239acc27bdec9231b996b178224d3/src/core/ready.js#L80
