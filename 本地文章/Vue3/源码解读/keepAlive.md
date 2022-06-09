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



### 三、setup

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















