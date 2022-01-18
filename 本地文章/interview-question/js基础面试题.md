### 1、控制台输出当前页面有多少种html标签

```js
new Set([...document.querySelectorAll('*')].map(node => node.nodeName)).size
```



### 2、统计当前页面出现次数最多的 3 种 HTML 标签

```js
console.table(
Object.entries([...document.querySelectorAll('*')]
    .map(node => node.nodeName)
    .reduce((pre, cur) => {
        pre[cur] = (pre[cur] || 0) + 1;
        return pre;
    }, {})).sort((a, b) => b[1] - a[1]).slice(0, 3)
)
```









