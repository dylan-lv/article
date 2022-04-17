## 题目十六：chainable-options

```ts
// template.ts
type Chainable = {
  option(key: string, value: any): any
  get(): any
}
```

```ts
// test-cases.ts
import { Alike, Expect } from '@type-challenges/utils'

declare const a: Chainable

const result1 = a
  .option('foo', 123)
  .option('bar', { value: 'Hello World' })
  .option('name', 'type-challenges')
  .get()

const result2 = a
  .option('name', 'another name')
  // @ts-expect-error
  .option('name', 'last name')
  .get()

type cases = [
  Expect<Alike<typeof result1, Expected1>>,
  Expect<Alike<typeof result2, Expected2>>,
]

type Expected1 = {
  foo: number
  bar: {
    value: string
  }
  name: string
}

type Expected2 = {
  name: string
}
```

### 可串联构造器

在 JavaScript 中我们很常会使用可串联（Chainable/Pipeline）的函数构造一个对象，但在 TypeScript 中，你能合理的给他附上类型吗？

在这个挑战中，你需要提供两个函数 `option(key, value)` 和 `get()`。在 `option` 中你需要使用提供的 key 和 value 扩展当前的对象类型，通过 `get` 获取最终结果。

可以假设 `key` 只接受字符串而 `value` 接受任何类型，你只需要暴露它传递的类型而不需要进行任何处理。同样的 `key` 只会被使用一次。



### 测试用例

- 示例1

```ts
const result1 = a
  .option('foo', 123)
  .option('bar', { value: 'Hello World' })
  .option('name', 'type-challenges')
  .get()
```

最终的结果期望是：

```ts
type Expected1 = {
  foo: number
  bar: {
    value: string
  }
  name: string
}
```

相当于用链式的方式，将参数注入相应的对象

- 示例2

```ts
const result2 = a
  .option('name', 'another name')
  // @ts-expect-error
  .option('name', 'last name')
  .get()
```

期望结果：

```ts
type Expected2 = {
  name: string
}
```

相同 `key` 的内容再次注入会报错



### 代码实现

- 原代码

```ts
type Chainable = {
  option(key: string, value: any): any
  get(): any
}
```

- 传入默认值（对象）

```ts
type Chainable<T = {}> = {
  option(key: string, value: any): any
  get(): any
}
```

- `key` 需要是一个字符串

```ts
type Chainable<T = {}> = {
  option<K extends: string>(key: K, value: any): any
  get(): any
}
```

- `value` 值随意

```ts
type Chainable<T = {}> = {
  option<K extends: string, V>(key: K, value: V): any
  get(): any
}
```

- `option` 的返回值是一个 `Chainable`，在类型 `T` 的基础上添加传入 `option` 的类型

```ts
type Chainable<T = {}> = {
  option<K extends: string, V>(key: K, value: V): Chainable<T & Record<K, V>>
  get(): any
}
```

- `get` 直接返回**现有类型** `T`

```ts
type Chainable<T = {}> = {
  option<K extends: string, V>(key: K, value: V): Chainable<T & Record<K, V>>
  get(): T
}
```

- 相同 `key` 报错

利用 `extends` 来判断 `K` 是否继承 `T`，如果继承则返回 `never`，否则可以使用 `K`（返回 `K`）

```ts
type Chainable<T = {}> = {
  option<K extends: string, V>(key: K extends T ? never : K, value: V): Chainable<T & Record<K, V>>
  get(): T
}
```

