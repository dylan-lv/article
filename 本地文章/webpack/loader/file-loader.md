# file-loader

## 一、什么是 file-loader

简单来说，`file-loader` 就是在 `JavaScript` 代码里 `import/require` 一个文件时，会将该文件生成到输出目录，并且在 `JavaScript` 代码里返回该文件的地址。



## 二、如何使用

### 1、安装

```bash
npm i file-loader -D
```

### 2、配置webpack

```js
// webpack.config.js
module: {
  rules: [
    {
      test: /.(png|gif|jpe?g)$/,
      use: [
        {
          loader: 'file-loader',
          options: {}
        }
      ]
    }
  ]
},
```

关于 `file-loader` 的 `options`，这里就不多说了，见 [file-loader options ](https://v4.webpack.js.org/loaders/file-loader/).

### 3、引入一个文件，可以是import或require

```js
// src/index.js
import logo from '../assets/image/logo.png'
console.log('logo的值：', logo);
```

### 4、打包

```bash
npm run build
```

如果使用了 `file-loader`， `dist` 目录这时候会生成我们用到的那个文件，在这里也就是 `logo.png`。

默认情况下，生成到 `dist` 目录的文件不会是原文件名，而是：`[原文件内容的 MD5 哈希值].[原文件扩展名]`。



## 源码解析

















