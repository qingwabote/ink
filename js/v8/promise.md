```cpp
  /**
   * Set callback to notify about promise reject with no handler, or
   * revocation of such a previous notification once the handler is added.
   */
  void SetPromiseRejectCallback(PromiseRejectCallback callback);
```
这里为什么要单独处理 Promise 抛出的错误，而不是与普通错误一样走 AddMessageListener?

传入 Promise 构造函数的 executor 是同步执行的，其内部的错误随之同步抛出，但通过 Promise.prototype.catch 注册的错误处理函数同 then 一样被设计成异步执行的，且允许在 Promise 构造函数执行后再调用 catch 传入错误处理函数，同步抛出的错误与异步执行的 catch 矛盾。

V8 设计了 SetPromiseRejectCallback 弥补这个矛盾 "Set callback to notify about promise reject with no handler, or revocation of such a previous notification once the handler is added."

*[isolate_promiseRejectCallback](https://github.com/qingwabote/zero/blob/master/native/main/sugars/v8sugar.cpp)*