## 题目十一：parameters

```ts
// template.ts
type MyParameters<T extends (...args: any[]) => any> = any
```

```ts
// test-cases.ts
import { Equal, Expect, ExpectFalse, NotEqual } from '@type-challenges/utils'

const foo = (arg1: string, arg2: number): void => {}
const bar = (arg1: boolean, arg2: {a: 'A'}): void => {}
const baz = (): void => {}

type cases = [
  Expect<Equal<MyParameters<typeof foo>, [string, number]>>,
  Expect<Equal<MyParameters<typeof bar>, [boolean, {a: 'A'}]>>,
  Expect<Equal<MyParameters<typeof baz>, []>>,
]
```

这道题目要求我们**实现内置的 `Parameters` 类型**，而不是直接使用它，可参考[TypeScript官方文档](https://www.typescriptlang.org/docs/handbook/utility-types.html#parameterstype)。

`TS` 官网对于 `Parameters` 类型的介绍：从函数类型的参数中使用的类型构造元组类型。

也就是获取**函数的参数以及其类型**。

- 示例1

```ts
type T0 = Parameters<() => string>; // type T0 = []
```

因为 `() => string` 这个函数的参数为空，所以返回空数组： `[]`

- 示例2

```ts
type T1 = Parameters<(s: string) => void>; // type T1 = [s: string]
```

`(s: string) => void`：该函数只有一个参数 `s`，并且参数类型为 `string`，所以返回值为：`[s: string]`

- 示例3

```ts
type T2 = Parameters<<T>(arg: T) => T>; // type T2 = [arg: unknown]
```

我们知道在 `TS` 类型中，`T` 代表任意类型的其中一种，所以这里的返回的 `arg` 的类型为 `unknown`

- 示例4

```ts
declare function f1(arg: { a: number; b: string }): void;
type T3 = Parameters<typeof f1>; // type T3 = [arg: { a: number; b: string; }]
```

有了上面的示例，这里就很好理解：函数参数 `arg` 是一个对象，里面有 `a` 和 `b` 两个属性（其中 `typeof` 负责将函数转换为相应的**函数类型**）

- 示例5

```ts
type T4 = Parameters<any>; // type T4 = unknown[]
```

- 示例6

```ts
type T5 = Parameters<never>; // type T5 = never
type T6 = Parameters<string>; // type T6 = never
type T7 = Parameters<Function>; // type T7 = never
```

其中 `T6` 和 `T7` 会报错，因为其不满足传入的**函数类型**约束



### 代码实现

- `Parameters` 的入参应该是一个携带参数的函数定义（不能是`Function`）：() => any

```ts
// 这里其实题目中已经有了，注意 T 是一个函数，args是其参数数组
type MyParameters<T extends (...args: any[]) => any> = any
```

- 拿到 `...args` 的类型，直接返回即可


我们知道在 `ts` 类型中，想定义参数的话，需要使用 `infer`，而 `infer` 只能在**等号右侧**定义

```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer P) => any
```

现在，我们已经拿到入参（`args`）定义的类型 `P` 了，那么只需要将 `P` 返回即可；不满足约束（`extends`）的返回 `never`

```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer P) => any ? P : never
```



至此，这道题就解决了。

总结：

- 入参限制类型：`() => any`，不满足的直接返回 `never`
- 使用 `infer` 在等号右侧定义入参（args）的类型 `P`
  - 满足条件则返回 `P`
  - 否则返回 `never`



















