## 题目九：concat

```ts
// template.ts
type Concat<T, U> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Concat<[], []>, []>>,
  Expect<Equal<Concat<[], [1]>, [1]>>,
  Expect<Equal<Concat<[1, 2], [3, 4]>, [1, 2, 3, 4]>>,
  Expect<Equal<Concat<['1', 2, '3'], [false, boolean, '4']>, ['1', 2, '3', false, boolean, '4']>>,
]
```

观察测试用例：

- `Concat` 接收两个参数，并且都是 `any` 类型的数组
- 得到的结果是两个参数数组合并成的一个数组，并且顺序和参数传入顺序一致



### 限制参数传入类型

```ts
type Concat<T extends any[], U extends any[]> = any
```



### 合并数组

在 `ts` 类型的**数组类型**中，也可以使用类似 `js` 的**扩展运算符**。

```ts
type Concat<T extends any[], U extends any[]> = [...T, ...U]
```





















