

![1650439500125](C:\Users\win 10\AppData\Roaming\Typora\typora-user-images\1650439500125.png)

## 为什么会出现 React fiber 架构

 React 15 `Stack Reconciler` 是通过`递归`更新子组件 。由于递归执行，所以更新`一旦开始`，中途就`无法中断`。当层级很深时，递归更新时间超过了 `16ms`，用户交互就会卡顿。 

在`setState`后，react 会立即开始`reconciliation`过程，从`父节点`（Virtual DOM）开始`遍历`，以找出不同。将所有的`Virtual DOM遍历完`成后，reconciler`才`能给出当前需要`修改真实DOM的信息`，并传递给`renderer`，进行渲染，然后屏幕上才会显示此次`更新`内容。

## React Fiber 是什么？

官方的一句话解释是“`React Fiber是对核心算法的一次重新实现`”。Fiber 架构调整很早就官宣了，但官方经过两年时间才在`V16版本`正式发布。官方概念解释太笼统， 其实简单来说 `React Fiber` 是一个`新的任务调和器`（`Reconciliation`）简称`协调`

简单理解就是把一个`耗时长的任务`分解为一个个的工作单元（每个工作单元运行时间很短，不过总时间依然很长）。在执行工作单元之前，由`浏览器判断是否有空余时间执行`，`有时间就执行`工作单元，执行完成后，继续判断是否还有空闲时间。`没有时间就终止执行`让浏览器执行其他任务（如 GUI 线程等）。等到下一帧执行时判断是否有空余时间，有时间就从终止的地方继续执行工作单元,`一直重复`到任务结束。

> `Fiber架构` = `Fiber节点` + `Fiber调度算法` 

要让终止的任务`恢复执行`，就必须知道`下一工作单元`对应那一个。所以要实现工作单元的连接，就要使用`链表`，在每个工作单元中`保存下一个工作单元的指针`，就能恢复任务的执行。

要知道每一帧的空闲时间，就需要使用 `requestIdleCallback` Api。传入回调函数，回调函数接收一个参数（剩余时间），如果有剩余时间，那么就执行工作单元，如果时间不足了，则继续`requestIdleCallback`，等到下一帧继续判断。

>  📚`所以 Fiber 架构就是用 异步的方式解决旧版本 同步递归导致的性能问题`。 

`requestIdleCallback`被React重写了，使用到的底层API主要是 requestAnimationFrame 和 MessageChannel



如何实现中断继续

```javascript

if(fiber.child) {
    return fiber.child
}

let nextFiber = fiber

while(nextFiber) {
    if(nexNode.sibling) {
        return nextFiber.sibling
    }
    nextFiber = nextFiber.return
}
```



## react-router

单页面应用路由实现原理是，切换url，监听url变化，从而渲染不同的页面组件。

主要的方式有`history`模式和`hash`模式。

### 1 history模式原理

#### ①改变路由

```
**history.pushState**
history.pushState(state,title,path)
复制代码
```

1 `**state**`：一个与指定网址相关的状态对象， popstate 事件触发时，该对象会传入回调函数。如果不需要可填 null。

2 `**title**`：新页面的标题，但是所有浏览器目前都忽略这个值，可填 null。

3 `**path**`：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个地址。

```
**history.replaceState**
history.replaceState(state,title,path)
复制代码
```

参数和`pushState`一样，这个方法会修改当前的` history `对象记录， `history.length` 的长度不会改变。

#### ②监听路由

```
**popstate事件**
window.addEventListener('popstate',function(e){
    /* 监听改变 */
})
复制代码
```

同一个文档的 `history` 对象出现变化时，就会触发` popstate` 事件  `history.pushState` 可以使浏览器地址改变，但是无需刷新页面。**注意⚠️的是：用 `history.pushState()` 或者 `history.replaceState()` 不会触发 `popstate` 事件**。 `popstate` 事件只会在浏览器某些行为下触发, 比如点击后退、前进按钮或者调用 `history.back()、history.forward()、history.go()`方法。

### 2 hash模式原理

#### ①改变路由

```
**window.location.hash**
```

通过`window.location.hash ` 属性获取和设置 `hash `值。

#### ②监听路由

```
**onhashchange**
window.addEventListener('hashchange',function(e){
    /* 监听改变 */
})
```

![1650527991864](C:\Users\win 10\AppData\Roaming\Typora\typora-user-images\1650527991864.png)

### hooks为什么要写到最外边

- hooks必须按照统一的顺序执行，因为每调用一个hooks都会在该组件的hooks列表中push一条数据，用于记录上次的数据
- 组件更新，hooks再次执行的时候，会按照调用的顺序作为index来对比hooks列表中的数据对比，此时index要保证一一对应
- 如果在条件语句中引用hooks，每次条件不同，那么hooks执行的顺序就会不一致，对比数据就会出现顺序混乱
- hooks不在最外层调用的时候，react会报错。

## Redux

```
function createStore(reducer) {
 let currentState
 let subscribers = []

 function dispatch(action) {
   currentState = reducer(currentState, action);
   subscribers.forEach(s => s())
 }

 function getState() {
   return currentState;
 }

 function subscribe(subscriber) {
     subscribers.push(subscriber)
     return function unsubscribe() {
         ...
     }
 }

 dispatch({ type: 'INIT' });

 return {
   dispatch,
   getState,
 };
}
```

# React 中 setState 什么时候是同步的，什么时候是异步的？

在React中，**如果是由React引发的事件处理（比如通过onClick引发的事件处理），调用setState不会同步更新this.state，除此之外的setState调用会同步执行this.state** 。所谓“除此之外”，指的是绕过React通过addEventListener直接添加的事件处理函数，还有通过setTimeout/setInterval产生的异步调用。

**原因：** 在React的setState函数实现中，会根据一个变量isBatchingUpdates判断是直接更新this.state还是放到队列中回头再说，而isBatchingUpdates默认是false，也就表示setState会同步更新this.state，但是，**有一个函数batchedUpdates，这个函数会把isBatchingUpdates修改为true，而当React在调用事件处理函数之前就会调用这个batchedUpdates，造成的后果，就是由React控制的事件处理过程setState不会同步更新this.state**。

**注意：** setState的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。

```js
class Example extends React.Component {
  constructor() {
    super();
    this.state = {
      val: 0
    };
  }
  
  componentDidMount() {
    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 1 次 log

    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 2 次 log

    setTimeout(() => {
      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 3 次 log

      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 4 次 log
    }, 0);
  }

  render() {
    return null;
  }
};
```

1、第一次和第二次都是在 react 自身生命周期内，触发时 isBatchingUpdates 为 true，所以并不会直接执行更新 state，而是加入了 dirtyComponents，所以打印时获取的都是更新前的状态 0。

2、两次 setState 时，获取到 this.state.val 都是 0，所以执行时都是将 0 设置成 1，在 react 内部会被合并掉，只执行一次。设置完成后 state.val 值为 1。

3、setTimeout 中的代码，触发时 isBatchingUpdates 为 false，所以能够直接进行更新，所以连着输出 2，3。

输出： 0 0 2 3

