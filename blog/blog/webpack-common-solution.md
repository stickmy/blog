---
title: 记录 webpack 打包后的两个问题
date: 2018-03-12 11:52:47
type: post
tag: webpack
meta:
  - name: description
    content: webpack 常见问题
  - name: keywords
    content: webpack 前端
---

### 打包后访问不到静态资源目录

原因: 打包后以相对路径去访问静态资源目录, 导致没有找到资源

解决方案: 
`index.html`中加入下面这一行

```html
<base href="/" />
```

<!-- more -->

> HTML <base> 元素 指定用于一个文档中包含的所有相对URL的基本URL。一份中只能有一个<base>元素。 --- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/base)

### 使用 browserHistory 时, 直接访问子路由 404

原因: 当直接访问或者刷新子路由时, 浏览器会向服务器请求 `example.com/child`, 服务器
会去根目录下找 `child.html` 文件, 但是我们服务器实际上并没有这样的物理路径文件, 因为
我们的内容都是通过 `React-Router` 渲染出来的, 所以报了 `404 error`

解决方案:
这里给出一个 `nginx` 的方案:
既然我们知道了出错的原因, 自然而然能想到一个方案, 就是让所有的请求都指向 `index.html`, 
子路由转交给 `React-Router` 处理

```nginx
location / {
    try_files $uri /index.html;
}
```

当然也有其他的一些解决方案, 比如使用 `webpack-dev-server` 的服务器, 加上 `–history-api-fallback` 参数.

使用 `Node` 服务器的也有对应解决方案, 这里也不做赘述了.

