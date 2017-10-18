# mirrorx

基于redux,react-routerv4,redux-react的mvvm数据管理框架。

## api

### model

mirror.model({name, initialState, reducers, effects})

定义数据模型和状态(state)，状态(state)修改，发送异步请求。

- name

类型：string

每一个model，都必须要写一个唯一的`name`。 `routing`不可以作为model的name。


```
import mirror from 'mirrorx'

mirror.model({
  name: 'app',
})
```

#### model内的Option

##### initialState

类型：any

声明初始的state

```
import mirror from 'mirrorx'

mirror.model({
  name: 'app',
  initialState: 'yonyou'
})
```
多个state,这样写。

```
import mirror from 'mirrorx'

mirror.model({
  name: 'app',
  initialState: {
      show: false,
      data: []
  }
})
```
#### reducers

类型：object，内部是function

- reducers是修改state的方法。
- 每个reducers要返回一个新的state对象。
- reducers会被挂载到mirror的actions对象下。以`actions.[modal名].[方法名]`调用
- 每个reducers都是一个纯函数。

```
import mirror from 'mirrorx'
import mirror, {actions} from 'mirrorx'

mirror.model({
  name: 'app',
  initialState: 0,
  reducers: {
   add(state, key) {
     console.log(key);
     return state
   },
  },
})
```

`add(state, data)`state是在initialState声明的内容。data是通过调用该方法传入的数据。



调用方法:

```
import React, { Component } from 'react';
import mirror, { Link, actions, connect } from 'mirrorx';

function Button({app}) {
    return (
        <button
            onClick={() => actions.app.add(1)}
        >
        {app}增长
        </button>
    )
}

//连接model和组件
export default connect((app) => {
    return app;
})(Button);

```

##### effects

类型：对象，属性为async function

`effects`是写与函数外部发生交互的操作。比如：请求接口，异步操作等。

`effects`不要直接修改state，如果要修改state，要调用reducers内定义的方法。

在 effects 中定义的所有方法都会以相同名称添加到 actions.<modelName> 上，调用方式与reducers相同。`effects`不能和reducers重名。


```
import mirror, {actions} from 'mirrorx'

mirror.model({
  name: 'app',
  initialState: 0,
  reducers: {
    add(state, data) {
      return state + data
    },
  },
  effects: {
    async myEffect(data, getState) {
      const res = await Promise.resolve(data)
      actions.app.add(res)
    }
  },
})
```

`myEffect(data, getState)`,`data`是传入的数据。`getState`执行`getState()`方法可以获取当前的State值。

在`effects`中，你可以使用任何函数（普通函数。Promise等）。我们这里建议大家使用es7的async/await语法。

