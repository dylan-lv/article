# element-plus源码解读之Loading组件



## 一、功能需求

- 组件功能
  - 显示/隐藏Loading
  - 渐变Transition
  - loading 图标（动画）
  - 文字信息
  - 全屏Loading

- 指令模式
- 服务模式



## 二、注入app

作为一个指令，`v-loading` 会通过 `app.directive` 的方式来注入 `app`

```ts
// packages\components\Loading\index.ts
import { Loading } from './src/service'
import { vLoading } from './src/directive'
import "./src/index.scss";
export const ElLoading = {
  install(app: App) {
    app.directive('loading', vLoading)
    app.config.globalProperties.$loading = Loading
  },
  directive: vLoading,
  service: Loading,
}
export default ElLoading
export { vLoading, vLoading as ElLoadingDirective, Loading as ElLoadingService }
```

- `install`：`Vue` 项目中调用 `use` 就会执行 `install` 中的代码
  - `app.directive('loading', vLoading)`：指令的方式
  - `app.config.globalProperties.$loading = Loading`：**服务**的方式
- `directive、service`：将 `loading` 组件的指令和服务两种方式挂载在 `ElLoading` 上，并一起导出



## 三、v-loading` 指令

### 指令挂载钩子：`mounted`

```ts
// packages\components\Loading\src\directive.ts
export const vLoading: Directive<ElementLoading, LoadingBinding> = {
  mounted(el, binding) {
    if (binding.value) {
      createInstance(el, binding)
    }
  },
  ...
}
```

指令挂载（mounted）期间，如果 `binding.value` 存在（`v-loading` 初始值不为 `false`），则执行 `createInstance(el, binding)`（创建实例）



### createInstance

```ts
// packages\components\Loading\src\directive.ts
import { Loading } from './service'
const INSTANCE_KEY = Symbol("ElLoading");
const createInstance = (el: ElementLoading, binding: DirectiveBinding<LoadingBinding>) => {
  const vm = binding.instance
  // v-loading的值可以传对象，如果是对象的话，这里通过key来获取相应的value
  const getBindingProp = <K extends keyof LoadingOptions>(
    key: K
  ): LoadingOptions[K] => isObject(binding.value) ? binding.value[key] : undefined
  // 尝试获取vm中对应的[key]值，如果没拿到则使用传入的[key]
  const resolveExpression = (key: any) => {
    const data = (isString(key) && vm?.[key]) || key
    if (data) return ref(data) // 如果data有值则包裹ref
    else return data
  }
  const getProp = <K extends keyof LoadingOptions>(name: K) =>
    resolveExpression(
    	// 先尝试从传入的参数中获取
      getBindingProp(name) 
    	// 再尝试从 element-loading-xxx 中获取
      || el.getAttribute(`element-loading-${hyphenate(name)}`) // hyphenate：将驼峰转为中划线
    )
  // 全屏信息：先从Prop中拿，如果没拿到，则从指令修饰符（binding.modifiers）中拿
  const fullscreen = getBindingProp('fullscreen') ?? binding.modifiers.fullscreen
  
  const options: LoadingOptions = {
    text: getProp('text'),
    svg: getProp('svg'),
    svgViewBox: getProp('svgViewBox'),
    spinner: getProp('spinner'),
    background: getProp('background'),
    customClass: getProp('customClass'),
    fullscreen,
    // 将指令挂载的当前元素赋值给 `target`；如果fullscreen不为undefined，则target为undefined
    target: getBindingProp('target') ?? (fullscreen ? undefined : el), 
    body: getBindingProp('body') ?? binding.modifiers.body,
    lock: getBindingProp('lock') ?? binding.modifiers.lock,
  };

  el[INSTANCE_KEY] = {
    options,
    instance: Loading(options),
  };
};
```

这里做的事情：

- 初始化 `options`
- `Loading(options)`：初始化 `Loading` 组件
- 将 `options` 和**组件实例**挂载到 `el` 上



### Loading(options)

我们进到 `service.ts` 文件中，来看看 `Loading` 函数是如何做的实例化操作

