---
id: js-monad
title: javascript 实现 Monad
date: 2019-07-24 02:12
---

## Monad 的定义

```haskell
class Monad m where
  return :: a -> m a

  >>= :: m a -> (a -> m b) -> m b

  >> :: m a -> m b -> m b
  x >> y = x >>= \_ -> y

  fail :: String -> m a
  fail = error
```