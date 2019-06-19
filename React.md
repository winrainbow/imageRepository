### React

1. create-react-app
2. 组件化思想
3. JSX
4. 开发调试工具
5. 虚拟DOM
6. 生命周期
7. React-transition-group
8. Redux
9. Antd
10. UI,容器组件
11. 无状态组件
12. redux-thunk
13. redux-saga
14. Styled-components
15. Immutable.js (react推出的，避免数据误操作)
16. redux-immutable
17. axios

#### React简介

react.js灵活性更大，vue.js更丰富的api

#### React开发环境：

1. 引用.js文件来使用React
2. 使用脚手架工具（自动构建开发流程和目录，实现js文件相互引用）
   1. grunt、webpack
3. Create-react-app 官方提供的脚手架工具，推荐使用


* 安装 Create-reat-app

  npm install -g create-react-app

* 安装node 和 npm


#### public 目录下

favicon.icon 左上脚图标

index.html是一个模板

manifest.json文件？ PWA 本地缓存，快捷方式，icons,

#### src目录

index.js是程序的入口文件  ReactDom.render(<App/>,document.getelementById('root'))

index.css文件 ： // all in js  style

App： import App from './App' // 代表 App.js

App.js:是一个组件  render方法中，返回一个html标签

#### JSX语法

```jsx
class App extends Component{
    render(){
        return (
            <div>  // 不仅可以使用h5标签，还可以使用自己定义的组件
                Hello world
            </div>
        )
    }
}
```



#### PropTypes DefaultProps

#### 虚拟DOM中的diff算法

两个JS对象的差异。

当render()，或者说调用setState()方法时执行时，执行diff算法

setState()异步，会提升性能：频繁调用会合并

diff原理：

1. 同层对比，如同则下一层，否则全替换该层
2. 当DOM节点比对时，如果没有key ,无法快速对比，所以在 循环时加上key; index 不能保证原始的和新的一致，因为插入顺序会变

#### Ref: 获取dom标签，与setState共同使用时，有坑，因setState是异步，可以把执行放到setState（f,f2）第二个方法中

this.setState((prevState)=>({a:a,b:b}),()=>{ /*想要执行的内容*/})

不推荐使用，有些场景会有问题 

特别是异步时

#### React生命周期

生命周期函数：某一时期自动执行的函数

1. constructor(props) 也是一个生命周期方法，但不是React独有，是ES6语法中都有的

2. initialzation: constructor方法

3. Mounting: componentWillMount->render()->componentDidMount()   1和3 只在第一次被执行

4. Updation 

   1. Props:

   * componentWillReceiveProps、
   * shouldComponentUpdate: 组件将要变更时，执行 return  true:更新 false:不更新（render()）
   * componentWillUpdate
   * render()
   * componentDidUpdate  请求数据
   2. states:

      * shouldComponentUpdate
      * componentWillUpdate
      * render()
      * componentDidUpdate

5. ComponentWillUnMount
####性能优化

shouldComponentUpdate

axios:

#### CSS过渡动画

#### react-transition-group

#### Redux ：Reducer+Flux 数据层的框架

将数据放入公用的存储域  store

####Redux 工作流程

ActionCreators、store、Reducers、ReactComponents

store中管理数据，优先编写store

reducer 返回一个函数 入参：state,action  返回值：state

store 接收到数据，会把state和action 给reducer

actionTypes拆分：

store下面创建actiontypes.js

使用actionCreator 统一创建action

store必须是唯一的

只有store可以改变自己的内容：看似reducer在改变store内容？其实只是把新的数据给了store

reducer 必须是个纯函数：给定输入肯定是固定的输出，不会有副作用



#### UI组件 容器组件

ui组件：傻瓜组件

容器组件：smartcomponent

#### 无状态组件，性能高

```
const TODOLISTUI = (props)=>{
    return (
    	render（）中的return
    )
}
```



#### Redux异步请求数据

在then()方法中，创建action,并dispatch,然后store.subscribe后，开始更新数据

#### Redux-thunk 中间件

中间件原理：

