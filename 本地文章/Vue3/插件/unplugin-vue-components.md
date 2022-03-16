# unplugin-vue-components

vue 按需组件自动导入

### 自动导入 ui 库，该插件内置了大多数流行的库解析器

例如：element-plus、antdv、vant、view-ui

### 安装

```bash
npm install unplugin-vue-components -D
npm i ant-design-vue@next -S
```

### 配置文件

- vite配置

```ts
// vite.config.js
import Components from "unplugin-vue-components/vite";
import { AntDesignVueResolver } from "unplugin-vue-components/resolvers";
export default defineConfig({
  plugins: [
    Components({
      dirs: ["src/components"], // 配置需要默认导入的自定义组件文件夹，该文件夹下的所有组件都会自动 import
      resolvers: [AntDesignVueResolver()],
      dts: true,
    }),
  ],
});
```

插件会生成一个ui库组件以及指令路径components.d.ts文件





