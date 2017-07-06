# Spring初学笔记：bean的创建与获取
2017/03/31 10:12:00


## 开始学习Spring

Web开发相关的东西接触了不少，服务端这块，用Python，Erlang，Node.js甚至Clojure等都做过小的项目开发，接触的框架有一些，自己也动手实现过底层部分，比如用C语言解析multipart/form-data，接收文件传输。

然而没有认真使用过Java做开发。虽然较早就接触过Servlet，JSP，但没有进一步去做更多尝试了。

相对于Python做开发，Java的那一套显得麻烦得多。再加上很多牛人对Java，XML，Spring之类的东西嗤之以鼻，所以我从来都没有想过去用它。

然而这几年渐渐认识到，“牛人”评价事物经常非常片面，带有很多主观情绪。很多“好东西”，你用了才知道其实毛病不少，只是这些毛病被选择性忽视了。

“好东西”其实不好，那么“坏东西”是不是也不坏呢？

于是我开始接触一些之前瞧不起的东西，趁着公司后台服务的重构，顺水推舟抛开Node.js，学习Spring。


## XML和Annotation

学习Spring碰到的最大麻烦，在于第三方资料大都是讲Spring2.5版本之前，通过XML实现DI的方式，而新的Spring都是基于Java Annotation。

后来发现，这种劣势其实也是一种优势，它提供了两种不同的视角来观察DI，让我看到究竟是什么因素导致了解耦，从而更容易把握概念的本质。

从使用上来讲，初步感觉是，XML的方式更好理解，因为更多的控制权在使用者这里，用起来更像是一个“库”而非“框架”。Annotation的方式用起来更方便，但是理解麻烦一些，恐怕需要看Spring源码才能清楚具体机制。

对和我一样初步接触Spring的人，我建议从XML的方式出发，这个更容易把握到其运行机制，算是个捷径吧。


## Bean

Bean算是Spring的基础了，具体解释网上一大片，且基本一致，不存在争议。如果是使用XML方式，这个概念会非常清晰。

但如果使用Annotation，理解可能就没那么顺利了，当你以为@Component用在“类”上是声明一个Bean，却发现还有个@Bean用在“方法”上，似乎也是声明一个Bean。

我在[这篇博客][reference]上找到了一个清晰的梳理。Bean按使用来说有两个操作：一是定义（或者叫“注册”），一是使用。

对于XML方式，beans.xml里的<bean>标签，就是Bean的定义；而appCtx.getBean("x")方法的调用，就是Bean的获取，即Bean的使用。

对于Annotation方式，@Component, @Controller等注解标注的类，就是Bean的定义；@Autowired标注的属性，就是Bean的使用。

这样一对应，理解起来就清晰多了。


[reference]: http://www.cnblogs.com/bossen/p/5824067.html

