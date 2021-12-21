# loader-utils@2.0.2

<a href="https://github.com/webpack/loader-utils/tree/v2.0.2" target="_blank">loader-utils@2.0.2</a>

以下内容是 loader-utils@2.0.2 的源码及解析

## 一、parseQuery

功能：将传递的字符串（例如：loaderContext.resourceQuery）解析为查询字符串，并返回一个对象。

字符串会以如下方式转换：

- ​	=> Error
- ?  => {  }
- ?flag  =>  { flag: true }
- ?+flag  =>  { flag: true }
- ?-flag  =>  { flag: false }
- ?xyz=test  =>  { xyz: "test" }
- ?xyz=1  =>  { xyz: "1" }  // numbers are NOT parsed
- ?xyz[]=a  =>  { xyz: ["a"] }
- ?flag1&flag2  =>  { flag1: true, flag2: true }
- ?+flag1, -flag2  =>  { flag1: true, flag2: flase }
- ?xyz[]=a, xyz[]=b  =>  { xyz: ["a", "b"] }
- ?a%2C%26b=c%2C%26d  =>  { "a,&b": "c,&d" }
- ?{data:{a:1},isJSON5:true}  =>  { data: { a: 1 }, isJSON5: true }



源码如下：

```js
const JSON5 = require('json5');

const specialValues = {
  null: null,
  true: true,
  false: false,
};

// 字符串转json
function parseQuery(query) {
  // 当使用loader时的options为字符串时，接收方this.query也会接受是字符串，并以?开头
  if(query.substr(0, 1) !== '?') {
    throw new Error(
      "A valid query string passed to parseQuery should begin with '?'"
    )
  }

  query = query.substr(1); // 去掉问号

  if(!query) return {};

  // 字符串内包含的是json
  if(query.substr(0, 1) === '{' && query.substr(-1) === '}') return JSON5.parse(query);

  // 根据 , & 这两个符号分隔字符串
  const queryArgs = query.split(/[,&]/g);
  const result = {};

  queryArgs.forEach(arg => {
    const idx = arg.indexOf('=');

    if(idx >= 0) { // key=value
      let name = arg.substr(0, idx);
      let value = decodeURIComponent(arg.substr(idx + 1));

      // 将特殊字符串转成特殊字符
      if(Object.prototype.hasOwnProperty.call(specialValues, value)) {
        value = specialValues[value]
      }

      // 处理name为 'key[]'，这种类型的字符串
      if(name.substr(-2) === '[]') {
        name = decodeURIComponent(name.substr(0, name.length - 2));

        if(!Array.isArray(result[name])) {
          result[name] = [];
        }

        result[name].push(value);
      } else {
        if(arg.substr(0, 1) === '-') { // -string
          result[decodeURIComponent(arg.substr(1))] = false;
        } else if(arg.substr(0, 1) === '+') { // +string
          result[decodeURIComponent(arg.substr(1))] = true;
        } else {
          result[decodeURIComponent(arg)] = true;
        }
      }
    }
  })
}

module.exports = parseQuery;
```



下面我们之间从主函数 `parseQuery` 看起

### 1、判断是否是类url参数的字符串

```js
if(query.substr(0, 1) !== '?') {
  throw new Error(
    "A valid query string passed to parseQuery should begin with '?'"
  )
}
```

如果该字符串不是 ? 开头，则抛异常

> 注：webpack的loader使用时，如果配置了 options 参数，并且 options 的值为 `字符串` 时，loader接受参数时(this.query)，这个参数会已问号(?)开头。
>
> 另外，后面的代码做了一些其他的处理，可能是为了处理 url 参数（这里不太确定）



### 2、类型判断以及字符串切割

