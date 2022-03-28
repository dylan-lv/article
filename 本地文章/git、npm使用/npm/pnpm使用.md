### pnpm初始化

```bash
pnpm init -y
```

### 配置工作空间

```yml
# pnpm-workspace.yaml
packages:
  - "packages/*"
```

### 安装依赖

```bash
pnpm i
```

此时 `packages` 中的项目依赖都会装好



### 全项目依赖

```json
"devDependencies": {
  "@unocss/autocomplete": "workspace:*"
}
```

`workspace:*` 时，会去找 `monorepo` 中另外的包，会做一个 `link` 进来，就不需要一个真的包。

等你发包的时候，它会把 `workspace:*` 替换成对应这个包的版本



### 打包

```bash
pnpm -r run build
```

这条命令执行后，会去跑每个包下面的 `build`

它会根据依赖树，先跑最底层的依赖，然后再跑上层的依赖



### 发布

```bash
pnpm -r publish
```



### root加包

```bash
pnpm add bumpp -D -w
```











