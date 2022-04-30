## 题目四十二：omitbytype

```ts
// template.ts
type OmitByType<T, U> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

interface Model {
  name: string
  count: number
  isReadonly: boolean
  isEnable: boolean
}

type cases = [
  Expect<Equal<OmitByType<Model, boolean>, { name: string; count: number }>>,
  Expect<Equal<OmitByType<Model, string>, { count: number; isReadonly: boolean; isEnable: boolean }>>,
  Expect<Equal<OmitByType<Model, number>, { name: string; isReadonly: boolean; isEnable: boolean }>>,
]
```

从 `T` 中，选择一组类型不可分配给 `U` 的属性。



### 测试用例

- `OmitByType<Model, boolean>`

从 `Model` 中，去除值为 `boolean` 的内容，期望结果是：`{ name: string; count: number; }`

- `OmitByType<Model, string>`

从 `Model` 中，去除值为 `string` 的内容，期望结果是：`{ count: number; isReadonly: boolean; isEnable: boolean }`

- `OmitByType<Model, number>`

从 `Model` 中，去除值为 `number` 的内容，期望结果是：`{ name: string; isReadonly: boolean; isEnable: boolean }`



### 代码实现

- 原代码

```ts
type OmitByType<T, U> = any
```

- 对 `T` 进行遍历

```ts
type OmitByType<T, U> = {
  [P in keyof T]: T[P]
}
```

- 将当前遍历项的 `key` 转换为 `never` 或 `P`

在转换之前判断一下 `T[P]` 是否 `extends U` 

```ts
type OmitByType<T, U> = {
  [P in keyof T as T[P] extends U ? never : P]: T[P]
}
```



