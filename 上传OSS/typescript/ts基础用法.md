# TypeScript Study

## 一、TS基本类型

### 1.1、boolean类型

```js
let flag : boolean = false
flag = true
```



### 1.2、number类型

```js
let flag : number = 1
flag = 2
```



### 1.3、string类型

```js
let flag : string = 'a'
flag = 'b'
```



### 1.4、array类型

```js
const list1 : number[] = [1, 2, 3]
// ES5：var list1 = [1, 2, 3]

const list2 : Array<number> = [1, 2, 3] // Array<number>泛型语法
// ES5：var list2 = [1, 2,3 ]
```

**ReadonlyArray类型**

告诉别人，这个数组不能改变！

```js
function doStuff(values : ReadonlyArray<string>) {
// function doStuff(values : readonly string[]) {
  // 我们可以读取values
  const copy = values.slice()
  // 但是我们不能修改vlaues（下面这行代码会报错）
  values.push("hello!") // Error：类型“readonly string[]”上不存在属性“push”。
}
```

`注意：没有ReadonlyArray这个构造函数，我们不能 new ReadonlyArray(x1, x2 ...)`

但是我们可以把一个常规的Array分配给ReadonlyArray

```js
const myArray : ReadonlyArray<string> = ["red", "green", "blue"]
```



### 1.5、枚举类型

TypeScript 支持基于数字和基于字符串的枚举。

#### (1) 数字枚举

```js
enum Direction {
  NORTH,
  SOUTH,
  EAST,
  WEST,
}

let dir : Direction = Direction.NORTH
dir = Direction.SOUTH
```

默认情况下，NORTH的初始值为0，其余的成员会从1开始自动增长。

也可以设置每个数字的初始值。

#### (2) 字符串枚举

```js
enum Direction {
  NORTH = "NORTH",
  SOUTH = "SOUTH",
  EAST = "EAST",
  WEST = "WEST",
}
```

#### (3) 常量枚举

使用`const`关键字修饰枚举。

```js
const enum Direction {
  NORTH,
  SOUTH,
  EAST,
  WEST,
}

let dir : Direction = Direction.NORTH
```

编译成ES5代码：

```js
"use strict";
var dir = 0 /* NORTH */;
```



### 1.6、any类型

`any`类型：任何类型都能归为`any`类型。

- 可以对`any`类型进行任何操作，不进行类型错误检查。
- 但滥用`any`将丢失`TS`的保护特性



### 1.7、unknown类型

- 我们可以给`unknown`类型赋任何值。
- 但并不能把`unknown`的值赋给**非unknown、any的类型**。
- 不能对`unknown`类型进行任何操作（与`any`正好相反）。

```js
let value : unknown

// 赋值
const value1 : unknown = value // OK
const value2 : any = value // OK
const value3 : boolean = value // Error

// 操作
value.foo.bar; // Error
value.trim(); // Error
```



### 1.8、tuple元组类型

#### (1) 元组类型

元组类型：长度和元素类型都确定的数组。

```js
type StringNumberPair = [string, number]
```

该类型数组第一个元素类型为string，第二个元素为number。

#### (2) 元组可选属性

元组可选属性，只能用于最后一个。

```js
type Either2dOr3d = [number, number, number?]
```

#### (3) 元组readonly

readonly 元组类型，即元素不允许发生改变

```js
function doSomething(pair : readonly [string, number]) {
  pair[0] = "hello"; // Error
}
```



### 1.9、never类型

`never`类型：永不存在的值。常用于`分支收窄`中（if、switch）。

**使用never的好处**

使用never避免出现新增了联合类型没有对应的实现，保证类型安全。

**分支收窄**

我们需要对参数类型进行区分，进行不同操作。此时在一个if中判断一个类型后，在外边TS会自动缩小范围为另一个类型

```js
function padLeft(padding : number | string, input : string) {
  if(typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
  }
  return padding + input
}
```



### 1.10、void类型

`void`：没有任何类型。例如没有返回值的函数。

```js
function pop() : void{
  
}
```



### 1.11、null 和 undefined

