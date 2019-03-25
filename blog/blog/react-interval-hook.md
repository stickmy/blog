---
id: react-interval-hook
title: 如何实现一个 interval hook
date: 2019-03-23 14:00
---

> 原文: https://overreacted.io/making-setinterval-declarative-with-react-hooks/

```js
useInterval(() => {
  // ...
}, 1000);
```

## 有了自带的 setInterval 为何还要再实现一个

> its arguments are “dynamic”

可以注意到我们的 setInterval 是接受一个 dealy 值的, 并且这个值是可以由我们的代码控制的, 这意味着我们可以随时调整这个值来做动态的改变.

大概像这样:

<iframe src="https://codesandbox.io/embed/znjqq2ry1x?autoresize=1&fontsize=14&hidenavigation=1&view=preview" style="width:100%; height:240px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

## class component 实现

```js {13-27}
class Counter extends React.Component {
  state = {
    count: 0,
    delay: 1000
  };

  tick = () => {
    this.setState({
      count: this.state.count + 1
    });
  };

  componentDidMount() {
    this.interval = setInterval(this.tick, this.state.delay);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  componentDidUpdate(prevProps, prevState) {
    // delay 变化， 重置定时器
    if (prevState.delay !== this.state.delay) {
      clearInterval(this.interval);
      this.interval = setInterval(this.tick, this.state.delay);
    }
  }

  handleDelayChange = e => {
    this.setState({
      delay: Number(e.target.value)
    });
  };

  render() {
    const { delay, count } = this.state;
    return [
      <h1>{count}</h1>,
      <input value={delay} onChange={this.handleDelayChange} />
    ];
  }
}
```

## attempt 1

```js
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  });
  return <h1>{count}</h1>;
}
```

我们一开始一般会写出这样的实现, useEffect 设置 interval, return cleanup. 然而这样写会有个奇怪的表现...

react 在默认在每次 render 之后会重新执行 effects, 这其实也是 react 所预期的, 因为这样能避免 [a whole class of bugs](https://reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update).

我们通常会使用 effect 来订阅, 退订一些 api, 但是在 setInterval 上使用的时候就会有问题, 因为执行 `clearInterval` 和 `setInterval` 是有时间差的, 当 react 渲染过于频繁的时候, 就会出现 interval 压根没机会执行的情况!

```js
setInterval(() => {
  ReactDOM.render(<Counter />, rootElement);
}, 100);
```

<iframe src="https://codesandbox.io/embed/myv8w4438?autoresize=1&fontsize=14&hidenavigation=1&view=preview" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>


## attempt 2

在上一个阶段中, 我们的问题是重复执行 effects 导致了 interval 被清理的太早.

我们知道 useEffect 可以传入一个参数来决定是否重复执行 effects, 试一下

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

<iframe src="https://codesandbox.io/embed/lxpkjo54m9?autoresize=1&fontsize=14&hidenavigation=1&view=preview" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

好的, 现在我们 counter 更新到 1 就停止了

发生了什么?!

这其实是个很常见的闭包问题, 也有了对应的 [lint](https://github.com/facebook/react/pull/14636).

我们的 effects 现在只会运行一次, 所以 effects 每次捕获的 count 值都是第一次 render 的 count 值(0), 所以 `count + 1` 一直是 1

有一种 fix 的方式是, 用 setState 的函数参数, `setCount(count => count + 1)`, 这样我们就可以读取最新的 state, 但是这种方式不是万能的, 比如不能读取最新的 props


## 使用 Refs

我们回到上个问题, count 无法被正确读取的原因是 count 的值一直引用的是第一次 render 的.

**那如果我们在每次 render 的时候动态地改变 `setInterval(fn, delay)` 中 fn 函数, 使这个函数带上最新的 props 和 state, 并且这个 fn 函数要能在多次 render 之间可持续（persist）, 这样 setInterval 执行的时候, 就可以实时的读取这个函数拿到最新的值**

第一版实现:

```js
function setInterval(callback) {
	const savedCallback = useRef();

	useEffect(() => {
		savedCallback.current = callback;
	});

	useEffect(() => {
		// 每次运行当前 ref 最新的 callback
		// 不要用赋值语句 (tick = savedCallback.current)，否则仍然是之前的引用
		const tick = () => savedCallback.current();
		const id = setInterval(tick, 1000);
		return () => clearInterval(id);
	}, []);
}
```

<iframe src="https://codesandbox.io/embed/oj86zzryj9?autoresize=1&fontsize=14&hidenavigation=1&view=preview" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

支持动态 `delay` 和 `暂停` 的最终版:

```js
function setInterval(callback, delay) {
	const savedCallback = useRef();

	useEffect(() => {
		savedCallback.current = callback;
	});

	useEffect(() => {
		// 每次运行当前 ref 最新的 callback
		// 不要用赋值语句 (tick = savedCallback.current)，否则仍然是之前的引用
		const tick = () => savedCallback.current();
		if (delay !== undefined) {
			const id = setInterval(tick, delay);
			return () => clearInterval(id);
		}
	}, [delay]);
}
```

我们可以用这个 hook 做一些更加好玩的事 -- 用一个 interval 控制另一个 interval 的速度

<iframe src="https://codesandbox.io/embed/72z7p6qqv6?autoresize=1&fontsize=14&hidenavigation=1&view=preview" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
