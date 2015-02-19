原问题：[What is the difference between client-side and server-side programming?](http://stackoverflow.com/questions/13840429/what-is-the-difference-between-client-side-and-server-side-programming)

提问者给了一段代码：

```
<script type="text/javascript">
    var foo = 'bar';
    <?php
        file_put_contents('foo.txt', ' + foo + ');
    ?>

    var baz = <?php echo 42; ?>;
    alert(baz);
</script>
```

问，为什么`file_put_contents('foo.txt', ' + foo + ');`并没有把`bar`写进`foo.txt`，而浏览器端却执行了`alert(42);`。

问题很基础，但却非常典型，因为它是一个web程序的最核心流程。对于web程序员来说，是必须要了解的。

###Web请求处理
典型的web请求是用户在终端浏览器上打开一个url，然后等待一会儿，看到对应的页面内容。

```
                    |
               ---------->
              HTTP request
                    |
+--------------+    |    +--------------+
|              |    |    |              |
|    browser   |    |    |  web  server |
| (JavaScript) |    |    |  (PHP etc.)  |
|              |    |    |              |
+--------------+    |    +--------------+
                    |
  client side       |      server side
                    |
               <----------
          HTML, CSS, JavaScript
                    |
```

解释一下其中的数据流：

* 用户在浏览器输入http://domain.com/a.php，回车
* domain.com对应的web server处理这个http request
  * 找到a.php
  * 调用php-cgi来解析a.php中的php脚本，也就是`<?php //code ?>`
  * a.php中的php脚本将被转换为纯文字，成为页面中其他html/javascript的一部分
  * web server将不包含php脚本的a.php返回给客户端，里面只包含html/css/javascript，这样客户端浏览器可以正常解析
* 客户端浏览器解析a.php中的html/css/javascript代码

###问题解答
我们把原问题对应到上面的数据流，web server端的a.php内容为：

```
<script type="text/javascript">
    var foo = 'bar';
    <?php
        file_put_contents('foo.txt', ' + foo + ');
    ?>

    var baz = <?php echo 42; ?>;
    alert(baz);
</script>
```

web server的php-cgi解析a.php，执行`file_put_contents('foo.txt', ' + foo + ');`时，该php语句中的foo是未定义的，因为php-cgi看不到前面的JavaScript代码。因此这行php代码的执行结果是把`' + foo + '`写到`foo.txt`里。

web server返回给客户端浏览器的a.php的内容为：

```
<script type="text/javascript">
    var foo = 'bar';

    var baz = 42;
    alert(baz);
</script>
```

在进行web应用编程时，一定要分清楚你的脚本是在什么环境下执行。

###扩展 - Isomorphic JavaScript
SPA(Single-page application)应用曾经火过，但因为众所周知的缺陷，导致后来火又灭了。

作为对SPA的改进，提出了[Isomorphic JavaScript](http://venturebeat.com/2013/11/08/the-future-of-web-apps-is-ready-isomorphic-javascript/)的概念，主要就是不单纯由浏览器完成所有工作，服务器端也同样完成业务逻辑和页面渲染。同时前台后台还能共用代码。

这个方向非常不错，但最大的问题，也是跟上面的问题一样，需要在程序中分清楚当前到底是在server side还是client side。

Airbnb在2013年发布了[Rendr](https://github.com/rendrjs/rendr)，它是一个不错的Isomorphic框架，有兴趣的可以试试。




