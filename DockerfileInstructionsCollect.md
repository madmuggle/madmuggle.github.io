# Dockerfile指令汇总
2017/02/06 17:25:00


对于Dockerfile的指令，原本我是打算用到哪里学到哪里，碰到问题网上搜。因为它的指令规律性不强，有很多需要记住的小地方，比如[上一篇][prev]博客里提到的几个问题。

但是网上很多文章写得不全，[官方文档][doc]又太长太全，为了节省时间，我把它整理收集到这里，只列出典型作法，方便日后查找。

和其他文章一样，这篇文章可能不定期更新。

Docker的配置可以通过`inspect`命令查看：

```sh
docker inspect myimg
```

所以配置的效果从容器外部也是可以知道的。


## FROM

FROM必须是Dockerfile的第一条“指令”，之前只能放注释，否则在运行`build`时会报错。可多次出现，创建混合的image。

```
FROM ubuntu
```

## MAINTAINER

创建者信息，不写并无大碍。

```
MAINTAINER madmuggle@126.com
```

## RUN

执行系统的任何命令

```
RUN apt-get update
```

## ENV

设置环境变量

```
ENV RABBITMQ_URI='amqp://muggle:mypassword@192.168.1.2'
```

## USER

Docker默认用户是root，可以切换成其他的用户

```
USER muggle
```

## WORKDIR

Docker默认工作目录是`/`，切换目录也可以用RUN，但需要每一条RUN都切换一次。所以用WORKDIR比较方便。

WORKDIR对RUN, CMD, ENTRYPOINT起作用。

```
WORKDIR /srv/
```

## COPY

拷贝文件，具体信息可见[上一篇][prev]文章。

```
COPY ./package.json /srv/blah/
```

## ADD

与COPY类似，具体信息可见[上一篇][prev]文章。

```
COPY ./package.json /srv/blah/
```

## ENTRYPOINT

指定入口的程序，具体信息可见[上一篇][prev]文章。

```
ENTRYPOINT [ "echo", "cmd" ]
```

## CMD

主要用于指定默认入口的参数，具体信息可见[上一篇][prev]文章。

```
CMD [ "echo", "cmd" ]
```

## ARG

Docker1.9新加入的指令，跟ENV用法有点类似，但ARG定义的变量只在建立image时有效。

```
ARG user=muggle
```

## VOLUME

容器使用的是AUFS，当容器关闭后，所有的更改都会丢失，如果需要持久化，就需要指定Volume。相当于挂载一个外部的硬盘。

```
VOLUME ["/data"]
```

## EXPOSE

对外暴露容器的端口。

```
EXPOSE 80 3000 8888
```

## ONBUILD

ONBUILD指定的命令在构建镜像时并不执行，而是在它的子镜像中执行。

```
ONBUILD ADD . /app/src
```

## LABEL

LABEL可以单条指定，多次使用：

```
LABEL version="1.0"
```

也可以分组指定：

```
LABEL m.l1="v1" m.l2="v2" other="v3"
```


[doc]: https://docs.docker.com/engine/reference/builder/
[prev]: /DockerfileInstructions.html

