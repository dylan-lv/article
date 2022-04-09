## 题目八：if

```ts
// template.ts
type If<C, T, F> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<If<true, 'a', 'b'>, 'a'>>,
  Expect<Equal<If<false, 'a', 2>, 2>>,
]

// @ts-expect-error
type error = If<null, 'a', 'b'>
```

根据测试代码可以知道，`If` 接收三个参数，并根据第一个参数是 `true` 还是 `false` 来确定是返回第二个参数还是第三个参数

- 第一个参数明显是 `boolean` 类型

```ts
type If<C extends boolean, T, F> = any
```

- 如果 `C` 是 `true` 的话，返回 `T`，否则返回 `F`

```ts
type If<C extends boolean, T, F> = C extends true ? T : F
```

























