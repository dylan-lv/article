# 清除代码中的console

在日常开发环境中，为了方便调试我们往往会加入许多`console`打印。但是我们不希望在生产环境中存在打印的值。那么这里我们自己实现一个`loader`去除代码中的`console`

> 知识点普及之`AST`。`AST`通俗的来说，假设我们有一个文件`a.js`,我们对`a.js`里面的1000行进行一些操作处理,比如为所有的`await`增加`try catch`,以及其他操作，但是`a.js`里面的代码本质上来说就是一堆字符串。那我们怎么办呢，那就是转换为带标记信息的对象(抽象语法树)我们方便进行增删改查。这个带标记的对象(抽象语法树)就是`AST`。

```bash
npm i -D @babel/parser @babel/traverse @babel/generator @babel/types
```

- @babel/parser： 将源代码解析成 `AST`
- @babel/traverse：对`AST`节点进行递归遍历，生成一个便于操作、转换的`path`对象
- @babel/generator：将 `AST`解码生成 `js`代码
- @babel/types：通过该模块对具体的`AST`节点进行进行增、删、改、查

```js
// loader/drop-console-loader.js
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generator = require('@babel/generator').default;
const t = require('@babel/types');

module.exports = function(source) {
  const ast = parser.parse(source, { sourceType: 'module' })
  traverse(ast, {
    CallExpression(path) {
      if(t.isMemberExpression(path.node.callee) && t.isIdentifier(path.node.callee.object, {name: "console"})){
        path.remove()
      }
    }
  })
  const output = generator(ast, {}, source);
  return output.code
}
```















