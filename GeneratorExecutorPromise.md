# Generator执行器 - Promise版
2016/11/26 22:34:00
Javascript


其实我并不想把Generator和Promise扯在一起，他们俩儿各有自己的味道，合在一起之后，虽然提供了一个与ES7的Async功能一致的接口，却显得不那么精巧了。

但是为了凑个完整，和[前一篇][GeneratorExecutorThunk]形成照应，所以还是留下这篇文章吧。正好代码改动也很小。

同样，先定一个Promise版的`readFile`

```js
function readFile(fileName) {
  return new Promise((res, rej) => {
    fs.readFile(fileName, 'utf-8', (e, d) => e ? rej(e) : res(d))
  })
}
```

Promise版的`exec`和Thunk版的差别很小

```js
function exec(genFn) {
  var gen = genFn()

  function next(d) {
    var r = gen.next(d)
    if (!r.done) r.value.then(next)
  }

  next()
}
```

```js
exec(function* () {
  var t1 = yield readFile('a.txt')
  var t2 = yield readFile('b.txt')
  console.log(t1, t2)
})
```

在Node.js下测试依旧正常

同样是只有不到10行代码就实现了。如果把异常处理等各种细节考虑进去，代码会变长，但是核心不会变。

实际上，[co][co]源码里的那个`next`，跟上例中的`next`是一致的用法，不过提取了相应的重复代码，让递归变成了两个函数的相互调用：

```js
function exec(genFn) {
  var gen = genFn()

  function onFulfilled(r) {
    next(gen.next(r))
  }

  function next(r) {
    if (!r.done) r.value.then(onFulfilled)
  }

  onFulfilled()
}
```

Have fun.


[GeneratorExecutorThunk]: /articles/GeneratorExecutorThunk.html
[co]: https://github.com/tj/co/blob/master/index.js
