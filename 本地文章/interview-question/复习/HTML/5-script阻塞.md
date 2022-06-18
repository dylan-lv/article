## Script放在body或者header中会阻塞吗？

默认 `script` 标签是同步执行的，会发生阻塞。

**原因：**

- 浏览器解析 html 文件时，是从上至下解析的
- 解析到 DOM 中的 script 时，会暂停 DOM 的构建
- 脚本加载并执行完毕，才会继续向下解析



JS 脚本存在会**阻塞 DOM 解析**进而影响页面渲染速度。

```html
<body>
  <p>111</p>
  <script src=".../index.js?delay=1000"></script>
  <p>222</p>
  <p>333</p>
</body>
```

上面的代码将 `script` 标签放在了其他标签的中间，我们假设 `script` 加载的内容会在 1 秒后返回，则首屏渲染的效果如下：

- 直接渲染处 `111`
- 向服务端请求 `script` 脚本
- 1 秒后，脚本回来，执行脚本
- 渲染 `222`、`333` 这两个标签



### async

如果我们不想让 `script` 阻塞，需要将其变成异步执行的，加上 async 标记

```html
<body>
  <p>111</p>
  <script src=".../index.js?delay=1000" async="async"></script>
  <p>222</p>
  <p>333</p>
</body>
```



### defer

defer 也是异步执行，它实际上和 async 很像，

```html
<body>
  <p>111</p>
  <script src=".../index.js?delay=1000" defer="defer"></script>
  <p>222</p>
  <p>333</p>
</body>
```



### 区别

- `async` ：它会在 js 资源被下载完毕之后立刻执行，会在 `window` 的 `onLoad` 事件之前执行，多个`async`的 `script` 脚本，有可能出现打乱顺序的情况
- `defer`：它会在页面解析完毕之后，按照原本的顺序执行，在 `DOMContentLoaded` 事件触发之前执行。









