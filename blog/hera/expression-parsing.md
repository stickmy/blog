---
title: Expression Parsing
date: 2020-07-11 16:32:06
type: post
tag: Compiler
meta:
  - name: keywords
    content: expression parser
---

如果说有这样一个表达式：`3 + 5 * 2` 它是如何被解析的呢？

## 几条规则

1. 操作符是有优先级的，比如可以规定 * 优先级比 + 高，那么 `5 * 2` 就得先被解释执行
2. 操作符是分 postfix prefix infix 几种的，infix 又分 left assoc 和 right assoc，它们都有不同的解析规则
3. 我们说连接 term 的是 op，在 op 左右的是 term，每个 term 可以是个 pure value，也可以是个 expression，不熟悉的可以先写下 BNF 就会清晰很多了。这就意味着解析的过程中会有很多的递归操作

## 操作符的优先级定义

操作符有两个性质:

1. 具有优先级
2. 有不同类型: `prefix`、`postfix、infix`、`left assoc infix`、`right assoc infix`

我们采取如下数据结构：

```ts
type OpTable = Op[];

class Op {
	prefix: OpParser[];
	postfix: OpParser[];
	infix: OpParser[];
	infixl: OpParser[];
	infixr: OpParser[];
}
```

用操作符在 OpTable 中的先后顺序来代表优先级的高低，同一优先级的操作符又分为不同的类型。我们用这样的数据结构就可以表达出上面说的这两种性质

## 解析方式

由于操作符是分优先级的，所以我们要先解析优先级高的：

```ts
OpTable.reduce(makeParser, simpleExpr);
```

核心算法是 makeParser 这个函数，至于 simpleExpr 是什么？我们放到最后说

这里我们先介绍几个函数：

1. `choice`: 它接受一个 parser 数组，按顺序解析，直到其中一个 parser 解析成功就停止解析
2. `or`: 它是 `choice` 的展开写法，`choice(parserArray) == or(...parserArray)`
3. `pure`: 用于将一个值包装进 parser 中，相当于一个 lift 的作用

### signed term parsing

```ts
const prefixOp = choice(ops.prefix);
const postfixOp = choice(ops.postfix);

const postfixP = or(postfixOp, pure((x: T) => x));
const prefixP = or(prefixOp, pure((x: T) => x));
```

前缀、后缀操作符的定义就是上面这样，它们的作用主要是为 term 带上前缀、后缀的解析

```ts
const termP = seq(m => {
	const pre = m(prefixP);
	const x = m(term);
	const post = m(postfixP);
	if (m.success) {
		return post(pre(x));
	}
});
```

### infix op parsing

中缀操作符解析中难点在于有左联、右联的区分，举个例子

`2 * 5 + 3 + 6` 这个表达式

- __如果 * 是右联的，那么我们的解析顺序是：__

1. 2 * expr
2. expr = 5 + expr
3. expr = 3 + 6

即先把 * 右边的当成一个 expr，然后递归解析这个 expr

- __如果 * 是左联的，那么我们的解析顺序是：__

1. x = 2 * 5
2. y = x + 3
3. y + 6

即碰到一个左联的操作符，优先将这个操作符的左值和右值进行计算得到一个值：x，然后再将这个 x 当做 `下一个 expr 的左值` 带入到递归解析中

我们总结一下上面的两个规则，写成代码：

```ts
function rassocP1(x: T) {
	return or(rassocP(x), pure(x));
}

function lassocP1(x: T): Parser<T> {
	return or(lassocP(x), pure(x));
}

// Right Assoc Parsing
function rassocP(x: T) { // x 为左值
	return seq(m => {
		const f = m(rassocOp); // op
		// y 为右值, 由于是右联的操作符, 所以右值的计算需要当成一个 expr 递归解析
		const y: T = m(
			seq(s => {
				const z = s(termP);
				return s(rassocP1(z));
			})
		);
		if (m.success) {
			return f(x, y);
		}
	});
}

// Left Assoc Parsing
function lassocP(x: T) { // x 为左值
	return seq(m => {
		const f = m(lassocOp); // op
		const y = m(termP); // y 为 右值
		if (m.success) {
			// 由于是左联, f(x, y) 当成下一个表达式的左值再带入递归解析
			return m(lassocP1(f(x, y)));
		}
	});
}

// None Assoc Parsing
function nassocP(x: T): Parser<T> {
	return seq(m => {
		const f = m(nassocOp);
		const y = m(termP);
		if (m.success) {
			return m(pure(f(x, y)));
		}
});
```

完善一下 makeParser 这个函数

```ts
function makeParser(term: Parser, ops: Op) {
	// 上面的几个 parser
	return seq(m => {
		const x = m(termP);
		if (m.success) {
			return m(or(rassocP(x), lassocP(x), nassocP(x), pure(x)));
		}
	});
}
```

## 最后再说一下 simpleExpr 是什么？

就是在一个语言中，它的 expression 由什么组成，比如在 JavaScript 中，expr 可以是 function apply、arrayLiteral、operator apply 等等