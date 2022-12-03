---
title: Redux
date: 2016-12-20
tags: [前端,react]
categories: react
---

> 核心概念
> - 所有的状态存放在Store。组件每次重新渲染，都必须由状态变化引起。
> - 用户在 UI 上发出action。
> - reducer函数接收action，然后根据当前的state，计算出新的state。

<!--more-->

##### Redux 应用的架构

<img src='/img/1482389627719.png' />
Redux 层保存所有状态，React 组件拿到状态以后，渲染出 HTML 代码。

##### Reducer机制

<img src='/img/1482389496876.png' />
