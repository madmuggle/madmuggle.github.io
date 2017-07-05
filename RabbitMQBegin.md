# RabbitMQ初学记录
2016/12/14 13:11:00
RabbitMQ


接触「[RabbitMQ][rabbitmq]」几天了，看了些资料，对它和与之相关的概念，有了基本了解。这篇文章不是关于技术细节，而是用我目前的理解描述这个系统中涉及到的一些概念，以及为什么是这个样子。

有些说法可能不准确，但不会毫无价值，因为我自己在学习过程中至少有过一两次恍然大悟的感觉。我猜其他同样不熟悉消息队列系统的人，会有类似的感觉。


## AMQP里的角色

RabbitMQ是一个实现了[AMQP][amqp]的消息队列系统，AMQP是一个独立于RabbitMQ的标准协议。如Wikipedia描述，它是一个`wire-level`的协议规范。可以拿它同HTTP或SMTP类比。

这种消息系统里有三个角色：Producer, Broker, Consumer。如果分得更细，Broker实际上是由Exchange和Queue等部分组成，但是这个不属于本文主题，不展开。

任何消息，都是由Producer出发，经过Broker，到达Consumer，从角色上来讲是单向的。但是程序是可以在不同角色间切换的，所以双向数据流动也不是问题。

通常说起RabbitMQ，其实指的是RabbitMQ Server，它扮演的只是Broker这个角色。Producer和Consumer则是RabbitMQ用户扮演的角色。


## 连接和通道

RabbitMQ里面，要与Broker交互，得建立一个Connection。这个和任何其他基于网络的服务一样，意料之中。然而RabbitMQ里，你还得再建立一个Channel。

按理说，只需要一个Connection的概念足够达到隔离多客户端的目的了，为什么还要加入一个Channel的概念呢？

因为不同于HTTP协议，AMQP建立的是长连接。发一条消息建立一个新连接，肯定是行不通的。因此需要复用连接。

但是消息需要隔离开，所以就新加入了一个概念叫做`Channel`。


## 客户端

使用消息队列，我们需要做的，就是“投递消息”和“获取消息”两件事而已。而这两件事都需要我们以“客户端”的身份进行。使用RabbitMQ的整个过程，我们都是那个“由Erlang编写的服务器”的“客户端”。

因此不管是Producer还是Consumer的代码，都需要相同的步骤去建立Connection，然后建立Channel，接着才能干正事。

RabbitMQ官方提供了一些[客户端][rabbitmqclient]，然而它跟MongoDB，Redis提供的“命令行客户端”不是一个概念。它所谓的客户端，只是针对不同语言的“库”。

这又一次提醒我们，大家说RabbitMQ的时候，都是在说RabbitMQ Server。


## Node.js的例子

之所以想把这个例子放上来，一个原因是它照应前面说的“连接”和“通道”的概念；另一个原因是官方推荐的库`amqplib`是纯回调的风格，代码多了很难看。而这个例子灵活的运用了Promise的特性，代码组织得更漂亮。


## 通用的部分

前面提到，Producer和Consumer均是Broker的客户端，它们的共同点并不仅限于此，在实际操作方式上，两者也有很多相同的步骤。

通道需要在连接建立好了之后才能建立，发送消息又需要在通道建立好了之后再进行。所以可以这样处理“创建连接”的过程。

```js
function newConnection(url) {
  return new Promise((res, rej) => {
    amqp.connect(url, (err, conn) => err ? rej(err) : res(conn))
  })
}
```

然后我们建立一个全局的Connection（实际是一个包含了Connection的Promise对象），接下来所有的消息都是通过它发送出去的。

```js
var conn = newConnection('amqp://localhost')
```

Channel不同于Connection，我们需要经常创建新的Channel，所以先定义这么一个函数

```js
function newChannel(conn) {
  return new Promise((res, rej) => {
    conn.createChannel((err, ch) => err ? rej(err) : res(ch))
  })
}
```

这么做的好处是你可以非常方便地给同一个Connection绑定新的Channel，并且供后续代码使用。就像这样：

```js
conn.then(newChannel)
```


## Producer端

由于Node.js的客户端借口只支持Buffer，所以为了后面处理更方便，我们先写一个小函数，进行必要的转换：

```js
function ensureBuffer(msg) {
  if (Buffer.isBuffer(msg)) return msg
  else return Buffer.from(msg)
}
```

由于队列可能一开始不存在，所以一般会用`assertQueue`确保这个队列存在。所以发消息通常需要`assertQueue`和`sendToQueue`两个步骤

```js
function mySend(ch, queue, msg) {
  ch.assertQueue(queue, { durable: false })
  ch.sendToQueue(queue, ensureBuffer(msg))
}
```

再定义一个接口函数来发送消息。这个函数充分利用了Promise的特性，确保了我们每次都使用“旧”的Connection，“新”的Channel。

```js
function send(queue, msg) {
  return conn.then(newChannel).then(ch => mySend(ch, queue, msg))
}
```

接着就可以轻轻松松地发消息了！

```js
send('hello', 'hello, world!').catch(console.error)
send('cool', 'cool!')
```


## Consumer端

类似Producer的做法，同样先定义一个辅助函数

```js
function myConsume(ch, queue, action) {
  ch.assertQueue(queue, { durable: false })
  ch.consume(queue, action, { noAck: true })
}
```

