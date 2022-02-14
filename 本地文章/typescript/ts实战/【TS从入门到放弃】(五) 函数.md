# TS从入门到放弃【五】：函数

首先，定义一个 add 函数

```ts
function add(a: number, b: number): number {
  return a + b;
}
// 或者
const add = (a: number, b: number): number => a + b;
```

有时一个库会有回调函数，我们会对回调函数有一定要求

接下来我们来看一个完整的函数类型

```ts
// 定义
let add1: (x: number, y: number) => number
// 实现
add1 = (a: number, b: number): number => a + b
add1 = (a: string, b: number) => a + b; // Error：不能将类型“number”分配给类型“string”。
```



### 使用函数体之外的变量

函数中如果使用了函数体之外的变量，这个变量的类型是不体现在函数类型定义中的

```ts
let arg3 = 3;
add1 = (a: number, b: number): number => a + b + arg3;
```



### 使用接口（或类型别名）定义函数类型

```ts
interface Add {
  (x: number, y: number): number;
}
// 上面代码会被ts-lint转换为: 
type Add = (x: number, y: number) => number

type isString = string;
```

使用：

```ts
let addFunc: Add;
addFunc = (a: number, b: number): number => a + b;
```



### 函数的参数

**可选参数**

```ts
let addFunc;
addFunc = (arg1: number, arg2: number) => arg1 + arg2;
addFunc = (a, b, c) => a + b + (c ? c : 0);
// 可选参数，必须放在必选参数后面
type AddFunction = (a: number, b: number, c?: number) => number;
let addFunction: AddFunction;
addFunction = (x: number, y: number) => x + y;
addFunction = (x: number, y: number, z: number) => x + y + z;
```

**默认参数**

```ts
const addFunc = (x: number, y = 3) => x + y;
addFunc(2, 'a'); // Error：TS已经根据默认值推断出来y是number类型了
```

**剩余参数**

```ts
// 获取参数：es5方法
function handleData() {
  if(arguments.length === 1) return arguments[0] * 2
  if(arguments.length === 2) return arguments[0] * arguments[1]
  return Array.prototype.slice.apply(arguments).join('_')
}
// 获取参数：es6方法
const handleData = (...args) => {
  console.log(args);
}
// 获取剩余参数
const handleData = (a: number, ...args: number[]) => {
  // ...
};
```



### 函数重载

参数个数不同、参数类型不同

```ts
function handleData(x: string): string[];
function handleData(x: number): number[];
function handleData(x: any): any {
  if(typeof x === 'string') {
    return x.split('')
  } else {
    return x.toString().split('').map(item => Number(item))
  }
}
handleData('abc'); // ['a', 'b', 'c']
handleData(123); // [1, 2, 3]
```





















