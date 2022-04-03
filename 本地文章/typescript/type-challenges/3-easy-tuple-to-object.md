## 题目三：easy-tuple-to-object

```ts
// template.ts
type TupleToObject<T extends readonly any[]> = any
```

```ts
// test-cases.ts
import { Equal, Expect } from '@type-challenges/utils'

const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const

type cases = [
  Expect<Equal<TupleToObject<typeof tuple>, { tesla: 'tesla'; 'model 3': 'model 3'; 'model X': 'model X'; 'model Y': 'model Y'}>>,
]

// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>
```



### 1. typeof

先来看测试代码：

```ts
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const
```

这里声明的是 `js` 变量or常量（`let,const 这类都属于 js 内容`）

而下面的 `type cases` ，却属于 `ts` 中的**类型**，变量和类型属于两个不同的轨道线；如果想使用变量对类型进行一定的操作的话，就需要使用 `typeof` 这个关键字了，也就是我们测试代码中 `TupleToObject` 传入的参数 `typeof tuple`

我们可以看一下，使用 `typeof` 关键字转换 `tuple` 后，它是什么内容：

```ts
type r = typeof tuple
```

把鼠标放在 `r` 上，可以看到 `r` 的值为：`type r = readonly ["tesla", "model 3", "model X", "model Y"]`



### 2. as const

如果我们不写 `as const` 的话，来看看效果

```ts
const tuple = ['tesla', 'model 3', 'model X', 'model Y']
type r = typeof tuple
```

此时，把鼠标放在 `r` 上，可以看到 `type r = string[]`。发现它是一个包含 `string` 的数组类型。

这里就会涉及到一个比较基础的知识点：**字面量类型**

**通过 let 创建**

```ts
let str = "abc"
type t = typeof str // type t = string
```

如果我们是通过 `let` 创建的 `str`，则 `type t = string`

**通过 const 创建**

```ts
const str = "abc"
type t = typeof str // type t = "abc"
```

如果我们是通过 `const` 创建的 `str`，则 `type t = "abc"`。意味着类型 `t` 是不可以被修改的。

**原因**

`const` 创建的是一个常量，我们只要创建好了这个常量，它就不可被修改了。所以把它应用到**类型**之后，它就变成了**字面量类型**。



回到我们的 `as const`，如果我们使用了 `as const`，相当于将数组 `tuple` 中的元素都变成了**常量**，之后不能再对齐修改了。

例如：`tuple[0] = "abc"`，就会报错。



### 3. 题解

1. 返回一个对象
2. 遍历数组（类型数组）

我们先尝试一下使用 `keyof` 来遍历数组

```ts
type TupleToObject<T extends readonly any[]> = {
  [P in keyof T]: P 
}
```

来看一下测试：

```ts
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const
type r = TupleToObject<typeof tuple> // type r = readonly ["0", "1", "2", "3"]
```

可以看到，`r` 虽然不是我们想要的内容，但是我们误打误撞拿到了数组的下标索引。

那么，应该怎样去遍历数组呢？

这里涉及到一个新的语法 `[number]`，让我们来试一下

```ts
type TupleToObject<T extends readonly any[]> = {
  [P in T[number]]: P 
}
```

再来看一下测试代码

```ts
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const
type r = TupleToObject<typeof tuple> // type r = { tesla: "tesla"; "model 3": "model 3"; ...}
```

搞定~



### 4. 遍历数组：[number]

`ts` 中遍历数组的方式 `[number]`



### 5.  期望报错

```ts
// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>
```

在测试代码中，这里是期望会报错的

在 `TupleToObject` 中，它是不可以接收**数组**和**对象**类型的。

对象的 `key` 只能接收三种类型 `number/string/symbol`，所以我们只需要在 `TupleToObject` 的接收入参的时候进行限制就可以了

```ts
type TupleToObject<T extends readonly (string|number|symbol)[]> = {
  [P in T[number]]: P 
}
```







