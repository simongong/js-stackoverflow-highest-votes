原问题：[How do I get started with Node.js](http://stackoverflow.com/questions/2353818/how-do-i-get-started-with-node-js)

这个问题很基础也很实用，对于一项很火的新技术，如何学？

现在答案中排名第一的我觉得不如排名第二的，拍第二的答案更有针对性和逻辑性。

首先，你需要了解Node.js的核心概念。

* [Node的理念——异步程序](http://blog.shinetech.com/2011/08/26/asynchronous-code-design-with-node-js/)
* [理解Node的核心——Event Loop（异步 != 并发）](http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/)
* [Node中的模块化——CommonJS](http://docs.nodejitsu.com/articles/getting-started/what-is-require)
* [Node API](http://nodejs.org/api/index.html)

然后，你需要去社区看看，[NPM](http://npmjs.org/)是什么。

* [管理项目包依赖的命令行工具](http://docs.nodejitsu.com/articles/getting-started/npm/what-is-npm)
* [Node和NPM是如何一起工作的（node_modules目录和package.json）](http://nodejs.org/api/modules.html)
* [NPM中已有的Node.js包](http://search.npmjs.org/)

最后，你需要知道有哪些工具和利器是你可以利用的。

**编程工具**

* [Underscore](http://documentcloud.github.com/underscore/) 包含几乎所有你需要的核心的工具方法
* [Lo-Dash](http://lodash.com/) 来源于Underscore，可以当它是underscore的增强版
* [CoffeeScript](http://coffeescript.org/) 包含语法糖、更安全的JavaScript
* [JSHint](http://jshint.com/) JavaScript代码检查工具，很多代码编辑器都有对应插件


**代码测试**

* [Mocha](http://visionmedia.github.io/mocha/) 是最受欢迎的测试框架
* [Vows](http://vowsjs.org/) 擅长异步测试
* [Expresso](http://visionmedia.github.com/expresso/) 传统的单元测试框架
* [node-unit](https://github.com/caolan/nodeunit) 单元测试框架

**Web开发框架**

* [Express.js](https://en.wikipedia.org/wiki/Express.js) 目前最火的框架
* [Meteor](http://www.meteor.com/) 打包了jQuery, Handlebars, Node.js, [WebSocket](http://en.wikipedia.org/wiki/WebSocket), [MongoDB](http://en.wikipedia.org/wiki/MongoDB)和DDP
* [Tower](http://towerjs.org/)对Express.js的抽象，目标是做成Ruby on Rails的clone
* [Geddy](http://geddyjs.org/)
* [RailwayJS](https://npmjs.org/package/railway) 带Ruby on Rails影子的MVC框架
* [SailsJS](https://sailsjs.org/#!) 实时的MVC框架
* [Sleek.js](https://sleekjs.com) 基于Express.js简洁web框架
* [Hapi](http://hapijs.com) 以配置为中心的框架，内置支持input validation, caching, authentication等
* [Koa](http://koajs.com/) 由原Express.js团队设计，目标是比Express更好用

**Web模板工具**

* [Jade](https://github.com/visionmedia/jade) HAML/Slim
* [EJS](https://github.com/visionmedia/ejs) 比较传统的模板语言
* [Underscore's template method](http://documentcloud.github.com/underscore/#template)

**Networking工具**

* [Connect](http://www.senchalabs.org/connect/) Node.js中网络程序的核心，相当于Python中的WSGI
* [Request](https://github.com/mikeal/request) 很火的HTTP库.
* [socket.io](https://github.com/LearnBoost/socket.io) 方便地构建一个WebSocket服务.

**命令行交互工具**

* [Optimist](https://github.com/substack/node-optimist) 简化node命令参数的解析.
* [Commander](https://github.com/visionmedia/commander.js)
* [Colors](https://github.com/Marak/colors.js) 美化CLI输出