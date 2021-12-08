# 低代码（low-code）

首先我们了解一下什么是低代码。

> 低代码（Low-Code Development，LCD）开发者主要通过图形化用户界面和配置来创建应用软件，而不是像传统模式那样主要依靠手写代码。低代码开发模式的开发者，通常不需要具备非常专业的编码技能，或者不需要某一专门领域的编码技能，而是可以通过平台的功能和约束来实现专业代码的产出。

​		实际上，一个可用的低代码平台的搭建并非难事。首先，我们需要明确，低代码平台解决的最大问题是复用，复用也是目前前端开发中的一个重要课题，特别是当前的主流前端js框架，例如 **vue**，**react** 等，都是组件化的开发方式，又如形形色色的ui组件库的出现，像 **ant-design**， **element** 等都是来解决重复造轮子的问题。



## 简易版的低代码平台

​		我们在前端开发脚手架中，通常会创建一些通用的组件，然后在各个需要这个组件的地方进行引用，来提示开发效率。在脚手架中引用一些ui组件库也是出于这样的目的。后来，为了防止重复的造轮子，我们通常会对一些成熟的ui组件库中组件根据我们的业务需要进行二次封装，形成一个具有更多功能的区块，例如B端系统中最为常见的列表数据增删改查界面就是这样一个区块。

<img src=".\assets\low-code-table.png" alt="low-code-table" />



​		我们可以通过一个json数据来描述这个区块，数据大概是下图所示的样子，包含了这个区块的编码，中文名称，以及可传入到这个区块中的属性及初始属性值，例如这个表格包含的列信息、按钮信息等等。而这个json也是低代码平台搭建中最为核心的部分，它在后续介绍的可视化拖拉拽页面设计中扮演极其重要的角色，它是页面设计和页面渲染间串联的纽带。

```json
{
  code: "SearchTable",
  name: "搜索表格",
  props: {
    columns: [
      {
        name: "name",
        desc: "姓名",
        formType: "input",
        isSearch: true
      },
      {
        name: "age",
        desc: "年龄",
        formType: "numberInput"
      }
    ],
    buttons: [
      {
        name: "add",
        desc: "新增"
      }
    ]
  }
}
```

​		当我们展示类似的界面时，只需要在页面文件中引用这个搜索表格模板组件，再把对应的描述json信息传递到组件内就可以了。但是这样似乎还是采用了编码的方式去解决复用问题，距离我们的低代码还有些距离。

​		这时，我们可以将这些可动态控制的组件属性通过在线表单进行填写，存储到数据库中。前端工程中，我们可以进行判断，将使用这个区块模板的页面路由所对应的组件都指向我们封装好的列表增删改查模板。这样，进入到这个页面，我们只需要将之前存储到数据库的区块描述信息通过api接口获取到，再传递到组件内部，将json数据中我们设置的模板属性值赋值到组成模板的各个组件上就可以完成这个模板的渲染了。这也是我们在实现可视化拖拉拽低代码平台之前所使用的方式。这种方式虽然形式上非常简单，却解决了我们组件复用的问题，并且在调整组件属性时只需要通过在线的表单配置，无需打包、发版。这个界面渲染的流程便是低代码平台的核心逻辑：通过组件元数据拼装成一个页面的描述信息，然后通过渲染器组件将描述信息转化页面dom元素。

<img src=".\assets\low-code-table1.png" alt="low-code-table" />

我们上面实现的流程除了图形化的配置外，基本可以通过在平台上界面的配置实现一个搜索表格页面的开发了，并且不需要专业的编码技能。

当然，上面的过程中所使用到的模板实际是我们预设好的，只是通过我们传递进去固定属性的不同属性值进行一些差异化的渲染，一些按钮等自定义功能的扩展也只能通过插槽等形式去实现。但当我们想要实现其它模板的界面时，就显得无能为力了，因为组成模板的组件是我们提前预设好的，并不是通过组件像搭积木一样灵活搭建的。



## 页面设计器

<img src=".\assets\lowcode1.png" alt="low-code-table" />

可以看到，大多数页面设计器都包含了如下所示的几个区域：

