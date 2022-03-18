# proxyRefs

### 测试代码

```ts
describe("ref", () => {
  it("proxyRefs", () => {
    // user中的age是一个 ref 类型
    const user = {
      age: ref(18),
      name: "xiaohong",
    };
    // ref类型只要给到proxyRefs之后，我们在后续去访问里面 ref 类型的时候，就可以省略.value了
    // 场景：使用在 template 里面，setup可能会返回 ref 值，但是在template里面不需要 .value
    const proxyUser = proxyRefs(user);
    expect(user.age.value).toBe(18);
    expect(proxyUser.age).toBe(18);
    expect(proxyUser.name).toBe("xiaohong");

    // set逻辑
    proxyUser.age = 20;
    expect(proxyUser.age).toBe(20);
    expect(user.age.value).toBe(20);

    proxyUser.age = ref(10);
    expect(proxyUser.age).toBe(10);
    expect(user.age.value).toBe(10);
  });
});
```

我们从单测中来了解它的作用

- 首先，创建了一个 `user` 对象
  - 对象中有个 `age`，它是 `ref` 类型
- `user` 通过 `proxyRefs` 赋值给 `proxyUser`后，`proxyUser` 就可以不通过 `.value` ，直接访问到值了
- 赋值 `set` 的话，原先的值也会改变（响应式）

一般使用在 `template` 中：

我们在使用 `vue3` 的时候，`setup` 函数可以返回 `ref` 类型的数据，但是在 `template` 中使用的时候并不需要通过 `.value` 来访问



### 实现

- get（代理 get、set 可以使用 Proxy）
  - 如果我们访问的是 `age` ，发现它是一个 `ref` 类型，就返回 `.value`
  - 否则不是一个`ref`类型的话，就直接返回值

```ts
// reactivity\ref.ts
export function proxyRefs(objectWithRefs) {
  return new Proxy(objectWithRefs, {
    get(target, key) {
      // get -> age(ref) 那么就返回 .value，不是ref的话就返回本身的值
      return unRef(Reflect.get(target, key));
    },
  });
}
```

- set 
  - 如果 `proxyUser.age` 被赋值（set）了，那么 `proxyUser.age` 和 `user.age.value` 这两个值同时会发生改变
  - 看看被赋值对象是不是一个 `ref` 类型
    - 如果是的话，就修改它的 `.value` （赋值的对象不是一个 `ref` 类型）
    - 否则直接覆盖
    - rawObj（ref） -> setObj（ref）：覆盖
    - rawObj（ref） -> setObj（no-ref）：赋值 `.value`
    - rawObj（no-ref） -> setObj（ref）：覆盖
    - rawObj（no-ref） -> setObj（no-ref）：覆盖

```ts
export function proxyRefs(objectWithRefs) {
  return new Proxy(objectWithRefs, {
    get(target, key) {
      // get -> age(ref) 那么就返回 .value，不是ref的话就返回本身的值
      return unRef(Reflect.get(target, key));
    },
    set(target, key, value) {
      // 如果是ref类型，则修改 .value
      if (isRef(target[key]) && !isRef(value)) {
        // 之前是 ref，现在不是ref。替换
        return (target[key].value = value);
      } else {
        return Reflect.set(target, key, value);
      }
    },
  });
}
```











