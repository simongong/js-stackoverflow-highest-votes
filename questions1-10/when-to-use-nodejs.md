原问题：[How to decide when to use Node.js?](http://stackoverflow.com/questions/5062614/how-to-decide-when-to-use-node-js)

要想弄明白什么时候该用Node.js，需要弄清楚这几个问题：
- [Node.js的出现是为了解决什么问题](#problems)
- [什么Node.js](#nodejs)
- [Node.js的特点](#pros-and-cons)
- [Node.js与HTTP Server](#nodejs-vs-http-server)

##Problems
###Web Server的性能问题。
- 并发连接数瓶颈（多进程、线程）
- 扩展只能靠硬件

Node.js的解决办法：不再使用多进程、线程来处理多个web请求，而是使用多事件、队列来处理。这样就不存在资源瓶颈问题。  
Node.js自身保持单进程单线程的运行模式。

###前后端编程语言的障碍
后端语言Java/PHP等，前端语言JavaScript。
- 编译运行环境、语法
- 前后端通信、数据封装解包
- 代码复用
- 工程师沟通障碍

Node.js的解决办法：使用V8引擎、模块化。这样后端应用程序也能使用JavaScript编写，并且代码都是模块化的，在前端也能加载和使用这些模块（通过某些工具）。

##Nodejs
> Node.js® is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

以上内容取自[Node.js官方网站](http://nodejs.org)首页的介绍。很直白，说明Node.js：
- 是一个平台
- 基于Chrome的Javascript runtime (a.k.a. `V8 Engine`)
- 能用来构建快速、可扩展的网络应用
- 使用事件驱动、非阻塞IO模式
- 适合用来构建跨平台、数据交换频繁的实时应用

网站首页下方给出了一个使用Node.js的示例：创建一个WebServer。说明基于Node.js可以实现WebServer的功能，但同时也意味这Node.js不仅仅只能做WebServer的事。

##Pros and Cons
任何一项新技术的诞生，都是为了解决原有技术下无法解决的问题，或者有待改进的方面。同时，新技术的引进又可能会带来新的问题。
[source](http://stackoverflow.com/questions/1884724/what-is-node-js)

###Node.js的优点
Node.js能解决的问题，或者说使用Node.js能带来的好处有：
- 前后端一致的开发环境，工程师顺畅的沟通
- 可扩展的高并发连接能力(scalability of **active connections**)
- 模块化开发，靠社区来扩展功能

###Node.js的不足
目前Node.js存在的问题：
- 对V8的依赖，对Google的依赖 [source](http://www.olympum.com/future/nodejs-to-v8-or-not-to-v8/)  
Node.js需要一个JavaScript引擎，用来负责解释并运行JavaScript程序。而V8已经在Chrome中证明了它是一个高效的JavaScript引擎。同时V8并不依赖于Chrome，它可以单独被嵌入任何程序，依然可以正常工作。因此，Node.js使用了V8，没有V8就没有Node.js现在的性能。
- Debug困难，回调机制的通病 [source1](http://blog.nodejs.org/2012/04/25/profiling-node-js/) [source2](http://mcavage.me/presentations/dtrace_conf_2012-04-03/)
- 对计算密集型请求处理能力较弱  
使用Web Workers模式来解决，开启一个子进程来处理计算密集型任务。
- 异常存活性  
一旦发生异常，会使整个进程崩溃，已经发送到该进程的后续其他请求的处理也就无法继续进行。使用cluster能够一定程度上解决。
- 应对网络攻击  
在应对网络攻击方面还比较脆弱，比如：[CVE-2013-4450](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-4450)

##Nodejs vs HTTP Server
Node.js常常跟HTTP Server一起出现，但这只是其冰山一角，被大家经常看到而已，并不代表它的全部。

在Node.js之前，常见的HTTP Server是Apache，继而出现的Nginx。我们来看看这三个东西到底是什么，各自之间有何异同。

###HTTP Server
HTTP Server本身是一个应用程序，运行在服务器OS上。它的工作就是监听HTTP80/HTTPS443端口，然后：

- 接收客户端发来的HTTP请求
- 分析请求内容
- 获取url对应的服务器端的实际资源
- 将数据包装成HTTP响应，返回给客户端

理论和实际都很简单，你完全可以对照[Hypertext Transfer Protocol -- HTTP/1.1](http://tools.ietf.org/html/rfc2616)自己动手写一个HTTP Server。使用任何语言都行，只要它能：

- 监听系统端口
- 访问系统资源
- 发送数据

也就是说，只要这种编程语言能够访问系统资源，能够进行socket网络通信，就能用来写一个HTTP Server。

####Apache
A stable and scalable HTTP Server, but poor ability in terms of concurrency.

####Nginx
A high performance HTTP Server.

Apache和Nginx完成了HTTP Server的功能，你部署并运行它，你就得到了一个监听着80端口的HTTP Server, which means a ready to running environment out of box.

###Nodejs
> Web is a first class citizen in Node, designed with streaming and low latency in mind. This makes Node well suited for the foundation of web library or framework.

可以看出，HTTP Server只是Node.js能够做的事情的一部分而已。从实际程序设计和实现上看，Node.js：
- 是用python和C++写的一个运行环境（因为V8是用C++写的） [源码](https://github.com/joyent/node/tree/master/src)
- 提供用JavaScript写的一些访问系统资源的node模块 [源码](https://github.com/joyent/node/tree/master/lib)

比如我们常常使用的[http模块](https://github.com/joyent/node/blob/master/lib/http.js)，使用它就能构建一个HTTP Server，运行在服务器上监听用户请求。

Node.js只是提供了构建HTTP Server的素材，你需要自己编程去实现这个HTTP Server，当然是比较容易地就能完成。

Node.js is a ready to programming environment.

同时，除了HTTP server之外，Node.js还能做很多其他的事，任何web-based application都行。比如使用socket.io这个模块，你能实现websocket协议通信，而非HTTP通信。

事实上在很多情况下，Node.js的确是被用作一个更广泛的应用服务器，同时与HTTP Server共存：
- 编写多个Node.js应用运行在不同端口，提供不同的服务
- 利用Nginx的high performance HTTP Serving特长，通过Nginx来反向代理到80、443端口

Node.js这时候扮演的是应用服务器的角色。

##Summaries
Node.js是一个能在浏览器之外、普通OS上运行JavaScript的平台。  
它的核心是：事件驱动、一切皆异步；一切皆JavaScript。能最大化利用资源，能消除前后端在编程语言上的障碍。  
你可以用它来构建一个Web Server，还可以用它来做其他任何有趣的事。

现在整理一下SO原问题的答案。

当你的web application需要同时保持多个活动连接active connections，并且每个连接的处理并不是计算密集型，不用消耗太多的CPU时间，可以考虑使用Node.js。

比如:
- Browser-based chat application
- Online game
- Collaboration tools
- Twitter队列  
每秒成千上万条的tweet要写入数据库，只能排队完成。传统的Apache+PHP的方式，需要使用memcached之类的缓存插件来处理这个队列。而Node.js直接就可以完成，它的较色就是收集tweet并将信息传递个另一个负责数据库写入的进程。  
看上去功能好像差不多，但区别在于Node.js能够更简单与优雅地完成任务。

从以上应用场景可以看出，Node.js主要是扮演信息收集者的角色：收集信息，发送任务；收到任务执行结果，返回结果。具体的任务，是由其他Node.js模块或者其他应用程序完成。

传统Web服务器是“一个连接一个线程”，独享独占；Node.js采用的是“一个连接一个流程”的方式，排队共享。它只负责流程的启动和结束处理，流程中的其他环节，由程序员完成。

最后再次提醒两点：
- 多上[Node.js官网](http://nodejs.org)，那里有你需要的一切
- 如果你决定使用它，确保你已经对它有充分地了解
