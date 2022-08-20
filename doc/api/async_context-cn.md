# Asynchronous context tracking

<!--introduced_in=v16.4.0-->

> Stability: 2 - Stable

<!-- source_link=lib/async_hooks.js -->

## Introduction

这些类用于关联状态并在回调和承诺链中传播状态.
它们允许在 Web 请求的整个生命周期或任何其他异步持续时间中存储数据。类似于其他语言的线程本地存储.

`AsyncLocalStorage` 和 `AsyncResource` 类是 `node:async_hooks` 模块的一部分:

```mjs
import { AsyncLocalStorage, AsyncResource } from 'node:async_hooks';
```

```cjs
const { AsyncLocalStorage, AsyncResource } = require('node:async_hooks');
```

## Class: `AsyncLocalStorage`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
changes:
 - version: v16.4.0
   pr-url: https://github.com/nodejs/node/pull/37675
   description: AsyncLocalStorage is now Stable. Previously, it had been Experimental.
-->

此类创建通过异步操作保持一致的存储.

虽然您可以在 `node:async_hooks` 模块之上创建自己的实现，但应该首选 `AsyncLocalStorage`，因为它是一种高性能且内存安全的实现，其中涉及不明显的显着优化.

以下示例使用 `AsyncLocalStorage` 构建一个简单的记录器，为传入的 HTTP 请求分配 ID，并将它们包含在每个请求中记录的消息中.

```mjs
import http from 'node:http';
import { AsyncLocalStorage } from 'node:async_hooks';

const asyncLocalStorage = new AsyncLocalStorage();

function logWithId(msg) {
  const id = asyncLocalStorage.getStore();
  console.log(`${id !== undefined ? id : '-'}:`, msg);
}

let idSeq = 0;
http.createServer((req, res) => {
  asyncLocalStorage.run(idSeq++, () => {
    logWithId('start');
    // Imagine any chain of async operations here
    setImmediate(() => {
      logWithId('finish');
      res.end();
    });
  });
}).listen(8080);

http.get('http://localhost:8080');
http.get('http://localhost:8080');
// Prints:
//   0: start
//   1: start
//   0: finish
//   1: finish
```

```cjs
const http = require('node:http');
const { AsyncLocalStorage } = require('node:async_hooks');

const asyncLocalStorage = new AsyncLocalStorage();

function logWithId(msg) {
  const id = asyncLocalStorage.getStore();
  console.log(`${id !== undefined ? id : '-'}:`, msg);
}

let idSeq = 0;
http.createServer((req, res) => {
  asyncLocalStorage.run(idSeq++, () => {
    logWithId('start');
    // Imagine any chain of async operations here
    setImmediate(() => {
      logWithId('finish');
      res.end();
    });
  });
}).listen(8080);

http.get('http://localhost:8080');
http.get('http://localhost:8080');
// Prints:
//   0: start
//   1: start
//   0: finish
//   1: finish
```

`AsyncLocalStorage` 的每个实例都维护一个独立的存储上下文.
多个实例可以安全地同时存在，没有相互干扰数据的风险.

### `new AsyncLocalStorage()`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

创建 `AsyncLocalStorage` 的新实例。 Store 仅在 `run()` 调用中或在 `enterWith()` 调用之后提供.

### `asyncLocalStorage.disable()`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

> Stability: 1 - Experimental

禁用 `AsyncLocalStorage` 的实例。对 `asyncLocalStorage.getStore()` 的所有后续调用都将返回 `undefined`，直到再次调用 `asyncLocalStorage.run()` 或 `asyncLocalStorage.enterWith()`.

调用 `asyncLocalStorage.disable()` 时，所有当前链接到实例的上下文都将退出.

在可以对 `asyncLocalStorage` 进行垃圾收集之前，需要调用 `asyncLocalStorage.disable()`。这不适用于由 `asyncLocalStorage` 提供的存储，因为这些对象与相应的异步资源一起被垃圾收集.

在当前进程中不再使用 `asyncLocalStorage` 时使用此方法.

### `asyncLocalStorage.getStore()`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

* Returns: {any}

返回当前商店.
如果在通过调用 `asyncLocalStorage.run()` 或 `asyncLocalStorage.enterWith()` 初始化的异步上下文之外调用，则返回 `undefined`.

### `asyncLocalStorage.enterWith(store)`

<!-- YAML
added:
 - v13.11.0
 - v12.17.0
-->

> Stability: 1 - Experimental

* `store` {any}

过渡到当前同步执行的剩余部分的上下文，然后通过任何后续异步调用持久化存储.

