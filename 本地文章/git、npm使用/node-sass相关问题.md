## 版本

node14对应的 版本

```bash
"node-sass": "^4.14.1"
"sass-loader": "^8.0.2"
```

node 16 对应的版本

```bash
"node-sass": "^6.0.1"
"sass-loader": "^10.0.1"
```



## 全局引用

webpack配置

```js
// 14版本
module.exports = {
  css: {
    loaderOptions: {
      sass: {
        prependData: '@import "@/styles/mixin.scss";' // 全局引入
      }
    }
  },
}
// 16版本
module.exports = {
  css: {
    loaderOptions: {
      sass: {
        additionalData: '@import "@/styles/mixin.scss";' // 全局引入
      }
    }
  },
}
```

由于sass-loader版本不同，loaderOptions中additionalData的键名也不同，

- sass-loader v8-, 选项名是 "data"
- sass-loader v8，选项名是 "prependData"
- sass-loader v10+, 选项名是 "additionalData"