- 最上方是**操作栏**，我们可进行页面的保存、预览、查看json信息、查看代码等操作；
- 左侧是**组件列表**，当然也可以添加一些切换，让我们的左侧区域支持查看页面树信息、配置数据源等其他操作；
- 中间是**画布**区域，我们可以将左侧的组件拖动到画布中，当然也支持画布中组件的赋值、删除等操作；
- 右侧是**属性配置**区域，当我们在画布中选中某个组件时，可以在右侧的属性配置区域罗列出当前组件可支持动态配置的属性，修改了属性后可以在画布中看到对应组件的样式变化。

下面，我们按上述的区域划分来看一下页面设计器区域都是如何实现的。



### 组件列表

首先，我们来看一下左侧的组件列表，列表中的每个组件，我们都需要使用一段`json`来进行描述，这段`json`我们将它称之为 **元数据**，元数据中描述了当前组件的中文名称，在列表中显示的图标及描述，和组件可进行配置的一些动态属性。我们以输入框组件为例，它的元数据大致可以定义为如下的样子：

```json
{
  code: "MyInput",
  name: "输入框",
  desc: "输入框的描述",
  icon: "input",
  props: {
    name: "字段名称",
    label: "label名称",
    labelCol: "",
    wrapperCol: "",
    required: false
  }
}
```

那么左侧的组件列表实际上就是这样的一个元数据对象组成的数组遍历而来的。



### 拖动

再来看一下将左侧组件列表的组件拖动到画布是如何实现的。拖动又分为顺序排列布局的拖动及自由布局拖动。顺序排列布局的拖动是指拖动到画布中的组件是自上而下顺序排列的，可以通过拖动调整上下顺序，当然我们也可以增加分栏这样的布局类型组件，实现组件的左右排列；自由布局拖动是指拖动到画布中的组件位置是自由的，我们松开鼠标的位置，就是这个组件在画布中的位置。考虑到我们主要服务的是B端项目，需要尽可能的使用户体验保持一致，这里呢我们采用的是顺序排列布局的拖动。这样用户拖动设计出的页面差异性不会太大，页面布局上又相对规整。



#### 将画布中的组件列表渲染为真实的组件

我们知道，画布中的列表实际也是通过组件元数据数组进行渲染的，而每个元数据项都对应了一个真实的组件，这样我们只需要将元数据项替换成UI组件进行渲染就可以了。在代码中大致是如下的样子：

```html
<template>
	<div v-for="item in list" :key="item.id">
    <my-input v-if="item.code === 'MyInput'" :data="item" />
    <my-select v-if="item.code === 'MySelect'" :data="item" />
    ...
  </div>
</template>
```

很明显，这样写不太优雅，我们用动态组件优化一下。

```html
<template>
	<div v-for="item in list" :key="item.id">
    <component :data="item" :is="item.code" />
  </div>
</template>
```



### 嵌套组件

通过一些优秀低代码产品，我们可以发现，他们组件列表都是进行分类显示的，布局类组件就是这样一类可以在组件内部再进行拖动的组件。这类组件包括栅格组件、容器组件、多页签组件、卡片组件等。我们知道，`list`就是最终页面渲染的组件列表，它是一个对象数组的数据结构，为了让它支持嵌套组件，我们需要在组件的元数据对象上增加一个属性，这个属性用来描述该组件下又嵌套了哪些组件，我们就命名这个属性为`children`。那么，包含嵌套组件的页面数据大致就是下面所示的样子。

```json
[{
  code: "MyCard",
  name: "卡片",
  props: {
    ...卡片组件相关配置属性
  },
  children: [{
    code: "MyContainer",
    name: "容器",
    props: {
      ...容器组件相关配置属性
    },
    children: [{
      ...
    }]
  }]
}]
```



### 属性配置区域

我们可以对画布中的组件添加点击事件，当点击某个组件时，我们能够获取到当前点击组件的组件类型，例如输入框、下拉选择等等，针对每一种组件，我们已经提前在元数据中的`props`属性定义了这个组件能够进行动态控制的参数，我们只需要将这些参数以合适的表单形式展示在右侧的属性配置区域就可以了，例如，按钮组件的`props`中有一个`text`属性，用来控制按钮的显示文案，那么我们就在右侧属性控制区域用一个输入框来做为控制这个属性的表单形式，当修改数据时，我们找到`list`中该组件所在的元数据对象，然后将该对象中`props`属性中`text`属性值修改为输入框中的内容。每个组件都会接受这个组件对应的元数据`props`参数，然后根据参数值进行渲染。例如按钮组件，现在按钮的文案时，我们可以使用`props.text`进行显示。



