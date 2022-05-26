# 更新element流程搭建

新建测试代码

```js
// example\update\App.js
import { h, ref } from "../../lib/guide-mini-vue.esm.js"

export const App = {
  name: "App",
  setup() {
    const count = ref(0);

    const onClick = () => {
      count.value++;
    }

    return { count, onClick }
  },
  render() {
    return h(
      "div",
      {
        id: "root"
      },
      [
        h("div", {}, "count:" + this.count), // 依赖收集
        h(
          "button",
          {
            onClick: this.onClick
          },
          "click"
        )
      ]
    )
  }
}
```

期望**响应式对象**发生改变了，**视图**也随之改变

此时页面显示的内容是：`count:[object Object]`。没有自动获取到 `ref` 的 `value`

我们希望的是 `this.count` 直接可以获取到 `count.value` 的值。这里可以借助我们之前写的 `proxyRefs`来解构 `ref`

对 `setup` 返回的值进行处理

```ts
// src\runtime-core\component.ts
function handleSetupResult(instance: any, setupResult: any) {
  if(typeof setupResult === "object") {
    instance.setupState = proxyRefs(setupResult) // 使用proxyRefs 包裹 setup 的返回值
  }
  finishComponentSetup(instance)
}
```

此时页面上显示的内容：`count:0`

接下来，我们希望点击按钮时，视图会更新

- 视图在显示之前都是虚拟节点
- 只不过这些虚拟节点包含了一些**视图渲染**所需要的属性：type、props、children
- 我们的更新逻辑就是让其重新生成虚拟节点，然后**对比**一下看看哪些节点需要更新，去更新它
- 更新逻辑 -> 两个对象的对比



### 生成虚拟节点的位置

在 `setupRenderEffect` 这个函数中会调用 `render` 函数，生成虚拟节点

因为我们在组件中的数据是响应式对象，我们需要的是响应式对象发生改变的时候重新调用 `render`

我们可以利用 `effect` 进行依赖收集，包裹 `render` 函数

```ts
// src\runtime-core\renderer.ts
function setupRenderEffect(instance, container) {
  effect(() => {
    const { proxy, vnode } = instance
    // 获取render函数的返回值（返回的是组件render的虚拟节点树）
    const subTree = instance.render.call(proxy)
    // 基于返回的虚拟节点，对其进行patch比对（打补丁）
    patch(subTree, container, instance)
  
    // 此处可以确定所有的 element 都被 mount 了
    vnode.el = subTree.el
  })
}
```

当调用 `render` 函数的时候，在 `render` 函数内就会触发依赖收集（这里的例子就是 `render` 中返回的 `count`）

- 此时会触发响应式对象中的 `get` 操作，
- 然后会收集当前传入 `effect` 的匿名函数，
- 当后续 `count` 发生改变了，就会触发它所有的依赖（也就是 `get` 收集的 `effect` 匿名函数）
- 当其触发 `effect` 匿名函数时，也就又会调用 `render` 函数了
- 然后就会返回一个全新的 `subTree`
- 这样我们其实就实现了：当响应式对象发生改变后，自动调用 `render`

此时页面上点击 `click` 按钮，就可以看到 `count` 值发生改变了，不过每次都会新增节点，这个是因为我们的 `patch` 目前还只有挂载功能



### 区别更新和挂载

在组件 `instance` 上添加一个参数 `isMounted` 来判断是否挂载

```ts
function setupRenderEffect(instance, container) {
  effect(() => {
    if(!instance.isMounted) {
      const { proxy, vnode } = instance
      // 获取render函数的返回值（返回的是组件render的虚拟节点树）
      const subTree = instance.render.call(proxy)
      // 基于返回的虚拟节点，对其进行patch比对（打补丁）
      patch(subTree, container, instance)
    
      // 此处可以确定所有的 element 都被 mount 了
      vnode.el = subTree.el
      
      instance.isMounted = true
    } else { // 如果instance.isMounted为 true，则说明已经挂载过了，此时需要更新
      console.log("update");
      const { proxy } = instance
      const subTree = instance.render.call(proxy) // 更新的时候再次调用 render 就能拿到全新的 虚拟节点了
    }
  })
}
```

添加参数

```ts
// src\runtime-core\component.ts
export function createComponentInstance(vnode, parent) {
  const component = {
    // ...
    isMounted: false,
    subTree: {},
  }
}
```



对比需要获取到之前的 `subTree`，那么在挂载时保存一下 `subTree`

```ts
function setupRenderEffect(instance, container) {
  effect(() => {
    if(!instance.isMounted) {
      const { proxy, vnode } = instance
      const subTree = instance.subTree = instance.render.call(proxy) // 在这里保存一下
      patch(subTree, container, instance)
    
      vnode.el = subTree.el

      instance.isMounted = true
    } else {
      console.log("update");
      const { proxy } = instance
      const subTree = instance.render.call(proxy) // 当前的subTree
      const prevSubTree = instance.subTree // 之前的 subTree
      instance.subTree = subTree // 更新 subTree
    }
  })
}
```



### 修改流程

- 修改 `patch` 传参

```ts
// 原先
function patch(vnode, container, parentComponent) {}
// 修改为
function patch(n1, n2, container, parentComponent) {}
```

n1 代表老的虚拟节点、n2 代表新的虚拟节点

- patch

```ts
function patch(n1, n2, container, parentComponent) {
  const { type, shapeFlag } = n2

  // Fragment -> 只渲染 children
  switch (type) {
    case Fragment:
      processFragment(n1, n2, container, parentComponent)
      break;
    case Text:
      processText(n1, n2, container)
      break;
    default: // 不是特殊的类型，继续走之前的逻辑
      if (shapeFlag & ShapeFlags.ELEMENT) {
        // 处理元素
        processElement(n1, n2, container, parentComponent)
      } else if (shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
        // 处理组件
        processComponent(n1, n2, container, parentComponent)
      }
      break;
  }
}
```

- 其余的调用全部修改

```ts
function processFragment(n1, n2, container, parentComponent) {
  mountChildren(n2, container, parentComponent)
}
function processText(n1, n2, container) {
  const { children } = n2
  const textNode = n2.el = document.createTextNode(children)
  container.append(textNode)
}
function processElement(n1, n2, container, parentComponent) {
  // element 类型也分为 mount 和 update，这里先实现mount
  mountElement(n2, container, parentComponent)

  // TODO 更新element
  // updateElement()
}
function processComponent(n1, n2, container, parentComponent) {
  // 挂载组件
  mountComponent(n2, container, parentComponent)
  // TODO 更新组件
}
function setupRenderEffect(instance, container) {
  effect(() => {
    if(!instance.isMounted) {
      // ...
      patch(null, subTree, container, instance)
      // ...
    } else {
      // ...

      // 更新
      patch(prevSubTree, subTree, container, instance)
    }
  })
}
function render(vnode, container) {
  patch(null, vnode, container, null);
}
function mountChildren(vnode, container, parentComponent) {
  vnode.children.forEach(v => patch(null, v, container, parentComponent))
}
```

接下来我们可以只关注于处理 `element`，因为在我们的测试代码中更新的部分是 `element` 类型

- 调整 `processElement` 的挂载更新逻辑

```ts
function processElement(n1, n2, container, parentComponent) {
  if(!n1) {
    mountElement(n2, container, parentComponent)
  } else {
    patchElement(n1, n2, container)
  }
}
```

```ts
function patchElement(n1, n2, container) {
  // 更新对比
  // props
  // childrens
}
```































