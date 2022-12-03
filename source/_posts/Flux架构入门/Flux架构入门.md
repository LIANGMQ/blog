---
title: Flux架构入门
date: 2016-12-19
tags: [前端,react]
categories: react
---

> 在 React 的基础上，使用 Flux 组织代码和安排内部逻辑，使得你的应用更易于开发和维护。

##### 一、基本概念
<img src="/img/1482309477147.png" />

<!--more-->

1. Flux分为四部分：
- View：视图层
- Action（动作）：视图层发出的消息（各种事件）
- Dispatcher（派发器）：用来接收Action、执行回调函数
- Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面。

2. **最大特点**：数据单向流动——与MVVM的数据双向绑定形成鲜明对比。
	1. 用户访问View
	2. View 发出用户的Action
	3. Dispatcher 收到Action，要求Store进行相应的更新
	4. Store更新后，发出一个‘chenge’事件
	5. View 收到“change”事件后，更新页面

3. 其中最流行的React实现：
`Redex`:函数式管理（与MobX响应式管理对比<state是可变对象>）state是不可变对象，适合大型项目；<img src="/img/1482309298325.png" />

##### 二、详说Flux四部分（[点我有源码](https://github.com/ruanyf/extremely-simple-flux-demo)）
在组件绑定`Action`事件，当调用该事件时会被`Action`的事件定义的`Dispatcher`派发到`store`，从而`store`进行数据修改，引发`this.on`监听，触发`this.emit()`，进而进行事件的广播，进行所有绑定事件监听的组件`this.store`数据更新。
1. `View`
视图层，包括各个组件
2. `Action`
每个Action都是一个对象，包含一个`actionType`属性（说明动作的类型）和一些其他属性（用来传递数据）。
**Action中方法使用Dispatcher把动作派发到Store。**
3. `Dispather`
Dispatcher 的作用是将 Action 派发到 Store。你可以把它看作一个路由器，负责在 View 和 Store 之间，建立 Action 的正确传递路线。注意，Dispatcher 只能有一个，而且是全局的。

	`AppDispatcher.register()`方法用来登记各种Action的回调函数。

	`Dispatcher`收到`ADD_NEW_ITEM`动作，就会执行回调函数，对ListStore进行操作。

	**记住，Dispatcher 只用来派发 Action，不应该有其他逻辑。**

4. `Store`
	Store 保存整个应用的状态。

	ListStore继承了`EventEmitter.prototype`，因此就能使用`ListStore.on()`和`ListStore.emit()`，来监听和触发事件了。

	Store 更新后（`this.addNewItemHandler()`）发出事件（`this.emitChange()`），表明状态已经改变。 View 监听到这个事件，就可以查询新的状态，更新页面了。


文章借鉴阮老师[Flux架构入门教程](http://www.ruanyifeng.com/blog/2016/01/flux.html)
