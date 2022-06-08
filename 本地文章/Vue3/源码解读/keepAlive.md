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