Example:

```js
const store = { id: 1 };
// Replaces previous store with the given store object
asyncLocalStorage.enterWith(store);
asyncLocalStorage.getStore(); // Returns the store object
someAsyncOperation(() => {
  asyncLocalStorage.getStore(); // Returns the same object
});
```

此转换将继续_整个_同步执行.
这意味着，例如，如果在事件处理程序中输入上下文，则后续事件处理程序也将在该上下文中运行，除非专门使用“AsyncResource”绑定到另一个上下文。这就是为什么 `run()` 应该优先于 `enterWith()` 的原因，除非有充分的理由使用后一种方法.

```js
const store = { id: 1 };

emitter.on('my-event', () => {
  asyncLocalStorage.enterWith(store);
});
emitter.on('my-event', () => {
  asyncLocalStorage.getStore(); // Returns the same object
});

asyncLocalStorage.getStore(); // Returns undefined
emitter.emit('my-event');
asyncLocalStorage.getStore(); // Returns the same object
```

### `asyncLocalStorage.run(store, callback[, ...args])`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

* `store` {any}
* `callback` {Function}
* `...args` {any}

在上下文中同步运行函数并返回其返回值。在回调函数之外无法访问商店.
回调中创建的任何异步操作都可以访问存储.

可选的 `args` 被传递给回调函数.

如果回调函数抛出错误，则`run()`也会抛出错误.
堆栈跟踪不受此调用影响，并且退出上下文.

Example:

```js
const store = { id: 2 };
try {
  asyncLocalStorage.run(store, () => {
    asyncLocalStorage.getStore(); // Returns the store object
    setTimeout(() => {
      asyncLocalStorage.getStore(); // Returns the store object
    }, 200);
    throw new Error();
  });
} catch (e) {
  asyncLocalStorage.getStore(); // Returns undefined
  // The error will be caught here
}
```

### `asyncLocalStorage.exit(callback[, ...args])`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

> Stability: 1 - Experimental

* `callback` {Function}
* `...args` {any}

在上下文之外同步运行函数并返回其返回值。在回调函数或在回调中创建的异步操作中无法访问存储。在回调函数中完成的任何 `getStore()` 调用将始终返回 `undefined`.

可选的 `args` 被传递给回调函数.

如果回调函数抛出错误，`exit()` 也会抛出错误.
堆栈跟踪不受此调用的影响，并且重新进入上下文.

Example:

```js
// Within a call to run
try {
  asyncLocalStorage.getStore(); // Returns the store object or value
  asyncLocalStorage.exit(() => {
    asyncLocalStorage.getStore(); // Returns undefined
    throw new Error();
  });
} catch (e) {
  asyncLocalStorage.getStore(); // Returns the same object or value
  // The error will be caught here
}
```

### Usage with `async/await`

如果在异步函数中，只有一个 `await` 调用要在上下文中运行，则应使用以下模式:

```js
async function fn() {
  await asyncLocalStorage.run(new Map(), () => {
    asyncLocalStorage.getStore().set('key', value);
    return foo(); // The return value of foo will be awaited
  });
}
```

在本例中，store 仅在回调函数和 `foo` 调用的函数中可用。在 `run` 之外，调用 `getStore` 将返回 `undefined`.

### Troubleshooting: Context loss

在大多数情况下，`AsyncLocalStorage` 可以正常工作。在极少数情况下，当前存储在异步操作之一中丢失.

如果您的代码是基于回调的，则使用 [`util.promisify()`][] 对其进行承诺就足够了，因此它开始使用本机承诺.

如果您需要使用基于回调的 API 或者您的代码采用自定义 thenable 实现，请使用 [`AsyncResource`][] 类将异步操作与正确的执行上下文相关联.
在您怀疑造成损失的调用之后，通过记录 `asyncLocalStorage.getStore()` 的内容来查找造成上下文损失的函数调用。当代码记录 `undefined` 时，最后调用的回调可能是造成上下文丢失的原因.

## Class: `AsyncResource`

<!-- YAML
changes:
 - version: v16.4.0
   pr-url: https://github.com/nodejs/node/pull/37675
   description: AsyncResource is now Stable. Previously, it had been Experimental.
-->

`AsyncResource` 类旨在通过嵌入器的异步资源进行扩展。使用它，用户可以轻松触发自己资源的生命周期事件.

`init` 钩子会在 `AsyncResource` 被实例化时触发.

以下是 `AsyncResource` API 的概述.

