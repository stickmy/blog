---
title: 从设计稿的异常看 viewport
date: 2018-06-21 00:37:33
type: post
tag: 适配
meta:
  -
    name: description
    content: 从设计稿的异常看 viewport
  -
    name: keywords
    content: pixelRatio scale viewport
---

### 为什么比设计稿大了一倍？

某日, 我拿到设计稿的时候, 在 `css` 写下了 `width: 200; height: 200`, 运行, 尼玛怎么大了一倍？！！ 然后经过一下午我就写了这篇文章

### 关于像素的几个概念

- 设备像素 (dp)

设备屏幕的物理像素 `device pixel`，简称 `dp`，任何设备的屏幕的物理像素都是不变的，单位为 `pt`

- 屏幕像素密度 (ppi)

指屏幕表面上存在的像素数量 `pixel per inch`，简称 `ppi`， 通过每英寸有多少物理像素来计算, 单位是 `dpi` (`dot per inch`)，不同的设备这个值是不一样的，理论来说，这个值越高越好，每英寸的图像就会更加清晰细腻

```
设 屏幕水平方向有 x 个物理像素，垂直方向有 y 个物理像素
屏幕尺寸: 屏幕对角线的长度，单位英寸

ppi = (√x ^ 2 + y ^ 2) / 屏幕尺寸
```

标准 `ppi` 的值是 160， `ppi` 值超过 `300` 的屏幕被称为超高密度屏幕，苹果叫它 `Retina` 屏。
这里[dpi](https://www.sven.de/dpi/)可以查看更多设备的 `ppi`

- 设备像素比 (dpr)

设备像素比 `device pixel ratio` , 可以通过 `window.devicePixelRatio` 获得

```
dpr = 设备像素 / 设备独立像素
```

在标准 `ppi` 也就是 `ppi = 160` 的情况下，这个值等于 1，也就是一个设备独立像素等于一个设备像素, 但是在如今很多设备都是高 ppi 的情况下，`dpr` 就不一定是 1 了，比如 `iPhone6` 就是 2, `iPhone6s` 就是 3。

- 设备独立像素 (dips)

`device independent pixels` 它其实是一种抽象出来的概念，单位是 `px`，这个名称可能不好理解，我们可以称呼它为 `设备无关像素` 也就是跟设备无关的，是一种抽象概念，简称为 `dips`，可以通过 `screen.width/height` 获得。

在 pc web 端，一般一个设备独立像素等于一个设备像素。在移动端那就不一定了，在移动端，设备的 `dpr` 不是一致的，根据上面的公式，一个设备独立像素等于设备像素除以 `dpr`

- css 像素

没错，css 像素就是设备独立像素，他们与设备像素的区别在于：他们是可拉伸的，而设备像素是固定的，不变的。

### 为什么会大了一倍

设计稿是按 `iPhone6` 来设计的, 而 `iPhone6` 的 `dpr` 为 2, 根据上面的公式，css 像素为物理像素的一半，这就解释了为什么会大了一倍

### viewport

这里有三个概念

- visual viewport

可视 viewport，也就是浏览器的视口

- layout viewport

布局 viewport，顾名思义，也就是我们写代码的时候，布局所使用视口

- ideal viewport

理想视口，通过 `<meta name="viewport" content="width=device-width,initial-scale=1">` 设置，即可获得每个设备的 `ideal viewport`

### scale 是在缩放什么

浏览器的缩放很诡异，这个缩放系数跟 `visual viewport` 的宽度成反比，缩放系数越大，`visual viewport` 宽度越小，所以最小的缩放系数 `minimum-scale` 决定了 `visual viewport` 的最大宽度

我们有时候会发现不同设备上的 `1px` 的粗细不一样，这就是因为他们的 `1px` 所对应的设备像素不一样，要解决这个问题，设置 `scale = 1 / dpr` 即可，事实上也可以根据这个特性来做适配，可以参考淘宝 [flexible](https://github.com/amfe/lib-flexible) 方案

### 吐槽

总体来说这上面的概念还是有点多的，而且容易乱，如果能看完还保持清醒，那应该是搞懂了~
