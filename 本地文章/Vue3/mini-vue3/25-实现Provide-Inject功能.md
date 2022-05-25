# 实现Provide-Inject功能

祖先组件通过 `Provide` 提供数据（存），子孙组件通过 `Inject` 来获取相应的数据（取）

特点：跨层级，不管两者相隔多深，都可以传递数据



### 添加测试代码

```js
// example\apiInject\App.js
import { h, provide, inject } from '../../lib/guide-mini-vue.esm.js'

const Provider = {
  name: "Provider",
  setup() {
    // 通过 provide 提供两个数据
    provide("foo", "fooVal");
    provide("bar", "barVal");
  },
  render() {
    return h("div", {}, [
      h("p", {}, "Provider"),
      h(Consumer)
    ])
  }
}

const Consumer = {
  name: "Consumer",
  setup() {
    // 通过inject获取数据
    const foo = inject("foo");
    const bar = inject("bar");

    return { foo, bar }
  },
  render() {
    return h("div", {}, `Consumer: - ${this.foo} - ${this.bar}`)
  }
}
```



### 添加 Provide 和 Inject 函数

```ts
export function provide(key, value){
  // 存数据
}

export function inject(key){
  // 取数据
}
```



### 导出

```ts
// src\runtime-core\index.ts
export { provide, inject } from "./apiInject"
```



### 实现 provide

`provide` 是提供存数据的方法，那么应该存在哪儿呢？

我们可以考虑把数据存在当前组件的实例对象上（instance）

```ts
// src\runtime-core\component.ts

// 在实例上添加属性 provides
export function createComponentInstance(vnode) {
  const component = {
    // ...
    provides: {},
  }
	// ...
}
```

注意：`provide` 和 `inject` 必须在 `setup` 函数中使用，因为其在实现的时候使用到了 `getCurrentInstance` 方法来获取当前组件实例，这个实例是在 `setup` 的作用域下才能获取到有效的值。

```ts
// src\runtime-core\apiInject.ts
export function provide(key, value){
  // 存数据

  // 获取当前组件的实例对象
  const currentInstance: any = getCurrentInstance()
  if(currentInstance) {
    const { provides } = currentInstance
    provides[key] = value
  }
}
```



### 实现 inject

我们通过当前组件实例，获取到它的父组件 `parent`

先添加 `parent` 属性

```ts
// src\runtime-core\component.ts

// 通过参数传入 parent
export function createComponentInstance(vnode, parent) {
  const component = {
    // ...
    parent,
  }
	// ...
}
```

实现 inject 函数

```ts
export function inject(key){
  // 取数据
  const currentInstance: any = getCurrentInstance()

  if(currentInstance) {
    const parentProvides = currentInstance.parent.provides
    return parentProvides[key]
  }
}
```



### 解决parent 的问题

```ts
// src\runtime-core\renderer.ts
function mountComponent(initialVNode: any, container: any, parentComponent) {
  const instance = createComponentInstance(initialVNode, parentComponent)
  // ...
}
function processComponent(vnode, container, parentComponent) {
  // 挂载组件
  mountComponent(vnode, container, parentComponent)
  // TODO 更新组件
}
function patch(vnode, container, parentComponent) {
  // ...
  processFragment(vnode, container, parentComponent)
  processElement(vnode, container, parentComponent)
  processComponent(vnode, container, parentComponent)
}
function mountChildren(vnode, container, parentComponent) {
  vnode.children.forEach(v => patch(v, container, parentComponent))
}
function processFragment(vnode, container, parentComponent) {
  mountChildren(vnode, container, parentComponent)
}
function mountElement(vnode, container, parentComponent) {
  // ...
  mountChildren(vnode, el, parentComponent)
}

function processElement(vnode, container, parentComponent) {
  mountElement(vnode, container, parentComponent)
}
```

在 `setupRenderEffect` 时 `patch` 过程中的 `parentComponent` 就是当前实例 `instance`

这里的虚拟节点 `subTree` 是通过当前实例 `instance` 的 `render` 函数得到的，相当于是当前实例的子节点

```ts
function setupRenderEffect(instance, container) {
  const { proxy, vnode } = instance
  const subTree = instance.render.call(proxy)
  patch(subTree, container, instance)
  vnode.el = subTree.el
}
```

