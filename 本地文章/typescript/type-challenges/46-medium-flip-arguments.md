## 题目四十六：flip-arguments

```ts
// template.ts
type FlipArguments<T> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<FlipArguments<() => boolean>, () => boolean>>,
  Expect<Equal<FlipArguments<(foo: string) => number>, (foo: string) => number>>,
  Expect<Equal<FlipArguments<(arg0: string, arg1: number, arg2: boolean) => void>, (arg0: boolean, arg1: number, arg2: string) => void>>,
]
```

实现类型版本的 `lodash` 中的 ``_.flip``。

类型 `FlipArguments<T>` 需要函数类型 `T`，并返回一个新函数类型，该函数类型的返回类型与 `T` 相同，但参数相反。

相当于：将函数中的参数翻转



### 代码实现

- 原代码

```ts
type FlipArguments<T> = any
```

- 将函数进行解构

```ts
type FlipArguments<T> = T extends (...args: infer Args) => infer Result
	? ...
	: ...
```

- 如果解构不出来，则说明不是传入的 `T` 不是函数，返回 `never`

```ts
type FlipArguments<T> = T extends (...args: infer Args) => infer Result
	? ...
	: never
```

- 解构出来后，将 `Args` 进行翻转

```ts
type FlipArguments<T> = T extends (...args: infer Args) => infer Result
	? (...args: Reverse<Args>) => Result
	: never
```

`Reverse` 函数我们上一题做过了