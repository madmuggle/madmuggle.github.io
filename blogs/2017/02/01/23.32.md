# Koa的中间件体系
2017/02/01 23:32:00


## Middleware

在Node.js的众多Web框架中，[Koa][koa-home]大概是最有个性的一个了。它跟其他框架最大的不同，在于对中间件的处理方式，其他部分并无大异。

传统意义上的中间件，每一个middleware就像一个滤网，或者说一道工序，一个完了之后才能开始另外一个。这种做法逻辑清晰，只是不够漂亮。

某些任务，例如要记录服务器某些操作耗时时间，传统的中间件你只能小心控制全局状态，比如数据库或者全局变量。

这种做法没问题，除非拿来跟Koa比较。


## Koa的做法

还是说上面那个“记录耗时时间”的例子，Koa的做法比起传统做法，显得特别清爽。下面是官网给出的例子：

```js
app.use(function* (next) {
	var start = new Date()
	yield next
	var ms = new Date() - start
	//...
})
```

那个`yield`表达式返回的时候，也就是后面的中间件已经执行完毕的时候。不需要任何全局量，就轻松自然地达到了统计时间的目的。


## 如何实现

实现Koa的这种做法其实很简单，Koa的源码里面，最核心的部分是这句：
```js
const fn = compose(this.middleware);
```

所有用`use`方法添加的中间件，都会被这个`compose`函数处理并关联。而那个`compose`函数，核心部分大概是下面这样的：

```js
var i = middleware.length

while (i--) {
	next = middleware[i].call(this, next)
}
```

`middleware`是一个数组，存放的是所有用`use`方法添加进来的中间件。在`compose`函数里，中间件会以定义的顺序被串接起来。

因此每次`yield next`的时候，都会等待下一个中间件先运行完。

如此简单巧妙！


## 对应过去

当所有中间件都只有一个`yield`表达式，而且它是函数的最后一个表达式时，Koa的中间件体系就跟原始的中间件体系效果一样了。

所以Koa的这种中间件，可以看作是传统中间件的一个超集。


[koa-home]: http://koajs.com/

