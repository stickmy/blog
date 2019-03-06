---
title: React-Router 原理分析
date: 2018-04-07 13:53:18
type: post
tag: 
  - js
  - react
meta:
  - name: description
    content: react router 机制, 原理分析
  - name: keywords
    content: react react-router js 前端
---

## 说明

本篇文章基于 `React-Router@4.0`, 原理上都是一样的, 但是 `react-router` 版本之间 `api` 差距巨大, 特此说明

## history

`react-router` 依赖一个第三方库 [history](https://github.com/ReactTraining/history), 这个库兼容了不同浏览器, 不同环境下对浏览器历史记录的管理, 拥有统一的 `api`,那我们就先看一下这个库

<!-- more -->

它有三种模式:

- `hash mode`: 一些老版本浏览器, 通过 `hash` 来实现路由, 对应 `createHashHistory`
- `browserHistory mode`: `html5` 的 `history`, 对应 `createBrowserHistory`
- `memoryHistory mode`: 用于 `node` 环境, 对应 `createMemoryHistory`

我们以 `browserHistory` 为例, 看下代码

```js
function createBrowserHistory(options = {}) {
  // ...
  return {
    length: globalHistory.length,
    action: "POP",
    location: initialLocation,
    createHref,
    push, // push location
    replace, // 替换 location
    go,
    goBack,
    goForward,
    block,
    listen // location发生改变时的 callback
  }
}
```

我们可以看到 `createBrowserHistory` 提供了一些 `api`, 可以用来监听 `location` 的变化, 执行 `location` 的前进, 后退, 替换等操作

`history` 看到这里就够了, 下面看看 `React-Router`

## react-router@4.0

我们上面看过了 `history` 的 `api`, 发现有几个钩子函数以及监听函数, 那么其实我们已经有一些思路了

**我们看代码, 下面的代码是我对源码加以注释过的, 地址在这 [react-router-analysis](https://github.com/Bloss/react-router-analysis)**

看代码 [Router](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/modules/Router.js)

```js
class Router extends React.Component {
  static propTypes = {
    // history mode ==> browserHistory, hashHistory, memoryHistory 三种模式
    history: PropTypes.object.isRequired,
    // <Route />
    children: PropTypes.node
  };

  static contextTypes = {
    router: PropTypes.object
  };

  static childContextTypes = {
    router: PropTypes.object.isRequired
  };

  getChildContext() {
    return {
      router: {
        ...this.context.router,
        history: this.props.history,
        route: {
          // 此处乃是浏览器 url 更新的行为
          // history.location, 当点击 Link 或者 push 路由的时候, location 会变化, Route 就会更新
          location: this.props.history.location,
          match: this.state.match // parent, 根路由 /
        }
      }
    };
  }

  state = {
    match: this.computeMatch(this.props.history.location.pathname)
  };

  computeMatch(pathname) {
    return {
      path: "/",
      url: "/",
      params: {},
      isExact: pathname === "/"
    };
  }

  componentWillMount() {
    const { children, history } = this.props;

    invariant(
      children == null || React.Children.count(children) === 1,
      "A <Router> may have only one child element"
    );

    // 注册 history 的监听函数, 回调执行 setState({ match }), 对应的 childContext 也会更新
    this.unlisten = history.listen(() => {
      this.setState({
        match: this.computeMatch(history.location.pathname)
      });
    });
  }

  componentWillReceiveProps(nextProps) {
    warning(
      this.props.history === nextProps.history,
      "You cannot change <Router history>"
    );
  }

  componentWillUnmount() {
    this.unlisten();
  }

  render() {
    const { children } = this.props;
    return children ? React.Children.only(children) : null;
  }
}
```

在上面的代码中, 我们看到组件中定义了 `getChildContext`, 是一个 `router` 对象, 而在这个对象中, 我们注意到有 `location` 这个对象, 它的值为 `this.props.history.location`, 这个值就是当我们执行 `push(path)` 或者点击 `Link` 的时候, 调用了 `history.pushState` 之后, 组件对应的 `location` 值, 那么 `react-router` 是如何根据这个值去寻找对应的组件并更新的呢, 这里就要说到关键的部分: `Route` 以及 `matchPath` 这个函数了.

我们先看看 [matchPath](https://github.com/Bloss/react-router-analysis/blob/master/react-router/matchPath.js)

```js
import pathToRegexp from "path-to-regexp";

const patternCache = {};
const cacheLimit = 10000;
let cacheCount = 0;

// 缓存解析的 path
const compilePath = (pattern, options) => {
  const cacheKey = `${options.end}${options.strict}${options.sensitive}`;
  const cache = patternCache[cacheKey] || (patternCache[cacheKey] = {});

  if (cache[pattern]) return cache[pattern];

  const keys = [];
  const re = pathToRegexp(pattern, keys, options);
  const compiledPattern = { re, keys };

  if (cacheCount < cacheLimit) {
    cache[pattern] = compiledPattern;
    cacheCount++;
  }

  return compiledPattern;
};

// 关键的匹配函数
const matchPath = (pathname, options = {}, parent) => {
  if (typeof options === "string") options = { path: options };

  const { path, exact = false, strict = false, sensitive = false } = options;

  if (path == null) return parent;

  const { re, keys } = compilePath(path, { end: exact, strict, sensitive });
  const match = re.exec(pathname);

  // 不匹配
  if (!match) return null;

  const [url, ...values] = match;
  const isExact = pathname === url;

  if (exact && !isExact) return null;

  return {
    path, // the path pattern used to match
    url: path === "/" && url === "" ? "/" : url, // 经过 matchPath 匹配后的 url
    isExact, // 是否完全匹配
    params: keys.reduce((memo, key, index) => {
      memo[key.name] = values[index];
      return memo;
    }, {})
  };
};
```

上面的代码就是 `react-router` 的匹配函数了, 第一个是缓存函数, 我们先不管, 看下第二个函数, 我们发现, 它根据传入的参数, 计算出了 `match` 这个值, 如果不匹配, 就 `return null` , 匹配的话则返回一个记载了组件状态的对象, 这个函数是在哪被执行的呢? 那么接下来就看看 `Route` 这个组件

**重点: 这个组件是 react-router 如何根据 url 渲染 react 组件的关键**

**[Route](https://github.com/Bloss/react-router-analysis/blob/master/react-router/Route.js)**

```js
import React from "react";
import PropTypes from "prop-types";
import matchPath from "./matchPath";

const isEmptyChildren = children => React.Children.count(children) === 0;

class Route extends React.Component {
  static propTypes = {
    computedMatch: PropTypes.object, // private, from <Switch>
    path: PropTypes.string,
    // 是否精确匹配, true: path === location.pathname => true
    // case: true:  /one /one/two ===> false
    exact: PropTypes.bool, 
    // 是否匹配 '/'
    //  /one/ /one ===> false
    //  /one/ /one/ ===> true
    strict: PropTypes.bool,
    sensitive: PropTypes.bool, // 匹配字母大小写
    component: PropTypes.func,
    render: PropTypes.func,
    children: PropTypes.oneOfType([PropTypes.func, PropTypes.node]),
    location: PropTypes.object
  };

  // 获取 Router 的 childContext
  static contextTypes = {
    router: PropTypes.shape({
      history: PropTypes.object.isRequired,
      route: PropTypes.object.isRequired,
      staticContext: PropTypes.object
    })
  };

  static childContextTypes = {
    router: PropTypes.object.isRequired
  };

  // react-context
  getChildContext() {
    return {
      router: {
        ...this.context.router,
        route: {
          location: this.props.location || this.context.router.route.location,
          match: this.state.match
        }
      }
    };
  }

  state = {
    // this.context.router: Router 中设置的 childContext
    match: this.computeMatch(this.props, this.context.router)
  };

  // 计算匹配路由
  computeMatch(
    { computedMatch, location, path, strict, exact, sensitive },
    router
  ) {
    // 用了 Switch, 用 Switch 去匹配路由, 其实 Switch 匹配也是根据 matchPath 来匹配的
    if (computedMatch) return computedMatch; // <Switch> already computed the match for us

    // route: Router 中设置的 context
    const { route } = router;
    // path: Route 自己的 path
    // route.location: react-router 路由变更时, 重新根据 history.location 对象匹配
    // path 和 route.location 匹配成功则说明该组件就是我们想要的组件
    const pathname = (location || route.location).pathname;

    return matchPath(pathname, { path, strict, exact, sensitive }, route.match);
  }

  // 一些警告, 不用管
  componentWillMount() {
    // ...
  }

  componentWillReceiveProps(nextProps, nextContext) {
    // ...
    this.setState({
      // Route 属性变更时, 重新计算路由
      match: this.computeMatch(nextProps, nextContext.router)
    });
  }

  render() {
    const { match } = this.state;
    const { children, component, render } = this.props;
    const { history, route, staticContext } = this.context.router;
    const location = this.props.location || route.location;
    const props = { match, location, history, staticContext };

    // 渲染 Route 的 component 属性
    // match 匹配成功, 则渲染组件, 否则不渲染
    // 此处就说明了 react-router 的逻辑, 当 pathname 和 location.path 匹配成功时, 就去渲染对应的 component
    if (component) return match ? React.createElement(component, props) : null;

    // Route 的 render 属性, 实际上通过 route Object 配置路由的时候, 就是通过该属性渲染的 component
    // 见 [renderRoutes](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-config/modules/renderRoutes.js)
    if (render) return match ? render(props) : null;

    // props: withRoute 中的 routeComponentProps
    // case: withRouter
    if (typeof children === "function") return children(props);

    if (children && !isEmptyChildren(children))
      // router v4 中, route 只允许有一个子组件 
      return React.Children.only(children);

    return null;
  }
}
```

上面代码中, 我们看到它获取了 `childContent` 的 `router` 对象, 然后获取到它的 `location` 值, 这个值我们还记得吗? 就是根据当前浏览器的 `url` 计算得出的一个 `location` 对象, 然后在 `componentWillReceiveProps` 生命周期中, 去执行 `matchPath` 函数, 传入 `location` 值, 得到一个组件状态对象 `match`, 如果 `return null` 则说明匹配失败, 在 `render` 函数中我们看到, `match` 如果为 `null`, 则该组件就不渲染, 否则就说明匹配成功, 渲染该组件.

这里的哲学思想其实 `v4.0` 和 `v3.0` 有很大的不同, 在 `v3.0` 中, 是根据 `location` 去寻找到对应的 `React Compoennt` , 然后渲染. 而 `v4.0` 中, 一切皆组件, 每一个路由都是一个 `Route`, `Route` 也是一个组件, 渲染不渲染完全根据它自己的匹配结果来决定, 也就是说将匹配逻辑放到了组件自身中. 如此一来, 路由规则完全动态了.

## 总结

我们将上述的零碎的东西串联起来:

点击 `Link` 或者 `push(path)` 时调用 `history.push` 去改变浏览器的 `url` 同时返回 `location` 对象, 然后触发 `Router` 中的设置的 `history.listen` 监听函数回调, 回调将 `location` 放到 `getChildContext` 中, 然后每个路由组件 `Route` 都会更新, 在 `componentWillReceiveProps` 中执行匹配函数 `mathPath`, 匹配成功则渲染, 否则不渲染, 这样每次 `Router` 都只会渲染一个 `Route` 组件. 

小小吐槽一下: `v4.0` 这样做, `Route` 中不能再嵌套 `Route`, 版本升级 `api` 变动太大.