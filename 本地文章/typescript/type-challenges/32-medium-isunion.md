## 题目三十二：isunion

```ts
// template.ts
type IsUnion<T> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<IsUnion<string>, false >>,
  Expect<Equal<IsUnion<string|number>, true >>,
  Expect<Equal<IsUnion<'a'|'b'|'c'|'d'>, true >>,
  Expect<Equal<IsUnion<undefined|null|void|''>, true >>,
  Expect<Equal<IsUnion<{ a: string }|{ a: number }>, true >>,
  Expect<Equal<IsUnion<{ a: string|number }>, false >>,
  Expect<Equal<IsUnion<[string|number]>, false >>,
  // Cases where T resolves to a non-union type.
  Expect<Equal<IsUnion<string|never>, false >>,
  Expect<Equal<IsUnion<string|unknown>, false >>,
  Expect<Equal<IsUnion<string|any>, false >>,
  Expect<Equal<IsUnion<string|'a'>, false >>,
]

```

实现一个类型 `IsUnion`，它接受输入类型 `T`，并返回 `T` 是否解析为**联合类型**。



**在条件类型中，联合将是分布的**，看看下面的例子：

```ts
type Test<T, T2 = T> = T extends T2 ? {t: T, t2: T2} : true
type a = Test<string | number> // {t: string, t2: string | number} | {t: number, t2: string | number}
type b = Test<string> // {t: string, t2: string}
```

当分布发生时，**T和T2之间存在差异**。

我们可以通过将T2赋回T来判断T是否为联合类型。我们用方括号来避免分配。



### 代码实现

- 原代码

```ts
type IsUnion<T> = any
```

- 设置类型 `U`

```ts
type IsUnion<T, U = T> = any
```

- 约束 `T` 继承于 `U`

```ts
type IsUnion<T, U = T> = T extends U
	? ...
	: never
```

- 因为条件类型中的联合类型是分布的，使用 `[]` 来避免分配 

```ts
type IsUnion<T, U = T> = T extends U
	? [U] extends [T]
		? false // 两项相等，说明不是联合类型
		: true
	: never
```













