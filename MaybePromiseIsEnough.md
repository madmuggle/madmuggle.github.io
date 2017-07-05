# Promise也许足够了
2016/11/20 13:17:00
Javascript


我在[这篇博客][ES6_Promise]里面，提到了`Promise`的一个小缺陷，即中间变量无法共享的问题。这个缺陷通过`generator`配合`co`，或者ES7的`async-await`，都是可以解决的。主流观点把它们看作异步的趋势。可能会吧，但是我觉得这类「伪同步」并没有传说中那么美好。

前几天，我将自己之前用Promise实现的程序，改写成用generator配合co实现。改写后能正常工作，但也仅此而已。并没有当初切换到Promise后的那种轻松感。我越来越觉得，不管是co还是async，这种做法其实是有点问题的。


## 恐怖谷

要说是什么问题，我觉得最合适的比喻就是[「恐怖谷理论」][uncanny_valley]了。它们很像同步，却又在某些地方，表现得跟同步很不一样。


## 举个例子

我们先定义一个延时函数，它返回一个Promise对象，以供`yield`使用。

```js
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}
```

定义一个使用`sleep`的`function*`，命名为`fn`

```js
function* fn(name) {
  console.log(`${name} starting...`)
  yield sleep(1000)
  console.log(`${name} finished`)
}
```

再定义一个使用`fn`的函数

```js
function test1() {
  fn('the first')
  fn('the second')
  fn('the third')
}
```

```js
test1()
//=> undefined
```

没有任何输出。那我们换个用法，一股脑全加上`co`

```js
function test2() {
  co(fn, 'the first')
  co(fn, 'the second')
  co(fn, 'the third')
}
```

```js
test2()
//   the first starting...
//   the second starting...
//   the third starting...
//=> undefined
//   the first finished
//   the second finished
//   the third finished
```

有输出，但不是我们想要的输出。因为`co`返回的是一个Promise对象，Promise对象并不会造成类似「阻塞」的效果。

要证明这一点，你可以这样做

```js
function test3() {
  co(fn, 'the first')
  .then(() => co(fn, 'the second'))
  .then(() => co(fn, 'the third'))
}
```

```js
test3()
//   the first starting...
//=> undefined
//   the first finished
//   the second starting...
//   the second finished
//   the third starting...
//   the third finished
```

这下对了。但这种用法完全比不上纯用Proimse，写在这里只是帮助理解`co`的运行机制。同时还想说明，要理解内部机制，依旧避不开Promise。

真要用`co`，你得把所有涉及到异步的函数都写成`function*`，它具有「传染性」。这一点ES7的async也不会改观，`await`和`yield`一样，不能出现在普通函数里面。

```js
function* test4() {
  yield fn('the first')
  yield fn('the second')
  yield fn('the third')
}
```

```js
co(test4)
//   the first starting...
//=> Promise { <pending> }
//   the first finished
//   the second starting...
//   the second finished
//   the third starting...
//   the third finished
```

结果正确了。

这个例子很简单，你可以简单地全加上`yield`。但是当普通函数和这类「特殊函数」混杂在一起，相互调用的时候，你需要思考哪些函数应该写成`function*`或`async`，哪里应该加上`yield`或者`await`。


## Why ?

归根结底，用co或者async，就是为了采用「同步思维」，然而它们并不能完全实现这个美好的愿景。不管包装得多严实，同步和异步就是不相容。你总会在某个地方，处理一个很别扭的东西。

而Promise没有模仿同步，只是别出心裁地组合代码，你不会用同步思维去处理Promise。说不定只用Promise是更好的选择。


## 后记

(2016/12/24)

由于工作需要，每天都要使用`co`，时间久了上面提到的很多问题都不觉得是问题了。关于它的「传染性」，似乎也不觉得有什么关系了。

最主要的是自从读懂了`co`的源码之后，对它的排斥心里慢慢就消散了。


[ES6_Promise]: /articles/ES6_Promise.html
[uncanny_valley]: https://en.wikipedia.org/wiki/Uncanny_valley
