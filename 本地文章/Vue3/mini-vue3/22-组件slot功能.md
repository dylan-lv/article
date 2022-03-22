# 组件slot功能

实现组件的slot功能，我们先来新建一下测试代码

```js
// App.js
export const App = {
  name: "App",
  render() {
    const app = h("div", {}, "App
    const foo = h(Foo, {}, h("p", {}, "123"))

    return h("div", {}, [app, foo])
  },
  setup() {
    return {}
  }
}
// Foo.js
export const Foo = {
  setup() {
    return {}
  },
  render() {
    const foo = h("p", {}, "foo")
    return h("div", {}, [foo]);
  }
}
```

我们希望将 `App.js` 中 `foo` 里的 `h("p", {}, "123")` ，这个 `p` 标签添加到 `Foo.js` 组件内。

其实就是获取 `Foo` 组件的 `vnode` 中的 `children`

```ts
// Foo.js
export const Foo = {
  setup() {
    return {}
  },
  render() {
    const foo = h("p", {}, "foo")
    // return h("div", {}, [foo]);
    return h("div", {}, [foo, this.$slots]);
  }
}
```



## componentPublicInstance

我们需要在 `componentPublicInstance.ts` 文件中实现，因为它（$slots）是 `$` 开头的，说明它其实是给用户提供的一个 `api`

```ts
// src\runtime-core\componentPublicInstance.ts
const publicPropertiesMap = {
  $el: (i) => i.vnode.el,
  $slots: (i) => i.slots
}
```



## 构建 slots

```ts
// src\runtime-core\component.ts
export function createComponentInstance(vnode){
  const component = {
    ...
    slots: {}
  }
}
```

```ts
export function setupComponent(instance){
  initProps(instance, instance.vnode.props)
  // 初始化 slots
  initSlots(instance, instance.vnode.children)
  setupStatefulComponent(instance)
}
```

```ts
// src\runtime-core\componentSlots.ts
export function initSlots(instance, children){
  // 先进行粗暴的赋值
  instance.slots = children
}
```

此时已经可以成功渲染出来了：我们在外部定义的 `p` 标签，在 `foo` 组件内容渲染出来了，这就是插槽的概念。



## 传入数组

```js
// App.js
export const App = {
  name: "App",
  render() {
    const app = h("div", {}, "App")
    // 传入数组
    const foo = h(Foo, {}, [h("p", {}, "22222"), h("p", {}, "33333")])

    return h("div", {}, [app, foo])
  },
  setup() {
    return {}
  }
}
```

此时会发现页面上并没有渲染出来 slots 的两个 vnode

这是为什么呢，我们来看一下 `Foo` 组件

```js
// Foo.js
export const Foo = {
  setup() {
    return {}
  },
  render() {
    const foo = h("p", {}, "foo")
    console.log(this.$slots); // 此时 slots 是一个数组
    // 我们h函数内部必须都是 vnode，不可以是数组形式。需要将其转换成 vnode
    return h("div", {}, [foo, this.$slots]);
  }
}
```

```js
// 转换之后
return h("div", {}, [
  foo, 
  h("div", {}, this.$slots)
]);
```

此时页面可以正常渲染。

### 封装

但是我们希望使用之前的用法，并可以正常渲染，这时就需要封装一下了

```ts
// src\runtime-core\helpers\renderSlots.ts
import { createVNode } from "../vnode";

export function renderSlots(slots){
  return createVNode("div", {}, slots)
}
```

```ts
// src\runtime-core\index.ts
export { createApp } from "./createApp";
export { h } from './h'
export { renderSlots } from './helpers/renderSlots'
```

```ts
export const Foo = {
  setup() {
    return {}
  },
  render() {
    const foo = h("p", {}, "foo")
    console.log(this.$slots);
    return h("div", {}, [
      foo, 
      renderSlots(this.$slots) // 将其替换成 renderSlots
    ]);
  }
}
```

但是现在变成只支持数组，并不支持单个节点了

所以我们需要再做一下处理，使其同时支持数组和单个vnode

- 如果不是数组的话，将其包裹一下变成数组

```ts
// src\runtime-core\componentSlots.ts
export function initSlots(instance, children){
  instance.slots = Array.isArray(children) ? children : [children]
}
```



## 指定渲染位置

- 获取到要渲染的元素
- 获取到渲染的位置

