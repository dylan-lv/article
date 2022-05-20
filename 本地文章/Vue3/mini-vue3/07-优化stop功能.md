# 优化stop功能

优化stop的另外一种边缘case。

我们先来看看之前写的 `stop` 单元测试

```ts
it("stop", () => {
  let dummy;
  const obj = reactive({ prop: 1 });
  const runner = effect(() => {
    dummy = obj.prop;
  });
  obj.prop = 2;
  expect(dummy).toBe(2);
  stop(runner);
  obj.prop = 3; // 这里直接 set
  expect(dummy).toBe(2);

  runner();
  expect(dummy).toBe(3);
});
```

这里我们之前实现的是，给 `obj.prop` 赋值 3，`effect的fn`也不会触发，`dummy` 依旧是 2

现在我们尝试用**另外一种方式**给 `obj.prop` 赋值 

```ts
it("stop", () => {
  ...
  obj.prop++;
  expect(dummy).toBe(2);
	...
});
```

此时可以看到测试失败了。我们之前的 `obj.prop = 3` 实际上只涉及到了 `set` 操作

`obj.prop++` 其实是两个操作，先 `get obj.prop`，再 `set obj.prop + 1`，也就是说，比之前多了一个 get 操作

- 我们来看一下 get 操作

```ts
export function track(target, key) {
  ...
  // 这里就会把当前的依赖给收集进去了
  if (dep.has(activeEffect)) return;
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}
```

可以看到 `get` 操作把依赖重新收集进去了，我们之前在 `stop` 中清理依赖的事情白干了！！！

这也是为什么我们在单测中把 `obj.prop` 变成 ++ 之后，测试就过不去了的原因。



### 解决方式

我们可以在触发依赖收集的过程中，做一些事情。

- 在 `track` 中添加一个变量，控制它是否应该收集依赖

```ts
let shouldTrack;
let activeEffect;
export function track(target, key) {
  ...
	if (!activeEffect) return;
  // 是否应该收集依赖
  if (!shouldTrack) return;
  if (dep.has(activeEffect)) return;
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}
```

- `shouldTrack` 赋值的位置

在 `run` 执行的时候，就会进行**依赖收集**

```ts
class ReactiveEffect {
  ...
  run() {
    // 调用fn的时候就会收集依赖，这里使用 shouldTrack 来做区分
    // this.active来区分是否已经 stop 了
    if (!this.active) {
      return this._fn();
    }

    shouldTrack = true;
    activeEffect = this;
    const result = this._fn();
    shouldTrack = false; // 执行完成 fn 后，关掉 shouldTrack。因为它是一个全局变量
    return result;
  }
}
```

此时再运行测试，可以发现测试已经通过了。

### 重构

`track` 中有两个判断

```ts
if (!activeEffect) return;
if (!shouldTrack) return;
```

- 可以将其放在函数最前面
  - 如果不需要做依赖收集的话，后面获取的逻辑也就不需要了

```ts
export function track(target, key) {
  // 如果不需要 track 的话，后面的收集依赖也就不需要了。
  if (!activeEffect) return;
  if (!shouldTrack) return;

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

  // !避免重复依赖收集
  if (dep.has(activeEffect)) return;
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}
```

- 将这两个抽离出来

```ts
export function track(target, key) {
  // 如果不需要 track 的话，后面的收集依赖也就不需要了。
  if (!isTracking()) return;
	...
}
function isTracking() {
  // shouldTrack 为 true，并且 activeEffect 有值，说明应该是一个正在收集的状态
  return shouldTrack && activeEffect !== undefined;
}
```



## 全部代码：

### effect.spec.ts

```ts
import { effect, stop } from "../effect";
import { reactive } from "../reactive";

describe.skip("effect", () => {
  it("happy path", () => {
    const user = reactive({
      age: 10,
    });

    let nextAge;
    effect(() => {
      nextAge = user.age + 1;
    });
    expect(nextAge).toBe(11);

    user.age++;
    expect(nextAge).toBe(12);
  });

  it("should return runner when call effect", () => {
    let foo = 10;
    const runner = effect(() => {
      foo++;
      return "fooo";
    });
    expect(foo).toBe(11);
    const r = runner();
    expect(foo).toBe(12);
    expect(r).toBe("fooo");
  });

  it("scheduler", () => {
    let dummy;
    let run: any;
    const scheduler = jest.fn(() => {
      run = runner;
    });
    const obj = reactive({ foo: 1 });
    const runner = effect(
      () => {
        dummy = obj.foo;
      },
      { scheduler }
    );
    expect(scheduler).not.toHaveBeenCalled();
    expect(dummy).toBe(1);
    obj.foo++;
    expect(scheduler).toHaveBeenCalledTimes(1);
    expect(dummy).toBe(1);
    run();
    expect(dummy).toBe(2);
  });

  it("stop", () => {
    let dummy;
    const obj = reactive({ prop: 1 });
    const runner = effect(() => {
      dummy = obj.prop;
    });
    obj.prop = 2;
    expect(dummy).toBe(2);
    stop(runner);
    //! stop的边缘 case
    // obj.prop = 3;
    obj.prop++; // 触发get的时候会去重新收集依赖
    expect(dummy).toBe(2);

    runner();
    expect(dummy).toBe(3);
  });

  it("onStop", () => {
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
import { extend } from "../shared";

let activeEffect;
let shouldTrack; // 控制是否应该收集依赖
class ReactiveEffect {
  private _fn: any;
  deps = [];
  active = true;
  onStop?: () => void;
  constructor(fn, public scheduler?) {
    this._fn = fn;
  }
  run() {
    // 调用fn的时候就会收集依赖，这里使用 shouldTrack 来做区分
    // this.active来区分是否已经 stop 了
    if (!this.active) {
      return this._fn();
    }

    shouldTrack = true;
    activeEffect = this;
    const result = this._fn();
    shouldTrack = false; // 执行完成 fn 后，关掉 shouldTrack。因为它是一个全局变量
    return result;
  }
  stop() {
    if (this.active) {
      cleanupEffect(this);
      if (this.onStop) {
        this.onStop();
      }
      this.active = false;
    }
  }
}

function cleanupEffect(effect) {
  effect.deps.forEach((dep: any) => {
    dep.delete(effect);
  });
  effect.deps.length = 0; // 执行cleanupEffect 之后，里面的 dep 已经空了，这里把数组也清空
}

const targetMap = new Map();

export function track(target, key) {
  // if (!activeEffect) return;
  // if (!shouldTrack) return;
  if (!isTracking()) return; // 如果不需要 track 的话，后面的收集依赖也就不需要了。

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

  // 之前已经在 dep 中了，就没必要再添加进去了
  if (dep.has(activeEffect)) return;
  dep.add(activeEffect);
  activeEffect.deps.push(dep);
}

function isTracking() {
  // shouldTrack 为 true，并且 activeEffect 有值，说明应该是一个正在收集的状态
  return shouldTrack && activeEffect !== undefined;
}

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

export function effect(fn, options: any = {}) {
  const _effect = new ReactiveEffect(fn, options.scheduler);
  extend(_effect, options);

  _effect.run();

  const runner: any = _effect.run.bind(_effect);
  runner.effect = _effect;

  return runner;
}

export function stop(runner) {
  runner.effect.stop();
}
```









