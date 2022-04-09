## 题目四：first-of-array

取出数组的第一个元素

```ts
// template.ts
type First<T extends any[]> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<First<[3, 2, 1]>, 3>>,
  Expect<Equal<First<[() => 123, { a: string }]>, () => 123>>,
  Expect<Equal<First<[]>, never>>,
  Expect<Equal<First<[undefined]>, undefined>>
]

type errors = [
  // @ts-expect-error
  First<'notArray'>,
  // @ts-expect-error
  First<{ 0: 'arrayLike' }>
]
```

知识点：

1. `extends` 类型条件判断
2. 获取 `tuple` 的 `length` 属性
3. `extends union` 判断的规则
4. `infer` 的使用



### 观察测试代码

- 数组 `[3,2,1]` 希望取出 3

```ts
Expect<Equal<First<[3, 2, 1]>, 3>>
```

- 数组 `[()=>123, {a:string}]`，希望取出函数 `()=>123`

```ts
Expect<Equal<First<[() => 123, { a: string }]>, () => 123>>
```

- 空数组，希望取出 `never`

```ts
Expect<Equal<First<[]>, never>>
```

- 唯一元素数组 `[undefined]`，取出唯一元素 `undefined`

```ts
Expect<Equal<First<[undefined]>, undefined>>
```

- 传入元素不是数组，希望报错

```ts
First<'notArray'>
First<{ 0: 'arrayLike' }>
```



### 解法一

取数组的第一个元素，直接赋值 `T[0]`

```ts
type First<T extends any[]> = T[0]
```

此时，回头看测试代码，可以发现只有一条报错了：`Expect<Equal<First<[]>, never>>`

推测：如果传入是一个空数组的话，直接使用 `T[0]` 的返回值并不是 `never`。

我们来看一下它的返回值究竟是什么：

```ts
type t0 = First<[]> // type t0 = undefined
```

可以看到它实际上返回了 `undefined`

那么在 `ts` 中应该如何去判断数组是否为空呢？

我们可以使用 `extends` 来判断

```ts
type First<T extends any[]> = T extends [] ? never : T[0]
```

`T extends []` 相当于**判断 `T` 是否为空数组**



### 解法二

可以通过判断传入对象的 `length` 属性来知道它是否是空数组

- `T["length"]`：直接通过这种方式来获取 `T` 的 `length` 值

```ts
type First<T extends any[]> = T["length"] extends 0 ? never : T[0]
```



### 解法三

我们之前通过 `T[number]` 这种写法，它将数组中的全部元素都遍历了一遍。

```ts
type ages = [1,2,3]
type t0 = ages[number] // type t0 = 3 | 1 | 2
```

可以看到，`t0` 是一个 `union` 联合类型

我们可以组合 `union` 和 `extends` 来得到解：看看某个值是否在 `union` 中

```ts
type ages = [1,2,3]
type t1 = 1 extends ages[number] ? "true" : "false" // type t1 = "true"
```

上述代码中的 `1` 会依次和 `age[number]` 这个联合类型中的内容去对比：`1和1对比`、`1和2对比`、`1和3对比`，只要有一个命中，就返回 `true`

如果 `ages[number]` 没有值的话，会返回 `never`

```ts
type ages = []
type t2 = ages[number] // type t2 = never
```

此时，我们可以知道

- `T[0]` 没有的话会返回 `undefined`
- `ages[number]` 没有的话会返回 `never`

**题解**

`T[0]` 如果在 `T[number]` 中，就返回 `T[0]`，否则返回 `never`

```ts
type First<T extends any[]> = T[0] extends T[number] ? T[0] : never
```



### 解法四

使用 `infer` 进行推断

在 `js` 中，我们可以通过如下方式获取到数组的第一个元素：

```ts
const getFirst = (arr) => {
  const [first, ...rest] = arr;
  return first ? first : "never";
}
```

上述的方式，在 `ts` 类型中可以使用 `infer` 来表示

```ts
type First<T extends any[]> = T extends [infer first, ...infer rest] ? first : never
```

- `infer`（推断）：可以理解为声明一个变量

逻辑：看看能否解构出一个 `first`，如果能的话返回 `first`，否则返回 `never`。

**扩展**

基于这种方式，我们可以扩展一下：获取数组的第一个元素之后的所有元素

```ts
type Tail<T extends any[]> = T extends [infer first, ...infer rest] ? rest : never
type t0 = Tail<[1,2,3]> // type t0 = [2, 3]
```

此时，`t0` 的结果就是 `[2, 3]`













