# Async hooks

<!--introduced_in=v8.1.0-->

> Stability: 1 - Experimental

<!-- source_link=lib/async_hooks.js -->

`node:async_hooks` 模块提供了一个 API 来跟踪异步资源.
它可以使用:

```mjs
import async_hooks from 'node:async_hooks';
```

```cjs
const async_hooks = require('node:async_hooks');
```

## Terminology

异步资源表示具有关联回调的对象.
这个回调可能被多次调用，例如 `net.createServer()` 中的 `'connection'` 事件，或者像 `fs.open()` 中的一次.
也可以在调用回调之前关闭资源。 `AsyncHook` 没有明确区分这些不同的情况，而是将它们表示为抽象概念，即资源.

如果使用[`Worker`][]s，每个线程都有一个独立的`async_hooks`接口，每个线程都会使用一组新的异步ID.

## Overview

以下是公共 API 的简单概述.

```mjs
import async_hooks from 'node:async_hooks';

// Return the ID of the current execution context.
const eid = async_hooks.executionAsyncId();

// Return the ID of the handle responsible for triggering the callback of the
// current execution scope to call.
const tid = async_hooks.triggerAsyncId();

// Create a new AsyncHook instance. All of these callbacks are optional.
const asyncHook =
    async_hooks.createHook({ init, before, after, destroy, promiseResolve });

// Allow callbacks of this AsyncHook instance to call. This is not an implicit
// action after running the constructor, and must be explicitly run to begin
// executing callbacks.
asyncHook.enable();

// Disable listening for new asynchronous events.
asyncHook.disable();

//
// The following are the callbacks that can be passed to createHook().
//

// init() is called during object construction. The resource may not have
// completed construction when this callback runs. Therefore, all fields of the
// resource referenced by "asyncId" may not have been populated.
function init(asyncId, type, triggerAsyncId, resource) { }

// before() is called just before the resource's callback is called. It can be
// called 0-N times for handles (such as TCPWrap), and will be called exactly 1
// time for requests (such as FSReqCallback).
function before(asyncId) { }

// after() is called just after the resource's callback has finished.
function after(asyncId) { }

// destroy() is called when the resource is destroyed.
function destroy(asyncId) { }

// promiseResolve() is called only for promise resources, when the
// resolve() function passed to the Promise constructor is invoked
// (either directly or through other means of resolving a promise).
function promiseResolve(asyncId) { }
```

```cjs
const async_hooks = require('node:async_hooks');

// Return the ID of the current execution context.
const eid = async_hooks.executionAsyncId();

// Return the ID of the handle responsible for triggering the callback of the
// current execution scope to call.
const tid = async_hooks.triggerAsyncId();

// Create a new AsyncHook instance. All of these callbacks are optional.
const asyncHook =
    async_hooks.createHook({ init, before, after, destroy, promiseResolve });

// Allow callbacks of this AsyncHook instance to call. This is not an implicit
// action after running the constructor, and must be explicitly run to begin
// executing callbacks.
asyncHook.enable();

// Disable listening for new asynchronous events.
asyncHook.disable();

//
// The following are the callbacks that can be passed to createHook().
//

// init() is called during object construction. The resource may not have
// completed construction when this callback runs. Therefore, all fields of the
// resource referenced by "asyncId" may not have been populated.
function init(asyncId, type, triggerAsyncId, resource) { }

// before() is called just before the resource's callback is called. It can be
// called 0-N times for handles (such as TCPWrap), and will be called exactly 1
// time for requests (such as FSReqCallback).
function before(asyncId) { }

// after() is called just after the resource's callback has finished.
function after(asyncId) { }

// destroy() is called when the resource is destroyed.
function destroy(asyncId) { }

// promiseResolve() is called only for promise resources, when the
// resolve() function passed to the Promise constructor is invoked
// (either directly or through other means of resolving a promise).
function promiseResolve(asyncId) { }
```

## `async_hooks.createHook(callbacks)`

<!-- YAML
added: v8.1.0
-->