### 操作栏区域

从上面的文章中可以看出，一个页面实际就是用一段带有层级结构的`json`来进行描述的。



#### 保存

保存时实际就是将这段`json`进行保存操作，我们可以将`json`存储到数据库中。



#### 预览

在上面讲解画布区域时，我们已讲到组件如何通过json进行渲染。

预览以及真实的页面渲染实际和画布中组件的展示实现原理完全一致。

其中的区别有两点：

（1）画布中的组件不支持交互操作，这里，我们需要屏蔽画布中组件的交互操作。我们可以通过`css`中的`after`伪类，设置`content`为`""`来实现。

（2）画布中的组件需要包裹一个`div`，这个`div`包含了复制、删除等功能。



## 组件间交互的实现（事件）



### 事件的分类

我们主要依托**事件**来进行组件间的交互。为了满足组件与组件、组件与系统、组件与服务器的交互，我们大致可以将事件分为三个类别：

1. **组件方法**：每个组件都会暴露出一些方法供其他组件进行调用。例如表格组件，我们可以暴露查询表格数据方法；弹窗组件，可以暴露打开、关闭方法。
2. **系统动作**：有些方法并不属于某个组件，而是系统全局的动作，例如路由的跳转、全局的消息提示、页面加载动画的显隐等等。
3. **API调用**：这一类主要用于处理组件与服务端的交互，也就是服务端api接口的调用。



### 事件的定义

对于组件方法，我们需要将组件暴露的方法定义在各个组件的`methods`中，并在组件加载时进行方法的监听。

```js
window.vm.$on(`${id}-${methodName}`, params => {
  // TODO
})
```

在组件销毁时取消监听

```js
window.vm.$off(`${id}-${methodName}`)
```

做好事件的监听后，我们就可以使用`emit`关键字触发事件了。

```js
componentMethod(id, methodName, params) {
  window.vm.$emit(`${id}-${methodName}`, params);
}
```

在事件配置的时候只需要调用`methods`中的`componentMethod`方法就可以了。



### 事件的配置

设计器中的每个组件都对应了一段`json`元数据，我们改变某个组件的属性时，会到设计器属性配置区域进行表单值的修改，从而修改整个页面`json`中该组件所对应的这一段`json`，同理，当我们进行某个组件的事件配置时，也会对组件的事件属性进行赋值，我们将这个事件属性定义为`event`。属性值为一段`js`函数字符串，如下所示

```json
{
  code: "MyButton",
  name: "按钮",
  props: {
    type: "default",
    size: "small",
    ...
    event: "function action(ctx){\n console.log(\"执行动作\") }\n"
  }
}
```

这样，我们执行事件时，只需要将函数字符串转换为函数进行调用就可以了。



### 事件的执行

上面讲到，我们将事件配置成了一段函数字符串，之后可以使用`eval`或者`Function`去执行，这里我们选择使用`Function`。

首先我们定义一个字符串函数解析方法`funcStrParse`。

```js
function funcStrParse(funcStr) {
  return Function(`use strict; return (${funcStr})`)
}
```

这个方法的参数是一段函数字符串，返回值是一个函数。执行时，我们只需要调用`funcStrParse`返回的函数即可。

```js
funcStrParse("这是一段函数字符串")(ctx);
```

我们执行这段函数时，传入了一个`ctx`参数，实际上，这个`ctx`参数就是`vue`组件中的`this`。上文中我们讲到，我们会在`vue`组件中的`methods`中定义一些系统方法（通过mixin引用）和组件方法。这样，我们在代码编辑器中编写事件时，就可以通过`ctx`变量调用`vue`组件`methods`中的方法了。



### 事件的参数

我们已经可以通过配置去调用组件中的方法了，那么我们如何获取到其它组件中的参数呢？我们知道，组件参数的传递方式有很多，最基本的就是通过`props`属性或者通过`project/inject`进行传递。考虑到设计器中的组件繁多，并且每个组件都有独特的互不相同的组件参数，这样会导致组件标签中传递的属性过多，并且取用十分不方便。我们需要将各个组件暴露的参数放到一个公共的地方，这样在使用的时候就非常方便了。这个公共的地方定义到`vuex`中的`store`中还是比较合适的。我们以页面为单位注册`store`的`module`，将所有的组件变量都存到所在页面`module`中`state`的`pageParams`对象变量中。如下图所示。其中`id`就是组件的`id`，也就是在`pageParams`变量中，我们以组件的`id`为`key`，以组件的参数集合对象为`value`。这样，我们就可以直接在事件配置中进行使用了。

