# TS从入门到放弃【四】：接口

先看个例子

```js
const getFullName = ({ firstName, lastName }) => {
  return `${firstName} ${lastName}`;
};
getFullName({ firstName: 'haha', lastName: 'dylan' })
// 但是下面的操作明显就不是我们想要的了
getFullName({ firstName: 'haha', lastName: 18 })
```

我们可以使用接口来进行限制

```ts
interface NameInfo {
  firstName: string;
  lastName: string;
}
const getFullName = ({ firstName, lastName }: NameInfo): string => {
  return `${firstName} ${lastName}`;
};
getFullName({
  firstName: 'haha',
  lastName: 'Lv'
});
```



### 1、可选属性

```ts
interface Vegetable {
  color?: string;
  type: string;
}
const getVegetables = ({ color, type }: Vegetable) => {
  return `A ${color ? (color + ' ') : ''}${type}`;
};
getVegetables({ type: 'tomato' }); // A tomato
getVegetables({ color: 'red', type: 'tomato' }); // A red tomato
```



### 2、多传入参数

```ts
interface Vegetable {
  color?: string;
  type: string;
}
const getVegetables = ({ color, type }: Vegetable) => {
  return `A ${color ? (color + ' ') : ''}${type}`;
};
getVegetables({ type: 'tomato', size: 2 }); // Error：类型“{ type: string; size: number; }”的参数不能赋给类型“Vegetable”的参数。
```

- 方法1：类型断言

```ts
getVegetables({ type: 'tomato', size: 2 } as Vegetable);
```

- 方法2：索引签名

```ts
interface Vegetable {
  color?: string;
  type: string;
  [prop: string]: any; // 索引签名
}
getVegetables({ type: 'tomato', size: 2 });
```

- 方法3：类型兼容性

原理：const b = a  ；如果想把 a 赋值给 b，那么 b 中有的属性 a 中必须得有，多了无所谓

```ts
// 变成参数抽出来
const vegetableInfo = {
  type: 'tomato',
  size: 2
};
getVegetables(vegetableInfo);
```



### 3、只读属性

```ts
interface Vegetable {
  color?: string;
  readonly type: string;
}
let vegetableObj: Vegetable = {
  type: 'tomato'
};
vegetableObj.type = 'carrot'; // Error：无法分配到 "type" ，因为它是只读属性。
```



我们还可以限定一个数组的元素只能读取，不能修改

```ts
interface ArrInter {
  0: number;
  readonly 1: string;
}
let arr: ArrInter = [1, 'a'];
arr[1] = 'b'; // Error：无法分配到 "1" ，因为它是只读属性。
```



### 4、定义函数结构

```ts
// 等同于类型别名：type AddFunc = (num1: number, num2: number) => number;
interface AddFunc {
  (num1: number, num2: number): number
}
const add: AddFunc = (n1, n2) => n1 + n2;
```



### 5、索引类型

索引为 number

```ts
interface RoleDic {
  [id: number]: string;
}
const role1: RoleDic = {
  'a': 'super_admin' // Error：不能将类型“{ a: string; }”分配给类型“RoleDic”。对象文字可以只指定已知属性，并且“'a'”不在类型“RoleDic”中。
};
const role2: RoleDic = {
  0: 'super_admin' // OK
};
```

索引为 string

```ts
interface RoleDic1 {
  [id: string]: string;
}
const role2: RoleDic1 = {
  a: 'super_admin',
  1: 'admin' // 这里会把 1 自动转换成字符串
};
```



### 6、接口的继承

```ts
interface Vegetables {
  color: string;
}
interface Tomato extends Vegetables {
  radius: number;
}
const tomato: Tomato = {
  radius: 1,
  color: 'red'
};
```



### 7、混合类型接口

给函数添加属性

**闭包**

```ts
const countUp = (() => {
  let count = 0;
  return () => {
    return ++count;
  };
})();
countUp(); // 1
countUp(); // 2
```

**给函数添加属性**

```ts
const countUp = () => {
  countUp.count++;
};
countUp.count = 0;
countUp();
countUp.count; // 1
countUp();
countUp.count; // 2
```

**使用ts，给函数添加属性**

```ts
interface Counter {
  (): void;
  count: number;
}
const getCounter = (): Counter => {
  const c = () => c.count++;
  c.count = 0;
  return c;
};
const counter: Counter = getCounter();
counter();
counter();
console.log(counter.count); // 2
```

























