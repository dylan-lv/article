## 题目二十六：merge

```ts
// template.ts
type Merge<F, S> = any;
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type Foo = {
  a: number;
  b: string;
};
type Bar = {
  b: number;
  c: boolean;
};

type cases = [
  Expect<Equal<Merge<Foo, Bar>, {
	a: number;
	b: number;
	c: boolean;
  }>>
]
```

实现 `Merge` 函数，该函数将两个类型合并成一个类型，第二个类型的键会覆盖第一个类型的键。



### 代码实现：方式1

- 原代码

```ts
type Merge<F, S> = any;
```

- 遍历对象

```ts
type Merge<F, S> = {
  [P in keyof F]: ...
};
```

- 遍历多个对象

```ts
type Merge<F, S> = {
  [P in keyof F | keyof S]: ...
};
```

- `S`的优先级更高，所以优先返回 `S` 的值

```ts
type Merge<F, S> = {
  [P in keyof F | keyof S]: P extends keyof S
  	? S[P]
  	: ...
};
```

- 接下来处理 `F` 的内容

```ts
type Merge<F, S> = {
  [P in keyof F | keyof S]: P extends keyof S
  	? S[P]
  	: P extends keyof F
  		? F[P]
  		: never
};
```



### 代码实现：方式2

- 原代码

```ts
type Merge<F, S> = any;
```

- 新建一个参数 `K`，`K` 是 `F` 和 `S` 中的相同项，例如测试用例中的参数 `b`

```ts
type Merge<F, S, K extends keyof F & keyof S> = any;
```

- 给默认值

```ts
type Merge<F, S, K extends keyof F & keyof S = keyof F & keyof S> = any;
```

- 使用 `Omit`提取出不存在于公共项的内容

`Omit`：参数1（对象），参数2（key），作用是去除掉参数1中 `key` 为参数2 的内容

```ts
type Merge<F, S, K extends keyof F & keyof S = keyof F & keyof S> = Omit<F, K> & Omit<S, K>;
```

此时，已经拿到了不重复内容的联合了，如测试用例中的：`{ a: number, c: boolean }`

- 将重复的部分联合进去

```ts
type Merge<F, S, K extends keyof F & keyof S = keyof F & keyof S> = Omit<F, K> & Omit<S, K> & Pick<S, K>;
```

`Pick`：从 `S` 中取出 `K`

- 最后将其作为一个整体返回出去

```ts
type Merge<F, S, K extends keyof F & keyof S = keyof F & keyof S> = Omit<Omit<F, K> & Omit<S, K> & Pick<S, K>, never>;
```

