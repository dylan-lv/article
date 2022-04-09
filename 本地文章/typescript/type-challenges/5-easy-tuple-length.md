## 题目五：tuple-length

```ts
// template.ts
type Length<T extends any> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

const tesla = ['tesla', 'model 3', 'model X', 'model Y'] as const
const spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT'] as const
type cases = [
  Expect<Equal<Length<typeof tesla>, 4>>,
  Expect<Equal<Length<typeof spaceX>, 5>>,
  // @ts-expect-error
  Length<5>,
  // @ts-expect-error
  Length<'hello world'>,
]
```

我们需要搞清楚下面两个问题：

- 什么是 `tuple` 类型？
- `tuple` 和普通数组有什么区别？



### 返回 `length`

我们可以直接使用数组下标的方式获取出来

```ts
type Length<T extends any> = T["length"]
```

但是此时会出现错误：`类型 "length" 无法用于索引类型 "T"`。

这是因为 `T` 上面可能没有 `length` 属性。

我们可以使用 `extends`（约束）来达到效果：

```ts
type Length<T extends any[]> = T["length"]
```

现在，`T["length"]` 这里就不会报错了。

接下来看一下测试，会发现测试还是失败的状态。

```ts
Expect<Equal<Length<typeof tesla>, 4>>
```



来看一下报错信息：`类型 "readonly ["tesla", "model 3", "model X", "model Y"]" 为 "readonly"，不能分配给可变类型 "any[]"。`

我们知道通过 `typeof` 一个常量，得到的是一个 `readonly` 的类型，而这个类型是不能赋值给 `any[]` 的

```ts
const tesla = ['tesla', 'model 3', 'model X', 'model Y'] as const
type r = typeof tesla // type r = readonly ["tesla", "model 3", "model X", "model Y"]
```

我们会发现，`typeof tesla` 的结果是一个 `readonly` 的数组，而我们实现的 `Length` 方法接收的参数是 `any[]`

```ts
type Length<T extends any[]> = T["length"]
```

那么解决方式就很明显了，给接收的参数加上 `readonly` 就好啦~

```ts
type Length<T extends readonly any[]> = T["length"]
```

至此，这道题就完成了。



### 什么是 `tuple` 类型？

回到我们开头提到的问题，什么是 `tuple` 类型呢？

我们来看一下官网的定义 [tuple-types](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)。

> 元组类型是另一种数组类型，它确切地知道它包含多少元素，以及它在特定位置包含哪些类型。

```ts
type StringNumberPair = [string, number];
```

上述的代码，我们一眼就可以知道 `StringNumberPair` 内只能有两个元素，并且清晰的知道第一个元素类型是 `srting`，第二个是 `number`

可以把 `tuple` 理解成一个**定死定长**的一个数组类型。

使用一些上述的 `StringNumberPair` 类型

```ts
const str: StringNumberPair = ["abc", 123];
```



### `tuple` 和普通数组的区别

 `tuple` 是一个定死的数组，它和普通数组获取 `length` 属性时的结果是有区别的。

```ts
type StringNumberPair = [string, number];
type stringArr = string[];

type t1 = StringNumberPair["length"] // type t1 = 2
type t2 = stringArr["length"] // type t2 = number
```

可以看到 `tuple` 的 `length` 属性是一个它长度的数值，而普通数组的 `length` 属性是 `number`

















