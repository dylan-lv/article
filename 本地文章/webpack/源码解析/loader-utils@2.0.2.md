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



## 六、getHashDigest（暂时不懂）

获取hash摘要

```js
const digestString = loaderUtils.getHashDigest(buffer, hashType, digestType, maxLength);
```

- `buffer`：内容需要被hash
- `hashType`：`sha1`, `md4`, `md5`, `sha256`, `sha512` 或者 其他 nodejs 支持的 hash 类型
- `digestType`：`hex`, `base26`, `base32`, `base36`, `base49`, `base52`, `base58`, `base62`, `base64`
- `maxLength`：字符的最大长度

```js
const baseEncodeTables = {
  26: 'abcdefghijklmnopqrstuvwxyz',
  32: '123456789abcdefghjkmnpqrstuvwxyz', // no 0lio
  36: '0123456789abcdefghijklmnopqrstuvwxyz',
  49: 'abcdefghijkmnopqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ', // no lIO
  52: 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ',
  58: '123456789abcdefghijkmnopqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ', // no 0lIO
  62: '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ',
  64: '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ-_',
};

function encodeBufferToBase(buffer, base) {
  const encodeTable = baseEncodeTables[base];
  if (!encodeTable) {
    throw new Error('Unknown encoding base' + base);
  }

  const readLength = buffer.length;
  const Big = require('big.js');

  Big.RM = Big.DP = 0;
  let b = new Big(0);

  for (let i = readLength - 1; i >= 0; i--) {
    b = b.times(256).plus(buffer[i]);
  }

  let output = '';
  while (b.gt(0)) {
    output = encodeTable[b.mod(base)] + output;
    b = b.div(base);
  }

  Big.DP = 20;
  Big.RM = 1;

  return output;
}

let createMd4 = undefined;

function getHashDigest(buffer, hashType, digestType, maxLength) {
  hashType = hashType || 'md4';
  maxLength = maxLength || 9999;

  let hash;

  try {
    hash = require('crypto').createHash(hashType);
  } catch (error) {
    if (error.code === 'ERR_OSSL_EVP_UNSUPPORTED' && hashType === 'md4') {
      if (createMd4 === undefined) {
        createMd4 = require('./hash/md4');
      }

      hash = createMd4();
    }

    if (!hash) {
      throw error;
    }
  }

  hash.update(buffer);

  if (
    digestType === 'base26' ||
    digestType === 'base32' ||
    digestType === 'base36' ||
    digestType === 'base49' ||
    digestType === 'base52' ||
    digestType === 'base58' ||
    digestType === 'base62' ||
    digestType === 'base64'
  ) {
    return encodeBufferToBase(hash.digest(), digestType.substr(4)).substr(
      0,
      maxLength
    );
  } else {
    return hash.digest(digestType || 'hex').substr(0, maxLength);
  }
}

module.exports = getHashDigest;
```



## 七、interpolateName

使用多个占位符或正则表达式插入文件名模板。模板和正则表达式在当前加载程序的上下文中设置名为 name 和 regExp 的查询参数。

```js
const interpoliatedName = loaderUtils.interpolateName(loaderContext, name, options);
```

在name参数中，以下标记会被替换

- `[ext]`：资源的扩展名
- `[name]`：资源的基础名称
- `[path]`：资源相较于上下文的相对路径或选项的路径
- `[folder]`：资源所在的文件夹
- `[query]`：参数，例如： ?foo=bar
- `[emoji]`：`options.content`中的随机emoji表情
- `[emoji:<length>]`：与上面相同，但具有可定制数量的表情符号
- `[contenthash]`：内容的hash（默认情况下是md4哈希的16进制摘要）
- `[<hashType>:contenthash:<digestType>:<length>]`：可以选择配置
  - `hashType`：sha1、md4、md5、sha256、sha512
  - `digestType`：hex、base26、base32、base36、base49、base52、base58、base62、base64
  - `length`：字符的长度
