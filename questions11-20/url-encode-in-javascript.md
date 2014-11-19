原问题：[Best practice: escape, or encodeURI / encodeURIComponent](http://stackoverflow.com/questions/75980/best-practice-escape-or-encodeuri-encodeuricomponent)

当向HTTP服务器发送查询请求时，URL中的查询参数query string该如何处理？

也就是URL encode问题。

##为什么要URL encode
为什么要对查询参数进行encode？因为URL标准规定。参考：[RFC1738 - Uniform Resource Locators (URL) - 2.2](http://www.ietf.org/rfc/rfc1738.txt)。至于URL标准为什么这么规定，从RFC看来可能的理由是：兼容性和安全性。

HTML文档接受的字符集为Unicode。而URL字符串接受的字符范围为ASCII中的一部分：

> "...Only alphanumerics [0-9a-zA-Z], the special characters "$-_.+!*'()," [not including the quotes - ed], and reserved characters used for their reserved purposes may be used unencoded within a URL."

因此，出现在HTML文档中的URL都应该被encode。

具体参考这篇文章：[URL Encoding](http://www.blooberry.com/indexdot/html/topics/urlencoding.htm)

##Percent-encoding
URL encoding是采用包含百分号的编码形式，也叫做Percent-encoding。看wikipedia，直接了当：[Percent-encoding](http://en.wikipedia.org/wiki/Percent-encoding)

##JavaScript中如何URL encode

* escape()
  * 从ECMAScript3开始已被废弃，别用
* encodeURI()
  * 当你想得到一个有效的URL时，使用它
  * 它会将一些无效字符转化为Percent-encoding。比如
```
// return `http://www.google.com/a%20file%20with%20spaces.html`
encodeURI("http://www.google.com/a file with spaces.html")
```
  * 别对一个URL字符串使用`encodeURIComponent()`，因为它是暴力转换
```
// return `http%3A%2F%2Fwww.google.com%2Fa%20file%20with%20spaces.html`
encodeURI("http://www.google.com/a file with spaces.html")
```
* encodeURIComponent()
  * 仅适用于对URL的参数部分进行转换
```
param1 = encodeURIComponent("http://xyz.com/?a=12&b=55");
url = "http://domain.com/?param1=" + param1 + "&param2=99";
http://www.domain.com/?param1=http%3A%2F%2Fxyz.com%2F%Ffa%3D12%26b%3D55&param2=99;
```
  * 不转换单引号字符'，存在起代码注入漏洞，比如`href='my_url'`尤其在使用字符串构建HTML的情况下。因此，对于HTML属性值最好用双引号或者先HTML encoded一遍

