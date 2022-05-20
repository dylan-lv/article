# computed

计算属性的使用和 `ref` 很类似，调用的时候都是 `.value`。但是它拥有 “缓存” 这个强大的功能



## 测试：happy path

先写一个 `happy path` 单测，实现 `computed` 基本功能

```ts
// reactivity\test\computed.spec.ts
describe("computed", () => {
  it("happy path", () => {
    const user = reactive({
      age: 1,
    });
    const age = computed(() => {
      return user.age;
    });
    expect(age.value).toBe(1);
  });
});
```



## 实现：happy path

```ts
// reactivity\computed.ts
class ComputedRefImpl {
  private _getter: any
  constructor(getter) {
    this._getter = getter;
  }
  get value() {
    return this._getter();
  }
}
export function computed(getter) {
  return new ComputedRefImpl(getter);
}
```

- 保存其 `getter` 方法
- 每次外部调用 `.value` 都返回 `getter` 的返回值

此时，`happy path` 单测就可以通过了



## 测试：lazy

我们来添加测试代码

```ts
// reactivity\test\computed.spec.ts
describe("computed", () => {
  it("should compute lazily", () => {
    // 计算属性可以缓存
    const value = reactive({
      foo: 1,
    });
    const getter = jest.fn(() => {
      return value.foo;
    });
    const cValue = computed(getter);

    // 懒执行：如果没有调用cValue.value的话，getter不会调用
    expect(getter).not.toHaveBeenCalled();

    expect(cValue.value).toBe(1);
    expect(getter).toHaveBeenCalledTimes(1);

    // 当我们再次触发get操作，需要验证 getter 还是只被调用了一次
    cValue.value;
    expect(getter).toHaveBeenCalledTimes(1);

    // 当我们响应式的值发生改变了
    value.foo = 2; // set 触发 trigger -> effect -> get 重新执行
    expect(getter).toHaveBeenCalledTimes(1);

    expect(cValue.value).toBe(2);
    expect(getter).toHaveBeenCalledTimes(2);

    cValue.value;
    expect(getter).toHaveBeenCalledTimes(2);
  });
});
```



### 懒执行

如果没有调用cValue.value的话，getter不会调用

```ts
expect(getter).not.toHaveBeenCalled();
expect(cValue.value).toBe(1);
expect(getter).toHaveBeenCalledTimes(1);
```

此时这三行代码的单测，都是可以直接通过的。

- 当我们再次触发get操作，需要验证 getter 还是只被调用了一次

```ts
cValue.value;
expect(getter).toHaveBeenCalledTimes(1);
```

我们再次调用 `cValue.value`，希望 `getter` 函数还是只被执行了一次；

此时测试会报错，告诉我们被执行了两次。

解决方法：调用完 `getter` 后，将其锁上

```ts
// reactivity\computed.ts
class ComputedRefImpl {
  private _getter: any
  private _dirty: boolean = true;
  constructor(getter) {
    this._getter = getter;
  }
  get value() {
    if(this._dirty) { // 调用完 get 后，将其锁上
     	this._dirty = false;
      this._value = this._getter();
    }
    return this._value;
  }
}
export function computed(getter) {
  return new ComputedRefImpl(getter);
}
```



### 响应式的值发生改变

```ts
value.foo = 2; // set 触发 trigger -> effect -> get 重新执行
expect(getter).toHaveBeenCalledTimes(1); // getter 没有再次被调用

expect(cValue.value).toBe(2); // 直到访问了 .value 属性，getter才再次被调用
expect(getter).toHaveBeenCalledTimes(2);

cValue.value; // 再次访问 .value 属性，此时 getter 又被锁上了
expect(getter).toHaveBeenCalledTimes(2);
```

- 响应式对象调用 set 后，会触发 trigger

```ts
class ComputedRefImpl {
  private _getter: any;
  private _dirty: boolean = true;
  private _value: any;
  private _effect: ReactiveEffect;
  constructor(getter) {
    this._getter = getter;

    // 这里的 _effect 已经被依赖收集了
    // 当响应式对象发生了改变，会触发 trigger，进而调用 this._effect.run 方法
    // 此时会调用 scheduler 方法（ReactiveEffect的第二个参数）
    this._effect = new ReactiveEffect(getter, () => { 
      if (!this._dirty) {
        this._dirty = true;
      }
    });
  }
  get value() {
    // 当依赖的响应式对象发生改变的时候，需要把 _dirty 改为 true
    // 调用完一次 get 之后就锁上
    if (this._dirty) {
      this._dirty = false;
      // this._value = this._getter();
      this._value = this._effect.run();
    }

    return this._value;
  }
}
export function computed(getter) {
  return new ComputedRefImpl(getter);
}
```



## 总结

- 计算属性的内部是有一个 `effect` 的，并且它有 `get value` 方法，
- 当用户调用 `get value` 的时候，`computed` 会调用 `effect.run`，也就是把用户传入的 `getter` 函数运行后产生的值返回出去
- 使用 `dirty` 实现缓存的能力，`dirty` 表示用户传入的 `getter` 是否有被调用过，如被调用则把自己修改为 `false`，并且保存此次得到的值（value），之后调用一直都会返回缓存的 `value`
- 当内部依赖的响应式对象发生改变后，会触发 `trigger`，由于这里设置了 `scheduler`，则 `trigger` 只会执行 `scheduler`方法
  - 在这里的 `scheduler` 方法中，只会把 `dirty` 设置为 `true`
  - 这样当用户之后再调用 `get value` 时，就可以得到最新的值了



## 完整代码

[github地址](https://github.com/hayeslv/mini-vue/tree/version/src9)



