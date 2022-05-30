# vite+vue3 的 SvgIcon组件

### 安装依赖

```bash
pnpm add -D vite-plugin-svg-icons
```

### vite.config.ts

```ts
import { createSvgIconsPlugin } from "vite-plugin-svg-icons"; // icon图标
export default defineConfig({
  plugins: [
    // 获取全部 SymbolId ： import ids from 'virtual:svg-icons-names'
    createSvgIconsPlugin({
      // 指定需要缓存的图标文件夹
      iconDirs: [path.resolve(process.cwd(), "src/icons")],
      // 指定symbolId格式
      symbolId: "icon-[dir]-[name]",
    }),
  ],
});
```

### svg图片存放位置

```ts
src
  - icons
    .svg
    .svg
vite.config.ts
```

### SvgIcon组件

```ts
// src\components\SvgIcon\index.tsx
import { computed, defineComponent } from "vue";
import "./index.scss";

// doc: https://panjiachen.github.io/vue-element-admin-site/feature/component/svg-icon.html#usage
function isExternal(path: string) {
  return /^(https?:|mailto:|tel:)/.test(path);
}

export default defineComponent({
  name: "SvgIcon",
  props: {
    iconClass: { type: String, required: true },
    className: { type: String, default: "" },
  },
  setup(props) {
    const external = computed(() => isExternal(props.iconClass));
    const iconName = computed(() => `#icon-${props.iconClass}`);
    const svgClass = computed(() => props.className ? `svg-icon ${props.className}` : "svg-icon");
    const styleExternalIcon = computed(() => ({
      mask: `url(${props.iconClass}) no-repeat 50% 50%`,
      "-webkit-mask": `url(${props.iconClass}) no-repeat 50% 50%`,
    }));

    return { external, iconName, svgClass, styleExternalIcon };
  },
  render() {
    return this.external
      ? <div style={this.styleExternalIcon} class="svg-external-icon svg-icon" {...this.$attrs}/>
      : <svg class={this.svgClass} aria-hidden="true" {...this.$attrs}>
        <use xlinkHref={this.iconName} />
      </svg>;
  },
});
```

```scss
// src\components\SvgIcon\index.scss
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}

.svg-external-icon {
  background-color: currentColor;
  mask-size: cover!important;
  display: inline-block;
}
```



### main.ts

```ts
import { createApp } from "vue";
import "virtual:svg-icons-register";
import SvgIcon from "~/components/SvgIcon"; // icon图标
const app = createApp(App);

app.component("SvgIcon", SvgIcon);
app.mount("#app");
```









