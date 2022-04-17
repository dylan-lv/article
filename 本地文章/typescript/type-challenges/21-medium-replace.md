## 题目二十一：replace

```ts
// template.ts
type Replace<S extends string, From extends string, To extends string> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Replace<'foobar', 'bar', 'foo'>, 'foofoo'>>,
  Expect<Equal<Replace<'foobarbar', 'bar', 'foo'>, 'foofoobar'>>,
  Expect<Equal<Replace<'foobarbar', '', 'foo'>, 'foobarbar'>>,
  Expect<Equal<Replace<'foobarbar', 'bra', 'foo'>, 'foobarbar'>>,
  Expect<Equal<Replace<'', '', ''>, ''>>,
]
```

实现 `Replace<S, From, To>` 将字符串 `S` 中的第一个子字符串 `From` 替换为 `To` 。

类似于 `js` 中的 `replace` 函数



### 代码实现

- 原代码

```ts
type Replace<S extends string, From extends string, To extends string> = any
```

- 如果 From 为 空则直接返回原字符串

```ts
type Replace<S extends string, From extends string, To extends string> = From extends ''
	? S
	: ...
```

- 找出字符串中的 `From`，并声明左右两侧的字符串，如果没找到则直接返回原字符串

```ts
type Replace<S extends string, From extends string, To extends string> = From extends ''
	? S
	: S extends `${infer L}${From}${infer R}`
		? ...
		: S
```

- 将 `From` 替换为 `To`

```ts
type Replace<S extends string, From extends string, To extends string> = From extends ''
	? S
	: S extends `${infer L}${From}${infer R}`
		? `${L}${To}${R}`
		: S
```









