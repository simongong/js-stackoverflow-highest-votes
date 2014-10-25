JSONP出现的场合，一般都有AJAX和JSON，它的出生晚于AJAX。

尽管多种名词术语混杂，我们从这几个问题来看JSONP：

* [背景](#background)
* [解决什么问题](#problem)
* [如何解决](#solution)

就能弄清楚JSONP，分清楚它和其他名词的关系。

##Background
###AJAX
首先我们要了解[AJAX](#https://developer.mozilla.org/en-US/docs/AJAX)：

1. Asynchronous JavaScript + XML
2. 不是一种新技术，而是一种解决方案，能够实现对web页面*增量更新但无需重载*
3. 该方案是多种现有技术的结合（2005年），核心是[XMLHttpRequest object](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

XMLHttpRequest是一个JavaScript对象，最早由微软设计，后被Mozilla/Apple/Google采用。该JavaScript对象提供了一种简单地方式，实现向一个目标URL发送HTTP GET请求，获取到数据，同时不会重载当前页面。

由于XHR它对HTTP是一种很好地补充，目前[正在被W3C标准化](https://xhr.spec.whatwg.org)。

常见的AJAX使用场景是：
> 1. 通过JavaScript向某个remote API获取数据  
2. 按需求提取并组织数据  
3. 操控当前页面的DOM，将数据显示。

###Same Origin Policy
为了防止[CSRF攻击](https://www.owasp.org/index.php/Cross-Site_Request_Forgery)，Netscape Navigator于1995年提出了[同源策略](http://en.wikipedia.org/wiki/Same-origin_policy)的概念。

该策略就是，对于一个web页面，只允许与其**同源** *(scheme://hostname:port)*的脚本操作该页面的DOM。

同源策略也对后来出现的XHR对象也适用，导致原生的XHR对象无法向非同源的URL发请求。

目前所有[现代浏览器都执行着这个同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。

但在很多合法情况下，也会产生跨源JavaScript请求。有需求就有方案，绕过同源策略的方式有iframe, Web Proxy, [Cros-Origin Resource Sharing](http://www.w3.org/TR/cors/)等。

##Problem
由上面背景情况介绍可知，AJAX通信的两个主要问题是：

1. 以何种格式来交换数据
2. 如何解决跨域请求

对于问题1，从AJAX名字可知，最早是以XML格式来交换数据的。但现在被广泛采用的时JSON格式。

对于问题2，如上所讲，现在有多种方式来绕过浏览器的同源策略。JSONP就是在AJAX下绕过同源策略的一种方式。

##Solution
[JSON-P](http://json-p.org)是一种轻量的数据交换格式。详细请查看其链接。

其实JSONP是一种hack，它基于浏览器现在对于拥有`src`属性的标签的处理：自动下载并解析。比如`<script>, <img>, <iframe>`等标签，它们被浏览器赋予了拥有跨域的权利。

然后我们通过一个示例，来看JSONP是怎么利用这个浏览器*后门*来实现跨域AJAX hack的。

* 在remote.com 上有remote.js文件，内容如下：
```
callback({"result": "data from remote.com"});
```
* 我们本地的jsonp.html页面中包含了如下JavaScript代码：
```
  <script type="text/javascript">
  var callback = function(data){
      alert('receive data from remote:' + data.result);
  };
  </script>
  <script type="text/javascript" src="http://remote.com/remote.js"></script>
```
* 在浏览器打开jsonp.html，我们会看到弹出窗，内容为：

> receive data from remote: data from remote.com

看到没，利用`<script src="remote.js">`不受同源策略限制的后门，达到向remote URL获取数据然后更新本地页面DOM的目的。这就是JSONP的核心。

由于它涉及了请求和响应两方面的一些规范，我们可以把JSONP看做是一个机制。该机制的一个要点就是允许用户传递一个callback参数给服务端，然后服务端返回数据时会将这个callback参数作为函数名来包裹住JSON数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。

###与普通JSON响应的区别
JSONP与常见的AJAX+JSON有些许区别：

* 客户端定义一个callback
* 服务器返回的数据与callback相关，数据本身是一句JavaScript函数调用的代码：`callback(data)`，该函数的参数`data`就是JSON数据

##总结

1. 传统的Web API只能在后端用Python/PHP/Java调用，因为JavaScript有同源策略限制
2. 为了让AJAX也能跨域，产生了JSONP
3. 工作原理：将API URL设置成`<script>`标签的`src`值或者使用jQuery的`$.getJSON(url, callback)`方法。
4. JSONP API接受一个callback参数，返回的response形如：`callback({jsondata});`，这样就相当于在客户端加载了一段js代码，这个代码是执行一个名为callback的方法，并以一个JSON对象作为参数。而在客户端提前定义好了该方法，因此就实现了用JavaScript跨域调用远程API获取到数据从而处理数据的目的。
4. 从原理可知，JSONP是一种通信机制，需要服务端和客户端配合。过程中使用了JSON作为数据交换格式。

Flickr的API就提供JSONP格式的响应。可以看看它的响应数据的结构：
https://www.flickr.com/services/api/explore/flickr.photos.search