```mjs
import { AsyncResource, executionAsyncId } from 'node:async_hooks';

// AsyncResource() is meant to be extended. Instantiating a
// new AsyncResource() also triggers init. If triggerAsyncId is omitted then
// async_hook.executionAsyncId() is used.
const asyncResource = new AsyncResource(
  type, { triggerAsyncId: executionAsyncId(), requireManualDestroy: false }
);

// Run a function in the execution context of the resource. This will
// * establish the context of the resource
// * trigger the AsyncHooks before callbacks
// * call the provided function `fn` with the supplied arguments
// * trigger the AsyncHooks after callbacks
// * restore the original execution context
asyncResource.runInAsyncScope(fn, thisArg, ...args);

// Call AsyncHooks destroy callbacks.
asyncResource.emitDestroy();

// Return the unique ID assigned to the AsyncResource instance.
asyncResource.asyncId();

// Return the trigger ID for the AsyncResource instance.
asyncResource.triggerAsyncId();
```

```cjs
const { AsyncResource, executionAsyncId } = require('node:async_hooks');

// AsyncResource() is meant to be extended. Instantiating a
// new AsyncResource() also triggers init. If triggerAsyncId is omitted then
// async_hook.executionAsyncId() is used.
const asyncResource = new AsyncResource(
  type, { triggerAsyncId: executionAsyncId(), requireManualDestroy: false }
);

// Run a function in the execution context of the resource. This will
// * establish the context of the resource
// * trigger the AsyncHooks before callbacks
// * call the provided function `fn` with the supplied arguments
// * trigger the AsyncHooks after callbacks
// * restore the original execution context
asyncResource.runInAsyncScope(fn, thisArg, ...args);

// Call AsyncHooks destroy callbacks.
asyncResource.emitDestroy();

// Return the unique ID assigned to the AsyncResource instance.
asyncResource.asyncId();

// Return the trigger ID for the AsyncResource instance.
asyncResource.triggerAsyncId();
```

### `new AsyncResource(type[, options])`

* `type` {string} The type of async event.
* `options` {Object}
  * `triggerAsyncId` {number} The ID of the execution context that created this
    async event. **Default:** `executionAsyncId()`.
  * `requireManualDestroy` {boolean} If set to `true`, disables `emitDestroy`
    when the object is garbage collected. This usually does not need to be set
    (even if `emitDestroy` is called manually), unless the resource's `asyncId`
    is retrieved and the sensitive API's `emitDestroy` is called with it.
    When set to `false`, the `emitDestroy` call on garbage collection
    will only take place if there is at least one active `destroy` hook.
    **Default:** `false`.

Example usage:

```js
class DBQuery extends AsyncResource {
  constructor(db) {
    super('DBQuery');
    this.db = db;
  }

  getInfo(query, callback) {
    this.db.get(query, (err, data) => {
      this.runInAsyncScope(callback, null, err, data);
    });
  }

  close() {
    this.db = null;
    this.emitDestroy();
  }
}
```

### Static method: `AsyncResource.bind(fn[, type[, thisArg]])`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
changes:
  - version:
    - v17.8.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/42177
    description: Changed the default when `thisArg` is undefined to use `this`
                 from the caller.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36782
    description: Added optional thisArg.
-->

* `fn` {Function} The function to bind to the current execution context.
* `type` {string} An optional name to associate with the underlying
  `AsyncResource`.
* `thisArg` {any}

将给定函数绑定到当前执行上下文.

返回的函数将有一个 `asyncResource` 属性引用函数绑定到的`AsyncResource`.

### `asyncResource.bind(fn[, thisArg])`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
changes:
  - version:
    - v17.8.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/42177
    description: Changed the default when `thisArg` is undefined to use `this`
                 from the caller.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36782
    description: Added optional thisArg.
-->

* `fn` {Function} The function to bind to the current `AsyncResource`.
* `thisArg` {any}

将要执行的给定函数绑定到此 `AsyncResource` 的范围.

返回的函数将有一个 `asyncResource` 属性引用函数绑定到的`AsyncResource`.

### `asyncResource.runInAsyncScope(fn[, thisArg, ...args])`

<!-- YAML
added: v9.6.0
-->

* `fn` {Function} The function to call in the execution context of this async
  resource.
* `thisArg` {any} The receiver to be used for the function call.
* `...args` {any} Optional arguments to pass to the function.

在异步资源的执行上下文中使用提供的参数调用提供的函数。这将建立上下文，在回调之前触发AsyncHooks，调用函数，在回调之后触发AsyncHooks，然后恢复原来的执行上下文.

### `asyncResource.emitDestroy()`

* Returns: {AsyncResource} A reference to `asyncResource`.