TypeScript 里，`undefined` 和 `null` 两者有各自的类型分别为 `undefined` 和 `null`。



### 1.12、object、Object 和 {} 类型

#### (1) object

非基础类型，基础类型包括`null` 和 `undefined`、`string`、`number`、`boolean`、`symbol`、`bigint`。

#### (2) Object

Object类的实例的类型

#### (3) {} 类型

没有成员的对象，不允许访问该对象的任意属性。

```js
const obj = {}
obj.prop = "age" // Error
```



## 二、TS断言

### 2.1、类型断言

我们开发者比TS更了解某个实体的类型，此时使用**类型断言**给该实体指定类型，此时不会进行类型扩展

#### (1) 尖括号语法

```js
const someValue : any = "this is a string"
const strLength : number = (<string>someValue).length
```

#### (2) as 语法

```js
const someValue : any = "this is a string"
const strLength : number = (someValue as string).length
```

#### (3) 类型扩展

**类型扩展**：某个变量的类型被扩展为通用类型，例如`string`、`number`等

1、const声明，将不会进行类型扩展

```js
const constantString = "hello world"
constantString; // 类型："hello world"
```

2、let/var 声明，会进行类型扩展

```js
let changingString = "hello world"
changingString; // 类型：string
```

3、引用特定的字符串和数字作为类型，不会进行类型扩展

```js
function printText(s : string, alignment : "left" | "right" | "center") {
  // ...
}
printText("hello,world", "left")
printText("good, good", "top") // 类型“"top"”的参数不能赋给类型“"left" | "right" | "center"”的参数。
```

**应用**

```js
const req = { url : "http://example.com", method : "GET" }
handlerRequest(req.url, req.method); // Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'
```

原因：method被推理为string，此时handleRequest接受的第二个参数采用文字类型`GET`和`POST`，不匹配。

解决方案：const 断言。对象内属性不会进行类型扩展。

```js
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```



### 2.2、非空断言

非空断言运算符`！`：从 x 值域中排除 null 和 undefined

使用场景：

#### (1) 忽略 undefined 和 null 类型

```js
function myFunc(maybeString : string | undefined | null) {
  const onlyString : string = maybeString; // Error：不能将类型“string | null | undefined”分配给类型“string”。
  const ignoreUndefinedAndNull : string = maybeString!; // OK
}
```

#### (2) 调用函数时忽略 undefined 类型

```js
function myFunc(numGenerator : NumGenerator | undefined) {
  const num1 = numGenerator(); // Error：不能调用可能是“未定义”的对象。
  const num2 = numGenerator!(); // OK
}
```



### 2.3、确定赋值断言

在实例属性和变量声明后使用`!`，明确告诉 TS 某变量已被明确赋值。

```js
let x : number;
initialize();

// 如果第一行是（let x! : number），则OK，告诉ts某变量已被明确赋值
console.log(2 * x); // Error

function initialize() {
  x = 10;
}
```



## 三、类型守卫

用于检测属性、方法和原型。

### 3.1、in 关键字

遍历

```js
type Keys = "name" | "sex"
type OtherKeys = {
  [key in Keys] : string // 类似 for ... in
}

const stu : OtherKeys = {
  name: "wang",
  sex: "man"
}
```

类型窄化、判断某个对象里是否有某个属性

```js
type Fish = {
  swim: () => void
}
type Bird = {
  fly: () => void
}
function move(animal : Fish | Bird) {
  if("swim" in animal) return animal.swim()
  return animal.fly()
}
```



### 3.2、typeof 关键字

判断某个对象的类型，跟 JS 的`typeof`差不多。

**注意：不能用在类型参数上（泛型）**



### 3.3、instanceof 关键字

判断某个对象是否为某个类的实例。



### 3.4、自定义类型保护的类型谓词

告诉 `TS` ，如果逻辑语句汇总返回的是`true`，则判断当前的`x`变量的类型为`number`类型。

```js
function isNumber(x : any) : x is number {
  return typeof x === "number"
}
function isString(x : any) : x is string {
  return typeof x === "string"
}
```



## 四、联合类型、交集类型、类型别名

### 4.1、联合类型