* `callbacks` {Object} The [Hook Callbacks][] to register
  * `init` {Function} The [`init` callback][].
  * `before` {Function} The [`before` callback][].
  * `after` {Function} The [`after` callback][].
  * `destroy` {Function} The [`destroy` callback][].
  * `promiseResolve` {Function} The [`promiseResolve` callback][].
* Returns: {AsyncHook} Instance used for disabling and enabling hooks

注册要为每个异步操作的不同生命周期事件调用的函数.

在资源的生命周期中，为相应的异步事件调用回调 `init()`/`before()`/`after()`/`destroy()`.

所有回调都是可选的。例如，如果只需要跟踪资源清理，则只需要传递 `destroy` 回调。可以传递给 `callbacks` 的所有函数的细节在 [Hook Callbacks][] 部分.

```mjs
import { createHook } from 'node:async_hooks';

const asyncHook = createHook({
  init(asyncId, type, triggerAsyncId, resource) { },
  destroy(asyncId) { }
});
```

```cjs
const async_hooks = require('node:async_hooks');

const asyncHook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId, resource) { },
  destroy(asyncId) { }
});
```

回调将通过原型链继承:

```js
class MyAsyncCallbacks {
  init(asyncId, type, triggerAsyncId, resource) { }
  destroy(asyncId) {}
}

class MyAddedCallbacks extends MyAsyncCallbacks {
  before(asyncId) { }
  after(asyncId) { }
}

const asyncHook = async_hooks.createHook(new MyAddedCallbacks());
```

因为 Promise 是通过异步钩子机制跟踪其生命周期的异步资源，所以 `init()`、`before()`、`after()` 和 `destroy()` 回调_不能_是返回 Promise 的异步函数.

### Error handling

如果任何 `AsyncHook` 回调抛出，应用程序将打印堆栈跟踪并退出。退出路径确实遵循未捕获异常的路径，但所有“未捕获异常”侦听器都被删除，从而强制进程退出。除非应用程序使用 `--abort-on-uncaught-exception` 运行，否则仍然会调用 `'exit'` 回调，在这种情况下，将打印堆栈跟踪并且应用程序退出，留下核心文件.

这种错误处理行为的原因是这些回调在对象生命周期中的潜在不稳定点运行，例如在
阶级的建构和毁灭。正因为如此，有必要迅速关闭该过程，以防止将来意外中止。如果执行综合分析以确保异常可以遵循正常的控制流程而不会产生意外的副作用，这可能会在未来发生变化.

### Printing in `AsyncHook` callbacks

因为打印到控制台是一个异步操作，`console.log()` 会导致 `AsyncHook` 回调被调用。在 `AsyncHook` 回调函数中使用 `console.log()` 或类似的异步操作将导致无限递归。调试时一个简单的解决方案是使用同步日志记录操作，例如`fs.writeFileSync(file, msg, flag)`.
这将打印到文件并且不会递归调用`AsyncHook`，因为它是同步的.

```mjs
import { writeFileSync } from 'node:fs';
import { format } from 'node:util';

function debug(...args) {
  // Use a function like this one when debugging inside an AsyncHook callback
  writeFileSync('log.out', `${format(...args)}\n`, { flag: 'a' });
}
```

```cjs
const fs = require('node:fs');
const util = require('node:util');

function debug(...args) {
  // Use a function like this one when debugging inside an AsyncHook callback
  fs.writeFileSync('log.out', `${util.format(...args)}\n`, { flag: 'a' });
}
```

如果日志记录需要异步操作，则可以使用 `AsyncHook` 本身提供的信息来跟踪导致异步操作的原因。当日志本身导致调用 `AsyncHook` 回调时，应该跳过日志记录。通过这样做，否则无限递归被打破.

## Class: `AsyncHook`

`AsyncHook` 类公开了一个用于跟踪异步操作的生命周期事件的接口.

### `asyncHook.enable()`

* Returns: {AsyncHook} A reference to `asyncHook`.

