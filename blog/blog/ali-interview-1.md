---
title: 鏖战阿里(一):国际 UED
date: 2018-08-12 14:20:43
type: post
tag: 面试
meta:
  -
    name: description
    content: 阿里面试 面经
  -
    name: keywords
    content: 面经 面试 阿里面试 阿里
---

写这篇文章记录总结一下拿到阿里 offer 的历程, 一共持续快两个月, 当然从标题看来, 应该知道这一次国际 UED 是挂了的(笑).
<!-- more -->

### 介绍

大概说一下发生的背景, 本科工作两年, 2个月前的一天, 在某 Boss 上, 阿里国际 UED 的面试官要了我的简历, 阿里面试之旅就算正式开始了.

### 电面一轮

1. **说说你了解的跨域解决方案**

说了主流的几种, 面试官着重问了关于 CORS 的, 比如如何触发 options 请求, 就大概说了说, 答案可以戳这 [MDN-CORS](https://developer.mozilla.org/zh/docs/Web/HTTP/Access_control_CORS)

2. **XSS 攻击与 CSRF 攻击**

这个我之前也有系统的了解过, 就说了说他们的联系与区别, 以及还扩展到了一些中间人攻击方式.

3. **对 `ES6` 了解如何? `let const var` 区别?**

这个就说了变量的几个特性, 以及 `let` 的暂时性死区特征

4. **react `setState` 的同步异步, 什么时候是同步的, 什么时候是异步的?**

这一块之前看过文章了解过, 主要是取决于 `batchingStrategy.isBatchingUpdates` 这个变量, 然后面试官问了一个问题: 

> 在 window 上绑定一个原生的事件监听函数, 在这个函数中去执行 setState, 是同步还是异步?

这个问题, 当时也算是答出来了, 但是答的比较模糊, 自己了解的也不够透彻, 当然这个问题会在系列二中的淘宝面试中重新出现, 想了解的可以戳我的语雀 [setState](https://www.yuque.com/stickmyc/react-analysis/xeo8tr), 以及一个 `demo`:

[![Edit react-setState](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/532qkn52kn)

5. **react 16 的 fiber 了解吗, 用来解决什么问题的? 对代码有什么影响?**

这个之前在知乎也看过相关的介绍, 戳这 [程墨Morgan的专栏](https://zhuanlan.zhihu.com/p/26027085)

6. **对 `css` 了解如何? 问了一个关于 BFC 的问题: 兄弟元素的 margin 塌陷**

这个因为之前也有系统的了解过, 就说了说触发 BFC 的几种方法.

7. **说一下原型链，对象，构造函数之间的一些联系**

这个相关的文章, 书都有看过不少了. 也有一些自己的理解, 所以答起来还是很轻松的


整个过程大概 40 分钟, 总体来说还是很顺利的.

### 二面笔试

隔了一天, 有面试官联系我笔试时间, 说到时候会有一封邮件发过来. 约在了当时的明天晚上 8 点.

大概到了第二天晚上 8:15 的样子, 有一封邮件到了, 是一个阿里的伯乐系统的链接, 点进去是一个写代码的界面, 一共三道题, 可以开视频, 语音(不过我没开), 还有一个界面可以随时跟面试官交流, (不要妄想百度搜索之类, 据说这个网页有监控, 如果你离开了这个网页的页面, 会有提示, 在面试官眼中是不诚信的表现了, 基本被挂). 

1. 第一题: **数组去重**

这个题还是比较简单的, 一开始写了 `es5` 和 `es6` 两种方法:

```js
[...new Set(array)]
```

```js
function removeDuplicateEs5(array) {
	return array.filter((item, index) => {
    return index === array.indexOf(item)
  })
}
```

面试官问: 第二种解法复杂度多少, 答曰: O(n^2), 问: 有没有复杂度更低的?

这个其实只要创建一个对象存储就好啦, 如下:

```js
function main(array) {
  var hash = {};
  
  return array.filter((item, index, array) => {
    return hash.hasOwnProperty(typeof item + item) 
      ? false 
      : (hash[typeof item + item] = true)
  })
}
```

注意类型的问题, 比如 `[1, '1']` 这样的.

2. 第二题: **对象的 key 从横杠模式 (Pascal) 转到驼峰 (Camel)**

这个题, 你当然可以用循环之类的实现, 不过很明显, 面试官是考正则的解法, 如下:

```js
function convertPascal(object) {
  var newObject = {}
  
	const convert = key => key.replace(/\_(\w)/g, (all, t) => {
    return t.toUpperCase()
  });

	Object.keys(object).forEach(key => {
    const newKey = convert(key);
    newObject[newKey] = object[key];
  })
    
  return newObject;
}
```

3. 第三题: **实现类的继承**

一开始写了 `es6` 的继承, 面试官要求用 `es5` 实现, 代码:

```js
var Superman = function() {};
var Sub = function() {};

Sub.prototype = Object.create(Superman.prototype, {
  constructor: Sub
})
```

写了个简易版的, 当然也还有寄生继承, 就是多了要实例化父类属性的操作, 可以自行去看对应的写法

二面, 总体难度不大, 都是些基础的东西, 基础扎实的话, 应该没有问题

### 三轮电面

这次的面试主要是项目相关. **比如现在团队的打包发布过程是怎么样的, 有什么了解或者改进吗?**

刚好团队刚发布了新的打包系统, 结合了现在的一些情况, 说了一些.

然后还问了几个之前面试没答出来的问题

::: tip 友情提示
所以, 同学们, 一定要记得要去看面试没答出来的问题啊!! 如果还是没答出来, 很减分.
:::

### 四轮电面

这次的面试官是国际 UED 部门的 leader, 问的问题既有项目也有技术相关, 更广度一些,

**说说你现在负责的项目**

主要说了一些现在在写小程序, 微信跟支付宝都有写, 包括 app, M 站的东西.

**然后面试官问了: 两个小程序有什么区别?**

我说了一些写法上, debug 上, 以及提到了微信小程序的一个 `dsl`: `mpvue`, 然后面试官问了解支付宝小程序的 `dsl` 吗? 其实是有的 `taro`, 不过我不熟悉, 所以回答了没有了解.

**问: 项目中怎么统计 pv uv?**

回答了埋点的方式, 通过在页面中的每个需要埋点的可点击元素中发送请求, 不过好像不是面试官要的答案, 像埋点 sdk 一类的用的也比较少, 所以这个问题答得不好

**问: React Router 了解吗, 说说 v3 和 v4 的区别**

这个我还是看过的, 大概说了一些区别, 然后面试官问了 `switch` 是干嘛的, 解决什么问题, 大概说了一下, 没有什么问题

**问: pwa 熟悉吗?**

自己也算实践过了一些, 说了一些 pwa 可缓存, 推送等特性.

问: 推送如何实现的?

答: 卒, 并不了解

**问: 前端性能**

我说了服务端渲染, 但是其实我对这个并不是很熟, 实践的很少, 所以面试官问了几个渲染完成的标准, 如何优化, 都答得不好

::: tip 提示
不熟的东西, 千万不要说啊!!!
:::

大概 30 分钟就结束了, 面完之后觉得要挂, 果然一般面试者的直觉都是很准的, 华丽丽挂掉了. 

评语是: 业务相关的稍弱.

#### 总结

自己对项目业务线方面掌握的不够全局, 也问到了一些薄弱点. 这次收获还是很大的, 也可以让自己查缺补漏.