xxx | xxx



### 4.2、交集类型

使用 `&` 运算符定义交集类型，对象属性叠加类型

```js
interface Colorful {
  color : string;
}
interface Circle {
  radius : number;
}

type ColorfulCircle = Colorful & Circle;

function draw(circle : ColorfulCircle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
```

#### (1) 同名基础类型属性的合并

此时成员`C`类型不一致，`C`的类型为`never`

```js
interface X {
  c : string;
  d : string;
}
interface Y {
  c : number;
  e : string;
}

type XY = X & Y
```

#### (2) 同名非基础类型属性的合并

可以正常合并

```js
interface D { d : boolean }
interface E { e : string }
interface F { f : number }

interface A { x : D }
interface B { x : E }
interface C { x : F }

type ABC = A & B & C // { x: { d: boolean, e: string, f: number } }
const abc : ABC = {
  x: {
    d: true,
    e: "some",
    f: 10
  }
}
```

#### (3) 类型别名

给类型起一个新的名字。

```js
type Point = {
  x : number;
  y : number;
}

function printRect(pt : Point) {
  ...
}
```



## 五、对象类型

### 5.1、对象属性修饰

- 可选：？
- 只读：readonly

**可选**

```js
interface PaintOptions {
  xPos : number;
  yPos : number;
  zPos?: number;
}
function paintShape(opts : PaintOptions) {
  // ...
}
paintShape({ xPos: 1, yPos: 2 })
paintShape({ xPos: 1, yPos: 2, zPos: 3 })
```

**readonly 只读**

被修饰属性无法被写入（类似const，不是一定的不变，只是意味着属性本身不能被重写）

```js
interface Home {
  readonly student : { name: string; age: number }
}
function visitForAge(home : Home) {
  home.student.age++ // 可以更新，并且不会报错
}
function evict(home : Home) {
  // Error：无法分配到 "student" ，因为它是只读属性。
  home.student = {
    name: 'dy',
    age: 40
  }
}
```



### 5.2、解构赋值重命名

**改名前：改名后**

接受属性shape，被重新定义名字为Shape，xPos被重新定义为number。

```js
function draw({ shape : Shape, xPos : number = 100 }) {
  console.log(shape); // Error：找不到名称“shape”。你是否指的是“Shape”?
  console.log(xPos); // Error：找不到名称“xPos”。
}
```



### 5.3、索引签名 Index Signatures

有时候我们不知道对象类型属性的所有名称，但是我们知道它的key类型对应的可能值类型。这时候可以使用索引签名来描述可能值的类型。

```js
//这是一个带有索引签名的接口，当索引为number类型时，值类型为string
interface StringArray {
  [index: number] : string;
}
```

**注意：索引签名属性类型必须为字符串/数字**



### 5.4、keyof 类型运算符

获取对象的key值的类型，为联合类型。



### 5.5、索引访问类型

 1.使用索引访问特定属性

```js
type Person = {
  age : number;
  name : string;
}
type Age = Person["age"] // ype Age = number
```

2.索引类型本身就是一种类型，所以我们可以使用 keyof 以及其他类型（例如联合类型）

```js
type Person = {
  age : number;
  name : string;
  alive : boolean;
}
type I1 = Person["age" | "name"]; // type I1 = string | number
type I2 = Person[keyof Person]; // type I2 = string | number | boolean
type AliveOrName = "alive" | "name"; // 只能等于alive或者name这两个字符串
type I3 = Person[AliveOrName]; // type I3 = string | boolean
```

3.若使用索引不存在的属性，Error

4.使用`number`索引，可以获取数组元素的类型

```js
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 42 },
]
type Person = typeof MyArray[number]; // type Person = { name: string; age: number; }
type Age = typeof MyArray[number]["age"]; // type Age = number
type Age2 = Person["age"]; // type Age2 = number
```



## 六、TS函数

### 6.1、参数类型和返回类型

```js
function createUserId(name : string, id : number) : string {
  return name + id;
}
```



### 6.2、函数类型

```js
let IdGenerator : (chars : string, nums : number) => string;

function createUserId(name : string, id : number) : string {
  return name + id;
}

IdGenerator = createUserId;
```



