# TS从入门到放弃【九】：TS中的类



我们先来看一个在TS中定义类的简单例子：

```ts
class Point {
  public x: number;
  public y: number;
  constructor (x: number, y: number){
    this.x = x;
    this.y = y;
  }
  public getPosition () {
    return `(${this.x}, ${this.y})`;
  }
}
const point = new Point(1, 2);
console.log(point); // Point {x: 1, y: 2}
```



### 1、修饰符

​		默认情况下，定义在实例的属性和方法会在创建实例后，添加到实例上。也就是我们显示的定义在 `this` 上的属性和方法，最后都会在实例上真实存在，作为它自己的属性和方法。

​		如果是定义在类里，没有定义在 `this` 上的方法，实例则可以继承这个方法。

​		而使用 `static` 修饰符修饰的属性和方法，是静态属性和静态方法。实例是没法访问和继承到的。

**class修饰符：**

- **public**（默认）：表示公共的属性和方法，用来指定创建实例后，可以被实例访问的属性和方法。
- **private**（私有）：只能被自己内部访问，在类的外面不能访问，该类被继承也不能访问private属性和方法
- **protected**（受保护）：可以被自己和继承该类的子类访问；子类访问时，只允许使用方法



现在有如下需求：

​		父类只提供**继承**，不能直接创建实例。（使用 `protected 修饰符`）

```js
class Parent {
  protected constructor(age: number) {
    this.age = age;
  }
}
```



### 2、readonly

使用继承来复用一些特性。

在类里可以使用 `readonly` 修饰符，将属性设置为**只读**

```ts
class UserInfo {
  public readonly name: string;
  constructor(name: string) {
    this.name = name;
  }
}
const userInfo = new UserInfo('dylan');
console.log(userInfo.name); // dylan
userInfo.name = 'haha'; // Error：无法分配到 "name" ，因为它是只读属性。
```

在给 `userInfo.name` 赋值的时候，代码就会报错。



### 3、参数属性

参数属性简单来说就是在 `constructor` 构造函数的参数前面，加上 `public`、`protected`、`private`、`readonly` 这几个访问修饰符之一。

```ts
class A {
  // 这里同时会帮你把name放到实例上（因为使用了 public 修饰符）
  constructor(public name: string) {}
}
```



### 4、静态属性

实例不会添加静态属性，也不会继承静态属性和方法。

```ts
class Parent {
  public static age: number = 18;
  private static nane: string = 'dylan';
  public static getAge() {
    return Parent.age;
  }
  constructor() {}
}
const p = new Parent();
console.log(p.age); // Error：拿不到
console.log(Parent.age); // 18：因为age是类的静态属性，只有类本身可以拿到
console.log(Parent.nane); // Error：因为name是私有属性，在类的外面无法访问
```



### 5、可选类属性

```ts
class Info {
  public name: string;
  public age?: number;
  constructor(name: string, age?: number, public sex?: string) {
    this.name = name;
    this.age = age;
  }
}
```

使用 Info 类创建实例：

```ts
const info0 = new Info();  // Error：因为第一个参数 name 是必选属性
const info1 = new Info('dylan'); 
const info2 = new Info('dylan', 18); 
const info3 = new Info('dylan', 18, 'man'); 
```

可以看出，第二个参数和第三个参数都是可选参数。



### 6、抽象类

一般用于被其他类继承，而不用于直接创建实例。抽象方法和抽象存取器，都不能有实际的代码块

```ts
abstract class People {
  constructor(public name: string) {}
  public abstract printName(): void;
}
```

上面的代码定义了一个 `printName` 抽象方法。

可以试一下使用 `People` 类能否创建实例：

```ts
const p1 = new People(); // Error：无法创建抽象类的实例
```

此时可以看到控制台报错。

我们使用 `Man` 这个类，继承这个抽象类

```ts
class Man extends People {
  constructor(name: string) {
    super(name);
    this.name = name;
  }
  // 需要在具体类中实现抽象类的抽象方法
  public printName() {
    console.log(this.name);
  }
}
```

注意，如果具体类继承了抽象类，则需要在具体类里实现全部的抽象方法。



**abstract还可以标记类中的属性和存取器**

我们来看实例：

```ts
abstract class People {
  public abstract _age: number;
  abstract get insideName(): string;
  abstract set insideName(value: string);
  constructor(public name: string) {}
}
class Man extends People {
  // 这里需要实现抽象属性：_age、insideName
  public _age: number;
  public insideName: string;
  constructor(name: string) {
    super(name);
    this.name = name;
  }
}
```

抽象属性和方法，都不能包含实际的代码块，只需要指定属性名、方法名、参数和返回值类型就行了。



### 7、类类型的接口

使用接口可以强制一个类必须包含某些内容

```ts
interface FoodInterFace {
  type: string;
}
```

接口检测的是使用该接口定义的类创建的实例

```ts
class FoodClass implements FoodInterFace {
  public type: string; // 这里必须要有type，而且必须是 string 类型
}
```



### 8、接口继承类

接口继承类后，会继承这个类的成员和成员类型（包含private、protected），但是不包括其实现。

当接口继承的类中包括private、protected修饰符修饰的成员时，这个接口只能被这个类或者它的子类实现。

```ts
class A1 {
  protected name: string;
}
interface I extends A1 {}
// 这里会报错，因为受保护的属性，只能在继承这个类的子类中去访问
// B1 并没有继承 A1，所以这里不能实现 I 接口
class B1 implements I { 
  public name: string;
}
// 下面这样是可以的
class B2 extends A1 implements I {
  public name: string;
}
```



### 9、在泛型中使用类

```ts
// 这里表示：T这里代表实例，传入的是一个类，返回的是类创建的实例；new() 表示调用这个类的构造函数
const create = <T>(c: new() => T): T => {
  return new c();
};
class Infos {
  public age: number;
}
create(Infos);
```













