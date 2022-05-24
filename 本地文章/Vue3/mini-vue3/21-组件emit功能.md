# 组件emit功能

在 `vue3` 中使用 `emit`，是在 `setup` 的第二个对象参数中拿到的

## 测试

```js
// example/componentEmit/App.js
import { h } from '../../lib/guide-mini-vue.esm.js'
import { Foo } from './Foo.js';

export const App = {
  name: "App",
  render() {
    return h("div", {}, [
      h("div", {}, "APP"),
      h(Foo, {})
    ])
  },
  setup() {
    return {}
  }
}
```



```js
// example/componentEmit/Foo.js
import { h } from "../../lib/guide-mini-vue.esm.js";
export const Foo = {
  setup(props) {
    const emitAdd = () => {
      console.log("emit add");
      // emit("add", 1, 2)
      // emit("add-foo", 1, 2)
    }
    return { emitAdd }
  },
  render() {
    const btn = h("button", {
      onClick: this.emitAdd
    }, "emitAdd")
    const foo = h("p", {}, "foo")
    return h("div", {}, [foo, btn]);
  }
}
```

此时，点击按钮的事件是没有任何问题的

接下来，加入 emit 使用：

```js
// Foo.js
export const Foo = {
  setup(props, { emit }) {
    const emitAdd = () => {
      console.log("emit add");
      emit("add") // 希望上层组件接收到 add
    }
    return { emitAdd }
  },
  render() {
    const btn = h("button", {
      onClick: this.emitAdd
    }, "emitAdd")
    const foo = h("p", {}, "foo")
    return h("div", {}, [foo, btn]);
  }
}
// App.js
export const App = {
  name: "App",
  render() {
    return h("div", {},
      [
        h("div", {}, "App"),
        h(Foo, {
          onAdd() { // 在Foo的第二个参数，使用 onAdd 接收 emit
            console.log("on-addddd");
          }
        })
      ]
    )
  },
  setup() {
    return {}
  }
}
```



## 实现

- `setup` 的第二个参数是个对象，对象中有 `emit`
- 调用 `emit` 的时候，可以传入一个事件名
- 有了事件名后，在当前组件找有没有事件函数：on+事件名（首字母大写）。然后调用就行了



### setup 中 ，第二个参数 emit

```ts
// src\runtime-core\component.ts
export function createComponentInstance(vnode){
  const component = {
    vnode,
    type: vnode.type,
    setupState: {},
    props: {},
    emit: () => {}
  }
  // 使用 bind 初始化 emit，用户使用的时候只需要传事件名，但是真实的 emit 实现中也可以拿到 instance 了
  component.emit = emit.bind(null, component) as any;

  return component
}
function setupStatefulComponent(instance) {
  const Component = instance.type;

  instance.proxy = new Proxy({ _: instance }, 
    PublicInstanceProxyHandlers
  )

  const { setup } = Component;
  if(setup) {
    // 挂载 emit
    const setupResult = setup(shallowReadonly(instance.props), {
      emit: instance.emit
    })

    handleSetupResult(instance, setupResult)
  }
}
```

```ts
// src\runtime-core\componentEmit.ts
export function emit(instance, event){
  const { props } = instance

  const handler = props["onAdd"]
  handler && handler()
}
```

此时，点击页面按钮，可以看到 `emit` 已经成功触发了

### 通用化

- ...args：处理用户传参

```ts
export function emit(instance, event, ...args){
  const { props } = instance

  // 首字母大写
  const capitalize = (str: string) => str.charAt(0).toUpperCase() + str.slice(1);

  const toHandlerKey = (str: string) => str ? "on" + capitalize(str) : ""

  const handlerName = toHandlerKey(event)
  // const handler = props["on" + capitalize(event)]
  const handler = props[handlerName]
  handler && handler(...args)
}
```



### 支持烤串的命名方式：`add-foo`

在 `App.js` 中接收的时候是：`onAddFoo`

将 `add-foo` 变为驼峰命名法

```ts
export function emit(instance, event, ...args){
  const { props } = instance
  // 烤串转驼峰
  const camelize = (str: string) => str.replace(/-(\w)/g, (_, c: string) => {
    // _：匹配的内容 -(\w)； c：(\w) 的内容
    return c ? c.toUpperCase() : ""
  })

  // 首字母大写
  const capitalize = (str: string) => str.charAt(0).toUpperCase() + str.slice(1);

  const toHandlerKey = (str: string) => str ? "on" + capitalize(str) : ""

  const handlerName = toHandlerKey(camelize(event)) // 加入烤串转驼峰方法
  const handler = props[handlerName]
  handler && handler(...args)
}
```



### 抽离功能函数

```ts
// src\shared\index.ts
// 烤串转驼峰
export const camelize = (str: string) => str.replace(/-(\w)/g, (_, c: string) => {
  // _：匹配的内容 -(\w)； c：(\w) 的内容
  return c ? c.toUpperCase() : ""
})

// 首字母大写
export const capitalize = (str: string) => str.charAt(0).toUpperCase() + str.slice(1);

export const toHandlerKey = (str: string) => str ? "on" + capitalize(str) : ""
```

```ts
// src\runtime-core\componentEmit.ts
import { camelize, toHandlerKey } from "../shared/index"

export function emit(instance, event, ...args){
  const { props } = instance
  
  const handlerName = toHandlerKey(camelize(event))
  const handler = props[handlerName]
  handler && handler(...args)
}
```

















