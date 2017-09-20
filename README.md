# Tinper-Mirror


一款简洁、高效、易上手的 React 框架。（fork from [mirror](https://github.com/mirrorjs/mirror)


## 特性

* 极简 API（只有 4 个新 API）
* 易于上手
* Redux action 从未如此简单
* 支持动态创建 model
* 强大的 hook 机制

## 快速开始

### 初始化项目

使用 [uba](https://github.com/iuap-design/uba) 创建一个新的 app：

```sh
$ npm i -g uba
$ uba init
Available official templates:
? Please select : (Use arrow keys)
> template-iuap-react-solution
```


### `index.js`

```js
import React from 'react'
import mirror, {actions, connect, render} from 'tinper-mirrorx'

// 声明 Redux state, reducer 和 action，
// 所有的 action 都会以相同名称赋值到全局的 actions 对象上
mirror.model({
  name: 'app',
  initialState: 0,
  reducers: {
    increment(state) { return state + 1 },
    decrement(state) { return state - 1 }
  },
  effects: {
    async incrementAsync() {
      await new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve()
        }, 1000)
      })
      actions.app.increment()
    }
  }
})

// 使用 react-redux 的 connect 方法，连接 state 和组件
const App = connect(state => {
  return {count: state.app}
})(props => (
    <div>
      <h1>{props.count}</h1>
      {/* 调用 actions 上的方法来 dispatch action */}
      <button onClick={() => actions.app.decrement()}>-</button>
      <button onClick={() => actions.app.increment()}>+</button>
      {/* dispatch async action */}
      <button onClick={() => actions.app.incrementAsync()}>+ Async</button>
    </div>
  )
)

// 启动 app，render 方法是加强版的 ReactDOM.render
render(<App/>, document.getElementById('root'))
```

## 示例项目

* [tinper-bee-admin](https://github.com/uba-templates/template-tinper-bee-admin)
* [template-iuap-react-solution](https://github.com/uba-templates/template-iuap-react-solution)

## FAQ

#### 是否支持 [Redux DevTools 扩展](https://github.com/zalmoxisus/redux-devtools-extension)？

支持。

#### 是否可以使用额外的 Redux middleware？

可以，在 `mirror.defaults` 接口中指定即可，具体可查看文档。

#### Mirror 用的是什么版本的 react-router？

react-router 4.x。

