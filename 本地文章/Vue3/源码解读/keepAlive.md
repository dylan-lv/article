# KeepAlive

​		keepAlive是Vue的内置组件，作用是**将组件缓存在内存当中，防止重复渲染DOM**，属于**空间换时间**的方式。配置了**keepAlive**的路由或组件，只会在页面初始化的时候执行 `created -> mounted` 生命周期，第二次及以后再进入该页面将不再执行这段生命周期，而会直接去读取缓存信息。

​		被 `keepAlive` 包裹的组件和页面，激活时的声明周期为：`created -> mounted -> activated`。

​		其中 `created -> mounted` 是第一次进入时才会执行，`activated` 在每次激活时都会执行，属于 `keepAlive` 的一个特定的生命周期，

deactivated：组件离开时触发



## 使用方式

### 方式一：路由配置

#### 1、App.vue文件

```vue
<template>
  <router-view v-slot="{ Component }">
    <!-- 路由的meta信息中，设置了keepAlive为true的路由将会被缓存 -->
    <keep-alive>
      <component :is="Component" v-if="$route.meta.keepAlive" :key="route.name"  />
    </keep-alive>
    <!-- 没设置keepAlive的路由就是默认的路由 -->
    <component :is="Component" v-if="!$route.meta.keepAlive" :key="route.name"  />
  </router-view>
</template>
```

#### 2、添加 `keepAlive` 属性

- 添加三个测试页面：a、b、c

```ts
// router.ts
const routes: Array<RouteRecordRaw> = [
  {
    path: "/a",
    name: "a",
    meta: {
      keepAlive: true
    },
    component: () => import("~/pages/a.vue"),
  },
  {
    path: "/b",
    name: "b",
    meta: {
      keepAlive: true
    },
    component: () => import("~/pages/b.vue"),
  },
  {
    path: "/c",
    name: "c",
    component: () => import("~/pages/c.vue"),
  },
];
```

#### 3、修改 a、b、c三个组件

```vue
<template>
  <div>
    a
    <button @click="jump('/b')">跳转b</button>
    <button @click="jump('/c')">跳转c</button>
  </div>
</template>

<script lang="ts">
import { defineComponent, onActivated, onMounted } from "vue";
import { useRouter } from "vue-router";
export default defineComponent({
  setup() {
    const route = useRouter();
    onMounted(() => {
      console.log("a mounted");
    });
    onActivated(() => {
      console.log("a activated");
    });

    const jump = (path: string) => {
      route.push({ path });
    };

    return { jump };
  },
});
</script>
```

b和c组件类似

#### 4、测试结果

- 起始页面为 a，输出：`a mounted -> a activated`
- 跳转页面 b，输出：`b mounted -> b activated`
- 跳转页面c，输出：`c mounted` -----**页面c**不是 `keepAlive`组件，故没有 `activated` 生命周期
- 跳转页面a，输出：`a activated`
- 跳转页面b，输出：`b activated`
- 跳转页面c，输出：`c mounted`



#### 5、动态设置路由 keepAlive 属性

​		我们用完 `keepAlive` 缓存之后，如果想让当前页面或者下一个页面不再缓存（不存在于`keepAlive`中），可以在 `beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave`中操作。

```ts
export default defineComponent({
  setup() {
    // ...
  },
  beforeRouteLeave(to, from, next) {
    // 设置下一个路由的meata
    to.meta.keepAlive = false
    // 设置当前路由的meta
    from.meta.keepAlive = false
    next() // 进行下一步
  }
});
```



### 方式二：组件配置缓存

​		有些时候我们仅需要缓存页面中的某些组件，或者是在使用动态组件 `component` 进行组件切换的时候对组件进行缓存

```vue
<script setup lang="ts">
import { ref } from "vue";
import aComp from "./a.vue";
import bComp from "./b.vue";
  
const component = ref(true);
const change = () => {
  component.value = !component.value;
};
</script>

<template>
  <div>
    <button @click="change">切换</button>
    <keep-alive>
      <aComp v-if="component" />
      <bComp v-else />
    </keep-alive>
  </div>
</template>
```



## 源码解析

### 一、对外暴露的  `KeepAlive` 组件

```ts
export const KeepAlive = KeepAliveImpl as any as {
  __isKeepAlive: true
  new (): {
    $props: VNodeProps & KeepAliveProps
  }
}
```

