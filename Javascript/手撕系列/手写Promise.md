# 手写promise

```javascript
class myPromise {
  static PENDING = "pending";
  static FULFILLED = "fulfilled";
  static REJECTED = "rejected";
  constructor(executor) {
    this.status = myPromise.PENDING;
    this.value = null;
    // 定义数组，把需要异步执行的函数压入栈中
    this.callbacks = [];
    try {
      /** class中默认是严格模式，executor的执行上下文是window
       *  不加bind的话，resolve的执行上下文是undefined，无法找到this.status
       * */
      executor(this.resolve.bind(this), this.reject.bind(this));
    } catch (error) {
      // 如果executor中执行出错，则应该执行拒绝
      this.reject(error);
    }
  }
  resolve(value) {
    // 2.保证一旦状态改变，就不会再变
    if (this.status == myPromise.PENDING) {
      this.status = myPromise.FULFILLED;
      this.value = value;
      /**
       * 4. 如果在promise中有setTimeout(()=>{resolve(); console.log('nihao')})
       * 保证nihao比resolve先执行，就把resolve放入setTimeout中异步执行
       */
      setTimeout(() => {
        // 4. 一段时间后，异步状态得到改变，再取出栈中回调函数的实参，然后执行，还要把value值传入
        this.callbacks.map((callback) => {
          callback.onFulfilled(value);
        });
      });
    }
  }
  // 2.保证一旦状态改变，就不会再变
  reject(reason) {
    if (this.status == myPromise.PENDING) {
      this.status = myPromise.REJECTED;
      this.value = reason;
      setTimeout(() => {
        this.callbacks.map((callback) => {
          callback.onRejected(reason);
        });
      });
    }
  }
  then(onFulfilled, onRejected) {
    // then方法可以接受两个回调函数作为参数。这两个函数都是可选的，不一定要提供。
    if (typeof onFulfilled != "function") {
      // 7. 支持值得穿透，直接把this.value向下传递
      onFulfilled = () => this.value;
    }
    if (typeof onRejected != "function") {
      onRejected = () => this.value;
    }
    /**
     * 5.then返回promise，且支持链式调用
     * 且若前一个promise的状态是拒绝的，并不会影响下一个then的状态
     */
    let promise = new myPromise((resolve, reject) => {
      /**
       * 4.若promise内部是存在setTimeout的异步操作，状态是在一段时间后才改变，
       * 此时状态为pending, 需要把then的回调函数实参压入栈中
       *  */
      if (this.status == myPromise.PENDING) {
        this.callbacks.push({
          onFulfilled: (value) => {
            // 5.onFulfilled是前一个promise状态改变后的回调方法
            // 5.resolve和reject是当前promise状态改变后的回调方法
            // 9.promise是同步代码，已经被声明了，在此时传入函数时，为pending状态
            this.parse(promise, onFulfilled(value), resolve, reject);
          },
          onRejected: (value) => {
            this.parse(promise, onRejected(value), resolve, reject);
          },
        });
      }
      // 只有当promise执行完，状态改变，才会执行then方法中的resolve或者reject
      if (this.status == myPromise.FULFILLED) {
        // 3.promise外部的同步代码要比then方法先执行，因此要放入任务队列中
        setTimeout(() => {
          this.parse(promise, onFulfilled(this.value), resolve, reject);
        });
      }
      if (this.status == myPromise.REJECTED) {
        setTimeout(() => {
          this.parse(promise, onRejected(this.value), resolve, reject);
        });
      }
    });
    return promise;
  }
  parse(promise, result, resolve, reject) {
    // 9. 这样的写法不允许：let p = promise.then(value=>{return p})
    if (promise == result) {
      throw new TypeError("Chaining cycle detected");
    }
    // 如果then中的resolve或者reject执行出错，则应该执行拒绝
    try {
      if (result instanceof myPromise) {
        // 8. 如果前一个promise的返回值是promise，则需要把当前的回调函数传入
        result.then(resolve, reject);
      } else {
        // 5. result是前一个promise的返回值，再执行当前promise的回调
        resolve(result);
      }
    } catch (error) {
      // 6. 类似executor，若当前promise的回调执行出错，则应该执行拒绝回调
      reject(error);
    }
  }
  static resolve(value) {
    return new myPromise((resolve, reject) => {
      // Promise.resolve(new Promise()))
      if (value instanceof myPromise) {
        value.then(resolve, reject);
      } else {
        resolve(value);
      }
    });
  }
  static reject(value) {
    return new myPromise((resolve, reject) => {
      reject(value);
    });
  }
  // 在参数对象里所有的promise对象都成功的时候才会触发成功，
  // 一旦有任何一个里面的promise对象失败则立即触发该promise对象的失败

  static all(promises) {
    const values = [];
    return new myPromise((resolve, reject) => {
      promises.forEach((promise) => {
        promise.then(
          (value) => {
            values.push(value);
            if (values.length == promises.length) {
              resolve(values);
            }
          },
          (reason) => {
            reject(reason);
          }
        );
      });
    });
  }
  // 当参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值
  // 或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象。
  static race(promises) {
    return new myPromise((resolve, reject) => {
      promises.map((promise) => {
        promise.then(
          (value) => {
            resolve(value);
          },
          (reason) => {
            reject(reason);
          }
        );
      });
    });
  }
  // Promise.any() 接收一个Promise可迭代对象，只要其中的一个 promise 成功，就返回那个已经成功的 promise 
  // 如果可迭代对象中没有一个 promise 成功（即所有的 promises 都失败/拒绝），就返回一个失败的 promise 
  static any(promises) {
    let values = [];
    return new myPromise((resolve, reject) => {
      promises.map((promise) => {
        promise.then(
          (value) => {
            resolve(value);
          },
          (reason) => {
            values.push(reason);
            if (values.length == promises.length) {
              reject(reason);
            }
          }
        );
      });
    });
  }
}
```
