---

title: Async/Await
date: 2017-1-5
tags: [前端,JavaScript]
categories: JavaScript

---


> await/async 是 ES7 最重要特性之一，相对于Ajax、Promise、Generators等，它是目前为止 JS 最佳的异步解决方案（回调函数、事件监听、发布/订阅、Promise对象）了。

**插一句**：这里我就只介绍下`Generators`，详细可以去看阮老师的[Generators函数的含义与用法](http://www.ruanyifeng.com/blog/2015/04/generator.html)

<!--more-->

##### 强劲的Generators
> 小总结：调用Generator函数，返回一个遍历器对象，代表Generator函数的内部指针。以后，每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield语句后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。

```
//单纯的暂缓执行函数（不包含next方法）
function* f() {
  console.log('执行了！')
}

var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);
//输出结果为：2秒以后打印“执行了”。
```
```
//generator函数(包含next方法)
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
//输出为：
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```
---
言归正传，下面来看下`Async/Await`方法
看个例子：
```
var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve();
        }, time);
    })
};

var start = async function () {
    // 在这里使用起来就像同步代码那样直观
    console.log('start');
    await sleep(3000);
    console.log('end');
};

start();

//输出结果：控制台先输出start，稍等3秒，输出end
```

##### 基本规则
1. `async`表示这是一个`async函数`，**await只能用在这个函数里面**。

2. `await` 表示在这里等待promise返回结果了，再继续执行。

3. `await` 后面跟着的应该是一个**`promise对象`**（当然，其他返回值也没关系，只是会立即执行，不过那样就没有意义了…）

##### 返回值
await等待的虽然是promise对象，但不必写.then(..)，直接可以得到返回值。
```
var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            // 返回 ‘ok’
            resolve('ok');
        }, time);
    })
};

var start = async function () {
    let result = await sleep(3000);
    console.log(result);
};
// 3秒以后收到 ‘ok’
```
##### 捕捉错误
直接用标准的	`try catch`语法捕捉错误。
```
var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            // 模拟出错了，返回 ‘error’
            reject('error');
        }, time);
    })
};

var start = async function () {
    try {
        console.log('start');
        await sleep(3000); // 这里得到了一个返回错误

        // 所以以下代码不会被执行了
        console.log('end');
    } catch (err) {
        console.log(err); // 这里捕捉到错误 `error`
    }
};
```
##### 循环多个**`await`**
await看起来像同步代码，所以可以理所当然的**写在`for循环`里**，不必担心以往需要**闭包**才能解决的问题。
```
..省略以上代码

let arr = [1,2,3,4,5,6,7,8,9,10];

// 错误示范
arr.forEach(function (v) {
    console.log(`当前是第${v}次等待..`);
    await sleep(1000); // 错误!! await只能在async函数中运行
});

// 正确示范
for(var v of arr) {
    console.log(`当前是第${v}次等待..`);
    await sleep(1000); // 正确, for循环的上下文还在async函数中
}
```
> 结论：await只能`在async函数的上下文`中。

##### 结合Fetch
```
(async () => {
    let parallelDataFetch = await* [
        (await fetch('products.json')).json(),
        (await fetch('users.json')).json(),
        (await fetch('comments.json')).json()
    ];
    console.log('Async parallel+fetch >>>', parallelDataFetch);
}());
```

**`promise与async比较`**
async本身就是基于Promise的语法糖。如果你懒得看语法细节，监测当前上下文能否支持async的代码里就能看出这一点：
```
function(){
return (async function(){
  return 42;
})() instanceof Promise
}   //返回true
```
然而，很多Promise方法你不能用async来定义，而只能用Promise的方式来定义。Promise就是用于解决回调函数的了，而async的写法没办法在异步回调里插入`resolve/reject`...
```
function returnPromise(){
    return new Promise(function(resolve,reject){
        setTimeout(function(){resolve(42)})
    })
}
//这个用async是不能改写的。
```
