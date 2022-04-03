# npm依赖管理peerDependencies

使用过 `npm` 的同学，必然会比较熟悉 `devDependencies`（开发依赖管理） 和 `dependencies`（生产依赖管理）。

但是对于没有写过 `npm` 库的同学而言，`peerDependencies` 可能就会有点陌生了



## `dependencies` 和 `devDependencies`

`dependencies` 和 `devDependencies` 是 `package.josn` 中的一个属性，其中放着运行代码时所需的依赖；在 `npm install` 时，这些依赖包会被安装；打包项目时， `dependencies` 中的包会被打包进去。



## `peerDependencies`

`peerDependencies` 也是 `package.json` 中的一个属性，翻译过来是**对等依赖**的意思，其中的包在 `npm install` 时并不会被安装；打包项目时，其中的包也不会被打包进去。

`peerDependencies` 中的包是没有显式依赖的，它默认库的使用者项目内已经安装过相关依赖，但是它不会自动检测并帮你安装。

例如：

```json
"peerDependencies": {
  "echarts": "^5.3.1",
  "vue": "^3.2.25"
},
```

在 `npm` 库的开发过程中，使用到了 `vue和echarts` 这两个依赖库，将其放在了 `peerDependencies` 中，此时当前库进行打包的时候，`vue和echarts` 这两个库就不会被打包进去。但是使用这个 `npm` 库的用户，需要在自己的项目中额外安装 `vue和echarts` 这两个依赖库。



## npm2和npm3+中`peerDependencies`的区别

在 `npm2` 中，`packageName` 包中 `peerDependencies` 的依赖会随着 `npm install packageName` 一起被强制安装。所以不需要库的使用者额外安装 `peerDependencies` 所需的依赖。

在 `npm3+` 中，则不会强制安装 `peerDependencies` 中的依赖。但是使用者没有提前安装 `peerDependencies` 中所需依赖的话， `npm3` 会在安装结束后打印警告信息：`xxx 是一个需要的依赖，但是没有被安装`。此时，使用者需要手动的在项目中安装 `xxx` 依赖。



`npm` 文档中对 `peerDependencies` 的介绍：

> 在开发插件时，你的插件需要某些依赖的支持，但是你又没必要去安装，因为插件的宿主回去安装这些依赖。此时就可以用 `peerDependencies` 去声明一下需要依赖的插件和版本。如果出问题的话，`npm` 会有警告来提示使用者去解决版本中的冲突。



## 版本号

- `~`：匹配最新的版本号，例如 `~2.3.0` 会匹配最新的 `2.3.x` 版本
- `^`：匹配次要的版本号，例如 `^2.0.0` 会匹配最新的 `2.x.x` 版本
- `>`、`<`、`>=`、`<=`：匹配区间的版本号：例如 `>=2.0.0` 会匹配 `2.0.0` 版本之后的版本









