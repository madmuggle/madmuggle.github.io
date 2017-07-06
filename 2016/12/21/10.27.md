# JS混乱的字符串切片函数
2016/12/21 10:27:00


截取字符串，我一般直接使用`slice`这个方法。数组和字符串都有这个方法，用法也完全一样。而且它规则清晰，支持负数下标，表现出的行为和Python中的复数下标一样。

然而今天看第三方代码的时候，发现了`substr`这个方法。网上搜了下，原来JS的字符串切片方法一共有3种：
`substr`，`substring`，`slice`。


## 大致区别

`slice`和`substring`的原则大体上相同，就是指定一个起始点，一个终止点，然后返回两者之间的字符串。大多数编程语言的切片函数，都是这种做法。

`substr`是另一种做法，第一个参数同样是起始点，但是第二个参数却不是终止点，而是起始点开始的偏移量。

举个例子，用`substring`或者`slice`，就会是这样

```js
'hello!'.substring(1, 3)
//=> 'el'
```

```js
'hello!'.slice(1, 3)
//=> 'el'
```

而用`substr`，则是

```js
'hello!'.substr(1, 3)
//=> 'ell'
```


## 混乱的细节

总体原则虽然很好区分，但细节上，`substring`和`substr`是相当混乱的，以至于我非常希望它们俩从来没有存在过。

函数`substring`有个非常恐怖的特性，如果你的终止点小于起始点，它不返回空，而是把终止起始换个位置！所以会有这种现象：

```js
'hello!'.substring(3, 1)
//=> 'el'
```

同`slice`做个对比:

```js
'hello!'.slice(3, 1)
//=> ''
```

我不清楚当初把`substring`设计成这样是为了解决什么样的问题，但是今天来看，这种行为是非常“违背常理”的。

除此之外，`substring`对负数下标的处理也令人意外，它把负数下标看作0：

```js
'hello!'.substring(-2)
//=> 'hello!'
```

这样做让负数变的没有意义了，还不如发现负数下标后报个异常。`slice`对负数下标的处理比较合适：负数代表“倒着数”

```js
'hello!'.slice(-2)
//=> 'o!'
```

至于`substr`，我看到[这里][s1]说，IE9以上和以下表现不一样。

IE9以上和`slice`一样：

```js
"Good news, everyone!".substr(-4);
//=> "one!"
```

IE8或更老和`substring`一样：

```js
"Good news, everyone!".substr(-4);
//=> "Good news, everyone!"
```

我没有验证过这个，因为反正有`slice`，没打算用其他的。


## 应对方法

对于第三方软件，没有办法限制它们使用`substr`之类的函数，所以只能被动地去回忆这些规则和区别，别无他法。

但是我自己的程序，永远只用`slice`。为什么呢？因为我记不住那么多的规则。



[s1]: http://www.jacklmoore.com/notes/substring-substr-slice-javascript/
