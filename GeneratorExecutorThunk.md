# Generator执行器 - Thunk版
2016/11/26 17:10:00
Javascript


实现一个和`co`功能类似的Generator执行器是需要费点功夫的。然而把要求放宽点，只是实现一个Generator执行器，忽略异常处理等诸多细节，只要求达到「能执行」的效果，那么就相对容易了。

这篇博客将讲述一个最简的执行器，仅用Generator和Thunk实现。是的，仅仅靠Generator和Thunk就能构建一个类似async/await的系统，不需要Promise。


## 对Promise的误解

我在[之前一篇博客][MaybePromiseIsEnough]里提到，`co`和`async`之流仍旧依赖Promise，并且自己也存在缺陷，所以还不如只用Promise。

我错了，`async`的确离不开Promise，这是它的定义决定的。但Generator执行器并不依赖于Promise。`co`只是自己选择了支持Promise，并不是它必须这么做。

这也是Generator相对于`async/await`的一个优点吧。它的灵活性是超过`async/await`的。

发现Generator可以脱离Promise之后，我对它又燃起了兴趣。


## 如此简单

要达到基本的「执行Generator」效果，你只需要这么一个非常简单的函数，实际上它简单到了足以摧毁你对`co`的最后一丝崇拜心理。

```js
function exec(genFn) {
  var gen = genFn()

  function next(e, d) {
    var r = gen.next(d)
    if (!r.done) r.value(next)
  }

  next()
}
```

大功告成！是的，你已经可以使用这个Generator执行器了，它的用法和`co`一样，所以也和`async`一样。


## 使用

我们先定义一个`thunk`版的`readFile`用于之后的测试。关于`thunk`，有兴趣可以看我[这篇文章][Thunk]。

```js
function readFile(fileName) {
  return callback => fs.readFile(fileName, 'utf-8', callback)
}
```

然后就可以测了

```js
exec(function* () {
  var t1 = yield readFile('a.txt')
  var t2 = yield readFile('b.txt')
  console.log(t1, t2)
})
```

结果同预料的一样。


[MaybePromiseIsEnough]: /articles/MaybePromiseIsEnough.html
[Thunk]: /articles/Thunk.html
