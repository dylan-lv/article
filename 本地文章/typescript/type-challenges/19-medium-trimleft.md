## 题目十九：trimleft

```ts
// template.ts
type TrimLeft<S extends string> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<TrimLeft<'str'>, 'str'>>,
  Expect<Equal<TrimLeft<' str'>, 'str'>>,
  Expect<Equal<TrimLeft<'     str'>, 'str'>>,
  Expect<Equal<TrimLeft<'     str     '>, 'str     '>>,
  Expect<Equal<TrimLeft<'   \n\t foo bar '>, 'foo bar '>>,
  Expect<Equal<TrimLeft<''>, ''>>,
  Expect<Equal<TrimLeft<' \n\t'>, ''>>,
]
```

实现 `TrimLeft<T>` ，它接收确定的字符串类型并返回一个新的字符串，其中新返回的字符串删除了原字符串开头的空白字符串。

例如

```ts
type trimed = TrimLeft<'  Hello World  '> // 应推导出 'Hello World  '
```



### 代码实现

整体思想和数组的扩展运算符相似

- 原代码

```ts
type TrimLeft<T extends string> = any
```

- 空白字符串

空白字符串有三种：`" "`(空格符)、`"\n"`(换行符)、`"\t"`(制表符)

- 使用字符串的扩展运算符

```ts
type TrimLeft<T extends string> = T extends `${" " | "\n" | "\t"}${infer Rest}`
	? TrimLeft<Rest>
	: T
```





