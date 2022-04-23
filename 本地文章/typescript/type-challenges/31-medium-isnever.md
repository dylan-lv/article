## 题目三十一：isnever

```ts
// template.ts
type IsNever = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<IsNever<never>, true>>,
  Expect<Equal<IsNever<never | string>, false>>,
  Expect<Equal<IsNever<''>, false>>,
  Expect<Equal<IsNever<undefined>, false>>,
  Expect<Equal<IsNever<null>, false>>,
  Expect<Equal<IsNever<[]>, false>>,
  Expect<Equal<IsNever<{}>, false>>,
]
```

实现一个类型 `IsNever`，它接受输入类型 `'T'`。

如果类型解析为 `“never”`，则返回 `“true”`，否则返回 `“false”`。



### 测试用例

通过测试用例可以知道：

- 传入 `never` 则返回 `true`
- 传入其他任何内容都是 `false`



### 代码实现

- 原代码

```ts
type IsNever<T> = any
```

- `never` 不继承任何类型，所以这里使用 `[]`处理

```ts
type IsNever<T> = [T] extends [never] ? true : false
```