- `[hash]`：选项的hash。
- `[hashType]:hash:<digestType>:<length>`：选项配置，同上
- `[N]`：正则表达式。通过将当前文件名与选项匹配而获得的第N个匹配项。



**在 loader 上下文中，[hash]和[contenthash] 是相同的，但我们建议使用 [contenthash] 以避免误导**

- 使用示例

```js
// loaderContext.resourcePath = "/absolute/path/to/app/js/javascript.js"
loaderUtils.interpolateName(loaderContext, "js/[hash].script.ext", { content: ... });
// => js/9473fdd0d880a43c21b7778d34872157.script.js

// loaderContext.resourcePath = "/absolute/path/to/app/js/javascript.js"
// loaderContext.resourceQuery = "?foo=bar"
loaderUtils.interpolateName(loaderContext, "js/[hash].script.[ext][query]", { content: ... });
// => js/9473fdd0d880a43c21b7778d34872157.script.js?foo=bar

// loaderContext.resourcePath = "/absolute/path/to/app/js/javascript.js"
loaderUtils.interpolateName(loaderContext, "js/[contenthash].script.[ext]", { content: ... });
// => js/9473fdd0d880a43c21b7778d34872157.script.js

// loaderContext.resourcePath = "/absolute/path/to/app/page.html"
loaderUtils.interpolateName(loaderContext, "html-[hash:6].html", { content: ... });
// => html-9473fd.html

// loaderContext.resourcePath = "/absolute/path/to/app/flash.txt"
loaderUtils.interpolateName(loaderContext, "[hash]", { content: ... });
// => c31e9820c001c9c4a86bce33ce43b679

// loaderContext.resourcePath = "/absolute/path/to/app/img/image.gif"
loaderUtils.interpolateName(loaderContext, "[emoji]", { content: ... });
// => 👍

// loaderContext.resourcePath = "/absolute/path/to/app/img/image.gif"
loaderUtils.interpolateName(loaderContext, "[emoji:4]", { content: ... });
// => 🙍🏢📤🐝

// loaderContext.resourcePath = "/absolute/path/to/app/img/image.png"
loaderUtils.interpolateName(loaderContext, "[sha512:hash:base64:7].[ext]", { content: ... });
// => 2BKDTjl.png
// use sha512 hash instead of md4 and with only 7 chars of base64

// loaderContext.resourcePath = "/absolute/path/to/app/img/myself.png"
// loaderContext.query.name =
loaderUtils.interpolateName(loaderContext, "picture.png");
// => picture.png

// loaderContext.resourcePath = "/absolute/path/to/app/dir/file.png"
loaderUtils.interpolateName(loaderContext, "[path][name].[ext]?[hash]", { content: ... });
// => /app/dir/file.png?9473fdd0d880a43c21b7778d34872157

// loaderContext.resourcePath = "/absolute/path/to/app/js/page-home.js"
loaderUtils.interpolateName(loaderContext, "script-[1].[ext]", { regExp: "page-(.*)\\.js", content: ... });
// => script-home.js

// loaderContext.resourcePath = "/absolute/path/to/app/js/javascript.js"
// loaderContext.resourceQuery = "?foo=bar"
loaderUtils.interpolateName(
  loaderContext, 
  (resourcePath, resourceQuery) => { 
    // resourcePath - `/app/js/javascript.js`
    // resourceQuery - `?foo=bar`

    return "js/[hash].script.[ext]"; 
  }, 
  { content: ... }
);
// => js/9473fdd0d880a43c21b7778d34872157.script.js
```

源码：

