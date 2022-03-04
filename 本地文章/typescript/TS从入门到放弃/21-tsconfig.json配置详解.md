# TS从入门到放弃【二十一】：tsconfig.json配置详解



## 一、compilerOptions

编译选项配置



### 1、target

用于指定TS编译完之后的版本目标。

```json
{
  "compilerOptions": {
    "target": "es5"
  }
}
```

我们知道 TS 编译之后也是 JS 代码，而且我们输出的目标不同，它编译完的代码是不同的



### 2、module

用来指定使用的模块标准

```json
"module": "commonjs"
```



### 3、lib

lib 用于指定要包含在编译中的库文件。

```json
"lib": ["DOM", "ES6"]
```

- 如果要使用 ES6 的新语法，则需要引入 ES6 这个库
- 如果涉及到 DOM，则需要引入 DOM 这个库



### 4、allowJs

allowJs 设置的值为 true 或 false，用来指定是否允许编译 JS 文件，默认是 false，即不编译 JS 文件。



### 5、checkJs

checkJs 的值为 true 或 false，用来指定是否检查和报告 JS 文件中的错误，默认是 false。



### 6、jsx

指定 jsx 代码用于的开发环境：`preserve`、`react-native`、`react` 等



### 7、declaration

declaration 的值为true或false，用来指定是否在编译的时候生成相应的 `.d.ts` 声明文件。

如果设为 **true**，编译每个 ts 文件之后**会生成一个 js 文件和一个声明文件**。

但是 `declaration` 和 `allowJs` 不能同时设为 `true`



### 8、declarationMap

declarationMap 值为true或false，指定是否为声明文件 `.d.ts` 生成 map 文件



### 9、sourceMap

sourceMap 的值为true或false，用来指定编译时是否生成 .map 文件



### 10、outFile

outFile 用于指定将输出文件合并为一个文件，他的值为一个文件路径名。

比如设置为 `./dist/main.js`，则输出的文件为一个 `main.js` 文件。

但是要注意，只有设置 `module` 值为 `amd` 和 `system` 模块时才支持这个配置。



### 11、outDir

outDir 用来指定输出文件夹，值为一个文件夹路径字符串，输出的文件都将放置在这个文件夹。



### 12、rootDir

用来指定编译文件的根目录，编译器会在根目录查找入口文件，如果编译器发现以 `rootDir` 的值作为根目录查找入口文件并不会把所有文件加载进去的话会报错，但是不会停止编译。



### 13、composite

值为 true 或 false。

用于指定是否编译构建引用项目。



### 14、removeComments

值为 true 或 false。默认为 false

用于指定是否将编译后的文件中的注释删掉。



### 15、noEmit

不生产编译文件。---这个一般很少用了。



### 16、importHelpers

值为true或false。默认为 false

指定是否引入 tslib 里的辅助工具函数。



### 17、downlevelIteration

当 `target` 为 `ES5` 或 `ES3` 时，为 `for-of`、`spread` 和 `destructuring` 中的**迭代器**提供完全支持。



### 18、isolatedModules

值为true或false，默认为true

指定是否将每个文件作为单独的模块，它不可以和 `declaration` 同时设定。



### 19、strict

值为true或false，默认为false

用于指定是否启动所有类型检查，如果设为 true 则会同时开启下面这几个严格类型检查。



### 20、noImplicitAny

值为true或false，默认为 false

如果我们没有为一些值设置明确的类型，编译器会默认任务这个值为 `any` 类型。

如果将 `noImplicitAny` 设为 `true`，则如果没有明确的类型会报错。



### 21、strictNullChecks

值为true或false

当设置为`true`时，`null 和 undefined` 值不能赋值给非这两种类型的值，别的类型的值也不能赋给他们，除了 any 类型，还有个例外就是 `undefined` 可以赋值给 `void` 类型



### 22、strictFunctionTypes

值为true或false

用来指定是否使用函数参数双向协变检查。



### 23、strictBindCallApply

值为true或false

设为 true 后会将 `bind、call 和 apply` 绑定方法的参数检查变为严格检查。



### 24、strictPropertyInitialization

值为true或false，默认为 false

设为 true 后会检查类的非 `undefined` 属性是否已经在构造函数里初始化。

如果要开启这项，需要同时开启 `strictNullChecks`



### 25、noImplicitThis

当 this 表达式的值为 any 类型的时候，生成一个错误。



### 26、alwaysStrict

指定始终以严格模式检查每个模块，并且在编译之后的 JS 文件中加入 `"use strict"` 字符串，用来告诉浏览器该 JS 为严格模式。