```ts
// packages\components\Loading\src\service.ts
let fullscreenInstance: LoadingInstance | undefined = undefined
export const Loading = function(options: LoadingOptions = {}): LoadingInstance {
  if (!isClient) return undefined as any; // 只能在客户端渲染
  const resolved = resolvedOptions(options); // 对 options 做相应的处理，返回一个object对象
  
  if (resolved.fullscreen && fullscreenInstance) { // 如果是渲染全屏loading，则先销毁之前的全屏loading：可能存在之前的全屏loading
    fullscreenInstance.remvoeElLoadingChild()
    fullscreenInstance.close()
  }

  const instance = createLoadingComponent({ // 创建 Loading 组件实例
    ...resolved,
    closed: () => {
      resolved.closed?.()
      if (resolved.fullscreen) fullscreenInstance = undefined
    },
  });

  addStyle(resolved, resolved.parent, instance); // 处理全屏loading的样式，给样式赋值
  addClassList(resolved, resolved.parent, instance); // 判断 `parent` 是否有定位，如果没有的话就为其添加定位
  // 将addClassList函数挂载到parent上，方便之后销毁时使用
  resolved.parent.vLoadingAddClassList = () => addClassList(resolved, resolved.parent, instance)
	/**
	 * 将 loading-number 添加到parent
	 * 因为如果在某处触发了 “全屏加载”(v-loading.body)，并且它的父对象是document.body（带有边距）
   * 则全屏加载的destroySelf函数还会删除 'el-loading-parent--relative' ，
   * 然后v-loading.body的位置将报错。
   */
  let loadingNumber: string | null = resolved.parent.getAttribute('loading-number')
  if (!loadingNumber) { // 计算loadingNumber
    loadingNumber = '1'
  } else {
    loadingNumber = `${Number.parseInt(loadingNumber) + 1}`
  }
  // 给parent设置loading-number
  resolved.parent.setAttribute('loading-number', loadingNumber)
  // 将 `Loading` 组件的dom添加进 parent
  resolved.parent.appendChild(instance.$el); 

  // 实例渲染完之后, 再修改 visible 来触发 transition
  nextTick(() => (instance.visible.value = resolved.visible)); // 赋值用户传入的 visible
  
  if (resolved.fullscreen) { // 如果是全屏loading，则给fullscreenInstance赋值
    fullscreenInstance = instance
  }
  return instance;
};
```



#### resolvedOptions

对用户的传参进行处理

```ts
import { isString } from '@vue/shared'
const resolvedOptions = (options: LoadingOptions): LoadingOptionsResolved => {
  let target: HTMLElement;
  // 如果 options.target 是字符串的话，就拿到对应的dom节点；否则就直接是dom节点了，直接赋值
  if (isString(options.target)) {
    target = document.querySelector(options.target) || document.body;
  } else {
    target = options.target || document.body;
  }
  return {
    parent: target === document.body || options.body ? document.body : target,
    background: options.background || '',
    svg: options.svg || '',
    svgViewBox: options.svgViewBox || '',
    spinner: options.spinner || false,
    text: options.text || '',
    fullscreen: target === document.body && (options.fullscreen ?? true),
    lock: options.lock ?? false,
    customClass: options.customClass || '',
    visible: options.visible ?? true,
    target,
  };
};
```



#### addStyle

```ts
import { useZIndex } from '@element-plus/hooks'
const addStyle = async (
  options: LoadingOptionsResolved,
  parent: HTMLElement,
  instance: LoadingInstance
) => {
  const { nextZIndex } = useZIndex() // 获取下一级z-index

  const maskStyle: CSSProperties = {} // 最外层遮罩（mask）的样式
  if (options.fullscreen) {
    instance.originalPosition.value = getStyle(document.body, 'position')
    instance.originalOverflow.value = getStyle(document.body, 'overflow')
    maskStyle.zIndex = nextZIndex()
  } else if (options.parent === document.body) {
    instance.originalPosition.value = getStyle(document.body, 'position')
    /**
     * await dom render when visible is true in init,
     * because some component's height maybe 0.
     * e.g. el-table.
     */
    await nextTick()
    for (const property of ['top', 'left']) {
      const scroll = property === 'top' ? 'scrollTop' : 'scrollLeft'
      maskStyle[property] = `${
        (options.target as HTMLElement).getBoundingClientRect()[property] +
        document.body[scroll] +
        document.documentElement[scroll] -
        Number.parseInt(getStyle(document.body, `margin-${property}`), 10)
      }px`
    }
    for (const property of ['height', 'width']) {
      maskStyle[property] = `${
        (options.target as HTMLElement).getBoundingClientRect()[property]
      }px`
    }
  } else {
    instance.originalPosition.value = getStyle(parent, 'position')
  }
  for (const [key, value] of Object.entries(maskStyle)) {
    instance.$el.style[key] = value
  }
}
```





