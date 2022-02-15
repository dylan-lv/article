# TS从入门到放弃【六】：泛型

首先来看一个简单的场景

```ts
const getArray = (value: any, times: number = 5): any[] => {
  return new Array(times).fill(value)
}
getArray(8); // [8, 8, 8, 8, 8]
getArray(8, 3).map(item => item.length); // 数字没有length属性：[undefined, undefined, undefined]
getArray('abc', 4).map(item => item.length); // [3, 3, 3, 3]
```

上面这种形式虽然十分灵活（参数可以任意传），但是也**丢失了类型检测的功能**



### 使用泛型约束函数类型

```ts
const getArray = <T>(value: T, times: number = 5): T[] => {
  return new Array(times).fill(value)
}
```

与之前不同的地方在于多了 `T` 这个泛型参数，可以理解为这个函数：传入了 `T`（某个类型）作为 value 的类型，返回由 `T`组成的数组

> 这个 T 相当于“变量”，可以使用任意字母，不过习惯上使用 T、U、K...等大写字母

调用：

```ts
getArray<number>('abc'); // Error：类型“string”的参数不能赋给类型“number”的参数。
getArray<number>(8, 3).map(item => item.length); // Error：类型“number”上不存在属性“length”。
getArray<string>('abc').map(item => item.length); // [3, 3, 3, 3, 3]
```



**接下来使用两个泛型变量**

```ts
// 参数1是T类型，参数2是U类型，返回类型是元组类型 T,U组成的数组
const getArray = <T, U>(param1: T, param2: U, times: number): [T, U][] => {
  return new Array(times).fill([param1, param2]);
};
// 也可以明确泛型调用，不明确的话，TS会自动推导泛型类型：getArray<number, string>(1, 'a', 3);
getArray(1, 'a', 3); // [[1, 'a'], [1, 'a'], [1, 'a']]
```



### 使用泛型定义函数类型

```ts
// 定义函数
let getArray: <T>(arg: T, times: number) => T[];
// 函数体
getArray = (arg: any, times: number) => {
  return new Array(times).fill(arg)
}
getArray(123, 3).map(item => item.length); // Error：类型“number”上不存在属性“length”。
getArray(123, 3).map(item => item + 1); // [124, 124, 124]
```



- 使用**类型别名**来定义函数类型

```ts
type GetArray = <T>(arg: T, times: number) => T[]
let getArray: GetArray = (arg: any, times: number) => {
  return new Array(times).fill(arg);
};
```

- 使用**接口**来定义函数类型

```ts
interface GetArray {
  <T>(arg: T, times: number): T[];
}
let getArray: GetArray = (arg: any, times: number) => {
  return new Array(times).fill(arg);
};
```



### 将泛型变量定义到最外层

```ts
interface GetArray<T> {
  (arg: T, times: number): T[];
  array: T[];
}
```



### 泛型约束

​		**泛型约束**简单来说就是对**泛型变量**的条件限制。

​		**泛型变量**可以在定义一个类型结构的时候，让多个地方保证多个值是同一种类型（或者是由这个类型构成的其他元素）。但有时我们的需求是对**可以赋给泛型变量的类型**有一定的限制，所以就会用到**泛型约束**。

```ts
const getArray = <T>(arg: T, times): T[] => {
  return new Array(times).fill(arg);
}
// 现在想让外面只能传带length属性的arg：比如数组，字符串，自己定义的对象（里面包含length属性）
interface ValueWithLength {
  length: number;
}
const getArray1 = <T extends ValueWithLength>(arg: T, times): T[] => {
  return new Array(times).fill(arg);
}
getArray1(5, 3); // Error：类型“number”的参数不能赋给类型“ValueWithLength”的参数
getArray6('a', 2); // ['a', 'a']
getArray6([1, 2, 3], 2); // [[1, 2, 3], [1, 2, 3]]
getArray6({a: '1', length: 1}, 2); // [{a: '1', length: 1}, {a: '1', length: 1}]
```



### 在泛型约束中使用类型参数

使用情况：当我们定义一个对象，然后对**访问对象上存在的属性**作要求时

代码示例：

```ts
const getProps = (object, propName) => {
  return object[propName];
}
const objs = {
  a: 'a1',
  b: 'b1'
};
getProps(objs, 'a');
getProps(objs, 'c'); // 这里也没有报错，但实际上这里返回的是 undefined
```

如果想要对上面代码提供类型提示，让使用这个函数的人在编译阶段就意识到这个错误

```ts
// K是T中属性名的一种（属性名数组）：此时就使用 K extends keyof T
const getProps = <T, K extends keyof T>(object: T, propName: K) => {
  return object[propName];
}
const objs = {
  a: 'a1',
  b: 'b1'
};
getProps(objs, 'a');
getProps(objs, 'c'); // Error：类型“"c"”的参数不能赋给类型“"a" | "b"”的参数。
```























