https://github.com/nodejs/help/issues/3124

我们有 JS 引擎和它的宿主（比如浏览器），浏览器可以随意调用 JS 引擎（下面简称 JS）的函数，比如 JS 里有个 main 函数，浏览器调用这个 main 函数，JS 获得了运行的机会，它从入口 main → foo → bar... 一路执行下去，直到函数栈彻底执行完，然后歇菜。

那么异步函数怎么处理，比如异步 http 请求，main → foo → sendHttpRequest(callback) 注意这个 callback，后面要用。比如 sendHttpRequest 是浏览器导出给 JS 的 API，它被调用后，浏览器新开了一个线程（因为要异步嘛）去执行 http 请求，这个新线程收到来自 JS 的 callback 函数，等 http 完成后，这个新线程显然要调用这个 callback 让逻辑回到 JS 中去，但它是新线程，显然和 JS 不在一个线程里，而 callback 必须在 JS 线程里执行，怎么办。

这就涉及到线程同步的问题，我们可以用生产者消费者模式，构建一个线程安全的任务队列，执行完 http 请求的那个新线程不直接调用 callback，而是把它放入任务队列，然后自己就可以狗带了。那么，谁从这个任务队列里弹出任务并执行呢。

浏览器创建一个 event loop，它和 JS 处于同一个线程，每隔一段时间清空一次这个任务队列，执行里面的所有任务。最简单的实现是每次 event loop 锁住这个队列，清空，执行所有任务，解锁。那么，当执行某个任务的过程中又有新的任务添加，比如在 setTimeOut 的 callback 里又调用了 setTimeOut(0)，参数 0 确保它立刻完成并尝试添加回调任务到队列，新任务可能要等 event loop 解锁队列后才能添加进来，所以它的执行对应的会延迟到下一个 event loop。这种设计一直可以满足需求，直到 Promise 的出现，Promise 无法接受这么高的延迟，于是，浏览器又新建了一个任务队列，放在原有的队列后执行，命名为微任务，而以前的就叫宏任务。

微任务确保执行任务的过程中产生的新任务能被及时的添加并执行，比如只在从任务队列里弹出任务时加锁，执行中解锁，确保新的任务能被添加，并且直到队列里的任务全部执行后才退出循环。


## v8::platform::PumpMessageLoop
https://groups.google.com/g/v8-users/c/lRe7WrCkHCE/m/NmMfpDZ-DAAJ
未见 cocos(v3.7.1) 这样操作，cocos 怎么没出问题？
