# 手写promise

```js
/**
 * promise 自身的状态
 * 1.state 存放当前的状态
 * 2.value 存放当前的值
 * 3.then 方法的返回值也是一个promise
 * 4.catch 放法
 * 5.finally 放法
 * 6.静态放法，Promise.all、Promise.resolve
 */

//  Promise的三种状态
const PENDING = 'PENDING';
const FULFILLED = 'FULFILLED';
const REJECTED = 'REJECTED';
class MyPromise {
    constructor(fn) {
        // 默认状态
        this.status = PENDING;
        // 初始值
        this.value = undefined;
        // then链式调用
        this.resolveCallBacks = [];
        // reject
        this.rejectCallBacks = [];
        const resolve = val => {
            // 如果val值是一个promise对象并且不是数组
            if ((typeof val === 'object' || typeof val === 'function') && val.then) {
                promiseResolutionProcedure(this, val, resolve, reject);
                return;
            }
            // setTimeout是因resolve是同步的，在then还没有挂在的时候就执行了resolve
            setTimeout(() => {
                if (this.status === PENDING) {
                    // 防止then多次执行
                    this.status = FULFILLED;
                    this.value = val;
                    // 执行所有then方法
                    this.resolveCallBacks.map(fn => fn(val));
                }
            })
        }
        const reject = (val) => {
            if (this.status === PENDING) {
                this.value = val;
                this.status = REJECTED;
                this.rejectCallBacks.map(fn => fn(val));
            }

        };
        fn(resolve, reject);
    }
    // 解决空then报错问题。val透传
    then(onFulfilled = val => val, onRjected = err => {
        throw new Error(err);
    }) {
        let promise = null;
        // 处理已经完成的promise
        if (this.status === FULFILLED) {
            promise = new MyPromise((resolve, reject) => {
                 // 第二个then的参数是第一个then结果
                const x = onFulfilled(this.value);
                 // 判断是不是thenable对象
                promiseResolutionProcedure(promise2, x, resolve, reject);
            })
        }

        // 处理已经完成的promise
        if (this.status === REJECTED) {
            promise = new MyPromise((resolve, reject) => {
                 // 第二个then的参数是第一个then结果
                const x = onRjected(this.value);
                 // 判断是不是thenable对象
                promiseResolutionProcedure(promise2, x, resolve, reject);
            })
        }

        // 处理尚未完成的Promise
        // 调用次数不能超过一次
        // PENDING状态才会去缓存callback
        if (this.status === PENDING) {
            // 需要返回一个promise ,才能实现链式调用
            promise = new MyPromise((resolve, reject) => {
                this.resolveCallBacks.push(() => {
                    // 第二个then的参数是第一个then结果
                    const x = onFulfilled(this.value);
                    // 判断是不是thenable对象
                    promiseResolutionProcedure(promise, x, resolve, reject);
                });
                this.rejectCallBacks.push(() => {
                    // 第二个then的参数是第一个then结果
                    const x = onRjected(this.value);
                    // 判断是不是thenable对象
                    promiseResolutionProcedure(promise, x, resolve, reject);
                });
            });
        }
        return promise;
    }
    // 实现静态方法allpromsie.all
    static all(promiseArray) {
        return new MyPromise((resolve, reject) =>{
            const resultArray = [];
            let successTimes = 0;

            function processResult(i, data) {
                resultArray[i] = data;
                successTimes++;
                if (successTimes === promiseArray.length) {
                    // 代表全部成功了
                    resolve(resultArray);
                }
            }
            for (let i = 0; i < promiseArray.length; i++ ) {
                promiseArray[i].then(data => {
                    processResult(i, data);
                }, err => {
                    // 处理失败
                    reject(err);
                })
            }
        })
    }
}



function promiseResolutionProcedure(promise, x, resolve, reject) {
    if (promise === x) {
        // 判断是否是循环
        throw new Error('循环');
    }
    // 处理promise
    if (x instanceof MyPromise) {
        // 如果是pending的状态
        if (x.status === PENDING) {
            x.then(y => promiseResolutionProcedure(promise, y, resolve, reject), reject, reject);
        } else {
            x.status = FULFILLED && resolve(x.value)
            x.status = REJECTED && reject(x.value)
        }
    }

    // 判断thenable对象
    if ((typeof x === 'object' || typeof x === 'function') && x !== null) {
        if (typeof x.then === 'function') {
            // y是不是也是一个thenable, 需要循环递归调用
            x.then(y => promiseResolutionProcedure(promise, y, resolve, reject), reject);
        } else {
            resolve(x);
        }
    } else {
        resolve(x);
    }
}

// 实现最基本功能
new MyPromise((resolve, reject) => {
    // setTimeout(() => {
    resolve('resolve的数据')
    // }, 1000)
}).then(data => {
    console.log(data, 'sdfa');
    return { //thenable对象 promise解析的时候会当成一个promise去解析的
        then(r, j) {
            r('df')
        }
    }
});
```