# NodeJS的Keygrip库
2017/04/07 12:45:00
Node.js


今天看Koa相关的源码，发现了[keygrip][keygrip-github]这个库，用来处理HTTP cookie。这样做的主要目的大概是为了安全，但具体是如何达到安全的效果，我还没搞清楚。

先针对这个库如何使用做了个常识，整理成笔记放在这里，之后有新的收获再加进来。

```js
var keys = require('keygrip')([ 'blah1', 'blah2' ]);
```

针对任何一个字符串进行一个签名：
```js
var x = keys.sign('Hello, World');
```

得到的结果是一个加密过的字符串：
```js
x
//=> 'VHlAODdRQBAfPufgdTwtK3dX1io'
```

接下来就可以用这个加密过的字符串验证其他的输入：
```js
keys.verify('Hello, World', x)
//=> true
```

作为对比，传一个不同于之前的字符串试试：
```js
keys.verify('Hello, World!', x)
//=> false
```

[keygrip-github]: https://github.com/crypto-utils/keygrip