为给定的 `AsyncHook` 实例启用回调。如果没有提供回调，则启用是无操作的.

`AsyncHook` 实例默认是禁用的。如果 `AsyncHook` 实例应该在创建后立即启用，可以使用以下模式.

```mjs
import { createHook } from 'node:async_hooks';

const hook = createHook(callbacks).enable();
```

```cjs
const async_hooks = require('node:async_hooks');

const hook = async_hooks.createHook(callbacks).enable();
```

### `asyncHook.disable()`

* Returns: {AsyncHook} A reference to `asyncHook`.

从要执行的 `AsyncHook` 回调全局池中禁用给定 `AsyncHook` 实例的回调。一旦一个钩子被禁用，它不会被再次调用，直到启用.

为了 API 一致性，`disable()` 还返回 `AsyncHook` 实例.

### Hook callbacks

异步事件生命周期中的关键事件分为四个区域：实例化、回调调用前后、实例销毁时.

#### `init(asyncId, type, triggerAsyncId, resource)`

* `asyncId` {number} A unique ID for the async resource.
* `type` {string} The type of the async resource.
* `triggerAsyncId` {number} The unique ID of the async resource in whose
  execution context this async resource was created.
* `resource` {Object} Reference to the resource representing the async
  operation, needs to be released during _destroy_.

在构造具有_possibility_ 以发出异步事件的类时调用。这_并不_意味着实例必须在调用 `destroy` 之前调用 `before`/`after`，只是存在这种可能性.

可以通过执行诸如打开资源然后在可以使用资源之前将其关闭之类的操作来观察此行为。下面的代码片段演示了这一点.

```mjs
import { createServer } from 'node:net';

createServer().listen(function() { this.close(); });
// OR
clearTimeout(setTimeout(() => {}, 10));
```

```cjs
require('node:net').createServer().listen(function() { this.close(); });
// OR
clearTimeout(setTimeout(() => {}, 10));
```

每个新资源都分配有一个在当前 Node.js 实例范围内唯一的 ID.

##### `type`

`type` 是一个字符串，用于标识导致调用 `init` 的资源类型。一般会对应资源的构造函数名称.

Valid values are:

```text
FSEVENTWRAP, FSREQCALLBACK, GETADDRINFOREQWRAP, GETNAMEINFOREQWRAP, HTTPINCOMINGMESSAGE,
HTTPCLIENTREQUEST, JSSTREAM, PIPECONNECTWRAP, PIPEWRAP, PROCESSWRAP, QUERYWRAP,
SHUTDOWNWRAP, SIGNALWRAP, STATWATCHER, TCPCONNECTWRAP, TCPSERVERWRAP, TCPWRAP,
TTYWRAP, UDPSENDWRAP, UDPWRAP, WRITEWRAP, ZLIB, SSLCONNECTION, PBKDF2REQUEST,
RANDOMBYTESREQUEST, TLSWRAP, Microtask, Timeout, Immediate, TickObject
```

这些值可以在任何 Node.js 版本中更改。此外，[`AsyncResource`][] 的用户可能会提供其他值.

还有 `PROMISE` 资源类型，用于跟踪 `Promise` 实例和它们调度的异步工作.

使用公共嵌入器 API 时，用户可以定义自己的“类型”.

可能存在类型名称冲突。鼓励嵌入器使用唯一的前缀，例如 npm 包名称，以防止在监听钩子时发生冲突.

##### `triggerAsyncId`

`triggerAsyncId` 是导致（或“触发”）新资源初始化并导致调用 `init` 的资源的 `asyncId`。这与 `async_hooks.executionAsyncId()` 不同，后者仅显示_何时_创建资源，而 `triggerAsyncId` 显示_为什么_创建资源.

下面是 `triggerAsyncId` 的简单演示:

```mjs
import { createHook, executionAsyncId } from 'node:async_hooks';
import { stdout } from 'node:process';
import net from 'node:net';

createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = executionAsyncId();
    fs.writeSync(
      stdout.fd,
      `${type}(${asyncId}): trigger: ${triggerAsyncId} execution: ${eid}\n`);
  }
}).enable();

net.createServer((conn) => {}).listen(8080);
```

