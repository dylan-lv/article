# 更新element的props

三个场景：

- 修改：之前的属性值和现在的属性值不一样了
- 删除：属性值变为 `null || undefined`
- 删除：属性值没有了



### 测试代码

```js
// example\update\App.js
import { h, ref } from "../../lib/guide-mini-vue.esm.js";

export const App = {
  name: "App",
  setup() {
    const count = ref(0);
    const onClick = () => {
      count.value++;
    };
    const props = ref({
      foo: "foo",
      bar: "bar",
    });
    const onChangePropsDemo1 = () => {
      props.value.foo = "new-foo";
    };
    const onChangePropsDemo2 = () => {
      props.value.foo = undefined;
    };
    const onChangePropsDemo3 = () => {
      props.value = {
        foo: "foo",
      };
    };
    return { count, onClick, onChangePropsDemo1, onChangePropsDemo2, onChangePropsDemo3, props };
  },
  render() {
    return h(
      "div",
      { id: "root", ...this.props },
      [
        h("div", {}, "count:" + this.count),
        h("button", { onClick: this.onClick }, "click"),
        h("button", { onClick: this.onChangePropsDemo1 }, "changeProps - 值改变了 - 修改"),
        h("button", { onClick: this.onChangePropsDemo2 }, "changeProps - 值变成了 undefined - 删除"),
        h("button", { onClick: this.onChangePropsDemo3 }, "changeProps - key 在新的里面没有了 - 删除"),
      ]
    );
  },
};
```

多了三个按钮，分别对应三个问题场景



### patchElement

```ts
// src\runtime-core\renderer.ts
function patchElement(n1, n2, container) {
  console.log('patchElement');
  // 更新对比
  const oldProps = n1.props || {}
  const newProps = n2.props || {}
  
  const el = n2.el = n1.el // n1.el是初始化得到的，赋值给n2.el可以保证下次调用时（更新）可以拿到正确的 el

  patchProps(el, oldProps, newProps)

  // childrens
}
```



### patchProps

```ts
// src\runtime-core\renderer.ts
function patchProps(el, oldProps, newProps) {
  for (const key in newProps) { // 循环新的Props
    const prevProp = oldProps[key]
    const nextProp = newProps[key]

    if(prevProp !== nextProp) {
      hostPatchProp(el, key, prevProp, nextProp)
    }
  }
}

function mountElement(vnode, container, parentComponent) {
  // ...
  for (const key in props) {
    const value = props[key]
    hostPatchProp(el, key, null, value)
  }
}
```



### 修改 patchProp

```ts
// src\runtime-dom\index.ts
function patchProp(el, key, prevVal, nextVal) {
  const isOn = (key: string) => /^on[A-Z]/.test(key);
  if (isOn(key)) {
    const event = key.slice(2).toLowerCase()
    el.addEventListener(event, nextVal)
  } else {
    el.setAttribute(key, nextVal)
  }
}
```



此时点击按钮 **changeProps - 值改变了 - 修改** 即可看到 `foo` 的值改变了

问题场景1就解决了。



### 删除

```ts
// src\runtime-dom\index.ts
function patchProp(el, key, prevVal, nextVal) {
  const isOn = (key: string) => /^on[A-Z]/.test(key);
  if (isOn(key)) {
    const event = key.slice(2).toLowerCase()
    el.addEventListener(event, nextVal)
  } else {
    if(nextVal === undefined || nextVal === null) { // 如果 nextVal 为 undefined或null就删除属性
      el.removeAttribute(key)
    } else { // 否则才是设置
      el.setAttribute(key, nextVal)
    }
    
  }
}
```



### 新的props中不存在，则删除

```ts
// src\runtime-core\renderer.ts
function patchProps(el, oldProps, newProps) {
  // 循环新的Props
  for (const key in newProps) { 
    const prevProp = oldProps[key]
    const nextProp = newProps[key]

    if(prevProp !== nextProp) {
      hostPatchProp(el, key, prevProp, nextProp)
    }
  }
  // 循环老的Props
  for (const key in oldProps) { 
    if(!(key in newProps)) {
      hostPatchProp(el, key, oldProps[key], null)
    }
  }
}
```



### 代码优化

新旧不相等，才做对比

```ts
// src\runtime-core\renderer.ts
function patchProps(el, oldProps, newProps) {
  if(oldProps !== newProps) { // 新旧不相等，才做对比
    // 循环新的Props
    for (const key in newProps) { 
      const prevProp = oldProps[key]
      const nextProp = newProps[key]

      if(prevProp !== nextProp) {
        hostPatchProp(el, key, prevProp, nextProp)
      }
    }
    // 循环老的Props
    for (const key in oldProps) { 
      if(!(key in newProps)) {
        hostPatchProp(el, key, oldProps[key], null)
      }
    }
  }
}
```

如果老的 `props` 不是一个空对象，才会进行对比

```ts
// src\runtime-core\renderer.ts
const EMPTY_OBJ = {}

function patchElement(n1, n2, container) {
  const oldProps = n1.props || EMPTY_OBJ
  const newProps = n2.props || EMPTY_OBJ
	// ...
}

function patchProps(el, oldProps, newProps) {
  if(oldProps !== newProps) {
    // ...
    if(oldProps !== EMPTY_OBJ){
      // 循环老的Props
      for (const key in oldProps) { 
        if(!(key in newProps)) {
          hostPatchProp(el, key, oldProps[key], null)
        }
      }
    }
  }
}
```







































