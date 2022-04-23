## 题目三十八：startswith

```ts
// template.ts
type StartsWith<T extends string, U extends string> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<StartsWith<'abc', 'ac'>, false>>,
  Expect<Equal<StartsWith<'abc', 'ab'>, true>>,
  Expect<Equal<StartsWith<'abc', 'a'>, true>>,
  Expect<Equal<StartsWith<'abc', 'abcd'>, false>>,
]
```

实现 `'StartsWith<T，U>'`，它接受两种精确的字符串类型，并返回`'T'`是否以`'U'`开头



### 代码实现

- 原代码

```ts
type StartsWith<T extends string, U extends string> = any
```

- 直接对 `T` 进行拆分，但是拆分内容中包含 `U`

```ts
type StartsWith<T extends string, U extends string> = T extends `${U}${infer _}` ? true : false
```

- 也可以写成

```ts
type StartsWith<T extends string, U extends string> = T extends `${U}${string}` ? true : false
```





















