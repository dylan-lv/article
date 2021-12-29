# npm中，你不了解的.npmrc文件



## .npmrc的作用

.npmrc，可以理解成npm running cnfiguration, 即npm运行时配置文件。我们知道，npm最大的作用就是帮助开发者安装需要的依赖包，但是要从哪里下载？下载哪一个版本的包，把包下载到电脑的哪个路径下？

这些都可以在.npmrc中进行配置。

在设置.npmrc之前，我们需要知道：在你的电脑上，不止存在一个.npmrc文件，而是有多个。在我们安装包的时候，npm按照如下顺序读取这些配置文件：

1. 项目配置文件：你可以在项目的根目录下创建一个.npmrc文件，只用于管理这个项目的npm安装。
2. 用户配置文件：在你使用一个账号登陆的电脑的时候，可以为当前用户创建一个.npmrc文件，之后用该用户登录电脑，就可以使用该配置文件。可以通过 **npm config get userconfig** 来获取该文件的位置。
3. 全局配置文件： 一台电脑可能有多个用户，在这些用户之上，你可以设置一个公共的.npmrc文件，供所有用户使用。该文件的路径为：**$PREFIX/etc/npmrc**，使用 **npm config get prefix** 获取$PREFIX。如果你不曾配置过全局文件，该文件不存在。
4. npm内嵌配置文件：最后还有npm内置配置文件，基本上用不到，不用过度关注。



## 如何设置.npmrc

### 1、设置项目配置文件

在项目的根目录下新建 .npmrc 文件，在里面以 **key=value** 的格式进行配置。比如要把npm的源配置为淘宝源，可以参考以下代码：

```js
registry=https://registry.npm.taobao.org
```

### 2、设置用户配置文件

你可以直接通过 **npm config get userconfig** 命令找到该文件的路径，然后直接仿照上述方法该文件，也可以通过 **npm config set** 命令继续设置，命令如下：

```js
config set registry https://registry.npm.taobao.org
```

最终，命令行会帮助我们修改对应的配置文件。只不过使用命令行更加快捷。

如果想要删除一些配置，可以直接编辑.npmrc文件，也可以使用命令进行删除，比如：

```js
npm config delete registry
```

### 3、设置全局配置文件

方法和设置用户配置文件如出一辙，只不过在使用命令行时需要加上 **-g** 参数。

```js
npm config set registry https://registry.npm.taobao.org -g
```

除此之外，这里列出一些常用的npm设置命令，有兴趣的话，可以了解一下，挺好玩的：

```js
npm config set <key> <value> [-g|--global]  //给配置参数key设置值为value；
npm config get <key>          //获取配置参数key的值；
npm config delete <key>       //删除置参数key及其值；
npm config list [-l]      //显示npm的所有配置参数的信息；
npm config edit     //编辑配置文件
npm get <key>     //获取配置参数key的值；
npm set <key> <value> [-g|--global]    //给配置参数key设置值为value；
```



## .npmrc文捷配置

配置仓库：

```js
registry=https://repo.huaweicloud.com/repository/npm/
```

根据`scope`设置仓库：

```js
@aa:registry=https://repo.huaweicloud.com/repository/npm/
```

配置bin目录：

```js
prefix=/usr/local/npm
```

配置node-sass等模块仓库：

```js
sass_binary_site=https://repo.huaweicloud.com/node-sass
phantomjs_cdnurl=https://repo.huaweicloud.com/phantomjs
chromedriver_cdnurl=https://repo.huaweicloud.com/chromedriver
operadriver_cdnurl=https://repo.huaweicloud.com/operadriver
```

自定义缓存目录：

```js
cache=~/.cache/npm_cache
```

配置仓库认证信息，用于私有仓库读取或上传npm包：

```js
always-auth=true
_auth="用户名:密码"的base64编码
```

在**多私库且用户名密码不同**的情况下，可以对单个私库配置_auth：

```bash
# :前为仓库地址去掉http/https
//repo.huaweicloud.com/repository/npm/:_auth=xxxxxx
```

除了使用用户名密码，还可以通过配置`_authToken`配置认证信息：

```bash
_authToken=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

配置当通过https向注册中心发出请求时，不进行SSL密钥验证（一般不推荐，不安全）：

```bash
strict-ssl=false
```



## 如何配置发布组件

npm配置组件发布的方式有两种，一种是通过配置`packege.json`实现，另一种则是通过配置文件`.npmrc`实现。

`package.json`配置方式：

```json
# @aa是组件的scope。scope在模块名name中使用时，以@开头，后边跟一个/
{
        "name": "@aa/xxx", // 发布npm包的名字
        "version": "1.0.0", // 你的npm包版本
        "description": "xxxx", // 包的描述
        "main": "dist/btn.js", // 指定组件的主入口文件
        "publishConfig": {
            "registry": "要发布的私有仓库地址，然后在.npmrc配置用户名密码"
        }
        ......
}
```

`.npmrc`配置方式：

```json
# package.json不做任何仓库的配置:
{
        "name": "@aa/xxx", // 发布npm包的名字
        "version": "1.0.0", // 你的npm包版本
        "description": "xxxx", // 包的描述
        "main": "dist/btn.js", // 指定组件的主入口文件
        ......
}

# .npmrc配置仓库地址和用户名密码：
@aa:registry=私仓地址
```

配置好仓库信息后，执行发布命令即可将打包好的组件发布到仓库中：

```bash
npm publish
```



**使用带有scope的模块**

```js
# 在package.json的dependencies标签中加上即可使用。
"dependencies": {
    "@aa/mypackage": "^1.3.0"
}
```





















