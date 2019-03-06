---
title: 前端优化(一)
date: 2018-04-06 14:13:36
type: post
tag:
  - js
  - 优化
meta:
  - name: description
    content: 前端优化
  - name: keywords
    content: 优化 前端优化 前端
---


## 当说前端优化的时候,我们在说什么

随着 `Angular`, `React`, `Vue` 等框架以及 `webpack` 的出现, 前端优化已经变成了怎么细分组件, 怎么利用 `webpack` 做 `code-spliting` 以及利用 `ssr` 做首屏直出的主场, 那么我们今天就从浏览器解析 `html` 的角度来讲一讲前端优化

### script 标签

我们知道在 `html` 中 一个普通的 `script` 标签是按他们在文档中出现的顺序来加载的

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
    <script type="text/javascript" src="example/1.js"></script>
    <script type="text/javascript" src="example/2.js"></script>
  </head>
  <body>

  </body>
</html>
```

下面一段代码, 肯定是 `1.js` 先解析执行完成之后, 才是 `2.js` 再解析执行

我们知道, `html` 是按顺序解析的, 也就是说等 `head` 标签里的内容都下载,解析执行完成之后, 才能呈现页面的内容, 因为浏览器在遇到 `<body>` 标签才会呈现内容, 那么如果一个页面需要很多 `javascript` 的话, 那么浏览器在呈现页面的时候, 肯定会出现明显的延迟, 用户就会看到一大片空白, 为了避免这个问题, 我们一般将 `script` 标签放到 `body` 元素中的最后面, 比如这样:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <!-- 这里放页面内容 -->
    <script type="text/javascript" src="example/1.js"></script>
    <script type="text/javascript" src="example/2.js"></script>
  </body>
</html>
```

#### script 标签属性

**defer:**

标记了这个属性的 `script` 脚本是表明脚本在执行时不会影响页面的构造, 也就是说这个脚本会**立即下载**, 但是会延迟到整个页面都解析完毕后再执行

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
    <script defer="defer" type="text/javascript" src="example/1.js"></script>
    <script defer="defer" type="text/javascript" src="example/2.js"></script>
  </head>
  <body>

  </body>
</html>
```

上面的这段代码, 虽然我们把 `script` 标签放在了 `head` 中, 但是它们并不会立即执行, 而是等到浏览器遇到 `</html>` 标签后再执行, 这两个脚本会先于 `DOMContentLoader` 事件执行, 并且这两个脚本会按照它们出现的先后顺序执行

但是在开发的复杂环境中, 延迟脚本并不一定会按照顺序执行, 也不一定先于 `DOMContentLoader` 事件执行, 因此我们还是将延迟脚本放到页面的底部, 这是最佳的选择

**async:**

标记了这个属性的脚本表明这个脚本会立即下载, 并在下载完成之后立即执行. 但是它们并不会按照出现的顺序来执行, 但是一定会在页面的 `load` 事件之前执行, 那么我们知道, 标记了这个属性的脚本一定要确保没有相互依赖, 且它们不能修改 `dom`, 所以我们可以将一些无关操作 `dom` 的初始化脚本, 标记上这个属性, 放到 `head` 中, 那么这个脚本的下载执行会与文档的加载并行执行(异步)

我们看一下 [阿里云](https://www.aliyun.com) 是怎么应用的:

![](https://blog-1252181333.cossh.myqcloud.com/blog/084821.png)

我们可以看到, 在 `head` 中的脚本几乎都是加上 `async` 标签的

::: tip
标记了这两个属性的脚本都只适用于外部脚本文件
:::

### 浏览器下载资源

**同一个域名下**, `firefox` 可以同时下载两种图片( `chrome` 可以同时下载 4 个 ), 不过只有图片跟 `css` 这样, `javascript` 脚本还是一个接一个下载, 不过我们前面提到的 `async` 属性, 可以发起异步加载

有了这个限制, 我们下载资源的速度明显被限制了, 那么我们怎么解决呢?

我们可以将 `css` 和 `javascript` 脚本放到一个子域名里(比如 `g.example.com`), 而图片放到另一个子域名里(比如 `img.example.com`), 这样的话, 不同的子域名的资源可以被并行下载

那么可能有人想, 我们为每一个资源都分配一个子域名, 那是不是更好呢?

答案是否定的. 因为我们下载资源的过程是一个长连接, 而长连接的保持可能需要服务器在维护, 尤其是 `http` 协议不是那么可靠, 一些无用的长连接, 可能服务器也在维护, 这样的话, 访问的次数越多, 越糟糕. 另外一方面, 解析域名是需要 `dns` 的, 过多的子域名解析, 会导致 `dns` 开销增大.

最佳实践是使用两个子域名: `css` 和 `javascript` 一个, 图片一个

### 其他的方案

1. 合并图片, 利用雪碧图来减少 `http` 请求

2. 如果首页有轮播之类的内容, 可以只加载第一张图片, 等到文档加载完全之后, 用 `js` 动态加载剩余图片

3. 利用 `textarea` 优化, 将一些 `script` 和一些首屏用不到的 `html` 节点放到 `textarea` 中, 监控窗体滑动, 当 `textarea` 出现在视野中的时候, 取出 `textarea` 的值, 动态的插入到 `dom` 中

4. 也有很多其他的优化点, 这里暂时不提, 我会在下一篇中, 利用 `ssr` 以及 `webpack` 的一些技术方案来说说前端优化
