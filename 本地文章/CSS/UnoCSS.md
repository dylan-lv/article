## 初始化UnoCSS

安装unocss和三个预设，第一个是工具类预设，第二个是属性化模式支持，第三个是icon支持

```bash
pnpm i -D unocss @unocss/preset-uno @unocss/preset-attributify @unocss/preset-icons
```

在vite.config.ts中引入

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

import Unocss from 'unocss/vite'
import { presetUno, presetAttributify, presetIcons } from 'unocss'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    Unocss({
      presets: [
          presetUno(), 
          presetAttributify(), 
          presetIcons()],
    }),
  ],
})

```

在main.ts中引入uno.css

```ts
import { createApp } from 'vue'
import App from './App.vue'

import 'uno.css'

createApp(App).mount('#app')
```

在app.vue中编写一个button

```html
  <button
    class="
      py-2
      px-4
      font-semibold
      rounded-lg
      shadow-md
      text-white
      bg-green-500
      hover:bg-green-700
      border-none
      cursor-pointer
    "
  >
    Click me
  </button>
```

现在就可以看到效果了。

接下来我们看一个打开一个神奇的工具UnoCSS Inspector，打开[localhost:3000/_unocss](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A3000%2F__unocss%23%2F)，

这是一个可以直观地查看unocss通过预设规则生成了什么工具类，可以查看构建的css文件大小，css规则数量以帮助我们更加方便地调整和优化代码，



## 属性化模式预设

[`@unocss/preset-attributify`](https://github.com/unocss/unocss/tree/main/packages/preset-attributify)

使用属性化可以增强代码的可读性，比如上面的button可以改写成

```html
<button
    p="y-2 x-4"
    font="semibold"
    shadow="lg"
    text="white"
    bg="green-500 hover:green-700"
    border="rounded-lg none "
    cursor="pointer"
>Click me</button>
```



## 纯CSS图标支持

使用`@unocss/preset-icons`预设，再配合 `iconify` 图标框架提供的图标集，我们可以直接用css使用上w个图标而不需要过多的配置！

首先我们去 [icones](https://icones.js.org/) 官网（方便浏览和使用iconify）浏览我们需要的icon，比如这里我用到了 `Google Material Icons` 图标集里面的 `baseline-add-circle` 图标

我们先记住Google Material Icons所在的网页路径是ic

然后安装这个图标集

```bash
pnpm i -D @iconify-json/ic
```

包名后面的路径跟icones官网的路径是相对应的，其他图标集同理

然后我们在html中就可以直接使用这个图标，还能用unocss对它进行样式定制甚至让它动起来！

```html
<div class="i-ic-baseline-add-circle text-3xl bg-green-500" />
```

图标的使用语法是`i+${图标集缩写名}+${图标名}`，这里的图标集是ic，图标名是baseline-add-circle







<div class="i-ic-baseline-add-circle text-3xl bg-green-500" />





