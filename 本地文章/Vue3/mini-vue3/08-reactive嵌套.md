# reactive嵌套

reactive和readonly对象中嵌套有其他对象。

先来写一下测试文件

```ts
// reactivity\test\reactive.spec.ts
describe("reactive", () => {
  ...
  test("nested reactive", () => {
    // 嵌套了其他的 object
    const original = {
      nested: {
        foo: 1,
      },
      array: [{ bar: 2 }],
    };
    const observed = reactive(original);
    // 看看里面的值，是否是 reactive 的
    expect(isReactive(observed.nested)).toBe(true);
    expect(isReactive(observed.array)).toBe(true);
    expect(isReactive(observed.array[0])).toBe(true);
  });
});
// reactivity\test\readonly.spec.ts
describe("readonly", () => {
  it("happy path", () => {
    const original = { foo: 1, bar: { baz: 2 } };
    const wrapped = readonly(original);
    expect(wrapped).not.toBe(original);
    expect(wrapped.foo).toBe(1);

    expect(isReadonly(wrapped)).toBe(true);
    expect(isReadonly(original)).toBe(false);

    // readonly嵌套
    expect(isReadonly(wrapped.bar)).toBe(true);
    expect(isReadonly(original.bar)).toBe(false);
  });
});
```

`reactive、readonly` 对象内部的对象，也需要是 `reactive、readonly` 的

- 功能实现

```ts
// reactivity\baseHandlers.ts
function createGetter(isReadonly = false, shallow = false) {
  return function get(target, key) {
    if (key === ReactiveFlags.IS_REACTIVE) {
      return !isReadonly;
    } else if (key === ReactiveFlags.IS_READONLY) {
      return isReadonly;
    }
    const res = Reflect.get(target, key);

    // 处理嵌套逻辑：看看 res 是不是一个 object
    if (isObject(res)) {
      return isReadonly ? readonly(res) : reactive(res);
    }

    if (!isReadonly) {
      track(target, key);
    }
    return res;
  };
}
```

```ts
// shared\index.ts
export const isObject = (value) => {
  return value !== null && typeof value === "object";
};
```

