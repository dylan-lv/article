## 一、概述

随着前端技术的不断发展，TypeScript(简称:TS)已经在逐步取代JavaScript(简称:JS),尤其在以Vue3使用TS重构后，TS更是成为前端框架编写的主力语言。

- 在使用TS的时候，最大的一个好处就是可以给JS各种类型约束，使得JS能够完成静态代码分析，推断代码中存在的类型错误或者进行类型提示
- TS完成类型推断，需要事先知道变量的类型，如果我们都是用TS书写代码，并且给变量都指定了明确的类型，这时TS可以很好的完成类型推断工作
- 但是有时，我们不免会引入外部的 JS库，这时TS就对引入的JS文件里变量的具体类型不明确了，为了告诉TS变量的类型，因此就有了.d.ts (d即declare)，ts的声明文件。

**类型定义文件d.ts(TypeScript Declaration File)**



## 二、什么是“.d.ts” 文件

​		基于 Typescript 开发的时候，很麻烦的一个问题就是类型定义。导致在编译的时候，经常会看到一连串的找不到类型的提示。“d.ts”文件用于为 TypeScript 提供有关用 JavaScript 编写的 API 的类型信息。简单讲，就是你可以在 ts 中调用的 js 的声明文件。

​		TS的核心在于静态类型，我们在编写 TS 的时候会定义很多的类型，但是主流的库都是 JS编写的，并不支持类型系统。

​		这个时候你不能用TS重写主流的库，这个时候我们只需要编写仅包含类型注释的 d.ts 文件，然后从 TS 代码中，可以在仍然使用纯 JS 库的同时，获得静态类型检查的 TS 优势。在此期间，解决的方式经过了许多的变化，从**DefinitelyTyped**到 **typings**。最后是 **@types**。在 Typescript 2.0 之后，推荐使用**@types**方式。



### 2.1 @types

​		在 Typescript 2.0 之后，TypeScript 将会默认的查看 ./node_modules/@types 文件夹，自动从这里来获取模块的类型定义，当然了，你需要独立安装这个类型定义。

​		默认情况下，所有的 @types 包都会在编译时应用，任意层的 node_modules/@types 都会被使用，进一步说，在 `./node_modules/@types/` , `../node_modules/@types/`, `../../node_modules/@types/` 都被应用。如果你的类型定义不在这个文件夹中，可以使用 typesRoot 来配置，只有在 typeRoots 中的包才会被包含，配置如下：

```json
{
   "compilerOptions": {
       "typeRoots" : ["./typings"]
   }
}
```

现在，只有在 ./typings 中的才会应用，而 ./node_modules/@types 中的则不会。 如果配置了 types，则只有列出的包才会包含。

```json
{
   "compilerOptions": {
       "types" : ["node", "lodash", "express"]
   }
}
```

这样将只会包含 `./node_modules/@types/node`, `./node_modules/@types/lodash` 和 `./node_modules/@types/express `，其它的则不会被包含进来。如果配置为`"types": []`则不会包含任何包。



### 2.2 *.d.ts和@types关系

`@types`是`npm`的一个分支，用来存放`*.d.ts`文件，如果对应的`npm`包存放在`@types`中，要使用必须下载！如果是自己本地的`*.d.ts`申明文件，则和@types没有任何关系！



## 三、编写语法

从类型`type`角度分为：基本类型（string、number、boolean、undefined、symbol）其混合 下面我们介绍下 “.d.ts” 的几种声明的写法



### 3.1 全局类型

- 变量
- 函数
- 用interface 声明函数
- class
- 对象
- 混合类型
- 模块化的全局变量



### 3.2 模块化的全局变量

定义全局变量的时候需要引入(别人写的)文件



### 3.3 模块化（CommonJS）

通过require的方式引入模块化的代码

```js
// d.ts
declare module "ever" {
    export let a: number
    export function b(): number
    export namespace c{
        let c: string
    }
 }
 // 引用
 cosnt ever = require('ever)
 ever.a = 100
 ever.b = function() {
     return 100 + 300
 }
```



### 3.4 ES6的模块化方式（import export）

```js
export declare let a1: 1
export declare let a2: 2
// 或
declare let a1: 1
declare let a2: 2

export { a1,a2 }
```



### 3.5 UMD

有一种代码，既可以通过全局变量访问到，也可以通过require的方式访问到。

```js
declare namespace ${
    let a:number
}
 
declare module "$" {
    export = $
}
```



### 3.6 其他

有时候我们扩展了一些内置对象。给Date的内置对象扩展方法

```js
interface Date {
    format(f: string): string
}
```



## 四、案例

```js
/** 作为函数使用 */
declare function People(w: number): number
declare function People(w: string): number

declare class People {
    /** 构造函数 */
    constructor(name: string, age: number)
    constructor(id: number)

    // 实例属性和实例方法
    name: string
    age: number
    getName(): string
    getAge(): number

    /** 作为对象，调用对象上的方法或者变量 */
    static staticA(): number
    static aaa: string
}

/** 作为对象，调用对象上的方法或者变量 */
declare namespace People {
    export var abc: number
}
```

























