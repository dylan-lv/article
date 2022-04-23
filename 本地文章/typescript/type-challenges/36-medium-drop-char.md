## 题目三十六：drop-char

```ts
// template.ts
type DropChar<S, C> = any
```

```ts
// test-cases
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  // @ts-expect-error
  Expect<Equal<DropChar<'butter fly!', ''>, 'butterfly!'>>,
  Expect<Equal<DropChar<'butter fly!', ' '>, 'butterfly!'>>,
  Expect<Equal<DropChar<'butter fly!', '!'>, 'butter fly'>>,
  Expect<Equal<DropChar<'    butter fly!        ', ' '>, 'butterfly!'>>,
  Expect<Equal<DropChar<' b u t t e r f l y ! ', ' '>, 'butterfly!'>>,
  Expect<Equal<DropChar<' b u t t e r f l y ! ', 'b'>, '  u t t e r f l y ! '>>,
  Expect<Equal<DropChar<' b u t t e r f l y ! ', 't'>, ' b u   e r f l y ! '>>,
]
```

从字符串中删除指定的字符。



### 代码实现

- 原代码

```ts
type DropChar<S, C> = any
```

- 传入的两个参数都是 `string` 类型

```ts
type DropChar<S extends string, C extends string> = any
```

- 将 `S` 进行拆分，看看能不能拆出包含 `C` 的字符串

```ts
type DropChar<S extends string, C extends string> = S extends `${infer First}${C}${infer Last}`
	? ...
	: ...
```

- 如果没拆出来，说明 `S` 中已经没有 `C` 了，则直接返回

```ts
type DropChar<S extends string, C extends string> = S extends `${infer First}${C}${infer Last}`
	? ...
	: S
```

- 如果拆出来了，则去除掉当前的字符串 `C`，递归调用 `DropChar` 方法

```ts
type DropChar<S extends string, C extends string> = S extends `${infer First}${C}${infer Last}`
	? DropChar<`${First}${Last}`, C>
	: S
```

































