# Node.js的require
2016/11/23 10:46:00

现在几乎每天都在用Node.js，但是说起它的一个关键特点`require`，还是偶尔会发现一点意料之外的特性。伴随着Node.js的发展，这些特性以后可能会继续变化。等有新发现，再补充到这篇博客里面。


## require anything

这个好像是`webpack`的口号。然而，虽然种类不多，但其实Node.js原生的`require`也可以识别和操作Javascript之外的东西。

创建一个文件，命名为`a.js`，它的内容如下

```js
{ "a": 1, "b": 1 }
```

直接`require`这个文件是会报错的。

```js
require("./a.js")
//  SyntaxError: Unexpected token :
```

现在把`a.js`重命名为`a.json`，不修改文件内容。

```js
require("./a.json")
//=> { a: 1, b: 1 }
```

可见Node.js的原生`require`是可以识别不同文件类型的，并且对应不同的文件，使用不同的操作。这一点跟webpack很相似呢。

不知道Node.js的`require`将来会不会允许我们自己定义loader。
