# 实现Fragment和Text类型节点



首先我们回顾一下上一节中 `slot` 的示例，这个示例中有个问题：渲染出来的 `slot` 都是通过 `div` 包裹的（多了一层 `div`）

![8](F:\study\文章积累\本地文章\Vue3\mini-vue3\assets\8.png)

多这一层 `div` ，是因为我们之前要解决 **`slot` 中的`children` 里面不能有 `array`** 这个问题

这一节的 `Fragment` 就可以解决这个问题了

```ts
// src\runtime-core\helpers\renderSlots.ts
export function renderSlots(slots, name, props){
  const slot = slots[name];
  if(slot) {
    if(typeof slot === "function") {
      // 只需要把第三个参数（也就是这里的 children）渲染出来就行了
      return createVNode("div", {}, slot(props))
    }
  }
}
```

## 实现Fragment

### 修改 `renderSlots`

```ts
// src\runtime-core\helpers\renderSlots.ts
export function renderSlots(slots, name, props){
  const slot = slots[name];
  if(slot) {
    if(typeof slot === "function") {
      // 只需要把第三个参数（也就是这里的 children）渲染出来就行了
      // 将 div 修改为 Fragment（特殊的type）
      return createVNode("Fragment", {}, slot(props))
    }
  }
}
```

### `patch`函数（渲染工作）

```ts
// src\runtime-core\renderer.ts
function patch(vnode, container) {
  const { type, shapeFlag } = vnode

  // Fragment -> 只渲染 children
  switch(type) {
    case "Fragment":
      processFragment(vnode, container)
      break;
    default: // 不是特殊的类型，继续走之前的逻辑
      if (shapeFlag & ShapeFlags.ELEMENT) {
        // 处理元素
        processElement(vnode, container)
      } else if (shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
        // 处理组件
        processComponent(vnode, container)
      }
      break;
  }
}
```

### 实现 `processFragment`

```ts
// src\runtime-core\renderer.ts
function processFragment(vnode, container) {
  // Fragment其实就只需要渲染所有的children就行了
  mountChildren(vnode, container)
}
```

此时可以看到包裹的 `div` 已经没了



### 优化

`patch` 函数中的 `switch` 这里写死了字符串 `Fragment`

以及 `renderSlots` 函数中也有一个字符串 `Fragment`

```ts
// src\runtime-core\renderer.ts
function patch(vnode, container) {
  const { type, shapeFlag } = vnode

  // Fragment -> 只渲染 children
  switch(type) {
    case "Fragment":
      processFragment(vnode, container)
      break;
    default: // 不是特殊的类型，继续走之前的逻辑
      // ...
  }
}
// src\runtime-core\helpers\renderSlots.ts
export function renderSlots(slots, name, props){
  const slot = slots[name];
  if(slot) {
    if(typeof slot === "function") {
      return createVNode("Fragment", {}, slot(props))
    }
  }
}
```

所以我们可以把它抽离出来，将其放入虚拟节点中

```ts
// src\runtime-core\vnode.ts
export const Fragment = Symbol("Fragment")
```

进行替换

```ts
// src\runtime-core\helpers\renderSlots.ts
import { createVNode, Fragment } from "../vnode";
export function renderSlots(slots, name, props){
  const slot = slots[name];
  if(slot) {
    if(typeof slot === "function") {
      return createVNode(Fragment, {}, slot(props))
    }
  }
}

// src\runtime-core\renderer.ts
function patch(vnode, container) {
  const { type, shapeFlag } = vnode
  switch(type) {
    case Fragment:
      processFragment(vnode, container)
      break;
    default:
      // ...
  }
}
```



## 实现 Text

修改测试用例

```js
// example\componentSlot\App.js
export const App = {
  name: "App",
  render() {
    const app = h("div", {}, "App")
    const foo = h(Foo, {}, {
      header: ({age}) => [
        h("p", {}, "headerrrrr" + age),
        "你好！！" // 添加text文本节点
      ],
      footer: () => h("p", {}, "footerrrrrr"),
    })
    return h("div", {}, [app, foo])
  },
  setup() {
    return {}
  }
}
```

目前的代码无法做到。

因为 `element` 需要指定标签类型（div、p等等），但是 `text` 节点没有任何标签，所以它也是一个需要特殊处理的节点

在这里，我们封装一个渲染函数

```js
// example\componentSlot\App.js
export const App = {
  name: "App",
  render() {
    const app = h("div", {}, "App")
    const foo = h(Foo, {}, {
      header: ({age}) => [
        h("p", {}, "headerrrrr" + age),
        createTextVNode("你好！！") // 封装渲染函数
      ],
      footer: () => h("p", {}, "footerrrrrr"),
    })
    return h("div", {}, [app, foo])
  },
  setup() {
    return {}
  }
}
```

### `createVNode`函数

```ts
// src\runtime-core\vnode.ts
export const Text = Symbol("Text")
export function createTextVNode(text: string){
  return createVNode(Text, {}, text)
}
```

### `patch`实现

```ts
// src\runtime-core\renderer.ts
function patch(vnode, container) {
  const { type, shapeFlag } = vnode
  switch(type) {
    case Text:
      processText(vnode, container)
      break;
    // ...
  }
}
```

### `processText`实现

```ts
function processText(vnode, container) {
  const { children } = vnode
  const textNode = vnode.el = document.createTextNode(children)
  container.append(textNode)
}
```

### 导出

```ts
// src\runtime-core\index.ts
export { createTextVNode } from "./vnode"
```

实际上我们在使用 `Vue` 的时候，是不需要写 `createTextVNode` 对文本节点进行包裹的，`Vue` 的编译模块会帮我们把文本节点包裹 `createTextVNode`





































