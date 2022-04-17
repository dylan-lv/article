# Vue3开发一个 v-loading 自定义指令

在 `vue2` 时期使用过 `element-ui` 组件库的同学应该都使用过 `v-loading`，指令性的方式让元素内部出现加载图。

这里，我们在 `vue3` 中开发一个 `v-loading` 自定义指令。

效果：在div元素上添加 `v-loading` 指令，即可出现加载图

```vue
<div class="table" v-loading="loading">...</div>
```

注：本文使用 `tsx` 方式实现。



## 一、编写 Loading 组件

### 1.1 Loading组件逻辑

在文件 `packages\Loading\src\index.tsx` 中

- 设置 `title` 属性和 `setTitle` 函数
  - 之后指令实现过程中可以拿到当前 `Loading` 组件实例，通过组件实例的 `setTitle` 方法，可以在指令的相应钩子中动态设置 `title`
- `render` 函数中
  - `hay-loading-mask`：loading 的蒙层
  - `hay-loading-spinner`：包含旋转的内层
  - i 标签（hay-icon-loading）：旋转图标
  - p 标签（hay-loading-text）：显示内容区域

```tsx
import { defineComponent, ref } from "vue";
import "./index.scss";

export default defineComponent({
  name: "HayLoading",
  setup() {
    const title = ref("");
    const setTitle = (newTitle: string) => {
      title.value = newTitle;
    };

    return { title, setTitle };
  },
  render() {
    return <div class="hay-loading-mask">
      <div class="hay-loading-spinner">
        <i class="hay-icon-loading"></i>
        {this.title && <p class="hay-loading-text">{this.title}</p>}
      </div>
    </div>;
  },
});
```



### 1.2 Loading组件样式

在 `packages\Loading\src\index.scss` 文件中

```scss
.hay-loading-mask {
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  background-color: rgba(255, 255, 255, 0.9);
  z-index: 2000;
  transition: opacity .3s;
  .hay-loading-spinner {
    position: absolute;
    width: 100%;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    i{
      display: block;
      width: 30px;
      height: 30px;
      border: 2px solid #409eff;
      border-top-color: transparent;
      border-radius: 100%;

      animation: circle infinite 0.75s linear;
    }
    .hay-loading-text{
      color: #409eff;
      margin: 3px 0;
      font-size: 14px;
    }
  }
}

.el-icon-loading {
  animation: circle infinite 0.75s linear;
}

// 转转转动画
@keyframes circle {
  0% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
```



## 二、自定义指令执行逻辑

我们需要考虑的问题：

1. `Loading` 组件依赖于父元素有定位，那么如果父元素没有定位的情况下，就应该给父元素动态的添加一个相对定位的样式
2. `name`：自定义指令的名称
3. 流程逻辑：`v-loading` 指令主要是将 `Loading` 组件生成的DOM动态插入到指令作用的DOM对象上
   1. `v-loading=true`：插入到DOM对象上
   2. `v-loading=false`：删除动态插入的 `el`。
4. 指令使用时开始，主要包含两个钩子函数：`mounted` 和 `updated` 。
   1. `mounted` 是在注册的时候运行，且只会运行一次；
   2. `updated` 会在 `v-loading=“xx”` 值改变的时候执行。
   3. 这些钩子会传入两个参数：`el`、`binding`
      1. `el`：指向指令所在的dom，如 `<div v-loading="true">` 那么 el 就是当前这个元素。
      2. `binding`：传入的一些值以及相关实例等

### 2.1 动态指令参数

官网：指令的参数可以是动态的。例如，在 `v-mydirective:[argument]="value"` 中，`argument` 参数可以根据组件实例数据进行更新！这使得自定义指令可以在应用中被灵活使用。

我这边因为是使用的 `tsx` 组件，`:[argument]` 的形式不能使用，暂未找到其他方式。所以本文的自定义指令考虑在 `value` 上做操作（bool、object 两种类型）



### 2.2 父元素定位

```ts
// packages\Loading\index.ts
const relativeCls = "hay-loading-parent--relative";
```