这里可以看出，实际上的 `KeepAlive` 组件，是 `KeepAliveImpl` 实现的，`KeepAlive` 仅仅是对齐类型做了一些约束



### 二、`KeepAliveImpl` 参数

```ts
const KeepAliveImpl: ComponentOptions = {
  name: `KeepAlive`,
  // 私有属性，标记该组件是一个KeepAlive组件
  __isKeepAlive: true,
  props: {
    // 用于匹配需要缓存的组件
    include: [String, RegExp, Array],
    // 用于匹配不需要缓存的组件
    exclude: [String, RegExp, Array],
    // 用于设置缓存上限
    max: [String, Number]
  },
  setup(props: KeepAliveProps, { slots }: SetupContext) {
 		// ...
    return () => {
      // 如果不存在默认插槽，则返回空
      if(!slots.default) {
        return null
      }
      // 获取子节点
      const children = slots.default()
      // 获取第一个子节点
      const rawVNode = children[0]
      // 返回原始虚拟节点
      return rawVNode
    }
  }
}
```

`KeepAliveImpl` 是一个对象，对象中的参数分别是

- `name`：组件名
- `__isKeepAlive`：辨别是否是 `keep-alive` 组件时使用
- `props`：接收的参数
- `setup` 中可以获取 `props`（接收的参数），以及 `slots`（插槽信息）
  - 返回默认插槽中的虚拟节点

结论：

- `KeepAlive` 组件是一个**抽象组件**：组件中并没有 `template` 模板或者返回一个 `render` 函数
- 在 `setup` 函数中，通过参数 `slots.default()` 获取 `KeepAlive` 的子组件列表
- 返回第一个子组件的 `rawVnode`（虚拟节点），且仅支持缓存**第一个子节点**





#### 1、 `KeepAliveProps`

这里我们可以看到 `keep-alive` 组件接收三个可选参数，分别是 `include`（包含）、`exclude`（排除）、`max`（最大缓存数）

```ts
export interface KeepAliveProps {
  include?: MatchPattern
  exclude?: MatchPattern
  max?: number | string
}
```

`include` 和 `exclude` 的类型 `MatchPattern` 可以接收

1. 纯字符串
2. 正则规则
3. 字符串和正则规则组成的数组

```ts
type MatchPattern = string | RegExp | (string | RegExp)[]
```



### 三、setup中：添加或删除缓存组件

#### 1、生成 `cache` 和 `keys`

- `cache`：映射**缓存组件**的 `key: VNode`
- `keys` ：记录当前被缓存的 `VNode` 的 `key`

```ts
const KeepAliveImpl: ComponentOptions = {
 	setup(props: KeepAliveProps, { slots }: SetupContext) {
    // 获取组件实例
    const instance = getCurrentInstance()!;
    // 获取实例上下文
    const sharedContext = instance.ctx as KeepAliveContext;
    // 缓存VNode
    const cache: Cache = new Map();
    // 记录被缓存的VNode的key
    const keys: Keys = new Set();
    // 当前组件
    let current: VNode | null = null
  }
}
```



#### 2、watch

当 `include` 或 `exclude` 发生变化的时候，调整 `cache` 和 `keys` 中的内容

`name => matches(include, name)`：如果 `name` 匹配上了，则返回 `true`，否则返回 `false`

`name => !matches(exclude, name)`：如果 `name` 匹配上了，则返回 `false`，否则返回 `true`

```ts
watch(
  () => [props.include, props.exclude], // 监听 include 和 exclude
  ([include, exclude]) => {
    include && pruneCache(name => matches(include, name))
    exclude && pruneCache(name => !matches(exclude, name))
  },
  // flush-post，在DOM更新之后执行
  { flush: 'post', deep: true }
)
```



#### 3、matches

判断参数1中是否存在参数2，返回一个 `bool` 值

```ts
function matches(pattern: MatchPattern, name: string): boolean {
  if (isArray(pattern)) { // 如果是数组，则拿出数组中的每一项，继续递归调用 matches
    return pattern.some((p: string | RegExp) => matches(p, name))
  } else if (isString(pattern)) { // 如果是字符串，则根据逗号（,） 拆分，看看 name 是否包含在其中
    return pattern.split(',').includes(name)
  } else if (pattern.test) { // 如果是正则，则直接使用 test 检测
    return pattern.test(name)
  }
  return false
}
```



