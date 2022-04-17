## 题目十五：deep-readonly

```ts
// template.ts
type DeepReadonly<T> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<DeepReadonly<X>, Expected>>,
]

type X = {
  a: () => 22
  b: string
  c: {
    d: boolean
    e: {
      g: {
        h: {
          i: true
          j: 'string'
        }
        k: 'hello'
      },
      l: ['hi']
    }
  }
}

type Expected = {
  readonly a: () => 22
  readonly b: string
  readonly c: {
    readonly d: boolean
    readonly e: {
      readonly g: {
        readonly h: {
          readonly i: true
          readonly j: 'string'
        }
        readonly k: 'hello'
      },
      readonly l: ['hi']
    }
  }
}
```

根据题目和测试用例我们可以知道，这里要实现一个**深度递归**



### Record

在解题之前，我们先来了解一下 `ts` 内置方法 `Record`

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

将K中的每个属性([P in K]),都转为T类型。

```ts
type t1 = Record<string, unknown> // type t1 = { [x: string]: unknown; }
```

`Record<K,T>`构造具有给定类型`T`的一组属性`K`的类型。在将一个类型的属性映射到另一个类型的属性时，`Record`非常方便。

他会将一个类型的所有属性值都映射到另一个类型上并创造一个新的类型.

```ts
interface User {
  name: string
  age: number
}

let user: Record<number, User> = {
  0: { name: "hay0", age: 18 },
  1: { name: "hay1", age: 20 }
}
```



### 代码实现

- 原代码

```ts
type DeepReadonly<T> = any
```

- 遍历

```ts
type DeepReadonly<T> = {
  [P in keyof T]: T[P]
}
```

- 添加 `readonly` 修饰符

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P]
}
```

- 判断是否是对象，是对象则进行递归，不是则直接返回

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
  	? DeepReadonly<T[P]>
  	: T[P]
}
```

测试用例依然无法通过，这时我们就需要用到 `Record` 这个方法了，将 `object` 替换为 `key` 为 `string`，`value` 为 `unknown` 的方式

- Record

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends Record<string, unknown>
  	? DeepReadonly<T[P]>
  	: T[P]
}
```



