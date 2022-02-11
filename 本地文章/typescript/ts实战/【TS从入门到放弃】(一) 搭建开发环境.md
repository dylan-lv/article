# TS从入门到放弃【一】：搭建开发环境

### 1、创建项目文件夹

```bash
mkdir ts-study
cd ts-study
```

### 2、npm初始化

```bash
npm init -y
```

内容如下

```json
{
  "name": "ts-study",
  "version": "1.0.0",
  "description": "学习ts",
  "main": "./src/index.ts",
  "scripts": {
  },
  "keywords": [ "typescript" ],
  "author": "dylan<xxxx@163.com>",
  "license": "MIT"
}
```

### 3、安装依赖

```bash
npm i typescript tslint -g
```

安装完这两个依赖，就可以使用tsc命令

在项目目录下执行：

```bash
tsc --init
```

此时可以看到项目中出现了 `tsconfig.json` 文件

![ts-1](.\assets\ts-1.png)

**这个文件里面可以写注释**



### 4、tslint

```bash
tslint --init
```

根目录会出现一个名为 `tslint.json` 的文件



### 5、webpack依赖

```bash
npm i webpack@4 webpack-cli@3 webpack-dev-server@3 -D
npm i typescript@4 ts-loader@4 cross-env@7 -D
npm i html-webpack-plugin@3 clean-webpack-plugin@2 -D
```

#### （1）创建webpack配置文件

![ts-2](.\assets\ts-2.png)

```js
// build/webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
  entry: "./src/index.ts",
  output: {
    filename: "main.js"
  },
  resolve: {
    extensions: [".js", ".ts", ".tsx"] // 自动解析文件扩展
  },
  module: {
    rules: [{
        test: /\.tsx?$/,
        use: "ts-loader",
        exclude: /node_modules/
      }]
  },
  devtool: process.env.NODE_ENV === "production" ? false : "inline-source-map",
  devServer: {
    contentBase: "./dist", // 基于哪个文件夹运行
    stats: "errors-only", // 只有当出现错误的时候，控制台才会打印
    compress: false, // 不启动压缩
    host: "localhost",
    port: 8089
  },
  plugins: [
    new CleanWebpackPlugin({
      cleanOnceBeforeBuildPatterns: ['./dist'], // 在build之前，把dist文件夹清理掉
    }),
    new HtmlWebpackPlugin({
      template: "./src/template/index.html"
    })
  ]
}
```



#### （2）添加html模板

目录如下：

![ts-3](.\assets\ts-3.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TS-learning</title>
</head>
<body>
  
</body>
</html>
```



#### （3）添加运行指令：package.json

```json
"scripts": {
  "start": "cross-env NODE_ENV=development webpack-dev-server --config ./build/webpack.config.js"
},
```



#### （4）新建index.ts文件

![ts-4](.\assets\ts-4.png)

#### （5）运行

```bash
npm start
```

即可看到运行成功了！

![ts-5](.\assets\ts-5.png)



#### （6）打包

```json
"scripts": {
  "build": "cross-env NODE_ENV=production webpack --config ./build/webpack.config.js"
},
```

此时是production模式，就不会生成sourcemap了

运行如下代码，即可成功打包

```bash
npm run build
```
