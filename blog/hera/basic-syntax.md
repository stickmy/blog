---
title: Hera(二) 基础语法
date: 2020-05-24 15:32:06
type: post
tag: Hera
meta:
  - name: keywords
    content: Hera
---

::: tip
本文的示例代码都可以在 [https://herascript.github.io/index.html?name=basic-syntax](https://herascript.github.io/index.html?name=basic-syntax) 找到
:::

## statement

在 Hera 中，所有的语句都必须以 `;` 结尾

## comment

```js
// Single comment

/**
 * Multiline comment
 * /
```

## variable

Hera 有两个关键字来定义变量，`var` 和 `val`(`var final`的意思)，var 编译到 js 的 var，val 编译到 js 的 const

不同的是如果你需要给一个可变量重新赋值，需要用 `:=` 操作符

```js
var x = "Hello Hera";
x := "Hi";
```

## Number

与大多数语言类似

```js
val a = 123;
val b = 0.123;
val c = 1e2;
val d = 1e-2;
val e = 0.1e-2;
val e1 = 0x11;
```

## Array

跟 js 一样的定义数组的语法 `val x = [1,2,3];`
但是 access array 的语法有些区别，用 `[array] !! [cursor]`，比如 `x !! 1`

## function

Hera 中支持普通 function 和 箭头函数, 以及 operator as function，分别举三个例子

```js
function Hello() {
	return "Hello";
}

val f = x => {
	return "Arrow function";
}

val m = (+);
m(2, 3);
```

## Partial function

在 js 中，我们需要偏函数的时候，往往需要先对其柯里化，比如

```js
function add() {
	return x + y;
}
// 对 add 柯里化
function addCurry(x) {
	return y => x + y;
}

const add2 = addCurry(2);
add2(3); // 2 + 3 = 5
```

在 Hera 中，我们是不需要这样操作的

```js
function add(x, y) {
	return x + y;
}
val add2 = add(2);
add2(3); // 3 + 2 = 5

function sub(x, y) {
	return x - y;
}
val sub2 = sub(, 2);
sub2(5); // 5 - 2 = 3
```

## operator as function

在 Hera 中，我们可以像调用 function 一样，调用一个 operator

```js
(+)(2, 3);
(* 2)(3);
(2 *)(3);
(+ (2 * 3))(4); // 2 * 3 + 4 = 10
```

## if

跟 js 中的语法一样，支持 `if`, `else if`, `else`

## for

跟 js 的语法差不多, 不多说

```js
var m = 0;
for (var i = 0;i < 10; i++) {
	m := m + i;
}
```
