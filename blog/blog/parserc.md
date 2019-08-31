---
id: parserc
title: Parserc
date: 2019-08-31 15:49
---

一些 paserc 的备忘录, 可以用 [Hoogle](https://hoogle.haskell.org) 查询

## sepBy

```haskell
sepBy :: ReadP a -> ReadP sep -> ReadP [a]
```

这个函数接受两个 parser 函数作为参数。第一个函数解析有效内容，第二个函数解析一个分隔符。sepBy 首先尝试解析有效内容，然后去解析分隔符，然后有效内容与分隔符依次交替解析，直到解析完有效内容之后无法继续解析到分隔符为止。它返回有效内容的列表

```haskell
sepBy many (noneOf, ",") (char ',')
```

## endBy

```haskell
endBy :: ReadP a -> ReadP sep -> ReadP [a]
```

它与 sepBy相似，不过它期望它的最后一个有效内容之后，还跟着一个分隔符（就是 parse “a\nb\nc\n”这种，而 sepBy 是 parse “a,b,c” 这种）。也就是说，它将一直进行 parse，直到它无法继续消耗任何输入

## try

```haskell
try :: ParsecT s u m a -> ParsecT s u m a
```

Parsec 有一个内置函数叫做 try 用来支持超前查看(lookahead)，try 接受一个 parser 函数，将它应用到输入。如果这个 parser 没有成功，那么 try 表现地就像它不曾消耗任何输入。所以，如果你在 <|> 的左侧应用 try，那么，即使左侧 parser 在失败时会消耗掉一些输入， Parsec 仍然会去尝试右侧的 parser。try 只有在 <|> 左侧时才会有效

** <|> **

```haskell
class Applicative f => Alternative f where
	<|> :: fa -> fa -> fa

instance Applicative.Alternative (ParsecT s u m) where
  (<|>) = mplus
```

它会首先尝试操作符左边的 parser 函数，如果这个 parser 没能成功消耗任何输入字符（没有消耗任何输入，即是说，从输入字符串的第一个字符，就可以判定无法成功解析，例如，我们希望解析”html”这个字符串，遇到的却是”php”，那从”php”的第一个字符’p’，就可以判定不会解析成功。而如果遇到的是”http”，那么我们需要消耗掉”ht”这两个字符之后，才判定匹配失败，此时，即使已经匹配失败，”ht”这两个字符仍然是被消耗掉了），那么，就尝试操作符右边的 parser

** <?> **

```haskell
(<?>) :: (ParsecT s u m a) -> String -> (ParsecT s u m a)
```

它跟 <|> 操作符很像，首先尝试操作符左边的 parser， 不过，左边解析失败时并不是去尝试另一个 parser，而是呈现一段错误信息

** *> **

```haskell
instance Applicative.Applicative (ParsecT s u m) where
	p1 *> p2 = p1 `parserBind` const p2
```

它接受两个 parser 作为参数，首先应用第一个 parser，但是忽略其返回结果，而只用作消耗输入，然后应用第二个 parser，并返回其结果。换句话说，它很像 (>>)

** <* **

```haskell
instance Applicative.Applicative (ParsecT s u m) where
	p1 <* p2 = do { x <- p1 ; void p2 ; return x }
```

它返回 <* 左侧的结果

** <*> **

从 GenParser 的 Applicative 实例的实现中, 我们知道 <*> 就是 ap

它返回两侧参数的结果, 我们可以总结 <*, *>, <*> 是箭头指向哪一侧就返回哪一侧结果