```cjs
const { createHook, executionAsyncId } = require('node:async_hooks');
const { stdout } = require('node:process');
const net = require('node:net');

createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = executionAsyncId();
    fs.writeSync(
      stdout.fd,
      `${type}(${asyncId}): trigger: ${triggerAsyncId} execution: ${eid}\n`);
  }
}).enable();

net.createServer((conn) => {}).listen(8080);
```

使用“nc localhost 8080”访问服务器时的输出:

```console
TCPSERVERWRAP(5): trigger: 1 execution: 1
TCPWRAP(7): trigger: 5 execution: 0
```

`TCPSERVERWRAP` 是接收连接的服务器.

`TCPWRAP` 是来自客户端的新连接。当建立新连接时，会立即构造“TCPWrap”实例。这发生在任何 JavaScript 堆栈之外。 （0 的 `executionAsyncId()` 意味着它是从 C++ 执行的，上面没有 JavaScript 堆栈。）只有这些信息，就导致资源被创建的原因而言，不可能将资源链接在一起，所以 `triggerAsyncId` 被赋予了传播什么资源负责新资源存在的任务.

##### `resource`

`resource` 是一个对象，表示已初始化的实际异步资源。这可以包含有用的信息，这些信息可能会根据 `type` 的值而有所不同。例如，对于 `GETADDRINFOREQWRAP` 资源类型,
`resource` 提供了在 `net.Server.listen()` 中查找主机 IP 地址时使用的主机名。不支持访问此信息的 API，但使用 Embedder API，用户可以提供和记录自己的资源对象。例如，这样的资源对象可能包含正在执行的 SQL 查询.

在某些情况下，出于性能原因，资源对象会被重用，因此将其用作“WeakMap”中的键或向其添加属性是不安全的.

##### Asynchronous context example

下面是一个示例，其中包含有关在 `before` 和 `after` 调用之间调用 `init` 的附加信息，特别是 `listen()` 的回调将是什么样子。输出格式稍微复杂一些，使调用上下文更容易看到.

```js
const async_hooks = require('node:async_hooks');
const fs = require('node:fs');
const net = require('node:net');
const { fd } = process.stdout;

let indent = 0;
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    const indentStr = ' '.repeat(indent);
    fs.writeSync(
      fd,
      `${indentStr}${type}(${asyncId}):` +
      ` trigger: ${triggerAsyncId} execution: ${eid}\n`);
  },
  before(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(fd, `${indentStr}before:  ${asyncId}\n`);
    indent += 2;
  },
  after(asyncId) {
    indent -= 2;
    const indentStr = ' '.repeat(indent);
    fs.writeSync(fd, `${indentStr}after:  ${asyncId}\n`);
  },
  destroy(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(fd, `${indentStr}destroy:  ${asyncId}\n`);
  },
}).enable();

net.createServer(() => {}).listen(8080, () => {
  // Let's wait 10ms before logging the server started.
  setTimeout(() => {
    console.log('>>>', async_hooks.executionAsyncId());
  }, 10);
});
```

仅启动服务器的输出:

```console
TCPSERVERWRAP(5): trigger: 1 execution: 1
TickObject(6): trigger: 5 execution: 1
before:  6
  Timeout(7): trigger: 6 execution: 6
after:   6
destroy: 6
before:  7
>>> 7
  TickObject(8): trigger: 7 execution: 7
after:   7
before:  8
after:   8
```

如示例所示，`executionAsyncId()` 和 `execution` 分别指定当前执行上下文的值；这是通过调用`before`和`after`来描述的.

仅使用 `execution` 绘制资源分配结果图如下:

```console
  root(1)
     ^
     |
TickObject(6)
     ^
     |
 Timeout(7)
```

