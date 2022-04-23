## vite-plugin-pages

修改 `tsconfig.json`

```ts
{
  "compilerOptions": {
    "types": ["vite/client", "vite-plugin-pages/client"],
  }
}
```

修改 `vite.config.ts`

```ts
import Pages from 'vite-plugin-pages'
const config: UserConfig = {
  plugins: [
    Pages({
      extensions: ['vue', 'md'],
      pagesDir: 'pages',
      extendRoute(route) {
        const path = resolve(__dirname, route.component.slice(1))

        if (!path.includes('projects.md')) {
          const md = fs.readFileSync(path, 'utf-8')
          const { data } = matter(md)
          route.meta = Object.assign(route.meta || {}, { frontmatter: data })
        }

        return route
      },
    }),
  ]
}
```



## gray-matter



## fs-extra

文件系统



## vite-ssg

静态页面生成



## dayjs



## nprogress





## icon

```bash
yarn add @iconify/iconify -S
yarn add vite-plugin-purge-icons @iconify/json -D
```

配置vite

```json
import PurgeIcons from 'vite-plugin-purge-icons'
export default {
  plugins: [
    PurgeIcons({
      /* PurgeIcons Options */
    })
  ]
}
```

main.ts

```ts
import { createApp } from 'vue'
import App from './App.vue'

import '@purge-icons/generated' // <-- This

createApp(App).mount('#app')
```

使用

```html
<span class="iconify" data-icon="uil:github-alt" />
```



