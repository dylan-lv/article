# isReactive和isReadonly

### isReactive

我们首先补一下测试

```ts
// reactivity\test\reactive.spec.ts
import { isReactive, reactive } from "../reactive";
describe("reactive", () => {
  it("happy path", () => {
    const original = { foo: 1 };
    const observed = reactive(original);
    expect(observed).not.toBe(original);
    expect(observed.foo).toBe(1);

    // 判断这个对象是否是 reactive 类型
    expect(isReactive(observed)).toBe(true);
    expect(isReactive(original)).toBe(false);
  });
});
```

我们通过之前的 `createGetter` 方法，这里的 get 操作中包含了 `isReadonly` 参数，通过这个参数就可以知道当前触发的 get 操作是什么类型

所以我们只需要想办法触发 `get` 操作就行了

```ts
// reactivity\reactive.ts
export function isReactive(value) {
  return value["is_reactive"];
}
// reactivity\baseHandlers.ts
function createGetter(isReadonly = false) {
  return function get(target, key) {
    if (key === "is_reactive") {
      return !isReadonly;
    }
		...
  };
}
```

此时单测会有一个报错：`expect(isReactive(original)).toBe(false);` 中，`isReactive(original)` 是 `undefined`

这是因为 `original` 不是一个 reactive 对象，所以不会调用 get，这个原始对象上又没有 `is_reactive` 参数，所以会返回 `undefined`

我们只需要对 `isReactive` 函数返回的内容转换成 `bool` 值就行了

```ts
// reactivity\reactive.ts
export function isReactive(value) {
  return !!value["is_reactive"];
}
```

此时单测就通过了

### 优化代码

我们把 `is_reactive` 参数名抽离成一个枚举

```ts
// reactivity\reactive.ts
export const enum ReactiveFlags {
  IS_REACTIVE = "__v_isReactive",
}
export function isReactive(value) {
  return !!value[ReactiveFlags.IS_REACTIVE];
}
// reactivity\baseHandlers.ts
function createGetter(isReadonly = false) {
  return function get(target, key) {
    if (key === ReactiveFlags.IS_REACTIVE]) {
      return !isReadonly;
    }
		...
  };
}
```



### isReadonly

同 `isReactive` 类似，我们先写测试

```ts
// readonly.spec.ts
describe("readonly", () => {
  it("happy path", () => {
    const original = { foo: 1, bar: { baz: 2 } };
    const wrapped = readonly(original);
    expect(wrapped).not.toBe(original);
    expect(wrapped.foo).toBe(1);
		// 补充测试
    expect(isReadonly(wrapped)).toBe(true);
    expect(isReadonly(original)).toBe(false);
  });
});
```

代码实现：

```ts
// reactivity\reactive.ts
export const enum ReactiveFlags {
  IS_REACTIVE = "__v_isReactive",
  IS_READONLY = "__v_isReadonly",
}
export function isReadonly(value) {
  return !!value[ReactiveFlags.IS_READONLY];
}
// reactivity\baseHandlers.ts
function createGetter(isReadonly = false) {
  return function get(target, key) {
    if (key === ReactiveFlags.IS_REACTIVE) {
      return !isReadonly;
    } else if (key === ReactiveFlags.IS_READONLY) {
      return isReadonly;
    }
		...
  };
}
```



## 全部代码：

### reactive.spec.ts

```ts
import { isReactive, reactive } from "../reactive";

describe.skip("reactive", () => {
  it("happy path", () => {
    const original = { foo: 1 };
    const observed = reactive(original);
    expect(observed).not.toBe(original);
    expect(observed.foo).toBe(1);

    // 补一下测试
    expect(isReactive(observed)).toBe(true);
    // 会得到一个 undefined，因为没有触发 get
    expect(isReactive(original)).toBe(false);
  });
});
```

### readonly.sepc.ts

```ts
import { isReadonly, readonly } from "../reactive";

describe.skip("readonly", () => {
  it("happy path", () => {
    const original = { foo: 1, bar: { baz: 2 } };
    const wrapped = readonly(original);
    expect(wrapped).not.toBe(original);
    expect(wrapped.foo).toBe(1);
		// 补一下测试
    expect(isReadonly(wrapped)).toBe(true);
    expect(isReadonly(original)).toBe(false);
  });

  it("warn then call set", () => {
    console.warn = jest.fn();
    const user = readonly({
      age: 10,
    });
    user.age = 11;
    expect(console.warn).toBeCalled();
  });
});
```

### reactive.ts

```ts
import { mutableHandlers, readonlyHanders } from "./baseHandlers";

export const enum ReactiveFlags {
  IS_REACTIVE = "__v_isReactive",
  IS_READONLY = "__v_isReadonly",
}

export function reactive(raw) {
  return createActiveObject(raw, mutableHandlers);
}

export function readonly(raw) {
  return createActiveObject(raw, readonlyHanders);
}

export function isReactive(value) {
  return !!value[ReactiveFlags.IS_REACTIVE];
}

export function isReadonly(value) {
  return !!value[ReactiveFlags.IS_READONLY];
}

function createActiveObject(raw, baseHandlers) {
  return new Proxy(raw, baseHandlers);
}
```

### baseHandlers.ts

```ts
import { track, trigger } from "./effect";
import { ReactiveFlags } from "./reactive";

const get = createGetter();
const set = createSetter();
const readonlyGet = createGetter(true);

function createGetter(isReadonly = false) {
  return function get(target, key) {
    if (key === ReactiveFlags.IS_REACTIVE) {
      return !isReadonly;
    } else if (key === ReactiveFlags.IS_READONLY) {
      return isReadonly;
    }

    const res = Reflect.get(target, key);

    // 通过isReadonly这个变量就可以知道当前触发的是什么类型
    if (!isReadonly) {
      track(target, key);
    }
    return res;
  };
}
function createSetter() {
  return function set(target, key, value) {
    const res = Reflect.set(target, key, value);

    trigger(target, key);
    return res;
  };
}

export const mutableHandlers = {
  get,
  set,
};

export const readonlyHanders = {
  get: readonlyGet,
  set(target, key, value) {
    console.warn("不能调用");
    return true;
  },
};
```







