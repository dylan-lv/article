# shallowReadonly和isProxy

## shallowReadonly

这个函数其实就是 `shallow` 和 `readonly` 两个的结合体

这种形式一般用于做程序中的优化，防止把全部的对象都转换成响应式对象了。

- shallow：表层
  - 最外层是响应式对象，内部就不是响应式对象了

我们先写一下测试代码：

```ts
// src5\reactivity\test\shallowReadonly.spec.ts
describe("shallowReadonly", () => {
  test("should not make non-reactive properties reactive", () => {
    // 创建出的props只能是最外层的对象是响应式对象，内部的（例如n）就不是响应式对象了。
    // 创建出的响应式对象是 Readonly 类型
    // 也就是 表层是 readonly，内部嵌套的都是正常的
    // 这种形式一般用于做程序中的一些优化，不然的话它会把所有的对象都转化成响应式对象
    const props = shallowReadonly({ n: { foo: 1 } });
    expect(isReadonly(props)).toBe(true);
    expect(isReadonly(props.n)).toBe(false);
  });
  // 这里和 readonly 是一致的
  it("warn then call set", () => {
    console.warn = jest.fn();
    const user = shallowReadonly({
      age: 10,
    });
    user.age = 11;
    expect(console.warn).toBeCalled();
  });
});
```



### 实现

```ts
// reactivity\reactive.ts
export function shallowReadonly(raw) {
  return createActiveObject(raw, shallowReadonlyHanders);
}
```

```ts
// reactivity\baseHandlers.ts
const shallowReadonlyGet = createGetter(true, true);
function createGetter(isReadonly = false, shallow = false) {
  return function get(target, key) {
    if (key === ReactiveFlags.IS_REACTIVE) {
      return !isReadonly;
    } else if (key === ReactiveFlags.IS_READONLY) {
      return isReadonly;
    }

    const res = Reflect.get(target, key);

    // !如果是 shallow 类型的话，下面的嵌套就不要再执行了
    if (shallow) {
      return res;
    }

    if (isObject(res)) {
      return isReadonly ? readonly(res) : reactive(res);
    }

    if (!isReadonly) {
      track(target, key);
    }
    return res;
  };
}
// set和readonlyHanders的set是一样的，所以这里直接使用 extend 扩展
export const shallowReadonlyHanders = extend({}, readonlyHanders, {
  get: shallowReadonlyGet,
});
```



## isProxy

先加一下单元测试

```ts
// reactivity\test\reactive.spec.ts
describe("reactive", () => {
  it("happy path", () => {
    const original = { foo: 1 };
    const observed = reactive(original);
    ...
    // 使用 isProxy检测对象
    expect(isProxy(observed)).toBe(true);
  });
});
// reactivity\test\readonly.spec.ts
describe.skip("readonly", () => {
  it("happy path", () => {
    const original = { foo: 1, bar: { baz: 2 } };
    const wrapped = readonly(original);
    ...
    // 使用 isProxy检测对象
    expect(isProxy(wrapped)).toBe(true);
  });
});
```

### 实现

```ts
// reactivity\reactive.ts
// 检测 object 是不是通过 reactive 或者 readonly 创建的
export function isProxy(value) {
  return isReactive(value) || isReadonly(value);
}
```