#### 4、pruneCache：修剪缓存

参数 `filter` 是一个函数，接收 `name` 返回 `bool` 值

`filter` 作用：当前 `name` 是否应该被缓存

```ts
function pruneCache(filter?: (name: string) => boolean) {
  // 遍历缓存
  cache.forEach((vnode, key) => {
    // 获取组件名称
    const name = getComponentName(vnode.type as ConcreteComponent)
    // 组件名称存在，并且 没有过滤条件或没有通过过滤条件，则进行实际的修剪操作
    if (name && (!filter || !filter(name))) {
      pruneCacheEntry(key)
    }
  })
}
```

将需要修剪的组件从缓存中删除

```ts
function pruneCacheEntry(key: CacheKey) {
  // ...
  cache.delete(key)
  keys.delete(key)
}
```



#### 5、setup 返回的内容

```ts
setup(props, { slots }) {
  // ...
  
  // 正在操作的缓存组件的key
  let pendingCacheKey: CacheKey | null = null
  return () => {
    pendingCacheKey = null
    // 如果默认插槽中没有内容，则返回null
    if (!slots.default) {
      return null
    }
    // 默认插槽内组件
    const children = slots.default()
    // 第一个子组件
		const rawVNode = children[0]
    // 获取第一个子组件的 vnode
    let vnode = getInnerChild(rawVNode)
    // 组件对象
    const comp = vnode.type
    // 获取组件名称
    const name = getComponentName(
      isAsyncWrapper(vnode) // 对于异步组件，组件名称校验就应该基于被加载的组件
        ? (vnode.type as ComponentOptions).__asyncResolved || {}
        : comp
    )
    // 从props取出参数
    const { include, exclude, max } = props
    // 筛选VNode
    if (
      (include && (!name || !matches(include, name))) ||
      (exclude && name && matches(exclude, name))
    ) {
      // include没有命中，或者exclude命中了，则直接返回
      current = vnode
      return rawVNode
    }
    const key = vnode.key == null ? comp : vnode.key
    // 根据key，从缓存中获取 VNode
    const cachedVNode = cache.get(key)
    
    if (cachedVNode) { // 如果拿出了cachedVNode，说明之前已经添加过了
      // ...
      
      // 对其进行：先删除，然后再次添加（可以更新它在 Set 中的排序，没错，KeepAlive内部用的是LRU缓存淘汰算法）
      keys.delete(key)
      keys.add(key)
    } else {
      // 如果之前没有缓存，则直接添加
      keys.add(key)
      // 如果超过最大值了，则删除最旧的缓存
      if (max && keys.size > parseInt(max as string, 10)) {
        pruneCacheEntry(keys.values().next().value)
      }
    }
  }
}
```



### 四、构建缓存

```ts
const KeepAliveImpl = {
  setup(props, { slots }) {
    // 获取当前组件实例
 		const instance = getCurrentInstance()!;
    // 获取当前实例上下文
    const sharedContext = instance.ctx as KeepAliveContext;
    // 服务端渲染方式
    if (__SSR__ && !sharedContext.renderer) {
      // ...
    }
    // 缓存
    const cache: Cache = new Map()
    const keys: Keys = new Set()
    let current: VNode | null = null
    // 渲染之后缓存子节点
    let pendingCacheKey: CacheKey | null = null
    const cacheSubtree = () => {
      if (pendingCacheKey != null) {
        cache.set(pendingCacheKey, getInnerChild(instance.subTree))
      }
    }
    // 挂载和更新时，缓存组件
    onMounted(cacheSubtree)
    onUpdated(cacheSubtree)
    return () => {
     	pendingCacheKey = null
      // 获取内部子节点
      let vnode = getInnerChild(rawVNode)
      // 获取子节点对象
      const comp = vnode.type as ConcreteComponent
      // 准备缓存需要的 key
      const key = vnode.key == null ? comp : vnode.key
      // 通过key获取vnode
      const cachedVNode = cache.get(key)
      // 更新 pendingCacheKey 
      pendingCacheKey = key
    }
  }
}
```



### 五、清空缓存

