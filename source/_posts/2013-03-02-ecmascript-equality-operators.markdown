---
layout: post
title: "[译]等运算符"
date: 2012-07-22 17:58
comments: true
categories: 
---
在这篇文章中将阐述一些等运算符的技术特点。

众所周知，在ECMAScript中‘等于’是不可传递的。

<strong>不可传递的等号</strong>

就像当：
<blockquote>A == B

B==C</blockquote>
在有些情况下这并不代表：
<blockquote>A==C</blockquote>
举个栗子：
<pre>console.log(
  '0'== 0, // true
   0  == '',   // true
  '0'== ''// false
);</pre>
再来一个：
<pre>var a = new String("foo");
var b = "foo";
var c = new String("foo");
console.log(
  a == b, // true
  b == c,   // true
  a == c // false
);</pre>
<div>使用'=='时会触发隐式的类型转换，所以导致了上述例子的结果。因此我们建议完全不使用‘==’，而应当用‘===’、即严格相等。另外，非严格相等‘==’被称为ECMAScript中的'邪恶部分'。<strong>严格‘===’与非严格‘==’</strong></div>
<div>当然了，我们的主要目标（这适用于所有语言）是改善语言的抽象性，以此更容易得理解语言的结构以及减少歧义。程序员不需要记很多关于简单运算符的知识来避免歧义。</div>
<div>最迷惑人的情况就是falsy values（就像0、''、null）和等式右边的布尔值比较的时候。在这种情况下甚至都没有ToBoolean隐式转换。为了解释清楚这个问题，我们先考虑一下在使用‘==’时的普通隐式类型转换。</div>
<div>null和undefined在下面的情况下总是相等。</div>
<pre>console.log(
  null== undefined, // true
  undefined == null// true
);</pre>
<div>一个比较表达式的操作数中若有一个是number类型的，那到最后（经过一些中间的类型转换）另外一个操作数总是会被转换成number类型。</div>
<div><strong>toNumber是‘==’的强制转换</strong></div>
<div>实际上，toNumber是‘==’主要使用到的转换。记住这个你就能很容易地确定下来隐式转换后的结果。如果两边的类型不一样，那就顺序给每一个操作数进行toNumber,直到类型都是number型，也就得到正确的结果了。</div>
<div>先从简单的toNumber转换开始</div>
<pre>console.log(
   1  == "1", // true -&gt; ToNumber("1") -&gt; 1 == 1
  "1"== 1, // the same, ToNumber("1") but for the first operand
);</pre>
<div>而对于对象的类型转换来说，先做toPrimitive，接着再toNumber（但除非toPrimitive没有返回一个string并且等式左边是一个string，那就不会再toNumber了，因为等式两边没有一个是number型啊没理由toNumber）。而toPrimitive依靠对象的内部方法[[DefaultValue]]，这个方法返回对象toString或者valueOf的结果。</div>
<pre>console.log(
  1 == {}, // false, default valueOf/toString
  1 == ({valueOf: function() {return1;}}), // true
  1 == ({toString: function() {return"1";}}), // true
  1 == [1] // true, with default valueOf/toString
);</pre>
<div><strong>Not ToBoolean but still ToNumber</strong></div>
<div>关于布尔型，还是那句话，没有toBoolean转换，依然是toNumber。</div>
<div>记住true的toNumber是1，false的toNumber是0。</div>
<pre>console.log(
  1 == true, // true, ToNumber(true) -&gt; 1 == 1
  0 == false, // true, ToNumber(false) -&gt; 0 == 0
  // more interesting examples
  "001"== true, // true, ToNumber("001") == ToNumber(true) -&gt; 1 == 1
  "002"== true, // false, by the same conversion chain -&gt; 2 != 1
  "0x0"== false, // true, the same (for hex-value and boolean) -&gt; 0 == 0
  ""== false, // true,
  " \t\r\n "== false// true
);</pre>
<div>在这种情况下推荐使用‘===’或者显示的toBoolean转换。</div>
<pre>console.log(
  !"0", // false
  !"0x0", // false
  "0"=== false// false
  // etc.
);</pre>
<div>注意，null和undefined在使用‘==’时也是不等于false的。还是因为toNumber而不是toBoolean（当然了更不会和false严格相等了、即’===‘）。<a href="http://bclary.com/2004/11/07/#a-11.9.3">11.9.3</a> 中的第19步，</div>
<blockquote>
<div>If Type(<em>y</em>) is Boolean, return the result of the comparison <em>x</em> == <a href="http://bclary.com/2004/11/07/#a-9.3">ToNumber</a>(<em>y</em>).</div></blockquote>
<div>然后返回false。</div>
<pre>console.log(
  null== false, // false, because null != ToNumber(false)
  undefined == false, // false, because undefined != ToNumber(false)
);</pre>
<div><strong>Boolean false object is true</strong></div>
<div>一些比较狗血的情况，当你new了一个object对象并传入false当做初始化参数，注意了，当使用Boolean函数或者是’!‘操作符都不会触发显式的类型转换。因为对于对象的toBoolean永远返回true。</div>
<div>栗子：</div>
<pre>var bool = newBoolean(false);
var data = !bool ? "false": "true"; // ToBoolean won't help
console.log(data); // "true"
data = bool === false? "false": "true";
console.log(data); // "true"
console.log(
  Boolean(bool), // true
  !!bool // true
);</pre>
<div>这种情况下使用’==‘会导致toNumber</div>
<pre>data = bool == false? "false": "true";
console.log(data); // "false"</pre>
<div>你也可以试试valueOf</div>
<pre>console.log(bool.valueOf() === false); // OK, false, as with ==</pre>
<div><strong>手动类型强制转换</strong></div>
<div>另外，你可以手动类型转换来比较string，number和boolean型。</div>
<pre>""+ a == ""+ b // comparing strings
+a == +b // comparing numbers
!a == !b // comparing booleans
// the same, but more human-read versions
String(a) == String(b)
Number(a) == Number(b)
Boolean(a) == Boolean(b)</pre>
<div>’+‘操作符调用操作数的valueOf方法，而’String‘调用toString方法。</div>
<pre>var foo = {
  toString: function() { return"toString"; },
  valueOf: function() { return"valueOf"; }
};
console.log(""+ foo); // "valueOf"
console.log(String(foo)); // "toString"</pre>
<div>尽管用’+‘连接的是“”这种空字符串，但最后的结果仍旧可以得到一个string类型。</div>
<div><strong>总是显式的使用‘===’</strong></div>
<div>之前已经提到过，避免使用‘==’，而应当只使用‘===’。因此你必须亲自来处理所有的类型转换。比如说：</div>
<pre>if(a === 1 || a.toString() === "1"|| a === true) {
  ...
}</pre>
<div>但如果用‘==’的话，会很简短：</div>
<pre>if(a == 1) {
  ...
}</pre>
<div>对于许多程序员来说，”显式优于隐式“这种思路是赞且灵活方便的。所以在编写ECMAScript代码中应当被接受并使用。</div>
<div><strong>‘==’的安全使用情况</strong></div>
<div>注意，还是有不少情况下‘===’并非必要，而且不会给代码带来任何鲁棒性。在使用ECMA-262规范下的标准行为及算法时，有些情况是完全安全的。举个被广泛使用但却无用的栗子:</div>
<pre>if(typeoffoo === "string") {
  ...
}</pre>
<div>有一件主要的事情你应当知道，那就是当操作数的类型是相同的情况下，‘==’和‘===’的处理机制是完全相同的。曾经有过一个测试 <a href="http://dmitrysoshnikov.com/ecmascript/the-quiz/#q2">quiz question</a> 。如果你对这个问题有兴趣的话可以查看ES3和ES5规范的相关章节。 <a href="http://bclary.com/2004/11/07/#a-11.9.3">11.9.3 The Abstract Equality Comparison Algorithm</a> 和<a href="http://bclary.com/2004/11/07/#a-11.9.6">11.9.6 The Strict Equality Comparison Algorithm</a>.</div>
<div><strong>总结</strong></div>
<div>好吧，如果不打算完全不用‘==’的话，那对使用‘==’和‘===’有什么一般建议呢？</div>
<div>还是像刚才说的，当你不确定一个函数的返回值或者一个比较变量的值，同时又要考虑类型或者是一个对象的特性，那就使用‘===’。而在其他情况下，包括完全安全的一些应用场景，比如typeof的运算结果去和string比较，这时‘==’就足够。</div>
<div>现在还是有一些类库（太坏了，就是在说jquery和prototype）在使用严格相等去比较typeof以及其他类似的可以完全用‘==’代替并且安全的结构。当然了也有类库（dojo，mootools，sencha）意识到了这一点而不这么做。</div>
<div>在一些细节优化上吹毛求疵的话，‘==’在比较相同类型的两个值时比‘===’来的更快。然而在现在许多的实现当中，这点差距实在是微不足道，甚至可以无视。此外，在其他一些实现中比较相等类型值‘===’甚至更快。在比较不同类型时，显然是‘===’更快。</div>
<div>当然了，一个系统的正常运行才是更关键的。所以，就像刚才说的，当你不确定一些操作或是函数的返回类型、但又不得不去考虑他们的时候，使用‘===’吧。客观的讲，我们可以想象一个程序员在某些情况下会不记得一些完全安全的‘==’用法比如typeof，所以他可能就选择‘===’来当做唯一的比较相等的操作符，这样就不用被‘==’的那些狗血尿性所扰，而专注于其他更重要的事情上去。</div>
<div><strong>延伸阅读</strong></div>
<div>
<ul>
	<li><a href="http://bclary.com/2004/11/07/#a-9">9. Type Conversion</a>;</li>
	<li><a href="http://bclary.com/2004/11/07/#a-11.9.3">11.9.3 The Abstract Equality Comparison Algorithm</a>;</li>
	<li><a href="http://bclary.com/2004/11/07/#a-11.9.6">11.9.6 The Strict Equality Comparison Algorithm</a>.</li>
</ul>
<div>原文：<a href="http://dmitrysoshnikov.com/notes/note-2-ecmascript-equality-operators/">http://dmitrysoshnikov.com/notes/note-2-ecmascript-equality-operators/</a></div>
</div>
