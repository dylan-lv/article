## 题目二十二：replaceall

```ts
// template.ts
type ReplaceAll<S extends string, From extends string, To extends string> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<ReplaceAll<'foobar', 'bar', 'foo'>, 'foofoo'>>,
  Expect<Equal<ReplaceAll<'foobar', 'bag', 'foo'>, 'foobar'>>,
  Expect<Equal<ReplaceAll<'foobarbar', 'bar', 'foo'>, 'foofoofoo'>>,
  Expect<Equal<ReplaceAll<'t y p e s', ' ', ''>, 'types'>>,
  Expect<Equal<ReplaceAll<'foobarbar', '', 'foo'>, 'foobarbar'>>,
  Expect<Equal<ReplaceAll<'barfoo', 'bar', 'foo'>, 'foofoo'>>,
  Expect<Equal<ReplaceAll<'foobarfoobar', 'ob', 'b'>, 'fobarfobar'>>,
  Expect<Equal<ReplaceAll<'foboorfoboar', 'bo', 'b'>, 'foborfobar'>>,
  Expect<Equal<ReplaceAll<'', '', ''>, ''>>,
]
```

实现 `ReplaceAll<S, From, To>` 将一个字符串 `S` 中的所有子字符串 `From` 替换为 `To`。

我们刚刚已经实现了 `replace` 方法，那么 `replaceall` 是不是可以直接递归就OK了呢？来试试



### 代码实现

- 原代码

```ts
type ReplaceAll<S extends string, From extends string, To extends string> = any
```

- 排除空字符串

```ts
type ReplaceAll<S extends string, From extends string, To extends string> = From extends ""
	? S
	: ...
```

- 如果找到，则解构出 `From`左右的值，否则直接返回 `S`

```ts
type ReplaceAll<S extends string, From extends string, To extends string> = From extends ""
	? S
	: S extends `${infer L}${From}${infer R}`
		? ...
		: S
```

- 将替换后的结果继续用 `ReplaceAll` 处理

```ts
type ReplaceAll<S extends string, From extends string, To extends string> = From extends ""
  ? S
  : S extends `${infer L}${From}${infer R}`
    ? ReplaceAll<`${L}${To}${R}`, From, To>
    : S
```

此时，可以看到**测试用例**中，有两个测试依旧是报错状态

```ts
Expect<Equal<ReplaceAll<'foobarfoobar', 'ob', 'b'>, 'fobarfobar'>>,
Expect<Equal<ReplaceAll<'foboorfoboar', 'bo', 'b'>, 'foborfobar'>>,
```

将测试代入我们的代码逻辑，来看看：

`foobarfoobar` 经过第一次替换后，变为 `fobarfobar`；由于递归的原因，会发现此时替换后的字符串依旧可以匹配上 `ob`，经过第二次替换后变为 `fbarfbar`，明显和我们期望的 `fobarfobar` 不同。

我们应该把代码修改为**从左往右遍历**的方式来依次替换

```ts
type ReplaceAll<S extends string, From extends string, To extends string> = From extends ""
  ? S
  : S extends `${infer L}${From}${infer R}`
    ? `${L}${To}${ReplaceAll<R, From, To>}`
    : S
```

这样就不会再次处理**已经处理过的内容**了。

