将数组变为 `object`，通过 `key` 来精确到要渲染的元素

```js
// App.js
export const App = {
  name: "App",
  render() {
    const app = h("div", {}, "App")
    // 将数组替换为对象
    const foo = h(Foo, {}, {
      header: h("p", {}, "headerrrrr"),
      footer: h("p", {}, "footerrrrrr"),
    })

    return h("div", {}, [app, foo])
  },
  setup() {
    return {}
  }
}
```

```js
// Foo.js
export const Foo = {
  setup() {
    return {}
  },
  render() {
    const foo = h("p", {}, "foo")
    console.log(this.$slots);
    return h("div", {}, [
      renderSlots(this.$slots, "header"),
      foo, 
      renderSlots(this.$slots, "footer")
    ]);
  }
}
```

```ts
// src\runtime-core\helpers\renderSlots.ts
export function renderSlots(slots, name){
  const slot = slots[name];
  if(slot) {
    return createVNode("div", {}, slot)
  }
}
```

同时，初始化 `slots` 的时候也要做一些处理：之前将其作为一个数组了，现在是一个对象

```ts
// src\runtime-core\componentSlots.ts
export function initSlots(instance, children){
  // instance.slots = Array.isArray(children) ? children : [children]
  const slots = {}
  for (const key in children) {
    const value = children[key]
    slots[key] = Array.isArray(value) ? value : [value]
  }
  instance.slots = slots
}
```

现在，页面上就可以看到渲染的内容了，并且它们都在**正确的位置**上。



### 重构

```ts
// src\runtime-core\componentSlots.ts
export function initSlots(instance, children){
  normalizeObjectSlots(children, instance.slots)
}

function normalizeObjectSlots(children, slots) {
  for (const key in children) {
    const value = children[key]
    slots[key] = normalizeSlotValue(value)
  }
}

function normalizeSlotValue(value) {
  return Array.isArray(value) ? value : [value]
}
```



## 作用域插槽

上面实现的其实就是**具名插槽**，现在我们来实现一下作用域插槽。

作用域插槽：可以将组件内部的变量传出去。

```js
// App.js
export const App = {
  name: "App",
  render() {
    const app = h("div", {}, "App")
    const foo = h(Foo, {}, {
      header: ({age}) => h("p", {}, "headerrrrr" + age),
      footer: () => h("p", {}, "footerrrrrr"),
    })

    return h("div", {}, [app, foo])
  },
  setup() {
    return {}
  }
}
```

```js
// Foo.js
export const Foo = {
  setup() {
    return {}
  },
  render() {
    const foo = h("p", {}, "foo")
    const age = 18
    return h("div", {}, [
      renderSlots(this.$slots, "header", {
        age
      }),
      foo, 
      renderSlots(this.$slots, "footer")
    ]);
  }
}
```

```ts
// src\runtime-core\helpers\renderSlots.ts
export function renderSlots(slots, name, props){
  const slot = slots[name];
  if(slot) {
    // slot是一个function了
    if(typeof slot === "function") {
      return createVNode("div", {}, slot(props))
    }
  }
}
```

```ts
// src\runtime-core\componentSlots.ts
function normalizeObjectSlots(children, slots) {
  for (const key in children) {
    const value = children[key]
    // 直接调用一下 value 
    slots[key] = (props) => normalizeSlotValue(value(props))
  }
}
```



### 重构/优化

```ts
// src\runtime-core\vnode.ts
export function createVNode(type, props?, children?) {
  ...
  // 组件 + children（object）：才需要slots
  if(vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
    if(typeof children === "object") {
      vnode.shapeFlag |= ShapeFlags.SLOT_CHILDREN
    }
  }

  return vnode;
}
```

```ts
// src\shared\ShapeFlags.ts
export const enum ShapeFlags {
  ELEMENT = 1, // 0001
  STATEFUL_COMPONENT = 1 << 1, // 0010
  TEXT_CHILDREN = 1 << 2, // 0100
  ARRAY_CHILDREN = 1 << 3, // 1000
  SLOT_CHILDREN = 1 << 4,
}
```

```ts
// src\runtime-core\componentSlots.ts
export function initSlots(instance, children) {
  // 检查一下是否需要slots处理
  const { vnode } = instance
  if (vnode.shapeFlag & ShapeFlags.SLOT_CHILDREN) {
    normalizeObjectSlots(children, instance.slots)
  }
}
```

























