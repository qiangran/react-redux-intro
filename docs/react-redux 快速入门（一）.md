##react-redux--redux浅识

看完reudx官网文章之后，有一种“憋闷”的感觉，这种感觉来自“我已经反复阅读了相关文档，但是实践到具体项目的时候，还是不会”，本系列文章旨在扫除这种障碍，能够让你简单上手使用。
在这里，我们假设你已经有丰富的react开发经验。如果没有,建议先阅读官方文档[ react documentation](https://reactjs.org/docs/getting-started.html)。

下面进入正题
## reudx 是什么？ 
  * 一种架构模式；
  * 一个可预测化状态管理容器；

**（说完跟没有说似得，一步一步来）**
## redux 有哪些东西?
  * action 
  * reducer 
  * store  
    
**（上面只是简单概括，咱们先入门，再深究）**

###  action 是什么？
一个普通的javascript对象，定义state的操作类型和值。

```
{
  type: ‘描述操作的类型，其实就是简单描述这个state是干什么的，约定是大写的字符串常量，必须存在’,
  value1:’任意值，根据需求，可以定义多个’，
  value2:’任意值，根据需求，可以定义多个’，
}
```
**（看，没有任何复杂的东西，请务必记住action的结构，一般通过action创建函数来创建action,具体后面会聊）**
###  reducer 是什么？
一个纯函数,接收旧的 state 和 action，返回新的 state。


```
(previousState, action) => newState

previousState:旧的state
action:上面定义的action
newState:新的state

```
注意：
官方文档特别强调，要保持 reducer 纯净。永远不要在 reducer 里做这些操作：

* 修改传入参数；
* 执行有副作用的操作，如 API 请求和路由跳转；
* 调用非纯函数，如 Date.now() 或 Math.random()。

总之，只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。

```
function todoApp(state, action) {
  switch (action.type) {
    case ‘ADD_TODO':
      return Object.assign({}, state, {  //不要修改 state。 使用 Object.assign() 新建了一个副本
        text: action.text
      })
    default:
      return state //在 default 情况下返回旧的 state。遇到未知的 action 时，一定要返回旧的 state。
  }
}

```

###  store 是什么？
它由Redux提供的 createStore(reducer， defaultState？) 方法生成，拥有以下方法

```
getState() 获取全局 state，类似一个类中的get方法
dispatch(action) 更新指定的 state，当执行该方法时候，redux会自动调用
reducer函数，这是唯一能改变store中数据的方式，
subscribe(listener) 注册监听器，store发生变化的时候被调用
subscribe(listener) 返回的函数注销监听器

```
到这里，我们知道：

store集中存储了应用中所有的state（一个大大的对象树），这些state的修改的快照，通
过有规范的action对象来描述。然后编写专门的函数来决定每个 action 如何改变应用的 state，这个函数被叫做 reducer。

### 如何让action ,reducer,store“三驾车”启动起来
直接上图

![image](https://github.com/qiangran/react-redux-intro/blob/master/docs/images/01.jpg?raw=true)

当调用store.dispath(action)方法时，redux会主动触发对应的reducer执行，从而得到新的state,state发生变化，store.subscribe()又会被调用，于是我们可以在这个方法中做UI重绘，或者通过store.getState()获取全新的state，以便后续使用。
就是这样！

结合官网的例子

```
import { createStore } from 'redux';

/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成一个新的 state。
 *
 * state 的形式取决于你，可以是基本类型、数组、对象、惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 *
 * 下面例子使用 `switch` 语句和字符串来做判断，但你可以写帮助类(helper)
 * 根据不同的约定（如方法映射）来判断，只要适用你的项目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// 创建 Redux store 来存放应用的状态。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() =>
  console.log(store.getState())
);

// 改变内部 state 惟一方法是 dispatch 触发一个 action，这里自动调用reducer函数
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

怎么样，到这里是不是对redux的使用有了基本的了解，那么下面看看它是怎么和react默契配合，相得益彰的吧

####下一章 react-redux--搭配react







