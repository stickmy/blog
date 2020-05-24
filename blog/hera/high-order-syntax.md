---
title: Hera(三) 进阶语法
date: 2020-05-24 16:48:19
type: post
tag: Hera
meta:
  - name: keywords
    content: Hera
---

::: tip
本文的示例代码都可以在 [https://herascript.github.io/index.html?name=user-defined-operator](https://herascript.github.io/index.html?name=user-defined-operator) 以及 [https://herascript.github.io/index.html?name=native-directive](https://herascript.github.io/index.html?name=native-directive) 找到
:::

这篇文章主要讲两个语法 `user-defined-operator` 以及 `native directive`

## user-defined-operator

在 Hera 中，允许自定义操作符，比如

```js
infixl 4 @ -> xs @ x => xs[x];
[1,5,3,4] @ 2 - 1; // [1,5,3,4][2-1] = 5
```

我们定义了一个叫做 `@` 的 operator，作用是 access array
自定义操作符必须要处理的问题是操作符之间的优先级，比如 2 + 3 - 2, 先运算 `3 - 2` 是因为 \* 的优先级比 + 高

在 Hera 中，数字越小，优先级越高。`@` 的优先级是 4，Hera 中 `-` 的优先级是 2, 所以这里先运算 `-`，得到表达式 `[1, 5, 3, 4][2 - 1]`

自定义操作符的语法是:
`[notation] [priority] [op] -> [hera expression] => [native expression]`

notation: postfix | prefix | infix | infixl | infixr
分为 前缀、后缀、中缀声明, 中缀声明又分为左联、右联

priority: 操作符的优先级, 数字越小, 优先级越高

op: 需要自定义的操作符, 在这个例子中是 `@`

hera expression: 操作符在 hera 中的使用方式

native expression: 此操作符对应的原生表达式, 我们是编译到 JavaScript, 所以这里是 js 的表达式

在 Hera 中所有操作符都是通过这种语法定义的（包括内置的操作符: x - \* / ++ -- 等等, 它们都是由 stdlib 定义的）

## native directive

如果在 Hera 中有语言满足不了的情况下，可以通过 `native` 关键字去使用 js expression

比如定义一个 print

```js
function print() {
	native "console.log(...arguments)";
}
```

比如定义一个 foldl

```js
// 定义 foldl
function foldl(f, x, xs) {
	return native "xs.reduce(f, x)";
}

// Sum
foldl((+), 0, [1, 2, 3]); // 6
```
