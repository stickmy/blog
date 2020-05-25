---
title: Hera(四) parsec (扫盲文)
date: 2020-05-25 19:32:06
type: post
tag: Hera
meta:
  - name: keywords
    content: Hera
---

在 [https://blog.stickmy.me/hera/create](https://blog.stickmy.me/hera/create) 中提到一开始我是想写一个 js parsec 的，Hera 是它的衍生品，今天就来说下这个 parsec，它的实现都是按照 Haskell parsec 来实现的，思路一样，但是很多地方实现的方式不一样（主要是受限于 JavaScript）

## parsec 是什么

> parsec 是个工程化的产物，他并没有什么先进的 parse 手段，只是一个实现 parser 的技巧，并且也存在一些缺点(比如不支持左递归)

在一般的 parser 中，我们大多数时候用递归下降去手写一个 parser，就会去构造状态机，那么就会有很多的状态需要管理，代码里就出现了一堆 switch case：

```ts
switch (state) {
	let next;
	case State.Content:
		break;
	case State.SingleCommentLine:
		// do something
		next = State.Content;
		break;
	case State.Brace:
		// do something
		next = State.BodyStatement;
		break;
	// case xxx;
	// case yyy;
}
```

那么 parse combinator 解决了这个问题，它提供了很多的小的组合子，而基于这些小的组合子就可以写出更加强大的 parser

## 几个例子

```ts
lexer.parens(Parsec.sepBy(lexer.identifier, lexer.comma));
```

这行代码就可以解析出函数中的参数列表

1. `lexer.parens` 解析 `()`
2. `sepBy` 组合子接受两个 parser 作为参数，我们这里叫做 p 和 sep, p 解析有效 token，sep 解析分隔符 token，然后有效内容与分隔符依次交替解析，直到解析完有效内容之后无法继续解析到分隔符为止。比如上面的例子中就是解析 `token ,` 这样的循环字符串

在 Haskell 中有 `do` 这样的 operator，我也实现了类似功能的函数，叫做 seq，不过写起来没有 Haskell 这么爽

比如 JSON parser 的一个片段

```haskell
p_text :: CharParser () JValue
p_text = spaces *> text
    <?> "JSON text"
    where text = JObject <$> p_object
              <|> JArray <$> p_array

p_series :: Char -> CharParser () a -> Char -> CharParser () [a]
p_series left parser right =
	between (char left <* spaces) (char right) $
					(parser <* spaces) `sepBy` (char ',' <* spaces)

p_array :: CharParser () (JAry JValue)
p_array = JAry <$> p_series '[' p_value ']'

p_object :: CharParser () (JObj JValue)
p_object = JObj <$> p_series '{' p_field '}'
    -- 产生 name, value 对
    where p_field = (,) <$> (p_string <* char ':' <* spaces) <*> p_value
```

我们用 js parsec 实现就是

```ts
p.seq(m => {
	// 过滤空格
	m(p.many(spaceParser));
	// 如果解析失败, 错误信息为 JSON text
	return m(p.label('JSON text', p.or(
		p.fmap(x => JObject, p_object); // 解析 object, 并将结果 fmap 到 JObject
		p.fmap(x => JArray, p_array); // 解析 array, 并将结果 fmap 到 JArray
	)));
});

// 解析 json array
const p_array = p.between('[', p_value, ']');

// 解析 json object
const p_object = p.between(
	'{',
	// 产生 name, value 对
	p.seq(m => {
		const name = m(p_string);
		m(p.string(':'));
		m(p.many(spaces));
		const value = m(p_value);
		return { name, value };
	}),
	'}'
);
```
