# json5

json5 是 json的一个超集，通过引入部分 ECMAScript 5.1 的特性来扩展 json 的语法，以减少 json 格式的某些限制。同时，保持兼容现有的 json 格式。

json5 更像是 Javascript 中的对象，但是它不是 json 官方的扩展，所以需要 json5 作为文件扩展名。



## 安装

```bash
npm i json5 -S
```



## 与 json 的区别

### 对象 Objects

- 对象的键名可以是任何 ES 5.1 的标识符。
- 对象可以以一个逗号结尾。



### 数组 Arrays

- 数组可以以一个逗号结尾



### 字符串 Strings

- 字符串可以使用单引号表示
- 字符串可以占据多行文本，以 '' 换行
- 字符串可以包括转义字符



### 数字 Numbers

- 可以使用 16 进制数字。
- 数字可以以小数点开头/结尾。
- 数字可以取 Infinity / -Infinity / NaN
- 数字可以以一个明确的加号"+"开头。



### 注释 Comments

- 允许单行/多行注释



### 空白符 White Space

- 允许多余的空白符



