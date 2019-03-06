---
title: Y-combinator
date: 2018-05-25 23:52:01
type: post
tag: 函数式
meta:
  -
    name: description
    content: y-combinator y 组合子
  -
    name: keywords
    content: y-combinator y 组合子 函数式 fp
---

### 停机问题

我们调试代码的时候, 经常会发现死循环的 bug, 不知道大家有没有尝试过发明一个工具用于自动检查代码里有没有死循环的 bug

答案是: 没有. 我们来看看为什么这么说?

设有这么一个函数可以实现停机判定:

```js
function halting(func, input) {
  return if_func_will_halt_on_input
}
```

我们再构造另一个函数:

```js
function M(func) {
  if(halting(func, func)) {
    for(;;) // 死循环
  }
}
```

接下来, 我们调用它

```js
M(M)
```

我们发现, 出现了悖论, 当 `M` 以 `M` 作为输入时, 到底停不停机. 所以停机问题不可判定.

图灵当时是如何想到这个绝妙证明的? 我们来说说它的同构问题 `Y-combinator`

### 从阶乘函数递归说起

我们写一个阶乘函数, 来计算 `0-n` 的阶乘

```js
function fact(n) {
  return n < 2 ? 1 : n * fact(n - 1)
}
```

我们改写成箭头函数表达

```js
var fact = n => n < 2 ? 1 : n * fact(n - 1)
```

相信大家都写过这种递归, 但是从理论来讲, 这种写法真的没问题吗?

### 问题在哪

我们再来看这个函数, 我们发现这个箭头函数在定义时用到了自身. 一个函数尚未定义完成, 如何引用自身? 没有问题吗? 如果有问题, 那这段代码为什么可以运行?

### 匿名函数如何引用自身

我们知道, 要引用自身, 必须得具名, 这样才能引用自身, 如果函数式匿名的, 那么如何引用自身?

答案是将这个 `lambda` 表达式当作参数传入自身(这里要求这个语言得是函数一等公民语言, 这样函数才能当做值传递)

```js
var fact = (self, n) => n < 2 ? 1 : n * self(self, n - 1)

fact(fact, 5) // => 120
```

ok, 我们现在解决这个问题了, 真的解决了吗? js 引擎是这样做的吗? 这种做法我们发现对定义函数有了特殊的要求, 我们能否定义个工具函数 `F`, 将我们引用自身的匿名函数传给这个 `F`, 会返回一个真正的递归函数. 这样应该才是正确的优化方法.

### 才刚刚开始

```js
var fact = (self, n) => n < 2 ? 1 : n * self(self, n - 1)

var factory = f => n => f(f, n)

fact = factory(fact)

fact(5)
```

好的, 我们现在利用了一层柯里化, 让函数运行的参数优化了, 但是我们的函数定义的问题仍然没有解决

### 不动点

设想我们以某种方式完美定义出了能够在内部自己调用自己的递归函数 `p`, 它的定义如下:

```js
var fact = n => n < 2 ? 1 : n * fact(n - 1)
```

同时有个工具函数 `F` 可将这个 `fact` 转为真正的递归函数:

```js
var F = self => n < 2 ? 1 : self(n - 1)
```

不要问我既然我们已经有了真正的递归函数 `fact`, 还要传给 `F` 干嘛 :). 因为它只是我们的构想, 尚未被真正的定义出来, 我们这样做只是为了引入一个概念 `不动点`

我们现在把 `fact` 当做参数传递给 `F`, 结果是:

```js
F(fact) = n < 2 ? 1 : n * fact(n - 1)
```

发现了吗? `F(f) = f`, 这个就是所谓的 `不动点`. `f` 在 `F` 的作用下结果仍然是 `f`, 也就是说 `f` 在 `F` 作用下是不动的

### Y combinator

设想我们有个函数 `Y`, 可以让伪递归函数 `F` 变成真正的递归函数:

```js
Y(F) = f
```

结合上面的 `F(f) = f`, 我们发现 `Y` 具有这样的性质 `Y(F) = f = F(f) = F(Y(F))`

对于 `f` 有 `f = F(f)`, 但是我们知道 `f` 是无法调用自身的, 还记得我们怎么解决这个问题的吗?

```js
var m = self => F(self(self))

m(m) // 这样调用
```

所以我们的 `Y` 应该生成这样的一个 `f`:

```js
var Y = function (F) {
  var magic = self => F(self(self))
  return magic(magic)
}
```

`Y` 的神秘面纱终于被揭开, 它会求出我们的 `不动点`. 就是这么奇怪的一个东西, 居然可以让我们的伪递归变成真正的递归, 我们看看效果:

```js
var Y = function (F) {
  var magic = self => F(self(self))
  return magic(magic)
}

var F = self => n => n < 2 ? 1 : n * self(n - 1)

var fact = Y(F)

fact(4)
```

我们会发现上面这段代码并不能跑起来, 会栈溢出, 这其实是因为 js 是一门 `call by value` 的语言, 作为实参传进函数中的变量是必须先求值的, 这里我们利用一个延迟求值的技巧 `v => f(x(x))(v)`, 我们利用这个技巧改写一下 `Y`:

```js
var Z = function (F) {
  var magic = self => v => F(self(self))(v)
  return magic(magic)
}
```

这个东西就是 `Z-combinator`, 我们将它改写成函数式:

```js
const Z = f => (x => v => f(x(x))(v))(x => v => f(x(x))(v))
```

`λ` 演算的写法

```lambda
Z = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))
```

我们试试:

```js
const Z = f => (x => v => f(x(x))(v))(x => v => f(x(x))(v))

var F = self => n => n < 2 ? 1 : n * self(n - 1)

var fact = Z(F)

fact(4) // 24
```

是的, 我们成功了, 它是如此让人目眩神迷

### 扩展

其实整个 `Y-combinator` 可以由 `哥德尔不完备定理` 中的一个核心构造式推导而来. 推荐阅读 `刘未鹏` 的这篇文章:[康托尔、哥德尔、图灵——永恒的金色对角线](http://mindhacks.cn/2006/10/15/cantor-godel-turing-an-eternal-golden-diagonal/)
