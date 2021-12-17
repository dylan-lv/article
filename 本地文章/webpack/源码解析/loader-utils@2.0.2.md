# loader-utils@2.0.2

<a href="https://github.com/webpack/loader-utils/tree/v2.0.2" target="_blank">loader-utils@2.0.2</a>

以下内容是 loader-utils@2.0.2 的源码及解析

## 一、parseQuery

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
  //! 注：像是处理类似url的字符串
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

      //! 注：处理name为 'key[]'，这种类型的字符串
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





















