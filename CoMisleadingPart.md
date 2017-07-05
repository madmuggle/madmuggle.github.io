# co真正的误导之处
2016/12/24 22:08:00
javascript


用法上[`co`][co]和`async`非常相似，唯一区别出现在当你在使用和不使用`yield`/`await`的时候。这个区别掩盖了一些关键细节，导致`co`的某些行为比较费解。


## 真正的误导之处

`async`的用法比较单纯，不管是不是“阻塞调用”，即前面有或没有`await`，都只需要这样去调用它：

```js
asyncFn()
```

而`co`，“阻塞调用”时，即有`yield`时，是
```js
genFn()
```

和`async`相同。但没有`yield`的时候，你却需要写成

```js
co(genFn)
```

其实这是一个误导，因为正常情况下应该一直是`co(genFn)`。有问题的是`yield`后面居然不需要这样写。


## 为何称之误导

因为Generator函数和Async函数有很大区别，async函数被调用的时候，函数内的表达式会自动执行。而当你写出一个Generator函数：

```js
function* genFn() {
  console.log('...............')
}
```

然后你想执行它怎么办

```js
genFn()
```

这是错的，genFn里面的表达式没有被执行，要执行里面的内容你需要
```js
genFn().next()
```

所以要执行它，必须要又一个“执行器”（比如`co`）来递归调用它的`next`。因此每次执行都应该是这样的

```js
co(genFn)
```

实际上，你可以把每个`async`版的`fn()`，对应到一个`co`版的`co(fn)`

```js
asyncFn()
```

等同于

```js
co(genFn)
```

而

```js
await asyncFn()
```

等同于

```js
yield co(genFn)
```

那为什么`yield`后面的Generator Function不需要这样写，而是可以直接：

```js
yield getFn()
```

这是误解，因为在`co`内部，其实进行了“特殊处理”。


## 原因分析

通过`yield`返回的对象，如果发现是`Generator`或`Generator Function`，`co`会对它进行这样的处理：

```js
if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj)
```

而在另一个地方，Generator Function会被调用，转成Generator：

```js
if (typeof gen === 'function') gen = gen.apply(ctx, args)
```

所以其实下面三种写法，效果是相同的：

```js
yield co(genFn)
```

```js
yield genFn()
```

```js
yield genFn
```


## 利与弊

`await`后面必须是Promise对象，`async function`调用后返回的也是Promise对象，这种特性让async-await系统非常清晰。

相比之下，`co`的`yield`后面除了可以接受Promise，还可以接受Generator, Generator Function, Thunk, Array甚至是Object。这导致了`co`的行为更加复杂。

因此现实情况就是，如果要用好`co`，你需要花一些时间去了解`co`是如何实现的。

但是这个复杂并不绝对是坏事。

假如`co`严格要求`yield`后面必须是Promise对象，那么理解起来会很容易，但是你却需要针对不同的函数用不同的写法。

如果一个函数返回Promise对象，你要写成

```js
yield fn()
```

如果一个函数是Generator Function，你要写成

```js
yield co(fn)
```

这两种函数通常是同时存在的，并且辅助函数可能在两类间转换，这个在你定义了很多异步辅助函数的时候是非常头痛的。

由于`co`针对不同的`yield`做了很多事，所以现实中我们不会碰到这个问题，不用担心把Generator Function改成返回Promise的时候，需要修改一大堆的`yield`。

现在我觉得`co`的做法，是利大于弊的。

至于极少数的非阻塞调用情况，就只能注意下，写成

```js
co(fn)
```

然后期待`async`普及开的那一天。


## 题外话

其实`async`系统可以构建在Generator上面，只需要增加一个小机制。

当匹配到代码里面的函数调用的时候，即发现`fn()`的时候，对`fn`的类型做个检查。如果`fn`是普通函数，就照常调用它，如果是`async`函数，就变换成`co(fn)`的形式。

有了上述基础，对`async function`这种新语法，可以简单替换成`function* `，然后做一点限制。相应的`await`直接替换成`yield`。

不知道实际JS引擎，是不是这样实现async的。


[co]: https://github.com/tj/co/blob/master/index.js
