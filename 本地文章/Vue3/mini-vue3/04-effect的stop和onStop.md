# effect的stop和onStop

实现 `effect` 的 `stop` 和 `onStop` 功能

## stop

首先，我们来看一下单元测试的代码：

```ts
import { effect, stop } from "../effect";
import { reactive } from "../reactive";
describe("effect", () => {
  it("stop", () => {
    let dummy;
    const obj = reactive({ prop: 1 });
    const runner = effect(() => {
      dummy = obj.prop;
    });
    obj.prop = 2;
    expect(dummy).toBe(2);
    // 调用 stop 的时候，应该把当前 effect 从 deps 中删除掉
    stop(runner);
    obj.prop = 3;
    expect(dummy).toBe(2); // 停止更新

    // stop只是终止“依赖触发”的 effect 执行，不影响返回的 runner 函数
    runner();
    expect(dummy).toBe(3);
  });
})
```

**stop 功能点：**

- `effect` 中导出一个 `stop` 方法
- `stop` 方法的参数是一个 `runner` 函数
- 在后续更新响应式的值的时候，可以看到依赖响应式的值 `dummy` 就不再更新了
- 直接调用 `runner` 的话，`dummy` 的值又会更新



那么，如何实现这个功能呢？

我们知道，之前在触发 `set` 的时候，会手动更新所有的依赖：

set -> trigger -> 取出 dep -> 遍历执行 effect.run

```ts
export function trigger(target, key) {
  const depsMap = targetMap.get(target);
  const dep = depsMap.get(key);

  for (const effect of dep) {
    if (effect.scheduler) {
      effect.scheduler();
    } else {
      effect.run();
    }
  }
}
```

思路：那么，如果我们不想通知的话，只需要把 `effect` 从 `dep` 中删除掉就可以了

### （1）stop方法

- 接收一个runner函数
- `runner` 函数上需要挂载 `effect`（通过 `runner` 获取 `effect`）
- 执行 `effect` 的 `stop` 方法

```ts
export function stop(runner) {
  // 接收的runner，就是effect返回出来的runner
  runner.effect.stop();
}
```



### （2）runner 挂载 effect

通过上面的代码，我们知道需要在 `runner` 方法上挂载当前 `effect` 实例

```ts
export function effect(fn, options: any = {}) {
  // 接收 options 对象，获取scheduler
  const _effect = new ReactiveEffect(fn, options.scheduler);
  _effect.run();

  // 在 _effect.run 里面涉及到 this 指针的问题
  const runner: any = _effect.run.bind(_effect);
  runner.effect = _effect; // 双向挂载：effect能得到 runner，runner中也保存effect

  return runner;
}
```



### （3）effect 实现 stop功能

我们的 `effect` 实例是通过 `new ReactiveEffect` 出来的，现在，需要在他上面实现 `stop` 功能

```ts
class ReactiveEffect {
  private _fn: any;
  constructor(fn, public scheduler?) {
    this._fn = fn;
  }
  run() {
    activeEffect = this;
    return this._fn();
  }
  stop() {
    
  }
}
```

我们需要拿到保存了当前 `effect` 的 `dep`，并将 `dep` 中的当前 `effect` 删除掉。

但是在 `effect` 这里我们没法得知哪些 `dep` 保存了当前 `effect`，所以我们需要记录一下。

那么在哪里记录呢？通过观察代码可以知道，`activeEffect` 是在 `track` 时被收集进 `dep` 的，那此时我们可以反向对 `dep` 进行收集：

```ts
export function track(target, key) {
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
  // activeEffect有可能是undefined，因为有可能是单纯的reactive，并没有使用 effect
  if(!activeEffect) return;
  dep.add(activeEffect); // dep 收集 effect
  activeEffect.deps.push(dep); // 反向收集：effect可以知道自己被存储在哪些 dep 中
}
```

`effect` 将 `dep` 收集进自己的 `deps` 中，然后在 `stop` 中就可以执行清空操作了

```ts
class ReactiveEffect {
  private _fn: any;
  deps = [];
  ...
  stop() {
    this.deps.forEach((dep: any) => {
      dep.delete(this);
    });
  }
}
```



#### 优化代码

- 抽离 `cleanupEffect`：清空 `dep` 中的 `effect`

```ts
class ReactiveEffect {
  private _fn: any;
  deps = [];
  ...
  stop() {
    cleanupEffect(this)
  }
}
function cleanupEffect(effect) {
  effect.deps.forEach((dep: any) => {
    dep.delete(effect);
  });
  effect.deps.length = 0;
}
```

- 性能问题：用户可能会频繁调用stop，给个 `active` 状态后，即使外部多次调用stop，也只会清空一次

```ts
class ReactiveEffect {
  private _fn: any;
  deps = [];
  active = true;
  ...
  stop() {
    if(this.active) {
      this.active = false;
      cleanupEffect(this);
    }
  }
}
function cleanupEffect(effect) {
  effect.deps.forEach((dep: any) => {
    dep.delete(effect);
  });
  effect.deps.length = 0;
}
```

通过 `active` 状态，只会在第一次执行 `stop` 时清空 `effect.deps`，之后在调用 `stop` 就不会执行清空操作了。



## onStop

我们来看一下单测

