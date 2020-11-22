#   react基础教程之hook，redux（vue&react本就可学其一会其一）

写在前面：本来是想写一篇从vue快速转到react的基础教程的，但是最近事情确实蛮烦人。每天断断续续的写也实在没心情写下去了

目前完成了 ：常见hooks和redux的使用

未完成：router，及开始的一些最基础的玩意

最基础的那部分也可以直接看react的文档吧，router等状态好点一定会补上的

## 一、基础知识

1.1 react和react-dom&jsx

1.2 class组件与function组件

1.3 setState的异步渲染

1.4 条件渲染与列表

1.5 生命周期

1.6 双向数据绑定

1.7 组件复合（相当于vue的插槽）

1.8 高级组件

### 1.9 组件通信上下文

这个东西就类似于vue的组件隔代传参 provide、inject

![1602575397792](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1602575397792.png)

直接看使用吧，除了在使用上稍微不同。

```jsx
import React, { Component } from 'react';

const context = React.createContext()
const { Provider, Consumer } = context


// 给一个数据
const store = {
    name: 'gxb',
    age: 18
}


// 爷爷组件
class Context extends Component {

    render() {
        return (
            <Provider value={store}>
                <ToolBar />
            </Provider>
        );
    }
}




// 儿子组件
function ToolBar(props) {
    return <Info />
}




// 孙子组件
class Info extends Component {

    render() {
        return (
            <Consumer>
                {store => {
                    return (
                        <div>
                            {store.name}
                            {store.age}
                        </div>
                    )
                }}
            </Consumer>
        );
    }
}


export default Context;
```

2.0（优化）PureComponent与React.memo  

## 二、常见Hook

### 2.1 useState  

**基本使用**

useState  的出现使得函数组件可以拥有自己的状态，它的基本使用还是十分简单的。

像这样

```js
 const [conut,setCount]=useState(0)
```

它接收一个初始状态值（可以是简单类型，也可是复杂类型，也可是一个具有返回值的函数），返回一个数组。数组有两项 1.你传入的状态数据 2. 修改状态数据的方法。故接收时你就可使用数组结构自定义命名解构出来了

一个完整的小栗子

```js
import React, { useState } from 'react';
const Hooks01 = () => {
    const [conut, setCount] = useState(0)
    return (
        <div>
            {conut}
            <button onClick={() => { setCount(conut + 1) }}>add</button>
        </div>
    );
}
export default Hooks01;
```

**与setState区别**

但是，区别于setState。如果参数是对象的话，你不能像setState那样使用。这里的useState是不会帮你进行合并的

栗子：

```js
![宫小白react02](C:\Users\T540P\Desktop\宫小白react02.gif)import React, { useState } from 'react';
const Hooks01 = () => {
    const [msg, setMsg] = useState({
        name:'gxb',
        age:18
    })
    return (
        <div>
            {msg.name}{msg.age}
            <button onClick={() => { setMsg({name:'zs'}) }}>add</button>
        </div>
    );
}
export default Hooks01;
```

![宫小白react02](C:\Users\T540P\Desktop\宫小白react02.gif)

这个时候你只能自己合并的，像这样

```jsx
 <button onClick={() => { setMsg({...msg,name:'zs'}) }}>add</button>
```

为啥要这么设计呢？

嗯...其实自己现在也还是蛮迷惑的。首先对于组件比较轻小使用它非常合适，再大一点就要用到“它的老爹”useReducer （毕竟useState是useReducer 的语法糖 ）了。但是这俩真的是各有各的恶心...useReducer可以做对局部更新状态不用全部重写，但是书写的复杂度又上来了啊。额...如果真的遇到稍微复杂的状态还是根据情况选择吧，useReducer不是唯一别问了我们还可以换会class组件

**异步更新**

和setState一样，为了保证性能它也是异步更新的

```js
import React, { useState } from 'react';
const Hooks01 = () => {
    const [count, setCount] = useState(0)
    const setSta = () => {
        setCount(count+1)
        console.log(count)
    }
    return (
        <div>
            {count}
            <button onClick={()=>{setSta()}}>add</button>
        </div>
    );
}
export default Hooks01;
```

![宫小白react03](C:\Users\T540P\Desktop\宫小白react03.gif)