### 27、noUnusedLocals

用于检查是否有定义了但是没有使用的变量。

对于这一点的检查，使用 ESLint 可以在你书写代码的时候做提示，可以配合使用。



### 28、noUnusedParameters

用于检查是否在函数体中有未使用的参数，这个也可以配合 ESLint 来做检查。



### 29、noImplicitReturns

用于检查函数是否有返回值。

设为 true 后，如何函数没有返回值则会报错



### 30、noFallthroughCasesInSwitch

用于检查 `switch` 中是否有 `case` 没有使用 `break` 跳出 `switch`



### 31、moduleResolution

用于选择模块解析策略，有 `node` 和 `classic` 两种类型。



### 32、baseUrl

用于设置解析非相对模块名称的基本目录，相对模块不会受 `baseUrl` 的影响。



### 33、paths

用于设置模块名到基于 `baseUrl` 的路径映射。



### 34、rootDir

可以指定一个路径列表，在构建时编译器会将这个路径列表中的路径内容都放在一个文件夹中。



### 35、typeRoots

用来指定**声明文件或文件夹的路径列表**。

如果指定了此项，则只有在这里列出的声明文件才会被加载。



### 36、types

用来指定需要包含的模块，只有在这里列出的**声明文件**才会被加载进来。



### 37、allowSyntheticDefaultImports

用来指定允许从没有默认导出的模块中默认导入。



### 38、esModuleInterop

通过为导入内容创建命名空间，实现 CommonJS 和 ES 模块之间的互操作性。



### 39、preserveSymlinks

不把符号链接解析为其真实路径，具体可以了解下 `webpack` 和 `nodejs` 的 `symlink` 相关知识。



### 40、sourceRoot

用于指定调试器应该找到 `TypeScript` 文件而不是源文件位置，这个值会被写进 `.map` 文件里。



### 41、mapRoot

用于指定调试器找到映射文件而非生成文件的位置，指定 `map` 文件的跟路径，该选项会影响 `.map` 文件中的 `sources` 属性。



### 42、inlineSourceMap

指定是否将 `map` 文件的内容和 `js` 文件编译在同一个 `js` 文件中。

如果设为 true，则 map 的内容会以 `//# sourceMappingURL=` 然后接**base64字符串**的形式插入在 js 文件底部。



### 43、inlineSources

用于指定是否进一步将 `.ts` 文件的内容也包含到输出文件中。



### 44、experimentalDecorators

指定是否启用实验性的装饰器特性。



### 45、emitDecoratorMetadata

指定是否为装饰器提供**元数据**支持。

关于**元数据**，也是**ES6的新标准**，可以通过 `Reflect` 提供的静态方法获取元数据。































## 二、files

```json
{
  "files": []
}
```

- files可以配置一个**数组列表**，里面包含指定文件的相对或绝对路径。
- 编译器在编译的时候只会编译包含在 files 中**列出的文件**。
- 如果不指定，则取决于有没有设置 include 选项
  - 如果没有 include 选项，则默认会编译根目录以及所有子目录中的文件。
- 这里列出的路径必须是指定文件，而不是某个文件夹，而且不能使用 * ? **/ 等通配符



## 三、include

```json
{
  "include": []
}
```

- include 也可以指定要编译的路径列表
- 但是和 files 的区别在于，这里的路径可以是文件夹，也可以是文件
- 可以使用相对和绝对路径，而且可以使用通配符
  - 比如 "./src" 即表示要编译 src 文件夹下的所有文件以及子文件夹的文件。



## 四、exclude

```json
{
  "exclude": []
}
```

- exclude表示要排除的、不编译的文件
- 他也可以指定一个列表，规则和include一样，可以是文件或文件夹
- 可以是相对路径或绝对路径，可以使用通配符



## 五、extends

```json
{
  "extends": ""
}
```

- extends 可以通过指定一个其他的 tsconfig.json 文件路径，来继承这个配置文件里的配置
- 继承来的文件配置会覆盖当前文件定义的配置。
- TS 在 3.2 版本开始，支持继承一个来自 Node.js 包的tsconfig.json配置文件。



## 六、compileOnSave

```json
{
  "compileOnSave"： true
}
```

- compileOnSave的值是true或false
- 如果设为true，在我们编译了项目中文件保存的时候，编译器会根据 tsconfig.json 的配置重新生成文件
  - 这个需要编辑器支持。



## 七、references

```json
{
  "references": []
}
```

- 一个对象数组，指定要引入的项目











