# TS从入门到放弃【十】：枚举



**枚举**是TS新增的一种数据类型。

这在其他语言中其实是很常见的，在`javascript` 中却没有，但我们可以通过 `javascript` 模拟出这种效果。

使用**枚举**可以给一些我们难以理解的**常量**赋予一组**具有含义的直观的名字**。你可以理解**枚举**就是一个**字典**。TS支持数字和字符串两种枚举。



### 1、简单示例

我们来看一下数字枚举。

```ts
enum Status {
  Uploading,
  Success,
  Failed,
}
```

我们可以通过两种形式来获取到每一个枚举名字对应的编码。

```ts
console.log(Status.Uploading); // 0：默认是第一个是 0
console.log(Status["Uploading"]); // 0
```

枚举的值默认是依次递增的，上述代码中 `Status.Uploading` 值为 0，`Status.Success` 值为1，`Status.Failed` 值为2 。



**我们也可以自己指定枚举值**

```ts
enum Status {
  Uploading,
  Success = 3,
  Failed,
}
```

此时 `Status.Uploading` 值为 0，`Status.Success` 值为3，`Status.Failed` 值为4 。



**数字枚举**在设定值的时候，可以使用**计算值和常量**。但是要注意，如果某个字段使用了**计算值或常量**，那么**该字段后面紧接着的字段**必须要设置**初始值**。

```ts
const test = 1;
enum Status {
  Uploading = 3,
  Success = test,
  Failed, // Error：枚举成员必须具有初始化表达式。
}
```

上述代码中，ts会在 `Failed` 这里报错，因为它没有设置初始值



**同理我们来尝试一下计算值**

```ts
const getIndex = () => 3
enum Status {
  Uploading = 2,
  Success = getIndex(),
  Failed = 5
}
```



### 2、反向映射

一个**枚举**，它不仅可以通过**字段名**得到它的**枚举值**，还可以通过**枚举值**得到**字段名**。

在 https://www.tslang.cn/play/index.html 上可以看到 TS代码编译后的样子。

我们把下面的代码放进去：

```ts
enum Status {
  Uploading,
  Success,
  Failed
}
```

在右边可以看到代码编译成 js 后的样子

```js
var Status;
(function (Status) {
    Status[Status["Uploading"] = 0] = "Uploading";
    Status[Status["Success"] = 1] = "Success";
    Status[Status["Failed"] = 2] = "Failed";
})(Status || (Status = {}));
```

可以看到，代码会先给 `Status["Uploading"]` 的值设置为 0。同时这个表达式返回值是 0；外部会给 `Status[0]`赋值为 Uploading。

也就是：

```js
Status[Status["Uploading"] = 0] = "Uploading";
// 等同于
Status["Uploading"] = 0
Status[0] = "Uploading"
```

Success 和 Failed 同理。

这种方式就给 Status 对象即添加了**从字段名到值**的映射，也添加了**从值到字段名**的映射。



### 3、字符串枚举

**字符串枚举**的值要求每个字段的值都必须是**字符串字面量**，或者是该枚举值中另一个**字符串枚举成员**。

我们来看一个示例：

```ts
enum Message {
  Error = 'Sorry, error',
  Success = 'haha, success',
  Failed = Error
}
console.log(Message.Success); // haha, success
console.log(Message.Failed); // Sorry, error
```

> 这里的 “其他枚举成员”指的是**同一个枚举**中的成员。因为字符串枚举是不能使用常量或计算值的，所以也不能使用其他枚举中的成员。



### 4、异构枚举

异构枚举简单来说就是即包含数字值，又包含字符串值的枚举。

```ts
enum Status {
  Failed = 0,
  Success = 'success',
}
```



### 5、枚举成员类型和联合枚举类型

当一个枚举值它**满足一定条件**的时候，那么这个枚举的**每个成员和枚举值本身**都可以作为**类型**来使用。

1. 不带初始值的枚举成员：`enum E { A }`
2. 值为字符串字面量：`enum E { A = 'a' }`
3. 值为数值字面量，或者带有负号的数值：`enum E { A = 1 }`

如果一个枚举中，它的全部成员都**满足上述三种情况之一**，那么这个枚举中的**每个成员以及这个枚举本身**就可以**作为类型**来使用。



#### 枚举成员类型

```ts
enum Animals {
  Dog = 1,
  Cat = 2,
}
```

接下来，我们定义一个接口 `Dog`

```ts
interface Dog {
  type: Animals.Dog;
}
```

type后面接的是一个`类型`（string、number...这种），现在我们把 `Animals.Dog` 作为一个**类型**来使用。

接下来，我们定义一个对象来实现 `Dog` 接口

```ts
const dog: Dog = {
  type: Animals.Dog
  // 或者 type: 10
}
```

这是因为 ts会判断 `Animals.Dog` 是一个 `number` 类型



#### 联合枚举类型

当我们的枚举值符合刚才的三个条件之一时，那么这个枚举值它本身就可以看作是一个包含所有成员的**联合类型**。

```ts
enum Status {
  Off,
  On
}
interface Light {
  status: Status;
}
```

此时 status 的值必须是 `Status.Off` 或者 `Status.On`

```ts
const light: Light = {
  status: Status.Off
};
```

如果我们那刚才的 `Animals` 来赋值，就会报错了：

```ts
const light: Light = {
  status: Animals.Dog // Error：不能将类型“Animals.Dog”分配给类型“Status”。
};
```



### const enum

普通枚举值定义之后，它都会把我们这个枚举值编译成一个 js 中真实存在的对象。

但有时候我们使用枚举值可能只是用来提高代码的可读性（只是用来做数值对比）。

`const enum`会在编译完代码之后，将枚举成员替换成具体的值（不会再是枚举对象了），例如：

```ts
const enum Status {
  Success = 0,
  Failed = 1
}
// 判断
res.code === Status.Success
```

在编译之后会直接替换

```ts
// 判断
res.code === 0
```



我们在 https://www.tslang.cn/play/index.html 左侧写入如下代码：

```ts
const enum Animals {
  Dog = 1,
}
const dog = Animals.Dog
```

会被编译为：

```js
var dog = 1 /* Dog */;
```

**但如果我们把 const 去掉**

```js
var Animals;
(function (Animals) {
    Animals[Animals["Dog"] = 1] = "Dog";
})(Animals || (Animals = {}));
var dog = Animals.Dog;
```

编译结构就会创建 `Animals` 这个对象。









