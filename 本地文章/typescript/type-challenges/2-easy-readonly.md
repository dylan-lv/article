## 题目二：easy-readonly

```ts
// type-challenges\7-easy-readonly\template.ts
type MyReadonly<T> = any
```

```ts
// type-challenges\7-easy-readonly\test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<MyReadonly<Todo1>, Readonly<Todo1>>>,
]

interface Todo1 {
  title: string
  description: string
  completed: boolean
  meta: {
    author: string
  }
}
```

在这道题中，我们实现的 `MyReadonly` 需要和系统实现的 `Readonly` 效果一致。

效果是调用 `MyReadonly<Todo1>` 后得到的类型会将 Todo1 中的所有类型都加上 `readonly` 修饰符：

```ts
type Type3 = MyReadonly<Todo1>
```

等同于下面的效果：

```ts
type Type3 = {
    readonly title: string;
    readonly description: string;
    readonly completed: boolean;
    readonly meta: {
        author: string;
    };
}
```

我们为了实现上面的效果，需要做如下这些工作：

- 返回一个对象

- 遍历传入的对象（接口）

- 加上 `readonly` 关键字

- 通过 `key` 获取传入对象（接口）的值，接口的值就是类型

  

### 1. 返回一个对象

```ts
type MyReadonly<T> = {  }
```



### 2. 遍历接口

使用 `in` 来遍历接口；

使用 `keyof` 获取 `T` 中的全部 `key`

```ts
type MyReadonly<T> = {
  [P in keyof T]: any
}
```



### 3.  获取对应的值

和 `js` 类似，通过 `T[P]` 就可以获取到传入接口的相应值（类型）了。类似于 `obj[key]`

```ts
type MyReadonly<T> = {
  [P in keyof T]: T[P]
}
```



### 4. 加上  `readonly` 关键字

```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]
}
```







































