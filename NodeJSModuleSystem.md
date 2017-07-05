# Node.js的模块系统
2017/02/23 16:14:00
Javascript


## Implementation

关于NodeJS的模块系统，之前有写过相关的博客，比如[require参数][require]，[模块入口][package]的。那些都是关于模块系统的使用方式，不涉及实现原理。

这篇博客就是对之前主题的一个补充。通过运行一些简单的例子，揭示Node.js的模块系统是如何运作，如何建立名字空间的。


## Inspection

要探索Node.js的模块系统，并不一定要去看源代码，有更简单的方式。我们先来创建一个“有问题”的模块：新建一个`a.js`文件，里面的内容如下：
```js
0_0
```

没错，这是一个不合格的JS代码文件。那么require它的时候是什么结果呢？

```js
require('./a')
```

错误信息如下：
```
(function (exports, require, module, __filename, __dirname) { 0_0
                                                               ^^
```

最关键的一点揭晓了：Node.js直接把模块的内容，也就是JS文件里的代码，全部塞进了一个函数里面。和前端常用的做法一模一样。

也就是说，在Node.js里面，名字空间的产生，依旧是通过闭包完成的，并没有引入语言级别的新特性。

上面的例子还揭示了另一件事：`__filename`, `__dirname`是传给模块的参数，标准的JS函数传参，并不是编译器的黑魔法。

再次感觉到闭包真的是非常重要的特性，虽然概念简单，却总能玩出花样。之前[实现Promise][promise]也是重度依赖闭包。

很喜欢Node.js这种做法，在不增加语言特性的情况下，就实现了模块系统，干净利落。


[require]: /NodeJSRequire.html
[package]: /PackageJsonAndNpm.html
[promise]: /ES6_Promise.html
