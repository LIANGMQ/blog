---
title: React生命周期
date: 2016-12-16
tags: [前端,react]
categories: react
---

> 在组件的整个生命周期中，随着该组件的props或者state发生改变，其DOM表现也会有相应的变化。一个组件就是一个状态机，对于特定地输入，它总返回一致的输出。其中一共提供了10个不同的周期（API）

<!--more-->

一个React组件的生命周期分为三个部分：实例化、存在期和销毁期。
<img src='/img/1482306008922.png' />

#### 实例化
---

当组件在客户端被实例化，第一次被创建时，以下方法依次被调用：
1. getDefaultProps
2. getInitialState
3. componentWillMount
4. render
5. componentDidMount

> 实例化完成后的更新（一个组件只执行一次**getDefaultProps**）
	1. getInitialState
	2. componentWillMount
	3. render
	4. componentDidMount

当组件在服务端被实例化，首次被创建时，以下方法依次被调用：
1. getDefaultProps
2. getInitialState
3. componentWillMount
4. render

componentDidMount不会再服务器端被渲染的过程中调用。

##### `getDefaultProps`
对于每个组件实例来讲，这个方法只会调用一次，该组件类的所有后续应用，getDefaultMount将不会在被调用，其返回对象可以用与设置默认的props值，会在实例中共享。
##### `getInitialState`
对于组件的每个实例来说，这个方法的调用**有且只有一次**，用来初始化每个实例的 state，在这个方法里，可以访问组件的 props。每一个React组件都有自己的 state，其与 props 的区别在于 state只存在组件的内部，props 在所有实例中共享。

getInitialState 和 getDefaultPops 的调用是有区别的，getDefaultPops 是对于**组件类**来说只调用一次，后续该类的应用都不会被调用，而 getInitialState 是对于**每个组件实例**来讲都会调用，并且只调一次。
```
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```
每次修改state，都会重新渲染组件，实例化后通过state更新组件，会依次调用下列方法:
1. shouldComponentUpdate
2. componentWillUpdate
3. render
4. componentDidUpdate

**更改this.state时用this.setState方法来修改。**
##### `componentWillMount`
该方法在首次渲染之前调用，也是在render调用之前修改state的最后一次机会。
##### `render`
该方法会创建一个虚拟DOM，用来表示组件的输出。对于一个组件来讲，render方法是唯一一个必需的方法。render方法需要满足下面几点：
1. 只能通过 this.props 和 this.state 访问数据（不能修改）
2. 可以返回 null,false 或者任何React组件
3. 只能出现一个顶级组件，不能返回一组元素
4. 不能改变组件的状态
5. 不能修改DOM的输出

render方法返回的结果并不是真正的DOM元素，而是一个虚拟的表现，类似于一个DOM tree的结构的对象。react之所以效率高，就是这个原因。
##### `componentDidMount`
该方法不会在服务端被渲染的过程中调用。该方法被调用时，已经渲染出真实的 DOM，可以再该方法中通过 **ReactDOM.findDOMNode() **访问到真实的 DOM(也可以使用 **this.getDOMNode()**)。
> 由于 this.refs.[refName] 属性获取的是真实 DOM ，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错，或者用nextTick()方法。

#### 存在期
---
此时组件已经渲染好并且用户可以与它进行交互，比如鼠标点击，手指点按，或者其它的一些事件，导致应用状态的改变，你将会看到下面的方法依次被调用：
1. componentWillReceiveProps
2. shouldComponentUpdate
3. componentWillUpdate
4. render
5. componentDidUpdate

##### `componentWillReceiveProps`
组件的props属性可以通过父组件来更改，这是，componentWillReceiveProps将被调用。可以在这个方法里更新 state,以触发 render 方法重新渲染组件。
```
componentWillReceiveProps: function(nextProps){
    if(nextProps.checked !== undefined){
        this.setState({
            checked: nextProps.checked
        })
    }
}
```
##### `shouldComponentUpdate`
如果你确定组件的 props 或者 state 的改变不需要重新渲染，可以通过在这个方法里通过返回 false 来阻止组件的重新渲染，返回 false 则不会执行 render 以及后面的 componentWillUpdate，componentDidUpdate 方法。
这种方法在大多数情况下没在开发中使用。

##### `componentWillUpdate`
这个方法和 componentWillMount 类似，在组件接收到了新的 props 或者 state 即将进行重新渲染前，componentWillUpdate(object nextProps, object nextState) 会被调用，注意不要在此方面里再去更新 props 或者 state。
##### `componentDidUpdate`
这个方法和 componentDidMount 类似，在组件重新被渲染之后，componentDidUpdate(object prevProps, object prevState) 会被调用。可以在这里访问并修改 DOM。

#### 销毁时
---
##### 	`componentWillUnmount`
每当React使用完一个组件，这个组件必须从 DOM 中卸载后被销毁，此时 componentWillUnmout 会被执行，完成所有的清理和销毁工作，在 componentDidMount 中添加的任务都需要在该方法中撤销，如创建的定时器或事件监听器。
