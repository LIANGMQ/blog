---
title: 初试React Router
date: 2016-12-17
tags: [前端,react]
categories: react
---

> React Router(路由库)——它是官方维护的，事实上也是唯一可选的路由库。它通过管理 URL，实现组件的切换和状态的变化，开发复杂的应用几乎肯定会用到。React Router 保持 UI 与 URL 同步。它拥有简单的 API 与强大的功能例如代码缓冲加载、动态路由匹配、以及建立正确的位置过渡处理

实例库可以参见官方的[示例库](https://github.com/reactjs/react-router-tutorial/tree/master/lessons)，共分为14部分，由浅到深；
[React Router中文API](https://react-guide.github.io/react-router-cn/docs/API.html#named-components)

<!--more-->

##### 1. React技术栈
<img src="/img/1482214350484.png"/>
`React技术栈`主要包括：ES6、Babel（解析jsx，react，es6等）、React、Webpack、Flex（布局）、SCSS（Less）、React-Router、Flux（Redux实现）、测试等

#### React Router
---
##### 2. 基本用法
React Router 安装命令：
`npm install --save react-router`
使用时，路由器Router就是React的一个组件。
```
import { Router } from 'react-router';
render (<Router />,document.getElementById("app"));
```
Router本身只是一个容器，真正的路由要通过Route组件定义。
```
import { Router, Route, hashHistory } from 'react-router';
render((
  <Router history={hashHistory}>
    <Route path="/" component={App}/>
  </Router>
), document.getElementById('app'));
```
> 其中Router有一个参数history，它的值hashHistory表示，路由的切换有URL的hash变化决定，即URl的#部分发生变化。
> 例如：用户访问`http://www.hao123.com/`,实际看到的是`http://www.hao123.com/#/`
##### 3. 嵌套路由
Router组件还可以嵌套。
```
<Router history={hashHistory}>
  <Route path="/" component={App}>
    <Route path="/repos" component={Repos}/>
    <Route path="/about" component={About}/>
  </Route>
</Router>
```
上面代码中，用户进入默认加载App组件，当访问`/repos`时，会在App组件内部加载Repos组件。
```
<App>
	<Repos />
</App>
```
Router组件还可以写成下面的样子
```
let routes =
  <Route path="/" component={App}>
    <Route path="/repos" component={Repos}/>
    <Route path="/about" component={About}/>
  </Route>;
<Router routes={routes} history={hasHistory}/>
```
或者(更推荐)
```
const routeConfig = [
  { path: '/',
    component: App,
    indexRoute: { component: Dashboard },
    childRoutes: [
      { path: 'about', component: About },
      { path: 'inbox',
        component: Inbox,
        childRoutes: [
          { path: '/messages/:id', component: Message },
          { path: 'messages/:id',
            onEnter: function (nextState, replaceState) {
              replaceState(null, '/messages/' + nextState.params.id)
            }
          }
        ]
      }
    ]
  }
]

React.render(<Router routes={routeConfig} />, document.body)
```
App组件可以这样写
```
export default React.createClass({
  render() {
    return (
      <div>
        <h1>React Router Tutorial</h1>
        <ul role="nav">
          <li><Link to="/about">About</Link></li>
          <li><Link to="/repos">Repos</Link></li>
        </ul>
      </div>
    )
  }
})
```
##### 4. path属性
path属性用来指定路由的匹配规则，这个属性该也可以省略。举个栗子：
```
<Route path="index" component={Inbox}>
  <Route path="messages/:id" component={Message} />
</Route>
```
<p style="color:red"> messages/:id 前面一定没有 /  ,否则匹配的路径是 /messages/:id ,不再是 index/messages/:id</p>
与
```
<Route component={Inbox}>
  <Route path="index/messages/:id" component={Message} />
</Route>
```
是等价的。

##### 5. 通配符
```
<Route path="/hello/:name">
// 匹配 /hello/michael
// 匹配 /hello/ryan

<Route path="/hello(/:name)">
// 匹配 /hello
// 匹配 /hello/michael
// 匹配 /hello/ryan

<Route path="/files/*.*">
// 匹配 /files/hello.jpg
// 匹配 /files/hello.html

<Route path="/files/*">
// 匹配 /files/
// 匹配 /files/a
// 匹配 /files/a/b

<Route path="/**/*.jpg">
// 匹配 /files/hello.jpg
// 匹配 /files/path/to/file.jpg
```
通配符规则如下：
1. `:paramName`
:paramName匹配URL的一个部分，直到遇到下一个/、?、#为止。这个路径参数可以通过this.props.params.paramName取出。
2. `()`
()表示URL这个部分是可选的。
3. `*`
`*`匹配任意字符，知道模式里面的下一个字符为止。匹配方式为费贪婪模式。
4. `**`
**匹配任意字符，直到查到下一个`/`、`?`、`#`为止，匹配方式是贪婪方式。

**path属性也可以使用相对路径（不以`/`开头），匹配到就会相对于父组件的路径。**
**路由匹配规则从上到下执行，一旦匹配就不再匹配其余的规则。故设置参数时特别注意将<span style="color:red;">带参数的路径一般写在路由规则的底部</span>。**

> 1. URL的`query`查询字符串`/foo？bar=baz`，可以用`this.props.location.query.bar`获取baz值；
> 2. `/inbox/messages/:id`要用`this.props.params.id`来获取id值；
> 3. URL地址可以用`event.target.elements[0].value`和`this.refs.refName.value`来获取。
##### 6. IndexRoute组件（默认路由）
```
<Router>
  <Route path="/" component={App}>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```
上面代码访问根路径`/`,不会加载任何子组件。App组件的`this.props.children`为`null`。
而IndexRoute就是来解决这个问题，显示指定Home是路由跟组件。（也可以用`{this.props.children || }`来渲染一些默认的UI组件 ）
```
<Router>
  <Route path="/" component={App}>
    <IndexRoute component={Home}/>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```
##### 7. Redirect组件
`<Redirect>`组件用于路由的跳转，即用户访问一个路由，会自动跳转到另一个路由。
```
<Route path="inbox" component={Inbox}>
  {/* 从 /inbox/messages/:id 跳转到 /messages/:id */}
  ＜Redirect from="messages/:id" to="/messages/:id" />
</Route>
```
现在访问`/index/message/5`，会自动跳转到`/message/5`。
##### 8. Link
Link组件用于取代`<a>`元素，生成一个链接，允许用户点击后跳转到另一个路由。它基本上就是`<a>`元素的React版本，可以接受Router的状态。
```
render() {
  return <div>
    <ul role="nav">
      <li><Link to="/about">About</Link></li>
      <li><Link to="/repos">Repos</Link></li>
    </ul>
  </div>
}
```
浏览器解析成
```
<div id="app">
 <div data-reactid=".0">
   <h1 data-reactid=".0.0">React Router Tutorial</h1>
   <ul role="nav" data-reactid=".0.1">
     <li data-reactid=".0.1.0">
       <a href="#/about" data-reactid=".0.1.0.0">About123</a>
     </li>
     <li data-reactid=".0.1.1">
        <a href="#/repos" data-reactid=".0.1.1.0">Repos</a>
      </li>
    </ul>
   </div>
 </div>
```
如果希望当前路由与其他路由有不同样式，可以用Link的activeStyle属性，或者使用activeClassName指定当前路由的Class。

##### 9. IndexLink
> 个人认为用法基本等同于IndexRoute

连接到根路由，对于根路由来说，activeStyle和activeClassName会失效，或者说总是生效，因为/会匹配任何子路由。而IndexLink组件会使用路径的精确匹配。
另一种方法是使用组建的`onlyActiveOnIndex`属性，也能达到同样效果。
实际上，indexlink就是对link组建的onlyacticeonindex属性的包装。

##### 10. history属性
前面刚看到Router组建的history属性，可否还有印象？
history属性用来监听浏览器地址的变化，并将URl解析成一个地址对象，供React Router 匹配。

history属性分为三种值：browserHistory、hashHistory、creatMemoryHistory

1. hashHistory
路有将URL的hash部分（#）切换，URL的形式类似`example.com/#/some/path`。
2. browserHistory
浏览器的路由就不再通过Hash完成了，而显示正常的路径`example.com/some/path`，背后调用的是浏览器的`History API`。
3. createMemoryHistory
主要用于服务器渲染。它创建一个内存中的history对象，不与浏览器URL互动。

##### 11. 表单处理
> Link组件用于正常的用户点击跳转，但是有时还需要表单跳转、点击按钮跳转等操作。这些情况怎么跟React Router对接呢？

```
<form onSubmit={this.handleSubmit}>
  <input type="text" placeholder="userName"/>
  <input type="text" placeholder="repo"/>
  <button type="submit">Go</button>
</form>
```
第一种方法：browserHistory.push
```
import { browserHistory } from 'react-router';
  handleSubmit(event) {
    event.preventDefault()
    const userName = event.target.elements[0].value
    const repo = event.target.elements[1].value
    const path = `/repos/${userName}/${repo}`
    browserHistory.push(path)
  },
```
第二种方法：context
```
export default React.createClass({

  // ask for `router` from context
  contextTypes: {
    router: React.PropTypes.object
  },

  handleSubmit(event) {
    event.preventDefault()
    const userName = event.target.elements[0].value
    const repo = event.target.elements[1].value
    const path = `/repos/${userName}/${repo}`
    this.context.router.push(path)
  },
})
```
##### 12. 路由钩子
每个路由都有Enter和Leave 钩子，用户进入或者离开时该路由触发。
例子：使用onEnter代替`<Redirect>`组件(还可以用来认证)。
```
<Route path="inbox" component={Inbox}>
  <Route
    path="messages/:id"
    onEnter={
      ({params}, replace) => replace(`/messages/${params.id}`)
    }
  />
</Route>
```

---
链接：
1. 阮老师[React Router 使用教程](http://www.ruanyifeng.com/blog/2016/05/react_router.html)编写。
2. [React Router](https://react-guide.github.io/react-router-cn/index.html)