[async教程](http://www.ruanyifeng.com/blog/2015/05/async.html)


### actions

使用`mirror.actions.modelName.functionName`方法调用定义在model内`reducers`和`effects`定义的方法。

#### actions.routing

如果你使用了Mirror内的`Router`组件，就可以使用这个对象，这个对象下有五个方法，用来更行location:

- push(location);
```
pathname      跳转的路径部分
search        查询url部分，包含?
state         数据对象，发给新的路由
action         PUSH/ REPLACE/ POP
key           此位置的唯一标识符

```
```
// Pushing a path string.
history.push('/the/path')

//
history.push({ pathname: '/the/path', search: '?the=search' })

//扩展现有的位置对象
history.push({ ...location, search: '?other=search' })
```

向history添加一条记录，并跳转到location
- replace(location)

替换当前location.

- go

往前或者往后跳转 history 中的 location。

- goForward

 往前跳转一条 location 记录，等价于 go(1)。

- goBack

往后跳转一条 location 记录，等价于 go(-1)。

事实上，这些方法来自于 [history API](https://github.com/ReactTraining/history/blob/v3/docs/GettingStarted.md#navigation)，所以意义和用法完全一致。

### hook钩子

mirror.hook((action, getState) => {})

这是一个非常强大的接口，能够让你监控每一个方法和路由跳转。

```
import mirror, {actions} from 'mirrorx'

// ...

const locationChangeHook = mirror.hook((action, getState) => {
  if (action.type === '@@router/LOCATION_CHANGE') {
    console.log('Location has just changed')
  }
})

const countHook = mirror.hook((action, getState) => {
  if (getState().app.count === 10) {
    console.log('You have just reached 10!')
  }
})

// 移除 hook
locationChangeHook()
countHook()
```
例子：路由跳转，如果是`/users`就开始请求加载数据

```
mirror.hook((action, getState) => {

  const {routing: {location}} = getState()

  if (action.type === LOCATION_CHANGE && location.pathname === '/users') {
    const query = qs.parse(location.search)
    actions.users.load(query)
  }
})
```

- 对于函数触发，`action.type`等于你的`<modal名称>/方法名称`。
- 对于路由跳转，`action.type`等于`@@router/LOCATION_CHANGE`
- `getState()`可以获取当前state的值。



### defaults

mirror.defaults(options)

设置一些默认的mirror选项。

- historyMode

默认值: browser

设置前端路由规则

- browser - 标准的 HTML5 hisotry API。
- hash - 针对不支持 HTML5 history API 的浏览器。
- memory - history API 的内存实现版本，用于非 DOM 环境。

```
mirror.defaults({
    historyMode: 'hash'
});
```

- initialState

默认值：`undefined`

当你从一些持久化的存储器或服务器渲染时，可以使用这个属性来设置。

```
mirror.defaults({
  initialState: {app: 1}
})

mirror.model({
  name: 'app',
  // ...
})

// ...

store.getState()
// {app: 1}
```

- middlewares

默认值： []

可以添加第三方或者时自定义的中间件。

假如你想使用一些第三方的 middleware，那么可以在这个选项中指定。同时，你需要调用 connect 且不传递 mapDispatchToProps 来获取 props.dispatch 方法，然后手动 dispatch action。

- reducers

默认值： {}

指定一些额外的 reducer。注意这里定义的 reducer 必须为标准的 Redux reducer，这些 reducer 会直接被 combineReducers 处理。

- addEffect

默认值: (effects) => (name, handler) => { effects[name] = handler }

### connect

connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

通过`connect`来连接你的组件和模型。

```
import {connect} from 'mirrorx';

connect(({menuModel}) => menuModel)(App);
```


### render

render([component], [container], [callback])

`render`是对ReactDOM的render的一个封装。将组件渲染添加到DOM上。api与eactDOM的render一致。

```
import {render} from 'mirrorx';

render(<App/>, document.getElementById('root'))

```

### Router

> Mirror 使用的是 [react-router@4.x](https://github.com/ReactTraining/react-router)，如果你有 react-router 2.x/3.x 的经验，那么你应该仔细阅读一下 react-router 官方的[迁移指南](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/guides/migrating.md)。

Mirror 的 `Router` 组件是加强版的 [react-router](https://github.com/ReactTraining/react-router/tree/master/packages/react-router) 的 `Router`。所加强的地方在于，`Redux store` 和 `history` 都自动处理好了，不需要你去做关联，也不需要你去创建 `history` 对象，你只需要关心自己的业务逻辑，定义路由即可。当然，如果你想自己创建一个 `history` 对象，然后通过 prop 传递给 `Router` 组件，也是没有任何问题的。

那 [`basename`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/BrowserRouter.md#basename-string) 以及 [`getUserConfirmation`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/BrowserRouter.md#getuserconfirmation-func) 等 props 呢？不用担心，Mirror 的 `Router` 全都能处理它们。你可以查看 [`BrowserRouter`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/BrowserRouter.md)、[`HashRouter`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/HashRouter.md) 和 [`MemoryRouter`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/MemoryRouter.md) 的文档获取更多信息。

因为 Mirror 没有将 `Router` 用到的 `history` 暴露出去，如果你需要手动更新 location，那么你可以使用 `actions.routing` 上的方法。

以下这些组件，都来自 `react-router`，Mirror 也都暴露出去了，你可以直接引入：

* [`Route`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/Route.md)
* [`Switch`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/Switch.md)
* [`Redirect`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/Redirect.md)
* [`Link`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/Link.md)
* [`NavLink`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/NavLink.md)
* [`Prompt`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/Prompt.md)
* [`withRouter`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/withRouter.md)

一个简单的例子：

```js
import {render, Router, Route, Link} from 'mirrorx'

// ...

const App = () => (
  <div>
    <nav>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/about">About</Link></li>
        <li><Link to="/topics">Topics</Link></li>
      </ul>
    </nav>

    <div>
      <Route exact path="/" component={Home}/>
      <Route path="/about" component={About}/>
      <Route path="/topics" component={Topics}/>
    </div>
  </div>
)


render(
  <Router>
    <App/>
  </Router>
, document.getElementById('root'))
```


## 其他使用拓展

### 动态载入modal

你可以在 app 中多次调用 render。第一次调用会使用 mirror.model 方法中定义的 reducer 和 effect 来创建 store。后续的调用将会 使用 replaceReducer 替换 store 的 reducer，并重新渲染整个 app。

这样处理的意义是什么呢？就是你可以动态载入 model 了。

举例来说，假如你有一个 `app.js`：

```
// app.js
import React from 'react'
import mirror, {actions, connect, render} from 'mirrorx'

mirror.model({
  name: 'foo',
  initialState: 0
})

const App = connect(({foo, bar}) => {
  return {foo, bar}
})(props => {
  return (
    <div>
      <div>{props.foo}</div>
      <div>{props.bar}</div>
    </div>
  )
})

render(<App/>, document.getElementById('root'))
```
render 之后，你的 app 会被渲染成下面这样：

```
<div>
  <div>0</div>
  <div></div>
</div>
```

然后，你在 app 的某个地方，定义了一个 bar model，或者是通过 ajax 加载过来的：

```
// ...

// 注入一个异步 model。注意这一步不会导致重新渲染
mirror.model({
  name: 'bar',
  initialState: 'state of bar'
})

// 调用 render 后，会重新渲染
render()
```

不传递参数调用 render 将会重新渲染你的 app。所以上述代码将会生成以下 DOM 结构：

```
<div>
  <div>0</div>
- <div></div>
+ <div>state of bar</div>
</div>
```
这在大型 app 中非常有用。

注意：Mirror 不建议传递 component 和 container 参数来重新渲染你的 app，因为这样做可能会导致 React mount/unmount 你的 app。如果你只希望重新渲染，永远不要传递任何参数给 render。


### 使用redux-saga

[mirror-saga](https://github.com/ShMcK/mirror-saga)








