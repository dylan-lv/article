## 题目六：exclude

```ts
// template.ts
type MyExclude<T, U> = any
```

```ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
    Expect<Equal<MyExclude<"a" | "b" | "c", "a">, Exclude<"a" | "b" | "c", "a">>>,
    Expect<Equal<MyExclude<"a" | "b" | "c", "a" | "b">, Exclude<"a" | "b" | "c", "a" | "b">>>,
    Expect<Equal<MyExclude<string | number | (() => void), Function>, Exclude<string | number | (() => void), Function>>>,
]
```

题目要求我们实现一个 `Exclude`，需要和系统内置的 `Exclude` 功能一致。

那么系统内置的 `Exclude` 有什么功能呢？

我们把测试用例赋值出来，看看最终的结果是什么样的

```ts
type t1 = Exclude<"a" | "b" | "c", "a"> // type t1 = "b" | "c"
type t2 = Exclude<"a" | "b" | "c", "a" | "b"> // type t2 = "c"
type t3 = Exclude<string | number | (() => void), Function> // type t3 = string | number
```

现在基本可以确定 `Exclude` 的效果是：从第一个参数中剔除掉第二个参数存在的数据。



### 使用 extends 进行循环

通过我们之前的题目，可以知道 `union` 类型是默认可以循环的：`"a" | "b"`这种就是 `union` 类型

可以这样来写：

```ts
type MyExclude<T, U> = T extends U
```

注意这里一定是先写 `T`，后写 `U`。相当于先循环 `T` 再循环 `U`。

它的实际上是一个三元表达式，后面需要跟一个 A 和 B：

```ts
type MyExclude<T, U> = T extends U ? A : B
```

来看看它的执行方式：假设 `T = "a" | "b" | "c"`，`U = "a"`

对比的时候是依次进行对比的，和我们 `js` 中的循环是一样的

- `"a"` 同 `"a"` 进行对比：如果匹配上，就返回 `A` ，否则返回 `B`
- `"b"` 同 `"a"` 进行对比：如果匹配上，就返回 `A` ，否则返回 `B`
- `"c"` 同 `"a"` 进行对比：如果匹配上，就返回 `A` ，否则返回 `B`

这样，我们得到的结果就是 A B B。（第一个匹配上了，返回了 `A`）

又因为，如果循环的过程中匹配上的话，我们是不希望返回这个元素的，所以我们把 `A` 替换为 `never` ；把 `B` 替换为 `T`

最终结果：

```ts
type MyExclude<T, U> = T extends U ? never : T
```



### 多个对比

一个对比和多个对比其实都是一样的，现在我们把 `U` 假设为 `"a" | "b"`

那么现在就是：``T = "a" | "b" | "c"`，`U = "a" | "b"`

- 第一轮：
  - `"a"` 同 `"a"` 进行对比
  - `"b"` 同 `"a"` 进行对比
  - `"c"` 同 `"a"` 进行对比
- 第二轮：
  - `"a"` 同 `"b"` 进行对比
  - `"b"` 同 `"b"` 进行对比
  - `"c"` 同 `"b"` 进行对比

匹配的结果是：ABB 和 BAB。

第一轮把 `a` 匹配上了，第二轮把 `b` 匹配上了，所以最终的结果就只剩一个 `c` 了





















