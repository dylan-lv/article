## 题目二十五：append-to-object

```ts
// template.ts
type AppendToObject<T, U, V> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type test1 = {
  key: 'cat'
  value: 'green'
}

type testExpect1 = {
  key: 'cat'
  value: 'green'
  home: boolean
}

type test2 = {
  key: 'dog' | undefined
  value: 'white'
  sun: true
}

type testExpect2 = {
  key: 'dog' | undefined
  value: 'white'
  sun: true
  home: 1
}

type test3 = {
  key: 'cow'
  value: 'yellow'
  sun: false
}

type testExpect3 = {
  key: 'cow'
  value: 'yellow'
  sun: false
  isMotherRussia: false | undefined
}


type cases = [
  Expect<Equal<AppendToObject<test1, 'home', boolean>, testExpect1>>,
  Expect<Equal<AppendToObject<test2, 'home', 1>, testExpect2>>,
  Expect<Equal<AppendToObject<test3, 'isMotherRussia', false | undefined>, testExpect3>>,
]
```



### 测试用例

- 示例1

`test1` 与 `testExpect1` 对比，`testExpect1` 多了一个参数 ：`home: boolean`

而 `AppendToObject` 方法的第二个参数 `home` 对应多出参数的**键**，第三个参数则对应多出参数的**值**

- 示例2

通过**示例1**可知对应的值可以是**类型**，通过**示例2**可知对应的值也可以是**具体的数值**

- 示例3

通过**示例3**可知对应的值可以是**联合类型**



### 代码实现

- 原代码

```ts
type AppendToObject<T, U, V> = any
```

- `T` 是一个 `object` 的键值对

```ts
type AppendToObject<T extends Record<string, unknown>, U, V> = any
```

- `U` 是一个 `string` 类型

```ts
type AppendToObject<T extends Record<string, unknown>, U extends string, V> = any
```

- 使用 `P in keyof` 的方式来遍历 `键值对`

```ts
type AppendToObject<T extends Record<string, unknown>, U extends string, V> = {
  [P in keyof T]: T[P]
}
```

- 对 `T` 的遍历过程中添加一项 `U`

此时，我们完成了对 `T` 的遍历（此方式之前使用过）；在对 `T` 的遍历过程中，我们需要将 `U` 也添加进去

```ts
type AppendToObject<T extends Record<string, unknown>, U extends string, V> = {
  [P in keyof T | U]: T[P]
}
```

上面的代码相当于

```ts
type AppendToObject<T extends Record<string, unknown>, U extends string, V> = {
  [P in ((keyof T) | U)]: T[P]
}
```

`P` 是从 `keyof T` 和 `U` 两项中取值的，其中 `keyof T` 又是 `T` 的键集合

- 对遍历的 `key` 内容处理完成后，接着对 `value` 进行处理

```ts
type AppendToObject<T extends Record<string, unknown>, U extends string, V> = {
  [P in ((keyof T) | U)]: T[P] extends U ? V : T[P]
}
```







