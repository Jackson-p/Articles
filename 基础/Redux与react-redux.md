# learn-redux

主要从redux到react-redux的进化过程，js-code仓库里的那个图已经很好的讲了redux的整个单向数据流是怎么流的，redux存在的目的就是当状态很多时就不需要通过state来进行管理了，而是由统一的store来管理。

## 纯redux

__index.js:__

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import { addliao, decliao, asyncAddliao, adjustLiao} from './index.redux'
import App from './App'

const store = createStore(adjustLiao, compose(
    applyMiddleware(thunk),
    window.__REDUX_DEVTOOLS_EXTENSION__?window.__REDUX_DEVTOOLS_EXTENSION__():()=>console.log('no redux tool')
))

function render(){
    ReactDOM.render(
        <App store={store} addliao={addliao} decliao={decliao} asyncAddliao={asyncAddliao} />,
        document.getElementById('root')
    )
}
render()
store.subscribe(render)
```

__App.js:__(负责纯view)

```js
import React from 'react'
import { Button } from 'antd-mobile'

export default class App extends React.Component{
    render(){
        const store = this.props.store
        const addliao = this.props.addliao
        const decliao = this.props.decliao
        const asyncAddliao = this.props.asyncAddliao
        const num = store.getState()
        return(
            <div>
                <h1>现在有{num}只liao</h1>
                <Button type="primary" onClick={() => store.dispatch(addliao())} style={{"cursor":"pointer","width":"50%","margin":"auto"}}>加liao</Button>
                <br />
                <Button type="primary" onClick={() => store.dispatch(decliao())} style={{"cursor":"pointer","width":"50%","margin":"auto"}}>减liao</Button>
                <br />
                <Button type="primary" onClick={() => store.dispatch(asyncAddliao())} style={{"cursor":"pointer","width":"50%","margin":"auto"}}>异步加liao</Button>
            </div>
        )
    }
}
```

__index.redux.js:__(定义action和reducer)

```js

const ADD_LIAO = "ADD_LIAO"
const DEC_LIAO = "DEC_LIAO"
//action
export function addliao(){
    return {type:ADD_LIAO}
}
export function decliao(){
    return {type:DEC_LIAO}
}

//reducer
export function adjustLiao(state=0,action){
    switch(action.type){
        case ADD_LIAO:
            return state+1
        case DEC_LIAO:
            return state-1
        default:
            console.log('state initialed')
            return state
    }
}

export function asyncAddliao(){
    return (dispatch) => setTimeout(() => dispatch(addliao()),1000)
}
```

为了更方便于与React的结合开发，省的把store传来传去，react-redux传入且仅传入了2个API，Provider和connect。

## react-redux

对于React进行优化的redux也就是react-redux的版本如下：

__index.js:__

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import { adjustLiao} from './index.redux'
import { Provider } from 'react-redux'
import App from './App'

const store = createStore(adjustLiao, compose(
    applyMiddleware(thunk),
    window.__REDUX_DEVTOOLS_EXTENSION__?window.__REDUX_DEVTOOLS_EXTENSION__():()=>console.log('no redux tool')
))

ReactDOM.render(
    <Provider store={store}>
        <App/>
    </Provider>,
    document.getElementById('root')
)
```

__App.js:__

```js
import React from 'react'
import { connect } from 'react-redux'
import { addliao, decliao, asyncAddliao } from './index.redux'

import { Button } from 'antd-mobile'

// @connect(
//     (state) => ({num:state}), //注意这里的括号不能省
//     {addliao, decliao, asyncAddliao}
// )
class App extends React.Component{
    render(){
        return(
            <div>
                <h1>现在有{this.props.num}只liao</h1>
                <Button type="primary" onClick={this.props.addliao} style={{"cursor":"pointer","width":"50%","margin":"auto"}}>加liao</Button>
                <br />
                <Button type="primary" onClick={this.props.decliao} style={{"cursor":"pointer","width":"50%","margin":"auto"}}>减liao</Button>
                <br />
                <Button type="primary" onClick={this.props.asyncAddliao} style={{"cursor":"pointer","width":"50%","margin":"auto"}}>异步加liao</Button>
            </div>
        )
    }
}

const mapStateToProps = (state) => {
    return { num : state }
}

const mapDispatchToProps = {addliao, decliao, asyncAddliao}

export default connect(mapStateToProps, mapDispatchToProps)(App)
```

__index.redux.js__ 没变化，由上总结

---

__Provider组件:__

避免了开发中对store的手动操作（更新，下层传递store问题等）。

__connect组件：__

一方面mapStateToProps可以把store里面的state“远程”传递出来作为当前组件的props即取操作。

另一方面mapDispatchToProps可以把已有的函数dispatch封装过后转移到当前组件的props上，以后的action直接通过this.props.“functions”触发。