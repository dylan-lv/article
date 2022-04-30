进入项目文件



### 初始化npm

```bash
pnpm init -y
```

### 新建文件 `pnpm-workspace.yaml`

```yaml
packages:
  - "packages/**"
  - "apps/**"
```

### 安装依赖

```ts
pnpm add -D eslint eslint-define-config @hayeslv/eslint-config typescript unbuild
```

### 配置 `script`

```json
// package.json
{
  "scripts": {
  	"dev": "pnpm --dir packages/platform dev",
  	"build": "pnpm run build -r",
  	"stub": "pnpm run stub -r",
	},
}
```

### `shared` 工程

```bash
# packages/shared
pnpm init -y
```

### 配置 `shared` 的 `package.json`

```json
// packages/shared/package.json
{
  "main": "./dist/index.cjs",
  "module": "./dist/index.mjs",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    }
  },
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "unbuild",
    "stub": "unbuild --stub"
  }
}

```

### 配置 `unbuild`

```ts
// packages/shared/build.config.ts
import { defineBuildConfig } from "unbuild";

export default defineBuildConfig({
  entries: [
    "src/index",
  ],
  declaration: true,
  rollup: {
    emitCJS: true,
  },
});
```

### 随便放个函数

```ts
// packages/shared/src/index.ts
export function sum(a: number, b: number): number {
  return a + b;
}
```



### 打包

```bash
pnpm run build -r
```



### 主项目配置 `tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "esnext",
    "target": "esnext",
    "resolveJsonModule": true, // 可以直接引入json
    "moduleResolution": "node", // 模块查找方式--node模式
    "baseUrl": ".",
    "useDefineForClassFields": true,
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "esModuleInterop": true,
    "lib": ["esnext", "dom"],
    "paths": {
      "~/*": ["src/*"],
      "@hayeslc/shared": ["./packages/shared/src/index"]
    }
  },
  // "include": [
  //   "apps/**/src/**/*",
  //   "packages/**/src/**/*",
  // ],
  "exclude": ["dist", "node_modules"]
}
```



### 另外一个项目中安装依赖

```bash
# packages/website
pnpm i @包名 
```

此时在 `website` 的 `package.json` 中会生成一个 `workspace`

```json
"dependencies": {
  "@hayeslc/shared": "workspace:^0.0.1",
},
```

现在就可以在 `main.ts` 中引用了

```ts
// packages/website/src/main.ts
import { sum } from "@hayeslc/shared";
console.log(sum(1, 2));
```