```
axios.get('/list.json').then((res)=>{
    const data = res.data;
    const action = initListAction(data);
    store.dispatch(action)
})
redux-thunk可以让上面的复杂代码，在action中执行

之前：actionCreator 返回的是一个对象
但使用thunk后，可返回一个函数
export const getTodoList=()=>{
    return (dispatch)=>{
        //作异步
        const action = initListAction(data);
        dispatch(action)
    }
}
返回的函数怎么用？
const action = getTodoList();
此时action是一个函数
store.dispatch(action)
当调用 dispatch时，action方法被执行
store发现action 是方法，则会执行该方法，并把dispatch方法传进去。


```

#### Redux中间件是什么？

action与store的中间

之前，action是object 直接给store

中间件：action是方法，执行后获得object后，给store

thunk 中间件，就是对dispatch方法的封装

中间件很多：logger、saga:解决异步，异步拆分出放到另一个文件

#### Redux-saga中间件

异步处理，放在一个单独的文件中

```
import {takeEvery，put} from 'redux-saga'
/**
Generator函数
*/
function* mySaga(){
    yield takeEvety("ACTION_TYPE",异步方法)
}

// mySaga接收到action后，执行异步方法
```

####React-Redux

#### combineReducers 来完成对数据的拆分

为了让reducer的代码不会变的太大，需要拆分

reducer是管理员小手册，小手册很大时，查找很慢，这时需要分类成多个小手册

combineReducers将reducer拆分成多个小的reducer

组件的展示和数据放在同一个目录下，利于管理和问题 定位

#### 使用Immutable.js来管理store中的数据

```javascript
import * as constant from './constant'
import {fromJS} from 'immutable'
const defaultState = fromJS({focused: false})
/**
 * reducer 不能对 原state 处理，只能返回一个新的state
 * immutable.js 用来帮助不修改 原state
 * immutable不可变更 ，需要将state变与immutable对象
 * immutable对象set方法，会结合之前的值和现在的值，生成一个全新的对象，不会修改原来的数据
 * @param state
 * @param action
 * @returns {{focused: boolean}}
 */
export default (state = defaultState, action) => {
    if (action.type === constant.SEARCH_FOCUS) {
        return state.set('focused',true)
    }
    if (action.type === constant.SEARCH_BLUR) {
        return state.set('focused',false)
    }
    return state
}
```

#### redux-immutable

import {combineReducer} from 'redux-immutable'

#### 路由：

根据url不同，展示不同的内容

#### overflow: hidden?? display:block?

#### 异步代码拆分优化



#### window事件

1. 在componentDidMount()方法中，window.addEventListener('scroll',方法)
2. 在componentWillUnmount()方法中，window.removeEventListener('scroll',方法)

##### 调优

1. shouldComponentUpdate() : pureComponent 实现了 shouldComponentUpdata()方法
2. pureComponent 和 immutable.js结合使用，否则有坑
3. 路由，只加载一次，使用link 代替a 标签，href 换成 True

       ```
<Link to = '/'/>
<Logo/>
</Link>
// link 需要 在router内部
第一种获取参数的方式：
<route path='/detail/:id'
this.props.match.params.id

第二种方式：
？id=
<route path='/detail'
this.props.location.search  需要解析search

       ```

#### 异步组件,进入页面才加载

* react-loadable:

  1. yarn add react-loadable

  2. 对应文件夹下创建loadable.js

     ```javascript
     import Loadable from 'react-loadable'
     import React from 'react'
     const LoadableComponent = Loadable({
         loader:()=>import('./'),
         loading:()=>{
         	return (<div>正在加载</div>)
     	}
     })
     export default ()=><LoadableComponent/>
     2,路由地方加载loadable.js
     3,获取传入的参数，需要如下处理：
     	import {withRoute} from 'react-route-dom'
     export default connnect(mapState,mapDispatch)(withRouter(Detail))
     ```

#### 项目上线流程

1. 前后端定好接口
2. xampp 可以启个服务器 
3. 控制台:npm run build  后，会多一个build文件夹
4. build文件给后端，后端将build文件夹下的内容复制放到 htdocs目录下
5. ​

























