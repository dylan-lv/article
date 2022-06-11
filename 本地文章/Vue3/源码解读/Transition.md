# Transition

- `v-enter-from`：输入的开始状态。在插入元素之前添加，在插入元素后删除一帧。
- `v-enter-active`：输入的活动状态。在整个进入阶段应用。在插入元素之前添加，在过渡/动画结束时将其删除。此类可用于定义进入过渡的持续时间，延迟和缓和曲线。
- `v-enter-to`：输入的结束状态。插入元素（同时`v-enter-from`删除）后添加一帧，在过渡/动画结束时删除。
- `v-leave-from`：离开的开始状态。触发离开过渡时立即添加，在一帧后移除。
- `v-leave-active`：离开的活动状态。在整个离开阶段都适用。触发离开过渡时立即添加，当过渡/动画结束时将其移除。此类可用于定义离开过渡的持续时间，延迟和缓和曲线。
- `v-leave-to`：离开的结束状态。触发离开过渡（同时`v-leave-from`删除）后添加一帧，在过渡/动画结束时删除。



## Transition使用

```vue
<script setup lang="ts">
import { ref } from "vue";

const visible = ref(true);
const change = () => {
  visible.value = !visible.value;
};
</script>

<template>
  <div>
    <button @click="change">切换</button>
    <transition name="fade">
      <div v-show="visible">一段文本</div>
    </transition>
  </div>
</template>

<style lang="scss" scoped>
.fade-enter-active,
.fade-leave-active {
  transition: all .5s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```



## TransitionGroup使用

可以同时渲染整个列表

- 默认情况下，它不会渲染一个包裹元素，但是可以通过指定 `tag` 来渲染一个指定的元素
- 过渡模式不可用了，因为我们不再相互切换特有的元素
- 内部元素需要提供一个指定的 `key` 
- CSS的过渡类会应用在内部元素中，而不是容器上

```vue
<script setup lang="ts">
import { ref } from "vue";

const list = ref([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const Push = () => {
  list.value.push(100);
};
const Pop = () => {
  list.value.pop();
};

</script>

<template>
  <div>
    <button @click="Push">添加</button>
    <button @click="Pop">删除</button>
    <transition-group name="fade">
      <div v-for="item in list" :key="item" style="margin: 10px;">{{ item }}</div>
    </transition-group>
  </div>
</template>

<style lang="scss" scoped>
.fade-enter-active,
.fade-leave-active {
  transition: all .5s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```



- 数字切换

```vue
<template>
  <div class="amount">
    <span>￥</span>
    <transition-group name="list" tag="div" style="position: relative;">
      <div
        v-for="(item , index) in todayAmountComputed"
        :key="item + index"
        style="display: inline-block;position: absolute;"
        :style="{ left: index * 53 + 'px', 'transition-delay': 0.1 * index + 's'}"
      >
        {{ item }}
      </div>
    </transition-group>
  </div>
</template>
<script setup lang="ts">
import { computed, onMounted, ref } from "vue";

const todayAmount = ref("0");
const todayAmountComputed = computed(() => todayAmount.value.split(""));
onMounted(() => {
  setTimeout(() => {
    todayAmount.value = "" + Math.floor(Math.random() * 10000);
  }, 1000);
});
</script>

<style lang="scss" scoped>
.amount {
  color: gold;
  font-size: 85px;
  display: flex;
  justify-content: center;
}

.list-enter-active,
.list-leave-active {
  transition: transform 1s ease, opacity 1s ease;
}

.list-enter-from {
  opacity: 0;
  transform: translateY(50%);
}

.list-leave-to {
  opacity: 0;
  transform: translateY(-50%);
}
</style>
```

- `transition-delay`：后面的数字延迟执行
- `list-enter-from`：从下方 50% 过来
- `list-leave-to`：到上方 50% 去























