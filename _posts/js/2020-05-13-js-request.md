---
layout: post 
title: "JS 请求封装与要点注意" 
author: "fedono"
---



- 关于请求相关的概念，实现，注意等

  - XMLHttpRequest 的封装

    ```js
    var request = new XMLHttpRequest();
    request.open(method, url, true);
    request.onload = function() {
      if (this.status > 200 && this.status < 400) {
        vat data = this.response;
      }
      else {
        //... 
      }
    }
    request.onerror = function() {};
    request.setRequestHeader('Content-Type', 'application/json');
    request.send();
    ```

  - 使用HTML 的 fetch 的封装

    ```js
    fetch('http://example.com/movies.json')
      .then(function(response) {
        return response.json();
      })
      .then(function(myJson) {
        console.log(myJson);
      });
    ```

- 有做过全局的请求处理吗？比如统一请求并设置登录态，比如报错统一弹 toast

  - 正常考虑返回的状态，如返回的数据是否有 `status === 200 / success === true`  或者查看 `headers['content-length']` 如果等于0就说明没有内容返回，则使用 Promise.reject 返回
  - 其他的异常也统一返回 reject 

- 比如给请求建立缓存，在设定多少时间内，直接返回之前的结果，而不是继续发起请求

  参考一下 [amis](http://github.com/baidu/amis) 中设置的缓存处理

  ```js
  // src/utils/api.ts:326
  export function getApiCache(api: ApiObject): ApiCacheConfig | undefined {
    // 清理过期cache
    const now = Date.now();
    let result: ApiCacheConfig | undefined;
  
    for (let idx = 0, len = apiCaches.length; idx < len; idx++) {
      const apiCache = apiCaches[idx];
  
      if (now - apiCache.requestTime > (apiCache.cache as number)) {
        apiCaches.splice(idx, 1);
        len--;
        idx--;
        continue;
      }
  
      if (isSameApi(api, apiCache)) {
        result = apiCache;
        break;
      }
    }
  
    return result;
  }
  
  export function setApiCache(
    api: ApiObject,
    promise: Promise<any>
  ): Promise<any> {
    apiCaches.push({
      ...api,
      cachedPromise: promise,
      requestTime: Date.now()
    });
    return promise;
  }
  ```

- 给 XHR 添加 hook，实现在各个阶段打印日志

  > 实现页面上通过 xhr 发请求的时候，在 xhr 的生命周期里，能够实现自定义的行为触发。
  >
  > 遇到这种问题，一般都是重写 XHR 请求，然后劫持各个事件，然后在事件中进行触发 hook，类似 Object.defineProperty 中的 get/set 一样

  ```js
  
  /*
  * 1. class 的使用，new 对象
  * 2. this 指向
  * 3. apply, call 的运用
  * 4. Object.defineProperty 的应用
  * 5. 代码的设计能力
  * 6. hook 的理解
  * */
  
  /*
  * 总体的逻辑
  * 1. 重写原生的 XMLHttpRequest 方法
  * 2. 获取到原生的方法和属性进行劫持
  * 3. 在原生的方法中增加beforeHooks 和 afterHooks
  * 4. 使用 Object.defineProperty 重写属性
  * */
  class XhrHook {
      constructor(beforeHooks = {}, afterHooks = {}) {
          this.XHR = window.XMLHttpRequest;
          this.beforeHooks = beforeHooks;
          this.afterHooks = afterHooks;
          this.init();
      }
  
      init() {
          let _this = this;
          // 这里使用 function 而不是使用箭头函数，是因为每次使用 XMLHttpRequest 都是使用 new
          // 使用箭头函数会改变 this 指向
          window.XMLHttpRequest = function() {
              this._xhr = new _this.XHR();
              _this.overwrite(this);
          }
      }
  
      overwrite(proxyXHR) {
          for (let key in proxyXHR._xhr) {
              if (typeof proxyXHR._xhr[key] === 'function') {
                  this.overwriteMethod(key, proxyXHR);
                  continue;
              }
  
              this.overwriteAttributes(key, proxyXHR);
          }
      }
  
      overwriteMethod(key, proxyXHR) {
          let beforeHooks = this.beforeHooks;
          let afterHooks = this.afterHooks;
  
          proxyXHR[key] = (...args) => {
              // beforeHooks 拦截如果返回 false，那么就不会执行原有的函数了
              if (beforeHooks[key]) {
                  const res = beforeHooks[key].call(proxyXHR, args);
                  if (res === false) {
                      return;
                  }
              }
  
              const res = proxyXHR._xhr[key].apply(proxyXHR._xhr, args);
              // afterHooks 中需要把原生的函数执行的结果传进来
              afterHooks[key] && afterHooks[key].call(proxyXHR._xhr, res);
  
              return res;
          }
      }
  
      overwriteAttributes(key, proxyXHR) {
          Object.defineProperty(proxyXHR, key, this.setPropertyDescriptor(key, proxyXHR));
      }
  
      setPropertyDescriptor(key, proxyXHR) {
          let obj = Object.create(null);
          let _this = this;
  
          obj.set = function(val) {
              // 不是 onreadystatechange/onload 这种属性，而是函数
              if (!key.startsWith('on')) {
                  proxyXHR['__' + key] = val;
                  return;
              }
  
              if (_this.beforeHooks[key]) {
                  this._xhr[key] = function (...args) {
                      _this.beforeHooks[key].call(proxyXHR);
                      val.apply(proxyXHR, args);
                  }
                  return;
              }
  
              this._xhr[key] = val;
          }
  
          obj.get = function() {
              return proxyXHR['__' + key] || this._xhr[key];
          }
  
          return obj;
      }
  }
  ```

  



## 参考

- [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html) 
- [浏览器同源政策及其规避方法](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html) 
- fetch 