```json
{
  id1: {
    paramName1: paramValue1,
    paramName2: paramValue2,
    paramName3: paramValue3
  },
  id2: {
    paramName1: paramValue1,
    paramName2: paramValue2,
    paramName3: paramValue3
  }
}
```



## 组件拖动自由布局的实现



### 如何使元素支持拖动

实现组件的自由拖动的核心就是 `html5` 中新添加的全局属性 `draggable` 属性，该属性规定了元素是否可进行拖动。属性值如下所示：

- **true**：规定元素是可拖动的
- **false**：规定元素不可拖动
- **auto**：使用浏览器的默认行为

当我们在元素元素标签中添加 `draggable` 属性时，该元素就可以进行拖动操作了。

```html
<div draggable></div>
```



### 拖动事件

#### 事件分类

元素可以进行拖动了，我们就可以通过元素的拖动事件进行拖动开始-结束的一些逻辑控制了，拖动事件主要分为两个类别，一类是拖动元素可以触发的：

- **dragstart**：鼠标点中元素并且开始移动时触发
- **drag**：拖拽过程中持续触发
- **dragend**：拖拽结束松开鼠标时触发

另一类是，是当拖拽元素到某个目标元素时，目标元素会触发的：

- **dragenter**：拖拽元素到目标上时触发
- **dragover**：拖动元素在目标元素中，持续触发
- **dragleave**：离开目标元素时触发
- **drop**：拖放元素到了目标元素中松开鼠标时触发



#### 拖动放置行为

在拖动事件中，我们会获取到拖动的事件对象 `(e)`，在拖动对象中我们能获取到一个重要的属性 `dataTransfer` ，我们可以通过 `dataTransfer` 的 `dropEffect` 属性控制被拖动的元素的放置行为，其值的说明如下所示。

- **none**：不能把元素拖放至此
- **move**：移动到目标
- **copy**：复制到目标
- **link**：目标打开拖动元素（拖动元素必须是链接并有URL）



## 页面设计器的实现

下面我们根据以上的知识点来实现一下页面设计器组件拖动的最简demo。 首先我们定义一下组件列表和画布区域

```html
<template>
  <div>
    <!-- 左侧组件列表 -->
    <div class="left">
      <div class="left-item" v-for="item in list1" :key="item.code" draggable="true">
        {{ item.name }}
      </div>
    </div>
    <!-- 画布区域 -->
    <div class="targetContent" ref="targetContent">
      <div
        class="item"
        v-for="item in list2"
        :key="item.id"
        :ref="item.id"
        :style="{
          top: `${item.top - 16}px`,
          left: `${item.left - 85}px`,
          'z-index': `${item.zIndex}`
        }"
      >
        <template v-if="item.code === 'MyInput'">
          <a-input></a-input>
        </template>
      </div>
    </div>
  </div>
</template>
```

并将组件列表和画布中的页面分别通过list1，和list进行遍历渲染。

```js
<script>
import _ from "lodash";
export default {
  data() {
    return {
      list1: [
        { code: "MyInput", name: "输入框", props: {} }
      ],
      list2: [],
    };
  }
}
</script>
```

下面我们来分析一下如何实现将组件列表中的组件拖动到画布中，上文中我们讲到，拖动的元素以及目标元素可以设置一系列的事件，那么我们就可以在组件列表渲染时，为每个组件设置一下 `dragstart` 事件，在该事件中我们需要做如下处理：

1. 设置拖动元素的放置行为为`移动`，即`move`。
2. 组件在目标元素经过时，必须要阻止默认行为，否则不能触发`drop`。
3. 设置组件离开目标元素时放置行为为`不能拖放`，即`none`。
4. 拖动元素在目标元素松手时添加元素到画布，即将组件元数据添加到`list2`中，元素所对应的元数据记录也了这个组件在画面中的坐标位置。

然后在`dragend`事件中监听以上动作。

下面我们通过代码的方式来实现以上过程。首先在组件列表进行遍历时，添加组件的`dragstart` 和`dragend`事件。

```js
@dragstart="e => dragstart(e, item)"
@dragend="dragend"
```

下面是这两个事件的实现。