在 `Loading` 组件**挂载**的的时候，判断一下其挂载的**父元素**是否有 `"absolute", "fixed", "relative"` 这三种属性。如果没有的话就在父元素上添加 `hay-loading-parent--relative` 样式。

```scss
// packages\base.scss
.hay-loading-parent--relative{ position: relative }
```



### 2.3 定义 loading 指令

- `name`：自定义指令名称
- `mounted`：指令挂载时的钩子函数
- `updated`：指令更新时的钩子函数
- `export default loadingDirective`：导出指令配置，如果要在全局中使用，就需要在main.js中引入并注册

```ts
// packages\Loading\index.ts
import type { DirectiveBinding } from "vue";
const loadingDirective = {
  name: "loading",
  mounted(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
    // ...
  },
  updated(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
  	// ...
	}
}
export default loadingDirective;
```



### 2.4 mounted 钩子

- `el`：指向指令所在的 `dom`
- `binding`：一些参数值以及实例内容

```ts
// packages\Loading\index.ts
const loadingDirective = {
  mounted(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
    // ...
  },
}
```

- 判断 `v-loading` 值为 `true`，则动态插入到指令作用的节点下
  - 如果创建组件对应的 `dom` 不存在，先用这个 `Loading` 组件新建一个 `vue` 实例（app对象）
  - 然后再动态去挂载，就会产生一个实例，在实例中拿到它的 `DOM` 对象
- 拿到它的实例,挂载到动态创建的 `DOM` 上，`vue` 开发是支持多实例的，可以创建多个实例
  - 创建的元素没挂载到 `body` 上，实际也没有完成 `dom` 层的挂载
  - 目的是创建出来的实例的 `DOM` 对象要挂载到 `el` 上（指令所在的 `DOM` ）

```ts
const app = createApp(Loading);
const instance = app.mount(document.createElement("div"));
```



- 因为 instance 在 mounted 中只创建一次，但是之后会经常用到，我们将其保存在 `el` 对象上，这样操作在其他钩子中也可以获取到这个实例

```ts
el.instance = instance;
```



- `title` 值可能在 `binding.arg` 中，也可能在 `object` 类型的 `value` 中

```ts
const title = binding.arg;
// 如果参数不是空 执行实例中的方法
if (typeof title !== "undefined") {
  (instance as any).setTitle(title);
}
// 看看 binding.value 是否是object类型；如果是的话，再看看其中是否有 text 参数；有则对 title 进行赋值
if (typeof binding.value === "object" && binding.value !== null && binding.value.text) {
  (instance as any).setTitle(binding.value.text);
}
```



- 控制 `loading` 的插入和移除
- `binding.value` 代表指令传递的值

```ts
if (binding.value) {
  // 如果binding.value有值，并且是 bool 类型，则直接append
  if (typeof binding.value === "boolean") append(el);
  if (typeof binding.value === "object" && binding.value !== null) {
    // object类型：并且参数 value 为 true， 进行append操作
    if (binding.value.value) append(el);
  }
}
```



### 2.5 updated 钩子

当组件更新的时候执行，因为指令不是一成不变的。比如由 `v-loading=true` 变为 `v-loading=false` 就会执行

```ts
// 如果loading前后值不一致
if (binding.value !== binding.oldValue) {
  // bool的情况
  if (typeof binding.value === "boolean") {
    // 如果是true那么就插入否则删除
    binding.value ? append(el) : remove(el);
  }
  if (typeof binding.value === "object" && binding.value !== null) {
    binding.value.value ? append(el) : remove(el);
  }
}
```



### 2.6 元素挂载操作 append

- 根据 `Loading` 组件样式，是使用 `absolute`，而当 `el` 不是 `fixed、retaive、absolute` 的时候给其动态添加定位属性
- 因为 `Loading` 组件生成的实例 `instance` 已经赋值给 `el.instance` 属性上了，所以在这里可以直接通过 `el` 拿到
  - `el.instance.$el` 就是 `Loading` 组件的 `DOM` 对象

