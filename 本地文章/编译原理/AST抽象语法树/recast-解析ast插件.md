# recast-解析ast插件

安装插件

```bash
npm i recast -S
```

它可以操作语法树

**parse.js**

```js
const recase = require('recast');

// 你的"机器"——一段代码
// 我们使用了很奇怪格式的代码，想测试是否能维持代码结构
const code =
`
function add(a, b) {
  return a +
    // 有什么奇怪的东西混进来了
    b
}
`

const ast = recase.parse(code);
// ast可以处理很巨大的代码文件
// 但我们现在只需要代码块的第一个body，即add函数
const add = ast.program.body[0]

console.log(add);
```

执行代码后，可以查看到add函数的结构

```js
FunctionDeclaration {
  type: 'FunctionDeclaration',
  id: ...
  params: ...
  body: ...
}
```

add.params内容

```js
[
  Identifier {
  	type: 'Identifier',
  	name: 'a'
  },
  Identifier {
  	type: 'Identifier',
  	name: 'b'
  }
]
```

add.body.body[0].argument内容

```js
BinaryExpression {
  type: 'BinaryExpression',
  operator: '+',
  left: Identifier {
    type: 'Identifier',
    name: 'a',
  },
  right: Identifier {
    type: 'Identifier',
    name: 'b',
  }
}
```



## recast.types.builders：制作模具

一个机器，你只会拆开重装，不算本事。

拆开了，还能改装，才算上得了台面。

recast.types.builders里面提供了不少“模具”，让你可以轻松地拼接成新的机器。

最简单的例子，我们想把之前的 `function add(a, b){...}` 声明，改成匿名函数式声明 `const add = function(a ,b){...}`

1. 创建一个VariableDeclaration变量声明对象，声明头为const， 内容为一个即将创建的VariableDeclarator对象。
2. 创建一个VariableDeclarator，放置add.id在左边， 右边是将创建的FunctionDeclaration对象
3. 创建一个FunctionDeclaration，如前所述的三个组件，id params body中，因为是匿名函数id设为空，params使用add.params，body使用add.body。

这样，就创建好了`const add = function(){}`的AST对象。

在之前的parse.js代码之后，加入以下代码：

```js
// 引入“变量声明”，“变量符号”，“函数声明” 三种模具
const { variableDeclaration, variableDeclarator, functionExpression } = recase.types.builders;

// 将准备好的组件置入模具，并组装回原来的 ast 对象
ast.program.body[0] = variableDeclaration("const", [
  variableDeclarator(add.id, functionExpression(
    null, // 匿名函数表达式
    add.params,
    add.body
  ))
]);

// 将ast对象重新转回可以阅读的代码
const output = recase.print(ast).code;
console.log(output);
```

执行代码后，打印结果如下：

```js
const add = function(a, b) {
  return a +
    // 有什么奇怪的东西混进来了
    b
};
```

最后一行

```js
const output = recast.print(ast).code;
```

其实是 recase.parse 的逆向过程，具体公式为：

```js
recase.print(recase.parse(source)).code === source
```

打印出美化格式的代码段（删除注释）：

```js
const output = recase.prettyPrint(ast, { tabWidth: 2 }).code;
```

输出为：

```js
const add = function(a, b) {
  return a + b;
}
```



## 命令行修改js文件

除了parse/print/builder以外，Recast有三项主要功能：

- run：通过命令行读取 js 文件，并转化成 ast 以供处理。
- tnt：通过 assert() 和 check() ，可以验证 ast 对象的类型。
- visit：遍历 ast 树，获取有效的 ast 对象并进行更改。

**demo.js**

```js
function add(a, b) {
  return a + b;
}

function sub(a, b) {
  return a - b
}

function commonDivision(a, b) {
  while (b !== 0) {
    if (a > b) {
      a = sub(a, b)
    } else {
      b = sub(b, a)
    }
  }
  return a
}
```



### recast.run --- 命令行文件读取

新建一个名为`read.js`的文件，写入代码

```js
const recase = require('recast');

recase.run(function(ast, printSource) {
  printSource(ast)
})
```

命令行输入

```bash
node read demo.js
```

可以看到 js 文件内容打印在了控制台上。

我们可以知道，`node read`可以读取`demo.js`文件，并将demo.js内容转化为ast对象。

同时它还提供了一个`printSource`函数，随时可以将ast的内容转换回源码，以方便调试。



### recast.visit --- AST节点遍历

read.js

```js
const recase = require('recast');

recase.run(function(ast, printSource) {
  recase.visit(ast, {
    visitExpressionStatement: function({ node }) {
      console.log(node);
      return false;
    }
  })
})
```

recast.visit将AST对象内的节点进行逐个遍历。



注意：

- 你想操作函数声明，就使用 `visitFunctionFunctionDeclaration` 遍历，想操作赋值表达式，就使用 `visitExpressionStatement`。只要在 AST对象文档中定义的对象，在前面加 visit ，即可遍历。
- 通过 node 可以取到AST对象
- 每个遍历函数后必须加上 return false，或者选择以下写法，否则报错：

```js
const recast = require('recast');
recast.run(function(ast, printSource) {
  recast.visit(ast, {
    visitExpressionStatement: function(path) {
      const node = path.node;
      printSource(node);
      this.traverse(path);
    }
  })
})
```

调试时，如果你想输出AST对象，可以`console.log(node)`

如果你想输出AST对象对应的源码，可以`printSource(node)`

命令行输入`
node read demo.js`进行测试。

> `#!/usr/bin/env node` 在所有使用`recast.run()`的文件顶部都需要加入这一行，它的意义我们最后再讨论。



## TNT --- 判断AST对象类型

TNT，即recast.types.namedTypes，就像它的名字一样火爆，它用来判断AST对象是否为指定的类型。

TNT.Node.assert()，就像在机器里埋好的炸药，当机器不能完好运转时（类型不匹配），就炸毁机器(报错退出)

TNT.Node.check()，则可以判断类型是否一致，并输出False和True

上述Node可以替换成任意AST对象，例如TNT.ExpressionStatement.check(),TNT.FunctionDeclaration.assert()

```js
// read.js
const recase = require('recast');
const TNT = recase.types.namedTypes;

recase.run(function(ast, printSource) {
  recase.visit(ast, {
    visitExpressionStatement: function(path) {
      const node = path.value;
      // 判断是否为 ExpressionStatement，正确则输出一行字
      if(TNT.ExpressionStatement.check(node)) {
        console.log('这是一个ExpressionStatement');
      }
      this.traverse(path);
    }
  })
})
```

```js
// read.js
const recase = require('recast');
const TNT = recase.types.namedTypes;

recase.run(function(ast, printSource) {
  recase.visit(ast, {
    visitExpressionStatement: function(path) {
      const node = path.value;
      // 判断是否为ExpressionStatement，正确不输出，错误则全局报错
      TNT.ExpressionStatement.assert(node)
      this.traverse(path);
    }
  })
})
```

















































