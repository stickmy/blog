---
title: Redux 与 Koa 中间件对比
date: 2018-06-15 00:11:09
type: post
tag: 
  - koa
  - react
  - redux
meta:
  -
    name: description
    content: Redux 与 Koa 中间件对比
  -
    name: keywords
    content: redux koa middleware 中间件
---

### 先扯淡
好吧, 这两个其实根本不是一个类型的东西, 一个是 `nodejs` 框架, 一个是数据流管理方案. 不过, 我还是要来对比...

虽然, 他们不是一种东西, 但是从他们的中间件的角度来看, 其实实现了同一种效果. 所以我们来对比一下实现的差异
<!-- more -->
### Redux

我们看下 `applyMiddleware` 这个方法

```js
function applyMiddleware (...middlewares) {
    return createStore => (...args) => {
        const store = createStore(...args)
        let dispatch = store.dispatch

        let chain = []

        const middlewareAPI = {
            getState: store.getState,
            dispatch: (...args) => dispatch(...args)
        }
        /**
         * 这里可以看出每一个中间件都应该是一个三阶的函数
         * 1. 第一阶用于传入 middlewareAPI, return next => action
         * 2. 第二阶用于传入 store.dispatch, return action => {}
         * 3. 第三阶用于留给用户调用 
         */
        chain = middlewares.map(middleware => middleware(middlewareAPI))

        /**
         * compose 函数会将所有的中间件串联成一个中间件, 中间件会从左到右依次执行
         */
        dispatch = compose(...chain)(store.dispatch)

        return {
            ...store,
            dispatch
        }
    }
}

// [a, b, c] => (...args) => a(b(c(...args)))
function compose (...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }
  
  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...arg) => a(b(...arg)))
}
```

我们可以看到, 通过 `applyMiddleware` 对中间件的几次调用, 可以推测出中间件应该是一个三阶的高阶函数, 形如 `middlewareAPI => store.dispatch => action`, 这里注意 `compose` 组合函数, 它会将函数从左到右依次执行

现在假设我们有两个中间件 `logger1`, `logger2`

`logger1`:

```js
const logger1 = ({ getState, dispatch }) => next => action => {
  console.log('logger1 start')
  next(action)
  console.log('logger2 end')
}

const logger2 = ({ getState, dispatch }) => next => action => {
  console.log('logger2 start')
  next(action)
  console.log('logger2 end')
}
```

调用他们: 

```js
const store = createStore(0, reducer, applyMiddleware(logger1, logger2))
```

结果的输出顺序是什么? 可以 `clone` 这个仓库 [redux-play](https://github.com/Bloss/redux-play) 的代码, 运行一下, 看看结果 

打印日志: 

```js
`logger1 start`
`logger2 start`
`logger2 end`
`logger1 end`
```

我们看看执行的流程图, 我们称 `compose` 返回的函数为 `composed`

```js
---------- composed 从左至右执行 ------------
----          logger1 start            ----
----               ↓                   ----
-- logger1.next() //logger2.action => {} --
----               ↓                   ----
----           logger2 start           ----
-- logger2.next() // store.dispatch      --
----               ↓                   ----
----           logger2 end             ----
----               ↓                   ----
----           logger1 end             ----
```

看到这里, 可以明白 `redux` 中间件的工作原理了: 后面一个中间件的 `next` 会被前一个中间件包裹, 所以每个中间件执行到 `next` 的时候都会阻塞, 然后去执行下一个中间件的 `next`. 所有 `next` 执行结束之后, `next` 后面的代码再反向执行

### Koa

`koa` 在 `http.createServer ` 中的 `callback` 中执行了一段代码 `compose(this.middleware)`. 这段代码是中间件执行的关键. 这段代码跟 `redux` 中的 `compose` 作用其实是一样的: 将 `middlewares` 串联成一个函数执行

`compose`:

```js
function compose (middleware) {
  return function (context, next) {
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) throw Error('中间件 next 不能执行多次')
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispath.bind(null, i + 1)))
      } catch (e) {
        return Promise.reject(e)
      }
    }
  }
}
```

上面 `dispatch.bind(null, i + 1)` 就是中间件中的 `next`, 所以当我们在一个中间件中多次执行 `next` 的时候, 第一次执行 `next` 之后, `index` 的值会被 `i` 覆盖, 第二次执行 `next` 的时候触发了 `i <= index`, 所以在 `koa2` 中一个中间件中多次执行 `next` 是会报错的.

其次, `compose` 函数最终返回的都是 `Promise`, 这其实是一种兼容写法, 这样无论中间件的形式是普通函数还是 `async` 函数还是 `generator` 函数, 都可以用 `await` 来监听它. 所以第一个中间件执行到 `next` 的时候, 就会触发第二个中间件的执行, 第二个中间件执行到 `next` 的时候就会触发第三个中间件, 以此类推, 所有 `next` 执行完之后, 再回溯执行. 这就是 `koa` 的洋葱模型了.

流程图

```js
----------     composed 执行    ------------
----------      dispatch(0)    ------------
----               ↓                   ----
----    await next() // dispatch(1)    ----
----               ↓                   ----
----    await next() // dispatch(2)    ----
----               ↓                   ----
----  中间件执行结束, Promise.resolve()  ----
----               ↓                   ----
----------       开始回溯执行      ----------
```

### 不算小结的小结

- `koa` 中间件天生支持异步操作, 而 `redux` 需要诸如 `redux-thunk` 这样的东西来实现异步, 原理就是将 `disptach` 转移给用户, 让用户选择何时执行, 譬如 `Promise` 执行完毕后再 `dispatch`

- 构建中间件的两种方式, 也可以说是一种: 在 `next` 中去执行下一个中间件, 达到串联执行所有中间件的目的, 它的特征是 `next` 之后的代码会阻塞执行从而形成回逆. 这篇写完以后再也不说 `redux` 中间件了, 感觉都说腻了 😂