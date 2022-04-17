## 题目十七：promise-all

```ts
// template.ts
declare function PromiseAll(values: any): any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

const promiseAllTest1 = PromiseAll([1, 2, 3] as const)
const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const)
const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)])

type cases = [
  Expect<Equal<typeof promiseAllTest1, Promise<[1, 2, 3]>>>,
  Expect<Equal<typeof promiseAllTest2, Promise<[1, 2, number]>>>,
  Expect<Equal<typeof promiseAllTest3, Promise<[number, number, number]>>>
]
```

键入函数`PromiseAll`，它接受PromiseLike对象数组，返回值应为`Promise<T>`，其中`T`是解析的结果数组。

### 测试用例

- 示例1

```ts
const promiseAllTest1 = PromiseAll([1, 2, 3] as const) // 期望：Promise<[1, 2, 3]>
```

`[1, 2, 3] as const` 会将 `[1, 2, 3]`作为具体的类型传入 `PromiseAll`，期望值是 `Promise<[1, 2, 3]>`，说明只需要简单的遍历并输出即可

- 示例2

```ts
const promiseAllTest2 = PromiseAll([1, 2, Promise.resolve(3)] as const) // 期望：Promise<[1, 2, number]>
```

这个示例不同的点在于数组中的第三个参数是一个 `Promise` 类型，这个 `Promise` 类型中传入的数值 `3`。同时，期望的值在该位置变为 `number`，也就是数值 `3` 的类型

这里可以判断：如果**传入的数组中**有 `Promise` 类型，则返回值相应的位置会输出对应 `Promise` 中接收的类型

- 示例3

```ts
const promiseAllTest3 = PromiseAll([1, 2, Promise.resolve(3)]) // 期望：Promise<[number, number, number]
```

`as const`操作的作用是**将相应的值变为类型**，这个示例中没有使用 `as const`，则期望的结果对应位置全部是相应值的类型



### 代码实现

- 原代码

```ts
declare function PromiseAll(values: any): any
```

- `values` 类型

`values` 类型应该是一个数组，数组中可能是任意类型（使用 `unknown`）

```ts
declare function PromiseAll(values: unknown[]): any
```

在外部使用 `as const` 操作后，相应的**类型**会变为 `readonly` 的，所以 `values` 接收的时候，也需要加上 `readonly` 修饰符

```ts
declare function PromiseAll(values: readonly unknown[]): any
```

- 传入的 `values` 类型，后面也会用到，所以使用泛型 `T` 来表示

```ts
declare function PromiseAll<T extends unknown[]>(values: readonly [...T]): any
```

这里如果时间使用 `values: readonly T` 的话会报错：仅允许对数组和元组文本类型使用 "readonly" 类型修饰符。

因为 `T` 继承 `unknown[]`，但不一定是**数组**

- 最后返回一个 `Promise` 类型，并且 `Promise` 的参数是一个**数组**

```ts
declare function PromiseAll<T extends unknown[]>(values: readonly [...T]): Promise<[]>
```

- 对返回值 `Promise` 的参数数组进行遍历

```ts
declare function PromiseAll<T extends unknown[]>(values: readonly [...T]): Promise<{
  [P in keyof T]: T[P]
}>
```

- 如果遍历的内容 `T[P]` 是 `Promise` 类型的话，则返回该 `Promise` 的入参类型

```ts
declare function PromiseAll<T extends unknown[]>(values: readonly [...T]): Promise<{
  [P in keyof T]: T[P] extends Promise<infer R> ? R : T[P]
}>
```













