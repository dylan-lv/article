# Promise实现并发控制

`题目`：输入url数组与并发数，实现一个并发数量控制的请求队列函数

**思路：**

控制并发数量，关键点就是利用promise，当一个请求完成后，去发起下一个请求。

1. 由于要完成一个继续下一个，并保证同时有limit个任务在同时执行。所以我们需要一个递归的执行函数run;
2. 首先在函数执行时，需要将执行中的任务队列按顺序填满，数量为limit；
3. 执行函数中，需要在执行完成后，判断是否有下一个待执行的任务；这里我们声明变量 `i` 来计数，与总任务数对比判断是否需要递归执行下一个任务。

**代码实现：**

```js
const request = url => {
  // 实际场景这里用axios等请求库发请求即可，这里使用定时器模拟延时
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('完成一个任务', url, new Date());
      resolve({ url, date: new Date() })
    }, 1000);
  })
}

function limitQueue(urls, limit) {
  // 已完成任务数
  let i = 0;
  // 填充执行队列
  for(let excuteCount = 0; excuteCount < limit; excuteCount++) {
    run();
  }

  // 执行一个任务
  function run() {
    // 构造待执行任务，当该任务完成后，如果还有待完成的任务，则继续执行下一个任务
    new Promise((resolve) => {
      const url = urls[i++];
      resolve(request(url))
    }).then(() => {
      if(i < urls.length) run()
    })
  }
}
```

**测试代码：**

```js
const urls = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15];

(async () => {
  await limitQueue(urls, 4);
})()
```







