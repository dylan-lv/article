### 1、实现 input 的 v-model

```jsx
export default defineComponent({
	setup() {
   	const formModel = reactive({
      name: ''
    })
    return { formModel };
  },
  render() {
    return (
    	<el-form>
        <el-form-item label="活动名称">
          <el-input value={this.formModel.name} onInput={value => this.formModel.name = value}></el-input>
        </el-form-item>
      </el-form>
    )
  }
}
```



### 2、el-form

注意 el-form 的 :model 属性，在jsx中是这样写的 `props: {{ model: this.data }}` ，比较特别

```jsx
render(h) {
	return (
  	<el-form
    	ref="form"
    	props={{ model: this.data }} // 注意:model要改成这样写
  		rules={this.rules}>
    	<el-form-item label="名称" prop="name">
        <el-input v-model={this.form.name}></el-input>
        // 或者
        <el-input value={this.form.name} onInput={value => this.form.name = value}></el-input>
      </el-form-item>
    </el-form>
  )
}
```



### 3、使用 .sync 修饰符

原版

```tsx
<el-dialog
	title="详情"
  :visible.sync="visible"
  width="700px"
  append-to-body
>
</el-dialog>
```

使用 jsx

```tsx
<el-dialog
	title="详情"
  on={{ ['update:visible']: this.visibleShow }}
  visible={this.visible}
  width="700px"
  append-to-body
>
</el-dialog>

visibleShow() {
  this.visible = true;
}
```



**实例：el-drawer组件**

```tsx
// 父组件
export default defineComponent({
  setup() {
    let visible = ref(false);
    
    const handleVisible = (flag) => {
      visible.value = flag;
    }
    setTimeout(() => {
      visible.value = true;
    }, 2000);
    return { visible, handleVisible };
  },
  render() {
    return (
      <div class="page-content">
        ...
        <Child 
          visible={ this.visible } 
          on={{ ["update:visible"]: this.handleVisible }} 
        />
      </div>
    );
  },
});
```

```tsx
// 子组件
export default defineComponent({
  props: ["visible"],
  setup(props, ctx) {
    const handleClose = () => {
      ctx.emit("update:visible", false)
    }

    return { handleClose }
  },
  render() {
    return (
      <el-drawer
        title="我是标题"
        visible={this.visible}
        direction="rtl"
        before-close={this.handleClose}>
        <span>我来啦!</span>
      </el-drawer>
    )
  }
})
```



### 4、父子通信

```tsx
// 父组件直接传参，子组件通过props接收，通过 ctx.emit 向父组件传
export default defineComponent({
  props: ["visible"], // 接收参数
  setup(props, ctx) { // 或者 setup(props, { emit })
    const handleClose = () => {
      ctx.emit("update:visible", false); // emit向上传递
    }

    return { handleClose }
  },
}
```



### 5、el-table中用到的 scopedSlots

```tsx
<el-table-column label="操作" width="100" scopedSlots={{ default: (scope) => (
  <div>
    <span style="color: #3786FD; cursor: pointer;" onClick={() => this.tableHandler(scope.row)}>选择</span>
  </div>
) }}>
</el-table-column>
```













