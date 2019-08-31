---
title: 邂逅 Typescript
date: 2018/11/18 23:51:03
type: post
tag:
  - typescript
meta:
  -
    name: description
    content: Typescript impact
  -
    name: keywords
    content: Typescript React Redux
---

## 场景

有如下对象:

```js
const plugins = {
  hover: {
    state: {},
    reducer: () => {},
    component: () => <div>This is hover plugin</div>
  },
  drag: {
    state: {},
    reducer: () => {},
    component: () => <div>This is drag plugin</div>
  }
}
```

这是一个插件系统, 我们有如上两个插件, 第一个用来当我们 hover 在元素上的时候, 显示一些 hover 样式, 第二个用来当我们拖拽该元素的时候, 显示一些拖拽的工具栏.

可以看出每个 plugin 是由一个 React Component 和 初始的 state 以及 一个对应的 reducer 组成, 所以这里应该可以看出整个的插件系统的设计, 把每个 plugin 中的 state 组成一个总的 state, 每个 reducer 组成一个 combined reducer, 然后有一个总的 Provider 接受这个 redux context, 分发 props 给每个 plugin 中的 component 组成的 `this.props.children`.

## 第一步, 写一个 helper 函数

我们需要一个能将每个 plugin 中的属性 (state, component, reducer) 抽离出来的函数 `extractProperty`. 我们希望得到的结果是这样的:

```js
const combinedState = extractProperty(plugins, 'state')

combinedState = {
  hover: hoverState,
  drag: dragState
}
```

先写出 js 实现:

```js
const extractProperty = (plugins, propertyName) => {
	return (Object.keys(plugins).reduce((map, key) => {
		map[key] = plugins[key][propertyName];
		return map;
	}, {});
};
```

## 改造成 typescript 版本

首先我们给单个 plugin 一个类型定义:

```ts
export type DeepPartial<T> = { [K in keyof T]?: DeepPartial<T[K]> }

export interface Plugin<S = any> {
	state?: DeepPartial<S>;
	reducer?: Reducer<S>;
	component?: ComponentType<any>;
}
```

给出 plugins 类型定义:

```ts
export type PluginMapObject<S, K extends keyof S> = { [T in K]: Plugin<S[T]> };
```

用目前给出的两个定义改造一下之前的 helper 函数:

```ts
const extractProperty = <S, K extends keyof Plugin>(
	plugins: PluginMapObject<S, keyof S>,
	propertyName: K
): any => {
	return (Object.keys(plugins).reduce((map, key) => {
		map[key] = plugins[key][propertyName];
		return map;
	}, {});
};
```

这还不够, 经过这个 helper 函数抽取出来的数据结构也是具有它的类型的, 我们尝试给出它的类型

下面给出函数 return 的数据类型定义, 我们利用 S[K] 构造出 Plugin 的泛型, 配合 plugin 的 key 给出return 数据的类型定义

```ts
export type PluginPropertyMap<
  S,
  P extends keyof Plugin,
  T extends keyof S
> = {
	[K in T]: Plugin<S[K]>[P]
};
```

最终版本:

```ts
const extractProperty = <S, K extends keyof Plugin>(
	plugins: PluginMapObject<S, keyof S>,
	propertyName: K
): PluginPropertyMap<S, K, keyof S> => {
	return (Object.keys(plugins).reduce((map, key) => {
		map[key] = plugins[key][propertyName];
		return map;
	}, {}) as any) as PluginPropertyMap<S, K, keyof S>;
};
```

### 未完待续...