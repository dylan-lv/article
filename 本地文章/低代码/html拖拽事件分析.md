# html拖拽事件分析

- **图片和链接**默认支持拖拽，对于它们可以将 `draggable` 设置为 `false` 来防止拖拽
- 其他 `html` 元素添加 `draggable="true"` 属性即可支持拖拽。
  - 一旦某个元素设置了 `draggable = "true"`，就无法再用鼠标选中该节点内部的文字和子节点了。

- 在页面拖拽元素时，会触发一系列事件



## dragstart

- 在 `draggable` 元素上按下鼠标, **开始移动时**, 触发一次
- `event.traget`  是 `draggable` 元素本身
- 通常应该在这个事件的监听函数中，指定拖拽的数据（开始拖拽了，存好当前正在拖拽的元素以及相关的数据）



## drag

- 拖动 `draggable` 元素过程中，**持续触发**
- `event.traget` 是 `draggable` 元素本身.



## dragend

- 释放 `draggable` 元素时触发一次 （释放鼠标或按下 ESC 键）
- `event.traget` 是 `draggable` 元素本身；
- 它与 `dragstart` 是在同一个节点（当前被拖拽元素）上触发。
- 不管拖拽是否跨窗口，`dragend` 事件总是会被触发的



## drop

- 释放draggable元素时触发一次
- `event.traget` 是鼠标释放时指向的**目标节点**（任何元素, 除了body）
- 必须添加了dragover 事件, 才会触发 drop 事件
- 为了能触发drop事件，需要在dragover事件中阻止默认事件才可生效

```js
target.addEventListener("dragover", (e) => {
  e.preventDefault()
})
```

- 如果当前节点不允许 `drop`，那么即使在该节点上释放鼠标，也不会触发 `drop` 事件



## dragover

- 拖拽元素处于目标节点上时，**持续触发**
- `event.target` 是目标节点



## dragenter

- 拖动时进入某元素范围，触发依次
- `event.target` 是目标节点



##  dragleave 

- 拖动时离开某元素范围
- `event.target` 是目标节点



## 注意

- 拖拽过程中只会触发拖拽事件，不会触发鼠标事件
- 将文件从操作系统拖拽到浏览器内，不会触发 `dragstart` 和 `dragend` 事件
  - 这两个事件是**拖拽元素**的事件，浏览器无法监听到其自身之外的事件
- `dragenter` 和 `dragover` 事件一般用来获取拖拽元素，并允许其放下。
  - 由于网页的**大部分区域**不适合作为放下拖拉元素的目标节点，所以这两个事件的**默认**设置为**当前节点不允许接受被拖拉的元素**。
  - 如果想要在目标节点上放下的数据，首先必须阻止这两个事件的默认行为。

```html
<div id="box"></div>
<script>
	let box = document.querySelector("#box")
  box.addEventListener("dragover", (e) => {
    e.preventDefault()
  })
</script>
```



## 代码测试

```html
<div style="display: flex; height: 800px;">
  <div class="left" style="flex: 1; height: 100%;">
    <div draggable="true" id="box">box</div>
  </div>
  <div style="flex: 1; height: 100%;" id="board">22</div>
</div>
<script>
  let box = document.querySelector("#box")
  let board = document.querySelector("#board")
  box.addEventListener("dragstart", () => {
    console.log("dragstart");
  })
  box.addEventListener("drag", () => {
    // console.log("drag");
  })
  box.addEventListener("dragend", () => {
    console.log("dragend");
  })
  board.addEventListener("drop", () => {
    console.log("drop");
  }, false)
  board.addEventListener("dragover", (e) => {
    e.preventDefault()
    console.log("dragover");
  })
  board.addEventListener("dragenter", () => {
    console.log("dragenter");
  })
  board.addEventListener("dragleave", () => {
    console.log("dragleave");
  })
</script>
```

















