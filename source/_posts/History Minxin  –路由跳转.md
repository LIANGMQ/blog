---
title: History Minxin  --路由跳转
date: 2016-12-18
tags: [前端,react]
categories: react
---

在组建中添加路由的history对象
**注意：你的Router components不需要这个minxin，他在this.props.history中已经可用。这是为了组件更深次的渲染树中需要访问路由的hiatory对象。**

<!--more-->

#### Method
1. `pushState(state, pathName, query)`
跳转到另一个新的URL。
**arguments**
  - `state` - location 的 state。
  - `pathname` - 没有 query 完整的 URL。
  - `query` - 通过路由字符串化的一个对象。
2. `reqlaceState(state, pathName, query)`
在不影响history的长度的情况下（一个重定向），用新的URL替换当前这个。
3. `go(n)`
在history中使用`n`或`-n`进行前进或后退。
4. `goBack()`
在 history 中后退。
5. `goForward()`
在 history 中前进。
6. `createPath(pathname, query)`
使用路由配置，将 query 字符串化加到路径名中。
7. `createHref(pathname, query)`
使用路由配置，创建一个 URL。例如，它会在 pathname 的前面加上 #/ 给 hash history。
8. `isActive(pathname, query, indexOnly)`
根据当前路径是否激活返回 true 或 false。通过 pathname 匹配到 route 分支下的每个 route 将会是 true（子 route 是激活的情况下，父 route 也是激活的），除非 indexOnly 已经指定了，在这种情况下，它只会匹配到具体的路径。
arguments
- `pathname` - 没有 query 完整的 URL。
- `query` - 如果没有指定，那会是一个包含键值对的对象，并且在当前的 query 中是激活状态的 - 在当前的 query 中明确是 undefined 的值会丢失相应的键或 undefined
- `indexOnly` - 一个 boolean（默认：false）。

Example：
```
import { History } from 'react-router'

React.createClass({
  mixins: [ History ],
  render() {
    return (
      <div>
        <div onClick={() => this.history.pushState(null, '/foo')}>Go to foo</div>
        <div onClick={() => this.history.replaceState(null, 'bar')}>Go to bar without creating a new history entry</div>
        <div onClick={() => this.history.goBack()}>Go back</div>
     </div>
   )
 }
})
```
