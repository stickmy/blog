---
title: Hera(一) 奇怪的诞生方式, 真💧🌟
date: 2020-05-24 14:27:26
type: post
tag: Hera
meta:
  - name: keywords
    content: Hera
---

> 不会还有人不会写一个简单的编程语言吧？不会吧？不会吧？

好了开个玩笑，看下 HeraScript 是通过怎么一个奇怪的方式诞生的🐮

疫情在家无聊的时候, 看着 haskell 的 parsec，突然想到 "不知道 js 有没有类似的 parse combinator"， 上 github 搜了一下，发现有几个，不过都不是很好用，于是就照着 haskell parsec 的实现写了一个 js 版本的。实现完了之后，想写个 example 来验证一下，于是 HeraScript 就诞生了。可以在 [playground](https://herascript.github.io/index.html?name=basic-syntax) 试玩一下 👀

Hera 是编译到 js 执行的，有人问：那它跟 js 有什么区别啊？Hera 加了很多有的没的特性(也不用考虑其他因素，因为它本质上是个我练习的产物，好玩就行，能给到大家一些想法就更好了😐