---
title: 实现一个promise
date: 2019-09-10 11:35:22
tags: promise
---
### 一、提问环节
经过学习，学会了promise实现，但是有一点不懂，

就是 _resolve、_reject 里面为什么用异步？
> ** setTimeout(run, 0); ** 


``` javascript
/*
 * @Description: 实现一个promise
 * @Author: ymbo
 * @Date: 2019-09-09 11:07:26
 * @LastEditTime: 2019-09-11 17:49:59
 * @LastEditors: Please set LastEditors
 */

const PENDING = 'PENDING'
const FULFILLED = 'FULFILLED'
const REJECTED = 'REJECTED'

class myPromise {
    constructor(handle) {
        //  判断handle是否为函数
        if (typeof handle !== 'function') {
            throw new Error('handle must be a function！')
        }

        // promise状态
        this._status = PENDING
            // 当前value值
        this._value = undefined
            // 成功队列
        this._fulfilledQueues = []
            // 失败队列
        this._rejectedQueues = []
        try {
            handle(this._resolve.bind(this), this._reject.bind(this))
        } catch (error) {
            this._reject(error)
        }
    }

    // 成功
    // 1.这个value也可能是一个promise，这个promise的状态和值都将赋给当前的value和status
    _resolve(value) {
        // 如果不是pending状态则表示已完成，就不执行了
        if (this._status !== PENDING) { return }
        let run = () => {
            let runFulfilled = (val) => {
                let cd = undefined
                    // 当从pending变为fulfilled/rejected时，执行成功队列的内容
                while (cd = this._fulfilledQueues.shift()) {
                    cd(val)
                }
            }
            let runRejected = (err) => {
                let cd = undefined
                    // 当从pending变为fulfilled/rejected时，执行成功队列的内容
                while (cd = this._rejectedQueues.shift()) {
                    cd(err)
                }
            }
            if (value instanceof myPromise) {
                value.then(val => {
                    this._value = val
                    this._status = FULFILLED
                    runFulfilled(val)
                }, err => {
                    this._value = err
                    this._status = REJECTED
                    runRejected(err)
                })
            } else {
                this._value = value
                this._status = FULFILLED
                runFulfilled(value)
            }
        }
        setTimeout(run, 0);
    }
    _reject(err) {
        // 如果不是pending状态则表示已完成，就不执行了
        if (this._status !== PENDING) { return }
        let run = () => {
            // 状态变更
            this._status = REJECTED
                //  值变更
            this._value = err

            let cd = undefined
                // 当从pending变为fulfilled/rejected时，执行成功队列的内容
            while (cd = this._rejectedQueues.shift()) {
                cd(_value)
            }
        }
        setTimeout(run, 0);
    }

    // then函数
    // 1.then接受两个函数参数
    // 2.then返回一个新的promsie
    // 3.
    then(onFulfilled, onRejected) {
        const { _value, _status } = this
        return new myPromise((resolve, reject) => {
            let fulfilled = value => {
                try {
                    // 如果then的第一个参数不是函数，则直接将值传递下去
                    if (typeof onFulfilled !== 'function') {
                        resolve(value)
                    } else {
                        let res = onFulfilled(value)
                            // 如果res是一个promise，则需要等待这个promise执行完毕，再传递值
                        console.log('是一个promise', res instanceof myPromise)
                        if (res instanceof myPromise) {
                            res.then(resolve, reject)
                        } else {
                            // 如果返回值是值类型,则直接传递下去
                            resolve(res)
                        }
                    }
                } catch (error) {
                    resolve(error)
                }
            }

            let rejected = err => {
                try {
                    // 如果then的第一个参数不是函数，则直接将值传递下去
                    if (typeof onRejected !== 'function') {
                        reject(err)
                    } else {
                        let res = onRejected(err)
                            // 如果res是一个promise，则需要等待这个promise执行完毕，再传递值
                        if (res instanceof myPromise) {
                            res.then(resolve, reject)
                        } else {
                            // 如果返回值是值类型,则直接传递下去
                            reject(res)
                        }
                    }
                } catch (error) {
                    reject(error)
                }
            }
            switch (_status) {
                // 如果还在pending中就把函数放到指定队列里
                case PENDING:
                    this._fulfilledQueues.push(fulfilled)
                    this._rejectedQueues.push(rejected)
                    break;

                    // 如果处于 fulfilled 则直接执行
                case FULFILLED:
                    fulfilled(_value)
                    break;
                case REJECTED:
                    rejected(_value)
                    break;
            }
        })
    }


    finally(cb) {
        return this.then(
            value => MyPromise.resolve(cb()).then(() => value),
            reason => MyPromise.resolve(cb()).then(() => { throw reason })
        );
    }

    //catch
    catch (onRejected) {
        return this.then(undefined, onRejected)
    }
    // 静态resolve
    static resolve(value) {
        if (value instanceof myPromise) {
            return value
        } else {
            return new myPromise((resolve, reject) => {
                resolve(value)
            })
        }
    }

    // 静态reject
    static reject(err) {
        return new myPromise((resolve, reject) => {
            reject(err)
        })
    }

    //all
    // list内容可以是promise 或者其他
    static all(list) {
        return new myPromise((resolve, reject) => {
            /**
             * 返回值的集合
             */
            let values = []
            let count = 0
            for (let [i, p] of list.entries()) {
                // 数组参数如果不是MyPromise实例，先调用MyPromise.resolve
                this.resolve(p).then(res => {
                    values[i] = res
                    count++
                    // 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled
                    if (count === list.length) resolve(values)
                }, err => {
                    // 有一个被rejected时返回的MyPromise状态就变成rejected
                    reject(err)
                })
            }
        })
    }

    static race(list) {
        return new myPromise((resolve, reject) => {
            for (let [i, p] of list.entries()) {
                this.resolve(p).then(res => {
                    resolve(res)
                }, err => {
                    reject(err)
                })
            }
        })
    }
}
```