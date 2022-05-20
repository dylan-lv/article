# ref 功能

## 基本功能

```ts
// reactivity\test\ref.spec.ts
describe("ref", () => {
  it("happy path", () => {
    // ref给一个值1,返回a，通过a.value得到 1 这个值
    const a = ref(1);
    expect(a.value).toBe(1);
  });
});
```

- 代码实现：

```ts
// reactivity\ref.ts
class RefImpl {
  private _value: any;
  constructor(value) {
    this._value = value;
  }
  get value() {
    return this._value;
  }
}
export function ref(value) {
  return new RefImpl(value);
}
```

此时 `happy path` 测试就可以通过了。



## should be reactive

`ref` 需要是响应式的

```ts
// 添加测试代码
it("should be reactive", () => {
  const a = ref(1);
  let dummy;
  let calls = 0;
  // 通过effect做依赖收集
  effect(() => {
    calls++;
    dummy = a.value;
  });
  
  expect(calls).toBe(1); // 说明effect调用了一次
  expect(dummy).toBe(1); // 说明获取到了a.value的值

  a.value = 2; // 把它变成 2 的之后，下面两个值都需要改变
  expect(calls).toBe(2);
  expect(dummy).toBe(2);

  // 相同值不再触发依赖
  a.value = 2;
  expect(calls).toBe(2);
  expect(dummy).toBe(2);
});
```



### 收集依赖

收集依赖的操作，我们之前在 `effect.ts` 文件的 `track` 方法中已经做过了，会把 `activeEffect` 收集到 `dep` 对象中

现在 ref 中并没有 dep，那么我们来添加一下：

```ts
class RefImpl {
  private _value: any;
  public dep; // 声明 dep，用来存储 effect
  constructor(value) {
    this._value = value;
    this.dep = new Set(); // 初始化一下，dep对象是一个 Set
  }
  get value() {
    return this._value;
  }
}
```

现在我们已经有 dep 了，那么接下来在 `get value` 中做依赖收集的动作；

可以使用 `track` 方法，

区别在于我们之前 `reactive` 中调用 `track` 是基于它的 `key` 来获取 `dep`，但是我们的 `ref` 它只有一个 `value`，只会对应一个 `dep`。

所以我们在 `track` 中的获取操作都不需要了，只用把 `effect` 收集进 `dep` 就够了

```ts
// reactivity\effect.ts
export function track(target, key) {
  if (!isTracking()) return;
  let depsMap = targetMap.get(target);
  if (!depsMap) {
    depsMap = new Map();
    targetMap.set(target, depsMap);
  }
  let dep = depsMap.get(key);
  if (!dep) {
    dep = new Set();
    depsMap.set(key, dep);
  }
  trackEffects(dep);
}
// 抽离一下，因为 ref 没有target和key
export function trackEffects(dep) {
  if (dep.has(activeEffect)) return;
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}
// 抽离trigger
export function trigger(target, key) {
  const depsMap = targetMap.get(target);
  const dep = depsMap.get(key);
  triggerEffects(dep);
}
export function triggerEffects(dep) {
  for (const effect of dep) {
    if (effect.scheduler) {
      effect.scheduler();
    } else {
      effect.run();
    }
  }
}
```

```ts
class RefImpl {
  private _value: any;
  public dep;
  constructor(value) {
    this._value = value;
    this.dep = new Set();
  }
  get value() {
    // 有可能 ref 的使用过程中，并没有 effect，那么就不需要进行依赖收集了
    if (isTracking()){
      // 依赖收集
    	trackEffects(this.dep);
    }
    return this._value;
  }
  set value(newValue) {
    this._value = newValue; // 一定要先修改 value 的值
    // 触发依赖
    triggerEffects(this.dep);
  }
}
```

此时，除了 **相同值不再触发依赖** 这个测试之外，其余测试应该都通过了。



### 重复的值

如果 `ref` 被设置的是重复的值，那么 `effect` 的 `fn` 并不会被调用

```ts
class RefImpl {
  private _value: any;
  public dep;
  constructor(value) {
    this._value = value;
    this.dep = new Set();
  }
  get value() {
    if (isTracking()){
    	trackEffects(this.dep);
    }
    return this._value;
  }
  set value(newValue) {
    // 如果是相同的值，就直接 return 掉
    if(Object.is(newValue, this._value)) return;
    this._value = newValue;
    triggerEffects(this.dep);
  }
}
```

此时，测试就可以通过了

### 重构

- `Object.is` 抽离为 `hasChanged`

```ts
// shared\index.ts
export const hasChanged = (value, newValue) => {
  return !Object.is(value, newValue);
};
```

```ts
class RefImpl {
  ...
  set value(newValue) {
    // 如果是相同的值，就直接 return 掉
    if(!hasChanged(newValue, this._value)) return;
    ...
  }
}
```

- trackRefValue

