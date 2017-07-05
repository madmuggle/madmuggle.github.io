# 自己动手实现ES6 Promise
2016/10/27 13:10:00
Javascript


## 为什么要自己实现Promise ？

异步编程的重心早已转移到async上，Promise第三方实现众多，且已被主流平台原生支持，在这种前提下，为什么还要自己动手实现Promise呢？

首先，ES7的async依赖于Promise。理论上这不是必须的，thunk可以在这种地方取代Promise，但在可预见的未来，Promise都会是async的根基。

另外，Promise不需要编译器或解释器的支持。async和generator无法在稍微旧一点的环境上运行，除非先用babel之类工具编译一次。

而Promise，即使在非常老的JS运行时上，你也可以使用Promise。JS一开始就是词法作用域，支持闭包，而这就是Promise需要的全部了。

所以深入理解Promise是非常有用的，就算上面说的你都不会碰到，实现它的过程，也是对编程能力的一次历练。何乐而不为呢。


## 如何实现 ？

对于Promise这种代码量不大，但是行为复杂的程序，最好的学习方法是直接看代码。只看别人的解释可能会弄得似懂非懂。

先放一个[我自己的实现][myPromise]以供参考。

建议还搜一下其他人的实现。一开始不要去找那种“大而全”的，找那种“小”的实现，甚至是那种功能不全的，这种实现最容易帮助理解核心思路。


## 一些关键点

Promise最核心的方法是`then`，实现它就实现了Promise的大半。再把`all`和`race`实现，基本上就实现了完整的Promise，剩下的都是简单封装。

关于`then`，这个最核心的方法，实现前最好对它的使用有一定了解，不然可能会走一些弯路，产生不必要的疑惑。

每次调用`then`，都是在创建一个新的Promise对象。`then`就像一个锁链一样，将前后的两个`Promise`对象连接起来。

为了突出这一点，我在自己的实现里面，特意写成这种形式：

```js
Promise.prototype.then = function(resolveFn, rejectFn) {
	var pP = this
	return new Promise((res, rej) => thenHandler(res, rej, resFn, rejFn, pP))
}
```

调用`Promise.all`也会返回一个新的Promise对象。后面的`then`是挂在这个新的Promise对象上的。

```js
Promise.all = function(promises) {
	checkArray(promises)
	return new Promise((res, rej) => handleAll(promises, res, rej))
}
```


## pending, fulfilled, rejected

理解Promise的另一个关键点，在于理解它的“状态”。一个Promise对象有三种状态：pending, fulfilled, rejected

为什么要有状态？为了分情况处理。先举一个例子，假如我定义一个Promise，但是不给它绑定`then`：

```js
var x = myReadFile("/tmp/text.txt")
```

这条语句在运行的时候，那个文件的内容其实已经在某个时间点读出来了。一直缓存在某个地方。吃完饭我们再来运行：

```js
x.then(console.log)
//> Promise { <pending> }
//  blahblah..., the content of /tmp/text.txt
```

正确地读出了文件数据。

这是因为`then`会对Promise对象的状态进行判断：
- 如果是`pending`，就把`then`传入的函数存到Promise对象里。
- 如果是`fulfilled`，就直接调用`then`传来的函数。

这就是状态存在的主要意义。状态的存在的另外一个意义是处理`resolve`和`reject`函数多次调用的情况。这种情况很少见，初步实现暂可忽略。

然后就可以直接看代码了，需要的就是适当的耐心，乐观的态度～

如果读者对Promise不了解，想知道它运行的一些特点，可以继续往下看。


## 使用示例

接下来，我们通过两个例子来探索Promise的运行过程，为了减少重复代码，我们先定义一个函数，这个函数会返回一个Promise对象，这就是一切的开端。

```js
function myReadFile(filename) {
	return new Promise((res, rej) => {
		fs.readFile(filename, (e, d) => e ? rej(e) : res(d.toString()))
	})
}
```

使用Promise，代码经常看上去会是这个样子的

```js
myReadFile("theFirst.json")
.then(JSON.parse)
.then(fn1)
.then(fn2)
.then(fn3)
.catch(console.error.bind(console))
```

上面这个例子中，真正能够异步的，只有第一步而已。一旦那个`resolve`被调用，后面的一连串都会顺着执行。就像多米诺骨牌一样。

那么如果中间有另一个地方需要异步怎么办呢，比如我需要读取另外一个文件？ 你只需要在某个传给`then`的函数里面，返回一个新的Promise对象就行。

```js
myReadFile("theFirst.json")
.then(JSON.parse)
.then(fn1)
.then(d => myReadFile("theSecond.json"))
.then(JSON.parse)
.then(fn3)
.catch(console.error.bind(console))
```

上面这个例子中，`fn3`处理的是`theSecond.json`文件的内容。

这一点可能理解上有点别扭，大概需要实现了Promise之后，才能清楚其中的猫腻。

关于Promise，我能想到的需要注意的，暂时就这么多了。以后如果有新的点子，会继续放上来。


## Promise的缺陷

我对Promise非常喜爱，但是它的确有缺点，比如一些中间变量无法共用，我们拿`同步`例子来做个对比

```js
var a = readFileSync("blahblah.txt")
var b = fn1(a)
var c = fn2(b)
var d = fn3(c)
console.log(a, b, c, d)
```

而使用Promise的异步则是这个情况

```js
myReadFile("blahblah.txt")
.then(fn1)
.then(fn2)
.then(fn3)
.then(d => console.log(d))
```

其中`fn1`的返回值只有`fn2`能获取，`fn2`的返回值只有`fn3`能获取…… 如果需要像同步版本那样，获取所有中间值，就必须把它们存为全局或者上层闭包变量。

但是这个小小的缺点不太要紧。瑕不掩瑜，Promise彻底解决了`callback hell`，让我对Javascript另眼相看。


## 后记

随着对Promise使用时间的增长，我意识到了Promise的一些其他优点。它不仅仅是解决了回调的“代码金字塔”问题。

应该说，回调带来的真正问题，并不是代码不停往右边延展，而是你不能以正常的「函数」概念来思考问题。

什么是函数？我记得以前数学老师向我们解释：函数就是一个工厂里的机器，你给一个毛胚进去，它加工出一个产品。

当时我还没有编程的概念，当我学会编程之后，更加赞同那个朴实的比喻了。函数这东西，就是做转换，它不仅要有输入，还要有输出。

而回调是“没有”输出的。它有输出，但不是作为返回值呈现。你需要编写一个函数，然后想象这个函数会以上个函数调用的结果为输入参数，自动被调用。

这很别扭，不是因为没用习惯，而是因为需要多转好几个弯。

Promise重新赋予了我们“自然”的函数，这是它更重要的意义。


[myPromise]: https://github.com/madmuggle/SimplePromise

