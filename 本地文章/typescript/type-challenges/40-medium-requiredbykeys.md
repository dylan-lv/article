## 题目四十：requiredbykeys

```ts
// template.ts
type RequiredByKeys<T, K> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

interface User {
  name?: string
  age?: number
  address?: string
}

interface UserRequiredName {
  name: string
  age?: number
  address?: string
}

interface UserRequiredNameAndAge {
  name: string
  age: number
  address?: string
}

type cases = [
  Expect<Equal<RequiredByKeys<User, 'name'>, UserRequiredName>>,
  Expect<Equal<RequiredByKeys<User, 'name' | 'unknown'>, UserRequiredName>>,
  Expect<Equal<RequiredByKeys<User, 'name' | 'age'>, UserRequiredNameAndAge>>,
  Expect<Equal<RequiredByKeys<User>, Required<User>>>,
]
```

实现一个通用的 `RequiredByKeys<T，K>`，它接受两种类型的参数 `T` 和 `K`。

`K`指定应设置为必需的`T`属性集。当没有提供` K` 时，它应该使所有所需的属性都像正常的 `required<T>` 一样。



### 测试用例

`User` 中有三项，都是可选属性

- `RequiredByKeys<User, 'name'>`

将 `User` 中 `key` 为 `name` 的项改为必填的

- `RequiredByKeys<User, 'name' | 'unknown'>`

`unknown` 会被忽略

- `RequiredByKeys<User, 'name' | 'age'>`

如果传入的是联合属性，则将联合属性中的每一项都改为必填

- `RequiredByKeys<User>`

如果没传第二个参数，则和 `Required` 方法效果一致



### required

我们先来看一下 `TS` 内置的 `required` 方法

```ts
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

 作用：将全部项都外给必填（去掉可选 `?`）



### 代码实现

- 原代码

```ts
type RequiredByKeys<T, K> = any
```

- `K` 的默认值是 `T` 的全部 `key`

```ts
type RequiredByKeys<T, K = keyof T> = any
```

- 拿出 `T` 中 `key` 为 `K` 的内容

```ts
type RequiredByKeys<T, K = keyof T> = Pick<T, K & keyof T>
```

- 将其转换为必填的

```ts
type RequiredByKeys<T, K = keyof T> = Required<Pick<T, K & keyof T>>
```

- 与剩余的 `T` 中的内容合并

```ts
type RequiredByKeys<T, K = keyof T> = Required<Pick<T, K & keyof T>>
	& Omit<T, K & keyof T>
```

- 因为使用了合并，所以需要在复制一下

```ts
type Copy<T> = {
  [P in keyof T]: T[P]
}
type RequiredByKeys<T, K = keyof T> = Copy<
  Required<Pick<T, K & keyof T>> 
  & Omit<T, K & keyof T>
>
```



















