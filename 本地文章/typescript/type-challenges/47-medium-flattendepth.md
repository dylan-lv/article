## 题目四十七：flattendepth

```ts
// template.ts
type FlattenDepth = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<FlattenDepth<[]>, []>>,
  Expect<Equal<FlattenDepth<[1, 2, 3, 4]>, [1, 2, 3, 4]>>,
  Expect<Equal<FlattenDepth<[1, [2]]>, [1, 2]>>,
  Expect<Equal<FlattenDepth<[1, 2, [3, 4], [[[5]]]], 2>, [1, 2, 3, 4, [5]]>>,
  Expect<Equal<FlattenDepth<[1, 2, [3, 4], [[[5]]]]>, [1, 2, 3, 4, [[5]]]>>,
  Expect<Equal<FlattenDepth<[1, [2, [3, [4, [5]]]]], 3>, [1, 2, 3, 4, [5]]>>,
  Expect<Equal<FlattenDepth<[1, [2, [3, [4, [5]]]]], 19260817>, [1, 2, 3, 4, 5]>>,
]
```



### 测试用例

- 空数组直接返回
- 默认深度为 1
- 参数 1 是需要打平的数组，参数 2 是需要打平的深度



### 代码实现

- 原代码

```ts
type FlattenDepth = any
```

- 设置三个入参
  - 参数1：需要打平的数组
  - 参数2：需要打平的深度
  - 参数3：默认空数组，使用其 `['length']` 属性来判断当前深度

```ts
type FlattenDepth<T extends any[], C extends number = 1, U extends any[] = []> = any
```

- 开始遍历数组 `T`，使用 `extends` 方式，每次解构出来一个值，依次判断
  - 解构不出来的话，说明当前的 `T` 是个空数组了，直接返回

```ts
type FlattenDepth<T extends any[], C extends number = 1, U extends any[] = []> = T extends [infer F, ...infer R]
	? ...
	: T
```

- 判断解构出来的第一个值 `F`（也就是当前循环的值）是否是一个数组
  - 如果不是数组，则进行下一次循环 （继续遍历剩下的数据 `R`）

```ts
type FlattenDepth<T extends any[], C extends number = 1, U extends any[] = []> = T extends [infer F, ...infer R]
	? F extends any[]
		? ...
		: [F, ...FlattenDepth<R, C, U>]
	: T
```

- 如果解构出来的值 `F` 是一个数组，则继续判断当前深度 `U` 是否已经是所需的深度 `C` 了
  - 如果已经达到所需的深度了，则直接返回，不进行数组打平操作

```ts
type FlattenDepth<T extends any[], C extends number = 1, U extends any[] = []> = T extends [infer F, ...infer R]
	? F extends any[]
		? U['length'] extends C
			? [F, ...FlattenDepth<R, C, U>]
			: ...
		: [F, ...FlattenDepth<R, C, U>]
	: T
```

- 否则就是需要进行数组打平操作了
  - 这里我们可以知道肯定是返回一个数组
  - 数组会处理解构出来的两个值 （`F` 和 `R`）
  - 对 `F` 进行深一层的打平（`U` 添加一个属性，使其 `length` 多1）
  - 对 `R` 接着进行遍历

```ts
type FlattenDepth<T extends any[], C extends number = 1, U extends any[] = []> = T extends [infer F, ...infer R]
	? F extends any[]
		? U['length'] extends C
			? [F, ...FlattenDepth<R, C, U>]
			: [...FlattenDepth<F, C, [0, ...U]>, ...FlattenDepth<R, C, U>]
		: [F, ...FlattenDepth<R, C, U>]
	: T
```

