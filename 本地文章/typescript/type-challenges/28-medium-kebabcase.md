## 题目二十八：kebabcase

```ts
// template.ts
type KebabCase<S> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<KebabCase<'FooBarBaz'>, 'foo-bar-baz'>>,
  Expect<Equal<KebabCase<'fooBarBaz'>, 'foo-bar-baz'>>,
  Expect<Equal<KebabCase<'foo-bar'>, 'foo-bar'>>,
  Expect<Equal<KebabCase<'foo_bar'>, 'foo_bar'>>,
  Expect<Equal<KebabCase<'Foo-Bar'>, 'foo--bar'>>,
  Expect<Equal<KebabCase<'ABC'>, 'a-b-c'>>,
  Expect<Equal<KebabCase<'-'>, '-'>>,
  Expect<Equal<KebabCase<''>, ''>>,
  Expect<Equal<KebabCase<'😎'>, '😎'>>,
]
```



### 测试用例

- `Equal<KebabCase<'FooBarBaz'>, 'foo-bar-baz'>`

如果是首字母大写，则直接转换成小写

如果是非首字母大写，则将其转换成小写后，前方添加中划线（`-`）

- `Equal<KebabCase<'foo_bar'>, 'foo_bar'>`

只对中划线做处理



### 首字母小写的TS函数

```ts
/**
 * Convert first character of string literal type to lowercase
 */
type Uncapitalize<S extends string> = intrinsic;
```





### 代码实现

- 原代码

```ts
type KebabCase<S> = any
```

- 入参 `S` 是一个字符串

```ts
type KebabCase<S extends string> = any
```

- 将 `S` 拆分为左右两个字符串（左边可以当作是首字母，右边是剩余部分）
  - 如果拆分出来了，则进行下一步
  - 否则直接返回 `S`

```ts
type KebabCase<S extends string> = S extends `${infer L}${infer R}`
	? ...
	: S
```

- 首字母（`L`）不论如何，都要转成小写。在此基础上，判断 `R` 是否是首字母小写的

```ts
type KebabCase<S extends string> = S extends `${infer L}${infer R}`
	? R extends Uncapitalize<R>
		? `${Uncapitalize<L>}...`
		: `${Uncapitalize<L>}...`
	: S
```

- 如果 `R` 是首字母小写的，则不做任何处理，接着对其递归 `KebabCase`
- 否则添加中划线，再进行递归（不需要特意加上`Uncapitalize`函数，因为`KebabCase` 本身就含有首字母小写的功能了）

```ts
type KebabCase<S extends string> = S extends `${infer L}${infer R}`
	? R extends Uncapitalize<R>
		? `${Uncapitalize<L>}${KebabCase<R>}`
		: `${Uncapitalize<L>}-${KebabCase<R>}`
	: S
```







