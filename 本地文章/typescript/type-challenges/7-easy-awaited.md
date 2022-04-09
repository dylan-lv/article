## 题目七：awaited

```ts
// template.ts
type MyAwaited = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type X = Promise<string>
type Y = Promise<{ field: number }>
type Z = Promise<Promise<string | number>>

type cases = [
  Expect<Equal<MyAwaited<X>, string>>,
  Expect<Equal<MyAwaited<Y>, { field: number }>>,
  Expect<Equal<MyAwaited<Z>, string | number>>,
]

// @ts-expect-error
type error = MyAwaited<number>
```

通过测试代码我们可以知道，`MyAwaited` 接收一个 `Promise` 类型的参数，返回 `Promise` 类型参数内的参数。

并且 `Promise` 类型内还可以嵌套另外一个 `Promise`，我们最终需要返回的是最里面的普通类型

接下来，来实现我们的代码：

- 首先，我们知道 `MyAwaited` 接收的是一个 `Promise`

```ts
type MyAwaited<T extends Promise<any>> = any
```

- 我们需要返回 `Promise` 内的类型（仅条件类型的 "extends" 子句中才允许 "infer" 声明。）

```ts
type MyAwaited<T extends Promise<any>> = T extends Promise<infer P> ? P : T
```

此时，我们就返回 `Promise` 内部的类型 `P` 了。

- `Promise` 嵌套问题

如果传入的参数 `Promise` 内部还嵌套了 `Promise` 的话，此时的 `P` 就依然还是一个 `Promise`

```ts
type MyAwaited<T extends Promise<any>> = T extends Promise<infer P> 
	? P extends Promise<any>
		? MyAwaited<P>
		: P
	: T
```

