```ts
function append(el: HTMLElement & { instance: any }) {
  const style = getComputedStyle(el);
  if (["absolute", "fixed", "relative"].indexOf(style.position) === -1) {
    addClass(el, relativeCls);
  }
  el.appendChild(el.instance.$el);
}
```



### 2.7 元素移除操作

```ts
function remove(el: HTMLElement & { instance: any }) {
  removeClass(el, relativeCls);
  el.removeChild(el.instance.$el);
}
```



### 2.8 全部指令操作代码

`packages\Loading\index.ts` 文件中

```ts
import type { DirectiveBinding } from "vue";
import { createApp } from "vue";
import { addClass, removeClass } from "../shared";
import Loading from "./src";

const relativeCls = "hay-loading-parent--relative";

const loadingDirective = {
  name: "loading",
  mounted(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
    const app = createApp(Loading);
    const instance = app.mount(document.createElement("div"));
    
    el.instance = instance;
    const title = binding.arg;
    
    if (typeof title !== "undefined") {
      (instance as any).setTitle(title);
    }
    if (typeof binding.value === "object" && binding.value !== null && binding.value.text) {
      (instance as any).setTitle(binding.value.text);
    }

    if (binding.value) {
      if (typeof binding.value === "boolean") append(el);
      if (typeof binding.value === "object" && binding.value !== null) {
        if (binding.value.value) append(el);
      }
    }
  },
  updated(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
    const title = binding.arg;
    
    if (typeof title !== "undefined") {
      el.instance.setTitle(title);
    }
    
    if (binding.value !== binding.oldValue) {
      // bool的情况
      if (typeof binding.value === "boolean") {
        binding.value ? append(el) : remove(el);
      }
      // object的情况
      if (typeof binding.value === "object" && binding.value !== null) {
        binding.value.value ? append(el) : remove(el);
      }
    }
  },
};

function append(el: HTMLElement & { instance: any }) {
  const style = getComputedStyle(el);
  if (["absolute", "fixed", "relative"].indexOf(style.position) === -1) {
    addClass(el, relativeCls);
  }

  el.appendChild(el.instance.$el);
}

function remove(el: HTMLElement & { instance: any }) {
  removeClass(el, relativeCls);
  el.removeChild(el.instance.$el);
}

export default loadingDirective;
```



## 三、addClass和removeClass的实现

```ts
export function addClass (el, className) {
  // 如果当前元素样式列表中没有className
  if (!el.classList.contains(className)) {
    el.classList.add(className)
  }
}

export function removeClass (el, className) {
  el.classList.remove(className)
}
```



## 四、v-loading的注册与应用

### 4.1 全局注册

在main.js中：

```ts
import loadingDirective from 'packages\Loading\index.ts'
createApp(App).directive('loading', loadingDirective).mount('#app')
```

注册的时候使用**directive（‘指令名称’， 指令对象）**

因为叫**v-loading所以这里传入loading**, directive('loading', loadingDirective)全局注册后在这个app（对象）下就可以全局使用v-loading指令了



### 4.2 在组件中应用

```vue
<div class="recommend" v-loading:[loadingText]="loading">...</div>
```

`v-loading:[loadingText]` **这里的[]不是数组**！仅仅是vue3的一种语法。用于向binging.arg中传递动态参数

如果像组件中传递多个值的话：`loadingText: [ ... ] 或者 loadingText:  { a: .., b: ..}`

> 注：jsx无法使用，所以我们这里用的**传入Object类型的方式**



## 五、统一全局加载指令

在 `packages\index.ts` 文件中

```ts
/* 自定义指令 */
import Loading from "./Loading"; // v-loading指令
// 全部自定义指令
const directions: any[] = [
  HayLoading,
];
/**
 * 组件注册
 * @param {App} app Vue 对象
 * @returns {Void}
 */
const install = (app: App) => {
  // 自定义指令注册
  directions.forEach(direction => app.directive(direction.name, direction));
};
// 全部导出
export default {
  install,
  ...components,
};
```

```ts
// main.js
import HayUI from "packages\index.ts";
createApp(App)
  .use(HayUI)
  .mount("#app");
```















