# 组件props功能



## props功能点

- 可以通过 `setup` 传入
- 在 `render` 中可以通过 `this` 来访问到其中的值
- `props` 是不可以被修改的（readonly）

```js
// Foo.js
export const Foo = {
  // setup 第一个参数接收传进来的 props
  setup(props) {
    console.log(props);
  },
  render() {
    // 通过 this.count，访问 props 里面的 count
    return h("div", {}, "foo: " + this.count);
  }
}
```



## 功能1：可以通过 `setup` 传入

### 测试代码

```js
// App.js
import { Foo } from './Foo.js';
export const App = {
  name: "App",
  render() {
    return h(
      "div", 
      {
        id: "root",
        class: "red",
        onClick() {
          console.log('my click');
        },
        onMousedown() {
          console.log('mouse downnnnn');
        }
      },
      [
        h("div", {}, "hi, " + this.msg),
        h(Foo, { count: 1 })
      ]
    )
  },
  setup() {
    return {
      msg: 'mini-vue111'
    }
  }
}
```

此时页面打印：

```js
hi, mini-vue111
foo: undefined
```



### setupComponent

```ts
// src\runtime-core\component.ts
export function setupComponent(instance){
  initProps(instance, instance.vnode.props)
  // TODO initSlots
  // 处理component调用setup之后的返回值（初始化一个有状态的component）
  setupStatefulComponent(instance)
}
```

### initProps

```ts
// src\runtime-core\componentProps.ts
export function initProps(instance, rawProps){
  // 现在先将没有处理过的 props 赋值给 instance 就可以了
  // 后续还会在这里处理 arrts 等逻辑

  instance.props = rawProps || {}
}
```

### 声明props

```ts
// src\runtime-core\component.ts
export function createComponentInstance(vnode){
  const component = {
    vnode,
    type: vnode.type,
    setupState: {},
    props: {}, // 声明 props 属性
  }

  return component
}
```



### setupStatefulComponent

此时，调用setup的时候，就可以把 `props` 作为参数1传入了

```ts
// src\runtime-core\component.ts
function setupStatefulComponent(instance) {
  const Component = instance.type;

  instance.proxy = new Proxy({ _: instance }, 
    PublicInstanceProxyHandlers
  )

  const { setup } = Component;
  if(setup) {
    const setupResult = setup(instance.props) // 传入 props

    handleSetupResult(instance, setupResult)
  }
}
```

此时，控制台已经打印出 `{count: 1}` 了，这是 `Foo.js` 中的 `setup` 打印的，说明这里已经可以接收到 `props` 了



## 功能2：在render中通过this访问

```ts
// src\runtime-core\componentPublicInstance.ts
export const PublicInstanceProxyHandlers = {
  // 在target这里获取_，改名为 instance
  get({ _: instance }, key) { // key 应该对应 msg
    const { setupState, props } = instance
    // if(key in setupState) {
    //   return setupState[key]
    // }
    // 判断当前的key是否在当前的对象上
    if(hasOwn(setupState, key)) {
      return setupState[key]
    } else if(hasOwn(props, key)) {
      return props[key]
    }

    const publicGetter = publicPropertiesMap[key]
    if(publicGetter) { // 目前它为 $el
      return publicGetter(instance)
    }
  }
}
```

```ts
// src\shared\index.ts
export const hasOwn = (value, key) => Object.prototype.hasOwnProperty.call(value, key);
```

此时页面上的内容，已经可以看到 count 值了：

```js
hi, mini-vue111
foo: 1
```



## 功能3：只读

可以使用之前实现的 `readonly` 来做处理。实际上 `props` 使用的是 `shallowReadonly`，只做了第一层的处理。

```ts
// src\runtime-core\component.ts
function setupStatefulComponent(instance) {
  const Component = instance.type;
  instance.proxy = new Proxy({ _: instance }, 
    PublicInstanceProxyHandlers
  )

  const { setup } = Component;
  if(setup) {
    // 给 props 包裹 shallowReadonly
    const setupResult = setup(shallowReadonly(instance.props))
    handleSetupResult(instance, setupResult)
  }
}
```

```ts
// src\reactivity\reactive.ts
function createActiveObject(target, baseHandlers) {
  if (!isObject(target)) { // 如果不是对象的情况
    console.warn(`target ${target}，必须是一个对象`);
    return target;
  }
  return new Proxy(target, baseHandlers);
}
```























