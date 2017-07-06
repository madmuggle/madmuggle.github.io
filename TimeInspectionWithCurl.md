# 使用curl命令进行网络连接时间测定
2017/03/02 13:50:00


## 方便的curl

网络连接背后也包含了许多看不见的细节。比如域名查询，协议握手等等。在查找网络问题时，需要知道各个操作的细节，比如握手情况，各项操作耗时等等。

强大的抓包工具如Wireshark，当然是能够满足所有要求。但很多时候，常用的curl命令就足以完成这项任务。


## 具体操作

这里我们以测量“操作耗时”为目标来进行实验。关键在于curl的`-w`选项。用手册的原话来说，这个选项的作用是：
> Make curl display information on stdout after a completed transfer.

它的用法和编程语言里的打印函数类似，如C里的printf。它支持变量，你可以通过这些变量来获取时间量。

比如我想知道访问天猫的过程中，DNS查询，TCP握手，TLS握手各花了多少时间，可以用下面的命令：

```sh
curl https://www.tmall.com -o /dev/null -w '
DNS: %{time_namelookup}
TCP: %{time_connect}
TLS: %{time_appconnect}
'
```

输出结果是：

```
DNS: 0.033
TCP: 0.102
TLS: 1.100
```

可用的变量当然不止这些，总用时，重定向时间等等都可以用此法测量，具体的man手册上有详细介绍。