```js
dragstart(e, item) {
  this.dragItem = item;
  // 设置元素的放置行为——移动
  this.$refs.targetContent.addEventListener("dragenter", this.dragenter);
  // 在目标元素经过 必须要阻止默认行为 否则不能触发drop
  this.$refs.targetContent.addEventListener("dragover", this.dragover);
  // 离开目标元素时设置元素的放置行为——不能拖放
  this.$refs.targetContent.addEventListener("dragleave", this.dragleave);
  // 拖动元素在目标元素松手时添加元素到画布
  this.$refs.targetContent.addEventListener("drop", this.drop);
},
dragend(e) {
  this.$refs.targetContent.removeEventListener("dragenter", this.dragenter);
  this.$refs.targetContent.removeEventListener("dragover", this.dragover);
  this.$refs.targetContent.removeEventListener("dragleave", this.dragleave);
  this.$refs.targetContent.removeEventListener("drop", this.drop);
},
dragenter(e) {
  e.dataTransfer.dropEffect = "move";
},
dragover(e) {
  e.preventDefault();
},
dragleave(e) {
  e.dataTransfer.dropEffect = "none";
},
drop(e) {
  const { code } = this.dragItem;
  this.list2.push({
    top: e.offsetY,
    left: e.offsetX,
    zIndex: 1,
    code: code,
    id: Date.parse(new Date())
  });
  this.dragItem = null;
}
```

这样，我们组件列表中的组件就可以拖动到画布中了。

那拖动到画布中的组件又是如何实现通过拖动灵活的移动位置的呢？同样，我们可以将画布中的组件添加`mousedown`事件，在事件中我们添加`mousemove`事件的监听，当画布中的组件进行移动时，我们实时的将该被移动元素所对应的元数据坐标进行更新。下面是代码的实现。

```js
mousedown(e, item) {
  this.moveItem = item;
  document.addEventListener("mousemove", this.mousemove);
  document.addEventListener("mouseup", this.mouseup);
},
mousemove(e) {
  const _this = this;
  let { clientX, clientY } = e;
  const moveIdx = _.findIndex(this.list2, function(o) {
    return o.id === _this.moveItem.id;
  });
  let newList2 = _.cloneDeep(this.list2);
  newList2[moveIdx].top = clientY;
  newList2[moveIdx].left = clientX;
  this.list2 = newList2;
},
mouseup(e) {
  document.removeEventListener("mousemove", this.mousemove);
  document.removeEventListener("mouseup", this.mouseup);
}
```

这样画布中的组件也就支持移动了。

最后，附上完整代码：

```html
<template>
  <div>
    <!-- 左侧组件列表 -->
    <div class="left">
      <div
        class="left-item"
        v-for="item in list1"
        :key="item.code"
        draggable="true"
        @dragstart="e => dragstart(e, item)"
        @dragend="dragend"
      >
        {{ item.name }}
      </div>
    </div>
    <!-- 画布区域 -->
    <div class="targetContent" ref="targetContent">
      <div
        class="item"
        v-for="item in list2"
        :key="item.id"
        :ref="item.id"
        :style="{
          top: `${item.top - 16}px`,
          left: `${item.left - 85}px`,
          'z-index': `${item.zIndex}`
        }"
        @mousedown="e => mousedown(e, item)"
      >
        <template v-if="item.code === 'MyInput'">
          <a-input></a-input>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import _ from "lodash";
export default {
  data() {
    return {
      list1: [
        {
          code: "MyInput",
          name: "输入框",
          props: {}
        }
      ],
      list2: [],
      dragItem: null,
      moveItem: null
    };
  },
  methods: {
    dragstart(e, item) {
      this.dragItem = item;
      // 设置元素的放置行为——移动
      this.$refs.targetContent.addEventListener("dragenter", this.dragenter);
      // 在目标元素经过 必须要阻止默认行为 否则不能触发drop
      this.$refs.targetContent.addEventListener("dragover", this.dragover);
      // 离开目标元素时设置元素的放置行为——不能拖放
      this.$refs.targetContent.addEventListener("dragleave", this.dragleave);
      // 拖动元素在目标元素松手时添加元素到画布
      this.$refs.targetContent.addEventListener("drop", this.drop);
    },
    dragend(e) {
      this.$refs.targetContent.removeEventListener("dragenter", this.dragenter);
      this.$refs.targetContent.removeEventListener("dragover", this.dragover);
      this.$refs.targetContent.removeEventListener("dragleave", this.dragleave);
      this.$refs.targetContent.removeEventListener("drop", this.drop);
    },
    dragenter(e) {
      e.dataTransfer.dropEffect = "move";
    },
    dragover(e) {
      e.preventDefault();
    },
    dragleave(e) {
      e.dataTransfer.dropEffect = "none";
    },
    drop(e) {
      const { code } = this.dragItem;
      this.list2.push({
        top: e.offsetY,
        left: e.offsetX,
        zIndex: 1,
        code: code,
        id: Date.parse(new Date())
      });
      this.dragItem = null;
    },
    mousedown(e, item) {
      this.moveItem = item;
      document.addEventListener("mousemove", this.mousemove);
      document.addEventListener("mouseup", this.mouseup);
    },
    mousemove(e) {
      const _this = this;
      let { clientX, clientY } = e;
      const moveIdx = _.findIndex(this.list2, function(o) {
        return o.id === _this.moveItem.id;
      });
      let newList2 = _.cloneDeep(this.list2);
      newList2[moveIdx].top = clientY;
      newList2[moveIdx].left = clientX;
      this.list2 = newList2;
    },
    mouseup(e) {
      document.removeEventListener("mousemove", this.mousemove);
      document.removeEventListener("mouseup", this.mouseup);
    }
  }
};
</script>
<style lang="less" scoped>
.left {
  padding: 10px;
  position: absolute;
  width: 270px;
  background: rgb(247, 202, 202);
  top: 0;
  bottom: 0;
  left: 0;
}
.left-item {
  height: 100px;
  line-height: 100px;
  background: #fff;
}
.targetContent {
  background: rgb(173, 244, 247);
  height: 100vh;
  padding: 0 270px;
}
.item {
  position: absolute;
}
</style>
```



