# Vue中使用marked加载 .md 文件



### 首先安装 marked 插件

```bash
npm i marked -S
```



### 组件中引入 marked

```html
<template>
  <div v-html="mdText"></div>
</template>

<script>
import { ref } from "vue";
import marked from "marked";

export default {
  setup() {
    const mdText = ref(marked('# Marked in the browser\n\nRendered by **marked**.'));
    return { mdText };
  },
};
</script>
```

如下图：

<img src=".\assets\marked1.png" alt="marked1" />



### 加载外部.md文件

```html
<!-- test.md -->
# 标题一
## 标题二
### 标题三
#### 标题四
##### 标题五
```

**要能够从外部导入.md 文件，我们还需要给项目安装两个 loader。**

```bash
npm i html-loader@1 markdown-loader -S
```

> html-loader版本不能太高，不然会报错



然后在`vue.config.js`文件中加入下面的配置

```js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('md')
      .test(/\.md/)
      .use('html-loader')
      .loader('html-loader')
      .end()
      .use('markdown-loader')
      .loader('markdown-loader')
      .end()
  }
}
```



最后就能正常在组件中导入`helloMarked.md`文件

```html
<template>
  <div v-html="mdText"></div>
</template>

<script>
import { ref } from "vue";
import marked from "marked";
import testMd from "../docs/test.md";

export default {
  setup() {
    const mdText = ref(marked(testMd));
    return { mdText };
  },
};
</script>
```

如下图：

<img src=".\assets\marked2.png" alt="marked2" />



























