# fetch

封装fetch方法

## 超时和拦截器

超时是 XMLHttpRequest 自带的功能，但是Fetch却没有

拦截器是axios里的特色功能，可以对请求前的动作和接受响应后的动作进行拦截，处理。

## 超时实现

核心就是实验Promise.race()方法，将Fetch和用Promise包裹的定时器放在数组里传入，先触发resolve的将触发Promise.race()的resolve。

所以当定时器的Promise先完成，就会直接跳出，抛出超时错误。

代码示例：

```js
if(env === 'browser' && !window.fetch) {
  try{
    // 未实现fetch的浏览器
    require('whatwg-fetch');
    this.originFetch = window.fetch;
  } catch (err) {
    throw Error('No fetch avaibale. Unable to register fetch-intercept');
  }
} else if(env === 'browser' && window.fetch) {
  this.originFetch = window.fetch.bind(window)
}
if(env === 'node') {
  // node端
  try{
    this.originFetch = require('node-fetch');
  } catch(err) {
    throw Error('No fetch avaibale. Unable to register fetch-intercept');
  }
}
this.newFetch = (url: string, opts: object): Promise<any> => {
  const fetchPromise: Promise<any> = this.originFetch(url, opts);
  const timeoutPromise: Promise<any> = new Promise(function (resolve, reject) {
    setTimeout(() => {
      reject(new Error(`Fetch Timeout ${timeout}`));
    }, timeout);
  })
  return Promise.race([fetchPromise, timeoutPromise]);
}
```



## 拦截器实现

拦截器也是使用Promise实现，将请求相关拦截器，Fetch请求，响应相关拦截器堆叠起来，实现拦截处理，如示例代码，通过Promise进行同步调用

```js
private packageIntercept(fetch: Function, ...args: any[]): Promise<any> {
  let promise = Promise.resolve(args);
	// 堆叠请求拦截器
  this.interceptArr.forEach(({ request, requestError }: InterceptFuncObj)  => {
    if (request && requestError) {
      promise = promise.then((args: any[]) => request(...args), requestError());
    } else if (request && !requestError) {
      promise = promise.then((args: any[]) => request(...args));
    } else if (!request && requestError) {
      promise = promise.then((args: any[]) => args, requestError());
    }
  })
	// 调用fetch
  promise = promise.then((args: any[]) => fetch(...args));
	// 堆叠响应拦截器
  this.interceptArr.forEach(({ response, responseError }: InterceptFuncObj)  => {
    if (response && responseError) {
      promise = promise.then((args: any[]) => response(...args), (args: any[]) => responseError(...args));
    } else if (response && !responseError) {
      promise = promise.then((args: any[]) => {
        return response(args)
      });
    } else if (!response && responseError) {
      promise = promise.then((args: any[]) => args, (e: any) => responseError(e));
    }
  })
  return promise;
}
```

## 完整代码

```js
// 拦截器方法
interface InterceptFuncObj {
  request ?: Function,
  requestError ?: Function,
  response ?: Function,
  responseError ?: Function
}
// 拦截器初始
interface InterceptInitObj {
  register : Function,
  clear : Function
}

type FetchEnv = 'browser' | 'node'

export default class Fetch {
  private interceptArr: InterceptFuncObj[] = [];
  public originFetch: Function = () => {};
  public newFetch: Function = () => {};
  public interceptor: InterceptInitObj = {
    register: () => {},
    clear: () => {}
  };
  public constructor(env: FetchEnv, timeout: number = 5000) {
    if (env === 'browser' && !window.fetch) {
      try {
        require('whatwg-fetch')
        this.originFetch = window.fetch;
      } catch (err) {
        throw Error('No fetch avaibale. Unable to register fetch-intercept');
      }
    } else if(env === 'browser' && window.fetch) {
      this.originFetch = window.fetch.bind(window)
    }
    if (env === 'node') {
      try {
        this.originFetch = require('node-fetch');
      } catch (err) {
        throw Error('No fetch avaibale. Unable to register fetch-intercept');
      }
    }
    this.newFetch = (url: string, opts: object): Promise<any> => {
      const fetchPromise: Promise<any> = this.originFetch(url, opts);
      const timeoutPromise: Promise<any> = new Promise(function (resolve, reject) {
        setTimeout(() => {
          reject(new Error(`Fetch Timeout ${timeout}`));
        }, timeout);
      })
      return Promise.race([fetchPromise, timeoutPromise]);
    }
    this.interceptor = this.init();
  }

  private init(): InterceptInitObj {
    const that = this;
    this.newFetch = (function (fetch: Function): Function {
      return function (...args: any[]): Promise<any> {
        return that.packageIntercept(fetch, ...args);
      }
    })(this.newFetch)

    return {
      register: (interceptFuncObj: InterceptFuncObj): Function => {
        this.interceptArr.push(interceptFuncObj);
        return () => { // use to unregister
          const index: number = this.interceptArr.indexOf(interceptFuncObj);
          if (index >= 0) {
            this.interceptArr.splice(index, 1);
          }
        }
      },
      clear: (): void => {
        this.interceptArr = [];
      }
    }
  }

  private packageIntercept(fetch: Function, ...args: any[]): Promise<any> {
    let promise = Promise.resolve(args);
    this.interceptArr.forEach(({ request, requestError }: InterceptFuncObj)  => {
      if (request && requestError) {
        promise = promise.then((args: any[]) => request(...args), requestError());
      } else if (request && !requestError) {
        promise = promise.then((args: any[]) => request(...args));
      } else if (!request && requestError) {
        promise = promise.then((args: any[]) => args, requestError());
      }
    })
    promise = promise.then((args: any[]) => fetch(...args));
    this.interceptArr.forEach(({ response, responseError }: InterceptFuncObj)  => {
      if (response && responseError) {
        promise = promise.then((args: any[]) => response(...args), (args: any[]) => responseError(...args));
      } else if (response && !responseError) {
        promise = promise.then((args: any[]) => {
          return response(args)
        });
      } else if (!response && responseError) {
        promise = promise.then((args: any[]) => args, (e: any) => responseError(e));
      }
    })
    return promise;
  }
}
```



## 使用方法

```js
import Fetch from '@/utils/request/Fetch.ts'

const fetch = new Fetch('browser');
const fetchInterceptor = fetch.interceptor

fetchInterceptor.register({
  request: function(...request) {
    return request;
  },
  response: function(response) {
    console.log('response', response);
    return response;
  }
})
fetch.newFetch('http://localhost:7011/api/article/getArticleType?name=1', {
  headers: new Headers({
    'content-type': 'application/json'
  })
}).then(res => res.json())
  .then(res => console.log(res))
```











