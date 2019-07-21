---
id: f-a-m
title: Functor Applicative Monad
date: 2019-07-20 22:16
---

## Functor

我们有一个 func, 它接受一个类型为 a 的值, 返回一个 b 类型的值, 那么我们怎么让这个函数应用于一个类型为 f a 的值呢?
这就是 fmap, 它能够跟上下文很好的沟通, 也就是能应用于 fa.
看看 fmap 的实现:

```haskell
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```

我们看看 Maybe 的实现:

```haskell
instance Functor Maybe where
  fmap func Nothing -> Nothing
  fmap func (Just a) -> Just (func a)
```

如果我们想把函数应用于另一个函数上呢?
```haskell
fmap (+1) (*2)
```

```haskell
> import Control.Applicative
> let f = fmap (+1) (*2)
> f 2
5
```

由此可见, 函数同样也是 Functor, 函数的 fmap 就是函数的 compose

```haskell
instance Functor ((->) r) where
  fmap f g -> f . g
```

## Applicative

Functor 是将函数应用于被封装在上下文的值上, 与 Functor 类似 Applicative 的值也是被封装在上下文中, 但是 Applicative 的函数也是在封装在上下文中的。
Control.Applicative 封装了 `<*>`, 它知道如何将 __一个包装在上下文中的函数应用于被包装在上下文的值上__
举个例子:

```haskell
> Just (+3) <*> Just 2
Just 5
```

我们看看 Applicative 有哪些 Functor 不具备的功能
比如 Functor 如何处理接受两个被封装的值(wrapper values)的函数 which 接受两个参数

> tips: <$> 是 fmap 的中缀写法

```haskell
> (+) <$> (Just 5)
Just (+5)
> Just (+5) <$> (Just 2)
Error
```

Applicatives:

```haskell
> (+) <$> (Just 5)
Just (+5)
> Just (+5) <*> (Just 2)
Just 7
```

Applicative 只要装备了 `<$>` 和 `<*>` 之后, 就可以接受任何函数, 然后把对应的封装值塞给他们, 就能得到一个封装好的值

```haskell
> (*) <$> (Just 3) <*> (Just 4)
Just 12
```

它还有个简化的写法

```haskell
liftA2 (*) (Just 3) (Just 4)
```

## Monad

假设我们有一个 func, 它接受一个类型为 a 的参数, 返回一个 m b 类型的值, 那么有一种操作可以让 m a 类型的值也能被应用于这个函数, 这种操作叫做 `>>=`, 读作 bind.
举个例子:

```haskell
square x = if even x
           then Just (x * x)
           else Nothing
```

那么如果我们给这个函数一个 Just 4 会怎么样, 这时候我们就需要用到 bind 来将这个值塞进 square 中

```haskell
Just 4 >>= square -- Just 16
Just 3 >>= square -- Nothing
```

我们来看看 Monad 这个 type class 的定义:

```haskell
class Monad m where
  (>>=) :: m a -> (a -> m b) -> m b
```

我们看看 Maybe 是如何实现 Monad 的:

```haskell
instance Monad Maybe where
  Nothing >>= func = Nothing
  Just a  >>= func = func a
```

实战场景：

假如我们实现一个从 terminal 获取用户输入的文件名，然后在 terminal 打印出这个文件内容的小功能，如何做呢？
我们应该分三步：
1. 从命令行读取用户的输入 (getLine :: IO String)
2. 读取该文件名的文件内容 (readFile :: FilePath -> IO String)
3. 打印出这个内容 (putStrLn :: String -> IO ())

我们用 bind 串联起来上面三个步骤：

```haskell
getLine >>= readFile >>= putStrLn
```

haskell 还为它提供了语法糖 `do`:

```haskell
f = do
  fileName <- getLine
  contents <- readFile
  putStrLn contents
```

## 总结

- Functor 使用 fmap 或者 <$> 将一个普通函数 (a -> b) 应用于一个被封装的值 (m a) 上
- Applicative 使用 <*> 将一个被封装的函数 (m a -> m b) 应用于一个被封装的值(m a)上
- Monad 使用 >>= 将一个被封装的值 (m a) 应用到一个接受普通值返回封装值的函数上 (a -> m b )，返回 m b