```ts
const KeepAliveImpl = {
  setup(props, { slots }) {
    // 卸载组件
    function unmount(vnode: VNode) {
      // reset the shapeFlag so it can be properly unmounted
      resetShapeFlag(vnode)
      _unmount(vnode, instance, parentSuspense, true)
    }
    // 卸载组件，卸载cache
    onBeforeUnmount(() => {
      // 遍历缓存
      cache.forEach(cached => {
        const { subTree, suspense } = instance
        const vnode = getInnerChild(subTree)
        if (cached.type === vnode.type) {
          // current instance will be unmounted as part of keep-alive's unmount
          resetShapeFlag(vnode)
          // but invoke its deactivated hook here
          const da = vnode.component!.da
          da && queuePostRenderEffect(da, suspense)
          return
        }
        // 清理缓存
        unmount(cached)
      })
    })
    
  }
}
```



### 六、`activated`和`deactivate` 钩子

在 `Vue3` 中使用的方式：`onActivated` 和 `onDeactivated`

```ts
export function onActivated(
  hook: Function,
  target?: ComponentInternalInstance | null
) {
  registerKeepAliveHook(hook, LifecycleHooks.ACTIVATED, target)
}
export function onDeactivated(
  hook: Function,
  target?: ComponentInternalInstance | null
) {
  registerKeepAliveHook(hook, LifecycleHooks.DEACTIVATED, target)
}
```

这里他们都调用了同一个方法 `registerKeepAliveHook` 来注册 `KeepAlive` 组件的钩子

```ts
const KeepAliveImpl = {
  setup(props, { slots }) {
    // 在组件上下文（ctx）上挂载activate钩子
    sharedContext.activate = (vnode, container, anchor, isSVG, optimized) => {
      const instance = vnode.component!;
      // 移动节点
      move(vnode, container, anchor, MoveType.ENTER, parentSuspense)
      // 属性可能会发生改变，所以patch一下
      patch(
        instance.vnode,
        vnode,
        container,
        anchor,
        instance,
        parentSuspense,
        isSVG,
        vnode.slotScopeIds,
        optimized
      )
      // 后置任务池中添加任务
      queuePostRenderEffect(() => {
        // 非激活状态设为 false
        instance.isDeactivated = false
        if (instance.a) {
          invokeArrayFns(instance.a)
        }
        const vnodeHook = vnode.props && vnode.props.onVnodeMounted
        if (vnodeHook) {
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
      }, parentSuspense)

      if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
        // Update components tree
        devtoolsComponentAdded(instance)
      }
    }
    // 停止工作时
    sharedContext.deactivate = (vnode: VNode) => {
      const instance = vnode.component!
      move(vnode, storageContainer, null, MoveType.LEAVE, parentSuspense)
      queuePostRenderEffect(() => {
        if (instance.da) {
          invokeArrayFns(instance.da)
        }
        const vnodeHook = vnode.props && vnode.props.onVnodeUnmounted
        if (vnodeHook) {
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
        instance.isDeactivated = true
      }, parentSuspense)

      if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
        // Update components tree
        devtoolsComponentAdded(instance)
      }
    }
  }
}
```







### 总结

- 构建缓存
  - `KeepAlive`的缓存构建，是在 `onMounted` 和 `onUpdated` 中通过 `cacheSubtree` 完成的
  - 使用 `pendingCacheKey` 来记录 `pending` 状态下 `key`
  - 如果组件的 `VNode` 之前被缓存过，则会更新 `keys` 中它的位置（LRU缓存淘汰算法）

- 更新缓存、剪枝
  - `cache` 和 `keys` 用于记录缓存组件的信息
  - `pruneCache` 和 `pruneCacheEntry` 对缓存进行修剪。遍历 `cache`，对`cache` 和 `keys` 判断并操作
  - `watch`：监听 `include` 和 `exclude` 的变化，并更新 `cache` 和 `keys`
  - 条件筛选：`include` 没有命中，或者 `exclude` 命中了，则直接返回 `rawVNode`(原始vnode)，不会被缓存
  - `max`：判断缓存数量是否已经超过了上限，如果超过，则会删除掉缓存中**最旧的** `vnode`

- hooks
  - `activated` 会移动节点并调用 `patch` 方法更新页面，向任务调度器的后置任务池中添加 `VNode` 的相关钩子
  - `deactivated` 回移除 `VNode`，向任务调度器的后置任务池中卸载相关的 `VNode` 钩子














