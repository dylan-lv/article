#  初始化component主流程

目的，将下面的代码 run 起来

```html
// example/helloword/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    .red{ color: red; }
    .blue{ color: blue; }
  </style>
</head>
<body>
  <div id="app"></div>
  <script src="main.js" type="module"></script>
</body>
</html>
```

```js
// example/helloword/main.js
const rootContainer = document.querySelector("#app");
createApp(App).mount(rootContainer);
```

```js
// example/helloword/App.js
export const App = {
  name: "App",
  render() {
    // ui
    return h("div", "hi, mini-vue", )
  },
  setup() {
    return {
      msg: 'mini-vue111'
    }
  }
}
```



思维导图如下：

<img src=".\assets\2.png" alt="2"  />



## 构建入口，作为导出的出口文件

```ts
// runtime-core/index.ts
export { createApp } from "./createApp";
export { h } from "./h";
```



## createApp

- `createApp` 接收一个组件实例，返回一个对象
- 对象中包含一个 `mount` 方法
- `mount` 方法接收一个 `element` 实例（根容器）

```ts
// runtime-core/createApp.ts
import { render } from "./render"
import { createVNode } from "./vnode"

export function createApp(rootComponent) {
  return {
    mount(rootContainer) {
      // 先转换成 vnode，后续所有的逻辑操作，都会基于虚拟节点做处理
      // component -> vnode
      const vnode = createVNode(rootComponent)

      // 开始进一步的处理
      render(vnode, rootContainer)
    }
  }
}
```

## vnode

```ts
// runtime-core/vnode.ts
export function createVNode(type, props?, children?) {
  const vnode = {
    type,
    props,
    children,
  };
  return vnode;
}
```

## render

```ts
// runtime-core/render.ts
export function render(vnode, container) {
  patch(vnode, container); // 方便后续的递归处理
}
```

## patch

```ts
// runtime-core/render.ts
function patch(vnode, container) {
  // 处理组件
  processComponent(vnode, container)
  // TODO 处理元素
}
```

## processComponent

```ts
// runtime-core/render.ts
function processComponent(vnode, container) {
  // 挂载组件
  mountComponent(vnode, container)
  // TODO 更新组件
}
```

## mountComponent

```ts
// runtime-core/render.ts
function mountComponent(vnode, container) {
  // 抽离出 instance 实例，表示组件实例
  const instance = createComponentInstance(vnode)
  setupComponent(instance)
  // TODO setupRenderEffect
}
```

### createComponentInstance

```ts
// runtime-core/component.ts
export function createComponentInstance(vnode){
  const component = {
    vnode
  }
  // 先简单的挂过来，之后再做别的处理
  return component
}
```

## setupComponent

```ts
// runtime-core/component.ts
export function setupComponent(instance){
  // TODO initProps
  // TODO initSlots
  // 处理component调用setup之后的返回值（初始化一个有状态的component）
  // 无状态的conponent是“函数组件”
  setupStatefulComponent(instance)
}
```

### setupStatefulComponent

- 首先要获取到用户给的配置
  - 可以知道 vnode 的 type 属性，就是一开始的 rootComponent
  - 修改 createComponentInstance

```ts
export function createComponentInstance(vnode){
  const component = {
    vnode,
    type: vnode.type
  }
  return component
}
```

```ts
function setupStatefulComponent(instance) {
  // 首先要获取到用户给的配置
  const Component = instance.type;

  const { setup } = Component;
  if(setup) {
    // setup 可以返回 function 或 Object
    // function：组件的render函数
    // Object：会把Object对象注入到当前组件上下文中
    const setupResult = setup()

    // 处理setup的结果
    handleSetupResult(instance, setupResult)
  }
}
```

### handleSetupResult

```ts
function handleSetupResult(instance, setupResult) {
  // 基于上述的两种情况（setup可能会返回function或object）来做实现
  
  // TODO function
  if(typeof setupResult === "object") {
    // 将对应的值赋值到组件实例上
    instance.setupState = setupResult
  }
  // 保证组件的 render 一定是有值的
  finishComponentSetup(instance)
}
```

### finishComponentSetup

```ts
function finishComponentSetup(instance) {
  const Component = instance.type
  if(Component.render) {
    // 将组件上的render函数赋值给instance实例
    instance.render = Component.render
  }
}
```



## setupRenderEffect

```ts
function mountComponent(vnode, container) {
  // 抽离出 instance 实例，表示组件实例
  const instance = createComponentInstance(vnode)
  setupComponent(instance)
  setupRenderEffect(instance, container)
}
```

```ts
function setupRenderEffect(instance, container) {
  // 获取render函数的返回值（返回的是组件的虚拟节点树）
  const subTree = instance.render()
  // 基于返回的虚拟节点，对其进行patch比对（打补丁）
  patch(subTree, container)
}
```















