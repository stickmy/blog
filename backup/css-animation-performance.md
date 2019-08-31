---
title: css 动画性能浅谈
date: 2018-03-31 00:28:16
type: post
tag: css
meta:
  - name: description
    content: 动画性能 css3
  - name: keywords
    content: css css3 动画性能 回流 重绘 前端
---

## 场景

图中的两种动画有什么区别

![](https://blog-1252181333.cossh.myqcloud.com/blog/css-anmiation-carbon.png)

## 分析

第一种动画并不会动, 因为 `top`, `left` 值并没有发生变化

第二种动画会动, 因为 `translate` 是计算偏移值

那如果我们把第一种动画改成下面这样

```css
@keyframes move {
    50% {
        top: 10px;
        left: 30px;
    }
}
```

**现在这两种动画还有什么区别呢? 哪个性能好?**

我们先了解两个概念,`回流(reflow)` `重绘(repaint)`

`reflow`: 当元素的几何属性(`height`, `width` 等等)发生变化时, 相关元素都会重新渲染, 这个过程叫做回流

`repaint`: 当元素只需要更新样式属性, 而不影响布局的时候(比如换个背景色), 页面只会重绘这单个元素, 这个过程叫做重绘

那么我们看上面的场景, 第一种动画, 在发生回流操作, 第二种动画只是在重绘, 当然这个场景, 元素是绝对定位, 只会引起自己的重绘, 不会引起页面其他部分重新布局.

当我们设计一个动画的时候, 最好使用 `transform` 去实现, 因为 `transform` 不会发生回流, 我们还可以为此开启硬件加速, 当发生 `3D` 转换的时候, 浏览器会开启硬件加速, 我们可以为此做一个小 trick

```css
@keyframes move {
    50% {
        transform: translate(100px, 300px, 0)
        /* 引起 3d 变换, 开启硬件加速 */
    }
}
```

::: tip
硬件加速的开启也是比较耗性能的, 移动端尤其明显, 当我们开启硬件加速的时候, 最好为元素人为加上较高的 `z-index` 属性, 原因见[https://w3ctrain.com/2015/12/15/smoother-animation/](https://w3ctrain.com/2015/12/15/smoother-animation/)
:::