`TCPSERVERWRAP` 不是此图的一部分，尽管它是调用 `console.log()` 的原因。这是因为绑定到没有主机名的端口是一个同步操作，但是为了维护一个完全异步的 API，用户的回调被放置在一个 `process.nextTick()` 中。这就是为什么 `TickObject` 出现在输出中并且是 `.listen()` 回调的“父级”.

该图仅显示_何时_创建资源，而不是_为什么_，因此要跟踪_为什么_使用`triggerAsyncId`。可以用下图表示:

```console
 bootstrap(1)
     |
     ˅
TCPSERVERWRAP(5)
     |
     ˅
 TickObject(6)
     |
     ˅
  Timeout(7)
```

#### `before(asyncId)`

* `asyncId` {number}

当异步操作启动（例如 TCP 服务器接收新连接）或完成（例如将数据写入磁盘）时，将调用回调通知用户。 `before` 回调在所述回调执行之前被调用。 `asyncId` 是分配给即将执行回调的资源的唯一标识符.

`before` 回调将被调用 0 到 N 次。如果异步操作被取消，或者例如，如果 TCP 服务器没有接收到任何连接，则 `before` 回调通常会被调用 0 次。像 TCP 服务器这样的持久异步资源通常会多次调用 `before` 回调，而 `fs.open()` 等其他操作只会调用一次.

#### `after(asyncId)`

* `asyncId` {number}

在 `before` 中指定的回调完成后立即调用.

如果在回调执行期间发生未捕获的异常，则 `after` 将运行 _after_ 发出 `'uncaughtException'` 事件或运行 `domain` 的处理程序.

#### `destroy(asyncId)`

* `asyncId` {number}

在 `asyncId` 对应的资源被销毁后调用。它也从嵌入器 API `emitDestroy()` 异步调用.

一些资源依赖于垃圾收集进行清理，因此如果引用传递给 `init` 的 `resource` 对象，则可能永远不会调用 `destroy`，从而导致应用程序中的内存泄漏。如果资源不依赖于垃圾回收，那么这将不是问题.

#### `promiseResolve(asyncId)`

<!-- YAML
added: v8.6.0
-->

* `asyncId` {number}

在调用传递给“Promise”构造函数的“resolve”函数时调用（直接或通过其他解决promise的方法）.

`resolve()` 不做任何可观察到的同步工作.

如果通过假设另一个 `Promise` 的状态来解决`Promise`，则此时`Promise`不一定会被履行或拒绝.

```js
new Promise((resolve) => resolve(true)).then((a) => {});
```

调用以下回调:

```text
init for PROMISE with id 5, trigger id: 1
  promise resolve 5      # corresponds to resolve(true)
init for PROMISE with id 6, trigger id: 5  # the Promise returned by then()
  before 6               # the then() callback is entered
  promise resolve 6      # the then() callback resolves the promise by returning
  after 6
```

### `async_hooks.executionAsyncResource()`

<!-- YAML
added:
 - v13.9.0
 - v12.17.0
-->

* Returns: {Object} The resource representing the current execution.
  用于在资源中存储数据.

`executionAsyncResource()` 返回的资源对象通常是带有未记录 API 的内部 Node.js 句柄对象。在对象上使用任何函数或属性可能会使您的应用程序崩溃，应避免.

在顶级执行上下文中使用 `executionAsyncResource()` 将返回一个空对象，因为没有要使用的句柄或请求对象,
但是拥有一个代表顶层的对象可能会有所帮助.

```mjs
import { open } from 'node:fs';
import { executionAsyncId, executionAsyncResource } from 'node:async_hooks';

console.log(executionAsyncId(), executionAsyncResource());  // 1 {}
open(new URL(import.meta.url), 'r', (err, fd) => {
  console.log(executionAsyncId(), executionAsyncResource());  // 7 FSReqWrap
});
```

```cjs
const { open } = require('node:fs');
const { executionAsyncId, executionAsyncResource } = require('node:async_hooks');

console.log(executionAsyncId(), executionAsyncResource());  // 1 {}
open(__filename, 'r', (err, fd) => {
  console.log(executionAsyncId(), executionAsyncResource());  // 7 FSReqWrap
});
```

