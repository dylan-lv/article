# 自定义Canvas渲染器

```html
<!-- example\customRenderer\index.html -->
<body>
  <script src="https://pixijs.download/release/pixi.js"></script>
  <div id="app"></div>
  <script src="main.js" type="module"></script>
</body>
```

```js
// App.js
import {
  h
} from "../../lib/guide-mini-vue.esm.js"

export const App = {
  setup() {
    return {
      x: 100,
      y: 100
    }
  },
  render() {
    return h("rect", {
      x: this.x,
      y: this.y
    })
  }
}
```

```js
// main.js
import { createRenderer } from '../../lib/guide-mini-vue.esm.js'
import { App } from './App.js'
// canvas-render：pixijs
// console.log(PIXI);

const game = new PIXI.Application({
  width: 500,
  height: 500
})
document.body.append(game.view)

const renderer = createRenderer({
  createElement(type) {
    if(type === "rect") {
      const rect = new PIXI.Graphics()
      rect.beginFill(0xff0000)
      rect.drawRect(0, 0, 100, 100)
      rect.endFill()

      return rect
    }
  },
  patchProp(el, key, value) {
    el[key] = value
  },
  insert(el, parent) {
    parent.addChild(el)
  }
})

renderer.createApp(App).mount(game.stage)

// const rootContainer = document.querySelector("#app");
// createApp(App).mount(rootContainer);
```























