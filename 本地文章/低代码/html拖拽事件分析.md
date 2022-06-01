# html拖拽事件分析

- 图片和链接默认支持拖拽
- 其他 `html` 元素添加 `draggable="true"` 属性即可支持拖拽
- 在页面拖拽元素时，会触发一系列事件



## dragstart

在draggable元素上按下鼠标, 开始移动时, 触发一次

event.traget 是draggable元素本身.



## drag

拖动draggable元素过程中时, 只要鼠标不松就一直触发.

event.traget 是draggable元素本身.



## dragend

释放draggable元素时触发一次.
event.traget 是draggable元素本身；

### 

## drop

释放draggable元素时触发一次, event.traget 是鼠标释放时指向的任何元素, 除了body.

必须添加了dragover 事件, 才会触发 drop 事件

为了能触发drop事件，需要在dragover事件中阻止默认事件才可生效

```js
target.addEventListener("dragover", (e) => {
  e.preventDefault()
})
```



## dragover

拖动时在某元素上持续触发



## dragenter 拖动时进入某元素范围 / dragleave 拖动时离开某元素范围

分别触发一次, event.traget 是鼠标下方指向的元素.



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

















