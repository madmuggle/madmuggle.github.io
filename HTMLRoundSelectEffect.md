# HTML实现VSCode的圆角选中
2016/11/18 11:24:00
HTML

VSCode的细节做得很漂亮，在选中文本的时候，它的显示效果非常好看，在合适的地方，会呈现向左或向右的圆角。

其实在HTML里面，实现这样的圆角非常容易。其中一种方法是套两层`<span>`，以达到朝外的圆角效果。

以下是一个示例源码，读者可以拷贝到一个文件，用浏览器打开看效果

```html
<html>
<head>
<style>
body {
  background-color: black;
  color: green;
  font-size: 20px;
  line-height: 1;
}
.a-holder {
  background-color: white;
}
.a {
  background-color: black;
  border-bottom-right-radius: 8px;
}
.b {
  background-color: white;
  color: black;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  border-bottom-right-radius: 8px;
}
.c {
  background-color: white;
  color: black;
  border-top-left-radius: 8px;
  border-bottom-left-radius: 8px;
  border-bottom-right-radius: 8px;
}
.d-holder {
  background-color: white;
}
.d {
  background-color: black;
  border-top-left-radius: 8px;
}
</style>

<body>
  <span class="a-holder"><span class="a">A quick brown fox j</span></span><span class="b-holder"><span class="b">umps over the lazy dog.</span></span>
  <br>
  <span class="c-holder"><span class="c">A quick brown fox jumps over th</span></span><span class="d-holder"><span class="d">e lazy dog.</span></span>
</body>
</html>

```

