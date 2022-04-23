## 题目三十七：pickbytype

```ts
// template.ts
type PickByType<T, U> = any
```

```ts
// test-case.ts
import type { Equal, Expect } from '@type-challenges/utils'

interface Model {
  name: string
  count: number
  isReadonly: boolean
  isEnable: boolean
}

type cases = [
  Expect<Equal<PickByType<Model, boolean>, { isReadonly: boolean; isEnable: boolean }>>,
  Expect<Equal<PickByType<Model, string>, { name: string }>>,
  Expect<Equal<PickByType<Model, number>, { count: number }>>,
]
```

从“T”中，选择一组类型可分配给“U”的属性。



### 测试用例

- `Equal<PickByType<Model, boolean>, { isReadonly: boolean; isEnable: boolean }>`

从 `Model` 中拿出值为 `boolean` 的内容，期望结果为 ：`{ isReadonly: boolean; isEnable: boolean }`

- `Equal<PickByType<Model, string>, { name: string }>`

从 `Model` 中拿出值为 `string` 的内容，期望结果为 ：`{ name: string }`

- `Equal<PickByType<Model, number>, { count: number }>`

从 `Model` 中拿出值为 `number` 的内容，期望结果为 ：`{ count: number }`



### 代码实现

- 原代码

```ts
type PickByType<T, U> = any
```

- 因为我们需要从 `T` 中获取特定的内容，那么我们先对 `T` 进行遍历

```ts
type PickByType<T, U> = {
  [P in keyof T]: T[P]
}
```

- 遍历过程中，需要去除一些不必要的项，只留下存在于 `U` 中相对应的项

做法：将 `keyof T` 进行转换 （`as`）

看看值（`T[P]`）是否在 `U` 中，如果存在则正常返回 `P`，不存在则返回 `never`（返回 `never` 的话，当前循环项就不会出现在结果中了）

```ts
type PickByType<T, U> = {
  [P in keyof T as T[P] extends U ? P : never]: T[P]
}
```



















