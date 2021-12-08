# 装饰器（Decorator）

在不改变对象、函数结构的情况下为它添加功能或说明。

**举例1：@deprecated（使用后，在vscode中会出现删除线，表示这个属性即将被弃用）**

```tsx
interface UIInfo {
  /** @deprecated use box instead **/
  width : number;
  /** @deprecated use box instead **/
  height : number;
  
  box : BoxDescriptor
}
```

这里原先的代码是width和height，但是之后发现应该放进box中，所以之前的两个属性即将被弃用。



**举例2：禁止删除/修改某些属性**

```tsx
class Point {
  private _x : number;
  private _y : number;
  constructor(x : number, y : number) {
    this._x = x;
    this._y = y;
  }
  
  @configurable(false)
  get x() {
    return this._x;
  }
  
  @configurable(false)
  get y() {
    return this._y;
  }
}

const p = new Point(1, 1)
delete p.x // 报错
p.x = function () {...} // 报错
```



## 主要作用

- 替换原有实现
- 修改元数据

