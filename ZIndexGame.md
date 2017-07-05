# 玩转z-index
2016/12/06 17:53:00
Javascript, HTML


页面左上角的彩色方块全部是可以自由移动的，只需要HTML的普通DOM元素和JS就能实现这种效果。不需要SVG，不需要Canvas。关键就在`z-index`。

我只处理了鼠标事件，所以在手机上可能无法正常看到效果，建议在电脑上看。而且代码直接用的ES6语法，所以过老的浏览器也无法正常浏览。

如果对实现感兴趣，可以查看页面源代码，或者进[Github][ZIndexGame]。

更多`z-index`的文档可以在[这里][mdn]看到。


[mdn]: https://developer.mozilla.org/en-US/docs/Web/CSS/z-index
[ZIndexGame]: https://github.com/madmuggle/ZIndexGame



<script type="text/javascript">
// global variables
var colors = [
  'blue', 'green', 'red', 'orange', 'yellow', 'violet', 'purple', 'cyan'
]

var boxes = []
var activebox
var offx, offy

// reorder is just 'resetting z-index' of objects, 'z-index' big -> top
function lineupDivs(divlist, activeDiv) {
  var tmp = divlist.filter(x => x !== activeDiv)
              .sort((x, y) => x.style['z-index'] > y.style['z-index'])
  for (var i = 0, l = tmp.length; i < l; i++)
    tmp[i].style['z-index'] = i
  activeDiv.style['z-index'] = tmp.length
}

// transform e.g. '30px' -> 30
function nopx(x) {
  var val = parseInt(String.prototype.slice.call(x, 0, x.length - 2))
  if (val !== NaN)
    return val
  else
    throw(Error(x + ' is NOT pixel string'))
}

function createBox(id, left, top) {
  var box = document.createElement('div')
  box.setAttribute('id', 'box' + id)
  box.style.position = 'fixed'
  box.style.width = '200px'
  box.style.height = '30px'
  box.style.left = left
  box.style.top = top
  box.style.backgroundColor = colors[id % colors.length]
  box.style['z-index'] = id

  box.addEventListener('mousedown', () => {
    activebox = box
    activebox.moving = true
  })
  return box
}

// initialize boxes
([ 1, 2, 3, 4, 5, 6, 7, 8 ]).forEach(i => {
  var box = createBox(i, i * 10 + 'px', i * 10 + 'px')
  document.body.appendChild(box)
  boxes.push(box)
})

window.addEventListener('mousedown', e => {
  offx = nopx(activebox.style.left) - e.pageX
  offy = nopx(activebox.style.top) - e.pageY
  lineupDivs(boxes, activebox)
})

window.addEventListener('mousemove', e => {
  if (activebox.moving) {
    activebox.style.left = e.pageX + offx + 'px'
    activebox.style.top = e.pageY+ offy + 'px'
  }
})

window.addEventListener('mouseup', e => activebox.moving = false)

activebox = boxes[0]
</script>