想要达到同步的效果在函数式组件中还是比较简单的

方法一： 我们知道它set完了肯定会从新render吧，那么就这样写

```js
import React, { useState } from 'react';
const Hooks01 = () => {
    const [count, setCount] = useState(0)
    const setSta = () => {
        setCount(count+1)
    }
    console.log(count)
    return (
        <div>
            {count}
            <button onClick={()=>{setSta()}}>add</button>
        </div>
    );
}
export default Hooks01;
```

方法二：我们给setCount传值的时候传一个函数，那么我们就可以在那个函数中进行我们的一些额外操作啦

像这样

```js
import React, { useState } from 'react';
const Hooks01 = () => {
    const [count, setCount] = useState(0)
    const setSta = () => {
        setCount((count)=>{
            count++
            console.log(count)
            return count
        })
    }
    return (
        <div>
            {count}
            <button onClick={()=>{setSta()}}>add</button>
        </div>
    );
}
export default Hooks01;
```

在这里说一个额为的话题，读源码掌握原理的重要性。如果你不知道上面的这种用法，但是你了解useState的基本原理（这里还是蛮简单的，我贴下别人写过的最简形式）

```js
let val; 
function useState(initVal) {
    val = val|| initVal; 
    function setVal(newVal) {
        val = newVal;
        render(); 
    }
    return [val, setVal];
}
```

你就会了解到，在这里改变状态的函数为useState仅仅是一个新值，也就是说我们完全可以在这个改变状态的函数中传一个函数参数，只要我们别忘了给它返回去一个新值，我们在这个函数里面完全可以干我们自己的事情

这仅是一个最简单的栗子，读源码是为了可以更深入的理解某个东西，投入的时间和所获得的东西是完全可以成正比的。

好了，跑题了...。上面说过useState因为不会自动合并，所以状态比较复杂使用它是非常不友好的（总得复制）。故我们开始介绍下一个“它的老爹”----useReducer  

### 2.2 useReducer  

useReducer 就是借鉴了redux的思想，如果你还不了解redux是咋使用的。那么还是先移步下面的redux，读完它再来看这。

```js
 const [state,dispatch]=useReducer(reducer,initState)
```

它的使用和redux的思想一毛一样。useReducer接收两个参数，1 reducer，用来重新计算state的； 2 initState，初始值。还有第三个参数  initAction，它是useReducer初次执⾏行行时被处理理的action  用的比较少

返回一个state和dispatch。也即改变状态也是需要使用dispatch去发出一个action。reducer接收之后根据action的描述执行相应逻辑进行state的改变

因为和redux基本使用一样，我就写一个简单的栗子好了

```js
import React, { useReducer } from 'react';

const Hook03 = () => {

    const [state, dispatch] = useReducer(reducer, 0)
    function reducer(state, actions) {
        switch (actions.type) {
            case 'add':
                return state+actions.step
            case 'reduce':
                return state-1
            default:
                return state
        }
    }

    return (
        <div>
            {state}
            <button onClick={()=>{dispatch({type:'add',step:2})}}>加2</button>
            <button onClick={()=>{dispatch({type:'reduce'})}}>减一</button>
        </div>
    );
}

export default Hook03;
```

### 2.3 useEffect

函数组件相比于class组件的两个最大的不足一是没有自己状态二是没有生命周期

上面解决了状态的问题，下面就解决生命周期的问题吧

引入了Hooks之后，虽然还是没有生命周期的东西。但是有了一个useEffect钩子进行代替

useEffect可以看成是componentDidMount、componentDidUpdate、componentWillUnmount这⼏个生命周期方法的合并。虽然没有class组件那样精确，不过一般的副作用操作有了可以放置的地方

用法`   useEffect(callback,array) `

来一个简单的栗子,模拟是从后台获取数据

```jsx
import React, { useEffect, useState } from 'react';

const Hooks04 = () => {
    const [count, setCount] = useState(0)
    useEffect(() => {
        setTimeout(() => {
            setCount(1)
        }, 1000)
    })
    return (
        <div>
            {count}
        </div>
    );
}
export default Hooks04;
```

**参数array的三种情况**

很明显上面的栗子中，我没有传入第二个参数。下面来详细说下区别吧

