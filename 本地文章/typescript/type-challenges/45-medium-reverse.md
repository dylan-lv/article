## 题目四十五：reverse

```ts
// template.ts
type Reverse<T> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Reverse<['a', 'b']>, ['b', 'a']>>,
  Expect<Equal<Reverse<['a', 'b', 'c']>, ['c', 'b', 'a']>>,
]
```

实现 `TS` 版本的 `Array.reverse`



### 代码实现

- 原代码

```ts
type Reverse<T> = any
```

- `T` 是一个数组，再多设置一个 `Result` 在存放结果

```ts
type Reverse<T extends any[], Result extends any[] = []> = any
```

- 将 `T` 进行解构，如果解构不出来则直接返回 `Result`

```ts
type Reverse<T extends any[], Result extends any[] = []> = T extends [infer First, ...infer Rest]
	? ...
	: Result
```

- 继续对剩余的内容（`Rest`）进行翻转

```ts
type Reverse<T extends any[], Result extends any[] = []> = T extends [infer First, ...infer Rest]
	? Reverse<Rest>
	: Result
```

- 递归调用 `Reverse` 中的第二个参数（结果参数，将解构出来的 `First`加上）

```ts
type Reverse<T extends any[], Result extends any[] = []> = T extends [infer First, ...infer Rest]
	? Reverse<Rest, [First, ...Result]>
	: Result
```



### 注意

`TS` 这里不能直接对 `T` 中的内容进行翻转，需要借助另一个参数 `Result` 作为结果数组来进行操作