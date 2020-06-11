---
title: Promise1-link
date: 2020-06-10 21:53:16
tags:
---
##### promise 链式调用
https://promiseaplus.com promise调用规范
````javascript
function Promise (executor) {
 let self = this;
 // 保存成功和失败的值
 self.value = undefined;
 self.reason = undefined;
 self.onResolvedCallback = [];
 self.onRejectedCallback = [];
 // 保存一下当前这个promise的状态(promise有三个状态)
 self.status = 'pending';
 function resolve(value) {
  if (self.statue === 'pending') {
   self.value= value;
   self.status = 'resolved';
   self.onResolvedCallbacks.forEach(function (fn) {
    fn();
   })
  }
 }
 function reject(reason) {
  if (self.statue === 'pending') {
   self.reason = reason;
   self.status = 'rejected';
   self.onRejectedCallback.forEach(function (fn) {
    fn();
   })
  }
 }
 // executor是立即执行的
 try {
  executor(resolve, reject);
 } catch (e) {
 // 如果执行 执行器 的时候发生过异常那就走到then的失败函数中
  reject(e)
 }
}
// 解析链式调用的。
function resolvePromise(x, promise, resolve, reject)
// then方法中需要传递两个参数， 分别是成功的回调和失败的回调
Promise.prototype.then = function (onFufilled, onRejected) { 
 let self = this;
 // 调用then后返回一个promise
 let promise = new Promise(function (resolve, reject) {
  if（self.status === 'resolved'） {
  // 我们限制需要做的事情就是把then中成功或者失败后韩式执行的结果获取到
  //看一看是不是promose 如果是promise 就让promise 执行，取到最终这个promise的执行结果，让返回的promise成功或者失败
  // 如果x是普通值就让这个返回的promise变成成功态
   let x = onFufilled(self.value);
   resolvePromise(x, promise, resolve, reject);
  }
  if（self.status === 'rejected'） {
   let x = onRejected(self.reason);
   resolvePromise(x, promise, resolve, reject);
  }
  // executor 中有异步操作，此时调用
  if (self.status === 'pending') {
   self.onResolvedCallback.push(function () {
    let x = onFulfilled(self.value);
    resolvePromise(x, promise, resolve, reject);
   });
   self.onRejectCallback.push(function () {
    let x = onRejected(self.value);
    resolvePromise(x, promise, resolve, reject);
   })
  }
 })
 if（self.status === 'resolved'） {
  onFufilled(self.value);
 }
 if（self.status === 'rejected'） {
  onRejected(self.reason);
 }
 // executor 中有异步操作，此时调用
 if (self.status === 'pending') {
  self.onResolvedCallback.push(function () {
   onFulfilled(self.value);
  });
  self.onRejectCallback.push(function () {
   onRejected(self.value);
  })
 }
 return promise
}
module.exports = Promise
````
then执行之后返回一个新的promise
因为promise 的状态一旦失败就不能再成功了