```js
query = query.substr(1); // 获取问号后面的参数（这才是真正需要处理的）

if(!query) { // 处理参数为空的情况
  return {};
}

if(query.substr(0, 1) === '{' && query.substr(-1) === '}') { // 前后被大括号包含(类json)，直接使用json5处理
  return JSON5.parse(query);
}

const queryArgs = query.split(/[,&]/g); // 对字符串进行切割：, &
const result = {}; // 初始化最终的返回结果
```



### 3、遍历数组并返回结果

```js
queryArgs.forEach((arg) => {
  ...
};
return result;
```



### 4、key=value形式

```js
let name = arg.substr(0, idx); // 获取name
let value = decodeURIComponent(arg.substr(idx + 1)); // 获取value

// 将特殊字符串转成特殊字符
if(Object.prototype.hasOwnProperty.call(specialValues, value)) {
  value = specialValues[value]
}
```



**key[]=value**

`a[]=haha` => `resule: { a: ['haha'] }`

```js
if(name.substr(-2) === '[]') {
  name = decodeURIComponent(name.substr(0, name.length - 2));

  if(!Array.isArray(result[name])) {
    result[name] = [];
  }

  result[name].push(value);
}
```

**不是数组就直接赋值**

```js
name = decodeURIComponent(name);
result[name] = value;
```



### 5、单独字符串形式

`+a` => `result: { a: true }`

`-a` => `result: { a: false }`

`a` => `result: { a: true }`

```js
if(arg.substr(0, 1) === '-') { // -string
  result[decodeURIComponent(arg.substr(1))] = false;
} else if(arg.substr(0, 1) === '+') { // +string
  result[decodeURIComponent(arg.substr(1))] = true;
} else {
  result[decodeURIComponent(arg)] = true;
}
```



## 二、getOptions

使用方法：

```js
const options = loaderUtils.getOptions(this);
```

1. 如果 `this.query` 是字符串
   1. 尝试将字符串转换为一个 object
   2. 如果不是有效字符串，则抛出异常
2. 如果 `this.query` 是一个类对象，直接返回 `this.query`
3. 其他情况，直接返回空

源码如下：

```js
const parseQuery = require('./parseQuery');

function getOptions(loaderContext) {
  const query = loaderContext.query; // 使用时会传入this，这里就是 this.query

  if (typeof query === 'string' && query !== '') { // 如果是非空字符串，就使用parseQuery方法
    return parseQuery(loaderContext.query);
  }

  if (!query || typeof query !== 'object') { // 如果不是object，就返回空对象
    // Not object-like queries are not supported.
    return {};
  }

  return query;
}

module.exports = getOptions;
```



## 三、stringifyRequest

**作用**：将请求转换为字符串，使其可在 require 或 import 中使用，同时避免绝对路径

如果你在loader中生成代码，使用 `stringifyRequest` 替代 `JSON.stringify`

**Why？**

由于webpack在将模块路径转换为模块ID之前计算哈希，因此我们必须避免绝对路径，以确保不同编译之间的哈希一致。

- 如果请求和模块位于同一硬盘驱动器上，则将绝对路径解析为相对路径
- 如果请求和模块位于同一硬盘驱动器上，将 `\` 替换为 `/`
- 如果请求和模块位于不同的硬盘驱动器上，则不会更改路径
- 对结果使用  JSON.stringify



**示例：**

```js
loaderUtils.stringifyRequest(this, "./test.js");
// "\"./test.js\""

loaderUtils.stringifyRequest(this, ".\\test.js");
// "\"./test.js\""

loaderUtils.stringifyRequest(this, "test");
// "\"test\""

loaderUtils.stringifyRequest(this, "test/lib/index.js");
// "\"test/lib/index.js\""

loaderUtils.stringifyRequest(this, "otherLoader?andConfig!test?someConfig");
// "\"otherLoader?andConfig!test?someConfig\""

loaderUtils.stringifyRequest(this, require.resolve("test"));
// "\"../node_modules/some-loader/lib/test.js\""

