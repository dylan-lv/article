## 题目二十九：diff

```ts
// template.ts
type Diff<O, O1> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type Foo = {
  name: string
  age: string
}
type Bar = {
  name: string
  age: string
  gender: number
}
type Coo = {
  name: string
  gender: number
}

type cases = [
  Expect<Equal<Diff<Foo, Bar>, { gender: number }>>,
  Expect<Equal<Diff<Bar, Foo>, { gender: number }>>,
  Expect<Equal<Diff<Foo, Coo>, { age: string; gender: number }>>,
  Expect<Equal<Diff<Coo, Foo>, { age: string; gender: number }>>,
]
```

获取两个接口类型中的差值属性。



### 代码实现

- 原代码

```ts
type Diff<O, O1> = any
```

- 使用 `in keyof` 进行数组遍历

```ts
type Diff<O, O1> = {
  [P in keyof O]: ...
}
```

- 遍历中，需要拿出下面的项
  - `O` 中有的，并且 `O1` 中没有
  - `O1` 中有的，并且 `O` 中没有

```ts
type Diff<O, O1> = {
  [P in Exclude<keyof O, keyof O1> | Exclude<keyof O1, keyof O>]: ...
}
```

此时遍历的所有 `key` 项就已经排除掉了 `O` 和 `O1` 中重复的项了，接下来对 `value` 进行处理

- 如果当前遍历项存在于 `O`，则返回其对应的值

```ts
type Diff<O, O1> = {
  [P in Exclude<keyof O, keyof O1> | Exclude<keyof O1, keyof O>]: P extends keyof O
  	? O[P]
  	: ...
}
```

- 接下来判断当前遍历项是否存在于 `O1`，如果存在则返回其对应的值

```ts
type Diff<O, O1> = {
  [P in Exclude<keyof O, keyof O1> | Exclude<keyof O1, keyof O>]: P extends keyof O
  	? O[P]
  	: P extends O1
  		? O1[P]
  		: never
}
```

最后返回的 `never` 不会执行到，因为我们最开始对 `key` 的过滤就已经确定了它一定是在 `O` 或者 `O1` 中的

























