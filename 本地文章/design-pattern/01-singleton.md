# 单例（singleton）



确保一个类只有一个实例。

window、document也都算单例。



设计模式关注的是**设计目标**，并不是对设计实现的**强制约束**。闭包也可以实现单例，例如：

```js
const signleton = () => {
  const obj = new ...
  return () => {
    ...
  }
}
```

划重点：理解设计模式，灵活使用设计模式



典型的单例：

```js
class IDGen {
  private constructor(){}
  static instance = new IDGen()
  static get() { return instance }
}
```





## 总结

- 可以用于配置类、组件上下文中共用的类等
- 用于对繁重资源的管理（例如数据库连接池）

