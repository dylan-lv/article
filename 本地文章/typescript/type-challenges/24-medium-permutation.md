## 题目二十四：permutation

```ts
// template.ts
type Permutation<T> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Permutation<'A'>, ['A']>>,
  Expect<Equal<Permutation<'A' | 'B' | 'C'>, ['A', 'B', 'C'] | ['A', 'C', 'B'] | ['B', 'A', 'C'] | ['B', 'C', 'A'] | ['C', 'A', 'B'] | ['C', 'B', 'A']>>,
  Expect<Equal<Permutation<'B' | 'A' | 'C'>, ['A', 'B', 'C'] | ['A', 'C', 'B'] | ['B', 'A', 'C'] | ['B', 'C', 'A'] | ['C', 'A', 'B'] | ['C', 'B', 'A']>>,
  Expect<Equal<Permutation<never>, []>>,
]
```

实现**联合类型的全排列**，将联合类型转换成所有可能的全排列数组的联合类型。

测试用例中，

- 测试1：表示单个元素的全排列
- 测试2和3：不在乎顺序
- 测试4：`never` 的全排列为**空数组**



### 代码实现

- 原代码

```ts
type Permutation<T> = any
```

- `never` 的全排列为**空数组**（测试4要求）

至于为什么要使用 `[T] extends [never]` ，而不是使用 `T extends never`，我们之后再解释

```ts
type Permutation<T> = [T] extends [never]
	? []
	: ...
```

- 给定另一个泛型：`U`。初始值设置为 `T`，用来保存 `T` 之前的内容

此时在 `T extends U` 中，`U` 代表传入的联合类型，例如：`'A' | 'B' | 'C'`；`T` 则代表继承 `U` 的某个值，例如：`"A"` 或 `"B"` 或 `"C"`

```ts
type Permutation<T, U = T> = [T] extends [never]
	? []
	: (T extends U
		? [T, ...]
		: [])
```

可以理解为：此时的 `T` 已经不是之前的联合类型了，而是联合类型其中的一个类型，而 `U` 则是之前的联合类型

- 递归处理剩下的值，需要排除当前已经使用过的 `T`

```ts
type Permutation<T, U = T> = [T] extends [never]
	? []
	: (T extends U
		? [T, ...Permutation<Exclude<U, T>>]
		: [])
```

由于 `Permutation` 返回的是一个数组，所以这里使用了扩展运算符：`[T, ...Permutation<Exclude<U, T>>]`



### [T] extends [never]

我们回来看看为什么要使用 `[T] extends [never]` 而不是直接使用 `T extends never` 来做判断

- 首先我们需要知道，`never` 是所有类型的子类型

```ts
type p1 = never extends "x" ? string : number // string
```

- 再来看看如果将其放入**类型函数**中会如何

```ts
type P<T> = T extends "x" ? string : number
type p2 = P<never> // never
```

我们会发现这里传入 `never` 后，并没有返回**三元表达式**中的 `string` 或 `number`

这是因为 `never` 被认为是空的联合类型，也就是说，没有联合项的联合类型，所以还是满足上面的分配律。

然而因为没有联合项可以分配，所以 `P` 的表达式其实根本就没有执行，所以 `p2` 的定义也就类似于永远没有返回的函数一样，是 `never` 类型的

- 防止条件判断中的分配

在条件判断类型的定义中，将泛型参数使用 `[]` 括起来，即可阻断条件判断类型的分配。此时，传入参数 `T` 的类型将被当作一个整体，不再分配

```ts
type P<T> = [T] extends ["x"] ? string : number
type p1 = P<"x" | "y"> // number
type p2 = P<never> // string
```