- 不填，每次重新渲染时这个回调都会重新执行
- 填一个空数组，回调只会在初次渲染时调用
- 非空，里面就是依赖的状态了,状态变动回调重新执行（类似于vue的watcher吧，它首次渲染也是会执行一遍回调的）

栗子

```jsx
import React, { useEffect, useState } from 'react';

const Hooks04 = () => {
    const [count, setCount] = useState(0)
    const [state, setState] = useState(1)
    useEffect(() => {
        console.log(111);
    }, [state])
    return (
        <div>
            {count}
            <button onClick={() => { setCount(count + 1) }}>改变count</button>
            <button onClick={() => { setState(state + 1) }}>改变依赖</button>
        </div>
    );
}
export default Hooks04;

```

![宫小白react04](C:\Users\T540P\Desktop\宫小白react04.gif)

**释放资源**

class组件可在组件卸载的生命周期中执行这些资源释放的动作（如移出定时器等）

函数组件怎么做呢？其实也是蛮简单的。  在useEffect的第一参回调中，使它返回一个函数，将释放资源的操作放入这个函数中即可。也即这个函数的调用时机是在组件卸载时

```js
useEffect(() => {
    副作用...
    return () => {
        释放资源...
    }
}, [])
```

### 2.4 useContext 

可以认为是上面的通信通信上下文的进一步的改进吧，主要应用也是隔代通信（上到下）

用法：`  const ctx =useContext(Context)  `

这个context参数是 由上层组件中距离当前组件最近的 `组件.Provide` 的 `value` prop 决定的。

不看栗子理解，光说还是比较苍白的。来(和上面的context做下对比)

```jsx
import React, { useContext } from 'react';
const Context = React.createContext()

// 爷爷组件
const Hooks05 = () => {

    return (
        <div>
            爷爷组件
            <Context.Provider value={{ name: 'gxb', age: 18 }}>
                <Fa></Fa>
            </Context.Provider>
        </div >
    );
}

// 父亲组件
const Fa = () => {
    return (
        <div>
            父亲组件
            <Son></Son>
        </div>
    )
}

// 孙子组件

const Son = () => {
    const ctx = useContext(Context)
    return (
        <div>
            孙子组件
            {ctx.name}
        </div>
    )
}



export default Hooks05;
```

其实这时候你就可以发现，这里与上下的context写法的区别主要在于接收数据源的组件上

context的写法是再提供一个Consumer组件，在里面再搞一个函数，还是有点复杂的

而这里，直接使用useContext就可以拿到上面传下了的context。且提供方组件只要更新，该hook便会重新渲染即也可拿到改变的最新值（它的重新渲染即使你使用`React.memo`也不好使）

```jsx
import React, { useContext,useState } from 'react';
const Context = React.createContext()

// 爷爷组件
const Hooks05 = () => {
    const [count,setCount]=useState(0)
    return (
        <div>
            爷爷组件
            <button onClick={()=>{setCount(count+1)}}>按钮</button>
            <Context.Provider value={{ name: 'gxb', age: 18 }}>
                <Fa></Fa>
            </Context.Provider>
        </div >
    );
}

// 父亲组件
const Fa = () => {
    return (
        
        <div>
            父亲组件
            <Son></Son>
        </div>
    )
}

// 孙子组件

const Son = () => {
    const ctx = useContext(Context)
    console.log(1);
    return (
        <div>
            孙子组件
            {ctx.name}
        </div>
    )
}



export default Hooks05;
```

![宫小白react05](C:\Users\T540P\Desktop\宫小白react05.gif)

### 2.5  useMemo & useCallback  

useMemo和的useCallback这两个hook是比较类似的，它们都是与性能优化相关的

 先上个栗子，看这种情况

```jsx
import React, { useState } from 'react';
const Hooks06 = () => {
    const [count01, setCount01] = useState(0)
    const [count02, setCount02] = useState(0)
    return (
        <div>
            父组件：{count01}
            <button onClick={() => { setCount01(count01 + 1) }}>父组件count+1</button>
            <Son count02={count02}></Son>
        </div>
    );
}

// 子组件

function Son({ count02 }) {
    console.log(111);
    return (
        <div>
            子组件：{count02}
        </div>
    )
}





export default Hooks06;
```

![宫小白react06](C:\Users\T540P\Desktop\宫小白react06.gif)

