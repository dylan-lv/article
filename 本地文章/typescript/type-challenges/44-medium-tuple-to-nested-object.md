## 题目四十四：tuple-to-nested-object

```ts
// template.ts
type TupleToNestedObject<T, U> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<TupleToNestedObject<['a'], string>, { a: string }>>,
  Expect<Equal<TupleToNestedObject<['a', 'b'], number>, { a: { b: number } }>>,
  Expect<Equal<TupleToNestedObject<['a', 'b', 'c'], boolean>, { a: { b: { c: boolean } } }>>,
  Expect<Equal<TupleToNestedObject<[], boolean>, boolean>>,
]
```

给定一个只包含字符串类型的元组类型 `T`，以及一个类型 `U`，递归地构建一个对象。



### 代码实现

- 原代码

```ts
type TupleToNestedObject<T, U> = any
```

- `T` 继承自数组

```ts
type TupleToNestedObject<T extends any[], U> = any
```

- 将数组解构（如果拆分不出来东西，按照题目要求返回第二个参数 `U`）

```ts
type TupleToNestedObject<T extends any[], U> = T extends [infer First, ...infer Rest]
	? ...
	: U
```

- 按照题目要求，最后会返回一个对象，这里使用 `Record`

```ts
type TupleToNestedObject<T extends any[], U> = T extends [infer First, ...infer Rest]
	? Record<..., ...>
	: U
```

- `Record` 的第一个参数为 `key`，这里应该是数组中拆分出来的 `First`（`First`不能单独作为 `key`，使用 `&` 符号转为 `string`）

```ts
type TupleToNestedObject<T extends any[], U> = T extends [infer First, ...infer Rest]
	? Record<First & string, ...>
	: U
```

- `Record` 的值为 `TupleToNestedObject `的递归

```ts
type TupleToNestedObject<T extends any[], U> = T extends [infer First, ...infer Rest]
	? Record<First & string, TupleToNestedObject<Rest, U>>
	: U
```