这可用于实现连续本地存储，而无需使用跟踪 `Map` 来存储元数据:

```mjs
import { createServer } from 'node:http';
import {
  executionAsyncId,
  executionAsyncResource,
  createHook
} from 'async_hooks';
const sym = Symbol('state'); // Private symbol to avoid pollution

createHook({
  init(asyncId, type, triggerAsyncId, resource) {
    const cr = executionAsyncResource();
    if (cr) {
      resource[sym] = cr[sym];
    }
  }
}).enable();

const server = createServer((req, res) => {
  executionAsyncResource()[sym] = { state: req.url };
  setTimeout(function() {
    res.end(JSON.stringify(executionAsyncResource()[sym]));
  }, 100);
}).listen(3000);
```

```cjs
const { createServer } = require('node:http');
const {
  executionAsyncId,
  executionAsyncResource,
  createHook
} = require('node:async_hooks');
const sym = Symbol('state'); // Private symbol to avoid pollution

createHook({
  init(asyncId, type, triggerAsyncId, resource) {
    const cr = executionAsyncResource();
    if (cr) {
      resource[sym] = cr[sym];
    }
  }
}).enable();

const server = createServer((req, res) => {
  executionAsyncResource()[sym] = { state: req.url };
  setTimeout(function() {
    res.end(JSON.stringify(executionAsyncResource()[sym]));
  }, 100);
}).listen(3000);
```

### `async_hooks.executionAsyncId()`

<!-- YAML
added: v8.1.0
changes:
  - version: v8.2.0
    pr-url: https://github.com/nodejs/node/pull/13490
    description: Renamed from `currentId`.
-->

* Returns: {number} The `asyncId` of the current execution context. 用于跟踪何时调用.

```mjs
import { executionAsyncId } from 'node:async_hooks';

console.log(executionAsyncId());  // 1 - bootstrap
fs.open(path, 'r', (err, fd) => {
  console.log(executionAsyncId());  // 6 - open()
});
```

```cjs
const async_hooks = require('node:async_hooks');

console.log(async_hooks.executionAsyncId());  // 1 - bootstrap
fs.open(path, 'r', (err, fd) => {
  console.log(async_hooks.executionAsyncId());  // 6 - open()
});
```

从`executionAsyncId()`返回的ID与执行时间有关，而不是因果关系（由`triggerAsyncId()`覆盖）:

```js
const server = net.createServer((conn) => {
  // Returns the ID of the server, not of the new connection, because the
  // callback runs in the execution scope of the server's MakeCallback().
  async_hooks.executionAsyncId();

}).listen(port, () => {
  // Returns the ID of a TickObject (process.nextTick()) because all
  // callbacks passed to .listen() are wrapped in a nextTick().
  async_hooks.executionAsyncId();
});
```

默认情况下，Promise 上下文可能无法获得精确的 `executionAsyncIds`.
See the section on [promise execution tracking][].

### `async_hooks.triggerAsyncId()`

* Returns: {number} 负责调用当前正在执行的回调的资源ID.

```js
const server = net.createServer((conn) => {
  // The resource that caused (or triggered) this callback to be called
  // was that of the new connection. Thus the return value of triggerAsyncId()
  // is the asyncId of "conn".
  async_hooks.triggerAsyncId();

}).listen(port, () => {
  // Even though all callbacks passed to .listen() are wrapped in a nextTick()
  // the callback itself exists because the call to the server's .listen()
  // was made. So the return value would be the ID of the server.
  async_hooks.triggerAsyncId();
});
```

默认情况下，Promise 上下文可能无法获得有效的 `triggerAsyncId`。请参阅[承诺执行跟踪][]部分.

### `async_hooks.asyncWrapProviders`

<!-- YAML
added:
  - v17.2.0
  - v16.14.0
-->

* Returns: 提供者类型到相应数字 id 的映射.
  此映射包含 `async_hooks.init()` 事件可能发出的所有事件类型.

