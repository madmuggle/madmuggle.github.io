# HTML元素属性绑定的怪异行为
2016/12/22 16:59:00
HTML, Javascript


## 方便的属性绑定

在HTML里，有时候会直接把事件处理的代码写在属性里面，比如按键点击处理，跳转处理等等。在代码很短，或者在页面调试的时候，这个特性会带来很多方便。

举个例子，如果`form`里不方便放一个`input`版本的提交按钮，可以用一个`div`代替，就像这样：

```html
<div id="myButton" onclick="$('#myform').submit()">
...
</div>
```

这个`div`在被点击的时候，能做跟`button`一样的事。而这么做的好处是，你可以使用任何东西作为提交按钮了，比如SVG。


## 内部机制

上面那个例子，其实相当于把`onclick`属性从`div`里拿掉，然后在脚本里写上：

```js
$('#myButton').onclick = () => $('#myform').submit()
```

这么说可能没什么感觉，因为大家都知道会有类似的处理。但是当我发现`onclick`属性的内容只是被简单的移动到一个函数里面，还是有点惊讶的。

证据就是`onclick`属性里面可以写`return`。比如：

```html
<div id="test" onclick="return true">test</div>
```

`return`这个关键字在函数外面是没有意义的，所以可见`onclick`属性里面的代码，是被直接塞进一个函数里面的，它相当于：

```js
$('#test').onclick = function() {
  return true
}
```


## 阻止跳转

这个特性是在寻找“组织链接跳转”的办法时发现的。要阻止一个`a`链接在点击时发生跳转，只需要简单的给他的`onclick`属性设置一下就行了。

比如下面这个链接在被点击的时候不会发生任何跳转：

```html
<a href="http://baidu.com" onclick="return false">test</a>
```

这是因为，只要`a`的`onclick`事件处理函数返回非真值，它的跳转行为就会被禁止。

说实话这种做法有点Hack，真正直观的做法是捕获`a`的点击事件，然后调用事件的相应方法禁止默认行为，就像这样：

```js
$('a').onclick = e => {
  e.preventDefault()
  // ...
}
```

实际上，为了防止属性里写的`onclick`被覆盖掉，最好用`addEventListener`：

```js
$('a').addEventListener('click', e => {
  e.preventDefault()
  // ...
})
```

## 怪异

HTML的这个特性我觉得是比较怪异的，把代码直接塞进一个函数，总觉得缺点什么。不知道有没有对语法进行检查。也懒得去做实验查证了。

另外在函数之外使用`return`关键字，怎一个豪放了得。