父组件的状态发生改变，但是其传给子组件的数据始终是没有发生变化的。但是子组件每次仍会重新渲染。这并不是我们想要的吧

我们想要的和上面PureComponent一样，只有往下传的数据发生了变化，子组件才会重新渲染否则不渲染

使用useMemo实现一下

useMemo接收两个参数，一个回调。一个依赖项

回调的返回值就是你要传给下面组件的数据。

子组件也要稍微改变一下，再从react里解构出memo。一个HOC可将子组件进行包装改造



```jsx
import React, { useState, useMemo, memo } from 'react';
// 子组件

function Son({ count02 }) {
    console.log(111);
    return (
        <div>
            子组件：{count02}
        </div>
    )
}
Son = memo(Son)




const Hooks06 = () => {
    const [count01, setCount01] = useState(0)
    const [count02, setCount02] = useState(0)
    let count03 = useMemo(() => count02, [count02])
    return (
        <div>
            父组件：{count01}
            <button onClick={() => { setCount01(count01 + 1) }}>父组件count+1</button>
            <button onClick={() => { setCount02(count02 + 1) }}>子组件count+1</button>
            <Son count02={count03}></Son>
        </div>
    );
}



export default Hooks06;
```



![宫小白react07](C:\Users\T540P\Desktop\宫小白react07.gif)

**useCallback**  的用法与其类似，区别在于useCallback往下传的是一个方法

```js
const memoCallback= useCallback(callback,array)
```

useMemo中拿的是callback的返回值，而useCallback拿的是整个callback

## 三、router和redux的使用



**路由还有没写等待日后更新**

### 3.1 react-router

3.1.1 基本使用

3.1.2 动态路由与路由传参

3.1.3 路由守卫

### 3.2 redux

#### 3.2.1 核心概念

**redux三大重要组成**

![1602472173023](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1602472173023.png)

**store提供给view层的三个常用方法**

![1602471807913](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1602471807913.png)

注：注意state和store的关系，state仅是一个快照也即整个仓库的状态可能会随着你的操作或其他原因是不断发生变化的state仅是某一时刻的状态

**redux工作流程**

![1602471336337](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1602471336337.png)

#### 3.2.2 基本使用

我开始接触的时候，总是把这里的action和vuex的action做上联系导致开始迷迷糊糊的，注意一下这里的action仅是一个对象

来写一个简单的栗子

利用redux里提供的createStore函数，这个函数便是用来生成一个store的。

它有三个参数

1.reducer 2. 初始状态 3.applyMiddleware函数（用于组合中间件，下面再使用）

先搞一个最简单的

**store中**

```js
import { createStore } from 'redux'

const addReducer = (state = 0, action) => {
    switch (action.type) {
        case 'add':
            return state + 1
        case 'reduce':
            return state - 1
        default:
            return state
    }
}
const store = createStore(addReducer)
export default store
```

**组件中**

```jsx
import React, { Component } from 'react';
import store from './store/index'
class StoreShow extends Component {
    render() {
        return (
            <div>
                {store.getState()}
                <button onClick={() => store.dispatch({ type: 'add' })}>+</button>
                <button onClick=
                    {() => store.dispatch({ type: 'reduce' })}>-</button>
            </div>
        );
    }
}

export default StoreShow;
```

**入口文件中**

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './app'
import store from './store'

const render = () => {
  ReactDOM.render(
    <App />
    ,
    document.getElementById('root')
  );
}

render()
store.subscribe(render)
```

这样一个简单的加减器就出来了

![宫小白react01](C:\Users\T540P\Desktop\宫小白react01.gif)

#### 3.2.3 处理异步

**仅使用redux的情况**

我们模拟一个从后台所取的数据（用setTimeout模拟）

流程很简单，就是把数据取过来之后再调用dispatch把数据传到reduce

```jsx
import React, { Component } from 'react';
import store from './store/index'
class StoreShow extends Component {
    getStep = () => {
        return 2
    }
    asyncGetStep = () => {
        let that = this
        new Promise((res, rej) => {
            setTimeout(() => {
                res(that.getStep())
            },1000)
        }).then((res) => { store.dispatch({ type: 'add', step: res }) })
    }
    render() {
        return (
            <div>
                {store.getState()}
                <button onClick={() => this.asyncGetStep()}>+2</button>
                <button onClick=
                    {() => store.dispatch({ type: 'reduce' })}>-</button>
            </div>
        );
    }
}

