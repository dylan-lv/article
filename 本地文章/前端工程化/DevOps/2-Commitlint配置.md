# DevOps之Commitlint配置

为了便于团队开发协同工作，更好的维护项目，Commitlint便油然而生。

文中的项目是基于`Commitlint` + `husky` + `eslint`的基础上搭建的。

> **@commitlint**最新版本仅支持`Node>=12`，**husky** 最新版本也仅支持`Node>=12`。



## 安装 commitlint

```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```



## 配置 commitlint

```js
// commitlint.config.js
module.exports = {
  extends: ["@commitlint/config-conventional"],
};
```



## 安装其他依赖包

```bash
npm install eslint husky commitizen cz-conventional-changelog-zh conventional-changelog-cli --save-dev 
```



## package.json 配置

```json
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog-zh"
  }
},
"scripts": {
  "lint": "eslint .",
  "commit": "git add . && git cz",
  "prepare": "npx husky install"
},
```



## 激活 husky 配置

```bash
npm run prepare
pnpm husky add .husky/commit-msg 'npm commitlint --edit $1'
```

- 修改 commit-mgs 配置 （`.husky\commit-msg`）

```sh
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint && npx --no-install commitlint --edit $1
```



最后，执行 `npm run commit` 即可















