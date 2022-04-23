## 题目三十五：percentage-parser

```ts
// template.ts
type PercentageParser<A extends string> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type Case0 = ['', '', '']
type Case1 = ['+', '', '']
type Case2 = ['+', '1', '']
type Case3 = ['+', '100', '']
type Case4 = ['+', '100', '%']
type Case5 = ['', '100', '%']
type Case6 = ['-', '100', '%']
type Case7 = ['-', '100', '']
type Case8 = ['-', '1', '']
type Case9 = ['', '', '%']
type Case10 = ['', '1', '']
type Case11 = ['', '100', '']

type cases = [
  Expect<Equal<PercentageParser<''>, Case0>>,
  Expect<Equal<PercentageParser<'+'>, Case1>>,
  Expect<Equal<PercentageParser<'+1'>, Case2>>,
  Expect<Equal<PercentageParser<'+100'>, Case3>>,
  Expect<Equal<PercentageParser<'+100%'>, Case4>>,
  Expect<Equal<PercentageParser<'100%'>, Case5>>,
  Expect<Equal<PercentageParser<'-100%'>, Case6>>,
  Expect<Equal<PercentageParser<'-100'>, Case7>>,
  Expect<Equal<PercentageParser<'-1'>, Case8>>,
  Expect<Equal<PercentageParser<'%'>, Case9>>,
  Expect<Equal<PercentageParser<'1'>, Case10>>,
  Expect<Equal<PercentageParser<'100'>, Case11>>,
]
```

实现PercentageParser `<T extends string>`。

根据`/^（\+\\-）？（\d*）？(\%)?$/` 规律性匹配T并获得三个匹配。

结构应该是：[‘正负’、‘数字’、‘单位`]

如果未捕获，则默认为空字符串。



### 代码实现

- 原代码

```ts
type PercentageParser<A extends string> = any
```

- 设计函数 `MatchSymbol`，功能：从字符串中抽取符号

```ts
type MatchSymbol<T> = T extends `${infer F}${infer _}`
	? F extends '+' | '-'
		? F
		: ''
	: ''
```

- 设计函数 `MatchUnit`，功能：从字符串中抽取单位

```ts
type MatchUnit<T> = T extends `${infer _}%` ? '%' : ''
```

- 设计函数 `MatchValue`，功能：从字符串中抽取数值

```ts
type MatchValue<T> = T extends `${MatchSymbol<T>}${infer V}${MatchUnit<T>}` ? V : ''
```

- 最后实现 `PercentageParser`

```ts
type PercentageParser<A extends string> = [MatchSymbol<A>, MatchValue<A>, MatchUnit<A>]
```

















