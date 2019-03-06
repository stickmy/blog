---
title: 词法作用域
date: 2017-10-21 18:26:05
type: post
tag: js
meta:
  - name: description
    content: 词法作用域 动态作用域
  - name: keywords
    content: 词法作用域 动态作用域
---

### 作用域是什么
> 作用域(scope)是指名字(name)与实体(entity)的绑定(binding)保持有效的那部分计算机程序 [wiki](https://zh.wikipedia.org/wiki/%E4%BD%9C%E7%94%A8%E5%9F%9F)

<!-- more -->

**作用域规定了如何查找变量**

### 词法作用域与动态作用域

- 词法作用域的函数中遇到非形参非局部变量的时候，去函数定义的env中查询

- 动态作用域的函数中遇到非形参非局部变量的时候，去函数调用的环境中查询

JS采用的是词法作用域(静态作用域)

举个栗子：
```javascript
var value = 1

function foo() {
    console.log(value)
}

function bar() {
    var value = 2
    foo()
}

bar() // 1
```

1. 采用词法作用域
执行 `foo` 函数，`foo` 函数内部没有查找到局部变量 value，则在定义函数的环境中查找，value = 1，所以打印 1

2. 采用动态作用域(`bash`)
执行 `foo` 函数，`foo` 函数内部没有查找到局部变量 value，则从函数调用的作用域中查找，value = 2，所以打印 2