第一次渲染时，父组件是为空的

```ts
export function render(vnode, container) {
  patch(vnode, container, null);
}
```

此时页面上的内容就可以渲染出来了



## 调整功能

此时我们虽然可以将数据在页面中渲染出来了，但是我们只有两层：父组件和子组件。

现在我们让事情变得更复杂一些

```js
// 
const Provider = {
  name: "Provider",
  setup() {
    provide("foo", "fooVal");
    provide("bar", "barVal");
  },
  render() {
    return h("div", {}, [
      h("p", {}, "Provider"),
      h(ProviderTwo)
    ])
  }
}

const ProviderTwo = {
  name: "ProviderTwo",
  setup() {
  },
  render() {
    return h("div", {}, [
      h("p", {}, `ProviderTwo`),
      h(Consumer)
    ])
  }
}

const Consumer = {
  name: "Consumer",
  setup() {
    const foo = inject("foo");
    const bar = inject("bar");

    return { foo, bar }
  },
  render() {
    return h("div", {}, `Consumer: - ${this.foo} - ${this.bar}`)
  }
}
```

现在页面上显示 `undefined`，说明有问题了

当前的层级：Provider -> ProviderTwo -> Consumer

我们之前初始化的时候，每个组件都有自己的 `provides`，现在 `Consumer` 向上找的时候，找到 `ProviderTwo`，它是一个空对象，所以会返回 `undefined`

需求：我们的子组件寻找 `foo` 的时候，会先到其**父组件**中的 `provides` 查找，如果没找到，继续向上（父组件的父组件）中查找，直到找到为止

解决方法：将 `ProvidesTwo` 组件的原型指向其父级的 `provides` 就可以了，而且这样做的话，`ProvidesTwo`组件也会有自己的 `provides` 值

```ts
// src\runtime-core\component.ts
export function createComponentInstance(vnode, parent) {
  const component = {
    // ...
    provides: parent ? parent.provides : {}, // 当前组件的 provides 值初始化为其父组件的 provides
  }
}
```

```ts
// src\runtime-core\apiInject.ts
export function provide(key, value){
  // 存数据

  // 获取当前组件的实例对象
  const currentInstance: any = getCurrentInstance()
  if(currentInstance) {
    let { provides } = currentInstance
    const parentProvides = currentInstance.parent.provides // 父组件的 provides
    
    if(provides === parentProvides) { // 如果当前provides和其父组件相同，说明其还没有被赋值过
      // 将当前实例 provides 的原型指向父级 provides
      // 并且只有初始化的时候才可以执行
      provides = currentInstance.provides = Object.create(parentProvides)
    }

    provides[key] = value
  }
}
```



此时就解决了 `provides` 的问题



### inject默认值

```js
// example\apiInject\App.js
const Consumer = {
  name: "Consumer",
  setup() {
    const foo = inject("foo");
    const bar = inject("bar");
    const baz = inject("baz", "bazDefault"); // 取不到值的话，默认值为bazDefault

    return { foo, bar, baz, bazfunc }
  },
  render() {
    return h("div", {}, `Consumer: - ${this.foo} - ${this.bar} - ${this.baz}`)
  }
}
```



```ts
// src\runtime-core\apiInject.ts
export function inject(key, defaultValue){
  const currentInstance: any = getCurrentInstance()

  if(currentInstance) {
    const parentProvides = currentInstance.parent.provides
    if(key in parentProvides) { // 如果key存在于 Provides 中，则直接返回
      return parentProvides[key]
    } else if(defaultValue) { // 否则如果用户传入的默认值，则返回默认值
      return defaultValue
    }
  }
}
```



### inject默认值为函数

```ts
// src\runtime-core\apiInject.ts
export function inject(key, defaultValue){
  const currentInstance: any = getCurrentInstance()

  if(currentInstance) {
    const parentProvides = currentInstance.parent.provides
    if(key in parentProvides) {
      return parentProvides[key]
    } else if(defaultValue) {
      if(typeof defaultValue === "function") { // 判断如果默认值是函数，则返回其调用值
        return defaultValue()
      }
      return defaultValue
    }
  }
}
```



































