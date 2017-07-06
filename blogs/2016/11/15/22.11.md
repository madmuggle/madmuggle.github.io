# Generator，Python和JS
2016/11/15 22:11:00


最早接触到`yield`这个关键字是在Python里面，伴随着Generator和Comprehension了解到的。看到的例子，都是使用yield来进行一些简单的循环，保持中间状态等等。

起初没觉得它多重要，以为只是个另外一个普通的语法糖。而且怪异的流程让我不喜欢它，心想这种事不用它也能实现，而且代码可能还更好看。

后来接触了ES6和`co`，我才意识到这是个了不得的东西。


## Python的Generator

Python对「Comprehension」的支持（即用字面量创建对象）非常友好，相同规则的语法可以用在很多地方，比如：

List Comprehension

```python
[ i * 2 for i in range(10) if i ** 2 < 10 ]
#=> [0, 2, 4, 6]
```

Set Comprehension

```python
{ i * 2 for i in range(10) if i ** 2 < 10 }
#=> {0, 2, 4, 6}
```

Dict Comprehension

```python
{ k.upper(): v for k, v in { "a": 1, "b": 2 }.items() }
#=> {'A': 1, 'B': 2}
```

Generator
```python
( i * 2 for i in range(10) if i ** 2 < 10 )
#=> <generator object <genexpr> at 0x7f9c53a99410>
```

其中Generator最为特殊。


## Generator的意义

不同于List/Set/Dict Comprehension，Generator的最大意义不是提供了语法糖，而是提供了一种全新的流程控制方案。

Generator还有另外一种创建方法。先定义一个特殊的函数

```python
def blah():
  for i in range(10):
    if i ** 2 < 10: yield i * 2
```

然后就可以用如下方法产生一个Generator

```python
blah()
#=> <generator object blah at 0x7f9c53a99410>
```

Generator以一种怪异的方式运作，我们可以通过它最重要的一个方法`__next__`来一窥究竟：

```python
a = blah()
a.__next__()
#=> 0
a.__next__()
#=> 2
a.__next__()
#=> 4
a.__next__()
#=> 6
a.__next__()
#  Traceback (most recent call last):
#    File "<stdin>", line 1, in <module>
#  StopIteration
```

理解Generator的一个关键，是要明白`yield`是一种特殊的`return`。

虽然在Python里，使用同样的语法`def name():`来创建它们，但「Generator function」和「function」其实是两种区别很大的东西。


调用「Generator function」返回的是一个Generator对象，而不是那个“函数”的执行结果。


## Javascript的Generator

引入Generator的时候，Javascript定义了一个新的关键字`function*`，两者就被显式地区分开了。你一眼就能注意到，这不是一个普通的函数。

我觉得这是少数Javascript做得比Python好的地方。

```js
function* blah() {
  for (var i = 0; i < 10; i++)
    if ((i * i) < 10) yield i * 2
}
```

和Python一样，通过一个特定的方法（next）来使用

```js
a = blah()
//=> Generator {  }
a.next()
//=> Object { value: 0, done: false }
a.next()
//=> Object { value: 2, done: false }
a.next()
//=> Object { value: 4, done: false }
a.next()
//=> Object { value: 6, done: false }
a.next()
//=> Object { value: undefined, done: true }
```


## next的参数

Generator的`next`方法支持参数这一点非常重要，它是让Generator能够包装异步调用的基础特性。JS里面，继续使用`next`就行了。

```js
function* blah() {
  for (var i = 0; i < 10; i++) {
    var t = yield i * 2
    console.log('t is ', t)
  }
}
```

传递参数给`next`，就仿佛是赋值给那个`yield`表达式一样。

```js
a = blah()
//=> Generator {  }
a.next("a")
//=> Object { value: 0, done: false }
a.next("b")
//   t is  b
//=> Object { value: 2, done: false }
a.next("c")
//   t is  c
//=> Object { value: 4, done: false }
```

Python的`__next__`不支持这种特性

```python
def blah():
  for i in range(10):
    t = yield i * 2
    print('t is ', t)
```

当你给`__next__`传递参数的时候

```python
a = blah()
a.__next__(1)
#  Traceback (most recent call last):
#    File "<stdin>", line 1, in <module>
#  TypeError: expected 0 arguments, got 1
```

然而Python的Generator有个叫做`send`的方法，它可以完成这个工作。`send`必须传一个参数，而且初次调用必须传None。所以细节上和JS有点区别，但大体上是一致的。

```python
a = blah()
a.send(None)
#=> 0
a.send("b")
#   t is  b
#=> 2
a.send("c")
#   t is  c
#=> 4
```

至于为什么不把`__next__`和`send`合并为一个就不得而知了。不过对应Generator的行为，取名叫`send`的确更合语境。


## 异常

对于Generator，除了`next`，最重要的一个方法就是`throw`了，这个方法给了我们把外部的异常送进Generator内部的能力。让异常看上去像`yield`引起的。

这是一种非常特殊的能力。通常异常只能从“被调用者”传向“调用者”，但对于Generator，你可以作为“调用者”把异常传给“被调用者”。

Javascript和Python在异常的处理这块出奇的一致，名字都一样使用`throw`。

为了方便，先写一个永远返回"ok"的Generator函数

```js
var cnt = 0

function* blah() {
  while (true) {
    try { yield ++cnt }
    catch (e) { console.log(`error: ${e.message}`) }
  }
}
```

现在对它做点测试

```js
a = blah()
a.next()
//=> Object {value: 1, done: false}
a.next()
//=> Object {value: 2, done: false}
a.next()
//=> Object {value: 3, done: false}
a.throw(new Error("haha"))
//   error: haha
//=> Object {value: 4, done: false}
a.throw(new Error("haha"))
//   error: haha
//=> Object {value: 5, done: false}
a.next()
//=> Object {value: 6, done: false}
```

Python如何呢，同样的实验

```python
cnt = 0

def blah():
  global cnt
  while True:
    try:
      cnt += 1
      yield cnt
    except Exception as e:
      print("error:", str(e))
```

同样的测试

```python
a = blah()
a.__next__()
#=> 1
a.__next__()
#=> 2
a.__next__()
#=> 3
a.throw(Exception("haha"))
#   error: haha
#=> 4
a.throw(Exception("haha"))
#   error: haha
#=> 5
a.__next__()
#=> 6
```
