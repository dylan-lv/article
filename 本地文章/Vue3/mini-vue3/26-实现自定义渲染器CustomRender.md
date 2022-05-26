# 实现自定义渲染器CustomRender

这个是 Vue3 给我们提供的一个高阶 API，允许我们自定义渲染接口，可以让我们的程序渲染到任意一个平台（因为我们默认的是渲染到DOM平台）



我们来看看之前实现的代码中，挂载元素的函数

```ts
// src\runtime-core\renderer.ts
function mountElement(vnode, container, parentComponent) {
  const el = vnode.el = document.createElement(vnode.type)
  // children可能是：string、array
  const { props, children, shapeFlag } = vnode

  if (shapeFlag & ShapeFlags.TEXT_CHILDREN) {
    el.textContent = children
  } else if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    // children 中每个都是 vnode，需要继续调用 patch，来判断是element类型还是component类型，并对其初始化
    // 重构：children.forEach(v => patch(v, el))
    mountChildren(vnode, el, parentComponent)
  }

  // props
  for (const key in props) {
    const value = props[key]

    // 判断是否是事件的命名规范
    const isOn = (key: string) => /^on[A-Z]/.test(key);
    if (isOn(key)) {
      const event = key.slice(2).toLowerCase()
      el.addEventListener(event, value)
    } else {
      el.setAttribute(key, value)
    }
  }

  container.append(el)
}
```

### canvas

- 创建节点

```ts
// DOM
const el = vnode.el = document.createElement(vnode.type)
// canvas  
// new Element()
```

- 属性

```ts
// DOM
el.setAttribute(key, value)
// canvas 
// el.x = 10
```

- 添加视图

```ts
// DOM
container.append(el)
// canvas
addChild()
```

虽然他们的具体实现不同，但是表达的是同样的渲染意图

我们可以将其封装，让其不再依赖于具体的实现，而是依赖于稳定的接口



### 封装接口

```ts
// src\runtime-core\renderer.ts
function mountElement(vnode, container, parentComponent) {
  // const el = vnode.el = document.createElement(vnode.type)
  // 新建节点--替换为稳定的接口
  const el = vnode.el = createElement(vnode.type)
  const { props, children, shapeFlag } = vnode

  if (shapeFlag & ShapeFlags.TEXT_CHILDREN) {
    el.textContent = children
  } else if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    // children 中每个都是 vnode，需要继续调用 patch，来判断是element类型还是component类型，并对其初始化
    mountChildren(vnode, el, parentComponent)
  }

  // props
  for (const key in props) {
    const value = props[key]
    // 添加属性--替换为稳定的接口
    patchProp(el, key, value)
    // const isOn = (key: string) => /^on[A-Z]/.test(key);
    // if (isOn(key)) {
    //   const event = key.slice(2).toLowerCase()
    //   el.addEventListener(event, value)
    // } else {
    //   el.setAttribute(key, value)
    // }
  }

  // container.append(el)
  // 添加到视图--替换为稳定的接口
  insert(el, container)
}
```



接下来需要考虑，如何让用户来提供这些渲染接口

可以提供一个渲染器（`createRender({ createElement })`），让用户可以传入渲染接口

### createRenderer

利用闭包，将全部函数放入 `createRenderer` 中

```ts
// src\runtime-core\renderer.ts
export function createRenderer(options){
  // 从对应的 options 中解构出所需的渲染接口
  const {
    createElement,
    patchProp,
    insert
  } = options
 	// 之前的全部函数 
}
```

现在我们的代码就不再依赖于具体的实现了，而是依赖于具体的渲染接口



### runtime-dom

创建 `runtime-dom` 文件夹，在这里实现基于 `DOM` 的渲染接口的实现

```ts
// src\runtime-dom\index.ts
import { createRenderer } from "../runtime-core";

function createElement(type) {
  return document.createElement(type)
}
function patchProp(el, key, value) {
  const isOn = (key: string) => /^on[A-Z]/.test(key);
  if (isOn(key)) {
    const event = key.slice(2).toLowerCase()
    el.addEventListener(event, value)
  } else {
    el.setAttribute(key, value)
  }
}
function insert(el, parent) {
  parent.append(el)
}

const renderer = createRenderer({
  createElement,
  patchProp,
  insert
})
```



### createApp

打开 `createApp.ts` 文件，会发现此时是报错的：我们之前从 `renderer.ts` 中导出 `render` 对象，而现在是没有的

我们可以将其改为高阶函数，外面套一层 `createAppApi`，传入 `render`

```ts
// src\runtime-core\createApp.ts
export function createAppApi(render){ // 在这里传入 render
  return function createApp(rootComponent){
    return {
      mount(rootContainer) {
        // 先转换成 vnode，后续所有的逻辑操作，都会基于虚拟节点做处理
        const vnode = createVNode(rootComponent)
        render(vnode, rootContainer)
      }
    }
  }
}
```

接着在 `renderer.ts` 中调用 `createAppApi`，并将其调用结果返回命名为 `createApp`

```ts
// src\runtime-core\renderer.ts
export function createRenderer(options){
  const {
    createElement,
    patchProp,
    insert
  } = options

  function render(vnode, container) {
    patch(vnode, container, null);
  }
  
  // ...

  return {
    createApp: createAppApi(render)
  }
}
```

现在 `createApp` 就有了，但是它是存在 `render` 内部的，我们需要返回给用户使用的 `createApp`

```ts
// src\runtime-core\index.ts
// 删掉  export { createApp } from "./createApp";
```

```ts
// src\runtime-dom\index.ts
const renderer: any = createRenderer({
  createElement,
  patchProp,
  insert
})
// 包装一层
export function createApp(...args){
  return renderer.createApp(...args)
}
```

```ts
// src\index.ts
// 导出 createApp
export * from "./runtime-dom"
```





