### 6.3、可选参数和默认参数

```js
// 可选参数
function createUserId(name : string, id : number, age?: number) : string {
  return name + id;
}
// 默认参数
function createUserId2(
  name : string = "dylan",
  id : number,
  age?: number
) : string {
  return name + id;
}
```

**回调函数中的可选参数**

为回调函数编写函数类型时，`切勿编写可选参数`，除非你打算在不传递该参数的情况下调用该函数。



### 6.4、函数重载

同一函数名，接受的参数个数/类型不一致，称为函数重载。

格式：n（n>=2）个函数类型定义，紧跟实现函数体。

```js
// 函数类型定义
function makeDate(timeStamp : number) : Date
function makeDate(m : number, d : number, y : number) : Date

// 实现函数体
function makeDate(mOrTimeStamp : number, d?: number, y?: number) : Date {
  if(d !== undefined && y !== undefined) {
    return new Date(y, mOrTimeStamp, d)
  } else {
    return new Date(mOrTimeStamp)
  }
}
```



## 七、TS 接口

`接口：`命名对象的形状（Shape）

```js
interface Point {
  x : number;
  y : number;
}
function printCoord(pt : Point) {
  console.log(pt.x, pt.y);
}
```

**类型别名和接口的异同点**

- 相同：可以定义对象的形状
- 不同：接口可进行声明合并，类型别名不行。



## 八、TS 类

### 8.1、类的属性和方法

成员属性与静态属性，成员方法与静态方法

```js
class Greeter {
  // 静态属性
  static cname : string = "Dylan";
  // 成员属性
  greeting : string;
  // 构造函数 - 执行初始化操作
  constructor(message : string) {
    this.greeting = message;
  }
  // 静态方法
  static getClassName() {
    return "class name is greeter"
  }
  // 成员方法
  greet() {
    return "Hello, " + this.greeting;
  }
}

const greeter = new Greeter("world")
```



### 8.2、成员访问修饰符

TypeScript可以使用三种访问修饰符，分别是 `public`、`private` 和 `protected`。

- `public` 修饰的属性或方法是公有的，可以在任何地方被访问到，默认`public`
- `private` 修饰的属性或方法是私有的，不能在声明它的类的外部访问
- `protected` 修饰的属性或方法是受保护的，仅对自身和子类可见

```js
class MySafe {
  private secretKey = 12345;
}
const s = new MySafe();
// console.log(s.secretKey); // Error：属性“secretKey”为私有属性，只能在类“MySafe”中访问。
console.log(s["secretKey"]); // OK
```

**跨层级protected访问**

`protected`通过基类引用访问成员是否合法？在TS中不合法！

**派生类无法修改基类的private可见性**

```js
class Base {
  private x = 0
}
class Derived extends Base {
  x = 1; // Error：属性“x”在类型“Base”中是私有属性，但在类型“Derived”中不是。
}
```

**跨实例private访问**

TS允许跨实例 private 访问

```js
class A {
  private x = 10;
  public sameAs(other : A) {
    return other.x === this.x; // OK
  }
}
```

> 注意：private protected 只在类型检查过程中执行，这意味着，在JS运行时，还是可以访问到private或protected的成员。



### 8.3、私有字段#

使用私有字段`#`，在编译为 JS 后保持私有。

```js
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor () {}
}

let dog = new Dog()
console.log(dog.barkAmount); // 输出undefined
```

- 私有字段上不能使用成员访问修饰符。
- 私有字段不能在包含的类之外访问，甚至不能被检测到。



### 8.4、访问器

通过`setter`和`getter`实现数据的封装和有效性校验。

```js
class Employee {
  private _fullName : string = "123";
  get fullName() : string {
    return this._fullName;
  }
  set fullName(newName : string) {
    this._fullName = newName;
  }
}

const employee = new Employee();
console.log(employee.fullName);
employee.fullName = "dylan"
console.log(employee.fullName);
```



### 8.5、类的继承 extends

“子承父业”，类与类之间、接口与接口之间最常见的关系。

**继承中的方法覆盖**

