# 看得见摸不着的Attribute
2016/11/13 22:05:00
Javascript, Python

在面向对象的编程语言中，普遍存在一个特点，你可以定义一类特殊的属性，它不是真正的属性，你读或者写它时，其实是在调用特定的方法。本文将使用两种具有代表性的编程语言来举例：基于「类」的Python，和基于「原型链」的Javascript。


## Javascript的例子

Javascript的关键字`get`和`set`虽然用得比较少，但是规则是很直观的。简单说，就是在普通的函数定义前，加一个`get`或者`set`。

要具体解释这种语法，得先解释下Javascript里面奇怪的方法定义语法，可以参考下[这篇文章][article]。本篇文章只谈用法。

```js
var a = {
  set blah(v) { console.log(`your argument is ${v}`) },
  get blah() { return '<<<blah>>>' }
}
```

你并没有直接定义名为`blah`的属性，但是现在却可以使用它

```js
a.blah
//=> "<<<blah>>>"
a.blah = 3
//   your argument is 3
//=> 3
```

除此之外，`Object.defineProperty`也可以做同样的事

```js
var a = {}

Object.defineProperty(a, 'blah', {
  set: (v) => console.log(`your argument is ${v}`),
  get: () => '<<<blah>>>'
})
```

```js
a.blah
//=> "<<<blah>>>"
a.blah = 3
//   your argument is 3
//=> 3
```

## Python的例子

在Python里面，`@property`和`@varname.setter`是用来做这个事的。

```python
class MyClass:
  def __init__(self):
    self._val = 3

  @property
  def val(self):
    print("from @property, _val = ", self._val)

  @val.setter
  def val(self, val):
    print("from @val.setter")
    self._val = val
```

运行一下

```python
myobj = MyClass()
myobj.val
#=> from @property, _val =  3
myobj.val = 4
#=> from @val.setter
myobj.val
#=> from @property, _val =  4
myobj._val
#=> 4
```

跟JS的`get`和`set`表现一样，我们在访问一个不存在的属性`val`。

除此之外，Python的「descriptor」也能做类似的事情。

```python
class MyDescriptor:
  def __get__(self, obj, owner):
    print("from MyDescriptor.__get__, val = ", obj.val)
  def __set__(self, obj, val):
    print("from MyDescriptor.__set__")
    obj.val = val
```

接下来就可以使用「descriptor」了。

```python
class MyClass:
  desc = MyDescriptor()
  def __init__(self):
    self.val = 3
```

这个类里面的`desc`属性，会表现出跟Javascript里的`get`和`set`一样的行为。

```python
myobj = MyClass()
myobj.desc
#=> from MyDescriptor.__get__, val =  3
myobj.desc = 4
#=> from MyDescriptor.__set__
myobj.desc
#=> from MyDescriptor.__get__, val =  4
myobj.val
#=> 4
```

[article]: /articles/WeirdMethodDefinition.html
