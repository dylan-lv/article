## 题目四十一：mutable

```ts
// template.ts
type Mutable<T> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

interface Todo1 {
  title: string
  description: string
  completed: boolean
  meta: {
    author: string
  }
}

type cases = [
  Expect<Equal<Mutable<Readonly<Todo1>>, Todo1>>,
]
```

实现通用的`Mutable<T>`，它使`T`中的所有属性都是可变的（而不是只读的）。



### 代码实现

- 原代码

```ts
type Mutable<T> = any
```

- 遍历对象

```ts
type Mutable<T> = {
  [P in keyof T]: T[P]
}
```

- 在遍历过程中，祛除 `readonly` 修饰符

```ts
type Mutable<T> = {
  - readonly [P in keyof T]: T[P]
}
```





