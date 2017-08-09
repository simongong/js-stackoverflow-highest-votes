原问题：[How does Facebook disable the browser's integrated Developer Tools?](http://stackoverflow.com/questions/21692646/how-does-facebook-disable-the-browsers-integrated-developer-tools)

提问者用Chome打开Facebook时，发现Developer Tools里的Console被禁用了。
![chrome-dev-tools-at-facebook-page](http://i.stack.imgur.com/Wiatp.png)

甚至连Chrome console的自动提示功能都失效了。

![chrome-dev-console-auto-complete-disabled](http://i.stack.imgur.com/j0Zmx.png)

他很好奇是怎么做到的。

现在排名第一的答案，正是来自于当时做这件事的Facebook工程师。我们可以从他的回答中了解到一些原委。

* [为什么禁用Chome Console](#why-disable-chrome-console)
* [如何做到的](#how-to-make-it)

## Why Disable Chrome Console
这是为了防止[special social engineering attack](https://www.facebook.com/photo.php?v=956977232793) _(Facebook链接，需翻墙查看)_

### Social-Engineering Attack
上面链接里的视频很好的解释了这个问题。考虑到有些朋友可能打不开那个页面，我在这儿就复述一遍视频里讲的。

#### Share Baiting
钓鱼分享：做一个网友可能会感兴趣的视频或页面，诱发他分享到Facebook。其实这个分享链接是个钓鱼链接。
#### self-XSS
他的好友在Facebook上看到了他的这个分享，点击链接后发现需要做一些操作才能查看：

* 点击当前页面地址栏
* 按键盘的`J`键
* 按`Ctrl+v`粘贴，回车

然后就会发现好友分享的那个视频里什么也没有，反而自己的Facebook自动发布了一些广告或其他信息。这就是self-XSS攻击。

[Self-XSS](https://www.facebook.com/help/246962205475854) 又称为跨站脚本攻击，设计用于诱使您泄露 Facebook 帐户访问权限。如果诈骗人获得您帐户的访问权限，他们可以用您的名义发布内容或评论。要避免 Self-XSS 攻击，切勿复制并粘贴可疑链接。

### Why Chrome
因为Chrome是唯一能受到这种攻击的主流浏览器。IE9/Firefox都能有效防止这类攻击。

### Why disable console
常见的XSS攻击是类似这样：
```
// Normal request URL
http://host/request?q=xxx
// Request URL with XSS-attach
http://host/request?%q=3Cscript src=siteB/evil.js%3E%3Cscript%3E
```

防止XSS攻击是web server端程序要做的，因为一般是表单提交或者http request中产生XSS。但self-XSS攻击脚本的引入是用户自己完成的，脚本的下载和执行是Chrome完成的，Facebook后端处理post或者http request的程序根本没参与这个过程，什么也做不了。

但Facebook总得做点儿啥，不然用户可能认为这是它的bug，或者它做的不够才让spam盛行。在搞清楚这种攻击的执行原理之后，决定从Console下手，使得恶意脚本无法自执行。

但考虑到影响范围，Facebook只是选择一部分用户来启动这种应对手段。可能是筛选出可能受到这种攻击的用户。

## How To Make It
前面说了，搞Console的目的是使得恶意脚本无法自执行。

Chome把所有console里的代码都封装成这样来运行：
```
with ((console && console._commandLineAPI) || {}) {
  <code goes here>
}
```

### Command Line API
我们来看看这个`console._commandLineAPI`是什么。

简单的说，就是Chrome为其console提供了若干个接口，你在console里可以直接调用它。比如：`copy() && clear() && profile()`。

前面的self-XSS的原理之一，就是利用了`console._commandLineAPI`。

不只是Chrome，[Firefox](http://code.google.com/p/fbug/source/browse/branches/firebug1.7/content/firebug/commandLine.js?r=8810#1169)/[Safari](http://opensource.apple.com/source/WebCore/WebCore-7533.18.1/inspector/InjectedScriptHost.cpp)也同样提供这些console接口*（IE还不清楚）*。只是Safari/Firefox提供的接口比较有限，避免了这类sel-XSS攻击。

因此Facebook做的，就是hack掉`console._commandLineAPI`，重新定义它。

Nexflix[也做了类似的手段](http://stackoverflow.com/a/22216648/1295057)。

Facebook的hack大概长这样
```
Object.defineProperty(window, "console", {
    value: console,
    writable: false,
    configurable: false
});

var i = 0;
function showWarningAndThrow() {
    if (!i) {
        setTimeout(function () {
            console.log("%cWarning message", "font: 2em sans-serif; color: yellow; background-color: red;");
        }, 1);
        i = 1;
    }
    throw "Console is disabled";
}

var l, n = {
        set: function (o) {
            l = o;
        },
        get: function () {
            showWarningAndThrow();
            return l;
        }
    };
Object.defineProperty(console, "_commandLineAPI", n);
Object.defineProperty(console, "__commandLineAPI", n);
```