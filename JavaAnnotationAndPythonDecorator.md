# Java的Annotation和Python的Decorator
2017/03/20 16:25:00
Java, Python


## 一样？天差地别！

Java的Annotation和Python的Decorator看上去非常相似，而且当它们被用于服务端开发时，第三方框架都有使用Annotation或Decorator来进行路由配置。所以很多人在解释这两者时，都说和Java的Annotation一样；或者说Annotation和Decorator一样。

其实这个解释是不合适的，两者虽然看上去差不多，但实际的运行机制是完全不同的。


## Annotation

Annotation于Java而言是类型，并不是语法糖。如同贴纸一样，你在某个地方定义它，使用它，然后在另外一个地方读取它。它的存在完全不影响运行过程，就如同它的名字，只是“注释”。

定义一个Annotation，形式上和定义一个Java interface非常相似：

```java
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnno {
    String blah() default "boom";
}
```

然后在一个自定义的class上使用这个Annotation：

```java
@MyAnno(blah = "ccc")
class MyClass {
    @MyAnno(blah = "fff")
    public String name;

    @MyAnno(blah = "mmm")
    public void fn(@MyAnno(blah = "ppp") String arg) {
        return;
    }
}
```

要读取上面那个类的Annotation，只需要简单地这样做：

```java
Class x = new MyClass().getClass();
System.out.println(x.getAnnotation(MyAnno.class));
```

打印出的结果是：

```
@MyAnno(blah=ccc)
```

获取Field的Annotation也很容易：

```java
Field f = x.getField("name");
System.out.println(f.getAnnotation(MyAnno.class));
```

打印输出为：

```
@MyAnno(blah=fff)
```

与之相似，获取方法的Annotation：

```java
Method m = x.getMethod("fn", String.class);
System.out.println(m.getAnnotation(MyAnno.class));
```

输出：

```
@MyAnno(blah=mmm)
```

甚至方法参数的Annotation也可以依此方法获取：

```java
Parameter p = m.getParameters()[0];
System.out.println(p.getAnnotation(MyAnno.class));
```

输出：

```
@MyAnno(blah=ppp)
```


## Decorator

Python中的Decorator则做得相当灵活，它并不是像Java的Annotation那样，作为一个新的类型引入，而仅仅是对代码做一种“语法转换”。

也就是说，它是一层很浅的“语法糖”。

你并不需要定义一个Decorator，只需要定义一个普通的function，或class，或任意可调用的对象，然后在它前面写个"@"。

比如下面这个：

```python
@dec
def method(args): pass
```

其实相当于：

```python
def method(args): pass
method = dec(method)
```

基于上面这个原则，多个Decorator连续使用的时候，其语义也和Annotation完全不同，例如：

```python
@dec_a
@dec_b
@dec_c
def method(args): pass
```

其实是：

```python
def method(args): pass
method = dec_a(dec_b(dec_c(method)))
```

Decorator最大的特征，是它可以完全改变函数的语义。比如先定义一个会被用作Decorator的函数：

```python
def blah(x): return "cool"
```

然后使用它：

```python
@blah
def test(): print('orz')
```

接下来看这个：

```python
test.__class__
#=> <class 'str'>
```

这个Decorator将test从一个函数，变成了一个字符串！


## 总结

在功能上和灵活度上，Decorator是远远超过Annotation的。然而Decorator的使用，会对代码的直观程度造成比较大的影响，因为加上了Decorator之后，函数可能就不再是你看到的那个函数了。

相比之下，Annotation虽然比较笨拙难用，但是却不会影响函数的内容，对于代码的维护来讲，这可能会非常重要。


## 后记

似乎ES7中也加入了Decorator，而且和Python的Decorator一模一样，只是多了些限制，比如语法上强制要求Decorator只能用在class及class内的属性上，如果用在一个函数上会报语法错误。

但我刚才（2017/04/18）试了一下，现在的浏览器及Node.js(v7.8.0)都没有原生支持Decorator，想用Babel体验一下都得装插件，麻烦。还是等主流支持后再进一步探究吧。

