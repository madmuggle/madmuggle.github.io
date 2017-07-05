# GenServer和Redux
2016/11/16 23:45:00
Javascript, Erlang, Elixir


记得曾近看过一个视频，讲Erlang的，除了词句幽默之外，讲解的人还陈述了一个令我印象深刻的观点：「状态」在函数式编程语言中也存在，只不过它们存在于「函数调用之间」。

当时我刚接触Erlang，不理解这句话。那个时候，一旦要使用状态量，我的选择就是mnesia数据库。直到我学会了GenServer及相关的概念之后，才明白了那句话。


## Genserver的例子

这里用Elixir演示GenServer，原理上和Erlang版本没有任何区别。为了突出重点，这里只留下最关键的部分：

```elixir
defmodule MyGenServer do
  def handle_cast(obj, state): { :noreply, [ obj | state ] }
end
```

第一行只是定义一个「模块」，这是BEAM虚拟机的限制，函数必须定义在模块内部。Erlang
是这样，Elixir也不能免俗。我们只需要关注`handle_cast`就行了。

函数`handle_cast`会接受一个状态值，返回一个新状态值。它做的事很简单，就是往状态里面添加新元素`obj`。

每次函数调用都以前一次的返回值作为参数传入，如此便达到了状态在函数调用间保存并传递的效果，同时还保证了函数内部不存在状态，是「纯函数」。


## 异曲同工的Redux

最近因为工作需要，接触了Redux，惊讶地发现，这东西跟Erlang和Elixir的GenServer完全是一个思路。比如下面这个例子，我们先定义这样一个函数

```js
function myReducer(state, action) {
  return state ? `${state}${action.type}` : '>'
}
```

这个函数所做的事，和上面Elixir例子里的函数几乎一模一样：接收一个状态，然后返回一个新状态。

函数`myReducer`也没有修改state，只是读取它，然后根据它返回一个新的state。它同样是「纯函数」。

## subscribe & dispatch

接下来使用这个函数来创建出一个存状态的东西，这里叫它`store`

```js
var store = Redux.createStore(myReducer)
```

这个`store`就是一个状态存放点，它最关键的两个方法是：
- `store.subscribe`
- `store.dispatch`

希望在`store`里面的值改变的时候做点什么，只需要用`store.subscribe`注册一个函数就行。（在`myReducer`之外访问`state`要使用`store.getState`）

```js
store.subscribe(() => console.log(store.getState()))
```

从状态存放点「取值」的功能有了，还需要一个「存值」的功能吧。那么应该调用什么方法呢？很遗憾，没有这种方法!

这也正是Redux的关键。你无法把新的状态值写进那个状态存储器里面，你只能使用`store.dispatch`给之前定义的函数`myReducer`发消息。

```js
setInterval(() => store.dispatch({ type: '*' }), 1000)
setInterval(() => store.dispatch({ type: '-' }), 1000)
```

由于Redux对消息的格式有限制，必须是带有`type`字段的`Object`，所以写成了上面那样。

这个消息怎么整合到状态里面呢？答案就是我们前面定义的那个函数`myReducer`。它就是连接新状态和老状态的桥梁。


## 不是必须，只是很好

对于Erlang和Elixir而言，由于所有语言级别的数据结构都是Immutable的，所以如果需要「状态」，要么使用ETS数据库，进程字典这类东西，要么就使用类似GenServer的编程方法。

而Javascript并没有这种限制，要解决类似的问题，方案有很多。但Redux这种管理状态的方式，让人思路清晰。仅仅是这一点，就足以证明它的价值了吧。


## BTW

上面那个Redux示例的运行结果是如下的ascii图形

```
>*
>*-
>*-*
>*-*-
>*-*-*
>*-*-*-
>*-*-*-*
>*-*-*-*-
```