然后是用户接口函数

```js
function handle(queue, action) {
  return conn.then(newChannel).then(ch => myConsume(ch, queue, action))
}
```

使用起来和Producer一样，非常方便

```js
handle('hello', console.log).catch(console.error)
handle('cool', console.log)
```


## 监控和管理

RabbitMQ还是提供了一些用于监控和管理的工具的，最好用的当属那个Web界面的工具了。它是作为RabbitMQ服务器的插件提供的，默认的地址是<http://localhost:15672>，这个应用依赖于RabbitMQ提供的REST服务。

如果是在Ubuntu下用`apt-get`安装的RabbitMQ，可能需要手动使能这个插件：

```shell
rabbitmq-plugins enable rabbitmq_management
```

你也可以绕过它，通过curl来访问REST，只要不嫌麻烦。

除了Web工具，还有个叫`rabbitmqctl`的命令行工具，目前没用过，也没法讲。而且不出意外我感觉自己应该不会去用那个工具吧。毕竟有RESTful接口。


## 为什么用MQ

其实我一直对消息队列这种东西提不起兴趣，这次如果不是工作需要，我不可能主动学习使用RabbitMQ的。甚至在我迷恋Erlang的那段时间里，都没有想过去读一读RabbitMQ的源码。

其实当时我很不理解，对于Erlang这种自带消息队列的语言，为什么要用它去写一个消息队列系统？难道仅仅是为了给其他环境提供类似Erlang的能力吗？

这两天我才意识到，是因为我在心里给了MQ错误的定位。MQ并不是一个应用软件级别的东西，它其实是个非常基础的概念。对于网络而言，它比Socket更好。


## 不自然的Socket

Socket在我心里根深蒂固，因为我对网络最初的了解就源自于Socket。记得我还因为搞清楚了TCP的各种状态而感到自豪。

我以前觉得，Socket就是网络的根本。但客观来讲，Socket非常不自然。它之所以成为网络的基础，恐怕更多的是因为无奈，而不是合适。

第一个不自然是它基于「流」。现实世界中，消息就是消息，根本不需要「流」这个概念的存在。「流」是当年我们使用电信号在导线上传输数据而衍生出的概念。对于我们交流而言，它并不自然。

想想以前寄信，难道你需要考虑你信里的文字，只有一半送到了对方那里的情况吗？

第二个不自然在于本质上它是一对一的。服务器程序为了实现一对多的模型，都得采用繁琐的做法，依次进行“bind”，“listen”，“accept”。繁琐并不是问题，这几个操作是“不合逻辑”的，才是问题所在。

相比之下，MQ就是个非常自然的概念，虽然RabbitMQ是建立在Socket上的，但是MQ这个概念并不局限在Socket上，它其实可以成为一个替代Socket的概念。

所以使用MQ的根本原因是：网络应用本来就该这样写！

当然，类似“文件传输”这种直接能够对应到“流”上的概念，不在上述观点之内。但这种情况很少，也很简单，并不需要我们投入多少精力。

## 更合适的地方

然而「流」这个概念目前无法消除，这是我们的硬件决定的。但是我们可以让它待在更底层的地方，就像驱动一样。我们不应该直接使用Socket，使用「流」来写网络通讯程序。

当然这只是一个期望而已，如果有朝一日它真的实现了，编写网络程序一定会成为更快乐的事。

但是换个角度来想，这个愿望其实早就实现了。在Erlang里，一直都是这么做事的。所以Erlang的程序写起来思路特别清晰，即使它的语法挺另类。

希望又一天这种模型能更加普及吧。


## 后记

今天(2016/12/23)发现，`amqplib`本身就使用了Promise，所以我上面写的那两个函数是画蛇添足。RabbitMQ官网的例子该更新一下了。

要创建一个Connection提供Promise接口，直接这样就行了

```js
var conn = amqp.connect('amqp://localhost')
```

使用这个Connection，创建并使用Channel只需要直接：

```js
conn
.then(conn => conn.createChannel())
.then(ch => mySend(ch, 'blah', '...'))
```

另外`assertQueue`也实现为了Promise，所以其实那个辅助函数`mySend`也比较多余

```js
conn
.then(conn => conn.createChannel())
.then(ch => {
  ch.assertQueue(queue, { durable: false })
  return ch
})
.then(ch => {
  ch.sendToQueue(queue, ensureBuffer(msg))
})
```

看着挺乱对吧，这是[Promise的缺陷][promise]导致的（无法共享局部变量）。`co`或者`async`可以弥补这个遗憾

```js
co(function* () {
  var conn = yield amqp.connect('amqp://localhost')
  var ch = yield conn.createChannel()
  yield ch.assertQueue(queue, { durable: false })
  ch.sendToQueue(queue, ensureBuffer(msg))
})
```

但是这样会导致每次发送都新建Connection，所以把这步提取出来

```js
var globalConn = amqp.connect('amqp://localhost')
```

然后在那个Generator函数里这样用

```js
co(function* () {
  var conn = yield globalConn
  ...
})
```

现在易用性和易读性都得到了保障。


[rabbitmq]: http://www.rabbitmq.com/
[amqp]: https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol
[rabbitmqclient]: http://www.rabbitmq.com/clients.html
[promise]: /ES6_Promise.html
