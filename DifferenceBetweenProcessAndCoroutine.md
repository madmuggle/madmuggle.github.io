# 进程，线程和协程的区别
2017/01/05 13:53:00


## OS Process, Thread

在不同的操作系统上，“进程”和“线程”的定义不尽相同，但总体上遵循类似的标准。写这篇文章的时候，我脑袋里只有Windows和Posix的概念，其他的平台情况可能不同。

“进程”间的空间是独立的，“线程”间则共享内存空间。但它们俩有个非常重要的共同点：在调度上，切换是强制性的，且不可预测。

由于这个共同点，本文将操作系统的“进程”和“线程”放在一起说归为一类。


## Erlang Process

Erlang的“进程”相互之间不共享内存，跟操作系统的“进程”一样。进程切换也是VM强制进行，任何一行代码都可能被打断，用户无法控制，这一点又和操作系统进程一样。

所以这里把它同操作系统进程归为同一类。


## Goroutine

有人把Go语言的Goroutine归为协程，因为Go的调度系统并不像Erlang或者操作系统，只有在发生Syscall的时候，它才会进行调度。

但是Go语言里面有没有任何地方显式进行类似`yield`的操作，所以其实并不是“用户在控制调度”，因此也很难归到“协程”。

大概也正是这个原因造就了Goroutine这个词吧。


## Coroutine

“协程”这个概念出现的比较早，很多语言都对协程提供支持，除了很早就看到的Python，还有Lua和如日中天的Javascript。(Generator和Async-await都可以算作典型的协程)

“协程”最大的特点就是：调度是由用户自己完成的。

JS的协程里，“用户进行切换”这一点非常明显，就是显式地使用`yield`/`await`关键字：

```js
function* f(id) {
  for (var i = 0; i < 3; i++) {
    console.log(`<${id}>: i is now ${i}`)
    yield sleep(0)
  }
}
```

其中`sleep`的定义如下：

```js
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}
```

启动两个协程来运行它：

```js
co(f, 'a')
co(f, 'b')
```

结果在意料之中：

```
<a>: i is now 0
<b>: i is now 0
<a>: i is now 1
<b>: i is now 1
<a>: i is now 2
<b>: i is now 2
```

即使再怎么包装，那个`yield`/`await`关键字也是省不掉的，所以在JS里面，哪里会交出控制权是一目了然的。


## Gevent

然而Python的协程库Gevent情况又不同了，它没有显式地使用`yield`之类关键字。取而代之的是普通的函数调用。这就打乱了上面对协程的认识。

我们写一个类似上面JS的例子来做实验。

```py
def f(id):
  for i in range(3):
    print('<{}>: i is now {}'.format(id, i))
    gevent.sleep(0)
```

启动两个协程来运行`f`：

```py
a = gevent.spawn(f, 'a')
b = gevent.spawn(f, 'b')

a.join()
b.join()
```

输出的结果是：

```
<a>: i is now 0
<b>: i is now 0
<a>: i is now 1
<b>: i is now 1
<a>: i is now 2
<b>: i is now 2
```

结果和上面JS的例子完全一样。但没有了`yield`关键字，在不了解库的情况下，我们无从知道哪一步会交出控制权，进行调度。

尽管表现更像Goroutine，大家依旧都称Gevent为“协程”。


## 放宽定义？

或许我们可以放宽协程的定义，把非显式的用户切换也算到协程里面。但是这样的话，恐怕Goroutine也得叫协程了吧。

不知道Go语言的拥趸嫌不嫌弃呢？
