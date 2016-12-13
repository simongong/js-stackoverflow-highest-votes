原问题：[Why does Google prepend while(1); to their JSON responses?](http://stackoverflow.com/questions/2669690/why-does-google-prepend-while1-to-their-json-responses)

当你调用Google的某个API时，你会发现返回的JSON数据的头部有一个`while(1);`。比如，调用Google Calendar的turn on/off API时的返回数据为：
```
 while(1);[['u',[['smsSentFlag','false'],['hideInvitations','false'],
  ['remindOnRespondedEventsOnly','true'],
  ['hideInvitations_remindOnRespondedEventsOnly','false_true'],
  ['Calendar ID stripped for privacy','false'],['smsVerifiedFlag','true']]]]
```

提问者猜测这是用来防止调用者对返回数据做`eval()`处理，但这有点扯淡，因为你可以手动替换掉这个`while(1)`然后继续你的`eval()`处理。

即便如此，提问者还是觉得，至少，这能让调用者对返回的JSON产生警觉，从而认真写JSON解析代码。

而且，不止是用`while(1)`前缀，Google Docs使用`&&&START&&&`，Google Contacts使用`while(1);&&&START&&&`，Facebook使用`for(;;);`

那么，为什么API要在返回的JSON数据前加一个无限循环语句或者一个奇怪字符串？

这是为了防止[JSON hijacking](http://haacked.com/archive/2009/06/25/json-hijacking.aspx/)。

该攻击是专门针对返回JSON格式数据的API的。

攻击流程示例：
* Google提供一个JSON API：mail.google.com/json?action=inbox，它返回JSON格式的用户收件箱的前50封邮件
* 这个JSON API的初衷只是为了在Google内部站点的前端调用的。但既然包含用户私密信息，难免有恶意网站会盯上
* 由于浏览器的同源策略限制，其他非google.com的网站evil.com无法发起这个AJAX请求
* 但evil.com可以通过包含一个`<script>`等带有`src`属性的标签来指向这个API，这样就达到调用的目的
* 如果同时用户正好访问过Gmail，并且其cookies有效，那么evil.com就讲顺利获得该用户的Gmail邮件数据
* 浏览器对`<script>`标签的处理是：下载，然后作为JavaScript代码执行
* 同时，evil.com在其页面中[重写Object/Array的constructor](http://ejohn.org/blog/re-securing-json/)，该重写的结果是可以在页面中某个对象（array/hash）被赋值的时候，读取该对象的数据。

关键就在重写Array或Object。看一段类似代码：
```
// Defined by malicious site
function Array() {
  var obj = this;
  var ind = 0;
  var getNext = function(x) {
    obj[ind++] setter = getNext;  // Important! where data lead happens.
    if (x) alert("Data stolen from array: " + x.toString());
  };
  this[ind++] setter = getNext;
}

// This is the first step of a typical JSON reponse handling by web browser
// which will trigger `alert("Data stolen from array: private stuff");` defined previously
// which means the JSON data lead happens
var a = ["JSON data"];
```

这段代码重定义了全局的Array对象。重定义的结果是：每当一个property被设置为value的时候，这个value将被`alert(value)`。理论上，恶意代码能够通过这种方式获取到API返回的的JSON数据，然后发送给其他服务器。

可以看到，产生这个漏洞的原因有两个：
* JavaScript允许对全局基本对象（Object/Array/...）重定义
  * ECMAScript4中已修复该问题，不允许重定义全局基本对象，查看[ES4-1.4](http://www.ecmascript.org/es4/spec/incompatibilities.pdf)
* 浏览器解析`<script src="api">`的时候，将下载到的JSON做为一个JavaScript脚本来执行，执行方式就是对一个Object/Array变量的赋值语句
  * 已有人向Mozilla[提交了这个bug](https://bugzilla.mozilla.org/show_bug.cgi?id=376957)，已被修复

这两点完美配合，暴露出了上面这个漏洞。

如果使用`while(1)`或者`&&&BLAH&&&`能够防止这种攻击：
* 通过mail.google.com向该API发起的AJAX请求能正常或者全部返回的JSON数据，并能手动解析，去掉这个前缀
* 返回的这段JSON数据本身，不是一个有效的JavaScript脚本，无法作为JavaScript脚本来被执行
* 其他网站通过`<script src="mail.google.com/json?action=inbox">`方式，当浏览器获取到JSON数据然后把它作为JavaScript脚本执行时，要么将会陷入无限循环`while(1)`，要么爆出语法错误`&&&BLAH&&&`。

这种方法无法防止[Cross-Site Request Forgery (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery
