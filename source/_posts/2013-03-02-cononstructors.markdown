---
layout: post
title: "[译]JS构造器"
date: 2012-07-22 21:04
comments: true
categories: 
---

“在JS中每一个对象都有一个构造器属性，它指向初始化该对象的构造函数”

这听起来不错，构造器就像java中的静态类一样，甚至连new Constructor()语法都一样。如下：
<pre> function MyConstructor() {}
 var myobject = new MyConstructor();
 myobject.constructor == MyConstructor;     // true</pre>
但是life isn't that simple（life's a struggle - -）:
<pre>function MyConstructor() {}
MyConstructor.prototype = {};
var myobject = new MyConstructor();
myobject.constructor == MyConstructor;  // false</pre>
why？下面有一些定义

<strong>对象和方法：</strong>

JS中的对象就是与属性的集合，你可以get或者set它。JS不存在类的概念。

在JS中函数是第一类对象。方法=作为属性的函数。

<strong>原型：</strong>

对象的原型其实是一种内部（internal）属性，“[[Prototype]]”指向这个属性（<a href="http://www.ecma-international.org/publications/standards/Ecma-262.htm">Ecma-262</a>）。换句话说，obj.<span style="font-family: Monaco, Consolas, 'Andale Mono', 'DejaVu Sans Mono', monospace; font-size: x-small;"><span style="line-height: normal;">prototype</span></span> 并不就是对象obj的[[Prototype]]属性。Ecma标准没有提供任何方法来从一个对象中获得这个[[Prototype]]属性。

JS对象可以将一些属性放在[[Prototype]]中。（而且[[Prototype]]可以做相同的事情,all the way up to Object.prototype）。

<strong>属性查找：</strong>

当一个对象的‘propname’属性被访问时，系统就会检查这个对象是不是含有一个叫‘propname’的属性。如果这个属性不存在，那么就会去找这个对象的[[Prototype]]是否存在那个属性，递归下去。

这意味着对象如果共享一个[[Prototype]]，那么[[Prototype]]上的属性也被共享。

当一个对象被设置了‘propname’属性，这个属性就存在于对象中，而不会去管这个对象的[[Prototype]]链。

“[[Prototype]]”是当构造函数被调用时通过构造函数上的“prototype”属性设置的。

&nbsp;

图解：

有关prototype和[[Prototype]]属性就是如下所示，椭圆代表对象，箭头代表属性关系，绿色的是[[Prototype]]链。
<h3><a name="_1__function_myconstructor_____"></a>#1: function MyConstructor() {}</h3>
<p style="text-align: center;"><img class="aligncenter" src="http://joost.zeekat.nl/graph1.png" alt="" width="447" height="298" /></p>
非常简单。MyConstructor的prototype属性是自动创建的，MyConstructor.prototype结果也是个对象，这个对象有个‘constructor’的属性反过来指向MyConstructor。记住：只有函数本身自带（自动创建）的prototype对象才有默认的constructor属性。

下面说的虽然和主题不相关但能迷惑、开导你（主要是这个目的）。

MyConstructor的[[Prototype]] 是 Function.prototype而不是MyConstructor.prototype。还需要注意的是每个对象[[Prototype]]链止于Object.prototype。

Object.prototype的[[Prototype]]事实上是null，表明这已经是链的终点了。

为了清晰起见，下一步我们不说[[Prototype]]链了。

#2：MyConstructor.prototype = {}

<img src="http://joost.zeekat.nl/graph2.png" alt="" />

我们废除了MyConstructor.protoype最初的对象，而用一个匿名的对象“{}”取代它，则个对象没有一个constructor属性。

#3：var myobject = new MyConstructor();

<img src="http://joost.zeekat.nl/graph3.png" alt="" />

这张图中，根据属性查找的规则，我们可以看到myobject.constructor委派给了Object.prototype.constructor,而它指向了Object。换言之：
<pre> function MyConstructor() {}
 MyConstructor.prototype = {};
 var myobject = new MyConstructor();
 myobject.constructor == Object</pre>
<strong>那instanceof呢？</strong>

Javascript提供了instanceof操作符用来检查某个对象的原型链。根据之前的结论你可能会认为下面会返回false。
<pre> function MyConstructor() {}
 MyConstructor.prototype = {};
 var myobject = new MyConstructor();
 myobject instanceof MyConstructor  // true</pre>
但事实上它返回true，还可以注意到myobject委托在Object.prototype上：
<pre>function MyConstructor() {}
 MyConstructor.prototype = {};
 var myobject = new MyConstructor();

 myobject instanceof Object</pre>
instanceof执行时，它检查对象构造器的prototype属性，再次检查对象的[[Prototype]]链，换言之，它不依赖于constructor属性。

看起来很不错吧，还不够！
<pre> function MyConstructor() {}
 var myobject = new MyConstructor();
 MyConstructor.prototype = {};

 [ myobject instanceof MyConstructor,     // false !
   myobject.constructor == MyConstructor, // true !
   myobject instanceof Object ]           // true</pre>
下图是执行过后的原型链：

<img src="http://joost.zeekat.nl/graph4.png" alt="" />

<strong>构造器不是类</strong>

在一个基于类的对象系统中，典型的，类互相继承，然后对象是那些类的实例。在两个实例间是共享的（至少理论上是的）方法和属性也是类的属性。不能共享的属性（有些语言称方法）那是对象自己的（私有）属性。

JS的构造器不是这样的，事实上，构造器有他们自己的[[Prototype]]链，而且和它们初始化出来的对象的[[Prototype]]链完全分开。

&nbsp;

<strong>原文：<a href="http://joost.zeekat.nl/constructors-considered-mildly-confusing.html">http://joost.zeekat.nl/constructors-considered-mildly-confusing.html</a></strong>
