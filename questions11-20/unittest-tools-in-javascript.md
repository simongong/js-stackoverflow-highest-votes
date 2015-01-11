原问题：[JavaScript unit test tools for TDD](http://stackoverflow.com/questions/300855/javascript-unit-test-tools-for-tdd)

提问者习惯于TDD编程方式，找了一圈JavaScript中的测试工具，没找到一个完美的选项，能使得他继续TDD方式来进行JavaScript编程。

从多个答案中，可以看到对于JavaScript测试工具，的确有很多种选择，应接不暇。不过其中提到的一本书，我想是跟原问题直接相关的：[ Test-Driven JavaScript Development](http://tddjs.com/)。

我还没看过这本书，从目录来看，是包含了TDD理念、工具和方法、JavaScript中TDD的BP这几个方面。所以还是值得一读的。

对于答案中的多种测试工具，因为我自己的使用经验也不是很丰富，无法一一说明清楚。这里也只根据我自己的判断，选择其中几个做简单的列举。大家按需自取。

* [Karma](http://karma-runner.github.io/)
  * 基于Node.js的JavaScript测试平台，主要用来做单元测试
  * 来自于Angular团队
  * 你可以使用任何assertion库
  * 你可以包含其他测试框架（Jasmine, Mocha等）
  * WebStorm, Netbeans等支持
  * 不支持Nodejs程序的测试
  * 没有历史测试结果的记录
* mocha.js
* Sinon
* Buster.js
* TestSwarm