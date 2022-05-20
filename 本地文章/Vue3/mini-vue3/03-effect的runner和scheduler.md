# effect的runner和scheduler

完善 `effect` 的功能

## 一、runner

- 调用effect（fn）之后，其实是会返回一个 function（runner） 的
- 当调用 function（runner） 时会再次调用用户传给 effect 的 fn 函数 
- 调用 fn 的时候会把 fn 的返回值返回出去

添加测试代码

```ts
// src/reactivity/test/effect.spec.ts
describe("effect", () => {
  ...
  it("should return runner when call effect", () => {
    let foo = 10;
    // 返回一个 runner 函数
    const runner = effect(() => {
      foo++;
      return "fooo";
    });
    expect(foo).toBe(11);
    // 调用 runner 函数的时候，可以拿到返回值 r；并且fn也应该被执行了
    const r = runner();
    expect(foo).toBe(12); // 验证内部函数fn，是否被执行了
    expect(r).toBe("fooo"); // 验证对应的返回值
  });
})
```

修改 `effect.ts` 代码：

```ts
class ReactiveEffect {
  ...
  run() {
    activeEffect = this;
    // 当调用用户传入的 fn 之后，需要把 fn 的返回值给返回出去
    return this._fn();
  }
}
export function effect(fn, options: any = {}) {
  const _effect = new ReactiveEffect(fn);
  _effect.run();
  // runner需要调用fn，相当于run方法的功能
  // 在 _effect.run 里面涉及到 this 指针的问题
  return _effect.run.bind(_effect);
}
```

- `runner` 需要调用 `fn`，相当于 `run` 方法的功能
- 在 `_effect.run` 里面涉及到 this 指针的问题，所以使用 `bind` 绑定一下
- 在 `run` 方法中，调用用户传入的 `fn` 之后，需要把 `fn` 的返回值给返回出去



## 二、scheduler

我们先来看一下单测 

`src\reactivity\tests\effect.spec.ts`

```ts
it("scheduler", () => {
  let dummy;
  let run: any;
  const scheduler = jest.fn(() => {
    run = runner;
  });
  const obj = reactive({ foo: 1 });
  // 先调用 effect，传入 fn
  // 第二个参数是一个对象，里面有一个 scheduler
  const runner = effect(
    () => {
      dummy = obj.foo;
    },
    { scheduler }
  );
  // 验证 scheduler 一开始不会被调用
  expect(scheduler).not.toHaveBeenCalled();
  // 验证一开始会执行 fn
  expect(dummy).toBe(1);
  // 第一次触发依赖
  obj.foo++;
  // 验证 scheduler 被执行了一次
  expect(scheduler).toHaveBeenCalledTimes(1);
  // 验证 effect 回调函数（fn）没有再被执行
  expect(dummy).toBe(1);
  // 直接执行 run 方法
  run();
  // 验证 run 可以执行 fn 
  expect(dummy).toBe(2);
});
```

1. 通过 `effect` 的第二个参数给定了一个 `scheduler` 的函数
2. `effect` 第一次执行的时候，会执行 `fn`
3. 当响应式对象 `set update` 的时候，不会执行 `fn`，而是执行 `scheduler`
4. 如果执行 `runner` 的时候，会再次执行 `fn`



现在我们来更新 effect 功能

```ts
export function effect(fn, options: any = {}) {
  // 接收 options 对象，获取scheduler
  const _effect = new ReactiveEffect(fn, options.scheduler);

  _effect.run();

  const runner = _effect.run.bind(_effect);

  return runner;
}
```

- 构造函数中需要接收一下 `scheduler`

```ts
class ReactiveEffect {
  private _fn: any;
  constructor(fn, public scheduler?) {
    this._fn = fn;
  }
}
```



- 数据更新的时候，会触发 `trigger`，下面来更新一下 `trigger` 函数

```ts
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
```



## 全部代码如下：

### effect.spec.ts

```ts
import { effect } from "../effect";
import { reactive } from "../reactive";

describe("effect", () => {
  // 核心代码逻辑
  it("happy path", () => {
    const user = reactive({
      age: 10,
    });

    let nextAge;
    // user.age：依赖收集
    // effect一上来会调用fn，然后会触发 user.age 的get操作，触发get时进行依赖收集
    effect(() => {
      nextAge = user.age + 1;
    });
    expect(nextAge).toBe(11);

    // 更新：触发依赖
    // 触发依赖：user.age触发set操作时，会把所有收集到的fn拿出来调用一下
    user.age++;
    expect(nextAge).toBe(12);
  });
  it("should return runner when call effect", () => {
    // 调用effect（fn）之后，其实是会返回一个 function（runner） 的，当调用 function 时会再次调用 传给 effect 的 fn 函数 ，当调用 fn 的时候会把 fn 的返回值返回出去
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
    // 1. 通过 effect 的第二个参数给定了一个 scheduler 的函数
    // 2. effect 第一次执行的时候，还会执行 fn
    // 3. 当响应式对象 set update 的时候，不会执行 fn，而是执行 scheduler
    // 4. 如果执行 runner 的时候，会再次执行 fn
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
    // 第一次触发依赖
    obj.foo++;
    expect(scheduler).toHaveBeenCalledTimes(1);
    // effect 回调函数应该没有再被执行
    expect(dummy).toBe(1);
    // 直接执行 run 方法
    run();
    expect(dummy).toBe(2);
  });
});
```



### effect.ts

```ts
import { extend } from "../shared";

class ReactiveEffect {
  private _fn: any;
  constructor(fn, public scheduler?) {
    this._fn = fn;
  }
  run() {
    activeEffect = this;
    // 当调用用户传入的 fn 之后，需要把 fn 的返回值给返回出去
    return this._fn();
  }
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
  _effect.run();
  const runner: any = _effect.run.bind(_effect);
  return runner;
}
```





