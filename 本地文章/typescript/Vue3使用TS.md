首先我们需要在 script 标签上加一个配置 lang=“ts”，来标记当前组件使用了 TypeScript，然后代码内部使用 defineComponent 定义组件即可。

```vue
<script lang="ts">
import { defineComponent } from 'vue'
export default defineComponent({
  // 已启用类型推断
})
</script>
```

在 <script setup> 的内部，需要调整写法的内容不多。

```js
const count = ref(1) 
count.value.split('') // => Property 'split' does not exist on type 'number'
```

我们也可以显式地去规定 ref、reactive 和 computed 输入的属性

```vue
<script lang="ts" setup>
import { computed, reactive, ref } from "vue";

interface Course {
  name: string,
  price: number
}

const msg = ref(''); // 根据输入参数推导字符串类型
const msg1 = ref<string>(''); // 可以通过范型显示约束

const obj = reactive({});
const course = reactive<Course>({ name: 'vue3全家桶', price: 88 });

const msg2 = computed(() => ''); // 默认参数推导
const course2 = computed<Course>(() => {
  return { name: '玩转', price: 198 }
})
</script>
```



### Props 和 Emit

在 Vue 中，除了组件内部数据的类型限制，还需要对传递的属性 Props 声明类型。而在 <script setup> 语法中，只需要在 defineProps 和 defineEmits 声明参数类型就可以了。

```vue
<script lang="ts" setup>
import { computed, reactive, ref } from "vue";

interface Course {
  name: string,
  price: number
}

const msg = ref(''); // 根据输入参数推导字符串类型
const msg1 = ref<string>(''); // 可以通过范型显示约束

const obj = reactive({});
const course = reactive<Course>({ name: 'vue3全家桶', price: 88 });

const msg2 = computed(() => ''); // 默认参数推导
const course2 = computed<Course>(() => {
  return { name: '玩转', price: 198 }
})

const props = defineProps<{
  title: string,
  value?: number
}>()
const emit = defineEmits<{
  (e: 'update', value: number): void
}>()
</script>
```



示例：

初始化todolist

```ts
import {ref, Ref} from 'vue'
interface Todo{
  title:string,
  done:boolean
}
let todos:Ref<Todo[]> = ref([{title:'学习Vue',done:false}])
```



### vue-router

vue-router 提供了 Router 和 RouteRecordRaw 这两个路由的类型。

```js
import { createRouter, createWebHashHistory, Router, RouteRecordRaw } from 'vue-router'
const routes: Array<RouteRecordRaw> = [
  ...
]

const router: Router = createRouter({
  history: createWebHashHistory(),
  routes
})

export default router
```



