子类需要遵循其基类的规则，例如父类的一个方法不接收参数，子类的同名方法不能强制required参数，可以使用可选。

```js
class Base {
  greet() {
    console.log('hello world!');
  }
}
class Derived extends Base {
  greet(name?: string) {
    if(name === undefined) {
      super.greet()
    } else {
      console.log(`HELLO, ${name.toUpperCase()}`);
    }
  }
}
const d = new Derived();
d.greet(); // hello world!
d.greet("reader"); // HELLO, READER
// 通过基类引用派生类
const b: Base = d;
b.greet() // hello world!
```

假设，我们就是不遵守父类的规则，会发生什么？

```js
class Base {
  greet() {
    console.log('hello world!');
  }
}
class Derived extends Base {
  // Error：类型“Derived”中的属性“greet”不可分配给基类型“Base”中的同一属性。
  // Error：不能将类型“(name: string) => void”分配给类型“() => void”。
  greet(name: string) {
    console.log(`HELLO, ${name.toUpperCase()}`);
  }
}
```



### 8.6、抽象类 abstract

抽象类：提供抽象方法，不可实例化，一般作为基类存在。

`abstract`用于修饰抽象类和抽象方法

```js
abstract class Base {
  abstract getName() : string;
  printName() {
    console.log("Hello, " + this.getName());
  }
}
const b = new Base(); // Error：无法创建抽象类的实例
```

正常使用：

```js
abstract class Base {
  abstract getName() : string;
  printName() {
    console.log("Hello, " + this.getName());
  }
}
class Print extends Base {
  getName() {
    return 'dylan'
  }
}
const b = new Print();
b.printName()
```



## 九、泛型

泛型：让一个函数接受不同类型参数的一种模板。

### 9.1、泛型接口

```js
interface GenericIdentityFn<Type> {
  (arg : Type) : Type;
}
// Type为泛型，这里可以保证函数入参和返回值的类型一致
function identity<Type>(arg : Type) : Type {
  return arg;
}
const myIdentity : GenericIdentityFn<number> = identity
```



### 9.2、泛型类

静态成员不能使用类型参数，因为静态成员是通过构造函数访问的。

```js
class GenericNumber<NumType> {
  zeroValue!: NumType;
  add!: (x : NumType, y : NumType) => NumType;
}
const myGenericNumber = new GenericNumber<number>()
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) {
  return x + y;
}
```



### 9.3、泛型约束

#### (1) 类型参数约束

```js
function loggingIdentity<Type>(arg : Type) : Type {
  console.log(arg.length); // Error：类型“Type”上不存在属性“length”。
  return arg;
}
```

要求：我们希望将这函数的类型参数限制为有length属性的。

```js
function loggingIdentity<Type extends { length : number }>(arg : Type) : Type {
  console.log(arg.length);
  return arg;
}
```

**应用：使用泛型创建工厂模式**

```js
// 通用工厂：泛型 + call 签名
function create<Type>(c : { new() : Type }) : Type {
  return new c();
}
// 动物园工厂
class BeeKeeper {
  hasMask : boolean = true;
}
class ZooKeeper {
  name : string = "dylan";
}
class Animal {
  num : number = 4;
}
class Bee extends Animal {
  keeper : BeeKeeper = new BeeKeeper();
}
class Zoo extends Animal {
  keeper : ZooKeeper = new ZooKeeper();
}
function createAnimal<A extends Animal>(c : { new() : A }) : A {
  return new c();
}

console.log(createAnimal(Bee).keeper.hasMask);
```

#### (2) 类型参数相互约束

使用类型参数约束另外一个类型参数

```js
function getProperty<Type, Key extends keyof Type>(obj : Type, key : Key) {
  return obj[key];
}
const x = { a: 'a', b: 'b' }
getProperty(x, 'a')
getProperty(x, "m") // Error：类型“"m"”的参数不能赋给类型“"a" | "b"”的参数。
```



### 9.4、泛型参数的默认类型

```js
function createArray<T = string>(length : number, value : T) : Array<T> {
  const result : T[] = [];
  for(let i=0; i<length; i++) {
    result[i] = value
  }
  return result
}
```