export default StoreShow;
```

先不考虑中间件，一般为了简化我们仅是组件中拿到异步数据之后调用dispatch使其更新仓库

但是redux中好像是说`它只是支持同步数据流若是异步的会只能使用中间件`。我开始还真没有get到这句话的意思

我在知乎上看到一个回答

![1602557647776](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1602557647776.png)



**那么接下来就开始中间件吧**

什么是中间件呢，和node中的作用没啥大的区别。我一般喜欢把这些中间的东西全都理解成AOP，本事它们的作用就是在中间的某一位置拦截一下进行额外的处理

那么回到这里这里，我们要知道传统的 Action必须是一个对象，意思也就是说dispatch的参数也只能是一个对象。

但是我想这样搞呢？

```js
store.dispatch(setTimeout(()=>{},1000))
```

我希望它可以接受一个函数（即往往是异步逻辑的），在函数里面得到结果再通过放进action传出去

这时候就要请`redux-thunk`这个中间件进行帮忙了，它处理异步的方式就是按照上面的逻辑来搞的

安装`  npm install redux-thunk --save `

使用很简单

在创建仓库的入口文件中，再从redux导出一个applyMiddleware方法，把中间件当做参数传进去即可

applyMiddleware的作用便是进行中间件的组合，即组合它们的顺序

```js
import { createStore, applyMiddleware } from 'redux'
import logger from 'redux-logger'
import thunk from 'redux-thunk'

const countReducer = (state = 0, action) => {
    switch (action.type) {
        case 'add':
            return state + 1
        case 'reduce':
            return state - 1
        default:
            return state
    }
}
const store = createStore(fitstReducer, applyMiddleware(thunk, 其他中间件...))
export default store
```

这时dispatch也可以接受函数当做参数了，当然一个action需要派发给reducer还得需要这个dispatch方法。故这个函数的第一个参数就是这个dispatch方法

现在你就可这样写了

```js
store.dispatch(dispatch => {
    setTimeout(() => {
        dispatch({ type: 'add' })
    }, 1000)
})
```

#### 3.2.4 react-redux

这个东西在帮了我们一点忙的同时又给增加了些使用上的复杂度

store暴露给view使用的三个方法不用继续使用了（但是action的派发还是dispatch）

它帮助我们把状态的获取和改变作为props塞进组件中了，下面看使用

安装：`  npm install reactredux --save  `

增加的几个东西（可根据栗子和图进行比较）

![1602574006158](C:\Users\T540P\AppData\Roaming\Typora\typora-user-images\1602574006158.png)

还是借助上面的栗子来体会一下吧

组件中

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux'

const mapStateToProps = (state) => {
    return {
        count: state
    }
}

const mapDispatchToProps = (dispatch) => {
    return {
        add: () => dispatch({ type: 'add' }),
        reduce: () => dispatch({ type: 'reduce' })
    }
}

class StoreShow extends Component {
    state = {}

    render() {
        return (
            <div>
                {this.props.count}
                <button onClick={() => { this.props.add() }}>点击</button>
                <button onClick=
                    {() => { this.props.reduce() }}>减⼀</button>

            </div>
        );
    }
}

export default connect(mapStateToProps, mapDispatchToProps)(StoreShow)
```

现在注意这个容器组件，还没有数据进行注入呢。这时你还需要使用一下react-redux提供的Provider组件。

```jsx

    <Provider store={store}>
      <App />
    </Provider>

```

此时容器组件中的mapStateToProps就有货了



既然connect是高级组件，那么我们也可使用装饰器的写法了

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux'

@connect(
    state => ({ count: state }),
    dispatch => ({
        add: () => dispatch({ type: 'add' }),
        reduce: () => dispatch({ type: 'reduce' })
    })
)
class StoreShow extends Component {
    state = {}

    render() {
        return (
            <div>
                {this.props.count}
                <button onClick={() => { this.props.add() }}>点击</button>
                <button onClick=
                    {() => { this.props.reduce() }}>减⼀</button>

            </div>
        );
    }
}

export default StoreShow
```