loaderUtils.stringifyRequest(this, "C://module\\test.js");
// "\"../../test.js\""   =>  windows下，请求和module位于统一驱动下
// "\"C:\\nodule\\test.js\""    =>    windows下，请求和module位于不同驱动下

loaderUtils.stringifyRequest(this, "\\\\network-drive\\test.js");
// "\"\\\\network-drive\\\\test.js\""    =>  windows下，请求和module位于不同驱动
```





### 1、isAbsolutePath：是否是绝对路径

```js
function isAbsolutePath(str) {
  // 判断是否为绝对路径：unix或window环境
  return path.posix.isAbsolute(str) || path.win32.isAbsolute(str);
}
```

`posix` 是unix标准：全称“可移植操作系统接口”（Portable Operating System Interface of UNIX）

path 模块的默认操作会根据 Node.js 应用程序运行的操作系统的不同而变化。 比如，当运行在 Windows 操作系统上时，path 模块会认为使用的是 Windows 风格的路径。

- 在 posix 上：

```js
path.basename('C:\\temp\\myfile.html');
// 返回: 'C:\\temp\\myfile.html'
```

- 在 windows 上：

```js
path.basename('C:\\temp\\myfile.html');
// 返回: 'myfile.html'
```



要想在任何操作系统上处理 Windows 文件路径时获得一致的结果，可以使用  `path.win32`

- 在 posix 和 windows 上:

```js
path.win32.basename('C:\\temp\\myfile.html');
// 返回: 'myfile.html'
```



要想在任何操作系统上处理 POSIX 文件路径时获得一致的结果，可以使用  `path.posix`：

- 在 posix 和 windows 上:

```js
path.posix.basename('/tmp/myfile.html');
// 返回: 'myfile.html'
```



**path.isAbsolute(path)**

确定 `path` 是否为绝对路径。



### 2、isRelativePath：是否是相对路径

```js
// 匹配字符串：./a  .\\a  ../a  ..\\a
const matchRelativePath = /^\.\.?[/\\]/;

function isRelativePath(str) {
  return matchRelativePath.test(str);
}
```



### 3、stringifyRequest

```js
function stringifyRequest(loaderContext, request) {
  const splitted = request.split('!');
  // this.context：模块所在的目录。可以用作解析其他模块路径的上下文。
  const context = loaderContext.context || (loaderContext.options && loaderContext.options.context);

  return JSON.stringify(
    splitted
      .map(part => {
        // 首先，将单独路径与查询分开，因为查询可能再次包含路径
        const splittedPart = part.match(/^(.*?)(\?.*)/);
        const query = splittedPart ? splittedPart[2] : ''; // 拿到？和后面的内容（参数）
        let singlePath = splittedPart ? splittedPart[1] : part; // ？前面的内容（路径）

        if(isAbsolutePath(singlePath) && context) {
          // path.relative(from, to) 方法根据当前工作目录返回从 from 到 to 的相对路径
          singlePath = path.relative(context, singlePath);

          if(isAbsolutePath(singlePath)) {
            // 如果 singlePath 仍然匹配绝对路径，则 singlePath位于与上下文不同的驱动器上。
            // 在这种情况下，我们将其保留为“平台特定”，不更换任何分隔符
            return singlePath + query;
          }

          if(isRelativePath(singlePath) === false) {
            // 确保相对路径以 “./” 开头，否则它将进入模块目录查找（如node_modules）
            singlePath = './' + singlePath;
          }
        }

        return singlePath.replace(/\\/g, '/') + query;
      })
      .join('!')
  )
}
```





## 四、isUrlRequest

是否是有效的url请求

```js
const path = require('path');