```js
const path = require('path');
const emojisList = require('emojis-list');
const getHashDigest = require('./getHashDigest');

const emojiRegex = /[\uD800-\uDFFF]./;
const emojiList = emojisList.filter(emoji => emojiRegex.test(emoji));
const emojiCache = {};

function encodeStringToEmoji(content, length) {
  if(emojiCache[content]) {
    return emojiCache[content];
  }

  length = length || 1;

  const emojis = [];

  do {
    if(!emojiList.length) {
      throw new Error('Ran out of emoji');
    }

    const index = Math.floor(Math.random() * emojiList.length);

    emojis.push(emojisList[index]);
    emojisList.splice(index, 1);
  } while (--length > 0)

  const emojiEncoding = emojis.join('');

  emojiCache[content] = emojiEncoding;

  return emojiEncoding;
}

function interpolateName(loaderContext, name, options) {
  let filename;

  const hasQuery = loaderContext.resourceQuery && loaderContext.resourceQuery.length > 1;

  if(typeof name === 'function') {
    filename = name(
      loaderContext.resourcePath,
      hasQuery ? loaderContext.resourceQuery : undefined
    );
  } else {
    filename = name || '[hash].[ext]';
  }

  const context = options.context;
  const content = options.content;
  const regExp = options.regExp;

  let ext = 'bin';
  let basename = 'file';
  let directory = '';
  let folder = '';
  let query = '';

  if(loaderContext.resourcePath) {
    let resourcePath = loaderContext.resourcePath;
    // path.parse：返回一个对象，其属性表示 path 的重要元素。
    const parsed = path.parse(resourcePath);
		// 处理后缀
    if(parsed.ext) {
      ext = parsed.ext.substring(1);
    }
		// 处理文件夹路径
    if(parsed.dir) {
      basename = parsed.name;
      // path.sep：提供特定于平台的路径片段分隔符（windows \）（posix /）
      resourcePath = parsed.dir + path.sep;
    }

    if(typeof context !== 'undefined') {
      directory = path
        .relative(context, resourcePath + '_')
        .replace(/\\/g, '/')
        .replace(/\.\.(\/)?/g, '_$1');
      directory = directory.substring(0, directory.length - 1);
    } else {
      directory = resourcePath.replace(/\\/g, '/').replace(/\.\.(\/)?/g, '_$1');
    }

    if(directory.length === 1) {
      directory = '';
    } else if(directory.length > 1) {
      folder = path.basename(directory);
    }
  }

  if(loaderContext.resourceQuery && loaderContext.resourceQuery.length > 1) {
    query = loaderContext.resourceQuery;

    const hashIdx = query.indexOf('#');

    if(hashIdx >= 0) {
      query = query.substring(0, hashIdx);
    }
  }

  let url = filename;

  if(content) {
    // hash 和 contenthash在loader-utils上下文中是相同的
    // 为了向后兼容，让我们保留hash
    url = url
      .replace(
        /\[(?:([^:\]]+):)?(?:hash|contenthash)(?::([a-z]+\d*))?(?::(\d+))?\]/gi,
        (all, hashType, digestType, maxLength) => getHashDigest(content, hashType, digestType, parseInt(maxLength, 10))
      )
      .replace(/\[emoji(?::(\d+))?\]/gi, (all, length) => encodeStringToEmoji(content, parseInt(length, 10)));
  }

  url = url
    .replace(/\[ext\]/gi, () => ext)
    .replace(/\[name\]/gi, () => basename)
    .replace(/\[path\]/gi, () => directory)
    .replace(/\[folder\]/gi, () => folder)
    .replace(/\[query\]/gi, () => query)
  
  if(regExp && loaderContext.resourcePath) {
    const match = loaderContext.resourcePath.match(new RegExp(regExp));

    match &&
      match.forEach((matched, i) => {
        url = url.replace(new RegExp('\\[' + i + '\\]', 'ig'), matched);
      });
  }

  if(typeof loaderContext.options === 'object' && typeof loaderContext.options.customInterpolateName === 'function') {
    url = loaderContext.options.customInterpolateName.call(
      loaderContext,
      url,
      name,
      options
    )
  }
  
  return url;
}

module.exports = interpolateName;
```































