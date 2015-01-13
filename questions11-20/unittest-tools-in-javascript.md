原问题：[JavaScript unit test tools for TDD](http://stackoverflow.com/questions/300855/javascript-unit-test-tools-for-tdd)

提问者习惯于TDD编程方式，找了一圈JavaScript中的测试工具，没找到一个完美的选项，能使得他继续TDD方式来进行JavaScript编程。

从多个答案中，可以看到对于JavaScript测试工具，的确有很多种选择，应接不暇。不过其中提到的一本书，我想是跟原问题直接相关的：[ Test-Driven JavaScript Development](http://tddjs.com/)。

我还没看过这本书，从目录来看，是包含了TDD理念、工具和方法、JavaScript中TDD的BP这几个方面。所以还是值得一读的。

对于答案中的多种测试工具，因为我自己的使用经验也不是很丰富（只用过Karma, Mocha, Sinon），无法一一说明清楚。这里也只根据我自己的判断，选择其中几个做简单的列举。大家按需自取。

* [Karma](http://karma-runner.github.io/)
  * 基于Node.js的JavaScript测试平台，主要用来做单元测试
  * 来自于Angular团队
  * 你可以使用任何assertion库
  * 你可以包含其他测试框架（Jasmine, Mocha等）
  * WebStorm, Netbeans等支持
  * 不支持Nodejs程序的测试
  * 没有历史测试结果的记录
* [mocha.js](http://mochajs.org/)
  * 能输出test coverage
  * before/after/before each之类的钩子挺方便
  * 其他就不细说了，测试框架该有的都有，没有明显缺陷
* [Sinon](http://sinonjs.org/)
  * 三个主要功能：stub, spies, mock
  * 如果你的JS代码中有HTTP请求，很大可能需要在unittest中用它
* [Buster.js](http://docs.busterjs.org/en/latest/overview/)
  * 基于Node.js的JavaScript测试平台
  * 内置了assertion库，但你也可以使用其他的
  * 内置了SinonJS
  * 支持[浏览器测试](http://docs.busterjs.org/en/latest/browser-testing/)
    * 自带server，支持在浏览器中进行自动化测试
    * 支持PhtantonJS的headless测试
  * 支持Node端的测试
  * 缺点是目前还是beta版，可以等待稳定版出来后再用
* [TestSwarm](https://github.com/jquery/testswarm/)
  * 全面，支持几乎所有的浏览器和操作系统
  * 方便，多客户端同时运行，无需另开server端
  * 不支持Ant/Maven集成，测试失败不会中断build过程
  * 没有IDE插件支持

我之前项目使用的单元测试工具组合是这样的：
* Karma配置：
```
  config.set({
    frameworks: [ 'mocha' ],
    browsers: [ 'PhantomJS' ]
  });
```
* Assertion使用chai.should()和chai.expect
* HTTP请求使用sinon