## 自定义组件的实现

当我们面对一些较为复杂的页面时，极有可能提供在设计器组件列表中的组件无法提供有效的支持，面对这种情况我们如何解决呢？

这里我们就需要给页面设计器提供一种支持二次扩展开发的机制，让我们能够通过代码开发的方式对页面设计器的组件列表进行有效的补充。这样，我们只需要补充组件，就可以实现任何复杂的UI展示形式了。

首先我们需要先开发一个自定义的组件，进行编译打包，将打包好的组件文件存储到服务器端，维护组件列表，将自定义组件与服务器端的组件文件进行关联。这样。我们在将组件拖入到画布中时，只需要从服务器端获取到组件文件，利用`vue`中的组件注册机制进行组件的动态注册，最后就可以通过`vue`中的动态组件进行组件的使用和渲染了。

<img src=".\assets\lowcode2.png" alt="lowcode2" />

在以上过程中，需要重点关注一下如何进行组件的开发及编译和如何进行组件动态引用及注册。



### 组件的编译

开发好的vue组件如何进行编译才能动态加载注册呢？其实非常简单，我们只需要将vue组件编译成umd（通用模块定义规范，通过运行时或者编译时让同一个代码模块在使用 `CommonJs`、`CMD` 甚至是 `AMD` 的项目中运行）格式的`js`文件就可以了。 当组件使用`webpack`进行编译时，可以将`webpack`的`output`参数设置设置为`umd`模式。

```json
output: {
  library: "library-name",
  libraryTarget: "umd",
}
```

使用`vue-cli`进行编译则更为简单，`vue-cli`可将目标构建为库。

```
vue-cli-service build --target lib --name myLib [entry]
```

通过以上构建命令，可将vue组件编译为以下文件，使用时我们可选择`umd.js`或者`umd.min.js`文件。

<img src=".\assets\lowcode3.png" alt="lowcode3" />



### 组件的动态加载及注册

组件编译为的`js`文件有了，我们如何进行动态的加载呢？只需要动态的引入`script`标签然后将`src`属性设置为组件`js`文件的地址就可以了，我们可以通过下面的代码进行`js`文件的动态引入。

```js
const script = document.createElement("script");
script.src = '组件js地址' + `?t=${Date.now()}`;
script.onload = () => {
  const exportCom = window['组件打包时—name参数'].default;
  this.comName = exportCom;
};
document.body.appendChild(script);
```

这样我们就可以获取到远端的组件，再将该组件通过`component`动态组件进行渲染就可以了。

```html
<component :is="comName" ></component>
```























