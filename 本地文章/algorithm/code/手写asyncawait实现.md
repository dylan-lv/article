# 手写async/await实现

 async 函数是 generator 函数的语法糖。

既然用了 generator 函数，何必还要实现 async 呢？

async 和 generator 之间到底是如何相互协作，管理异步的？

### 示例

```js
const getData = () => new Promise(resolve => 
  setTimeout(() => resolve("data"), 1000)  
)
async function test() {
  const data = await getData()
  console.log('data:', data);
  const data2 = await getData()
  console.log('data2:', data2);
  return 'success'
}

// 这样一个函数：应该在1秒后打印data，再过1秒打印data2，最后打印 success
test().then(res => console.log(res))
```

### 思路

对于这个简单的案例来说，如果我们把它用 generator 函数表达，会是怎么样的呢？

```js
function * testG() {
  // await 被编译成了yield
  const data = yield getData();
  console.log('data:', data);
  const data2 = yield getData();
  console.log('data2:', data2);
  return 'success'
}
```

我们知道，generator函数是不会自动执行的，每一次调用它的next方法，会停留在下一个 yield 的位置。

利用这个特性，我们只要编写一个自动执行的函数，就可以让这个 generator 函数完全实现 async 函数的功能。

```js
const getData = () => new Promise(resolve => 
  setTimeout(() => resolve("data"), 1000)  
)
const test = asyncToGenerator(
  function * testG() {
    // await 被编译成了yield
    const data = yield getData();
    console.log('data:', data);
    const data2 = yield getData();
    console.log('data2:', data2);
    return 'success'
  }
)

test().then(res => console.log(res))
```

那么大体上的思路已经确定了：

asyncToGenerator 接受一个 generator 函数，返回一个 promise，

关键点在于，里面用 yield 来划分的异步流程，应该如何自动执行。



### 如果是手动执行

在编写这个函数之前，我们先模拟手动去调用这个`generator`函数去一步步的把流程走完，有助于后面的思考。

```js
function * testG() {
    // await 被编译成了yield
    const data = yield getData();
    console.log('data:', data);
    const data2 = yield getData();
    console.log('data2:', data2);
    return 'success'
  }
```

我们先调用 testG 生成一个迭代器

```js
let gen = testG()
```

然后开始执行第一次next

```js
// 第一次调用next，停留在第一个yield的位置
// 返回的promise里，包含了data需要的数据
let dataPromise = gen.next()
```

这里返回了一个promise，就是第一次getData()所返回的promise，注意

```js
const data = yield getData()
```

这段代码要切割成两部分来看，第一次调用next，其实只是停留在了 yield getData()这里，

data的值并没有被确定。

那么神秘时候data的值会被确定呢？

**下一次调用next的时候，传的参数会被作为上一个yield签名接受的值**

也就是说，我们再次调用 gen.next('这个参数才会被赋给data变量') 的时候

data的值才会被确定为 '这个参数才会被赋给data变量'

```js
gen.next('这个参数才会被赋给data变量')

// 然后这里的data才有值
const data = yield getData()
// 然后打印出data
console.log('data:', data)
// 然后继续走到下一个yield
const data2 = yield getData()
```

然后往下执行，直到遇到下一个yield，继续这样的流程

这是 generator 函数设计的一个比较难理解的点，但是为了实现我们的目标，还是得去学习它。

借住这个特性，如果我们这样去控制 yield 的流程就能实现异步串行了。

```js
const getData = () => new Promise(resolve => 
  setTimeout(() => resolve("data"), 1000)  
)

function * testG() {
  // await被编译成了yield
  const data = yield getData()
  console.log('data: ', data);
  const data2 = yield getData()
  console.log('data2: ', data2);
  return 'success'
}

let gen = testG()
let dataPromise = gen.next()

dataPromise.value.then((value1) => {
  // data1的value被拿到了 继续调用next并且传递给data
  var data2Promise = gen.next(value1)
  
  // console.log('data: ', data);
  // 此时就会打印出data
  
  data2Promise.value.then((value2) => {
      // data2的value拿到了 继续调用next并且传递value2
       gen.next(value2)
       
      // console.log('data2: ', data2);
      // 此时就会打印出data2
  })
})
```

这样的一个看着像 callback 的调用，就可以让我们的 generator 函数把异步安排的明明白白。



### 实现

有了这样的思路，实现这个高阶函数就变得很简单了。

先看一下整体结构

```js
function asyncToGenerator(generatorFunc) {
  return function() {
    const gen = generatorFunc.apply(this, arguments)
    return new Promise((resolve, reject) => {
      function step(key, arg) {
        let generatorResult
        try {
          generatorResult = gen[key](arg)
        } catch(error) {
          return reject(error)
        }
        const { value, done } = generatorResult
        if(done) {
          return resolve(value)
        } else {
          return Promise.resolve(value).then(val => step('next', val), err => step('throw', err))
        }
      }
      step('next')
    })
  }
}
```

接下来逐行讲解

```js
function asyncToGenerator(generatorFunc) {
  // 返回的是一个新的函数
  return function() {
    // 先调用generator函数，生成迭代器
    // 对应 let gen = testG()
    const gen = generatorFunc.apply(this, arguments);
    // 返回一个promise，因为外部是用 .then 的方式，或者 await 的方式去适应这个函数的返回值的
    // let test = asyncToGenerator(testG)
    // test().then(res => console.log(res))
    return new Promise((resolve, reject) => {
      // 内部定义一个step函数，用来一步步的跨过yield的阻碍
      // key有next和throw两种取值，分别对应了gen的next和throw方法
      // arg参数则是用来把promise resolve出来的值交给下一个yield
      function step(key, arg) {
        let generatorResult
        // 这个方法需要包裹在 try catch 中
        // 如果报错了，就把promise给reject掉，外部通过 .catch 可以捕获错误
        try {
          generatorResult = gen[key](arg);
        } catch (error) {
          return reject(error);
        }

        // gen.next() 得到的结果是一个 { value, done } 的结构
        const { value, done } = generatorResult;

        if(done) {
          // 如果已经完成了，就直接resolve这个promise
          // 这个done是在最后一次调用next后才会为true
          // 以本文的例子来说，此时的结果是 { done: true, value: 'success' }
          // 这个value也就是 generator 函数最后的返回值
          return resolve(value);
        } else {
          // 除了最后结束的时候外，每次调用gen.next()
          // 其实是返回 { value: Promise, done: false } 的结构
          // 这里要注意的是 Prommise.resolve 可以接受一个 promise 为参数
          // 并且这个primise参数被resolve的时候，这个then才会被调用
          return Promise.resolve(
            // 这个value对应的是yield后面的promise
            value
          ).then(
            // value这个promise被resolve的时候，就会执行next
            // 并且只要done不是true的时候，就会递归的往下解开promise
            // 对应 gen.next().value.then(value => gen.next())
            function onResolve(val) {
              step("next", val)
            },
            // 如果promise被reject了，就再次进入step函数
            // 不同的是，这次try catch中调用的是gen.throw(err)
            // 那么自然就被catch掉，然后把promise给reject了
            function onReject(err) {
              step("throw", err)
            }
          )
        }
      }
      step("next")
    })
  }
}
```



本文用最简单的方式实现了 asyncToGenerator 这个函数，这是babel编译async函数的核心，当然在babel中，generator函数也被编译成了一个很原始的形式，本文我们直接以generator替代。

























