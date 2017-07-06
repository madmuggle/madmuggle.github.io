# The Trick of SPA Router Without Hash
2017/04/20 10:08:00


之前有[一篇文章][old]提到了使用与不使用Hash实现SPA的情况。但对不使用Hash的情况下如何实现路由并没有给出细致的解释，或者说没有点到关键。

其实我也是昨天和同事讨论React-Router的实现，才慢慢理清楚关键点的。今天上午还特地写了一个简单的[demo][demo]来验证我的想法。

先说结论，实现不用Hash的SPA router，关键就在于几点：

- 不要使用HTML默认的a表示链接，要用div或span配合onclick来替代a。这样做的目的是在跳转前进行pushState操作。
- 页面的刷新或者链接跳转后，永远从最顶层开始重绘页面。
- 捕获到popstate事件时，也触发顶层重绘。

除了那篇文章提到的history.pushState函数，完成这种效果还需要依赖另外一个特性：浏览器后退前进按钮的点击事件是可以被监听的。

这里有个细节，就是只有调用过pushState之后，popstate事件才能被监听到，否则就是正常的后退前进功能。这就是为什么不能用普通的a元素的原因之一：你要在点击链接后，先pushState，然后再重绘页面。

进一步的细节从我的那个demo里可以了解到。那个demo最大的优点是没有使用任何第三方库，代码量超小，容易看清主要逻辑。

[old]: /HTML5HistoryHash.html
[demo]: https://github.com/madmuggle/SimpleSPARoute

