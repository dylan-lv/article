## 题目三十九：partialbykeys

```ts
// template.ts
type PartialByKeys<T, K> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

interface User {
  name: string
  age: number
  address: string
}

interface UserPartialName {
  name?: string
  age: number
  address: string
}

interface UserPartialNameAndAge {
  name?: string
  age?: number
  address: string
}

type cases = [
  Expect<Equal<PartialByKeys<User, 'name'>, UserPartialName>>,
  Expect<Equal<PartialByKeys<User, 'name' | 'unknown'>, UserPartialName>>,
  Expect<Equal<PartialByKeys<User, 'name' | 'age'>, UserPartialNameAndAge>>,
  Expect<Equal<PartialByKeys<User>, Partial<User>>>,
]
```

实现一个通用的 `PartialByKeys<T，K>`，它接受两种类型的参数 `T` 和 `K`。

`K`指定应设置为可选的`T`属性集。当没有提供 `K `时，它应该使所有属性都是可选的，就像正常的 `Partial<T>` 。



### 测试用例

```ts
interface User {
  name: string
  age: number
  address: string
}
```



- `PartialByKeys<User, 'name'>`

将 `User` 中的 `name` 属性变为**可选**，期望结果：`{ name?: string, age: number, address: string }`

- `PartialByKeys<User, 'name' | 'unknown'>`

联合类型中的 `unknown` 不会影响结果

- `PartialByKeys<User>`

如果不传入第二个参数，则全部变为可选



### 代码实现

- 原代码

```ts
type PartialByKeys<T, K> = any
```

- 使用 `Omit` 拿到 `K` 之外的内容。使用 `K & keyof T` 去除 `T` 之外的内容，例如：`unknown`

```ts
type PartialByKeys<T, K> = Omit<T, K & keyof T>
```

- 合并剩余的 `K` 中的内容，并加上可选符号 `?`

```ts
type PartialByKeys<T, K> = Omit<T, K & keyof T> & {
  [P in K & keyof T]?: T[P]
}
```

- 此时还是报错状态，还需要对齐遍历（Copy）一份，因为 `Omit` 合并的缘故

```ts
type Copy<T> = {
  [P in keyof T]: T[P]
}
type PartialByKeys<T, K = keyof T> = Copy<Omit<T, K & keyof T> & {
  [P in K & keyof T]?: T[P]
}>
```







