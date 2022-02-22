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













