# 编写一个算法解析以下符号，转换为json树的结构

```js
let str = `<xml><div><p><a/></p><p></p></div></xml>`
```

题解：

```js
const str = `<xml><div><p><a/></p><p></p></div></xml>`;

const startTagReg = /\<(\w+)\>/; // 匹配开始标签
const endTagReg = /<\<\/(\w+)>\>/; // 匹配结束标签
const clouseSelfTagReg = /\<(\w+)\/\>/; // 匹配自闭合标签
const tagReg = /\<\/?(\w+)\/?\>/g; // 全局匹配标签

const matchedTags = str.match(tagReg); // 在字符串中匹配得到的标签数组

let htmlTree = null; // 保存生成的节点树
const nodeStacks = []; // 保存遍历过程中的节点栈
const currentNode = undefined;
```



















