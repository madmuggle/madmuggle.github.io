# 实现MaterialUI输入框的动画效果
2016/12/16 18:21:00


## 漂亮的输入框动画

[MaterialUI][materialui]是一个非常好看的React CSS框架，它对细节的处理让人觉得舒服。我印象最深的是它的输入框。下边框会在鼠标点进去的时候优雅地显示出来，从中间扩展到两边，而且是一个由快到慢的过程。

就像这样：

<hr id="bar" style="margin: 20px auto 30px; width: 70%; border: 2px solid rgb(0, 188, 212); transform: scaleX(1); transition: 450ms cubic-bezier(0.23, 1, 0.32, 1);" />

之前尝试把自己的搜索输入框也做成这样的，但走错了方向。我以为可以通过设置`input`的`border-bottom`达到这个效果，实际上是不行的。


## 如何实现

这种效果的关键在于CSS3的两个特性：缩放，贝塞尔曲线。关于这两个主题，任何搜索引擎都能给出有用的答案，我就不重复了。

如果只是为了试验，我们可以非常容易做出这种效果。先定义一个HTML元素，任何元素都可以，不妨和MaterialUI一样使用`<hr>`：

```html
<hr id="bar">
```

把它的CSS设置成这样：

```css
#bar {
  transition: 450ms cubic-bezier(0.23, 1, 0.32, 1);
  border: 2px solid rgb(0, 188, 212);
}
```

然后用JS以1秒为间隔，反复修改`scaleX`的值，就能看到上面的效果了，就像这样：

```js
$('#bar').style.transform = 'scaleX(1)'
```

一秒后运行

```js
$('#bar').style.transform = 'scaleX(0)'
```

如果你不想自己写脚本，可以查看本页的HTML源码。


## 缺点

这种做法的缺点在于需要加入新的HTML元素，因为其实是利用了单独元素的缩放来模拟出下划线变化的效果。因为下划线功能的缺陷，而用独立的元素去模拟，我不太喜欢这种方式。

理想情况下，元素应该用来对应实际物件，或者创造层级，我是这样认为的。

也是因为这个原因，我放弃了把它用在自己博客上的想法。


## 更多

MaterialUI的漂亮效果体现在方方面面，而探究它的底层实现是一件既容易又有意思的事，可以拿来当茶余饭后的休闲活动来做。

如果有新发现，我会写更多的博客来分享。


<script>
var bar = document.querySelector('#bar')
var flag = true
function toggle() {
  if (flag = !flag)
    bar.style.transform = 'scaleX(1)'
  else
    bar.style.transform = 'scaleX(0)'
}
setInterval(toggle , 1000)
</script>


[materialui]: http://www.material-ui.com/
