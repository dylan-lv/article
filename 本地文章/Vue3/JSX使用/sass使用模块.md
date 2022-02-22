样式文件

```scss
// index.module.scss
.test{
	color: red;
}
```

jsx文件

```jsx
import Styles from './index.module.scss'
export default defineComponent({
	render() {
    return(
    	<div class={Styles['test']}>hahahah</div>
    )
  }
}
```





