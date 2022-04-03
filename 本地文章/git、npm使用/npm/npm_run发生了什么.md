# 运行 `npm run xxx` 的时候发生了什么

`npm run xxx`相信大家都经常用，但是 `npm run` 究竟发生了什么呢，它又是如何执行的呢，让我们来解开迷雾。

初步： `npm run xxx` 的时候，首先回去项目的 `package.json` 文件里找 `scripts` 中对应的 `xxx` 命令。例如启动 `vue` 项目 `npm run serve` 的时候，实际上就是执行了 `vue-cli-service serve` 这条命令。



- 为什么不直接执行 `vue-cli-service serve` 而要执行 `npm run serve` 呢？

因为直接执行 `vue-cli-service serve` 会报错，因为操作系统中没有存在 `vue-cli-service serve` 这条指令。



- 那既然 `vue-cli-service serve` 这条指令不存在操作系统中，为什么执行 `npm run serve` 的时候，也就是相当于执行了 `vue-cli-service serve`，为什么这样它就能成功，而且不报指令不存在的错误呢？

我们在安装依赖的时候，是通过 `npm i xxx` 来执行的，例如 `npm i @vue/cli-service`，`npm` 在安装这个依赖的时候，就会在 `node_modules/.bin/` 目录中创建好以 `vue-cli-service` 为名称的几个可执行文件了。

<img src=".\assets\npm1.png" alt="npm1" />

`.bin` 目录，这个目录不是任何一个 `npm` 包。

目录下的文件，表示这是一个软连接，打开文件可以看到文件顶部写着 `#!/bin/sh`，表示这是一个脚本。

由此我们可以知道，当使用 `npm run serve` 执行 `vue-cli-service serve` 时，虽然没有安装 `vue-cli-service` 的全局命令，但是 `npm` 会到 `./node_modules/.bin` 中找到 `vue-cli-service` 文件作为脚本来执行，则相当于执行了 `./node_modules/.bin/vue-cli-service serve`（最后的 serve 作为参数传入）。



- `.bin` 目录下的文件表示软连接，那这个 `bin` 目录下的那些软连接文件是哪里来的呢？它又是怎么知道这条软连接是执行哪里的呢？

`npm` 将 `bin/vue-cli-service.js` 作为 `bin` 声明了。

所以在 `npm install` 时，npm 读到该配置后，就将**该文件**软连接到 `./node_modules/.bin` 目录下，而 `npm` 还会自动把 `node_modules/.bin` 加入到 `$PATH`，这样就可以直接作为命令运行依赖程序和开发依赖程序，不用全局安装了。

<img src=".\assets\npm2.png" alt="npm2" style="zoom: 80%;" />

假如我们在安装包时，使用 `npm install -g xxx` 来安装，那么会将其中的 `bin` 文件加入到全局，比如 `create-react-app` 和 `vue-cli`，在全局安装后，就可以直接使用 `vue-cli projectName` 这样的命令来创建项目了。

也就是说，`npm i` 的时候，`npm` 就帮我们把这种软连接配置好了，其实这种软连接相当于一种映射，执行 `npm run xxx` 的时候，就会到 `node_modules/bin` 中找对应的映射文件，然后再找到相应的 `js` 文件来执行。



- 刚刚看到在 `node_modules/bin` 中有三个 `vue-cli-service` 文件，为什么会有三个文件呢？

如果我们在 `cmd` 里运行的时候，`windows` 一般是调用了 `vue-cli-service.cmd` 这个文件。所以当我们运行 `vue-cli-service serve` 这条命令的时候，就相当于运行 `node_modules/.bin/vue-cli-service.cmd serve`。

然后这个脚本会使用 `node` 去运行 `vue-cli-service.js` 这个 `js` 文件。

由于 `node` 中可以使用一系列系统相关的 `api`，所以在这个 `js` 中可以做很多事情，例如读取并分析运行这条命令的目录下的文件，根据模板生成文件等。



### 总结

1. 运行 `npm run xxx` 的时候，`npm` 会先在当前目录的 `node_modules/.bin` 查找要执行的程序，如果找到则运行。
2. 没有找到则从全局的 `node_modules/.bin` 中查找，`npm i -g xxx` 就是安装到全局目录。
3. 如果全局目录还是没找到，那么就从 `path` 环境变量中查找有没有其他同名的可执行程序。



