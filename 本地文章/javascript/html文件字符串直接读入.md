```js
var newWin = window.open('', '_blank')
newWin.document.write(localStorage.getItem('callbackHTML'))
// 关闭输出流
newWin.document.close()
```

