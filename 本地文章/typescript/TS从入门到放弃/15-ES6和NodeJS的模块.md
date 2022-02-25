# TS从入门到放弃【十五】：ES6和NodeJS的模块

因为我们接下来就会开始讲 **TS 的模块系统**了，所以在此之前，先来看看 **ES6 标准** 和 NodeJS中**CommonJS标准**的模块。

通过对这两个模块的学习，我们就能更方便的掌握 TS 的模块系统了。**TS 是按照 ES6的模块标准来实现的**。



## 一、ES6的模块

### 1、export

导出模块内容，一个模块一般就是一个文件。这个文件中的所有变量，外部都是无法获取的，除非使用 `export` 导出这个变量。

```js
// a.js
export const name = "dylan";
export const age = 18;
export const address = "guangzhou";
```

上述代码可以使用**解构赋值**，导出一个对象。

```js
// a.js
const name = "dylan";
const age = 18;
const address = "guangzhou";
export { name, age, address };
```



- 导出函数和类

```js
export function func() {}
export class A {}
```



- 使用 `as` 重命名

```js
function func1() {}
class B {}
const b = ''
export {
	func1 as Function1,
  B as ClassB,
  b as StringB,
  b as String
}
```

引用的时候就通过 `Function1`、`ClassB`、`StringB`、`String` 这几个名字 来调用。例如：`import { ClassB } from './a';`



- `export` 命令导出的是**对外的接口**，而不是具体的值。（例如：不能直接 `export` 一个字符串）

```js
export 'dylan'; // Error
const name = 'dylan';
export name; // Error
```



- `export` 导出的内容与其对应的值是**动态绑定**的。

也就是说在一些地方引入了导出的**接口（不是TS中的接口）**之后，当这个**接口在其模块中发生了变化**后，引入的地方也会相应的变化。

```js
// b.js
export let time = new Date();
setInterval(() => {
  time = new Date();
}, 1000);
// index.js
import { time } from "./b";
setInterval(() => console.log(time), 500);
```

在 `b.js`文件中，我们每秒更新 `time` 的值；

在 `index.js` 文件中，我们每半秒打印一下引入的 `time` 的值。可以发现打印的内容是一直在改变的。



- `export`可以出现在模块中处于**模块顶层**的任何位置

要求的是 `export` 不能出现在**模块作用域**里。ES6的模块设计是**静态编译**的，也就是我们引入的模块在**编译代码**的时候就已经引入过来了，而不是在代码执行的时候引入的。

例如把 `export` 放在 `if` 作用域中，是会报错的：

```js
if(true) {
  export const time1 = new Date(); // Error：修饰符不能出现在此处。
}
```





### 2、import

引入并加载一个模块（文件）

```js
// a.js
export const name = 'dylan';
export const age = 18;
// index.js
// 多个 export 语句导出的内容，在引入的时候可以用解构赋值的方式
import { name, age } from './a.js'
```



- 使用 `as` 关键字起别名

```js
import { name as nameProp, age } from './a.js'
console.log(nameProp); // dylan
```



- 引入的内容是只读的，如果赋值会报错

```js
import { name, age } from './a.js'
name = 'a'; // Error：无法为“name”赋值，因为它是导入。
```



- 如果引入的是一个对象的话，就可以修改它的属性

```js
// a.js
export const object = { name: 'dylan' };
// index.js
import { object } from './a.js'
object.name = 'haha';
console.log(object); // {name: 'haha'}
```

不建议改写从模块里引入的内容，会导致其他引入该对象的模块也发生变化。

```js
// b.js
import { object } from "./a";
setTimeout(() => {
  console.log(object); // {name: 'haha'}
}, 500);
```

可以看到， `b.js` 中的 `object` 也被改变了。



> `import` 和 `export`一样是静态编译的，所以在编译之前就得确定好文件路径（不能使用计算出的路径）。



- 多次引入只会执行一次，不会重复执行

```js
// b.js
console.log('haha');
// index.js
import './b';
import './b';
```

可以看到控制台只会打印一条 `haha`。



- 同样的，从一个模块中引入多个内容，会进行合并

```js
// a.js
export const name = 'dylan';
export const age = 18;
// index.js
import { name } from './a'
import { age } from './a'
```

在 `index.js` 中两次从模块 `a.js` 中引入了内容，但实际上它只会执行一次（一次性拿出`name`和`age`），相当于 `import { name, age } from './a'`



- 在 ES6 的模块中，我们可以使用 `*` 来引入一个模块里的全部内容，并且把它赋给一个变量

```js
// a.js
export const name = 'dylan';
export const age = 18;
export const object = { name: 'dylan' };
// index.js
import * as info from './a';
console.log(info);
```

打印出的内容：`{name: 'dylan', age: 18, object: { name: 'dylan' }, __esModule: true}`



### 3、export default

`export default` 就是输出一个名叫 `default` 的变量（方法）。

使用 `export default` 导出一个模块默认的内容，一个模块只能使用一次 `export default`。

```js
// a.js
export default function getName() {
  console.log('dylan')
}
// index.js
import func from './a'; // 名字可以与引入模块内容不一样
func(); // dylan
```



- 使用 `as` 来导出和引入

```js
// a.js
function getName() {
  console.log('dylan')
}
export { getName as default };

// index.js
import { default as func } from './a';
```



### 4、import和export的复合写法

```js
// a.js
import func from './b'
export default func

// index.js
import func from './a'
```

在 `index.js` 中，其实是使用了 `b.js` 中的内容，但是走了一层 `a` 模块

简便的写法：

```js
// a.js
export { default as func } from './b';

// index.js
import { func } from './a'
```

因为这种模式的 `export` 和 之前的一样了（`export const name = 'dylan'`），所以引入的时候只能通过 `import { func } from './a'` 这种模式来使用。

> `export default` 后面不能跟变量声明语句，但是可以直接接一个值（'a'、1）的



- 如果一个模块中既有 `export` 又有 `export default`，就可以一起导出

```js
// a.js
export const name = 'dylan'
export const age = 18
const sex = 'man'
export default sex

// index.js
import sex, { name, age } from './a'
```



- 复合写法

```js
export { name, age } from './a'
// 相当于
import { name, age } from './a'
export { name, age }
```



### 5、import() 方法

import() 方法可以用作动态加载，返回一个 Promise。这个方案还没有加入标准，但是 `webpack` 等编译工具已经把它实现了，用作一些异步加载。

```js
// a.js
document.title = 'haha';
// index.js
const state = 1;
if(state) {
  import('./a');
}
```



## 二、NodeJS的模块

`NodeJS` 模块是遵循 `CommonJS` 规范的，语法和 `ES6` 语法不同。

`NodeJS` 模块分为两种：内置模块（fs等）、用户自定义模块

### 1、exports

```js
// b.node.js
exports.name = "dylan";
exports.age = 18;

// a.node.js
const info = require("./b.node");
console.log(info); // { name: 'dylan', age: 18 }
```

可以看到导出了一个对象。

`exports` 的效果很像 ES6 中的 `export`，最后引入模块的时候都是引入了一个对象，导出的接口都作为对象的属性。



### 2、module.exports

可以直接导出一个接口，效果和 ES6 的 `export default` 相似。

```js
// b.node.js
module.exports = function () {
  console.log("dylan");
};

// a.node.js
const print = require("./b.node");
print();
```





