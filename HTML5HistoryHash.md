# HTML5的history
2016/11/17 18:32:00
HTML, Javascript


## Hash是唯一的选择吗？

一直以为单页应用的路由只能够通过`Hash`来实现，即网页地址里带`#`的那种写法。因为它的是一个纯粹的浏览器方的概念，Hash属于URL的一部分，但是它的改变不会导致重新向服务器发起请求。

说得更直白点，实际上服务器根本不知道Hash的存在，因为浏览器在发送请求的时候，直接把Hash去掉了。这个可以通过浏览器的开发者工具看到。

修改`window.location.href`也能够改变地址栏内容，然而一旦这个变了，浏览器必定重新向服务器发起请求并更新页面，也就不是“单页”了。

另外平时见到的单页应用全是用Hash的，比如「[网易云音乐][163music]」，因此我根本没多想就默默地下了结论：Hash是单页应用的基础。

但是今天亲眼见到了一个不使用`Hash`的SPA，然后才意识到Hash并不是唯一方法。


## HTML5的history

修改`window.location.href`的确无法保持在当前页，这一点没有变。但是修改浏览历史有别的途径，这是我忽略的。关键就在HTML5的`history`对象。

以前我只知道通过`history`的某些方法，可以实现JS来控制前进后退历史。却没有注意到，`history`还提供了修改历史的方法：`history.pushState`


## 一个小实验

在浏览器的`console`里面运行下面几句JS代码，观察浏览器URL的变化。

```js
history.pushState({name: "a"}, '', "/my/path/1");
history.pushState({name: "b"}, '', "/my/path/2");
history.pushState({name: "c"}, '', "/my/path/3");
```

现在点击「后退」按钮，可以看到能退回到之前的URL。整个过程中页面没有任何变化，没有新的请求被发出。

这就是不用`Hash`，却依旧能做单页应用的基础。


## 这么方便，为何不都用它?

相比使用`Hash`，这种做法还是有缺陷。`Hash`的改变，可以通过监听`"hashchange"`事件知道，很容易实现跟服务端一样的路由机制。而`非Hash`部分的改变无法监听。

所以这种SPA说白了，是一种「欺骗手段」，因为页面的跳转不是由URL变化触发的，而是改变页面内容后，顺带修改一下URL和历史。


## 限制

```js
history.pushState({name: "a"}, '', "http://a/b/c");
//=> SecurityError: The operation is insecure.
```

跨域自然是不允许。


[163music]: http://music.163.com/