调用所有的 `destroy` 钩子。这应该只被调用一次。如果多次调用它会抛出一个错误。这**必须**手动调用。如果资源留给 GC 收集，则永远不会调用 `destroy` 钩子.

### `asyncResource.asyncId()`

* Returns: {number} The unique `asyncId` assigned to the resource.

### `asyncResource.triggerAsyncId()`

* Returns: {number} The same `triggerAsyncId` that is passed to the
  `AsyncResource` constructor.

<a id="async-resource-worker-pool"></a>

### Using `AsyncResource` for a `Worker` thread pool

以下示例展示了如何使用 `AsyncResource` 类为 [`Worker`][] 池正确提供异步跟踪。其他资源池，例如数据库连接池，可以遵循类似的模型.

假设任务是两个数字相加，使用名为 `task_processor.js` 的文件，内容如下:

```mjs
import { parentPort } from 'node:worker_threads';
parentPort.on('message', (task) => {
  parentPort.postMessage(task.a + task.b);
});
```

```cjs
const { parentPort } = require('node:worker_threads');
parentPort.on('message', (task) => {
  parentPort.postMessage(task.a + task.b);
});
```

围绕它的 Worker 池可以使用以下结构:

```mjs
import { AsyncResource } from 'node:async_hooks';
import { EventEmitter } from 'node:events';
import path from 'node:path';
import { Worker } from 'node:worker_threads';

const kTaskInfo = Symbol('kTaskInfo');
const kWorkerFreedEvent = Symbol('kWorkerFreedEvent');

class WorkerPoolTaskInfo extends AsyncResource {
  constructor(callback) {
    super('WorkerPoolTaskInfo');
    this.callback = callback;
  }

  done(err, result) {
    this.runInAsyncScope(this.callback, null, err, result);
    this.emitDestroy();  // `TaskInfo`s are used only once.
  }
}

export default class WorkerPool extends EventEmitter {
  constructor(numThreads) {
    super();
    this.numThreads = numThreads;
    this.workers = [];
    this.freeWorkers = [];
    this.tasks = [];

    for (let i = 0; i < numThreads; i++)
      this.addNewWorker();

    // Any time the kWorkerFreedEvent is emitted, dispatch
    // the next task pending in the queue, if any.
    this.on(kWorkerFreedEvent, () => {
      if (this.tasks.length > 0) {
        const { task, callback } = this.tasks.shift();
        this.runTask(task, callback);
      }
    });
  }

  addNewWorker() {
    const worker = new Worker(new URL('task_processer.js', import.meta.url));
    worker.on('message', (result) => {
      // In case of success: Call the callback that was passed to `runTask`,
      // remove the `TaskInfo` associated with the Worker, and mark it as free
      // again.
      worker[kTaskInfo].done(null, result);
      worker[kTaskInfo] = null;
      this.freeWorkers.push(worker);
      this.emit(kWorkerFreedEvent);
    });
    worker.on('error', (err) => {
      // In case of an uncaught exception: Call the callback that was passed to
      // `runTask` with the error.
      if (worker[kTaskInfo])
        worker[kTaskInfo].done(err, null);
      else
        this.emit('error', err);
      // Remove the worker from the list and start a new Worker to replace the
      // current one.
      this.workers.splice(this.workers.indexOf(worker), 1);
      this.addNewWorker();
    });
    this.workers.push(worker);
    this.freeWorkers.push(worker);
    this.emit(kWorkerFreedEvent);
  }

  runTask(task, callback) {
    if (this.freeWorkers.length === 0) {
      // No free threads, wait until a worker thread becomes free.
      this.tasks.push({ task, callback });
      return;
    }

    const worker = this.freeWorkers.pop();
    worker[kTaskInfo] = new WorkerPoolTaskInfo(callback);
    worker.postMessage(task);
  }

  close() {
    for (const worker of this.workers) worker.terminate();
  }
}
```

