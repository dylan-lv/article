## 题目三十：anyof

```ts
// template.ts
type AnyOf<T extends readonly any[]> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<AnyOf<[1, 'test', true, [1], { name: 'test' }, { 1: 'test' }]>, true>>,
  Expect<Equal<AnyOf<[1, '', false, [], {}]>, true>>,
  Expect<Equal<AnyOf<[0, 'test', false, [], {}]>, true>>,
  Expect<Equal<AnyOf<[0, '', true, [], {}]>, true>>,
  Expect<Equal<AnyOf<[0, '', false, [1], {}]>, true>>,
  Expect<Equal<AnyOf<[0, '', false, [], { name: 'test' }]>, true>>,
  Expect<Equal<AnyOf<[0, '', false, [], { 1: 'test' }]>, true>>,
  Expect<Equal<AnyOf<[0, '', false, [], { name: 'test' }, { 1: 'test' }]>, true>>,
  Expect<Equal<AnyOf<[0, '', false, [], {}]>, false>>,
  Expect<Equal<AnyOf<[]>, false>>,
]
```

在类型系统中实现类似`Python`的`'any'`函数。该类型接受数组，如果数组中的任何元素为`true`，则返回`'true'`。如果数组为空，则返回`'false'`。



### 测试用例

通过测试用例可知：

- 该函数传入一个数组
- 如果数组中全部的值都是：`0 | '' | false | [] | {}` 这几个值中的，就返回 `false`
- 否则返回 `true`



### 代码实现

- 原代码

```ts
type AnyOf<T extends readonly any[]> = any
```

- 将上述会返回 `false` 的值设置为联合类型

```ts
type AnyOfFalse = 0 | "" | false | [] | { [key: string]: never }
type AnyOf<T extends readonly any[]> = any
```

- 使用 `T[number]` 进行数组遍历

```ts
type AnyOfFalse = 0 | "" | false | [] | { [key: string]: never }
type AnyOf<T extends readonly any[]> = T[number]
```

- 如果 `T[number]` 全部的值都继承自 `AnyOfFalse`，则返回 `false`，否则返回`true`（至少有一个不继承自 `AnyOfFalse`）

```ts
type AnyOfFalse = 0 | "" | false | [] | { [key: string]: never }
type AnyOf<T extends readonly any[]> = T[number] extends AnyOfFalse ? false : true
```