```ts
describe("effect", () => {
  it("onStop", () => {
    // 当用户调用 stop 之后，onStop 会被执行
    const obj = reactive({
      foo: 1,
    });
    const onStop = jest.fn();
    let dummy;
    const runner = effect(
      () => {
        dummy = obj.foo;
      },
      {
        onStop,
      }
    );

    stop(runner);
    expect(onStop).toHaveBeenCalledTimes(1);
  });
});
```

- 在 `effect` 的第二个参数中
- 当 `stop` 被调用之后，`onStop` 会被执行。也就是 `stop` 的回调函数，允许用户做一下额外的处理

### （1）在 effect 实例上添加 onStop

```ts
export function effect(fn, options: any = {}) {
  // 接收 options 对象，获取scheduler
  const _effect = new ReactiveEffect(fn, options.scheduler);

  _effect.onStop = options.onStop;

  _effect.run();

  const runner: any = _effect.run.bind(_effect);
  runner.effect = _effect; // 双向挂载：effect能得到 runner，runner中也保存effect

  return runner;
}
```

- 这里可以重构一下，因为后续可能会有多个 `options` 的内容
  - `_effect.onStop = options.onStop;` => `Object.assign(_effect, options)`
  - 可以更加语义化一些，并且将参数合并的方法提出来，变成公用方法：`extend(_effect, options)`

重构后代码如下：

```ts
// src/shared/index.ts
export const extend = Object.assign;
```

```ts
// src/reactivity/effect.ts
import { extend } from "../shared/index";
export function effect(fn, options: any = {}) {
  ...
  // _effect.onStop = options.onStop;
  extend(_effect, options)
	...
}
```



### （2）在 effect 构造函数上添加 onStop

我们在调用 `stop` 之后，执行 `onStop` 回调

```ts
class ReactiveEffect {
  ...
  onStop?: () => void;
  stop() {
    if (this.active) {
      cleanupEffect(this);
      if(this.onStop) {
        this.onStop()
      }
      this.active = false;
    }
  }
}
```





## 全部代码如下：

### effect.spec.ts

```ts
import { effect, stop } from "../effect";
import { reactive } from "../reactive";

describe("effect", () => {
  it("stop", () => {
    let dummy;
    const obj = reactive({ prop: 1 });
    const runner = effect(() => {
      dummy = obj.prop;
    });
    obj.prop = 2;
    expect(dummy).toBe(2);
    // 调用 stop 的时候，应该把当前 effect 从 deps 中删除掉
    stop(runner);
    obj.prop = 3;
    expect(dummy).toBe(2);

    // stop只是终止“依赖触发”的 effect 执行，不影响返回的 runner 函数
    runner();
    expect(dummy).toBe(3);
  });
  it("onStop", () => {
    // 当用户调用 stop 之后，onStop 会被执行
    const obj = reactive({
      foo: 1,
    });
    const onStop = jest.fn();
    let dummy;
    const runner = effect(
      () => {
        dummy = obj.foo;
      },
      {
        onStop,
      }
    );

    stop(runner);
    expect(onStop).toHaveBeenCalledTimes(1);
  });
});
```

### effect.ts

```ts
import { extend } from "../shared/index";

class ReactiveEffect {
  private _fn: any;
  deps = [];
  active = true;
  onStop?: () => void;
  constructor(fn, public scheduler?) {
    this._fn = fn;
  }
  run() {
    activeEffect = this;
    // 当调用用户传入的 fn 之后，需要把 fn 的返回值给返回出去
    return this._fn();
  }
  stop() {
    // this.deps.forEach((dep: any) => {
    //   dep.delete(this);
    // });
    // !代码优化，第一步：提取成函数
    // cleanupEffect(this);

    // !代码优化，第二步：可能会频繁调用stop，给个状态后，即使外部多次调用stop，也只会清空一次
    if (this.active) {
      cleanupEffect(this);
      if(this.onStop) {
        this.onStop()
      }
      this.active = false;
    }
  }
}

function cleanupEffect(effect) {
  effect.deps.forEach((dep: any) => {
    dep.delete(effect);
  });
}

const targetMap = new Map();
let activeEffect;

export function track(target, key) {
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

  // activeEffect有可能是undefined，因为有可能是单纯的reactive，并没有使用 effect
  if(!activeEffect) return;

  dep.add(activeEffect);
  activeEffect.deps.push(dep); // 反向收集：effect可以知道自己被存储在哪些 dep 中
}

export function trigger(target, key) {
  const depsMap = targetMap.get(target);
  const dep = depsMap.get(key);

  for (const effect of dep) {
    // 触发依赖的时候，看看effect中是否有 scheduler，如果有的话就执行，没有的话才会执行run方法
    if (effect.scheduler) {
      effect.scheduler();
    } else {
      effect.run();
    }
  }
}

export function effect(fn, options: any = {}) {
  // 接收 options 对象，获取scheduler
  const _effect = new ReactiveEffect(fn, options.scheduler);

  // !这里需要重构一下，因为后续可能有很多的options
  // _effect.onStop = options.onStop;
  // !重构1
  // Object.assign(_effect, options)
  // !重构2：具有语义化一些
  extend(_effect, options)

  _effect.run();

  // runner需要调用fn，相当于run方法的功能
  // 在 _effect.run 里面涉及到 this 指针的问题
  const runner: any = _effect.run.bind(_effect);
  runner.effect = _effect; // 双向挂载：effect能得到 runner，runner中也保存effect

  return runner;
}

export function stop(runner) {
  runner.effect.stop();
}
```

### shared/index.ts

```ts
export const extend = Object.assign;
```