#### addClassList

判断 `parent` 是否有定位，如果没有的话就为其添加定位

```ts
const addClassList = (parent: HTMLElement, instance: any) => {
  if (instance.originalPosition.value !== "absolute" && instance.originalPosition.value !== "fixed") {
    addClass(parent, "el-loading-parent--relative");
  } else {
    removeClass(parent, "el-loading-parent--relative");
  }
};
```



### createLoadingComponent

创建组件实例

```ts
// packages\components\Loading\src\loading.ts
export function createLoadingComponent(options: any) {
 	let afterLeaveTimer: number; // 定时器，Loading组件的显示隐藏渐变是300毫秒，在隐藏渐变完成后将Loading组件销毁（这里定时器定为400毫秒）
  const afterLeaveFlag = ref(false); // 是否需要销毁：用户可能会多次调用关闭Loading的方法，这个flag用来防止多次调用“销毁方法”
  const data = reactive({
    ...options, // 目前的包含：parent、visible、target 三项
    originalPosition: "", // 源元素的position信息
    originalOverflow: '', // 源元素的overflow信息
    visible: false, // visible 覆盖为 false
  });
  // 设置文字信息
  function setText(text: string) {
    data.text = text
  }
  // 销毁自身
  function destroySelf() {
    const target = data.parent
    if (!target.vLoadingAddClassList) { // 如果target.vLoadingAddClassList不存在，说明正在执行close方法
      let loadingNumber: number | string | null = target.getAttribute('loading-number')
      loadingNumber = Number.parseInt(loadingNumber as any) - 1
      if (!loadingNumber) { // 如果loadingNumber为0，则说明是最底一层loading了
        removeClass(target, 'el-loading-parent--relative') // 删除父级class
        target.removeAttribute('loading-number') // 删除父级loading-number属性
      } else { // 否则说明还不是最后一层loading，只修改loading-number属性
        target.setAttribute('loading-number', loadingNumber.toString())
      }
      removeClass(target, 'el-loading-parent--hidden')
    }
    remvoeElLoadingChild() // 移除Loading组件
  }
  function remvoeElLoadingChild(): void {
    // vm.$el：组件DOM
    vm.$el?.parentNode?.removeChild(vm.$el);
  }
  // loading关闭
  function close() {
    if (options.beforeClose && !options.beforeClose()) return
    const target = data.parent
    target.vLoadingAddClassList = undefined
    afterLeaveFlag.value = true; // 将是否需要销毁的flag设置为true
    clearTimeout(afterLeaveTimer);

    afterLeaveTimer = window.setTimeout(() => {
      if (afterLeaveFlag.value) {
        afterLeaveFlag.value = false; // 进入销毁逻辑后，将flag设置会false
        destroySelf(); // 销毁操作
      }
    }, 400); // 400毫秒后销毁，因为设置的动画渐变是 300 毫秒
    data.visible = false;
    
    options.closed?.() // 如果传入参数有closed函数，则执行一下（相当于回调函数）
  }
  // Transition结束后（显示/隐藏）执行的回调函数
  function handleAfterLeave() {
    if (!afterLeaveFlag.value) return
    afterLeaveFlag.value = false
    destroySelf()
  }
  // Loading组件
  const hayLoadingComponent = {
    name: "ElLoading",
    setup() {
      return () => {
        const svg = data.spinner || data.svg; // 看看用户是否传入了旋转器图标
        // 旋转器图标
        const spinner = h(
          "svg", // svg标签
          {
            class: "circular",
            viewBox: data.svgViewBox || "25 25 50 50", // 默认从 25 25 开始，向右侧和下侧延伸 50px
            ...(svg ? { innerHTML: svg } : {}),
          },
          [
            h("circle", { class: "path", cx: "50", cy: "50", r: "20", fill: "none" }), // svg标签内包裹circle标签
          ],
        );
        // 旋转器文字：如果用户传入了text，则生成p标签，否则为undefined
        const spinnerText = data.text
          ? h("p", { class: "el-loading-text" }, [data.text])
          : undefined;
        // 真正返回的渲染函数
        return h(
          Transition, // 最外层是一个Transition组件，使其有渐变的效果
          { name: "el-loading-fade", onAfterLeave: handleAfterLeave }, // Transition组件的属性
          {
            // 默认位置
            // withDirectives：允许将指令应用于VNode，返回一个包含应用指令的VNode
            // withDirectives(vnode, directives)
            default: () => withDirectives( 
              createVNode( // 创建vnode
                "div", // 最外层遮罩
                {
                  style: {
                    backgroundColor: data.background || "",
                  },
                  class: [
                    "el-loading-mask",
                    data.customClass, // 用户传入的自定义class
                      data.fullscreen ? 'is-fullscreen' : '',
                  ],
                },
                [
                  h(
                    "div", // 旋转器和文字
                    {
                      class: "el-loading-spinner",
                    },
                    [spinner, spinnerText],
                  ),
                ],
              ),
              [[vShow, data.visible]], // 指令对应的vnode绑定v-show，绑定依赖的值为 data.visible。详见withDirectives使用
            ),
          },
        );
      }
    }
  }
  // createApp创建组件实例并挂载到一个空的div上
  const vm = createApp(hayLoadingComponent).mount(document.createElement("div"));
  return {
    ...toRefs(data), // data中含有全部的options
    setText,
    remvoeElLoadingChild,
    close, // 销毁方法
    vm, // Loading组件实例
    // 通过 $el 来获取 vm 的dom
    get $el(): HTMLElement {
      return vm.$el;
    },
  };
}
export type LoadingInstance = ReturnType<typeof createLoadingComponent>
```



