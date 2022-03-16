# readonly和代码重构

### 先写readonly的单测

```ts
// reactivity/test/readonly.spec.ts
describe("readonly", () => {
  it("happy path", () => {
    // readonly和reactive没太大区别，但是它不能被 set（不会触发依赖）
    const original = { foo: 1, bar: { baz: 2 } };
    const wrapped = readonly(original);
    expect(wrapped).not.toBe(original);
    expect(wrapped.foo).toBe(1);
  });

  it("warn then call set", () => { // 被set时报警告，并且没有被赋值
    console.warn = jest.fn();
    const user = readonly({
      age: 10,
    });
    user.age = 11;
    expect(console.warn).toBeCalled();
  });
});
```



### 实现readonly

readonly 其实就是 reactive 只读方法的实现：

```ts
// reactivity\reactive.ts
export function readonly(raw) {
  return new Proxy(raw, {
    get(target, key) {
      const res = Reflect.get(target, key);
      return res;
    },
    set(target, key, value) {
      console.warn(`readonly不能赋值：${target}`)
      return true
    },
  });
}
```

这样的话，readonly 就实现了。



### 重构代码逻辑

先看下我们的 reactive.ts 文件，可以发现 reactive 方法和 readonly 方法很类似，这里我们可以抽离一下。

我们先把 get 抽离出来

```ts
function createGetter(isReadonly = false) {
  return function get(target, key) {
    const res = Reflect.get(target, key);
    if(!isReadonly) {
      track(target, key);
    }
    return res;
  }
}
```

抽离出 createGetter 函数后，就可以用它来代替我们 `reactive、readonly` 之前的逻辑了

```ts
export function reactive(raw) {
 	return new Proxy(raw, {
    get: createGetter(),
    set(target, key, value) {
      const res = Reflect.set(target, key, value);
      trigger(target, key);
      return res;
    },
  }) 
}
export function readonly(raw) {
  return new Proxy(raw, {
    get: createGetter(true),
    set(target, key, value) {
      console.warn(`readonly不能赋值：${target}`)
      return true
    },
  });
}
```

我们处理完了 get，接下来应该怎么做呢？

在程序里面有一个相对性，他就是要保持我们代码的一致性。我们既然把 `get` 抽离出来了，那么也把 `set` 抽离出来

```ts
function createSetter() {
  return function set(target, key, value) {
    const res = Reflect.set(target, key, value);
    trigger(target, key);
    return res;
  }
}
```

此时我们的原逻辑就变为：

```ts
export function reactive(raw) {
 	return new Proxy(raw, {
    get: createGetter(),
    set: createSetter(),
  }) 
}
export function readonly(raw) {
  return new Proxy(raw, {
    get: createGetter(true),
    set(target, key, value) { // 这里的 set 不需要抽离，因为它和上面的 set 有一定的区别
      console.warn(`readonly不能赋值：${target}`)
      return true
    },
  });
}
```

接下来，我们还可以继续进行抽象（代码抽离）

比如 `{ get: createGetter(), set: createSetter() }` 这个对象是专门处理 `reactive` 的

我们创建一个 `baseHandlers.ts` 文件，将这个对象以及 `createGetter、createSetter` 逻辑放进去

```ts
// \reactivity\baseHandlers.ts
function createGetter(isReadonly = false) {
  return function get(target, key) {
    const res = Reflect.get(target, key);
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
  get: createGetter(),
  set: createSetter(),
};
export const readonlyHanders = {
  get: createGetter(true),
  set(target, key, value) {
    console.warn('不能调用')
    return true;
  },
};
```

可以看到我们每次调用 `mutableHandlers` 时，都会重新创建 `get、set`。但是对于我们的程序而已，没必要每次都重新创建。这里我们可以使用缓存的方式：

```ts
const get = createGetter()
const set = createSetter()
const readonlyGet = createGetter(true)
export const mutableHandlers = {
  get,
  set
};
export const readonlyHanders = {
  get: readonlyGet,
  set(target, key, value) {
    console.warn('不能调用')
    return true;
  },
};
```

这样的话，我们的 `reactive.ts` 文件就干净多了

```ts
export function reactive(raw) {
 	return new Proxy(raw, mutableHandlers); 
}
export function readonly(raw) {
 	return new Proxy(raw, readonlyHanders); 
}
```

其实我们这里的代码是**低层次**的代码，它可以用函数封装一下来表示出我们要表达的意图，让代码可读写更高一些

```ts
export function reactive(raw) {
  return createActiveObject(raw, mutableHandlers);
}
export function readonly(raw) {
  return createActiveObject(raw, readonlyHanders);
}
function createActiveObject(raw, baseHandlers) {
  return new Proxy(raw, baseHandlers);
}
```



## 全部代码如下：

### readonly.spec.ts

```ts
import { readonly } from "../reactive";

describe("readonly", () => {
  it("happy path", () => {
    // readonly和reactive没太大区别，但是它不能被 set（不会触发依赖）
    const original = { foo: 1, bar: { baz: 2 } };
    const wrapped = readonly(original);
    expect(wrapped).not.toBe(original);
    expect(wrapped.foo).toBe(1);
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
import { track, trigger } from "./effect";

// 高阶函数（移走了）
// function createGetter(isReadonly = false) {
//   return function get(target, key) {
//     const res = Reflect.get(target, key);

//     if(!isReadonly) {
//       track(target, key);
//     }
//     return res;
//   }
// }
// function createSetter() {
//   return function set(target, key, value) {
//     const res = Reflect.set(target, key, value);

//     trigger(target, key);
//     return res;
//   }
// }

export function reactive(raw) {
  return createActiveObject(raw, mutableHandlers);
  // return new Proxy(raw, mutableHandlers);

  // return new Proxy(raw, {
  //   // get(target, key) {
  //   //   const res = Reflect.get(target, key);

  //   //   track(target, key);
  //   //   return res;
  //   // },
  //   get: createGetter(),
  //   // set(target, key, value) {
  //   //   const res = Reflect.set(target, key, value);

  //   //   trigger(target, key);
  //   //   return res;
  //   // },
  //   set: createSetter()
  // });
}

export function readonly(raw) {
  return createActiveObject(raw, readonlyHanders);
  // return new Proxy(raw, readonlyHanders);

  // return new Proxy(raw, {
  //   // get(target, key) {
  //   //   const res = Reflect.get(target, key);

  //   //   return res;
  //   // },
  //   get: createGetter(true),
  //   set(target, key, value) {
  //     console.warn(`111`)
  //     return true
  //   },
  // });
}

function createActiveObject(raw, baseHandlers) {
  return new Proxy(raw, baseHandlers);
}
```

### baseHandlers.ts

```ts
import { track, trigger } from "./effect";

const get = createGetter()
const set = createSetter()
const readonlyGet = createGetter(true)

// 高阶函数
function createGetter(isReadonly = false) {
  return function get(target, key) {
    const res = Reflect.get(target, key);
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
  // get: createGetter(),
  get,
  // set: createSetter(),
  set
};

export const readonlyHanders = {
  // get: createGetter(true),
  get: readonlyGet,
  set(target, key, value) {
    console.warn('不能调用')
    return true;
  },
};
```

















