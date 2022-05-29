# 更新element的children



children 支持 `text` 和 `array` 两种类型的数据，两两对比一共有四种情况

- text => text
- array => array
- text => array
- array => text



## 测试代码

```js
// example\patchChildren\App.js
import ArrayToText from "./ArrayToText.js";
import TextToText from "./TextToText.js";
import TextToArray from "./TextToArray.js";
import ArrayToArray from "./ArrayToArray.js";

export default {
  name: "App",
  setup() {},

  render() {
    return h("div", { tId: 1 }, [
      h("p", {}, "主页"),
      // 老的是 array 新的是 text
      h(ArrayToText),
      // 老的是 text 新的是 text
      // h(TextToText),
      // 老的是 text 新的是 array
      // h(TextToArray)
      // 老的是 array 新的是 array
      // h(ArrayToArray)
    ]);
  },
};
```



## 数组 -> 文本

```js
// example\patchChildren\ArrayToText.js
// 老的是 array
// 新的是 text
import { ref, h } from "../../lib/guide-mini-vue.esm.js";
const nextChildren = "newChildren";
const prevChildren = [h("div", {}, "A"), h("div", {}, "B")];

export default {
  name: "ArrayToText",
  setup() {
    const isChange = ref(false);
    window.isChange = isChange;

    return {
      isChange,
    };
  },
  render() {
    const self = this;

    return self.isChange === true
      ? h("div", {}, nextChildren)
      : h("div", {}, prevChildren);
  },
};
```

调试的时候直接在页面的控制台输入 `isChange.value = true` 就会触发更新了



### patchElement

```ts
// src\runtime-core\renderer.ts
function patchElement(n1, n2, container) {
  // ...

  patchChildren(n1, n2)
}
```



### patchChildren

```ts
// src\runtime-core\renderer.ts
function patchChildren(n1, n2) {
  const prevShapeFlag = n1.shapeFlag
  const shapeFlag = n2.shapeFlag
  
  if(shapeFlag & ShapeFlags.TEXT_CHILDREN) { // 新节点是 text
    if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) { // 老节点是 array
      // 1.把老的 children 清空
      unmountChildren(n1.children)
      // 2.设置 text
    }
  }
}
```

### 把老的 children 清空 ：unmountChildren

```ts
// src\runtime-core\renderer.ts
const {
  createElement: hostCreateElement,
  patchProp: hostPatchProp,
  insert: hostInsert,
  remove: hostRemove
} = options
function unmountChildren(children) {
  for(let i=0; i<children.length; i++) {
    const el = children[i].el
    hostRemove(el)
  }
}
```

### hostRemove

```ts
// src\runtime-dom\index.ts
function remove(child) {
  const parent = child.parentNode
  if(parent) {
    parent.removeChild(child)
  }
}
const renderer: any = createRenderer({
  createElement,
  patchProp,
  insert,
  remove
})
```

### 设置新的 text

```ts
// src\runtime-core\renderer.ts
function patchElement(n1, n2, container) {
  const oldProps = n1.props || EMPTY_OBJ
  const newProps = n2.props || EMPTY_OBJ

  const el = n2.el = n1.el

  patchProps(el, oldProps, newProps)

  patchChildren(n1, n2, el) // 传入上一级的 el
}
function patchChildren(n1, n2, container) {
  const prevShapeFlag = n1.shapeFlag
  const shapeFlag = n2.shapeFlag
  const c2 = n2.children
  
  if(shapeFlag & ShapeFlags.TEXT_CHILDREN) { // 新节点是 text
    if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) { // 老节点是 array
      // 1.把老的 children 清空
      unmountChildren(n1.children)
      // 2.设置 text
      hostSetElementText(container, c2)
    }
  }
}
```

### hostSetElementText

```ts
// src\runtime-dom\index.ts
function setElementText(el, text) {
  el.textContent = text
}

const renderer: any = createRenderer({
  createElement,
  patchProp,
  insert,
  remove,
  setElementText
})
```

现在再控制台输入 `isChange.value = true` 就可以看到更新效果了



## 文本 -> 文本

```js
// example\patchChildren\TextToText.js
// 新的是 text
// 老的是 text
import { ref, h } from "../../lib/guide-mini-vue.esm.js";

const prevChildren = "oldChild";
const nextChildren = "newChild";

export default {
  name: "TextToText",
  setup() {
    const isChange = ref(false);
    window.isChange = isChange;

    return {
      isChange,
    };
  },
  render() {
    const self = this;

    return self.isChange === true
      ? h("div", {}, nextChildren)
      : h("div", {}, prevChildren);
  },
};
```

### patchChildren

```ts
// src\runtime-core\renderer.ts
function patchChildren(n1, n2, container) {
  const prevShapeFlag = n1.shapeFlag
  const shapeFlag = n2.shapeFlag
  const c1 = n1.children
  const c2 = n2.children
  
  if(shapeFlag & ShapeFlags.TEXT_CHILDREN) { // 新节点是 text
    if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) { // 老节点是 array
      // 1.把老的 children 清空
      unmountChildren(n1.children)
      // 2.设置 text
      hostSetElementText(container, c2)
    } else { // 老节点是 text
      if(c1 !== c2) {
        hostSetElementText(container, c2)
      }
    }
  }
}
```

### 重构

 新节点是text，如果老节点是 arr 则需要清空，如果老节点是 text 则直接设置

```ts
// src\runtime-core\renderer.ts
function patchChildren(n1, n2, container) {
  const prevShapeFlag = n1.shapeFlag
  const shapeFlag = n2.shapeFlag
  const c1 = n1.children
  const c2 = n2.children

  if(shapeFlag & ShapeFlags.TEXT_CHILDREN) { // 新节点是 text
    // 新节点是text，如果老节点是 arr 则需要清空，如果老节点是 text 则直接设置
    if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) { // 老节点是 array
      // 1.把老的 children 清空
      unmountChildren(n1.children)
    }
    if(c1 !== c2) {
      hostSetElementText(container, c2)
    }
  }
}
```



## 文本 -> 数组

```ts
// src\runtime-core\renderer.ts
function patchChildren(n1, n2, container, parentComponent) {
  const prevShapeFlag = n1.shapeFlag
  const shapeFlag = n2.shapeFlag
  const c1 = n1.children
  const c2 = n2.children

  if(shapeFlag & ShapeFlags.TEXT_CHILDREN) { // 新节点是 text
    if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) { // 老节点是 array
      unmountChildren(n1.children)
    }
    if(c1 !== c2) {
      hostSetElementText(container, c2)
    }
  } else { // 新节点是 array
    if(prevShapeFlag & ShapeFlags.TEXT_CHILDREN) { // 老节点是text
      hostSetElementText(container, "") // 清空 text
      mountChildren(c2, container, parentComponent) // 挂载children
    }
  }
}
```

















































