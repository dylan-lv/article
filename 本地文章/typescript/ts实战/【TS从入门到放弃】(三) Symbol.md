# TS从入门到放弃【三】：Symbol

Symbol 代表一个独一无二的数据



### 1、Symbol 值是独一无二的

```js
const s1 = Symbol(); // Symbol()
const s2 = Symbol(); // Symbol()
s1 === s2; // false
```



### 2、Symbol可以传参

仅作为标识使用：好比两个同名的人，名字虽然一样，但确实是两个不同的人。

```js
const s1 = Symbol("dylan");
const s2 = Symbol("dylan");
console.log(s1); // Symbol(dylan)
console.log(s2); // Symbol(dylan)
s1 === s2; // false
s1.toString(); // 'Symbol(dylan)'
Boolean(s1); // true
!s1; // false
```



### 3、Symbol不能和其他类型做运算

```js
const s1 = Symbol('dy');
s1 + 1 // Error：运算符“+”不能应用于类型“unique symbol”和“number”。
```



### 4、作为属性名

```js
let prop = 'name';
const info = {
  [prop]: 'dylan', // 等同于 name: 'dylan'
  [`my${prop}is`]: 'dylan'
};
console.log(info); // {name: 'dylan', mynameis: 'dylan'}

const s2 = Symbol('name');
const info = {
  [s2]: 'dylanSymbol',
};
// 可以看到，这里的属性名是 Symbol 值，好处是可以保证这个属性不会被别的变量或属性名覆盖，此时只能通过 s2 来修改它
console.log(info); // { Symbol(name): 'dylanSymbol' }
// 修改symbol属性名的内容
info2[s2] = 'haha'; 
console.log(info2); // { Symbol(name): 'haha' }
```



### 5、属性名的遍历

```js
const s1 = Symbol('name');
const info1 = {
  [s1]: 'dylanSymbol',
  age: 18,
  sex: 'man'
};
```



下面几种方式都无法遍历到symbol属性名

```js
for(const key in info1) {
  console.log(key); // age、sex
}

console.log(Object.keys(info1)); // ['age', 'sex']

console.log(Object.getOwnPropertyNames(info1)); // ['age', 'sex']

console.log(JSON.stringify(info1)); // {"age":18,"sex":"man"}
```



可以使用下面的方法获取对象中的Symbol属性名

```js
console.log(Object.getOwnPropertySymbols(info1)); // [Symbol(name)]

console.log(Reflect.ownKeys(info1)); // ['age', 'sex', Symbol(name)]
```



### 6、Symbol的两个静态方法

**Symbol.for() 、 Symbol.keyFor()**

- Symbol.for它返回的也是一个Symbol值
- 但是当再次使用Symbol.for方法创建时
- 它首先会拿传入的字符串去全局范围找一下有没有使用这个字符串创建的Symbol
  - 全局范围：当前页面、iframe、service worker
- 如果有的话就直接返回原来的这个值，如果没有才会新创建一个

```js
const s3 = Symbol.for("dylan");
const s4 = Symbol.for("dylan");
console.log(s3 === s4); // true
```



如果传入的Symbol是使用Symbol.for注册的，Symbol.keyFor会返回当前Symbol初始化时的标识

```js
console.log(Symbol.keyFor(s2)); // undfined
console.log(Symbol.keyFor(s3)); // dylan
```



## 内置的Symbol值

指向 js 内部的一些属性和方法

### 1、Symbol.hasInstance

作用：外部对象调用 instanceof 后，就会调用 `Symbol.hasInstance` 所指向的方法

```ts
const obj = {
  [Symbol.hasInstance](otherObj) {
    console.log(otherObj);
    // return true; // 这里返回 true，外部的 instanceof 就会始终返回 true
  }
}
console.log({ a: 'a1' } instanceof (obj as any));
```

最终打印：`{a: 'a1'}`、`false`



### 2、Symbol.isConcatSpreadable

作用：它是一个可读写的 bool 值，当一个数组的这个值设为false时，这个数组的concat就不会被扁平化（拆开）

```js
let arr = [1, 2];
console.log([].concat(arr, [3, 4])); // [1, 2, 3, 4]
arr[Symbol.isConcatSpreadable] = false;
console.log([].concat(arr, [3, 4])); // [Array(2), 3, 4]
```



### 3、Symbol.species

作用：指定一个创建衍生对象的构造函数

先来看一个示例

```js
class C extends Array {
  public getName() {
    return 'dylan';
  }
}
const c = new C(1,2,3);
console.log(c); // [1, 2, 3]
const a = c.map(item => item + 1);
console.log(a); // [2, 3, 4]
c instanceof C; // true
c instanceof Array; // true
a instanceof C; // true
a instanceof Array; // true
```



加入 Symbol.species 后

```js
class C extends Array {
  constructor(...args) {
    super(...args);
  }
  static get [Symbol.species] () { // 指定一个创建衍生对象的构造函数
    return Array;
  }
  public getName() {
    return 'dylan';
  }
}
const c = new C(1,2,3);
const a = c.map(item => item + 1);
console.log(a instanceof C); // false
console.log(a instanceof Array); // true
```



### 4、Symbol.match

作用：当在一个字符串上调用 match 方法的时候，会调用这个方法

```js
let obj = {
  [Symbol.match] (str) {
    console.log(str.length); // 5
    return 'haha';
  }
};
'abcde'.match(obj); // haha
```



### 5、Symbol.replace

作用：当在一个字符串上调用 replace 方法的时候，会调用这个方法



### 6、Symbol.search

作用：当在一个字符串上调用 search 方法的时候，会调用这个方法



### 7、Symbol.split

作用：当在一个字符串上调用 split 方法的时候，会调用这个方法



### 8、Symbol.iterator

作用：数组的这个属性，指向该数组的默认遍历器方法

```js
const arr = [1, 2, 3];
const iterator = arr[Symbol.iterator]();
console.log(iterator); // Array Iterator {}
console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```



### 9、Symbol.toPrimitive

作用：当前对象转成原始类型时，会调用这个方法

```ts
let obj4: unknown = {
  [Symbol.toPrimitive](type) {
    console.log('toPrimitive：', type);
  }
};
const res1 = (obj4 as number)++; // toPrimitive：number
const res2 = `abc${obj4}`; // toPrimitive：string
```



### 10、Symbol.toStringTag

作用：当在对象上调用 toString 方法时，就会调用这个方法

```ts
let obj = {
  // [Symbol.toStringTag]: 'dylan'
  get [Symbol.toStringTag]() {
    return 'dylan'; // 和上面一样
  }
};
// 不设置的话，原本是[object Object]
console.log(obj.toString()); // [object dylan]
```



### 11、Symbol.unscopables

作用：指用于指定对象值，其对象自身和继承的从关联对象的 with 环境绑定中排除的属性名称。

```js
const obj = {
  a: "a1",
  b: "b1"
};
// @ts-ignore
with(obj) {
  console.log(a);
  console.log(b);
}
console.log(Array.prototype[Symbol.unscopables]); // 可以看到数组的哪些属性被过滤掉了
```







