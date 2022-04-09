## 题目十：includes

```ts
// template.ts
type Includes<T extends readonly any[], U> = any;
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Includes<['Kars', 'Esidisi', 'Wamuu', 'Santana'], 'Kars'>, true>>,
  Expect<Equal<Includes<['Kars', 'Esidisi', 'Wamuu', 'Santana'], 'Dio'>, false>>,
  Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 7>, true>>,
  Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 4>, false>>,
  Expect<Equal<Includes<[1, 2, 3], 2>, true>>,
  Expect<Equal<Includes<[1, 2, 3], 1>, true>>,
  Expect<Equal<Includes<[{}], { a: 'A' }>, false>>,
  Expect<Equal<Includes<[boolean, 2, 3, 5, 6, 7], false>, false>>,
  Expect<Equal<Includes<[true, 2, 3, 5, 6, 7], boolean>, false>>,
  Expect<Equal<Includes<[false, 2, 3, 5, 6, 7], false>, true>>,
  Expect<Equal<Includes<[{ a: 'A' }], { readonly a: 'A' }>, false>>,
  Expect<Equal<Includes<[{ readonly a: 'A' }], { a: 'A' }>, false>>,
  Expect<Equal<Includes<[1], 1 | 2>, false>>,
  Expect<Equal<Includes<[1 | 2], 1>, false>>,
  Expect<Equal<Includes<[null], undefined>, false>>,
  Expect<Equal<Includes<[undefined], null>, false>>,
]
```

本题想实现一个类似 `js` 数组中 `includes` 的方法：判断**参数2**是否存在于**参数1**中，如果存在则返回 `true`，否则返回 `false`

在解决这道题目之前，我们先来实现一个判断是否相等的方法（`Equal`）



### Equal

在 `ts` 类型中想要判断两个类型是否相等，主要是通过 `extends` 方式

如果传入任意一个类型，全部都满足：分别 `extends` 传入的两个类型，同时为 `true` 或同时为 `false`，则可以说明传入的这两个类型相等

代码实现：

```ts
type Equal<X, Y> = 
	(<T>() => T extends X ? 1 : 2) extends
  (<T>() => T extends Y ? 1 : 2) ? true : false
```

-  `T` 代表任意值



### Includes

接下来，开始实现 `Includes` 方法

在判断一个值是否存在于数组中的时候，很明显我们需要对数组进行遍历，然后在每次遍历的过程中对比数组当前值是否和传入的值相等。

- 数组遍历

我们可以通过解构的方式对数组进行遍历

```ts
type Includes<T extends readonly any[], U> = T extends [infer First, ...infer Rest] 
	? true 
	: false
```

使用 `infer` 定义类型只能在**等号后面**进行；

当上述代码的 `T` 中还有值的话（还能解构出来 `First`），就会执行 `true` 分支，否则会执行 `false` 分支。

执行 `false` 分支的话，代表无法再从 `T` 中解构出来值（`T` 为空），就会自动结束。

那么我们接下来的操作，就是修改 `true` 分支的内容



- 判断相等，如果相等则返回 `true`，否则进行下一轮循环

```ts
type Includes<T extends readonly any[], U> = T extends [infer First, ...infer Rest] 
	? Equal<First, U> extends true
		? true
		: Includes<Rest, U>
	: false
```

如果从 `T` 中解构出来的 `First` 和传入的 `U` 相等，则直接返回 `true`， 否则用数组剩下的内容进行下一轮循环 `Includes<Rest, U>`（类似 `js` 中的 `reduce`）











