# Vue3开发一个 v-loading 自定义指令

要实现在div元素上添加v-loading指令，即可出现加载图

```vue
<div class="recommend" v-loading="loading">...</div>
```



## 一、loading组件

```tsx
// packages\Loading\src\index.tsx
import { defineComponent, ref } from "vue";
import "./index.scss";

export default defineComponent({
  name: "HayLoading",
  setup() {
    const title = ref("");
    const setTitle = (str: string) => {
      title.value = str;
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

```scss
// packages\Loading\src\index.scss
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

```ts
// packages\Loading\index.ts
import type { DirectiveBinding } from "vue";
import { createApp } from "vue";
import { addClass, removeClass } from "../shared";
import Loading from "./src";

// 该样式添加在了base.scss中，.g-relative{position: relative}
const relativeCls = "g-relative";

// 定义 loading 指令
const loadingDirective = {
  name: "loading",
  // 主要写一些钩子函数 在钩子中去实现逻辑
  /*
    指令主要是将 loading 组件生成的DOM动态插入到指令作用的DOM对象上（v-loading=true），
    如果v-loading=false那么就删除动态插入的
   */
  // 指令挂载时的钩子函数
  mounted(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
    /*
      el指向指令所在的dom 如 <div v-loading="true" id="box"> 那么el就是#box
      binding.value就是代表的true
    */
    // 判断v-loading值为true动态插入到指令作用的节点下
    /*
      如果创建组件对应的dom不存在，先用这个loading组件新建一个vue实例（app对象），
      然后再动态去挂载，就会产生一个实例，在实例中拿到它的DOM对象
    */
    const app = createApp(Loading);
    // 拿到它的实例,挂载到动态创建的DOM上，vue开发是支持多实例的，可以创建多个实例
    /*
      创建的元素没挂载到BODY上，实际也没有完成dom层的挂载，
      目的是创建出来的实例的DOM对象要挂载到el上（指令所在的DOM）
    */
    const instance = app.mount(document.createElement("div"));
    /*
      因为 instance 在 mounted 中只创建一次，但是之后会经常用到，要保留起来
      如果要在其他的钩子函数也要访问它的话就存在参数的el对象上
      这样操作在其他钩子中也可以获取到这个实例
    */
    el.instance = instance;
    // 通过binding.arg拿到动态参数,如果组件中有多个参数可以考虑传进来的是一个数组
    const title = binding.arg;
    // 如果参数不是空 执行实例中的方法
    if (typeof title !== "undefined") {
      (instance as any).setTitle(title);
    }
    // 看看 binding.value 是否是object类型；如果是的话，再看看其中是否有 text 参数；有则对 title 进行赋值
    if (typeof binding.value === "object" && binding.value !== null && binding.value.text) {
      (instance as any).setTitle(binding.value.text);
    }

    // binding.value就是代表指令传递的值
    if (binding.value) {
      // 如果binding.value有值，并且是 bool 类型，则直接append
      if (typeof binding.value === "boolean") append(el);
      if (typeof binding.value === "object" && binding.value !== null) {
        // object类型：并且参数 value 为 true， 进行append操作
        if (binding.value.value) append(el);
      }
    }
  },
  // 当组件更新的时候执行，因为指令不是一成不变的
  // 比如由v-loading=true变为v-loading=false 就会执行
  updated(el: HTMLElement & { instance: any }, binding: DirectiveBinding) {
    // 通过binding.arg拿到动态参数
    const title = binding.arg;
    // 如果参数不是空 执行实例中的方法
    if (typeof title !== "undefined") {
      el.instance.setTitle(title);
    }
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
  },
};

// 元素挂载的操作
function append(el: HTMLElement & { instance: any }) {
  // 根据loading组件样式，是使用absolute，而当el不是fixed或retaive时候给其动态添加定位属性
  const style = getComputedStyle(el);
  // 判断el的样式中有无定位，===-1就是没有 希望v-loading不受样式限制
  if (["absolute", "fixed", "relative"].indexOf(style.position) === -1) {
    addClass(el, relativeCls);
  }

  // 因为loading组件生成的实例instance已经赋值给el.instance属性上了，所以在这里可以直接通过el拿到
  // el.instance.$el就是loading组件的DOM对象
  el.appendChild(el.instance.$el);
}

function remove(el: HTMLElement & { instance: any }) {
  removeClass(el, relativeCls);
  el.removeChild(el.instance.$el);
}

// 如果要在全局中使用，就在main.js中引入并注册
export default loadingDirective;
```



### 2.1 流程逻辑

v-loading指令主要是将loading组件生成的DOM动态插入到指令作用的DOM对象上（v-loading=true），如果v-loading=false那么就删除动态插入的。

指令使用时开始，主要包含两个钩子函数：mounted和updated，mounted是在注册的时候运行，且只会运行一次；updated会在v-loading=“xx”值改变的时候执行。

```ts
mounted(el, binding) {...}
updated(el, binding) {...}
```

- el ：指向指令所在的dom，如 `<div v-loading="true" id="box">` 那么 el 就是 `document.querySelector("#box")` 这个元素。
- binding：
  - arg："拼命加载中..."
  - dir
    - mounted：f mounted(el, binding)
    - updated： f updated(el, binding)
  - instance: Proxy {...}
  - modifiers: {}
  - oldValue: undefined
  - value: true

其中我们在代码中用到的有3处：

- **arg：**通过binding.arg拿到动态参数,如果组件中有多个参数可以考虑传进来的是一个数组，动态参数就是在使用时：

```vue
<div class="recommend" v-loading:[loadingText]="loading">
// or
<div class="recommend" v-loading.loadingggggg="loading">
```

- **value：**就是v-loading="loading" 中的loading的值
- **oldValue**：v-loading之前的值，如果不存在就是undefined

我们这里 value 可能是 bool 或 object 类型



### 2.2 保存组件实例

使用loading组件再创建一个vue实例，然后将实例保存到mounted(el)的el中（这个实例只会在mounted中创建一次），这样就可以在其他的钩子函数中也可以获取到这个实例了。

```ts
const app = createApp(Loading)
const instance = app.mount(document.createElement('div'))
el.instance = instance
```



### 2.3 考虑更广泛的应用场景

通过看样式代码我们发现，loading组件依赖于父元素有定位，那么如果父元素没有定位的情况下，是不是应该给父元素动态的添加一个相对定位的样式？所以在append的函数中：

```ts
const style = getComputedStyle(el)
// 判断el的样式中有无定位，===-1就是没有 
// 没有就添加一个样式 addClass实现请见下方
if (['absolute', 'fixed', 'relative'].indexOf(style.position) === -1) {
    addClass(el, relativeCls)
}
```



### 2.4 执行组件实例中的方法

因为loading组件中暴漏了一个setTitle的方法，而el.instance中保存了这个组件的实例，于是乎

```ts
el.instance.setTitle(title) // 执行组件中的方法
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

```ts
// packages\index.ts
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
import HayUI from "hay-ui";
createApp(App)
  .use(HayUI)
  .mount("#app");
```















