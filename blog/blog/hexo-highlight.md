---
title: hexo代码高亮的问题
date: 2017-09-01 00:23:34
type: post
tag: hexo
meta:
  - name: description
    content: hexo 代码高亮
  - name: keywords
    content: hexo highlight
---

## 分析
首先，看了一下代码块区域的dom结构
![](https://blog-1252181333.cos.ap-shanghai.myqcloud.com/blog/hexo-highlight-1.png)
发现代码区域的关键字的`class`为`keyword`

然后，发现`highlight`官网给出样式是下图这样的
![](https://blog-1252181333.cos.ap-shanghai.myqcloud.com/blog/hexo-highlight-2.png)

对比发现，我的`class`少了`hljs-`前缀，这样问题就很清晰了

## 思考
这段`markdown`代码是如何转换成`html`格式的呢，只要我们发现了转换的代码，
在其中加上这个前缀，问题不就解决了吗？

## 解决
查询了相关资料，发现了 hexo 的一个 issue : [#434](https://github.com/hexojs/hexo/issues/434)，其中提供这么一段代码`hljs.configure({classPrefix: ''})`，[highlight.js](https://github.com/hexojs/hexo-util/blob/master/lib/highlight.js#L8-L10)，发现它就在`node_modules/hexo-util/lib/highlight.js`下，这段代码可以给我们的 code css 加上一个前缀，那么这个问题就解决了。

当我调试的时候，发现仍然没有高亮，原因是我们并没有引入`highlight.js`的高亮 css 文件，
果然在 head 中引入 css 后，起作用了，代码高亮了，真是令人高兴的消息！

接下来，我又想定制高亮主题，我们可以在`_config.yml`文件中配置主题，从而在 head 中引入相关的 css 文件，这样就可以起到了定制主题的作用。

- **在 css 文件夹中引入 highlight 的全部主题 css 文件**

- **head：**

```
<% if (theme.highlight_theme){ %>
  <link rel="stylesheet" href="<%- config.root %>css/highlight/<%- theme.highlight_theme %>.css">
<% } %>
```

- **_config:**

```
highlight_theme: atom-one-dark
```


## 完~

