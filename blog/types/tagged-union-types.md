---
title: Tagged union types
date: 2020-08-30 14:57:26
type: post
tag: Type
meta:
  - name: keywords
    content: tagged union types
---

[Tagged union wikipedia](https://en.wikipedia.org/wiki/Tagged_union)

> also called a variant, variant record, choice type, discriminated union, disjoint union, sum type or coproduct

比如 Haskell 中的 algebraic data type 就是一种。带 tagged 的 union type 可以让编译器对【类型的所有cases 是否都做了处理】作出检查，从而避免掉很多错误。而且这种类型跟 pattern matching 配合起来使用非常方便。

在 Haskell 中，algebraic data type 通过类型构造器来做 tag；而在 typescript 中，则是需要通过 discriminated field 来做 tag，所以在 typescript 中，你会经常需要给 union type 加上 `kind:  "cat"` 这样的东西（ 这是因为 ts 支持 String literal type）

不过 typescript 也可以通过一些方法来自动加 tag

通过 index type query 和 index access operator 做 tagged union type：

```tsx
interface Cat {
    name: string;
    miao(): string;
}

interface Dog {
    name: string;
    wang(): string;
}

interface Animal {
    cat: { name: string, miao(): string };
    dog: { name: string, wang(): string };
}

type AddKind<T> = {
    [k in keyof T]: T[k] & { kind: k }
}

type Intermediate = AddKind<Animal>;
type Shape = Intermediate[keyof Animal];

// untagged union type
function badCase(x: Cat | Dog) {

}

// tagged union type
function sayHello(x: Shape) {
    switch(x.kind) {
        case "cat": return x.miao();
        case "dog": return x.wang();
    }
}
```

如果我们没有对所有的 cases 都做处理，编译器就可以检查出来：

![warnings](/static/tagged-union-types/uncheck.png)