```ts
class RefImpl {
  ...
  get value() {
    // if (isTracking()) {
    //   // 依赖收集
    //   trackEffects(this.dep);
    // }
    trackRefValue(this);
    return this._value;
  }
  ...
}
function trackRefValue(ref) {
  if (isTracking()) {
    // 依赖收集
    trackEffects(ref.dep);
  }
}
```



## ref 值是对象

在ref中，如果传入的value是一个Object对象，则转换成 reactive

添加测试代码：

```ts
it("should make nested properties reactive", () => {
  const a = ref({
    count: 1,
  });
  let dummy;
  effect(() => {
    dummy = a.value.count;
  });
  expect(dummy).toBe(1);
  a.value.count = 2;
  expect(dummy).toBe(2);
});
```

### 判断 value 是否是对象

在 ` ref` 中如果传入的是一个对象，那就将其转换成 `reactive` 类型的数据

```ts
class RefImpl {
  ...
  constructor(value) {
    // 看看 value 是不是对象
    this._value = isObject(value) ? reactive(value) : value;
    this.dep = new Set();
  }
 ...
}
```

### set 的时候对比

此时 `value` 有可能是一个 `reactive`，但是对比的时候我们需要是两个 `object` 进行对比

```ts
class RefImpl {
  ...
  constructor(value) {
    this._rawValue = value; // 保存原始的值
    this._value = isObject(value) ? reactive(value) : value;
    this.dep = new Set();
  }
  set value(newValue) {
    if (hasChanged(this._rawValue, newValue)) { // 使用原始的值和新值做对比
      this._rawValue = newValue;
      this._value = isObject(newValue) ? reactive(newValue) : newValue;
      triggerEffects(this.dep);
    }
  }
}
```

### 重构

可以看出下面这段代码和set中是一致的

```ts
this._value = isObject(value) ? reactive(value) : value;
```

将其进行抽离

```ts
class RefImpl {
  ...
  constructor(value) {
    this._rawValue = value;
    this._value = convert(value); // 切换
    this.dep = new Set();
  }
  set value(newValue) {
    if (hasChanged(this._rawValue, newValue)) {
      this._rawValue = newValue;
      this._value = convert(newValue); // 切换
      triggerEffects(this.dep);
    }
  }
}
function convert(value) {
  return isObject(value) ? reactive(value) : value;
}
```



## 全部代码：

### ref.spec.ts

```ts
import { effect } from "../effect";
import { ref } from "../ref";

describe("ref", () => {
  it("happy path", () => {
    const a = ref(1);
    expect(a.value).toBe(1);
  });
  it("should be reactive", () => {
    const a = ref(1);
    let dummy;
    let calls = 0;
    // 依赖收集
    effect(() => {
      calls++;
      dummy = a.value;
    });
    
    expect(dummy).toBe(1); // 说明effect调用了一次
    expect(calls).toBe(1); // 说明获取到了a.value的值

    a.value = 2; // 把它变成 2 的之后，下面两个值都需要改变
    expect(calls).toBe(2);
    expect(dummy).toBe(2);

    // same value should not trigger
    a.value = 2;
    expect(calls).toBe(2);
    expect(dummy).toBe(2);
  });
  it("should make nested properties reactive", () => {
    // 在ref中，如果传入的value是一个Object对象，则转换成 reactive
    const a = ref({
      count: 1,
    });
    let dummy;
    effect(() => {
      dummy = a.value.count;
    });
    expect(dummy).toBe(1);
    a.value.count = 2;
    expect(dummy).toBe(2);
  });
});
```

### ref.ts

```ts
import { hasChanged, isObject } from "../shared";
import { isTracking, trackEffects, triggerEffects } from "./effect";
import { reactive } from "./reactive";

class RefImpl {
  private _value: any;
  public dep;
  private _rawValue: any;
  constructor(value) {
    // value => reactive
    this._rawValue = value;
    // 1.看看 value 是不是对象
    // this._value = isObject(value) ? reactive(value) : value;
    this._value = convert(value);

    this.dep = new Set();
  }
  get value() {
    // if (isTracking()) {
    //   // 依赖收集
    //   trackEffects(this.dep);
    // }
    trackRefValue(this);
    return this._value;
  }
  set value(newValue) {
    // if (Object.is(newValue, this._value)) return;
    if (hasChanged(this._rawValue, newValue)) {
      this._rawValue = newValue;
      // 一定要先修改 value 的值，在去通知
      // this._value = isObject(newValue) ? reactive(newValue) : newValue;
      this._value = convert(newValue);
      // 触发依赖
      triggerEffects(this.dep);
    }
  }
}

function convert(value) {
  return isObject(value) ? reactive(value) : value;
}

export function ref(value) {
  return new RefImpl(value);
}

function trackRefValue(ref) {
  if (isTracking()) {
    // 依赖收集
    trackEffects(ref.dep);
  }
}
```

