此功能禁止使用 `process.binding('async_wrap').Providers`.
See: [DEP0111][]

## Promise execution tracking

默认情况下，由于 V8 提供的 [promise introspection API][PromiseHooks] 相对昂贵的性质，promise 执行不会分配 `asyncId`。这意味着默认情况下，使用 Promise 或 `async`/`await` 的程序将无法正确执行并触发 Promise 回调上下文的 id.

```mjs
import { executionAsyncId, triggerAsyncId } from 'node:async_hooks';

Promise.resolve(1729).then(() => {
  console.log(`eid ${executionAsyncId()} tid ${triggerAsyncId()}`);
});
// produces:
// eid 1 tid 0
```

```cjs
const { executionAsyncId, triggerAsyncId } = require('node:async_hooks');

Promise.resolve(1729).then(() => {
  console.log(`eid ${executionAsyncId()} tid ${triggerAsyncId()}`);
});
// produces:
// eid 1 tid 0
```

观察到 `then()` 回调声称已在外部范围的上下文中执行，即使涉及异步跃点。此外，`triggerAsyncId` 值为 `0`，这意味着我们缺少有关导致（触发）`then()` 回调被执行的资源的上下文.

通过 `async_hooks.createHook` 安装异步钩子可以启用 Promise 执行跟踪:

```mjs
import { createHook, executionAsyncId, triggerAsyncId } from 'node:async_hooks';
createHook({ init() {} }).enable(); // forces PromiseHooks to be enabled.
Promise.resolve(1729).then(() => {
  console.log(`eid ${executionAsyncId()} tid ${triggerAsyncId()}`);
});
// produces:
// eid 7 tid 6
```

```cjs
const { createHook, executionAsyncId, triggerAsyncId } = require('node:async_hooks');

createHook({ init() {} }).enable(); // forces PromiseHooks to be enabled.
Promise.resolve(1729).then(() => {
  console.log(`eid ${executionAsyncId()} tid ${triggerAsyncId()}`);
});
// produces:
// eid 7 tid 6
```

在这个例子中，添加任何实际的钩子函数都启用了对 Promise 的跟踪。上面的例子中有两个 Promise； `Promise.resolve()` 创建的 promise 和调用 `then()` 返回的 promise。在上面的示例中，第一个 Promise 获得了 `asyncId` `6`，而后者获得了 `asyncId` `7`。在执行 `then()` 回调期间，我们正在使用 `asyncId` `7` 的 promise 上下文中执行。这个承诺是由异步资源“6”触发的.

Promise 的另一个微妙之处是 `before` 和 `after` 回调仅在链式 Promise 上运行。这意味着不是由 `then()`/`catch()` 创建的 Promise 不会触发 `before` 和 `after` 回调。有关更多详细信息，请参阅 V8 [PromiseHooks][] API 的详细信息.

## JavaScript embedder API

处理自己的异步资源执行 I/O、连接池或管理回调队列等任务的库开发人员可以使用 `AsyncResource` JavaScript API，以便调用所有适当的回调.

### Class: `AsyncResource`

此类的文档已移动 [`AsyncResource`][].

## Class: `AsyncLocalStorage`

此类的文档已移动 [`AsyncLocalStorage`][].

[DEP0111]: deprecations.md#dep0111-processbinding
[Hook Callbacks]: #hook-callbacks
[PromiseHooks]: https://docs.google.com/document/d/1rda3yKGHimKIhg5YeoAmCOtyURgsbTH_qaYR79FELlk/edit
[`AsyncLocalStorage`]: async_context.md#class-asynclocalstorage
[`AsyncResource`]: async_context.md#class-asyncresource
[`Worker`]: worker_threads.md#class-worker
[`after` callback]: #afterasyncid
[`before` callback]: #beforeasyncid
[`destroy` callback]: #destroyasyncid
[`init` callback]: #initasyncid-type-triggerasyncid-resource
[`promiseResolve` callback]: #promiseresolveasyncid
[promise execution tracking]: #promise-execution-tracking
