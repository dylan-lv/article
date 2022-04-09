## 题目十三：omit

```ts
// template.ts
type MyOmit<T, K> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Expected1, MyOmit<Todo, 'description'>>>,
  Expect<Equal<Expected2, MyOmit<Todo, 'description' | 'completed'>>>
]

// @ts-expect-error
type error = MyOmit<Todo, 'description' | 'invalid'>

interface Todo {
  title: string
  description: string
  completed: boolean
}

interface Expected1 {
  title: string
  completed: boolean
}

interface Expected2 {
  title: string
}
```

题目要求我们实现 `ts` 的内置类型 `Omit`，可参考[TypeScript官方文档](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys)。

官网解释：通过从类型中选取所有属性，然后移除键（字符串文字或字符串文字的并集）来构造类型。

也就是：参数1是一个对象，参数2是**字符串或字符串的并集**，`Omit` 的作用就是将参数1中 `key` 为参数2的项删掉并返回



- 示例1

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}
type TodoPreview = Omit<Todo, "description">; 
```

`type TodoPreview`的值：

```ts
type TodoPreview = {
    title: string;
    completed: boolean;
    createdAt: number;
}
```

相当于从接口 `Todo` 中删除了 `key` 为 `description` 的这一项



- 示例2

```ts
type TodoInfo = Omit<Todo, "completed" | "createdAt">;
```

`type TodoInfo`的值：

```ts
type TodoInfo = {
    title: string;
    description: string;
}
```

相当于从接口 `Todo` 中删除了 `key` 为 `completed` 和 `createdAt` 的这两项



### 代码实现

- 原始代码

```ts
type MyOmit<T, K> = any
```

- `K` 是 `T` 中的键

可以让 `K` 继承自 `T` 的 `keys`

```ts
type MyOmit<T, K extends keyof T> = any
```

- `object` 的遍历

遍历的内容：`T` 的 `keys`（`keyof T`），值就是原先 `T` 中的值（`T[P]`）

```ts
type MyOmit<T, K extends keyof T> = {
  [P in keyof T]: T[P]
}
```

像上述代码这样的话，会遍历整个 `T`，没有起到过滤掉 `K` 的效果

- 过滤 `K`

使用 `as` 转换，判断 `P` 是否存在于 `K`，如果存在则 `as never`（会跳过当前循环项），否则 `as P`

```ts
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P]
}
```



### ts 内置 Omit写法

自己动手完成了一致，我们再来看一下官方的写法

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

- `K extends keyof any`

通过 `K extends keyof any`，我们可以知道这里其实是不在乎 `K` 继承自什么的（`K`中的内容可以不存在于 `T` 中）

- `Exclude`

我们之前实现过 `Exclude<T, K>`，它的作用就是去除 `T` 中的 `K` 项

```ts
type Exclude<T, K> = T extends K ? never : T;
```

- `Pick`

`Pick` 相当于遍历的作用

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
}
```

- `Pick<T, Exclude<keyof T, K>>`

整体而言，相当于：

1. 先把 `T` 中的 `K` 都给去除掉（假设去除后的内容为 `U`）
2. 再对 `U` 进行遍历

































