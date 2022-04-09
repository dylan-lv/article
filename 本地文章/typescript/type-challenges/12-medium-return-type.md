## 题目十二：return-type

```ts
// template.ts
type MyReturnType<T> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<string, MyReturnType<() => string>>>,
  Expect<Equal<123, MyReturnType<() => 123>>>,
  Expect<Equal<ComplexObject, MyReturnType<() => ComplexObject>>>,
  Expect<Equal<Promise<boolean>, MyReturnType<() => Promise<boolean>>>>,
  Expect<Equal<() => 'foo', MyReturnType<() => () => 'foo'>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn1>>>,
]

type ComplexObject = {
  a: [12, 'foo']
  bar: 'hello'
  prev(): number
}

const fn = (v: boolean) => v ? 1 : 2
const fn1 = (v: boolean, w: any) => v ? 1 : 2
```

这道题目要求我们**实现内置的 `ReturnType` 类型**，而不是直接使用它，可参考[TypeScript官方文档](https://www.typescriptlang.org/docs/handbook/utility-types.html#returntypetype)。

官方介绍：构造一个由函数类型的返回类型组成的类型。

也就是**获取函数的返回值类型**。



- 示例1

```ts
type T0 = ReturnType<() => string>; // type T0 = string
```

很显然，函数返回值类型是 `string`，所以 `T0` 的值也是 `string`

- 示例2

```ts
type T1 = ReturnType<(s: string) => void>; // type T1 = void
```

- 示例3

```ts
type T2 = ReturnType<<T>() => T>; // type T2 = unknown
```

`ts` 中的 `T`（泛型），代指任意类型中的一个，所以这里返回值是 `unknown`

- 示例4

```ts
type T3 = ReturnType<<T extends U, U extends number[]>() => T>; // type T3 = number[]
```

这里定义了两个泛型：`T` 和 `U`，其中 `U` 是继承自 `number[]` 类型，`T` 则是继承 `U` 类型。所以 `T` 也是 `number[]` 类型

- 示例5

```ts
declare function f1(): { a: number; b: string };
type T4 = ReturnType<typeof f1>; // type T4 = { a: number; b: string; }
```

这里定义了一个函数 `f1`，该函数的返回值类型是一个对象：`{ a: number, b: string }`

所以这里 `T4` 的值也就是 `f1` 的返回值类型 `{ a: number, b: string }`

- 示例6

```ts
type T5 = ReturnType<any>;
type T6 = ReturnType<never>;
type T7 = ReturnType<string>; // Error
type T8 = ReturnType<Function>; // Error
```

`any` 返回 `any`

`never` 返回 `never`

其他不满足条件的入参，直接报错



### 代码实现

- 原始代码

```ts
type MyReturnType<T> = any
```

- 入参应该是一个函数

```ts
type MyReturnType<T extends () => any> = any
```

- 入参的函数可能会有参数

```ts
type MyReturnType<T extends (...args: any[]) => any> = any
```

- 使用 `infer` 定义函数的返回类型 `P`

```ts
type MyReturnType<T extends (...args: any[]) => any> = T extends (...args: any[]) => infer P
```

- 如果该约束成功，则返回 `P` 否则返回 `any`

```ts
type MyReturnType<T extends (...args: any[]) => any> = T extends (...args: any[]) => infer P ? P : any
```















