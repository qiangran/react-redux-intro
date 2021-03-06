##react-redux--搭配react

先熟悉两个方法(大概浏览下即可，根据后面的例子对照学习)
bindActionCreator(actionCreators, dispatch)
--------------------------------
使用dispatch将一个action creator（action创建函数,前面有提到），或者一个 value 是 action creator 的对象进行包装，以便可以在react组件里面直接调用他们。
#### **入参**
1. actionCreators :Function | Object
2. dispatch: Function。

```
第一个入参是action creator函数
const toggleTodo = (id) => {
    return {
        type: 'TOGGLE_TODO',
        id
    };
};

let boundActionCreators = bindActionCreators(toggleTodo, dispatch);
//此时boundActionCreators  = (id) => dispatch(toggleTodo(id));

第一个入参是value为action action对象
const removeTodo = {
    removeTodo : id => {
      type: 'REMOVE_TODO',
      id
    }
};
let boundActionCreators = bindActionCreators(removeTodo, dispatch);
//此时boundActionCreators  = (id) => dispatch(removeTodo(id));
```
所以bindActionCreator实现思路就是

* 判断传入第一个的参数是否是object，如果是函数，就直接返回一个包裹dispatch的函数
* 如果是object，就根据相应的key，生成包裹dispatch的函数即可

（随口一问：为什么要用dispath包裹呢 ？想想上一篇的内容吧~~）

connect(...)
--------------------------------
* 将Redux store和React组件联系在一起
* connenct并不会改变它“连接”的组件，而是提供一个经过包裹的connect组件。

```
connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)

```
#### **入参**

接受4个不同的，可选择的参数。按照惯例，他们被称为：

1. mapStateToProps?: Function
2. mapDispatchToProps?: Function | Object
3. mergeProps?: Function
4. options?: Object

##### mapStateToProps?: (state, ownProps?) => Object
* 用于建立组件跟store的state的映射关系
* 作为一个函数，它可以传入两个参数(state, ownProps？)，结果一定要返回一个object
* 传入mapStateToProps之后，会订阅store的状态改变，在每次store的state发生变化的时候，都会被调用
* ownProps代表组件本身的props，如果写了第二个参数ownProps，那么当prop发生变化的时候，mapStateToProps也会被调用。例如，当 props接收到来自父组件一个小小的改动，那么你所使用的 ownProps 参数，mapStateToProps 都会被重新计算）。
* 如果不想订阅store的更新，在connect()中，可以直接将mapStateToProps方法用null 或者undefined代替

```
一个参数
const mapStateToProps = state => ({ todos: state.todos })

两个参数
const mapStateToProps = (state, ownProps) => ({
  todo: state.todos[ownProps.id]
})

```
##### mapDispatchToProps?: Object | (dispatch, ownProps?) => Object
* mapDispatchToProps用于建立组件跟store.dispatch的映射关系
* 它可以是一个对象或者是带有两个参数(dispatch, ownProps?)的函数 ，结果一定要返回一个object

* 如果mapDispatchToProps是一个函数,调用时最多使用两个参数，如下

```
一个参数
const mapDispatchToProps = dispatch => {
  return {
    // dispatching plain actions
    increment: () => dispatch({ type: 'INCREMENT' }),
    decrement: () => dispatch({ type: 'DECREMENT' }),
    reset: () => dispatch({ type: 'RESET' })
  }
}
两个参数
// binds on component re-rendering
;<button onClick={() => this.props.toggleTodo(this.props.todoId)} />

// binds on `props` change
const mapDispatchToProps = (dispatch, ownProps) => {
  toggleTodo: () => dispatch(toggleTodo(ownProps.todoId))
}

```
* 如果mapDispatchToProps是一个object，其中这个object所对应的value必须是actionCreator，这样redux里面会自动帮我们调用bindActionCreator

```
const mapDispatchToProps = {
    ...action
}

```
* 如果想使用默认的dispath方法，在connect()中，可以将mapDispatchToProps方法用null代替

```
// do not pass `mapDispatchToProps`
connect()(MyComponent)
connect(mapState)(MyComponent)
connect(
  mapState,
  null,
  mergeProps,
  options
)(MyComponent)
```

##### mergeProps?: (stateProps, dispatchProps, ownProps) => Object
* 将mapStateToProps(), mapDispatchToProps()以及组件自身的props合并
* 暂时忽略，期待后续

##### options?: Object
* 暂时忽略，期待后续


废话不说，上图

![image](https://github.com/qiangran/react-redux-intro/blob/master/docs/images/02.jpg?raw=true)


这是一个很常见的“输入框校验”需求，根据用户输入的数字，判断是否符合要求(输入必须为1000的整数倍)，给予相应的文字提示。

下面我们就使用react-redux来处理这个需求吧~~

* 动手写代码之前，先考虑下有哪些state 

```
	“错误信息提示” -errorMsg
```
* 各个state有哪些操作可以修改

```
  	既是考虑下action的type的取值
```

***


```
action.js
--------------------------
showMutipleMsg(errorMsg){
	type:"MUTIPLE_ERROR_MESSAGE",
	errorMsg:"输入金额需为1，000的整数"
}
showMinMsg(errorMsg){
	type:"MIN_ERROR_MESSAGE",
 	errorMsg:"输入金额需不能小于2，000元"
}
 
--------------------------
//上面两个action都是对一个state的描述，操作类型相似，我们可以通过“action创建函数”改写：
changeErrorMsg(errorMsg){
	return {
		type:"CHANGE_ERROR_MESSAGE",
		errorMsg
	}
}
//action创建函数，其实就是生成 action 的方法,这样做将使 action 更容易被移植和测试。


```
```
 触发函数
 reducer.js
 changeInputErrorMsg(state,action){
 	switch (action) {
	    case 'CHANGE_ERROR_MESSAGE':
	      return Object.assign({}, state, {
	        errorMsg: action.errorMsg
	      })
	         
    default:
    return state
  }
 }
 
```
 
```
  被触发的组件
  InputError.js
  class InputError extends Component {
  
    static propTypes = {
        errorMsg: PropTypes.string
    }
    
    render () {
        return (
	        <div>
	           {this.props.errorMsg } 
	        </div>
        )
    }
}

const mapStateToProps = (state) => {
    return {
      errorMsg: state.errorMsg
    }
}

InputError = connect(mapStateToProps)(InputError)

export default InputError
 
```
 
```
 组件中的触发
 Input.js
 class Input extends Component {
  
    static propTypes = {
      onSwitchErrorMessage: PropTypes.func
    }
    
    handleSwitchError () {
    	 //各种计算校验，这里省略，如果不符合规范，dispatch action 去显示错误信息
        if (this.props. onSwitchErrorMessage) {
            this.props. onSwitchErrorMessage(errorMsg)
        }
    }
    
    render () {
        return (
        <div>
        	<input>
           <InputError/>
        </div>
        )
    }
}

 

const mapDispatchToProps = (dispatch) => {
    return {
        onSwitchErrorMessage: (errorMsg) => {
            dispatch(changeErrorMsg(errorMsg))
        }
    }
}

Input = connect(null, mapDispatchToProps)(Input)
export default Input
 
```
 
```
 入口文件
import { createStore } from 'redux'
import { Provider } from 'react-redux'

import Reducer from './components/reducer'  // 函数
const store = createStore(Reducer);

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>, 
document.getElementById('root'));
 
```

 下一章 react-redux--异步Action

 

 



