# 组件化

> 前端常见设计模式



用组件来**搭建**界面，组件是最小的界面元素。

### 按照最新的前端对组件理解，在组件化当中，一个组件包括：

- 视图（View）
- 数据（Data）：props，state
- 视图到数据的映射（view = f(data)）
- 组件的作用（effect）：根据数据渲染视图（view = f（data））之外的程序



### 组件可以被：

- 映射、变换
  - view = f(data)
  - view = f - g(data)
  - view = data => data.map(...).map(...).fillter(...)
- 组合（Composition）
- 记忆（Memorization）
  - 记忆是一种作用
- 列表（List）



### 组件有这些性质：

- 密封性（sealed）

  - 组件专注、完整

- 可预测性

  - view = f(data) with effects(...)

- 连续性（continuations）

  - a + b + c + d = a + (b + c + d)

    组件的渲染的先后顺序不影响组件渲染的结果

    -- 组件和并发渲染

    -- 组件可以和控制流（if/while/for）无缝结合

  - 每个组件是一个函数调用，是一个任务，它们没有**特殊性**。



### 组件的粒度

组件应该具有最小粒度。

实现组件列表当中，实际上应该实现两个组件：

- 业务无关：TreeView
- 业务相关：UILayerView

UILayerView = TreeView + useUILayerView



按照最新的组件化理解，通常我们会将组件分成：

- 基础组件（用于实现交互）
  - Draggable
  - Selectable
  - Button
  - ......
- 组合组件（在基础组件上组合实现更复杂的交互）
- 业务组件 = （基础组件 | 组合组件） + useXXX



### 组件间通信

**选项A：EventBus模型**

组件可以：

- 单播
- 广播

常见案例：

- Iframe中多个APP间通信
- Iframe中多个APP和Frame通信
- Native和HybridApp间postMessage通信



**选项B：单向数据流 + 状态机模型**

场景举例：

- UI交互制作
- 全局事件通知（例如加购物车、用户消息等）



**选项C：领域模型 + Emiter**

举例：

- 组件仅仅负责渲染等简单工作，背后的业务逻辑由复杂的领域模型完成











