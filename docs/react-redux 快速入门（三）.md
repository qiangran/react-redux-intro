##react-redux--异步Action
上两篇文章叙述的都是同步操作，每当 dispatch action 时，state 会被立即更新。但是实际应用中，我们有很多操作执行后，过一段时间，才会得到结果。那么怎么处理这种情况呢？

先熟悉一个概念
中间件
-----------------
本质就是一个通用函数，可以在程序的任何执行处介入，从而将处理方法扩展到现有系统上。

```
在store.dispatch(action) 执行时候，打印日志，这就是一个简单的中间件
let next = store.dispatch;
store.dispatch = action => {
  next(action);
  console.log('state after dispatch', getState())
}
```

引入中间件 redux-thunk
-----------------
redux-thunk中间件,允许action creator返回一个函数，这个函数会接收dispath和getState？为参数。
 
```
看下面三种不同的action
const INCREMENT_COUNTER = 'INCREMENT_COUNTER';

普通的,返回一个对象
function increment() {
  return {
    type: INCREMENT_COUNTER
  };
}

异步的，返回一个函数
function incrementAsync() {
  return dispatch => {
    setTimeout(() => {
      // Yay! Can invoke sync or async actions with `dispatch`
      dispatch(increment());
    }, 1000);
  };
}

带条件的，返回一个函数
function incrementIfOdd() {
  return (dispatch, getState) => {
    const { counter } = getState();

    if (counter % 2 === 0) {
      return;
    }

    dispatch(increment());
  };
}

```
使用中间件 redux-thunk
-----------------
需求：
进入页面之后，点击某个按钮，获取用户投资总额。。。

分析：
异步请求有3个关键action

* 开始请求--一般用于呈现加载进度条
* 请求成功--关闭进度条，加载数据
* 请求失败--关闭进度条，展示错误信息


```
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk'; //导入thunk
import rootReducer from './reducers/index';

// Note: this API requires redux@>=3.1.0
const store = createStore(
  rootReducer,
  applyMiddleware(thunk)//激活thunk中间件
);

//一个异步请求
function fetchAmount() {
  return fetch('https://www.renrendai.com/getAmount?userId=123')
}
//通知 reducer 请求开始的 action
requestPost(){
	return{
		type: REQUEST_POSTS,
		isFetch:true //进度条相关
	}
}

//通知 reducer 请求成功的 action
receviePostOnSuccess( data){
	return{
		 type: RECEIVE_POSTS,
		 isFetch:false,
		 amount: data.amount
	 }
}
//通知 reducer 请求失败的 action。
receviePostOnError( message){
  return{
	 type: RECEIVE_POSTS,
	 isFetch:false,
	 errorMsg:message
  }
}


//异步请求action 【将上面3个基础的action整合】
function getAmount(){
	retrun (dispath)=>{
	// 首次 dispatch：更新应用的 state 来通知API 请求发起了
    dispatch(requestPosts())
    //异步请求后端接口
		return fetchAmount().then(
			 data=>dispath(receviePostOnSuccess(data)),
			 error=> dispatch(receviePostOnError('error))
		)
	}
}

```
也很容易理解吧~~~

本文档是将最基础的使用方法提炼呈现，真正的react-redux博大精深，如果想继续深究，就赶紧找一个简单的项目开始练手吧。

参考文档：

[Redux 中文文档](https://www.redux.org.cn/)


[Redux](https://redux.js.org/introduction/getting-started)

[react-redux](https://react-redux.js.org/introduction/quick-start)

[Redux thunk](https://github.com/reduxjs/redux-thunk)

[Redux异步操作redux-thunk](https://blog.csdn.net/gao_xu_520/article/details/80527421)
