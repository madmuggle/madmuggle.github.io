# 解释epoll的好例子
2017/01/04 10:52:00
Linux


## 记不住的epoll

前年特地花了一些时间，写了点`select`，`poll`和`epoll`的程序来帮自己记住这不常用但很重要的知识。说“不常用”不太准确，因为Node.js和Nginx都是建立在这个上面的。

但今天回忆一下，发现依旧忘记了关键的地方，只记得`epoll`并不是Node.js那样的回调模式，并没有给它传函数。但其他的竟然一点也想不起来了。


## 漂亮的伪码

在[知乎][zhihu]上发现了一个非常形象的解释，只使用了几行伪码，把关键点全都点出来了，胜过千言万语。忍不住把那几段伪码粘贴过来，防止以后找不到出处。

阻塞IO:

```
//........................................... blocking nowhere
while true {
  for i in all_streams[]
    if i has data
      read until unavailable
}
```

select:

```
while true {
  select(all_streams[]) //................... blocking for data

  for i in all_streams[]
    if i has data
      read until unavailable
}
```

epoll:

```
while true {
  active_streams[] = epoll_wait(epollfd) //.. blocking for data

  for i in active_streams[]
    read utill unavailable
}
```


## 总结

我忘掉的那一环是：`select`和`epoll`都有一个函数，阻塞在那里，直到有数据到来才返回。

要解释计算机概念，“程序”终究是最好的语言啊。


[zhihu]: https://www.zhihu.com/question/20122137
