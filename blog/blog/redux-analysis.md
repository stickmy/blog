---
title: 从源码看 redux 机制
date: 2018-04-04 17:52:04
type: post
tag: 
  - react
  - redux
meta:
  - name: description
    content: redux 原理分析
  - name: keywords
    content: react react-redux js 前端
---


从几个方面来说:

#### 1. redux

我们看下 `redux` 源码

<!-- more -->

**[createStore](https://github.com/reactjs/redux/blob/master/src/createStore.js):**

```js
function createStore(reducer, preloadedState, enhancer) {
  // 传参判断
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState;
    preloadedState = undefined;
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.');
    }
    // 这边其实就是以前版本的中间件写法 applyMiddleware(createStore)
    return enhancer(createStore)(reducer, preloadedState);
  }

  // 一些判断
  // ...

  var currentReducer = reducer;
  var currentState = preloadedState;
  var currentListeners = [];
  var nextListeners = currentListeners;
  var isDispatching = false;

  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice();
    }
  }

  function getState() {
    return currentState;
  }

  // 注册 listener 方法, 传进一个 listener, 在 dispatch 的时候, 会调用该 listener
  // 该方法会返回一个 取消订阅的方法
  function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Expected listener to be a function.');
    }

    var isSubscribed = true;

    ensureCanMutateNextListeners();
    nextListeners.push(listener);

    return function unsubscribe() {
      if (!isSubscribed) {
        return;
      }

      isSubscribed = false;

      ensureCanMutateNextListeners();
      var index = nextListeners.indexOf(listener);
      nextListeners.splice(index, 1);
    };
  }

  // 当 dispatch 一个 action 的时候, 会执行该方法, 这样就完成了对 state 的监听
  // 是不是很熟悉, 可以看出该方法其实就是执行了 reducer(action)
  // 然后调用监听方法(这个方法一般来说就是 mapStateToProps)
  function dispatch(action) {
    // 一些异常判断
    // ...

    try {
      isDispatching = true;
      // 这边其实就是执行我们编写的 reducer
      currentState = currentReducer(currentState, action);
    } finally {
      isDispatching = false;
    }

    var listeners = currentListeners = nextListeners;
    // dispatch -> 执行 listener 监听方法
    for (var i = 0; i < listeners.length; i++) {
      var listener = listeners[i];
      listener();
    }

    return action;
  }

  // 替换 reducer, 一般有两个场景会用到
  // 1. 开发的时候, 热更新替换 reducer
  // 2. 异步 reducer 加载
  function replaceReducer(nextReducer) {
    // ...
  }

  // 初始化的时候, 得到初始 state
  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer
  }
}
```

`createStore` 方法传入了三个参数,  初始化的 `reducer`, 初始化的 `state`, 以及用来增强 `createStore` 的 `enhancer`(很多中间件就是这样传过来的, 这个后面会解释)

那其实看到这边, 我们已经能够大概理解它的工作方式了, 我们尝试使用一下

```js
const ADD = 'ADD'
const REMOVE = 'REMOVE'

export function counter(state = 0, action) {
  switch (action.type) {
    case ADD:
        return state + 1
    case REMOVE:
        return state - 1
    default:
        return 0
  }
}

export function add() {
  return { type: 'ADD' }
}
export function remove() {
  return { type: 'REMOVE' }
}
// 传入 reducer
const store = createStore(counter)
// 这样可以获取到初始的 state
const initialState = store.getState()

function listener() {
  const current = store.getState()
  console.log(`当前 count: ${current}`)
}
// 调用 subscribe 方法, 注册一个 listener, 每次 dispatch 的时候, 都会执行该方法
store.subscribe(listener)
store.dispatch({ type: 'ADD' })
store.dispatch({ type: 'REMOVE' })
```

但是似乎这样的写法, 跟我们预期甚远, 因为这样太过繁琐, 耦合度太高, 我们需要抽象, 这就是 `react-redux` 了

#### 2. react-redux

我们常用的 `react-redux` api, 一般有 `connect`, `Provider` 组件等等, 我们将根组件用 `Provider` 包裹起来, 然后我们子组件就能拿到 `store` 分发下来的值了

这里有一个疑问, 为什么经过 `Provider` 包裹, 子组件就都能使用 `store` 中的值了呢?

当子组件需要使用祖先组件中的 `state` 值的时候, 难道我们需要一层层将值通过 `props` 传下去吗?

不需要, 答案是使用了 [react context](https://github.com/brunoyang/blog/issues/9), 通过定义 `getChildContext` 以及 `childContextTypes`, 在子组件中, 我们就可以直接使用祖先组件中的值了

**看代码 [Provider](https://github.com/reactjs/react-redux/blob/master/src/components/Provider.js)**

```js
function createProvider(storeKey = 'store', subKey) {
  const subscriptionKey = subKey || `${storeKey}Subscription`
  
  class Provider extends React.Component {
    // 定义 getChildContext, 这样子组件可以直接获取到 store
    getChildContext() {
      return { [storeKey]: this[storeKey], [subscriptionKey]: null }
    }

    constructor(props, context) {
      super(props, context)
      this[storeKey] = props.store;
    }

    render() {
      // Children.only 限制 Provider 只接受一个子组件
      return Children.only(this.props.children)
    }
  }

  // 开发模式的一些优化
  if (process.env.NODE_ENV !== 'production') {
    Provider.prototype.componentWillReceiveProps = function (nextProps) {
      if (this[storeKey] !== nextProps.store) {
        warnAboutReceivingStore()
      }
    }
  }

  // 以下必须定义
  Provider.propTypes = {
      store: storeShape.isRequired,
      children: PropTypes.element.isRequired,
  }
  Provider.childContextTypes = {
      [storeKey]: storeShape.isRequired,
      [subscriptionKey]: subscriptionShape,
  }

  return Provider
}
```

接下来, 我们看看 `connect` 方法做了些什么

从 `connect()(Component)` 的书写方式来看, 我们已经可以初窥端倪, `connect` 极有可能是一个 `HOC`(高阶组件), 它将我们的组件传进去, 为组件注入一些方法, 属性, 再 `return` 出来

**看代码 [connect](https://github.com/reactjs/react-redux/blob/master/src/connect/connect.js)**

```js
// 我们将这个方法简化了一下, 实际上还做了很多处理
function createConnect() {
  return connect = (
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    {}
  ) => (WrapComponent) => {

    return class ConnectComponent extends React.Component{
      static contextTypes = {
        store: PropTypes.object
      }

      constructor(props, context){
        super(props, context)
        this.state = { props: {} }
      }

      componentDidMount(){
        const { store } = this.context
        // 为 store 注册一个监听函数
        store.subscribe(() => this.listener())
        this.listener()
      }

      // 监听函数
      listener = () => {
        const { store } = this.context
        /**
          * mapStateToProps 方法的参数 state 就是这样传过去的,
          * 这样每次当 store 变化的时候, 就会执行一次 listener 监听函数,
          * 如此, mapStateToProps 就会接受到新的 state, 组件就能拿到这个值了
          */
        const stateProps = mapStateToProps(store.getState())
        // 这边将传进来的 mapDispatchToProps, 经过 bindActionCreators 包裹了一下
        // bindActionCreators 其实就是: 
        // (...args) => dispatch(mapDispatchToProps(...args))
        // 见 [bindActionCreators](https://github.com/reactjs/redux/blob/master/src/bindActionCreators.js)
        // 这里你似乎有一个疑问, dispatch 的参数是 action
        // 而编写异步 reducers 的时候, return 的可不是 action, 这个就是 redux-thunk 做的事了, 下面会解释
        const dispatchProps = bindActionCreators(mapDispatchToProps, store.dispatch)
        this.setState({
          props: {
            ...this.state.props,
            ...stateProps,
            ...dispatchProps  
          }
        })
      }
      render() {
        return <WrapComponent {...this.state.props}></WrapComponent>
      }
    }
  }
}

export default createConnect()
```

我们看出 `connect` 就是为我们传入的组件注入需要的 `props` 和 `dispatch` 方法, 同时利用 `createStore` 暴露出来的 `subScribe` 方法注册一个 `store` 的监听函数, 这个监听函数获取到最新的 `state`, 当做参数传给 `mapStateToProps`, 这样当 `store` 变化的时候, 我们的组件就能接受到最新的 `props`

#### 3. applyMiddleware

上面我们提到了 `redux-thunk`, 它其实就是中间件的一种, 中间件可以加强我们的 `createStore` 方法

在了解 `redux-thunk` 之前, 我们先看看中间件是怎么回事

**看代码 [applyMiddleware](https://github.com/reactjs/redux/blob/master/src/applyMiddleware.js)**

```js
function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    const _dispatch = store.dispatch
    let chain = []

    // 实现一个新的 dispatch
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => _dispatch(...args)
    }

    /** 
      * 这里给每个中间件都传入了 getState 和 dispatch 方法
      * 从这里看, 中间件经过的几次调用
      * 1. middleware(middlewareAPI), 将 middlewareAPI 传入中间件
      * 2. compose(...chain)(store.dispatch) 给中间件传入 store.dispatch 方法
      * 3. 经过 1, 2 调用, 我们的中间件就变成了最终的 dispatch, 这个 dispatch 就是最终用户调用的
      * 所以, 由此猜测我们的中间件是一个三阶的高阶函数, 形如 middlewareAPI => store.dispatch => action
      */
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    // 这里我们为上面的 chain 进行柯里化, 传入 store.dispatch 方法
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}

// 不难看出, compose 其实是一个柯里化函数, 它将传入的 function 从右到左依次调用, 并且返回结果
// compose([a, b, c]) -> (...args) => a(b(c(...args)))
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

我们可以看到 `applyMiddleware` 方法 `return` 了 `store` 和 `dispatch`, 这个方法为每个中间件都传入了 `getState` 和 `dispatch` api, 然后再对中间件传入 `store.dispatch` 方法

看到这里可能你对这个函数还是云里雾里, 这里我们结合 `redux-thunk` 这个实例来看下

#### 4. redux-thunk

**看代码 [redux-thunk](https://github.com/gaearon/redux-thunk/blob/master/src/index.js)**

```js
function createThunkMiddleware(extraArgument) {
  /** 直接 return 了一个函数 wrapMiddleware, 该函数接受一个 ({ dispatch, getState })
    * 可以看出, 这里的 ({ dispatch, getState }) 就是上面的 middlewareAPI
    * 然后在 applyMiddleware 中, 执行了 wrapMiddleware, 这个方法 return 了一个函数 wrapDispatch
    * wrapDispatch 也接受一个参数, 在上面的代码中, 我们可以看出这个参数就是 store.dispatch
    * 接下来就很容易理解了, 为 dispatch 传入一个 action, 看到这里, 熟悉的感觉又回来了
    */
  return ({ dispatch, getState }) => next => action => {
    // 如果该 action 是方法, 那么我们就传入 dispatch, getState 执行, 并且 return 出来
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;
export default thunk
```

这里传入一个中间件的场景已经分析完了, 当我们传入多个中间件的时候, `applyMiddleware` 是怎么工作的呢

我们注意到中间件执行一次之后的函数参数叫做 `next`, 我们思考下有什么含义吗?

我们看下 `compose` 方法, 不难看出数组参数最后一个方法会最先执行

```js
applyMiddleware([m1, m2, m3])
// 那么会先执行 m3(middlewareApi)(store.dispatch) => next => action => next(action)
// 它 return 了一个方法: next => action => next(action), 该方法会被 m2(middlewareApi)(next) 调用
// 那么我们已经看出来了, next => action => next(action) 它就是该中间件的 dispatch
// 也就是 m2(middlewareApi)(next) 中的 next 参数
// 就这样, 每层中间件都 return 一个自己的 dispatch 给下一个中间调用, 完成了 dispatch 的传递
```

通过对 `applyMiddleware` 的分析, 我们发现自己写一个 `redux` 中间件也很简单