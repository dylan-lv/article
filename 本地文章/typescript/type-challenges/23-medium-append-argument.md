## 题目二十三：append-argument

```ts
// template.ts
type AppendArgument<Fn, A> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type Case1 = AppendArgument<(a: number, b: string) => number, boolean>
type Result1 = (a: number, b: string, x: boolean) => number

type Case2 = AppendArgument<() => void, undefined>
type Result2 = (x: undefined) => void

type cases = [
  Expect<Equal<Case1, Result1>>,
  Expect<Equal<Case2, Result2>>,
]
```

实现一个泛型 `AppendArgument<Fn, A>`，对于给定的函数类型 `Fn`，以及一个任意类型 `A`，返回一个新的函数 `G`。`G` 拥有 `Fn` 的所有参数并在末尾追加类型为 `A` 的参数。

```ts
type Fn = (a: number, b: string) => number
type Result = AppendArgument<Fn, boolean> 
// 期望是 (a: number, b: string, x: boolean) => number
```



### 代码实现

- 原代码

```ts
type AppendArgument<Fn, A> = any
```

- 我们要修改 `Fn` 的参数，就需要先将 `Fn` 结构出来

```ts
type AppendArgument<Fn, A> = Fn extends (...args: infer Args) => infer ReturnType
	? ...
	: ...
```

- 如果不能解构出来参数和返回值，就直接返回 `Fn`

```ts
type AppendArgument<Fn, A> = Fn extends (...args: infer Args) => infer ReturnType
	? ...
	: Fn
```

- 如果成功解构出来 `Fn` 的参数和返回值，将 `A` 添加进参数中，其余部分原样返回

```ts
type AppendArgument<Fn, A> = Fn extends (...args: infer Args) => infer ReturnType
	? (...args: [...Args, A]) => ReturnType
	: Fn
```



















