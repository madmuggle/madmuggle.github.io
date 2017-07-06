# Github的一些使用技巧
2017/01/12 17:46:00


## 修改TAB的显式方式

并不是只有编辑器才有这个功能，Github照样提供了这个选择，而且提供了两种途径：URL和配置文件。

只要在源码页面的URL后面加上一个`?ts=xx`参数，就能够把页面内的TAB设成特定大小，比如下面这个例子，就把TAB设成了两个字符的宽度：

<https://github.com/golang/go/blob/master/src/syscall/asm_linux_arm.s?ts=2>

另一种做法是增加配置文件，只需要在项目根目录加入一个名为`.editorconfig`的文件，文件里写上：
```
[*]
indent_size = 4
```

然后你项目里所有文件的TAB都会显示为4个字符宽度。

如果只想对特定文件生效，比如只想让`go`语言文件和`c`语言文件这样显示，只需要简单地修改成：

```
[*.{go,c}]
indent_size = 4
```



## 高亮特定行

这个特性在引用代码的时候特别有用，做法也很简单，只需要在URL后面加上`#Lxx`：

<https://github.com/golang/go/blob/master/src/syscall/asm_linux_arm.s#L45>

下次在论坛回答别人问题的时候，可以考虑下这样给出Github链接，比自己粘代码或截图要清晰得多。

