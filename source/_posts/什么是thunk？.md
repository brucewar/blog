---
title: 什么是thunk？
date: 2019-05-07 14:40:53
categories: React
tags: [React,redux,redux-thunk,thunk]
---
> 问：什么是`thunk`？
> 答：第一次听到这个词是通过`redux-thunk`

言归正传，当第一次听到Redux Thunk时，令人非常困惑。很可能就是因为`thunk`这个词。所以我们首先要弄清楚这一点。

### thunk

`thunk`是函数的另一个词，但它不仅仅只是我们所认识的函数。它是另一个函数返回的一个特殊的函数。如下所示：
```js
function wrapper_function() {
  // 这就是thunk，因为它把一些任务推迟做
  return function thunk() {
    console.log('do stuff now');
  };
}
```
你可能已经见过这样的代码，只是你不把它叫做`thunk`。如果想要执行“do stuff now”部分，你必须调用两次`wrapper_function()()`。

### redux-thunk

那么如何应用在Redux。
如果熟悉Redux，下面这些概念对你来说应该不陌生：**actions**，**action creators**，**reducers**和**middleware**。如果不熟悉的话，建议先看下[Redux基础教程](
https://redux.js.org/introduction/getting-started)。

在Redux中，**actions**只是一个普通的JavaScript对象，并且这个对象中包含`type`属性。除此之外，可以包含任何你想要的——描述要执行的action。
```js
// 1. 普通对象
// 2. 有type属性
// 3. 其他任意内容
{
  type: 'USER_LOGGED_IN',
  username: 'dave'
}
```
因为一直手工编写这些对象很烦人（更别说容易出错），Redux通过“action creators”这个概念解决：
```js
function userLoggedIn() {
  return {
    type: 'USER_LOGGED_IN',
    username: 'dave'
  };
}
```
两个完全相同的**action**，只是现在抽象了一层，你需要通过调用`userLoggedIn`函数创建。如果想在应用的不同位置触发同样的**action**，可以很方便的使用**action creator**。

### 烦人的Actions
Redux所谓的“actions”其实并没做任何事，它们只是普通的对象。如果让他们做点事岂不是更酷？比如说，API调用或者触发别的**actions**？
因为**reducers**应该是纯函数，不会改变其作用域范围之外的任何东西，所以我们不能在其内部做API调用或者触发**actions**。
如果你希望**action**执行某些操作，则这部分代码存在某个函数。这个函数（**thunk**）就是这一系列操作。
如果**action creator**能够返回一个函数（一系列的操作）而不是一个**action**对象岂不是更好，就像下面这样：
```js
function getUser() {
  return function() {
    return axios.get('/current_user');
  };
}
```
如果有一种方法可以告诉Redux把函数当作**actions**处理。
当然，这正是**redux-thunk**做的：它是一个**middleware**，系统中所有的action都会经过它，如果action是一个函数，就会调用那个函数。
上面那一小段代码中遗漏了一些东西，那就是Redux会传递两个参数给thunk函数：
* `dispatch`：可以用来触发新的action
* `getState`：访问当前的state
```js
function logOutUser() {
  return function(dispatch, getState) {
    return axios.post('/logout').then(function() {
      // 假设我们声明了一个叫 userLoggedOut 的 action creator，现在我们触发它
      dispatch(userLoggedOut());
    });
  }
}
```
依赖当前state，`getState`函数在决定是否获取数据还是返回缓存数据时非常有用。

### 确实非常小的库

**redux-thunk**库的所有代码如下：
```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    // 触发每个action这里都会被调用
    // 如果是函数就调用它
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }
    
    // 否则继续按正常处理
    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```
如果项目中安装了redux-thunk，并且把它当成中间件，触发的每个action都会进入这部分代码。如果action是函数就调用那个函数（不管函数返回什么），否则将它传递给下一个中间件，最终会交给Redux（`netx(action)`完成这部分工作）。

### 项目中使用redux-thunk

如果项目中已经安装了Redux，只需两步就可添加redux-thunk。
首先，安装包：
```bash
npm install --save redux-thunk
```
然后在使用redux的代码部分，引入`redux-thunk`中间件，添加到Redux中。
```js
// 你可以已经引入了createStore，还需要引入applyMiddleware
import { createStore, applyMiddleware } from 'redux';
// 引入`thunk` 中间件
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';
// 使用applyMiddleware创建thunk中间件
const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```
确保`thunk`作为`applyMiddleware`参数调用，否则不起作用。这样你已经完成了所有设置，现在可以dispatch任何函数。