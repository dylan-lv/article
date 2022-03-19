# element主流程

将 `element` 类型显示到视图中。

我们可以先看一下 element 的 type 类型

```ts
// src\runtime-core\render.ts
function patch(vnode, container) {
  console.log(vnode.type);
  // 处理组件
  processComponent(vnode, container)
  // TODO 处理元素
}
```

执行命令：`yarn build --watch`，然后使用 `Open witn Live Serve` 打开 `index.html`

如下图所示：

<img src=".\assets\3.png" alt="3" />

可以看到，打印出了两条数据：

- `vnode` 是一个 `component` 类型，`vode` 的 `type` 就是一个 `Object`
- `vnode` 是一个 `element` 类型，`vode` 的 `type` 就是一个 `String`

基于上面的特点，我们就可以区分 `component` 类型和 `element` 类型了

```ts
function patch(vnode, container) {
  if(typeof vnode.type === "string") {
    processElement(vnode, container)
  } else if(isObject(vnode.type)) {
    processComponent(vnode, container)
  }
}
```



## App.js中的props

```js
export const App = {
  name: "App",
  render() {
    // 添加第二个参数：props
    return h("div", {
      id: "root",
      class: "red"
    }, "hi, mini-vue")
  },
  setup() {
    return {
      msg: 'mini-vue111'
    }
  }
}
```



## processElement

```ts
// src\runtime-core\render.ts
function processElement(vnode, container) {
  // element 类型也分为 mount 和 update，这里先实现mount
  mountElement(vnode, container)
}
```



## mountElement

```ts
function mountElement(vnode, container) {
  const el = document.createElement(vnode.type)
  // children可能是：string、array
  const { props, children }  = vnode
  el.textContent = children // 这里暂定它是string
  
  // props
  for (const key in props) {
    const val = props[key]
    el.setAttribute(key, val)
  }

  container.append(el)
}
```



此时打开浏览器，即可看到渲染在页面上的内容了~

<img src=".\assets\4.png" alt="4" />

## array类型

我们刚刚实现了 `children` 是 `string` 类型时的 `mountElement`

先来修改一下 `App.js`

```js
// App.js
export const App = {
  name: "App",
  render() {
    return h(
      "div", 
      {
        id: "root",
        // class: "red"
      },
      // string
      // "hi, mini-vue"
      // array
      [
        h("p", { class: "red" }, "hi"),
        h("p", { class: "blue" }, "mini-vue"),
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

此时可以看一下页面的内容：

<img src=".\assets\5.png" alt="5" />

显示了两个 `Object`，这是因为我们还没有出来 `array` 类型

### mountElement

```ts
function mountElement(vnode, container) {
  const el = document.createElement(vnode.type)
  // children可能是：string、array
  const { props, children }  = vnode

  if(typeof children === "string") {
    el.textContent = children
  }else if(Array.isArray(children)) {
    // children 中每个都是 vnode，需要继续调用 patch，来判断是element类型还是component类型，并对齐初始化
    children.forEach(v => patch(v, el))
  }

  for (const key in props) {
    const val = props[key]
    el.setAttribute(key, val)
  }

  container.append(el)
}
```

现在我们再来看一下页面的状态：

<img src=".\assets\6.png" alt="6" />

可以看到，此时 `array` 中的内容就全部渲染出来了



## 重构

```ts
function mountElement(vnode, container) {
  const el = document.createElement(vnode.type)
  const { props, children }  = vnode

  if(typeof children === "string") {
    el.textContent = children
  }else if(Array.isArray(children)) {
    // children.forEach(v => patch(v, el))
    mountChildren(vnode, el)
  }

  for (const key in props) {
    const val = props[key]
    el.setAttribute(key, val)
  }

  container.append(el)
}

function mountChildren(vnode, container) {
  vnode.children.forEach(v => {
    patch(v, container)
  })
}
```