## 十、TS装饰器

### 10.1、装饰器简介

**什么是装饰器？**

- 它是一个表达式
- 该表达式被执行后，返回一个函数
- 函数的入参分别为 target、name 和 descriptor
- 执行该函数后，可能返回 descriptor 对象，用于配置 target 对象

**装饰器的分类**

- 类装饰器（Class decorators）
- 属性装饰器（Property decorators）
- 方法装饰器（Method decorators）
- 参数装饰器（Parameter decorators）

需要注意的是，若要启用实验性的装饰器特性，你必须在命令行或 `tsconfig.json` 里启用 `experimentalDecorators` 编译器选项

`tsconfig.json`

```json
{
  "compilerOptions": {
     "target": "ES5",
     "experimentalDecorators": true
   }
}
```



### 10.2、类装饰器

类装饰器：用于装饰类，接收一个参数（被装饰的类），返回一个函数。

**类装饰器声明模板：**

```js
declare type ClassDecorator = <TFunction extends Function>(
  target : TFunction
) => TFunction | void
```

**例子：**

```js
// 装饰器 Greeter 接收一个参数，返回一个函数
// 该函数的入参仅使用了 target
// 在函数内给 target 的原型对象上增加属性或方法
function Greeter(greeting : string) {
  return function(target : Function) {
    target.prototype.greet = function() : void {
      console.log(greeting);
    }
  }
}
// 使用装饰器，并传参
@Greeter("Hello typescript")
class Greeting {
  constructor() {
    // 内部实现
  }
}
const myGreeting = new Greeting();
(myGreeting as any).greet(); // Hello typescript
```



### 10.3、属性装饰器

属性装饰器：用于装饰类的属性，接受两个参数（目标对象、属性名）

**属性装饰器声明模板：**

```js
declare type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void;
```

**例子：**

```js
function defaultValue(value : string) {
  return function(target : any, propertyName : string) {
    target[propertyName] = value
  }
}
class HelloWordClass {
  constructor() {
    console.log('构造函数');
  }
  @defaultValue("dylan")
  private name : string | undefined
}
const p = new HelloWordClass()
console.log(p["name"]); // dylan
```



### 10.4、方法装饰器

方法装饰器：用于装饰类的方法，接受三个参数（被装饰类、方法名、描述符）。

**方法装饰器声明模板：**

```js
declare type MethodDecorator = <T>(
  target: Object, 
  propertyKey: string | symbol, 
  descriptor: TypedPropertyDescriptor<T>
) => TypedPropertyDescriptor<T> | void;
```

**例子：**`log方法装饰器`，劫持方法

```js
function log(target : Object, propertyKey : string, descriptor : PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function(...args : any[]) {
    console.log("======before function======= ", propertyKey);
    const result = originalMethod.apply(this, args);
    console.log("======after function======= ", propertyKey);
    return result;
  }
}
class Task {
  @log
  runTask(arg : any) : any {
    console.log("task start run：", arg);
    return "finished";
  }
}
const task = new Task();
const result = task.runTask("learn ts");
console.log("result：", result);
// 输出：
// ======before function=======  runTask
// task start run： learn ts
// ======after function=======  runTask
// result： finished
```



### 10.5、参数装饰器

参数装饰器：装饰函数的参数，接收三个参数（被装饰的类、方法名、参数索引值）

**参数装饰器声明模板：**

```js
declare type ParameterDecorator = (
  target: Object, 
  propertyKey: string | symbol, 
  parameterIndex: number
) => void;
```

**例子：**

```js
function log(target : Function, key : string, parameterIndex : number) {
  const functionLogged = key || target.prototype.constructor.name;
  console.log(`位于${functionLogged}第${parameterIndex}个参数`);
}
class Gretter {
  greeting : string;
  constructor(@log name : string, @log type : string) {
    this.greeting = name;
  }
}
```



## 十一、条件类型

### 11.1、条件类型的基本使用

使用`extends?x:y;`，如果`T`的类型 是 `U`的类型的 **子集**，那么取结果`X`，否则取结果`Y`，类似于三元表达式。

