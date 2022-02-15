# TS从入门到放弃【七】：类（基础部分）

本章为**es6中class**的继承部分，如已熟练使用可以跳过本章。

在 ts 中使用 class 的地方很多，并且class作为es6基础也有很多知识点，这里单独提出一章来细讲 class。



### 使用es5创造实例方法

在es6发布之前，如何使用es5创建来创建实例方法？

```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}
Point.prototype.getPosition = function() {
  return '(' + this.x + ', ' + this.y + ')';
}
```

接下来，我们来使用一下这个构造函数

```js
var p1 = new Point(2, 3);
console.log(p1); // Point {x: 2, y: 3}
console.log(p1.getPosition()); // (2, 3)
```

当使用 `new` 操作符调用 `Point` 构造函数时，可以理解为：

- 首先会创建一个空的对象
- 然后为这个对象设置属性 x 和 y，它们的值就是传入的 x 和 y
- 最后把这个对象（this）返回，并赋值给 `p1`

定义完构造函数后，我们紧接着在它上面定义了一个 `getPosition` 方法，这个实际上是在 Point 对象上定义的。创建实例 `p1`后，实例会继承这个方法。



### 使用es6的class来定义构造函数

将刚才的例子使用 es6 来编写

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  getPosition() {
    return `(${this.x}, ${this.y})`;
  }
}
const p1 = new Point(10, 20);
console.log(p1.getPosition()); // (10, 20)
```

跟刚才es5定义的效果是一样的。

这里 `constructor` 中的 `this` 指向用户创建的实例，`getPosition` 则创建在 `Point` 的原型对象上的。



### 查看属性是否是自身拥有的

- hasOwnProperty

```js
console.log(p1.hasOwnProperty('x')); // true
console.log(p1.hasOwnProperty('getPosition')); // false
console.log(p1.__proto__.hasOwnProperty('getPosition')); // true
```

上面的代码说明 getPosition 方法实际上是 p1 继承来的



### get、set关键字

先来看看在es5中是如何使用的

```js
var info = {
  _age: 18,
  set age(newVal) {
    this._age = newVal;
    if(newVal > 18) {
      console.log('怎么变老了？')
    } else {
      console.log('我又年轻了！')
    }
  },
  get age() {
    console.log('谁敢问我年龄？')
    return this._age;
  }
}
console.log(info.age); // 谁敢问我年龄？  18
info.age = 17; // 我又年轻了！
```

在es6中的class也可以使用 get、set

```js
class Info {
  constructor(age) {
    this._age = age;
  }
  set age(newVal) {
    this._age = newVal;
  }
  get age() {
    return this._age;
  }
}
```



### class表达式

首先，函数有两种定义形式

```js
const func1 = function(){};
function func2() {};
```

class 也有两种定义形式

```js
class Infos1 {
  constructor() {}
}
const Infos2 = class c {
  constructor() {}
}
// Infos2 等同于 Infos3
const Infos3 = class {
  constructor() {}
}
```



### 静态方法

我们定义类的时候，定义在类里面的方法都会被实例继承，例如之前代码中的 `getPosition` 方法

使用 `static` 关键字来标记静态方法

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  getPosition() {
    return `(${this.x}, ${this.y})`
  }
}
```

接下来我们定义一个**静态方法**

```js
class Point {
  ...
  static getClassName() {
    return Point.name
  }
}
```

我们来测试一下

```js
const p = new Point(1, 2);
console.log(p.getPosition()); // (1, 2)
console.log(p.getClassName()); // Error：p.getClassName is not a function
```

发现调用 `getClassName` 时报错了，这是因为**实例**不能继承**类的静态方法**

那么我们怎么样才能调用 `getClassName` 方法呢---当然是这个类自身才能调用

```js
console.log(Point.getClassName()); // Point
```



### 实例属性的其他写法

我们之前的实例属性都是在 `constructor` 中直接在 `this` 上添加属性

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}
```

还有一种方法，就是在外层定义

```js
class Point {
  z = 0 // 这里可以不设置初始值
  constructor(x, y) {
    ...
  }
}
```

然后打印 `new Point()`，即可看到 `point` 实例上有一个 `z` 属性了



### 静态属性

要给类添加静态属性的话，可以使用如下方式：

```js
class Point {
  constructor() {
    this.x = 0;
  }
}
Point.y = 2;
const p = new Point();
console.log(p.x); // 0
console.log(p.y); // undefined
console.log(Point.y); // 2
```

这样就成功给类添加了静态属性，当然这种方式显得十分繁琐，下面使用 `static` 来给类添加静态属性

```js
class Point {
  static y = 1;
  constructor() {
    this.x = 0;
  }
}
console.log(p.x); // 0
console.log(p.y); // undefined
console.log(Point.y); // 1
```



### 私有方法

我们封装插件的时候，有一些方法是希望只在内部使用的，不希望暴露给使用者。

目前 es6 并不支持私有方法和私有属性，所以我们需要使用一些技巧来实现。

- 通过命名：一般以**下划线**开头的方法代表是私有方法，不希望使用者使用。
  - 缺点：只是从名字来区分，别人要用的话还是能用的

```js
class Point {
  func1() {}
  _func2() {}
}
```

- 将私有方法移除模块：如果我们封装了模块，模块里面没有导出的 _func2 方法，在模块外面是没办法调用到的。

```js
const _func2 = () => {}
export class Point {
  func1() {
    _func2.call(this);
  }
}
```

- 利用 `Symbol` 的唯一性：用 `Symbol` 作为方法名，这样在模块外就无法访问到方法了。

```js
// a.js
const func1 = Symbol('func1');
export class Point {
  [func1]() {
    ...
  }
}
// b.js
import Point from 'a.js'
const p = new Point()
console.log(p)
```

打印 `p` 之后，可以在里面看到 `[[Prototype]]` 上是有私有方法的 `Symbol(func1): ƒ [func1]()`

此时如果想调用的话，必须要拿到这个 `Symbol` 值才行，但是 `Symbol` 又是唯一的，所以是没办法调用的。

至此我们通过 `Symbol` 实现了私有的效果。



### 私有属性

我们可以在属性前添加一个 `#`，来定义一个属性为**私有属性**。

```js
class Point {
    #x = 1;
}
let p = new Point();
p.x; // undefined
```



### new.target属性

作用：可以判断当前是否是使用 `new` 来调用的构造函数

一般用于构造函数中，它会返回 `new` 命令作用于的构造函数

```js
function Point() {
  console.log(new.target);
}
const p = new Point(); // 会打印 Point 构造函数
const p2 = Point(); // 如果是普通的直接调用，就会返回 undefined
```

同样的，在 `class` 中也可以使用 `new.target`

```js
class Point {
  constructor() {
    console.log(new.target);
  }
}
const p = new Point(); // 会打印 Point 构造函数
```

在类的继承中，`new.target`会返回子类而不是父类

```js
class Parent {
  constructor() {
    console.log(new.target);
  }
}
class Child extends Parent {
  constructor() {
    super();
  }
}
const c = new Child(); // 这里会打印子类的构造函数（虽然是在父类中使用的 new.target）
```



我们如果想定义一个类，这个类不能直接用来创建实例，只能通过继承它的子类来实例化。

此时就可以使用 `new.target` 了

```js
class Parent {
  constructor() {
    if(new.target === Parent) {
      throw new Error('不能实例化');
    }
  }
}
class Child extends Parent {
  constructor() {
    super();
  }
}
const p = new Parent(); // Error：不能实例化
const c = new Child(); // OK
```









