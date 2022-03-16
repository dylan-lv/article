# unplugin-auto-import

自动导入 vue3 的api

### 安装

```bash
npm i -D unplugin-auto-import
```

### 配置

vite.config.ts

```ts
import AutoImport from "unplugin-auto-import/vite"
export default defineConfig({
  plugins: [
    AutoImport({
      imports: ["vue"],
    }),
  ],
})
```

- 使用前

```vue
<template>
  <button @click="add">add</button>
  {{ count }}
</template>

<script setup lang="ts">

import { ref } from 'vue';
const count = ref(0)
function add () {
  count.value++
}
</script>
```

- 使用后

```vue
<template>
  <button @click="add">add</button>
  {{ count }}
</template>

<script setup lang="ts">

const count = ref(0)
function add () {
  count.value++
}
console.log(add)

</script>
```

