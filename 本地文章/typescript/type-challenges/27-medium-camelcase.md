## 题目二十七：camelcase

```ts
// template.ts
type CamelCase<S> = any;
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<CamelCase<'foo-bar-baz'>, 'fooBarBaz'>>,
  Expect<Equal<CamelCase<'foo-Bar-Baz'>, 'foo-Bar-Baz'>>,
  Expect<Equal<CamelCase<'foo-Bar-baz'>, 'foo-BarBaz'>>,
  Expect<Equal<CamelCase<'foo-bar'>, 'fooBar'>>,
  Expect<Equal<CamelCase<'foo_bar'>, 'foo_bar'>>,
  Expect<Equal<CamelCase<'foo--bar----baz'>, 'foo-Bar---Baz'>>,
  Expect<Equal<CamelCase<'a-b-c'>, 'aBC'>>,
  Expect<Equal<CamelCase<'a-b-c-'>, 'aBC-'>>,
  Expect<Equal<CamelCase<'ABC'>, 'ABC'>>,
  Expect<Equal<CamelCase<'-'>, '-'>>,
  Expect<Equal<CamelCase<''>, ''>>,
  Expect<Equal<CamelCase<'😎'>, '😎'>>,
]
```

将烤串转驼峰

### 测试用例

- `Equal<CamelCase<'foo-bar-baz'>, 'fooBarBaz'>`

正常的将烤串语法转为驼峰语法

- `Equal<CamelCase<'foo-Bar-Baz'>, 'foo-Bar-Baz'>`

如果相应需要转换的字母已经是大写了，则不进行转换

- `Equal<CamelCase<'foo_bar'>, 'foo_bar'>`

只会对**中划线**进行转换

- `Equal<CamelCase<'foo--bar----baz'>, 'foo-Bar---Baz'>`

多个中划线，只会删除一个后将其后方最近的一个字母转为大写

- `Equal<CamelCase<'a-b-c-'>, 'aBC-'>`

如果中划线后方没有字母了，则不会对其进行操作



### 代码实现

在进行功能实现之前，我们先来认识一个TS函数：`Capitalize`（将首字母转大写）

```ts
/**
 * Convert first character of string literal type to uppercase
 */
type Capitalize<S extends string> = intrinsic;
```

- 原代码

```ts
type CamelCase<S> = any;
```

- 传入的 `S` 是一个字符串

```ts
type CamelCase<S extends string> = any;
```

- 将其根据中划线（`-`）进行拆分，如果能拆出来则进行下一步操作，拆不出来则直接返回 `S`

```ts
type CamelCase<S extends string> = S extends `${infer F}-${infer R}` 
	? ...
	: S
```

- 判断拆分处理的右侧字符串，首字母是否是**大写**。
  - 如果是大写，则**不删除中划线**直接对右侧字符串**递归调用CamelCase**
  - 如果不是大写，则**删除中划线**，并将右侧首字母改为大写后，再递归 `CamelCase`

```ts
type CamelCase<S extends string> = S extends `${infer F}-${infer R}` 
	? R extends Capitalize<R>
		? `${F}-${CamelCase<R>}`
		: `${F}${CamelCase<Capitalize<R>>}`
	: S
```

















