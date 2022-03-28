# Vue3打包tsx组件

### package.json配置

- `"main": "dist/index.umd.js"`：umd规范
- `"module": "dist/index.esm.js"`：esm规范
- `unpkg`
- `types`
- `bumpp`：给git打tag
- `rimraf`：以包的形式包装rm -rf命令，用来删除文件和文件夹的，不管文件夹是否为空，都可删除；

```json
{
  "name": "hay-ui",
  "version": "0.1.3",
  "main": "dist/index.umd.js",
  "module": "dist/index.esm.js",
  "unpkg": "dist/index.min.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "rollup -c",
    "start": "rimraf dist/* && rollup -c -w",
    "release": "bumpp package.json --commit --push --tag"
  },
  "devDependencies": {
    "@babel/core": "^7.15.8",
    "@rollup/plugin-babel": "^5.3.0",
    "@rollup/plugin-commonjs": "^21.0.0",
    "@rollup/plugin-node-resolve": "^13.0.2",
    "@rollup/plugin-typescript": "^8.2.5",
    "@hayeslv/eslint-config": "^0.1.2",
    "@types/echarts": "^4.9.13",
    "@vue/babel-plugin-jsx": "^1.1.1",
    "rimraf": "^3.0.2", 
    "rollup": "^2.53.3",
    "rollup-plugin-peer-deps-external": "^2.2.4",
    "rollup-plugin-postcss": "^4.0.1",
    "rollup-plugin-scss": "^3.0.0",
    "rollup-plugin-terser": "^7.0.2",
    "bumpp": "^7.1.1",
    "eslint": "^8.11.0",
    "sass": "^1.49.9",
    "tslib": "^2.3.0",
    "typescript": "^4.5.4",
    "echarts": "^5.3.1",
    "vue": "^3.2.25"
  },
  "peerDependencies": {
    "echarts": "^5.3.1",
    "vue": "^3.2.25"
  },
  "files": [
    "dist"
  ],
  "eslintConfig": {
    "extends": "@hayeslv",
    "rules": {
      "@typescript-eslint/no-use-before-define": "off"
    }
  },
  "engines": {
    "node": ">=12",
    "npm": ">=6"
  }
}
```



### `.babelrc` 文件

```json
{
  "plugins": [
    ["@vue/babel-plugin-jsx"]
  ]
}
```



### tsconfig.json文件

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": ["esnext", "dom"],
    "declaration": true,
    "declarationDir": "./",
    "outDir": "build"
  },
  "include": ["packages"],
  "exclude": ["node_modules", "build", "dist", "example", "rollup.config.ts"]
}
```



### rollup.config.ts 文件

```ts
import { babel } from "@rollup/plugin-babel";
import resolve from "@rollup/plugin-node-resolve";
import commonjs from "@rollup/plugin-commonjs";
import typescript from "@rollup/plugin-typescript";
// import typescript2 from 'rollup-plugin-typescript2'
import { terser } from "rollup-plugin-terser";
import pkg from "./package.json";
import external from "rollup-plugin-peer-deps-external";
import scss from "rollup-plugin-scss";
// import dts from 'rollup-plugin-dts'

const extensions = [".ts", ".js", ".tsx"];
const globals = {
  vue: "Vue",
};
export default [
  {
    input: "packages/index.ts",
    output: [
      {
        name: "HayUI",
        file: pkg.main,
        format: "umd",
        globals,
      },
      {
        file: pkg.module,
        format: "es",
      },
      {
        name: "HayUI",
        file: pkg.unpkg,
        format: "umd",
        plugins: [terser()],
        globals,
      },
    ],
    plugins: [
      external(),
      scss({
        output: "dist/index.min.css",
      }),
      typescript({
        lib: ["es5", "es6", "dom"],
        target: "es5",
        sourceMap: false,
        tsconfig: "./tsconfig.json",
      }),
      // typescript2({
      //   rollupCommonJSResolveHack: true,
      //   clean: true
      // }),
      babel({ babelHelpers: "bundled", extensions }),
      resolve(),
      commonjs({ extensions }),
    ],
  },
  // {
  //   input: 'src/index.ts',
  //   output: [{ file: "dist/index.d.ts", format: "es" }],
  //   plugins: [dts()]
  // }
];
```



## 组件信息

### packages/index.ts

```ts
import type { App } from "vue";

/* 基础组件 start */
import HayChart from "./Chart"; // 图标
/* 基础组件 end */

// 所有组件
const components: any[] = [
  HayChart,
];

/**
 * 组件注册
 * @param {App} app Vue 对象
 * @returns {Void}
 */
const install = (app: App) => {
  // 注册组件
  components.forEach(component => app.component(component.name, component));
};

export {
  HayChart,
};

// 全部导出
export default {
  install,
  ...components,
};
```



### packages/types.ts

```ts
export type PublicProps<T, U = {}> = Readonly<T> & U; // vue 的公共 props
```



### packages/Chart 文件夹下

#### index.ts

```ts
import type { App } from "vue";
import HayChart from "./src/hayChart";

// 安装
HayChart.install = (app: App): void => {
  app.component(HayChart.name, HayChart);
};

export default HayChart;
```

#### src/hayChart.tsx

```tsx
import * as echarts from "echarts";
import { computed, defineComponent, onMounted, onUnmounted, ref, watch } from "vue";

export default defineComponent({
  name: "HayChart",
  props: {
    width: { type: Number, default: null },
    height: { type: Number, default: null },
    option: { type: Object, required: true },
  },
  setup(props) {
    const style = computed(() => {
      let str = "";
      props.width && (str += `width: ${props.width}px;`);
      props.height && (str += `height: ${props.height}px;`);
      return str;
    });
    let chartInstance: echarts.EChartsType | null = null;
    const chartsRef = ref();

    const echartRender = () => {
      if (chartInstance) clearEchart();
      chartInstance = echarts.init(chartsRef.value);
      const option = props.option;
      chartInstance.setOption(option);
    };
    const clearEchart = () => {
      chartInstance && chartInstance.dispose();
      chartInstance = null;
    };

    watch(() => props.option, () => {
      echartRender();
    });
    onMounted(() => {
      echartRender();
    });
    onUnmounted(() => {
      chartInstance && chartInstance.dispose();
    });

    return { style, chartsRef };
  },
  render() {
    return <div ref="chartsRef" class="canvas" style={this.style} />;
  },
});
```















