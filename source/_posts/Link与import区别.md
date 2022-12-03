---
title: link和@import的区别
date: 2014-7-26
tags: [前端]
categories: 前端
---

页面中使用CSS的3种方式：行内添加定义style属性值，页面头部内嵌调用和外面链接调用。
外面引用分为两种：link和@import。
```
XML/HTML代码
<link rel="stylesheet" rev="stylesheet" href="CSS文件" type="text/css" media="all" />   
XML/HTML代码
<style type="text/css" media="screen">   
@import url("CSS文件");   
</style>  
```
<!--more-->

两者都是外部引用CSS的方式，但是存在一定的区别：
1. link是XHTML标签，除了加载CSS外，还可以定义RSS等其他事务；@import属于CSS范畴，只能加载CSS。
2. link引用CSS时，在页面载入时同时加载；@import需要页面网页完全载入以后加载。
3. link是XHTML标签，无兼容问题；@import是在CSS2.1提出的，低版本的浏览器不支持。
4. link支持使用Javascript控制DOM去改变样式；而@import不支持。
