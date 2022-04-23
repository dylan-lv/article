## 题目三十三：replacekeys

```ts
// template.ts
type ReplaceKeys<U, T, Y> = any
```

```ts
// test-cases.ts
import type { Equal, Expect } from '@type-challenges/utils'

type NodeA = {
  type: 'A'
  name: string
  flag: number
}

type NodeB = {
  type: 'B'
  id: number
  flag: number
}

type NodeC = {
  type: 'C'
  name: string
  flag: number
}

type ReplacedNodeA = {
  type: 'A'
  name: number
  flag: string
}

type ReplacedNodeB = {
  type: 'B'
  id: number
  flag: string
}

type ReplacedNodeC = {
  type: 'C'
  name: number
  flag: string
}

type NoNameNodeA = {
  type: 'A'
  flag: number
  name: never
}

type NoNameNodeC = {
  type: 'C'
  flag: number
  name: never
}

type Nodes = NodeA | NodeB | NodeC
type ReplacedNodes = ReplacedNodeA | ReplacedNodeB | ReplacedNodeC
type NodesNoName = NoNameNodeA | NoNameNodeC | NodeB

type cases = [
  Expect<Equal<ReplaceKeys<Nodes, 'name' | 'flag', { name: number; flag: string }>, ReplacedNodes>>,
  Expect<Equal<ReplaceKeys<Nodes, 'name', { aa: number }>, NodesNoName>>,
]
```

实现一个类型 `ReplaceKeys`，替换联合类型中的键，如果某个类型没有这个键，只需跳过替换，一个类型需要三个参数。



### 测试用例

- `Equal<ReplaceKeys<Nodes, 'name' | 'flag', { name: number; flag: string }>, ReplacedNodes>`

`Nodes` 中将键为 `name` 的字段，值修改为 `number`；将键为 `flag` 的字段，值修改为 `string`

- `Equal<ReplaceKeys<Nodes, 'name', { aa: number }>, NodesNoName>`

准备在 `Nodes` 中修改键为 `name` 的字段，但是参数3中**没有对应的键为 `name` 的字段**，所以就将 `Nodes` 中键为 `name` 的字段，值修改为 `never`



### 代码实现

- 原代码
  - `U`：联合类型
  - `T`：需要修改的字段名
  - `Y`：修改的字段值，需要和 `T` 进行匹配

```ts
type ReplaceKeys<U, T, Y> = any
```

- 先排除 `never` 的情况

`never` 不继承自任何类型

```ts
type ReplaceKeys<U, T, Y> = U extends [never]
	? never
	: ...
```

- 遍历 `U`，并判断每个遍历的键是否存在于 `T` 中。如果不存在，则说明无需修改，直接原值返回即可

```ts
type ReplaceKeys<U, T, Y> = U extends [never]
	? never
	: {[P in keyof U]: P extends T
    ? ...
    : U[P]}
```

- 如果当前循环的键存在于 `T` 中，那么继续判断这个键是否存在于 `Y` 的键中，如果存在则使用 `Y` 的值替换，不存在则返回 `never`

```ts
type ReplaceKeys<U, T, Y> = U extends [never]
	? never
	: {[P in keyof U]: P extends T
    ? P extends keyof Y
     	? Y[P]
     
    : U[P]}
```























