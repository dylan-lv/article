# 实现getCurrentInstance

通过官网的 `demo` 我们知道：可以通过 `getCurrentInstance` 来获取当前的实例对象

这个方法是必须在 `setup` 中使用的

```ts
import { getCurrentInstance } from "vue"
const MyConponent = {
  setup() {
    const interalInstance = getCurrentInstance()
    interalInstance.appContext.config.globalProperties
  }
}
```

## 测试用例

```js
// example\currentInstance\App.js
import { h, getCurrentInstance } from '../../lib/guide-mini-vue.esm.js'
import { Foo } from './Foo.js';

export const App = {
  name: "App",
  setup() {
    const instance = getCurrentInstance();
    console.log("App:", instance);
  },
  render() {
    return h("div", {}, [
      h("p", {}, "currentInstance demo"),
      h(Foo)
    ])
  },
}
```

```js
// example\currentInstance\Foo.js
import { h, getCurrentInstance } from "../../lib/guide-mini-vue.esm.js";

export const Foo = {
  name: "Foo",
  setup() {
    const instance = getCurrentInstance();
    console.log("Foo:", instance);
    return {}
  },
  render() {
    return h("div", {}, "foo")
  }
}
```



## `getCurrentInstance` 函数

放在 `component.ts` 文件中，因为都是处理组件的

```ts
// src\runtime-core\component.ts

// 借助全局变量来获取组件实例
let currentInstance = null
export function getCurrentInstance(){
  // 返回组件实例
  return currentInstance
}
```

调用 `setup` 的时候对其进行赋值

```ts
function setupStatefulComponent(instance: any) {
  const Component = instance.type

  instance.proxy = new Proxy({ _: instance }, PublicInstanceProxyHandlers)

  const { setup } = Component
  if(setup) {
    currentInstance = instance // 在setup时获取当前组件实例
    const setupResult = setup(shallowReadonly(instance.props), {
      emit: instance.emit 
    })
		currentInstance = null // 清空
    handleSetupResult(instance, setupResult)
  }
}
```



## 导出

```ts
// src\runtime-core\index.ts
export { getCurrentInstance } from "./component"
```



## 重构

`currentInstance` 直接赋值给全局变量，不是特别好的一种实现方式。更好的方式是将其封装成一个函数。

后续更改的时候，直接调用 `setCurrentInstance` 函数即可

```ts
export function setCurrentInstance(instance){
  currentInstance = instance
}
function setupStatefulComponent(instance: any) {
  const Component = instance.type
  instance.proxy = new Proxy({ _: instance }, PublicInstanceProxyHandlers)

  const { setup } = Component
  if(setup) {
    // currentInstance = instance
    setCurrentInstance(instance)
    const setupResult = setup(shallowReadonly(instance.props), {
      emit: instance.emit
    })
		// currentInstance = null // 清空
    setCurrentInstance(null)

    handleSetupResult(instance, setupResult)
  }
}
```

好处在于，当我们后续想要跟踪 `currentInstance` 被谁赋值的时候，只需要在 `setCurrentInstance`函数中断点即可，这个函数也起到了中间层的作用。



















