## 题目二十：capitalize

```ts
// template.ts
type MyCapitalize<T extends string> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<MyCapitalize<'foobar'>, 'Foobar'>>,
  Expect<Equal<MyCapitalize<'FOOBAR'>, 'FOOBAR'>>,
  Expect<Equal<MyCapitalize<'foo bar'>, 'Foo bar'>>,
  Expect<Equal<MyCapitalize<''>, ''>>,
  Expect<Equal<MyCapitalize<'a'>, 'A'>>,
  Expect<Equal<MyCapitalize<'b'>, 'B'>>,
]
```

实现 `Capitalize<T>` 它将字符串的第一个字母转换为大写，其余字母保持原样。

### 代码实现

- 原代码

```ts
type MyCapitalize<T extends string> = any
```

- 从字符串中分解出首字母

```ts
type MyCapitalize<T extends string> = T extends `${infer First}${infer Rest}`
```

- 转换大写 `Uppercase`

```ts
type MyCapitalize<T extends string> = T extends `${infer First}${infer Rest}`
	? `${Uppercase<First>}${Rest}`
	: T
```



### Uppercase

来看一下 `TS` 内置方法 `Uppercase` 的实现

```ts
type Uppercase<S extends string> = intrinsic;
```

可以看到，这里只是简单的给了 `intrinsic` 这个关键字，就如它的含义一样，`intrinsic` 是 `TS` 内部用到的。

和其它TS提供的内置类型一样，`Uppercase` 这几个内置类型也是为了方便类型书写，但是专门针对字符串类型<字符串字面量、模板字符串>而提供的。

它们的共同特点是，用它们生成的类型涉及到了`值`的转换，而不是`类型`的转换，而这在TS里通过已有的类型书写方式是无法表达、也不太适合去表达的。

所以 TS 只能以内置关键字 `instrinsic` 来通过编译期来实现

> 官方解释：Intrinsic String Manipulation Types
>
> 为了帮助进行字符串操作，`TypeScript` 包括一组可用于字符串操作的类型。
>
> 为了提高性能，这些类型内置于编译器中，而在 `.d.ts`文件中无法找到