function isUrlRequest(url, root) {
  // 如果出现以下情况，则 url 不是请求

  // 1.它是一个绝对 url，并且不是 windows 下的路径(如："C:/dir/file")
  if(/^[a-z][a-z0-9+.-]*:/i.test(url) && !path.win32.isAbsolute(url)) {
    return false;
  }

  // 2.它是一个相对协议： 以 // 开头
  if(/^\/\//.test(url)) {
    return false;
  }

  // 3.它像是某个模板的url
  if(/^[{}[\]#*;,'§$%&(=?`´^°<>]/.test(url)) {
    return false;
  }

  // 4.如果未设置 root，并且它是相对root的“相对请求”，则它也不是请求
  if((root === undefined || root === false) && /^\//.test(url)) {
    return false;
  }

  return true;
}

module.exports = isUrlRequest;
```



## 五、urlToRequest

将某些资源URL转换为webpack模块请求

- 在调用 `urlToRequest` 之前，你需要先调用 `isUrlRequest` 以确保它是可请求的 url

```js
const url = "path/to/module.js";

if(loaderUtils.isUrlRequest(url)) {
  // 可请求 url 的逻辑
  const request = loaderUtils.urlToRequest(url);
} else {
  // 不可请求 url 的逻辑
}
```

**示例：**

```js
const url = "path/to/module.js";
const request = loaderUtils.urlToRequest(url); // "./path/to/module.js"
```

**模块URL：**

任何包含 ~ 的URL都将被解释为模块请求。 ~ 之后的任何内容都将被视为请求路径。

```js
const url = "~path/to/module.js";
const request = loaderUtils.urlToRequest(url); // "path/to/module.js"
```

**相对于root的URL**

相对于root的URL（以 / 开头）可以使用 root 参数相对于某个任意路径解析。

```js
const url = "/path/to/module.js";
const root = "./root";
const request = loaderUtils.urlToRequest(url, root); // "./root/path/to/module.js"
```

将相对root的 url 转换为模块 url，需要指定以 ~ 开头的根值：

```js
const url = "/path/to/module.js";
const root = "~";
const request = loaderUtils.urlToRequest(url, root); // "path/to/module.js"
```



**源码：**

```js
const matchNativeWin32Path = /^[A-Z]:[/\\]|^\\\\/i;

function urlToRequest(url, root) {
  // 不要重写空的url
  if(url === "") {
    return "";
  }

  const moduleRequestRegex = /^[^?]*~/;
  let request;

  if(matchNativeWin32Path.test(url)) {
    // windows 绝对路径，保留
    request = url;
  } else if(root !== undefined && root !== false && /^\//.test(url)) {
    // 设置了root，并且url是相对于root的
    switch(typeof root) {
      // 1. root是字符串：root就是url的前缀
      case "string":
        // 特殊情况：root 是 ~ 的话，转换成模块请求
        if(moduleRequestRegex.test(root)) {
          request = root.replace(/([^~/])$/, "$1/") + url.splice(1);
        } else {
          request = root + url;
        }
        break;
      // 2. root为“true”，允许绝对路径
      case "boolean":
        request = url;
        break;
      default:
        throw new Error(
          "Unexpected parameters to loader-utils 'urlToRequest': url = " +
            url +
            ', root = ' +
            root +
            '.'
        )
    }
  } else if(/^\.\.?\//.test(url)) {
    // 相对路径保持不变
    request = url;
  } else {
    // 其他每个url都像相对url一样被线程化
    request = "./" + url;
  }

  // ~ 使url成为一个模块
  if(moduleRequestRegex.test(request)) {
    request = request.replace(moduleRequestRegex, '')
  }

  return request;
}

module.exports = urlToRequest;
```



## 六、getHashDigest

获取hash摘要

```js
const digestString = loaderUtils.getHashDigest(buffer, hashType, digestType, maxLength);
```

- `buffer`：内容需要被hash
- `hashType`：`sha1`, `md4`, `md5`, `sha256`, `sha512` 或者 其他 nodejs 支持的 hash 类型
- `digestType`：`hex`, `base26`, `base32`, `base36`, `base49`, `base52`, `base58`, `base62`, `base64`
- `maxLength`：字符的最大长度

















