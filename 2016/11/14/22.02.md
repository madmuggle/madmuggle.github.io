# ES6奇怪的方法定义
2016/11/14 22:02:00


## 属于class的新语法

如果把`ES6`比作针对`JS`的全方位外科手术，那么`class`就是动在脸上的那一刀。不管爱不爱，`class`写法已经逐渐成为主流，尽管本质上，它只是对JS的「原型链」做了一个简单的包装。

现在你随处可看见这样的JS

```js
class Blah {
  constructor(v) { this.v = v }
  blah() { return this.v + 1 }
}
```

从效果上来讲，上面的写法其实相当于

```js
function Blah(v) { this.v = v }
Blah.prototype.blah = function() { return this.v + 1 }
```

然而直观上来讲，

```js
blah() { return this.v + 1 }
```

可以同下面这种语法比较

```js
function blah() { return this.v + 1 }
```

只是单纯得去掉`function`关键字。

之所以要这样类比，是因为Generator函数的写法也遵循这个原则

```js
class A {
  * a() { yield 'cool' }
}
```

上面的

```js
* a() { yield 'cool' }
```

其实相当于

```js
function* a() { yield 'cool' }
```

可以运行验证

```js
x = (new A()).a()
x.next()
//=> { value: 'blah', done: false }
```


## 非class内也存在

增加了关键字后，定义针对新关键字的语法，这种做法无可厚非。问题在于下面这种语法也是合法的

```js
var a = { blah() { return 1 }}
```

它的意思其实是

```js
var a = { blah: function() { return 1 }}
```

跟在`class`关键字里面表现一致。然而这种写法定义出的方法并不在`prototype`里面。跟`class`里面那种写法，做的事是完全不一样的。


不出意料，Generator函数也是如此

```js
var a = { * blah() { yield 'cool' } }
x = a.blah()
x.next()
//=> { value: 'cool', done: false }
```

不记得刚接触这个时的感觉了，因为现在我已经习惯这么用着了。但是同样的语法，做着不同却又有点相似的事情，是很糟糕的。

我不认为这是ES6的进步。
