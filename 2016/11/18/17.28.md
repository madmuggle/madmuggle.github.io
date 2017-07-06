# 常用编程环境的公共源切换
2016/11/18 17:28:00


现在的编程语言或编程环境离不开“包管理系统”或“镜像管理系统”之类的东西，而这些管理系统又离不开公共源。本篇博客收集常用的编程环境切换源的方法，可能会不定期更新。

源的更换方式可能有多种，除了命令行，还有配置文件可行，这篇文章只收集命令行方式。

## Node.js

写这篇文章的时候，[npm][npm]依旧是Node.js的官方包管理程序，如果[yarn][yarn]以后成为主流，就把它也加进来。

```sh
npm install react --registry https://registry.npm.taobao.org
```

## Python

```sh
pip install sh --index-url https://pypi.douban.com/simple
```

## Ruby

```sh
gem sources --add https://gems.ruby-china.org/ \
            --remove https://rubygems.org/
gem install rails
```

## Docker

Docker在启动Daemon时就指定其镜像仓库。

```sh
docker daemon --registry-mirror https://mirrors.com
```

或者不使用HTTPS

```sh
docker daemon --insecure-registry blah.com
```

[npm]: https://www.npmjs.com/
[yarn]: https://yarnpkg.com/

