## 题目十四：readonly-2

```ts
// template.ts
type MyReadonly2<T, K> = any
```

```ts
// test-cases.ts
import { Alike, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>,
  Expect<Alike<MyReadonly2<Todo1, 'title' | 'description'>, Expected>>,
  Expect<Alike<MyReadonly2<Todo2, 'title' | 'description'>, Expected>>,
]

interface Todo1 {
  title: string
  description?: string
  completed: boolean
}

interface Todo2 {
  readonly title: string
  description?: string
  completed: boolean
}

interface Expected {
  readonly title: string
  readonly description?: string
  completed: boolean
}
```



实现一个通用`MyReadonly2<T, K>`，它带有两种类型的参数`T`和`K`。

`K`指定应设置为Readonly的`T`的属性集。如果未提供`K`，则应使所有属性都变为只读，就像普通的`Readonly<T>`一样。



### 测试代码

- 示例1

```ts
interface Todo1 {
  title: string
  description?: string
  completed: boolean
}
Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>
```

希望 `MyReadonly2<Todo1>` 和 `Readonly<Todo1>` 是相同的。

`MyReadonly2` 期望传入**一个或两个**参数。如果传入一个参数的话，效果和 `Readonly` 一致，会将全部内容转换为 `readonly`

- 示例2

```ts
interface Expected {
  readonly title: string
  readonly description?: string
  completed: boolean
}
Expect<Alike<MyReadonly2<Todo1, 'title' | 'description'>, Expected>>
```

希望 `MyReadonly2<Todo1, 'title' | 'description'>` 和 `Expected` 是相同的。

说明 `MyReadonly2` 的功能就是：将参数1中含有参数2的内容都转换成**只读**的。



### 代码实现

- 原代码

```ts
type MyReadonly2<T, K> = any
```

- `K`可以是任意值的 `key`，和上一题 `Omit` 一样

```ts
type MyReadonly2<T, K extends keyof any> = any
```

- 遍历拿出参数2中存在于参数1的内容，并加上修饰符 `readonly`

```ts
type MyReadonly2<T, K extends keyof any> = {
  readonly [P in keyof T as P extends K ? P : never]: T[P]
}
```

上述代码我们在 `Omit` 这题中讲过，只不过 `Omit` 是过滤掉参数2中的内容，这里颠倒了一下，拿出参数2中的内容；

并且加上了 `readonly` 修饰符

- 使用 `Omit` 拿出另外一部分

上面我们已经拿出了存在于参数2 中的内容，现在我们使用上一题中实现的 `Omit` 来拿出不存在于参数2中的内容

```ts
Omit<T, K>
```

-  `&` 类型合并

我们已经拿出了：

1. 加上了 `readonly` 的部分（存在于参数2中）
2. 不存在于参数2中的部分

现在，我们只需要将两部分合并即可

```ts
type MyReadonly2<T, K extends keyof any> = {
  readonly [P in keyof T as P extends K ? P : never]: T[P]
} & Omit<T, K>
```

- `K` 的默认值

如果没有传入参数2，则 `MyReadonly2` 需要和 `Readonly` 同样的效果，即将全部内容转换为 `readonly` 的

这里，我们给参数2一个默认值来实现这个效果。

根据上面的代码实现，`K` 的默认值应该和 `T` 相等，这样才能将 `T` 中的全部内容进行转换

```ts
type MyReadonly2<T, K extends keyof any = keyof T> = {
  readonly [P in keyof T as P extends K ? P : never]: T[P]
} & Omit<T, K>
```







