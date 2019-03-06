---
title: 闭包
date: 2017-10-22 17:14:37
type: post
tag: js
meta:
  - name: description
    content: 闭包
  - name: keywords
    content: Closure js 闭包 前端
---

### 什么是闭包
闭包(Closure)在计算机科学中其实是个很普通的概念，大家不要想得过于复杂

**闭包**
> 闭包就是指能够访问自由变量的函数

<!-- more -->
**自由变量**
> 自由变量是指函数中使用的既不是函数参数也不是函数局部变量的变量

可能有点拗口，举个栗子 :chestnut: ：
```js
var a = 1
function b() {
    console.log(a) // 此处的 a 就是一个自由变量
}
// 此处 b 是一个闭包
```

再来一个栗子：
```javascript
function foo(x, y) {
    console.log(x, y) //此处的 x y 相对于 foo 都不是自由变量 
    function bar() {
        console.log(x, y) // 此处的 x y 相对于 bar 都是自由变量
    }
    // 此处 bar 已经形成闭包，无论被 return 与否
}
```

### JS闭包是词法作用域的体现

怎么理解？举个栗子：
```javascript
var t = []
for(var i = 0; i < 3; i++) {
    t[i] = function() {
        console.log(i)
    }
}

t[0]() // 3
t[1]() // 3
t[2]() // 3
```
当 for 循环执行完毕的时候，此时全局上下文中 i = 3。
当开始执行 `t[0]` 的时候，查找 `t[0]` 定义的时候的作用域中并没有 i 值，于是向上一级也就是全局作用域查找，i = 3，所以结果为 3。
`t[1]` `t[2]` 同理

改成闭包试试：
```javascript
var t = []
for(var i = 0; i < 3; i++) {
    t[i] = (function(i) {
        return function() {
            console.log(i)
        }
    })(i)
}

t[0]() // 0
t[1]() // 1
t[2]() // 2
```
当 for 循环执行完毕的时候，全局上下文中 i = 3
当开始执行 `t[0]` 的时候，`t[0]` 的作用域链为：
```javascript
// t[0] 作用域链
t[0]Context = {
    Scope: [AO, anonymousContext.AO, globalContext.AO]
}
// 匿名函数上下文
anonymousContext = {
    AO: {
        arguments: {
            0: 0,
            length: 1
        },
        i: 0
    }
}
```
`t[0]` 执行时，自己作用域并没有查询到 i 值，向上一级也就是匿名函数查询，查到 i = 0，于是打印了 0。
`t[1]` `t[2]` 同理，大家可以自己推敲一下

### 总结
从上面的例子可以看出，**闭包**就是函数可以记住并访问函数所在的词法作用域，并且保持着对词法作用域的引用
