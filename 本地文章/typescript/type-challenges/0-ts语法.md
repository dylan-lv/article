## keyof

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



## extends

K extends：约束 K 是属于 keyof T 的

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]
}
```



## in

[P in K]：遍历K，P就是每一次遍历的值

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]
}
```



## 取值

T[P]：取值

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]
}
```



## 遍历数组：[number]

`ts` 中遍历数组的方式 `[number]`

```ts
type TupleToObject<T extends readonly any[]> = {
  [P in T[number]]: P 
}
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const
type r1 = typeof tuple; // type r1 = readonly ["tesla", "model 3", "model X", "model Y"]
type r = TupleToObject<typeof tuple> // type r = { tesla: "tesla"; "model 3": "model 3"; ...}
```



## 获取数组中不存在的下标

此时返回 `undefined`

```ts
type First<T extends any[]> = T[0]
type t0 = First<[]> // type t0 = undefined
```



















