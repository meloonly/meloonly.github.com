---
layout: post
title: "null&amp;undefined"
date: 2012-07-09 20:43
comments: true
categories: 
---
ECMA-262中的定义：

Undefined
<blockquote>The Undefined type has exactly one value, called undefined. Any variable that has not been assigned a value</blockquote>
has the value undefined.

Null
<blockquote>The Null type has exactly one value, called null.</blockquote>
var a;

alert(a);//undefined

alert(typeof a);//undefined

alert(typeof b);//undefined

alert(typeof undefined);//undefined

alert(typeof null);//object  被认为是对象的占位符

alert(null == undefined);//true    EcmaScript中认为他们是相等的。

关于undefined

1、声明过变量在被赋值之前的默认值是undefined。

2、它是window的一个属性，alert('undefined' in window);//true。

3、可以在局部作用域中定义一个undefined达到优化目的。

4、函数未指明返回值（无return），也是undefined。

关于null

The null value is a primitive value that represents the null, empty, or non-existent reference.

EcmaScript的5种原始数值

Null Undefined Boolean Number String
