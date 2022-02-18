# TS从入门到放弃【八】：类（进阶部分）

本章为**es6中class**的进阶部分，如已熟练使用可以跳过本章。

在 ts 中使用 class 的地方很多，并且class作为es6基础也有很多知识点，这里单独提出一章来细讲 class。



### 1、es5的继承

在es5中如果想实现一个构造函数的继承，是需要修改原型链的。

```js
function Food() {
  this.type = 'food';
}
Food.prototype.getType = function() {
  return this.type;
}
```

`Food` 创建的实例可以继承 `getType` 方法。

接下来实现继承

```js
function Vegetables(name) {
  this.name = name;
}
Vegetables.prototype = new Food();
const tomato = new Vegetables('tomato')
console.log(tomato.getType()); // food
```

将实例对象 `new Food()` 赋给 `Vegetables` 原型对象，这样的话，在使用 `Vegetables` 构造函数创建实例的时候就能继承 `Food` 的方法。



### 2、es6类的继承

es6中的类实际上是构造函数的语法糖。es6的类转换成es5后还是类似上面es5代码的构造函数。

一起来看下面的例子，

```js
class Parent {
  constructor(name) {
    this.name = name;
  }
  getName() {
    return this.name;
  }
}
```

接下来创建一个 `Child` 类，继承 `Parent`

```js
class Child extends Parent{
  constructor(name, age) {
    super(name);
    this.age = age;
  }
}
```

> 给 `super` 传入的参数，相当于给父类构造函数的 `constructor` 传入的参数。
>
> 只有调用了 `super` 方法之后，才可以使用 `this`。
>
> 这是因为es5的构造函数和es6的类，这两种形式在继承时的机制上存在差异
>
> - es5中会先创建子构造函数的实例 this，然后再将父构造函数的属性、方法添加到 this上
> - es6的类则会先到父类取到实例对象 this，然后在调用 super 函数之后，再将子类属性和方法添加到 this 上
>   - 这也就是为什么我们要先调用 super 方法，然后才能使用 this

现在我们来创建实例，并查看结果：

```js
const c = new Child('dylan', 18);
console.log(c);
console.log(c.getName()); // dylan
```

其中 `c` 就是 Child 的实例，`c.getName()` 会输出 `dylan`

我们来看看 `c` 的实例判断

- c 是 Child 的实例：`c instanceof Child` 返回 true
- c 是 Parent 的实例：`c instanceof Parent` 返回 true

所以说，使用继承了父类的子类创建的实例，它即是子类的实例，又是父类的实例。

**子类会将父类的静态方法也都继承过来**。效果就相当于父类的静态方法在子类又定义了一遍。

```js
class Parent {
  ...
  static getNames() {
    return this.name;
  }
}
console.log(Child.getNames()); // Child
```

`getNames`方法中的 `this` 在这里指向了子类。name属性就是构造函数（类）的名字（Child）



**判断一个类的父类**

```js
console.log(Object.getPrototypeOf(Child) === Parent); // true
```

说明 `Child` 的原型就是 `Parent`



### 3、super

super即可以作为函数使用，又可以作为对象使用。

- super作为函数：代表父类的构造函数 `constructor`
  - 子类的构造函数必须调用一次 `super` 函数，之后才可以在 `constructor` 中使用 `this`。
  - 只能在 `constructor` 中调用，在其他地方调用就会报错。
  - 子类中调用 `super`的时候，父类中 `constructor` 的 `this` 指向子类的实例。
- super作为对象：
  - 在普通方法中，指向父类的原型对象。
  - 在静态方法中，指向父类本身。



接下来，我们来看看super作为对象的示例代码：

我们先定义一个父类

```js
class Parent {
  constructor() {
    this.type = 'parent';
  }
  getName() { // 普通方法
    return this.type;
  }
}
// 静态方法
Parent.getType = () => {
  return 'is parent'
}
```

在普通方法中，super指向父类的原型对象，我们来定义一个 `Child` 类

```js
class Child extends Parent {
  constructor() {
    super();
    console.log('constructor: '+ super.getName()); // super作为对象
  }
  getParentName() {
    console.log('getParentName: '+ super.getName());
  }
  getParentType() {
    console.log('getParentType: ' + super.getType());
  }
}
```

然后创建一个 `child` 实例

```js
const child = new Child();
```

此时控制台打印： `constructor: parent`，说明 `super` 此时是指向父类的原型对象的。注意：此时 super 是作为对象使用的。

```js
child.getParentName();
```

此时控制台打印：`getParentName: parent`，说明 `super` 在普通方法里也可以作为对象使用。



我们再来尝试一下调用父类本身定义的方法 `getType`

```js
child.getParentType(); // Error
```

会发现报错了，这是因为 `super` 指代的是父类的原型对象，而不是父类本身。



**在静态方法中，super指向父类本身**

```js
class Child extends Parent {
  ...
  static getParentType() {
    console.log('getParentType: ' + super.getType());
  }
}
Child.getParentType(); // getParentType: is parent
```



**在父类的原型定义方法，方法里的this指向子类的实例。**

代码示例：

```js
class Parent {
  constructor() {
    this.name = 'parent';
  }
  print() {
    console.log(this.name);
  }
}
class Child extends Parent {
  constructor() {
    super();
    this.name = 'child';
  }
  childPrint() {
    super.print();
  }
}
const child = new Child()
child.childPrint(); // child
```

可以看到这里打印的是 `child`，因为父类方法中的 `this` 实际上指向子类，子类的 `name` 是 `child`



### 4、`prototype` 和 `__proto__`

`__proto__` 不是es标准中定义的属性，而是大多数浏览器厂商在对 es5 的实现中添加的。

每一个对象都有一个 `__proto__` 属性，它指向对应的构造函数的 `prototype` 属性。

```js
var objs = new Object();
console.log(objs.__proto__ === Object.prototype); // true
```



- 子类的 `__proto__` 指向父类本身
- 子类的 `prototype` 属性的 `__proto__` 指向父类的 `prototype` 属性
- 实例的 `__proto__` 属性的 `__proto__` 指向父类实例的 `__proto__`