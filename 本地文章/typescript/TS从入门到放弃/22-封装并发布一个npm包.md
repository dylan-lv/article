# TS从入门到放弃【二十一】：封装并发布一个npm包



## 封装 array-map-dylan 方法

### 1、初始化

在 `array-map-dylan` 文件夹下执行

```bash
npm init -y
tsc --init
npm i -D typescript
```

在 `package.json` 中：

- 入口文件 `main`：用来指定使用这个库的人，在使用的时候引入的文件；而不是编译的入口文件。
  - 将其修改为：`./dist/array-map-dylan.js`

在 `tsconfig.json` 中：

- 将 `declaration` 设置为 `true`：打包后生成声明文件。
  - 在我们编译完成后，不仅会生成 `js` 文件，还会生成 `.d.ts` 文件
- 将 `outDir` 设置为 `./dist/`
  - 编译后的文件将打包到 `dist` 文件夹下
- 配置 `exclude`：`["./dist"]`
  - 打包时将忽略此文件夹



### 2、编写文件

库的作用：模拟 JS 数组中的 `map` 方法

```ts
// array-map.ts
const arrayMap = (array, callback) => {
  let i = -1;
  const len = array.length;
  const resultArray = [];
  while(++i < len) {
    resultArray.push(callback(array[i], i, array));
  }
  return resultArray;
};
export = arrayMap;
```

我们试着调用一下：

```ts
const result = arrayMap([1,2,3], item => item + 10)
```

打印 `result` ，结果为：`[11, 12, 13]`



### 3、定义类型

我们已经把具体的功能实现完了，但是还没有对类型进行定义。定义了类型之后，可以让使用者永远更友好的类型提示。

```ts
const arrayMap = (array: any[], callback: (item: any, index: number, arr: any[]) => any): any[] => {
  ...
};
```

我们可以看出，`arrayMap` 有两个参数和一个数组类型的返回值：

- 参数1：数组
  - 任意类型的数组
- 参数2：回调函数
  - 参数1：任意类型
  - 参数2：数字类型
  - 参数3：与 `arrayMap` 的第一个参数一致
  - 返回值：任意类型
- 返回值：数组
  - 任意类型的数组



### 4、执行编译

在根目录下执行命令 `tsc` 后，就能在 `dist` 文件夹下看到 `array-map.js` 和 `array-map.d.ts` 文件了。

- 在 js 环境下，`array-map.js` 毫无疑问是可以使用的
- 在 ts 环境下，示例如下：

```ts
// example/test.ts
import arrayMap from "../dist/array-map";
arrayMap([1,2,3], (item) => item + 10);
```

可以看到是没有问题的，并且我们把鼠标放在 `arrayMap` 上，可以看到对应的**类型声明**：

![ts-8](.\assets\ts-8.png)

现在我们使用 `tsc` 编译一下：`tsc example\test.ts`

就可以看到 `example/test.ts` 编译后的 js 文件了

```js
"use strict";
exports.__esModule = true;
var array_map_1 = require("../dist/array-map");
(0, array_map_1["default"])([1, 2, 3], function (item) { return item + 10; });
```



### 5、优化类型

我们上述的代码实现了**函数功能和类型**，但是类型中使用了很多 `any`，现在我们来将其优化一下：

我们先来看下函数：

```ts
const arrayMap = (array: any[], callback: (item: any, index: number, arr: any[]) => any): any[] => {
  ...
};
```

- 回调函数的参数1：`item: any` 的类型，应该和 `arrayMap` 参数1数组中内容的类型是一致的。
  - 这个要求他俩有关联，我们可以想到使用泛型

```ts
const arrayMap = <T>(array: T[], callback: (item: T, index: number, arr: any[]) => any): any[] => {
  ...
};
```

- 回调函数的第三个参数就是原数组本身：`arr: any[]) => any`，从es6的标准上来说，它应该是一个只读的，我们不希望在遍历的时候对原数组 `array` 进行修改
  - `ReadonlyArray` 是 TS 内置属性

```ts
const arrayMap = <T>(array: T[], callback: (item: T, index: number, arr: ReadonlyArray<T>) => any): any[] => {
  ...
};
```

- 回调函数返回的值，应该和 `arrayMap` 返回的值是有关系的。我们每一次遍历返回的元素，最后都会作为 数组中总的元素进行返回

```ts
const arrayMap = <T, U>(array: T[], callback: (item: T, index: number, arr: ReadonlyArray<T>) => U): U[] => {
  ...
};
```



### 6、查看效果

此时我们还没有对优化后的类型进行重新编译，可以先对比的看一下优化前后的效果：

我们改造一下测试文件 `test.ts`

```ts
import arrayMap from "../dist/array-map";
arrayMap([1,2,3], (item) => {
  return item + 10;
}).forEach(item => {
  console.log(item.length);
});
```

因为我们之前对 `arrayMap` 的返回值给予的是 any 类型的数组，所以即使返回的元素是数字类型，但是在后面使用 `.length` 属性，也不会报错。

![ts-9](.\assets\ts-9.png)

我们使用 `tsc` 来编译一下：在根目录输入 `tsc`

现在就可以看到有效的类型错误信息了！！！

![ts-10](.\assets\ts-10.png)

自此，我们的库就封装完成了。



### 7、npm发包

#### （1）登陆npm

在命令行输入：`npm login`

输入用户名和密码后登陆成功

#### （2）发包

- 要注意 `package.json` 中的 `name` 在 npm 上还未存在
- 每次发布需要修改版本号：`version`
- 新建 `.npmignore` 文件，它和 `.gitignore` 文件差不多
  - 发包时希望哪些文件被忽略掉（这里忽略 example 文件夹）

```bash
// .npmignore
/example/
```

- 发包：命令行输入 `npm publish`，即可发布成功







