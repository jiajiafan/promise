### 1.promise中文翻译
#### 请访问https://jiajiafan.github.io/promise/.
### 2.promise 源码实现
```javascript
const PENDING = "PENDING";
const FULFILLED = "FULFILLED";
const REJECTED = "REJECTED";
const isPromise = (obj) => {
    if ((typeof obj === 'object' && obj !== null) || typeof obj === 'function') {
        return typeof obj.then ==='function'
    } else {
        return false;
     }
}

function resolvePromise(promise2, x, resolve, reject) {
    if (x === promise2) {
       return  reject(new TypeError(
             "TypeError: Chaining cycle detected for promise #<Promise>11"
         ))
    }
    let called;
    if ((typeof x === 'object' && x !== null) || typeof x === 'function') {
        try {
            let then = x.then;
            if (typeof then === 'function') {
                then.call(x, y => {
                    if (called) return;
                    called = true;
                     resolvePromise(promise2, y, resolve, reject)
                }, r => {
                        if (called) return;
                        called = true;
                         reject(r)
                })
            } else {
                  resolve(x)
           }
        } catch (e) {
            if (called) return;
            called = true;
             reject(e)
        }
    } else {   
            resolve(x) 
    }
}
class Promise{
    constructor(executor) {
        this.value = undefined;
        this.reason = undefined;
        this.status = PENDING;
        this.fulfillCallbacks = []
        this.rejectCallbacks = [];
        let resolve = value => {
            if (value instanceof Promise) {
                return value.then(resolve,reject)
            }
            if (this.status === PENDING) {
                this.status = FULFILLED;
                this.value = value;
                this.fulfillCallbacks.forEach(fn=>fn())               
            }
        }
        let reject = reason => {
            if (this.status === PENDING) {
                this.status = REJECTED;
                this.reason = reason;
                this.rejectCallbacks.forEach(fn => fn())
            }
        }
        try {
            executor(resolve,reject)
        } catch (e) {
            reject(e)
        }
    }
    then(onFulfilled, onRejected) {
       onFulfilled= typeof  onFulfilled === 'function' ? onFulfilled : r => r;
        onRejected =typeof onRejected === 'function' ? onRejected : s => {
            throw s;
        }
        let promise2 = new Promise((resolve, reject) => {
            if (this.status === FULFILLED) {
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value)
                        resolvePromise(promise2,x,resolve,reject)
                    } catch (e) {
                       reject(e)
                   }
              })
            }
            if (this.status === REJECTED) {
                setTimeout(() => {
                    try {
                        let x = onRejected(this.reason)
                        resolvePromise(promise2, x, resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                }) 
            }
            if (this.status === PENDING){
                this.fulfillCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value)
                            resolvePromise(promise2,x,resolve,reject)
                        } catch (e) {
                            reject(e)
                        }
                       
                   })
                })
                this.rejectCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onRejected(this.reason);
                             resolvePromise(promise2, x, resolve, reject)
                        } catch (e) {
                            reject(e)
                       }
                   })
                })
            }
            
        })
        return promise2;   
    }
    catch(callback) {
        return this.then(null,callback)
    }
    finally(fn) {
        return this.then(value => {
            return Promise.resolve(fn()).then(()=>value)
        },reason => {
                return Promise.resolve(fn()).then(() =>{ throw reason })
        })
        
    }
    static try(fn) {
        return await (async ()=>fn())()
    }
    static all(promises) {
         return new Promise((resolve, reject) => {
             let arr = [];
             let index = 0;
              let addData=function(i, data) {
                  arr[i] = data;
                  if (++index === promises.length) {
                     resolve(arr)
                  }
              }
             for (let i = 0; i < promises.length; i++) {
                 let cur = promises[i]
                 if (isPromise(cur)) {
                     cur.then(value => {
                         addData(i,value)
                     },reject)
                 } else {
                    addData(i,cur)
                 }
             }
         })
    }
    static race(promises) {
        return new Promise((resolve, reject) => {
            for (let i = 0; i < promises.length; i++){
                let cur = promises[i]
                if (isPromise(cur)) {
                    cur.then(resolve,reject)
                } else {
                    resolve(cur)
                }
            }
        })
    }
    static resolve(value) {
        return new Promise((resolve, reject) => {
            resolve(value);
        })
    }
    static reject(reason) {
        return new Promise((resolve, reject) => {
            reject(reason)
        })
    }

}
```
-