```js
T extends U ? X : Y
```

**应用：**

一个createLabel函数，接受为number/string，此时使用函数重载实现

```js
interface IDLabel {
  id : number
}
interface NameLabel {
  name : string
}
function createLabel(id : number) : IDLabel
function createLabel(name : string) : NameLabel
function createLabel(nameOrId : string | number) : NameLabel | IDLabel
function createLabel(nameOrId : string | number) : NameLabel | IDLabel {
  if(typeof nameOrId === 'string') return { name: nameOrId }
  return { id: nameOrId }
}
```

使用条件类型修改：

```js
type NameOrId<T extends number|string> = T extends number ? IDLabel : NameLabel;
function createLabel<T extends number|string>(idOrName : T) : NameOrId<T> {
  throw "unimplemented";
}
```



### 11.2、infer类型推理

条件类型使用 inter 进行类型推理

inter：表示在 `extends` 条件语句中待推断的类型变量。

inter 声明的这个变量只能在 true 分支中使用。

```js
type Params<T> = T extends (...args : infer P) => any ? P : T
interface User {
  name : string
  age : number
}
type Func = (user : User) => void
type MyParam = Params<Func> // type MyParam = [user: User]
type AA = Params<string> // type AA = string
```



### 11.3、条件类型约束

希望效果：获取message，如果没有message则类型为never

```js
type MessageOf<T> = T["message"]; // Error：类型“"message"”无法用于索引类型“T”。
```

原因：T不一定有message属性

修改：约束T的范围，判断T是否有message，没有则never

```js
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;
```



## 十二、模板文字类型

- Uppercase<StringType>：将字符串转大写
- Lowercase<StringType>：将字符串转小写
- Capitalize<StringType>：将字符串第一个字符转大写，其他不变
- Uncapitalize<StringType>：将字符串第一个字符转小写，其他不变



## 十三、模块

TS 使用 `import type` 用于类型的导入。

```js
export type Cat = { bread : string }
import type { Cat } from './type.ts'
```

导入时内联type表明引入的是类型

```js
import { createCatName, type Cat, type Dog } from './animal.ts';

export type Animals = Cat | Dog;
const name = createCatName();
```



## 十四、内置类型别名

- `Partial<Type>`：将某个类型的属性全变为可选项。
- `Required<Type>`：将某个类型的属性全变为必选项。
- `Readonly<Type>`：将某个类型所有属性变为只读属性
- `Record<Keys,Type>`：将 K 中所有的属性的值转化为 T 类型。

```js
type Record<K extends keyof any, T> = {
  [P in K] : T
}
```

例子：

```js
interface CatInfo {
  age : number
  breed : string
}
type CatName = 'miffy' | 'boris' | 'mordred'
const cats : Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: 'Persian' },
  boris: { age: 5, breed: 'Maine Coon' },
  mordred: { age: 16, breed: 'British Shorthair' },
}
cats.boris // (property) boris: CatInfo
```

- `Pick <Type,Keys>`：从Type中取出一组属性Keys构造新的类型。
- `Omit<Type,Keys>`：从Type中删除Keys属性，构造类型。
- `Exclude<Type,ExcludedUnion>`：获取Type类排除ExcludedUnion后的联合成员。

```js
type T0 = Exclude<"a"|"b"|"c", "a">; // type T0 = "b" | "c"
type T1 = Exclude<"a"|"b"|"c", "a"|"b">; // type T1 = "c"
type T2 = Exclude<string|number|(()=>void), Function>; // type T2 = string | number
```

- `Extract<Type,Union>`：获取Type可分配给Union的联合成员，即求同。
- `NonNullable<Type>`：排除Type中的null、undefined后，得到的类型。
- `Parameters<T>`：获得函数的参数类型组成的元组类型。
- `ConstructorParameters<Type>`：获取构造函数参数类型的元组类型。
- `ReturnType<T>`：获取函数 Type 的返回类型。
- `InstanceType<Type>`：获取构造函数类型的实例类型。
- `ThisParameterType<Type>`：获取函数的this参数的类型
- `ThisType<T>`：指定上下文对象类型





























