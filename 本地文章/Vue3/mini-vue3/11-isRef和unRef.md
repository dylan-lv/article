# isRef和unRef

- isRef：用来判断响应式对象是否是 `ref` 类型
- unRef：如果是 `ref` 则返回 `ref.value` 的值；否则返回当前参数值

### 测试文件

我们先补充一下测试

```ts
// reactivity\test\ref.spec.ts
describe("ref", () => {
  it("isRef", () => {
    const a = ref(1);
    const user = reactive({
      age: 1,
    });
    expect(isRef(a)).toBe(true);
    // 值类型是不可能有__v_isRef的，返回的是 undefined。需要使用 !!
    expect(isRef(1)).toBe(false); 
    expect(isRef(user)).toBe(false);
  });
  it("unRef", () => {
    const a = ref(1);
    expect(unRef(a)).toBe(1);
    expect(unRef(1)).toBe(1); 
  })
});
```



### 实现 isRef

我们想要判断一个对象是否是 `ref`，只需要给 `ref` 对象一个标识（__v_isRef）就可以了。

我们在创建 `ref` 对象的时候，给 `ref` 对象一个标识

```ts
class RefImpl {
  public __v_isRef = true; // 给它一个标识，说明自己是 ref
  ...
}
```

后续进行判断是否是 `ref` 的时候（isRef），就可以使用这个标识了

如果是一个**普通值**，则 `ref.__v_isRef` 会返回 `undefined` ，我们使用 `!!` 对其进行转换

```ts
// reactivity\ref.ts
export function isRef(ref) {
  return !!ref.__v_isRef;
}
```



### 实现 unRef

```ts
// reactivity\ref.ts
export function unRef(ref) {
  // 看看是不是 ref 对象，如果是就返回 ref.value ，否则直接返回值（ref）
  return isRef(ref) ? ref.value : ref;
}
```







