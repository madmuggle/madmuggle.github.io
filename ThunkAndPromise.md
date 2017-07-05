# 再谈Thunk和Promise
2016/11/29 14:00:00
Javascript


前面两篇文章提到使用Promise和Thunk都能实现一个Generator执行器，这种共同点，总让人想更深入了解，这两者有没有更多相通之处呢？存不存在孰强孰弱？谁好谁坏？

还是从那两个Generator执行器讲起。其实要实现那种执行器，关键在于需要某种机制，把

```js
fs.readFile(fileName, 'utf-8', callbackFn)
```

这样的函数调用，转换成

```js
var t = yield myReadFile(fileName, 'utf-8')
```

也就是让那个回调函数`callbackFn`消失，并且最终结果作为值返还给`t`，从而达到“形如同步”的效果。


## 对比

之前有[一篇博客][Thunk]描述了一个6行的函数`toThunk`，把原始回调API转换成Thunk。其实我当时想写个类似的`toPromise`，动手才发现这个想法不靠谱。

应该说根本写不出来，除非你假设所有的回调函数全部是这样的两个参数形式

```js
function(error, data) {}
```

一旦回调是多个参数，你无法知道哪一个是你需要的数据。比如它是

```js
function(error, d1, d2) {}
```

你是`resolve(d1)`还是`resolve(d2)`？再比如类似`fs.exists`这样的函数，它的回调甚至是单参函数

```js
function(exists) {}
```

所以保险起见，你得为不同的原始回调版API，手动包装一个Promise版的新接口。

Thunk则不同，它只是单纯地把那个回调函数延后了，你最后得把那个回调函数原封不动地传给Thunk。这就是为什么存在通用的`toThunk`。举个例子

```js
myFunc(v1, v2, function(a, b, c) {})
```

等同于

```js
myThunk(v1, v2)(function(a, b, c) {})
```

相应的

```js
myFunc(function(a) {})
```

等同于

```js
myThunk()(function(a) {})
```

这是不是说明Thunk更好呢？

我们再次让主题回到Generator执行器。虽然代码基本相同，但是只有那个[Promise版][gePromise]的执行器才是通用执行器。因为所有的Promise对象对外提供相同的接口。

[Thunk版][geThunk]的执行器，由于`next`函数被定义成了`function next(e, d)`，所以它只能接受特定的thunk。（两个参数，且第一个表示error）

Thunk最终也卡在了Promise一样的问题上。而Promise由于之前已经遭过了“参数数目不定”的罪，这次反而迎来了春天。


## 谁好？

回调的参数没有规则限制，这种不确定性会给自动处理带来障碍。Promise和Thunk只是各自将这种不确定转移到了不同的地方。

所以没有谁好过谁。多么无聊的结论。


[Thunk]: /articles/Thunk.html
[gePromise]: /articles/GeneratorExecutorPromise.html
[geThunk]: /articles/GeneratorExecutorThunk.html