```cjs
const { AsyncResource } = require('node:async_hooks');
const { EventEmitter } = require('node:events');
const path = require('node:path');
const { Worker } = require('node:worker_threads');

const kTaskInfo = Symbol('kTaskInfo');
const kWorkerFreedEvent = Symbol('kWorkerFreedEvent');

class WorkerPoolTaskInfo extends AsyncResource {
  constructor(callback) {
    super('WorkerPoolTaskInfo');
    this.callback = callback;
  }

  done(err, result) {
    this.runInAsyncScope(this.callback, null, err, result);
    this.emitDestroy();  // `TaskInfo`s are used only once.
  }
}

class WorkerPool extends EventEmitter {
  constructor(numThreads) {
    super();
    this.numThreads = numThreads;
    this.workers = [];
    this.freeWorkers = [];
    this.tasks = [];

    for (let i = 0; i < numThreads; i++)
      this.addNewWorker();

    // Any time the kWorkerFreedEvent is emitted, dispatch
    // the next task pending in the queue, if any.
    this.on(kWorkerFreedEvent, () => {
      if (this.tasks.length > 0) {
        const { task, callback } = this.tasks.shift();
        this.runTask(task, callback);
      }
    });
  }

  addNewWorker() {
    const worker = new Worker(path.resolve(__dirname, 'task_processor.js'));
    worker.on('message', (result) => {
      // In case of success: Call the callback that was passed to `runTask`,
      // remove the `TaskInfo` associated with the Worker, and mark it as free
      // again.
      worker[kTaskInfo].done(null, result);
      worker[kTaskInfo] = null;
      this.freeWorkers.push(worker);
      this.emit(kWorkerFreedEvent);
    });
    worker.on('error', (err) => {
      // In case of an uncaught exception: Call the callback that was passed to
      // `runTask` with the error.
      if (worker[kTaskInfo])
        worker[kTaskInfo].done(err, null);
      else
        this.emit('error', err);
      // Remove the worker from the list and start a new Worker to replace the
      // current one.
      this.workers.splice(this.workers.indexOf(worker), 1);
      this.addNewWorker();
    });
    this.workers.push(worker);
    this.freeWorkers.push(worker);
    this.emit(kWorkerFreedEvent);
  }

  runTask(task, callback) {
    if (this.freeWorkers.length === 0) {
      // No free threads, wait until a worker thread becomes free.
      this.tasks.push({ task, callback });
      return;
    }

    const worker = this.freeWorkers.pop();
    worker[kTaskInfo] = new WorkerPoolTaskInfo(callback);
    worker.postMessage(task);
  }

  close() {
    for (const worker of this.workers) worker.terminate();
  }
}

module.exports = WorkerPool;
```

如果没有 `WorkerPoolTask​​Info` 对象添加的显式跟踪，回调似乎与各个 `Worker` 对象相关联。但是，`Worker`s 的创建与任务的创建无关，并且不提供有关何时安排任务的信息.

该池可按如下方式使用:

```mjs
import WorkerPool from './worker_pool.js';
import os from 'node:os';

const pool = new WorkerPool(os.cpus().length);

let finished = 0;
for (let i = 0; i < 10; i++) {
  pool.runTask({ a: 42, b: 100 }, (err, result) => {
    console.log(i, err, result);
    if (++finished === 10)
      pool.close();
  });
}
```

```cjs
const WorkerPool = require('./worker_pool.js');
const os = require('node:os');

const pool = new WorkerPool(os.cpus().length);

let finished = 0;
for (let i = 0; i < 10; i++) {
  pool.runTask({ a: 42, b: 100 }, (err, result) => {
    console.log(i, err, result);
    if (++finished === 10)
      pool.close();
  });
}
```

### Integrating `AsyncResource` with `EventEmitter`

由 [`EventEmitter`][] 触发的事件侦听器可能在与调用 `eventEmitter.on()` 时处于活动状态的执行上下文不同的执行上下文中运行.

以下示例显示如何使用 `AsyncResource` 类将事件侦听器与正确的执行上下文正确关联。相同的方法可以应用于 [`Stream`][] 或类似的事件驱动类.

```mjs
import { createServer } from 'node:http';
import { AsyncResource, executionAsyncId } from 'node:async_hooks';

const server = createServer((req, res) => {
  req.on('close', AsyncResource.bind(() => {
    // Execution context is bound to the current outer scope.
  }));
  req.on('close', () => {
    // Execution context is bound to the scope that caused 'close' to emit.
  });
  res.end();
}).listen(3000);
```

```cjs
const { createServer } = require('node:http');
const { AsyncResource, executionAsyncId } = require('node:async_hooks');

const server = createServer((req, res) => {
  req.on('close', AsyncResource.bind(() => {
    // Execution context is bound to the current outer scope.
  }));
  req.on('close', () => {
    // Execution context is bound to the scope that caused 'close' to emit.
  });
  res.end();
}).listen(3000);
```

[`AsyncResource`]: #class-asyncresource
[`EventEmitter`]: events.md#class-eventemitter
[`Stream`]: stream.md#stream
[`Worker`]: worker_threads.md#class-worker
[`util.promisify()`]: util.md#utilpromisifyoriginal