### 指令更新钩子：`updated`

```ts
export const vLoading: Directive = {
  ...
  updated(el, binding) {
    const instance = el[INSTANCE_KEY]; // mounted时候挂载的instance
    if (binding.oldValue !== binding.value) { // 新旧内容如果不一致，才进入更新
      if (binding.value && !binding.oldValue) { // 新值存在并且旧值不存在---创建实例
        createInstance(el, binding);
      } else if (binding.value && binding.oldValue) { // 新值旧值都存在
        if (isObject(binding.value)) { // 如果值是对象，则进行更新
        	updateOptions(binding.value, instance!.options)
        }
      } else { // 新值旧值都不存在，销毁实例
        instance?.instance.close();
      }
    }
  },
};
```

#### updateOptions

```ts
const updateOptions = (
  newOptions: UnwrapRef<LoadingOptions>, // 新配置
  originalOptions: LoadingOptions // 旧配置
) => {
  for (const key of Object.keys(originalOptions)) {
    if (isRef(originalOptions[key]))
      originalOptions[key].value = newOptions[key]
  }
}
```







### 指令销毁钩子：`unmounted`

```ts
export const vLoading: Directive<ElementLoading, LoadingBinding> = {
  ...
  unmounted(el) {
    el[INSTANCE_KEY]?.instance.close()
  },
}
```





## 四、样式

```scss
// packages\components\Loading\src\index.scss
.el-loading-mask { // 遮罩样式
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  background-color: rgba(255, 255, 255, 0.9);
  z-index: 2000;
  transition: opacity 2s;
  .el-loading-spinner {
    position: absolute;
    width: 100%;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    .circular {
      display: inline;
      height: 42px;
      width: 42px;
      animation: loading-rotate 2s linear infinite;
    }
    .path {
      animation: loading-dash 1.5s ease-in-out infinite;
      stroke-dasharray: 90,150;
      stroke-dashoffset: 0;
      stroke-width: 2;
      stroke: #409eff;
      stroke-linecap: round;
    }
    .el-loading-text{
      color: #409eff;
      margin: 3px 0;
      font-size: 14px;
    }
  }
}
@keyframes loading-rotate {
  100% {
    transform: rotate(360deg);
  }
}
@keyframes loading-dash {
  0% {
    stroke-dasharray: 1, 200;
    stroke-dashoffset: 0;
  }
  50% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -40px;
  }
  100% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -120px;
  }
}
// Transition渐变样式
.el-loading-fade-enter-active {
  transition: all .3s ease;
}
.el-loading-fade-leave-active {
  transition: all .3s ease;
}
.el-loading-fade-enter-from,
.el-loading-fade-leave-to {
  opacity: 0;
}
```





































