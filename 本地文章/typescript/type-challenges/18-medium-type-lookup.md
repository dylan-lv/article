## 题目十八：type-lookup

```ts
// template.ts
type LookUp<T, U> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

interface Cat {
  type: 'cat'
  breeds: 'Abyssinian' | 'Shorthair' | 'Curl' | 'Bengal'
}

interface Dog {
  type: 'dog'
  breeds: 'Hound' | 'Brittany' | 'Bulldog' | 'Boxer'
  color: 'brown' | 'white' | 'black'
}

type Animal = Cat | Dog

type cases = [
  Expect<Equal<LookUp<Animal, 'dog'>, Dog>>,
  Expect<Equal<LookUp<Animal, 'cat'>, Cat>>,
]
```

有时，您可能希望根据其属性在并集中查找类型。

在此挑战中，我们想通过在联合`Cat | Dog`中搜索公共`type`字段来获取相应的类型。换句话说，在以下示例中，我们期望`LookUp<Dog | Cat, 'dog'>`获得`Dog`，`LookUp<Dog | Cat, 'cat'>`获得`Cat`。



### 代码实现

- 原代码

```ts
type LookUp<T, U> = any
```

- 传入的参数 `T、U`

传入的 `T` 是一个联合类型，并且每一项（object）中都有 `type` 字段；

传入的 `U` 是一个字符串类型，对应 `type` 字段，用作筛选

```ts
type LookUp<T extends { type: string }, U extends string> = any
```

- 判断 `T` 是否 `extends` 相应的 `U`，如果是的话，就直接返回 `T`，否则返回 `never`

这里其实隐含了联合类型的遍历

```ts
type LookUp<T extends { type: string }, U extends string> = T extends { type: U } ? T : never
```



