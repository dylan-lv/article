# ts类型挑战【一】



本专栏基于 [type-challenges](https://github.com/type-challenges/type-challenges) 来进行 `typescript` 类型的训练。

这里给我们准备了很多题目，并且帮我们区分了难度（简单、中等、困难）。将这些题目刷完，我们 `ts` 的类型就可以玩的差不多了。



## 搭建环境

- 新建文件夹

```bash
$ mkdir tsl-ts
$ cd tsl-ts
```

- 初始npm包，并安装依赖

```bash
pnpm init -y
pnpm add -D @type-challenges/utils tsd
```

- 配置 `package.json`

```json
"tsd": {
  "directory": "test-dts",
  "compilerOptions": {
    "strict": false,
    "lib": [
      "esnext",
      "dom"
    ]
  }
}
```



## 热身

创建 `hello-world` 文件夹，并将 [type-challenges](https://github.com/type-challenges/type-challenges) 中的 `template.ts` 和 `test-cases.ts` 两个文件拷贝进来

```ts
// type-challenges\13-warm-hello-world\template.ts
type HelloWorld = any // expected to be a string
```

```ts
// type-challenges\13-warm-hello-world\test-cases.ts
import { Equal, Expect, NotAny } from '@type-challenges/utils'
type cases = [
  Expect<NotAny<HelloWorld>>,
  Expect<Equal<HelloWorld, string>>
]
```

可以看到 `test-cases.ts` 中冒红了，此时我们就可以开始解答这道热身题了。

通过测试文件，我们可以知道它希望 `HelloWorld` 是一个 `string` 类型，我们直接在 `template.ts` 文件中对其进行修改：

```ts
// 题目
// type HelloWorld = any // expected to be a string

// 题解
type HelloWorld = string
```

现在，我们就完成这道热身题了。



## 题目一：easy-pick

同样，我们先把  `template.ts` 和 `test-cases.ts` 两个文件拷贝进来

```ts
type MyPick<T, K> = any
```

```ts
import { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Expected1, MyPick<Todo, 'title'>>>,
  Expect<Equal<Expected2, MyPick<Todo, 'title' | 'completed'>>>,
  // @ts-expect-error
  MyPick<Todo, 'title' | 'completed' | 'invalid'>,
]

interface Todo {
  title: string
  description: string
  completed: boolean
}

interface Expected1 {
  title: string
}

interface Expected2 {
  title: string
  completed: boolean
}
```

先来看测试代码：

我们首先定义了一个接口 `Todo`，其中有三个属性：`title`、`description` 和 `completed`。

- 通过 `MyPick<Todo, 'title'>` 之后，得到一个只包含 `title` 属性的接口 `Expected1`
- 通过 `MyPick<Todo, 'title' | 'completed'>` 之后，得到一个包含 `title`、`completed` 这两个属性的接口 `Expected2`
- `MyPick<Todo, 'title' | 'completed' | 'invalid'>`
  - 这段代码会提示异常信息，因为属性 `invalid` 并不再 `Todo` 中 

现在，基本可以确定 `MyPick` 方法其实就是一个过滤器的作用，用参数2中的内容依次去参数1中寻找相应的属性，最后再返回。



**题解：**

1. 返回一个对象
2. 遍历参数2：[P in K]
3. 取值：T[P]
4. 看看 key 是否在对象中：K extends keyof T

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]
}
```

- keyof T：获取T中所有的key
- K extends：约束 K 是属于 keyof T 的
- [P in K]：遍历K，P就是每一次遍历的值
- T[P]：取值

在上述代码中，

keyof 相当于 lookup（查阅）。

extends 如果在 <> 中，就相当于约束。

keyof T 是一个 union 类型，K 也是一个 union 类型，extends 会依次进行对比。相当于两个数组进行对比。



### keyof

索引类型查询 `keyof T`，会为T生成允许的属性名称类型。`keyof T` 类型被视为字符串的子类型。

```ts
interface Person {
  name: string;
  age: number;
  location: string;
}
type K1 = keyof Person; // "name" | "age" | "location"
type K2 = keyof Person[]; // "length" | "push" | "pop" | "concat" | ...
type K3 = keyof { name: 1 }; // name
type K4 = keyof { [x: string]: Person }; // string
```

在上述代码中，

- `K1` 的值为 `"name" | "age" | "location"`，相当于接口 `Person` 中全部参数名的联合类型
- `K2` 的值为 `"length" | "push" | "pop" | "concat" | ...` ，相当于拿出了**数组**中全部的参数名
- `K3` 的值为 `"name"`。在对象中，`keyof` 会拿出对象 `key` 的参数名
- `K4` 的值为 `"string"`。对象的 `key` 为 `[x: string]` 这种模式时，`keyof` 会拿出 `x` 的类型 `string`





































