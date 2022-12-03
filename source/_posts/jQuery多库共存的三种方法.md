---
title: jQuery多库共存的三种方法
date: 2015-2-26
tags: [JavaScript]
categories: JavaScript
---

jQuery团队为用户提供了贴心的方法让jQuery能与其他js库(如Prototype)，实现起来很简单。

其实，多库共存就是为了解决“$ ”符号的冲突。

<!--more-->

#### 方法一：
利用jQuery的实用函数$.noConflict();这个函数归还$的名称控制权给另一个库，因此可以在页面上使用其他库。

这时，我们可以用"jQuery "这个名称调用jQuery的功能。
举个栗子：
```
$.noConflict();   
jQuery('#id').hide();
或者给jQuery一个别名
var $j=jQuery;
$j('#id').hide();
```
#### 方法二：
利用自执行函数`(function($){/*代码块*/})(jQuery)`
其实就是函数自执行改变改变`$`作用域的技巧
再举个栗子：
```
//声明函数  
var fun=function($){/*代码块*/};  
//调用函数  
fun(jQuery);
```
#### 方法三：
`jQuery(function($){/*代码块*/})`
通过传递一个函数作为jQuery的参数，因此把这个函数声明为就绪函数。

那它究竟是怎么工作的呢？

我们声明$为就绪函数的参数，因为jQuery总是吧jQuery对象的引用作为第一个参数传递，所以就保证了函数的执行。
