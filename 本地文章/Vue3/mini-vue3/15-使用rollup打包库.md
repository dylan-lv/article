# 使用rollup打包库

## 添加rollup

```bash
npm i rollup -D
npm i tslib @rollup/plugin-typescript -D
```



## 配置rollup

`rollup` 是天然支持 `esm` 的，所以我们可以直接写 `esm` 语法

```js
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
export default {
  input: "./src/index.ts",
  output: [
    // 库一般会打包多种类型：cjs、esm
    {
      format: "cjs",
      file: "lib/guide-mini-vue.cjs.js"
    },
    {
      format: "es",
      file: "lib/guide-mini-vue.esm.js"
    }
  ],
  plugin: [
    // 需要把ts编译一下，rollup是不理解ts的
    typescript()
  ]
}
```

```json
// package.json
"scripts": {
  "build": "rollup -c rollup.config.js"
},
```

- -c：指定其配置文件



## 优化

将 `rollup.config.js` 中的 `file` 写到 `package.json` 中

```json
// package.json
{
  "main": "lib/guide-mini-vue.cjs.js",
  "module": "lib/guide-mini-vue.esm.js"
}
```

```js
// rollup.config.js
import pkg from './package.json';
export default {
  output: [
    {
      format: "cjs",
      file: pkg.main
    },
    {
      format: "es",
      file: pkg.module
    }
  ],
}
```





## 入口文件

```ts
// src/runtime-core/index.ts
export { createApp } from "./createApp";

// src/index.ts
export * from "./runtime-core/index";
```



## 打包

执行命令

```bash
npm run build
```

即可看到 `lib` 文件夹下的两个文件了



## h方法

我们的 `example` 中除了 `createApp` 方法外，还有一个 `h` 方法

```ts
// runtime-core/index.ts
export { createApp } from "./createApp";
export { h } from './h'
```

 ```ts
 // runtime-core/h.ts
 import { createVNode } from './vnode'
 
 export function h(type, props?, children?){
   return createVNode(type, props, children)
 }
 ```



## 运行example

此时 `example` 中的 `main.js` 就可以使用打包后的内容了

```ts
// main.js
import { createApp } from '../../lib/guide-mini-vue.esm.js'
import { App } from './App.js'

const rootContainer = document.querySelector("#app");
createApp(App).mount(rootContainer);
```

```js
// App.js
import { h } from '../../lib/guide-mini-vue.esm.js'

export const App = {
  name: "App",
  render() {
    return h("div", "hi, mini-vue")
  },
  setup() {
    return {
      msg: 'mini-vue111'
    }
  }
}
```

使用 `Open with Live Server` 打开 `index.html`

此时会发现控制台报错：`instance.render is not a function`

这是因为我们第一次的 `vnode` 是组件类型，进一步 `patch` 渲染的时候这个 `vnode` 是 `element` 类型的 

回来看看我们写的 `patch` 函数，这里还有一个 `TODO` ，需要处理 `element` 类型

```ts
function patch(vnode, container) {
  // 处理组件
  processComponent(vnode, container)
  // TODO 处理元素
}
```

这个问题等下一节再做处理~













