# 留意Mac的命令行
2016/11/24 18:55:00
Shell


Mac OS X 以某种形式带有一个FreeBSD内核，可以运行整套的Unix命令，并且它的文件系统也是以Unix的形式呈现出来，所以从用户的角度来看，它就是一个Unix。但是如果你习惯了Linux，那么Mac的命令行可能有不少你需要留心的「坑」。

首先它的默认Shell使用的是Bash，和Linux一样，而它的命令集却用了BSD那套。这就经常引人犯错。GNU跟BSD的工具，在很多细节上不同。


## 命令行参数顺序

最容易察觉的一点就是命令行参数的顺序问题。在Linux下，参数的顺序是任意的，比如

```sh
ls -la /tmp/blah
```

和下面的命令一样

```sh
ls /tmp/blah -la
```

而Mac的Shell，`-la`这种必须写在前面。


## 细节，大坑

上面那个还无伤大雅，但是接下来这个就头疼了。Linux下，要将一个文件夹整个拷贝到另一个文件夹下，你会

```sh
cp -r ~/Downloads/ /tmp/
```

最后那个`/`不影响语义，所以你也可以写成

```sh
cp -r ~/Downloads /tmp/
```

而Mac下，第一条命令是将Downloads下的内容拷贝到/tmp下。第二条才是把Downloads整个拷贝到/tmp。Mac下的`cp`对`/`敏感。

开发npm包为了方便，有时会直接把它`cp`到`node_modules`下。如果多打了一个`/`，很可能把整个库都弄乱。

`/`不是问题，相同的命令表现不同才是问题。`nginx`的配置也对`/`敏感，但是并无不妥，因为你不会跟别的东西混淆。

另外你得用`-R`。Linux下，对于`cp`命令，`-r`和`-R`是相同的意思。而在Mac下，用`man cp`会看到：

> Historic versions of the cp utility had a -r option.  This implementation
supports that option; however, its use is strongly discouraged, as it
does not correctly copy special files, symbolic links, or fifo's.

如果Mac下，所有的命令都是这样对待`-r`和`-R`参数也就罢了，但是你`man rm`会看到：

> -r          Equivalent to -R.


## iproute2

你是不是记不住`ifconfig`和`route`参数的用法，这不是你的错。这些工具属于一个叫`net-tools`的软件包，它们的用法缺乏正交性，规律性差，属于典型的设计不好。

Linux发行版很早就开始使用`iproute2`了，也就是`ip`开头的那套命令。它们规律性很强，一旦学会，就不会忘记。

然而Mac下只有net-tools。
