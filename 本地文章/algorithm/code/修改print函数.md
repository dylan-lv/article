## 修改以下print函数，使之输出0到99，或者99到0

要求：

1. 只能修改 `setTimeout` 到 `Math.floor(Math.random() * 1000)` 的代码
2. 不能修改 `Math.floor(Math.random() * 1000)`
3. 不能使用全局变量

```js
function print(n) {
  setTimeout(() => {
    console.log(n)
  }, Math.floor(Math.random() * 1000))
}
for(let var i=0; i<100; i++) {
  print(i);
}
```



### 方法一

利用 `setTimeout`、`setInterval`的第三个参数，第三个以后的参数是作为第一个 `func()`的参数传进去

```js
function print(n) {
  setTimeout(() => {
    console.log(n)
  }, 1, Math.floor(Math.random() * 1000))
}
for(let var i=0; i<100; i++) {
  print(i);
}
```



### 方法二

修改`setTimeout`第一个函数参数

```js
function print(n) {
  setTimeout((() => {
    console.log(n)
    return () => {}
  }).call(n), Math.floor(Math.random() * 1000))
}
for(let var i=0; i<100; i++) {
  print(i);
}
```



### 方法三

利用异步函数

```js
function print(n) {
  setTimeout(async () => {
    await console.log(n)
  }, Math.floor(Math.random() * 1000))
}
for(let var i=0; i<100; i++) {
  print(i);
}
```



