## 题目三十四：remove-index-signature

```ts
// template.ts
type RemoveIndexSignature<T> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type Foo = {
  [key: string]: any
  foo(): void
}

type Bar = {
  [key: number]: any
  bar(): void
}

type FooBar = {
  [key: symbol]: any
  foobar(): void
}

type Baz = {
  bar(): void
  baz: string
}

type cases = [
  Expect<Equal<RemoveIndexSignature<Foo>, { foo(): void }>>,
  Expect<Equal<RemoveIndexSignature<Bar>, { bar(): void }>>,
  Expect<Equal<RemoveIndexSignature<FooBar>, { foobar(): void }>>,
  Expect<Equal<RemoveIndexSignature<Baz>, { bar(): void; baz: string }>>,
]
```

实现“RemoveIndexSignature<T>”，从对象类型中排除索引签名（类似：`[key: string]: any`）。



### 代码实现

- 原代码

```ts
type RemoveIndexSignature<T> = any
```

- 遍历传入的参数 `T`

```ts
type RemoveIndexSignature<T> = {
  [P in keyof T]: T[P]
}
```

- 将键值 `P` 放入字符串中，判断是否和原先的内容相同（索引签名不是字符串）

```ts
type RemoveIndexSignature<T> = {
  [P in keyof T as P extends `${infer R}` ? R : never]: T[P]
}
```



























