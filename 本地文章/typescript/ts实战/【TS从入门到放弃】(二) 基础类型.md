# TS从入门到放弃【二】：基础类型

### 1、布尔类型

```ts
let bool: boolean;
bool = false;
bool = 123; // Error：不能将类型“number”分配给类型“boolean”。
```



### 2、数值类型

```ts
let num: number = 123;
num = 0b1011; // 二进制
num = 0o171; // 八进制
num = 0x7b; // 十六进制
```



### 3、字符串类型

```ts
let str: string;
str = 'bac';
str = `数值是${num}`;
```



### 4、数组类型

```ts
let arr1: number[];
arr1 = [1, 2, 3];

let arr2: Array<number>;
arr2 = [1, 2, 3];

let arr3: (string | number)[];
arr3 = [1, '2', 3];
```



### 5、元组类型

```ts
let tuple: [string, number, boolean];
tuple = ['a', 1, false]; // 必须按照上面的顺序和类型
tuple = ['a', false, false]; // Error：不能将类型“boolean”分配给类型“number”。
tuple = ['a', 1, false, 12]; // Error：不能将类型“[string, number, false, number]”分配给类型“[string, number, boolean]”。
```



### 6、枚举类型

默认从0开始

```ts
enum Roles {
  SUPER_ADMIN, // 0
  ADMIN, // 1
  USER // 2
}
console.log(Roles.SUPER_ADMIN); // 0
console.log(Roles[Roles.SUPER_ADMIN]); // SUPER_ADMIN
```

可以进行主动赋值

```ts
enum Roles {
  SUPER_ADMIN, // 0
  ADMIN = 3, // 3
  USER // 4
}
console.log(Roles.USER); // 4
```

在 https://www.typescriptlang.org/play 可以看到枚举编译成 js 之后的代码如下

```js
var Roles;
(function (Roles) {
    Roles[Roles["SUPER_ADMIN"] = 0] = "SUPER_ADMIN";
    Roles[Roles["ADMIN"] = 3] = "ADMIN";
    Roles[Roles["USER"] = 4] = "USER";
})(Roles || (Roles = {}));
```



### 7、any类型

```ts
let value: any;
value = 'abc';
value = 2;
value = [1,2,3];
let arr: any[] = [1, 'a'];
```



### 8、void类型

```ts
const consoleText = (text: string): void => { // 不返回内容
  console.log(text);
};
let v: void;
v = undefined;
v = null; // tsconfig的strict需要关掉
```



### 9、null 和 undefined

```ts
let u: undefined;
u = undefined;
let n: null;
n = null;
// null 和 undefined 是其他类型的子类型
let num: number
num = undefined;
num = null;
```



### 10、never类型

表示永远不存在的类型，抛错or死循环，返回值就是 never 类型。

never类型是任意类型的子类型，没有任何类型是never的子类型 =》 never可以赋值给任意类型，任意类型都不能赋值给never类型 =》 它可以给别人，但别人不能给它

```ts
const errorFunc = (message: string): never => {
  throw new Error(message);
};
const infiniteFunc = (): never => {
  while(true) {}
};
// let neverVariable: never
let neverVariable = (() => {
  while(true) {}
})();
```



### 11、object（对象）类型

对象存的是内存地址的引用

```ts
function getObject(obj: object): void {
  console.log(obj);
}
getObject({ name: 'dylan' });
getObject(123); // Error：类型“number”的参数不能赋给类型“object”的参数。
```



### 12、类型断言

```ts
const getLength = (target: string | number): number => {
  if((target as string).length || (target as string).length === 0) {
    return (target as string).length;
  } else {
    return target.toString().length;
  }
};
```

