# shapeFlags

作用：描述 vnode（虚拟节点） 的一些类型。

- 在 `patch` 中，我们判断
  - `string`：判断是否是 `element` 类型
  - `object`：判断是否是 `stateful_component` 类型

```ts
function patch(vnode, container) {
  if(typeof vnode.type === "string") {
    processElement(vnode, container)
  } else if(isObject(vnode.type)) {
    processComponent(vnode, container)
  }
}
```

- 在 `mountElement` 中，我们判断 `children` 类型
  - `text_children`：`text` 类型
  - `array_children`：`array` 类型

```ts
function mountElement(vnode, container) {
  const el = vnode.el = document.createElement(vnode.type)
  // children可能是：string、array
  const { props, children }  = vnode
  if(typeof children === "string") {
    el.textContent = children
  }else if(Array.isArray(children)) {
    mountChildren(vnode, el)
  }
	...
}
```

我们把上面这些都称作当前节点的 `shapeFlags`

## 简单处理方式

我们自己写 `js`，可能会写成以下模式

```ts
const ShapeFlags = {
  element: 0,
  stateful_component: 0,
  text_children: 0,
  array_children: 0,
}
```

上述代码就是目前 vnode 可能存在的四种类型。

例如：当前的虚拟节点是 `stateful_component` 的，并且是 `array_children`，那么我们就只需要把它的 ShapeFlags 标记一下：

```ts
// 修改
ShapeFlags.stateful_component = 1;
ShapeFlags.array_children = 1;
```

判断值内容

```ts
// 查找
if(ShapeFlags.stateful_component) ...
if(ShapeFlags.array_children) ...
```



## 源码处理方式

上面的处理方式实际上已经可以解决问题了，但是 `key-value` 的方式不够高效，我们可以通过**位运算**的方式更高效的处理。

- 修改：使用 `|` 运算符
  - `0000 | 0001` -> `0001`
- 查找：使用 `&` 运算符
  - `0001 & 0001` -> `0001`

**接下来，修改现有逻辑**

```ts
// src\shared\ShapeFlags.ts
export const enum ShapeFlags {
  ELEMENT = 1, // 0001
  STATEFUL_COMPONENT = 1 << 1, // 0010
  TEXT_CHILDREN = 1 << 2, // 0100
  ARRAY_CHILDREN = 1 << 3 // 1000
}
```



### 虚拟节点赋值

- `vnode` 只有两种：`element` 类型、`component` 类型
- 处理 `children`

```ts
// src\runtime-core\vnode.ts
export function createVNode(type, props?, children?) {
  const vnode = {
    type,
    props,
    children,
    shapeFlag: getShapeFlag(type), // 基于 type 获取 shapeFlag
    el: null
  };
  
  // children
  if(typeof children === 'string') {
    vnode.shapeFlag |= ShapeFlags.TEXT_CHILDREN
  } else if(Array.isArray(children)) {
    vnode.shapeFlag |= ShapeFlags.ARRAY_CHILDREN
  }

  return vnode;
}

function getShapeFlag(type) {
  return typeof type === 'string' ? ShapeFlags.ELEMENT : ShapeFlags.STATEFUL_COMPONENT
}
```



### render中使用

修改 `patch` 方法

```ts
// src\runtime-core\render.ts
function patch(vnode, container) {
  const { shapeFlag } = vnode
  if(shapeFlag & ShapeFlags.ELEMENT) {
    processElement(vnode, container)
  } else if(shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
    processComponent(vnode, container)
  }

  // if(typeof vnode.type === "string") {
  //   processElement(vnode, container)
  // } else if(isObject(vnode.type)) {
  //   processComponent(vnode, container)
  // }
}
```

修改 `mountElement` 方法

```ts
// src\runtime-core\render.ts
function mountElement(vnode, container) {
  const el = vnode.el = document.createElement(vnode.type)
  // children可能是：string、array
  const { props, children, shapeFlag }  = vnode

  if(shapeFlag & ShapeFlags.TEXT_CHILDREN) {
    el.textContent = children
  } else if(shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    mountChildren(vnode, el)
  }

  // if(typeof children === "string") {
  //   el.textContent = children
  // }else if(Array.isArray(children)) {
  //   // children 中每个都是 vnode，需要继续调用 patch，来判断是element类型还是component类型，并对齐初始化
  //   // children.forEach(v => patch(v, el))
  //   mountChildren(vnode, el)
  // }

  // props
  for (const key in props) {
    const val = props[key]
    el.setAttribute(key, val)
  }

  container.append(el)
}
```





































