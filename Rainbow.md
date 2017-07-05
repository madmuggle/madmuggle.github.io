# 语法高亮插件rainbow
2016/09/28 11:23:00
HTML, CSS, Javascript

HTML的代码高亮插件有很多，都是用JS解析文本，然后替换成DOM来达到高亮效果。对比了一番，觉得还是「[rainbow][rainbow_homepage]」这款最合我胃口。


## 只要包含，无需配置

使用rainbow非常方便，只需要包含相关的文件，不需要写一行JS代码。脚本会自动处理页面内所有的`<code>`标签。

rainbow的官方网站经常无法访问，好在你可以使用公共CDN，比如[bootcdn][bootcdn]。

由于rainbow的使用非常简单，所以除非想研究原理，你根本不需要去它的官网。只需要在你的代码里面包含这几个部分：


## 随意挑选的主题

在HTML的head包含需要的高亮主题，有好多可以选择，你可以在[这里][rainbow_bootcdn]随意选。比如你可以使用朴实的github风格：

```html
<link rel="stylesheet" href="//cdn.for.rainbow/themes/github.min.css">
```

而且它的颜色方案全部是通过CSS设置的，你想自己定制主题很容易。


## 使用方法

用`<script>`标签包含你需要的JS文件，你可以把这个写在文档最后面，因为它不是被事件触发的，所以任何时候加载都没问题。

```html
<script src="//cdn.for.rainbow/rainbow.min.js"></script>
<script src="//cdn.for.rainbow/generic.min.js"></script>
<script src="//cdn.for.rainbow/language/javascript.min.js"></script>
<!-- ... -->
```

行了！接下来你只需要把你的代码这样呈现出来：

```html
<pre>
  <code class="lang-javascript">
(function() { console.log("hello, rainbow !") })()
  </code>
</pre>
```


## 更多效果展示

Javascript

```javascript
// This is a Node.js test, start a https server
var s = https.Server({cert: "mycert...", key: "mykey..."})

s.on("request", (req, res) => res.write(util.inspect(req)))
s.listen(3000, "127.0.0.1")
```

HTML

```html
<!-- HTML comment, This comment is here to test rainbow -->
<div class="blah">This is a simple div</div>
```

CSS

```css
.blah {
  /* A simple css comment to test the rainbow plugin */
  font-style: italic;
}
```

Cpp

```c
/* This is why C language so powerful and so ugly */
(char (*)(int, int)) (*p)(int (*)(int), char *);

main(int argc, char **argv) {
  std::cout << "hello, this is a cpp file." << std::endl;
}
```


## 后记

由于现在我移除了所有博客的语法高亮插件，所以在这个页面，你无法看到Rainbow的使用效果，建议在自己的电脑上拷贝代码进行实验。

实际上，我发现语法高亮在博客里作用并不大。博客里不应该出现大规模的代码，控制在几行内比较合适，否则会让文章很乱。

语法高亮出现在编辑器和Github之类的网站就行了。


[rainbow_homepage]: https://craig.is/making/rainbows
[bootcdn]: http://www.bootcdn.cn/
[rainbow_bootcdn]: http://www.bootcdn.cn/rainbow/

