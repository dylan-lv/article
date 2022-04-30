## 题目四十三：objectentries

```ts
// template.ts
type ObjectEntries<T> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

interface Model {
  name: string
  age: number
  locations: string[] | null
}

type ModelEntries = ['name', string] | ['age', number] | ['locations', string[] | null]

type cases = [
  Expect<Equal<ObjectEntries<Model>, ModelEntries>>,
  Expect<Equal<ObjectEntries<Partial<Model>>, ModelEntries>>,
  Expect<Equal<ObjectEntries<{ key?: undefined }>, ['key', undefined]>>,
  Expect<Equal<ObjectEntries<{ key: undefined }>, ['key', undefined]>>,
]
```

实现 `TS` 类型版的 `Object.entries`



### 测试用例

- `ObjectEntries<Model>`

将 `Model` 的内容转换为：`[key, value] | [key, value]` 的形式

- `Partial<Model>`

对可选值也不影响，视为不可选一样的效果

- `ObjectEntries<{ key?: undefined }>`

`undefined` 也不影响，直接让值为 `undefined`



### 代码实现

- 原代码

```ts
type ObjectEntries<T> = any
```

- 对 `T` 进行遍历

```ts
type ObjectEntries<T> = {
  [P in keyof T]: T[P]
}
```

- 将值修改为 `[key, value]` 的形式

```ts
type ObjectEntries<T> = {
  [P in keyof T]: [P, T[P]]
}
```

- 将整体转换为联合类型（`obj[key]`） 进行访问

```ts
type ObjectEntries<T> = {
  [P in keyof T]: [P, T[P]]
}[keyof T]
```



这时虽然整体实现了，但是会发现可选参数变成了 `[name, string | undefined]`，但是我们期望的是 `[name, string]`

所以这个实现方式不可取



### 代码实现2

- 原代码

```ts
type ObjectEntries<T> = any
```

- 手动进行遍历，先拿出 `T` 的**键数组**

```ts
type ObjectEntries<T> = keyof T
```

- 使用 `extends` 对 `keyof T` 进行逐个遍历，遍历项定义为 `P`

```ts
type ObjectEntries<T> = keyof T extends infer P
	? ...
	: never
```

- 上面是遍历 `T`，从 `T` 中拿出 `P`；接下来以 `P` 为主体进行判断

```ts
type ObjectEntries<T> = keyof T extends infer P
	? P extends keyof T // 这样才能在下一层中使用 P
		? ...
		: never
	: never
```

- 组装成数组

```ts
type ObjectEntries<T> = keyof T extends infer P
	? P extends keyof T // 这样才能在下一层中使用 P
		? [P, T[P]]
		: never
	: never
```

- 如果 `T[P]` 是可选的话，去除 `undefined`

```ts
type ObjectEntries<T> = keyof T extends infer P
	? P extends keyof T
		? [P, T[P] extends infer R | undefined
      ? R
      : T[P]]
		: never
	: never
```



















