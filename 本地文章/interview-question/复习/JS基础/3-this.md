# this 共有几种情况

判断 `this`，一定不要看在哪里定义。一定只看将来在哪里以及如何被调用。

1. `obj.fun()`，`this`指向 `.` 前的 `obj` 对象
2. `new` 构造函数，`this` 指向 `new` 正在创建的新对象
3. 构造函数 `.prototype.fun = function(){}`
   1. 因为将来原型对象中的方法，都是 `子对象.fun()` 的方式调用。所以 `this` 指向将来调用这个 `fun` 函数的 `.` 前的某个子对象。
4. `fun()`（**普通函数**调用）、**匿名函数自调用**和**回调函数**中的 `this` 指向 `window`。严格模式（usestrict）下，`this` 指向 `undefined`
5. DOM 事件处理函数里的 `this` 指向当前正在触发事件的 `.` 前的 DOM元素对象
   1. `button.onclick = function(){}`
   2. `button.addEventListener("click", function(){})`
   3. 注意：这里不能改成箭头函数，一旦改为箭头函数，`this` 就指向外层的 `window`
6. 箭头函数中的 `this` 指向当前函数之外最近的作用域中的 `this`
   1. 几乎所有匿名函数都可用箭头函数简化
   2. 箭头函数是对大多数匿名函数的简写



### 箭头函数修改this

先来看下面这个例子：

```js
var lilei = {
  name: "Li Lei",
  friends: ["laowang", "laizhang", "laoli"],
  info: function() {
    this.friends.forEach(function(item) {
      console.log(`${this.name} 认识 ${item}`)
    })
  }
}
lilei.info()
```

这段程序的输出结果：

```js
  认识 laowang
  认识 laizhang
  认识 laoli
```

可以看到并没有打印出 `this.name`，这是因为 `forEach` 中是一个**回调函数**，回调函数的 `this` 是指向 `window` 的。

而 `info` 函数中的 `this.friends` 为什么可以拿到呢？

这是因为我们调用 `info` 函数的时候，是通过 `lilei.info` 调用的，此时 `this` 会指向 `.` 前面的对象。

那么我们现在来用**箭头函数**修改上述的代码：

```js
var lilei = {
  name: "Li Lei",
  friends: ["laowang", "laizhang", "laoli"],
  info: function() {
    this.friends.forEach((item) => {
      console.log(`${this.name} 认识 ${item}`)
    })
  }
}
lilei.info()
```

此时打印的结果为：

```js
Li Lei 认识 laowang
Li Lei 认识 laizhang
Li Lei 认识 laoli
```

箭头函数其中的 `this` 会指向当前函数作用域之外**最近的** `this`



那么，`info` 函数能改成箭头函数吗？

根据箭头函数 `this` 指向外层的原理，我们会发现函数中的 `this.friends` 相当于 `window.friends` 了，此时就会出错。



### 将普通函数修改为箭头函数的效果（bind）

```js
var lilei = {
  name: "Li Lei",
  friends: ["laowang", "laizhang", "laoli"],
  info: function() {
    this.friends.forEach(
      function(item) {
      	console.log(`${this.name} 认识 ${item}`)
    	}.bind(this)
    )
  }
}
lilei.info()
```

箭头函数底层相当于 `.bind()`

所以，`call` 无法替换箭头函数中的 `this`。因为 `bind` 是永久绑定的（不可替代）

> `call` 和 `apply` 是临时替换一次函数中的 `this`
>
> `bind` 是永久替换函数中的 `this`



### 什么情况下使用箭头函数

- 可以改为箭头函数的情况
  - 如果函数中不包含 `this`
  - 希望函数内的 `this` 与函数外的 `this` 保持一致
- 不可改为箭头函数的情况
  - 不希望函数内的 `this` 与函数外的 `this` 保持一致
  - js事件的处理函数不能改

















