# Worker threads

<!--introduced_in=v10.5.0-->

> Stability: 2 - Stable

<!-- source_link=lib/worker_threads.js -->

`node:worker_threads` 模块允许使用并行执行 JavaScript 的线程。访问它:

```js
const worker = require('node:worker_threads');
```

工作者（线程）对于执行 CPU 密集型 JavaScript 操作很有用.
它们对 I/O 密集型工作没有多大帮助。 Node.js 内置的异步 I/O 操作比 Workers 效率更高.

与 `child_process` 或 `cluster` 不同，`worker_threads` 可以共享内存。他们通过传输 `ArrayBuffer` 实例或共享 `SharedArrayBuffer` 实例来实现.

```js
const {
  Worker, isMainThread, parentPort, workerData
} = require('node:worker_threads');

if (isMainThread) {
  module.exports = function parseJSAsync(script) {
    return new Promise((resolve, reject) => {
      const worker = new Worker(__filename, {
        workerData: script
      });
      worker.on('message', resolve);
      worker.on('error', reject);
      worker.on('exit', (code) => {
        if (code !== 0)
          reject(new Error(`Worker stopped with exit code ${code}`));
      });
    });
  };
} else {
  const { parse } = require('some-js-parsing-library');
  const script = workerData;
  parentPort.postMessage(parse(script));
}
```

上面的示例为每个 `parseJSAsync()` 调用生成一个 Worker 线程。在实践中，使用一个工人池来完成这些类型的任务。否则，创建工人的开销可能会超过他们的收益.

实现工作池时，使用 [`AsyncResource`][] API 通知诊断工具（例如，提供异步堆栈跟踪）有关任务及其结果之间的相关性。有关示例实现，请参阅 `async_hooks` 文档中的 ["将 `AsyncResource` 用于 `Worker` 线程池"][async-resource-worker-pool].

工作线程默认继承非进程特定的选项。参考 [`Worker constructor options`][] 了解如何自定义工作线程选项，特别是 `argv` 和 `execArgv` 选项.

## `worker.getEnvironmentData(key)`

<!-- YAML
added:
  - v15.12.0
  - v14.18.0
changes:
  - version:
    - v17.5.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41272
    description: No longer experimental.
-->

* `key` {any} Any arbitrary, cloneable JavaScript value that can be used as a
  {Map} key.
* Returns: {any}

在工作线程中，`worker.getEnvironmentData()` 返回传递给生成线程的`worker.setEnvironmentData()` 的数据克隆.
每个新的 Worker 都会自动收到自己的环境数据副本.

```js
const {
  Worker,
  isMainThread,
  setEnvironmentData,
  getEnvironmentData,
} = require('node:worker_threads');

if (isMainThread) {
  setEnvironmentData('Hello', 'World!');
  const worker = new Worker(__filename);
} else {
  console.log(getEnvironmentData('Hello'));  // Prints 'World!'.
}
```

## `worker.isMainThread`

<!-- YAML
added: v10.5.0
-->

* {boolean}

如果此代码未在 [`Worker`][] 线程内运行，则为 `true`.

```js
const { Worker, isMainThread } = require('node:worker_threads');

if (isMainThread) {
  // This re-loads the current file inside a Worker instance.
  new Worker(__filename);
} else {
  console.log('Inside Worker!');
  console.log(isMainThread);  // Prints 'false'.
}
```

## `worker.markAsUntransferable(object)`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

将对象标记为不可转移。如果 `object` 出现在 [`port.postMessage()`][] 调用的传输列表中，它会被忽略.

特别是，这对于可以克隆而不是传输的对象以及发送端的其他对象使用的对象是有意义的.
例如，Node.js 用这个标记它用于其 [`Buffer` 池][`Buffer.allocUnsafe()`] 的 `ArrayBuffer`.

此操作无法撤消.

```js
const { MessageChannel, markAsUntransferable } = require('node:worker_threads');

const pooledBuffer = new ArrayBuffer(8);
const typedArray1 = new Uint8Array(pooledBuffer);
const typedArray2 = new Float64Array(pooledBuffer);

markAsUntransferable(pooledBuffer);

const { port1 } = new MessageChannel();
port1.postMessage(typedArray1, [ typedArray1.buffer ]);

// The following line prints the contents of typedArray1 -- it still owns
// its memory and has been cloned, not transferred. Without
// `markAsUntransferable()`, this would print an empty Uint8Array.
// typedArray2 is intact as well.
console.log(typedArray1);
console.log(typedArray2);
```

浏览器中没有与此 API 等效的 API.

## `worker.moveMessagePortToContext(port, contextifiedSandbox)`

<!-- YAML
added: v11.13.0
-->

* `port` {MessagePort} The message port to transfer.

* `contextifiedSandbox` {Object} A [contextified][] object as returned by the
  `vm.createContext()` method.

* Returns: {MessagePort}

将 `MessagePort` 转移到不同的 [`vm`][] 上下文。原始的 `port` 对象被渲染为不可用，返回的 `MessagePort` 实例取而代之.

返回的 `MessagePort` 是目标上下文中的一个对象，继承自其全局 `Object` 类。传递给 [`port.onmessage()`][] 侦听器的对象也在目标上下文中创建，并从其全局 `Object` 类继承.

但是，创建的 `MessagePort` 不再继承自 [`EventTarget`][]，只能使用 [`port.onmessage()`][] 来接收使用它的事件.

## `worker.parentPort`

<!-- YAML
added: v10.5.0
-->

* {null|MessagePort}

如果此线程是 [`Worker`][]，则这是允许与父线程通信的 [`MessagePort`][]。使用 `parentPort.postMessage()` 发送的消息在父线程中使用 `worker.on('message')` 可用，而使用 `worker.postMessage()` 从父线程发送的消息在此线程中使用 ` parentPort.on（'消息'）`.

```js
const { Worker, isMainThread, parentPort } = require('node:worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  worker.once('message', (message) => {
    console.log(message);  // Prints 'Hello, world!'.
  });
  worker.postMessage('Hello, world!');
} else {
  // When a message from the parent thread is received, send it back:
  parentPort.once('message', (message) => {
    parentPort.postMessage(message);
  });
}
```

## `worker.receiveMessageOnPort(port)`

<!-- YAML
added: v12.3.0
changes:
  - version: v15.12.0
    pr-url: https://github.com/nodejs/node/pull/37535
    description: The port argument can also refer to a `BroadcastChannel` now.
-->

* `port` {MessagePort|BroadcastChannel}

* Returns: {Object|undefined}

从给定的 `MessagePort` 接收一条消息。如果没有消息可用，则返回“未定义”，否则返回一个具有单个“消息”属性的对象，该对象包含消息有效负载，对应于“消息端口”队列中最旧的消息.

```js
const { MessageChannel, receiveMessageOnPort } = require('node:worker_threads');
const { port1, port2 } = new MessageChannel();
port1.postMessage({ hello: 'world' });

console.log(receiveMessageOnPort(port2));
// Prints: { message: { hello: 'world' } }
console.log(receiveMessageOnPort(port2));
// Prints: undefined
```

使用此函数时，不会发出 `'message'` 事件并且不会调用 `onmessage` 侦听器.

## `worker.resourceLimits`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

* {Object}
  * `maxYoungGenerationSizeMb` {number}
  * `maxOldGenerationSizeMb` {number}
  * `codeRangeSizeMb` {number}
  * `stackSizeMb` {number}

提供此 Worker 线程内的一组 JS 引擎资源约束.
如果 `resourceLimits` 选项被传递给 [`Worker`][] 构造函数，这匹配它的值.

如果在主线程中使用this，则其值为空对象.

## `worker.SHARE_ENV`

<!-- YAML
added: v11.14.0
-->

* {symbol}

可以作为 [`Worker`][] 构造函数的 `env` 选项传递的特殊值，表示当前线程和 Worker 线程应该共享对同一组环境变量的读写访问权限.

```js
const { Worker, SHARE_ENV } = require('node:worker_threads');
new Worker('process.env.SET_IN_WORKER = "foo"', { eval: true, env: SHARE_ENV })
  .on('exit', () => {
    console.log(process.env.SET_IN_WORKER);  // Prints 'foo'.
  });
```

## `worker.setEnvironmentData(key[, value])`

<!-- YAML
added:
  - v15.12.0
  - v14.18.0
changes:
  - version:
    - v17.5.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41272
    description: No longer experimental.
-->

* `key` {any} 可用作 {Map} 键的任意、可克隆的 JavaScript 值.
* `value` {any} 将被克隆并自动传递给所有新的 `Worker` 实例的任意、可克隆的 JavaScript 值。如果 `value` 作为 `undefined` 传递，则之前为 `key` 设置的任何值都将被删除.

`worker.setEnvironmentData()` API 设置当前线程中的 `worker.getEnvironmentData()` 的内容以及从当前上下文生成的所有新的 `Worker` 实例.

## `worker.threadId`

<!-- YAML
added: v10.5.0
-->

* {integer}

当前线程的整数标识符。在相应的工作对象（如果有的话）上，它可以作为 [`worker.threadId`][].
该值对于单个进程中的每个 [`Worker`][] 实例都是唯一的.

## `worker.workerData`

<!-- YAML
added: v10.5.0
-->

一个任意的 JavaScript 值，其中包含传递给此线程的 `Worker` 构造函数的数据的克隆.

根据[HTML结构化克隆算法][]，数据被克隆就像使用[`postMessage()`][`port.postMessage()`].

```js
const { Worker, isMainThread, workerData } = require('node:worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename, { workerData: 'Hello, world!' });
} else {
  console.log(workerData);  // Prints 'Hello, world!'.
}
```

## Class: `BroadcastChannel extends EventTarget`

<!-- YAML
added: v15.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41271
    description: No longer experimental.
-->

`BroadcastChannel` 的实例允许与绑定到相同频道名称的所有其他 `BroadcastChannel` 实例进行异步一对多通信.

```js
'use strict';

const {
  isMainThread,
  BroadcastChannel,
  Worker
} = require('node:worker_threads');

const bc = new BroadcastChannel('hello');

if (isMainThread) {
  let c = 0;
  bc.onmessage = (event) => {
    console.log(event.data);
    if (++c === 10) bc.close();
  };
  for (let n = 0; n < 10; n++)
    new Worker(__filename);
} else {
  bc.postMessage('hello from every worker');
  bc.close();
}
```

### `new BroadcastChannel(name)`

<!-- YAML
added: v15.4.0
-->

* `name` {any} 要连接的频道的名称。允许使用 `${name}` 将任何 JavaScript 值转换为字符串.

### `broadcastChannel.close()`

<!-- YAML
added: v15.4.0
-->

关闭“BroadcastChannel”连接.

### `broadcastChannel.onmessage`

<!-- YAML
added: v15.4.0
-->

* Type: {Function} 收到消息时使用单个“MessageEvent”参数调用.

### `broadcastChannel.onmessageerror`

<!-- YAML
added: v15.4.0
-->

* Type: {Function} 用接收到的消息调用不能反序列化.

### `broadcastChannel.postMessage(message)`

<!-- YAML
added: v15.4.0
-->

* `message` {any} Any cloneable JavaScript value.

### `broadcastChannel.ref()`

<!-- YAML
added: v15.4.0
-->

与`unref()`相反。如果它是唯一剩下的活动句柄（默认行为），则在先前的 `unref()`ed BroadcastChannel 上调用 `ref()` 不会让程序退出。如果端口是 `ref()`ed，再次调用 `ref()` 无效.

### `broadcastChannel.unref()`

<!-- YAML
added: v15.4.0
-->

如果这是事件系统中唯一的活动句柄，则在 BroadcastChannel 上调用 `unref()` 允许线程退出。如果 BroadcastChannel 已经 `unref()`ed 再次调用 `unref()` 无效.

## Class: `MessageChannel`

<!-- YAML
added: v10.5.0
-->

`worker.MessageChannel` 类的实例表示异步,
双向通信通道.
`MessageChannel` 没有自己的方法。 `new MessageChannel()` 产生一个具有 `port1` 和 `port2` 属性的对象，它们引用链接的 [`MessagePort`][] 实例.

```js
const { MessageChannel } = require('node:worker_threads');

const { port1, port2 } = new MessageChannel();
port1.on('message', (message) => console.log('received', message));
port2.postMessage({ foo: 'bar' });
// Prints: received { foo: 'bar' } from the `port1.on('message')` listener
```

## Class: `MessagePort`

<!-- YAML
added: v10.5.0
changes:
  - version:
    - v14.7.0
    pr-url: https://github.com/nodejs/node/pull/34057
    description: This class now inherits from `EventTarget` rather than
                 from `EventEmitter`.
-->

* Extends: {EventTarget}

`worker.MessagePort` 类的实例代表异步双向通信通道的一端。可用于在不同的 [`Worker`][] 之间传输结构化数据、内存区域和其他`MessagePort`.

此实现匹配 [browser `MessagePort`][]s.

### Event: `'close'`

<!-- YAML
added: v10.5.0
-->

一旦通道的任一侧断开连接，就会发出“关闭”事件.

```js
const { MessageChannel } = require('node:worker_threads');
const { port1, port2 } = new MessageChannel();

// Prints:
//   foobar
//   closed!
port2.on('message', (message) => console.log(message));
port2.on('close', () => console.log('closed!'));

port1.postMessage('foobar');
port1.close();
```

### Event: `'message'`

<!-- YAML
added: v10.5.0
-->

* `value` {any} The transmitted value

`'message'` 事件会针对任何传入的消息发出，包含 [`port.postMessage()`][] 的克隆输入.

此事件的侦听器接收到传递给 `postMessage()` 的 `value` 参数的克隆，并且没有其他参数.

### Event: `'messageerror'`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

* `error` {Error} An Error object

反序列化消息失败时会发出“messageerror”事件.

目前，当在接收端实例化发布的 JS 对象时发生错误时会发出此事件。这种情况很少见，但可能会发生，例如，当在 `vm.Context` 中接收某些 Node.js API 对象时（其中 Node.js API 当前不可用）.

### `port.close()`

<!-- YAML
added: v10.5.0
-->

禁止在连接的任一侧进一步发送消息.
当通过此 `MessagePort` 不会发生进一步的通信时，可以调用此方法.

[`'close'` 事件][] 在作为通道一部分的两个 `MessagePort` 实例上发出.

### `port.postMessage(value[, transferList])`

<!-- YAML
added: v10.5.0
changes:
  - version:
      - v15.14.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37917
    description: Add 'BlockList' to the list of cloneable types.
  - version:
      - v15.9.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37155
    description: Add 'Histogram' types to the list of cloneable types.
  - version: v15.6.0
    pr-url: https://github.com/nodejs/node/pull/36804
    description: Added `X509Certificate` to the list of cloneable types.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: Added `CryptoKey` to the list of cloneable types.
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33360
    description: Added `KeyObject` to the list of cloneable types.
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33772
    description: Added `FileHandle` to the list of transferable types.
-->

* `value` {any}
* `transferList` {Object\[]}

向该通道的接收端发送一个 JavaScript 值.
`value` 以与 [HTML 结构化克隆算法] [] 兼容的方式传输.

特别是，与“JSON”的显着差异是:

* `value` may contain circular references.
* `value` may contain instances of builtin JS types such as `RegExp`s,
  `BigInt`s, `Map`s, `Set`s, etc.
* `value` may contain typed arrays, both using `ArrayBuffer`s
  and `SharedArrayBuffer`s.
* `value` may contain [`WebAssembly.Module`][] instances.
* `value` may not contain native (C++-backed) objects other than:
  * {CryptoKey}s,
  * {FileHandle}s,
  * {Histogram}s,
  * {KeyObject}s,
  * {MessagePort}s,
  * {net.BlockList}s,
  * {net.SocketAddress}es,
  * {X509Certificate}s.

```js
const { MessageChannel } = require('node:worker_threads');
const { port1, port2 } = new MessageChannel();

port1.on('message', (message) => console.log(message));

const circularData = {};
circularData.foo = circularData;
// Prints: { foo: [Circular] }
port2.postMessage(circularData);
```

`transferList` 可以是 [`ArrayBuffer`][]、[`MessagePort`][] 和 [`FileHandle`][] 对象的列表.
传输后，它们在通道的发送端不再可用（即使它们不包含在 `value` 中）。与 [子进程][] 不同，目前不支持传输网络套接字等句柄.

如果 `value` 包含 [`SharedArrayBuffer`][] 实例，则可以从任一线程访问这些实例。它们不能在 `transferList` 中列出.

`value` 可能仍然包含不在 `transferList` 中的 `ArrayBuffer` 实例；在这种情况下，底层内存被复制而不是移动.

```js
const { MessageChannel } = require('node:worker_threads');
const { port1, port2 } = new MessageChannel();

port1.on('message', (message) => console.log(message));

const uint8Array = new Uint8Array([ 1, 2, 3, 4 ]);
// This posts a copy of `uint8Array`:
port2.postMessage(uint8Array);
// This does not copy data, but renders `uint8Array` unusable:
port2.postMessage(uint8Array, [ uint8Array.buffer ]);

// The memory for the `sharedUint8Array` is accessible from both the
// original and the copy received by `.on('message')`:
const sharedUint8Array = new Uint8Array(new SharedArrayBuffer(4));
port2.postMessage(sharedUint8Array);

// This transfers a freshly created message port to the receiver.
// This can be used, for example, to create communication channels between
// multiple `Worker` threads that are children of the same parent thread.
const otherChannel = new MessageChannel();
port2.postMessage({ port: otherChannel.port1 }, [ otherChannel.port1 ]);
```

消息对象立即克隆，发布后可以修改，没有副作用.

有关此 API 背后的序列化和反序列化机制的更多信息，请参阅[`node:v8` 模块的序列化 API][v8.serdes].

#### Considerations when transferring TypedArrays and Buffers

所有的 `TypedArray` 和 `Buffer` 实例都是底层 `ArrayBuffer` 的视图。也就是说，真正存储原始数据的是 `ArrayBuffer`，而 `TypedArray` 和 `Buffer` 对象提供了一种查看和操作数据的方式。在同一个 `ArrayBuffer` 实例上创建多个视图是可能的并且很常见.
使用传输列表传输 `ArrayBuffer` 时必须非常小心，因为这样做会导致共享相同 `ArrayBuffer` 的所有 `TypedArray` 和 `Buffer` 实例变得不可用.

```js
const ab = new ArrayBuffer(10);

const u1 = new Uint8Array(ab);
const u2 = new Uint16Array(ab);

console.log(u2.length);  // prints 5

port.postMessage(u1, [u1.buffer]);

console.log(u2.length);  // prints 0
```

对于 `Buffer` 实例，具体来说，底层的 `ArrayBuffer` 是否可以转移或克隆完全取决于实例的创建方式，而这通常无法可靠地确定.

`ArrayBuffer` 可以用 [`markAsUntransferable()`][] 标记以指示它应该始终被克隆并且永远不会被转移.

根据 `Buffer` 实例的创建方式，它可能拥有也可能不拥有其底层的 `ArrayBuffer`。除非知道 `Buffer` 实例拥有它，否则不得转移 `ArrayBuffer`。特别是，对于从内部 `Buffer` 池创建的 `Buffer`（例如使用 `Buffer.from()` 或 `Buffer.allocUnsafe()`），无法传输它们并且它们总是被克隆，这发送整个 `Buffer` 池的副本.
此行为可能会带来意外的更高内存使用率和可能的安全问题.

有关 `Buffer` 池的更多详细信息，请参阅 [`Buffer.allocUnsafe()`][].

使用 `Buffer.alloc()` 或 `Buffer.allocUnsafeSlow()` 创建的 `Buffer` 实例的 `ArrayBuffer` 始终可以被传输，但这样做会使这些 `ArrayBuffer` 的所有其他现有视图不可用.

#### Considerations when cloning objects with prototypes, classes, and accessors

因为对象克隆使用 [HTML 结构化克隆算法][]，所以不保留不可枚举的属性、属性访问器和对象原型。特别是，[`Buffer`][] 对象在接收端将被读取为普通的 [`Uint8Array`][]s，并且 JavaScript 类的实例将被克隆为普通 JavaScript 对象.

```js
const b = Symbol('b');

class Foo {
  #a = 1;
  constructor() {
    this[b] = 2;
    this.c = 3;
  }

  get d() { return 4; }
}

const { port1, port2 } = new MessageChannel();

port1.onmessage = ({ data }) => console.log(data);

port2.postMessage(new Foo());

// Prints: { c: 3 }
```

This limitation extends to many built-in objects, such as the global `URL` object:

```js
const { port1, port2 } = new MessageChannel();

port1.onmessage = ({ data }) => console.log(data);

port2.postMessage(new URL('https://example.org'));

// Prints: { }
```

### `port.hasRef()`

<!-- YAML
added: v18.1.0
-->

> Stability: 1 - Experimental

* Returns: {boolean}

如果为 true，`MessagePort` 对象将保持 Node.js 事件循环处于活动状态.

### `port.ref()`

<!-- YAML
added: v10.5.0
-->

与`unref()`相反。如果它是唯一剩下的活动句柄（默认行为），则在先前的 `unref()`ed 端口上调用 `ref()` 不会让程序退出。如果端口是 `ref()`ed，再次调用 `ref()` 无效.

如果使用 `.on('message')` 附加或删除侦听器，则端口会自动 `ref()`ed 和 `unref()`ed，具体取决于事件的侦听器是否存在.

### `port.start()`

<!-- YAML
added: v10.5.0
-->

开始在此 `MessagePort` 上接收消息。当将此端口用作事件发射器时，一旦附加了“消息”侦听器，就会自动调用它.

存在此方法是为了与 Web `MessagePort` API 进行奇偶校验。在 Node.js 中，它仅在不存在事件侦听器时用于忽略消息.
Node.js 在处理 .onmessage 方面也存在分歧。设置它会自动调用 `.start()`，但取消设置它会让消息排队，直到设置新的处理程序或丢弃端口.

### `port.unref()`

<!-- YAML
added: v10.5.0
-->

如果这是事件系统中唯一的活动句柄，则在端口上调用 `unref()` 允许线程退出。如果端口已经 `unref()`ed 再次调用 `unref()` 无效.

如果使用 `.on('message')` 附加或删除侦听器，则端口会自动 `ref()`ed 和 `unref()`ed，具体取决于事件的侦听器是否存在.

## Class: `Worker`

<!-- YAML
added: v10.5.0
-->

* Extends: {EventEmitter}

`Worker` 类代表一个独立的 JavaScript 执行线程.
大多数 Node.js API 都在其中可用.

Worker 环境中的显着差异是:

* The [`process.stdin`][], [`process.stdout`][], and [`process.stderr`][]
  streams may be redirected by the parent thread.
* The [`require('node:worker_threads').isMainThread`][] property is set to `false`.
* The [`require('node:worker_threads').parentPort`][] message port is available.
* [`process.exit()`][] does not stop the whole program, just the single thread,
  and [`process.abort()`][] is not available.
* [`process.chdir()`][] and `process` methods that set group or user ids
  are not available.
* [`process.env`][] is a copy of the parent thread's environment variables,
  unless otherwise specified. Changes to one copy are not visible in other
  threads, and are not visible to native add-ons (unless
  [`worker.SHARE_ENV`][] is passed as the `env` option to the
  [`Worker`][] constructor).
* [`process.title`][] cannot be modified.
* Signals are not delivered through [`process.on('...')`][Signals events].
* Execution may stop at any point as a result of [`worker.terminate()`][]
  being invoked.
* IPC channels from parent processes are not accessible.
* The [`trace_events`][] module is not supported.
* Native add-ons can only be loaded from multiple threads if they fulfill
  [certain conditions][Addons worker support].

可以在其他 `Worker` 中创建 `Worker` 实例.

与 [Web Workers][] 和 [`node:cluster` 模块][] 一样，可以通过线程间消息传递来实现双向通信。在内部，`Worker` 有一对内置的 [`MessagePort`][] 在创建 `Worker` 时已经相互关联。虽然父端的 `MessagePort` 对象没有直接暴露，但它的功能通过 [`worker.postMessage()`][] 和 [`worker.on('message')`][] 事件暴露父线程的`Worker`对象.

要创建自定义消息通道（鼓励使用默认的全局通道，因为它有助于关注点分离），用户可以在任一线程上创建一个 `MessageChannel` 对象，并将该 `MessageChannel` 上的一个 `MessagePort` 传递给通过预先存在的通道的其他线程，例如全局通道.

请参阅 [`port.postMessage()`][] 了解有关如何传递消息以及可以通过线程屏障成功传输哪种 JavaScript 值的更多信息.

```js
const assert = require('node:assert');
const {
  Worker, MessageChannel, MessagePort, isMainThread, parentPort
} = require('node:worker_threads');
if (isMainThread) {
  const worker = new Worker(__filename);
  const subChannel = new MessageChannel();
  worker.postMessage({ hereIsYourPort: subChannel.port1 }, [subChannel.port1]);
  subChannel.port2.on('message', (value) => {
    console.log('received:', value);
  });
} else {
  parentPort.once('message', (value) => {
    assert(value.hereIsYourPort instanceof MessagePort);
    value.hereIsYourPort.postMessage('the worker is sending this');
    value.hereIsYourPort.close();
  });
}
```

### `new Worker(filename[, options])`

<!-- YAML
added: v10.5.0
changes:
  - version: v14.9.0
    pr-url: https://github.com/nodejs/node/pull/34584
    description: The `filename` parameter can be a WHATWG `URL` object using
                 `data:` protocol.
  - version: v14.9.0
    pr-url: https://github.com/nodejs/node/pull/34394
    description: The `trackUnmanagedFds` option was set to `true` by default.
  - version:
    - v14.6.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34303
    description: The `trackUnmanagedFds` option was introduced.
  - version:
     - v13.13.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32278
    description: The `transferList` option was introduced.
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31664
    description: The `filename` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version:
     - v13.4.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30559
    description: The `argv` option was introduced.
  - version:
     - v13.2.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/26628
    description: The `resourceLimits` option was introduced.
-->

* `filename` {string|URL} The path to the Worker's main script or module. Must
  be either an absolute path or a relative path (i.e. relative to the
  current working directory) starting with `./` or `../`, or a WHATWG `URL`
  object using `file:` or `data:` protocol.
  When using a [`data:` URL][], the data is interpreted based on MIME type using
  the [ECMAScript module loader][].
  If `options.eval` is `true`, this is a string containing JavaScript code
  rather than a path.
* `options` {Object}
  * `argv` {any\[]} List of arguments which would be stringified and appended to
    `process.argv` in the worker. This is mostly similar to the `workerData`
    but the values are available on the global `process.argv` as if they
    were passed as CLI options to the script.
  * `env` {Object} If set, specifies the initial value of `process.env` inside
    the Worker thread. As a special value, [`worker.SHARE_ENV`][] may be used
    to specify that the parent thread and the child thread should share their
    environment variables; in that case, changes to one thread's `process.env`
    object affect the other thread as well. **Default:** `process.env`.
  * `eval` {boolean} If `true` and the first argument is a `string`, interpret
    the first argument to the constructor as a script that is executed once the
    worker is online.
  * `execArgv` {string\[]} List of node CLI options passed to the worker.
    V8 options (such as `--max-old-space-size`) and options that affect the
    process (such as `--title`) are not supported. If set, this is provided
    as [`process.execArgv`][] inside the worker. By default, options are
    inherited from the parent thread.
  * `stdin` {boolean} If this is set to `true`, then `worker.stdin`
    provides a writable stream whose contents appear as `process.stdin`
    inside the Worker. By default, no data is provided.
  * `stdout` {boolean} If this is set to `true`, then `worker.stdout` is
    not automatically piped through to `process.stdout` in the parent.
  * `stderr` {boolean} If this is set to `true`, then `worker.stderr` is
    not automatically piped through to `process.stderr` in the parent.
  * `workerData` {any} Any JavaScript value that is cloned and made
    available as [`require('node:worker_threads').workerData`][]. The cloning
    occurs as described in the [HTML structured clone algorithm][], and an error
    is thrown if the object cannot be cloned (e.g. because it contains
    `function`s).
  * `trackUnmanagedFds` {boolean} If this is set to `true`, then the Worker
    tracks raw file descriptors managed through [`fs.open()`][] and
    [`fs.close()`][], and closes them when the Worker exits, similar to other
    resources like network sockets or file descriptors managed through
    the [`FileHandle`][] API. This option is automatically inherited by all
    nested `Worker`s. **Default:** `true`.
  * `transferList` {Object\[]} If one or more `MessagePort`-like objects
    are passed in `workerData`, a `transferList` is required for those
    items or [`ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST`][] is thrown.
    See [`port.postMessage()`][] for more information.
  * `resourceLimits` {Object} An optional set of resource limits for the new
    JS engine instance. Reaching these limits leads to termination of the
    `Worker` instance. These limits only affect the JS engine, and no external
    data, including no `ArrayBuffer`s. Even if these limits are set, the process
    may still abort if it encounters a global out-of-memory situation.
    * `maxOldGenerationSizeMb` {number} The maximum size of the main heap in MB.
    * `maxYoungGenerationSizeMb` {number} The maximum size of a heap space for
      recently created objects.
    * `codeRangeSizeMb` {number} The size of a pre-allocated memory range
      used for generated code.
    * `stackSizeMb` {number} The default maximum stack size for the thread.
      Small values may lead to unusable Worker instances. **Default:** `4`.

### Event: `'error'`

<!-- YAML
added: v10.5.0
-->

* `err` {Error}

The `'error'` event is emitted if the worker thread throws an uncaught exception. In that case, the worker is terminated.

### Event: `'exit'`

<!-- YAML
added: v10.5.0
-->

* `exitCode` {integer}

一旦工人停止，就会发出“退出”事件。如果 worker 通过调用 [`process.exit()`][] 退出，`exitCode` 参数是传递的退出代码。如果工作人员被终止，则 `exitCode` 参数为 `1`.

这是任何 `Worker` 实例发出的最终事件.

### Event: `'message'`

<!-- YAML
added: v10.5.0
-->

* `value` {any} The transmitted value

当工作线程调用 [`require('node:worker_threads').parentPort.postMessage()`][] 时，会发出 `'message'` 事件.
查看 [`port.on('message')`][] 事件了解更多详情.

从工作线程发送的所有消息都在 [`'exit'` 事件][] 在 `Worker` 对象上发出之前发出.

### Event: `'messageerror'`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

* `error` {Error} An Error object

反序列化消息失败时会发出“messageerror”事件.

### Event: `'online'`

<!-- YAML
added: v10.5.0
-->

当工作线程开始执行 JavaScript 代码时，会发出 `'online'` 事件.

### `worker.getHeapSnapshot()`

<!-- YAML
added:
 - v13.9.0
 - v12.17.0
-->

* Returns: {Promise} A promise for a Readable Stream containing
  a V8 heap snapshot

返回 Worker 当前状态的 V8 快照的可读流.
有关详细信息，请参阅 [`v8.getHeapSnapshot()`][].

如果 Worker 线程不再运行，这可能发生在发出 [`'exit'` 事件][] 之前，返回的 `Promise` 会立即被拒绝，并出现 [`ERR_WORKER_NOT_RUNNING`][] 错误.

### `worker.performance`

<!-- YAML
added:
  - v15.1.0
  - v14.17.0
  - v12.22.0
-->

可用于从工作实例查询性能信息的对象。类似于 [`perf_hooks.performance`][].

#### `performance.eventLoopUtilization([utilization1[, utilization2]])`

<!-- YAML
added:
  - v15.1.0
  - v14.17.0
  - v12.22.0
-->

* `utilization1` {Object} The result of a previous call to
  `eventLoopUtilization()`.
* `utilization2` {Object} The result of a previous call to
  `eventLoopUtilization()` prior to `utilization1`.
* Returns {Object}
  * `idle` {number}
  * `active` {number}
  * `utilization` {number}

与 [`perf_hooks` `eventLoopUtilization()`][] 相同的调用，除了返回工作实例的值.

一个区别是，与主线程不同，worker 内的引导是在事件循环内完成的。因此，一旦 worker 的脚本开始执行，事件循环的利用率就立即可用.

没有增加的“空闲”时间并不表示工作人员被困在引导程序中。下面的例子展示了worker的整个生命周期如何从不积累任何“空闲”时间，但仍然能够处理消息.

```js
const { Worker, isMainThread, parentPort } = require('node:worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  setInterval(() => {
    worker.postMessage('hi');
    console.log(worker.performance.eventLoopUtilization());
  }, 100).unref();
  return;
}

parentPort.on('message', () => console.log('msg')).unref();
(function r(n) {
  if (--n < 0) return;
  const t = Date.now();
  while (Date.now() - t < 300);
  setImmediate(r, n);
})(10);
```

工作人员的事件循环利用仅在 [`'online'` 事件][] 发出后可用，如果在此之前或在 [`'exit'` 事件][] 之后调用，则所有属性都有值的“0”。

### `worker.postMessage(value[, transferList])`

<!-- YAML
added: v10.5.0
-->

* `value` {any}
* `transferList` {Object\[]}

向通过 [`require('node:worker_threads').parentPort.on('message')`][] 接收的工作人员发送消息.
有关详细信息，请参阅 [`port.postMessage()`][].

### `worker.ref()`

<!-- YAML
added: v10.5.0
-->

在 `unref()` 的对面，在先前 `unref()`ed worker 上调用 `ref()` 不会让程序退出，如果它是唯一的活动句柄（默认行为）。如果worker是`ref()`ed，再次调用`ref()`没有效果.

### `worker.resourceLimits`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

* {Object}
  * `maxYoungGenerationSizeMb` {number}
  * `maxOldGenerationSizeMb` {number}
  * `codeRangeSizeMb` {number}
  * `stackSizeMb` {number}

为这个 Worker 线程提供一组 JS 引擎资源约束.
如果 `resourceLimits` 选项被传递给 [`Worker`][] 构造函数，这匹配它的值.

如果worker已经停止，返回值为空对象.

### `worker.stderr`

<!-- YAML
added: v10.5.0
-->

* {stream.Readable}

这是一个可读流，其中包含写入工作线程内 [`process.stderr`][] 的数据。如果 `stderr: true` 没有传递给 [`Worker`][] 构造函数，则数据通过管道传输到父线程的 [`process.stderr`][] 流.

### `worker.stdin`

<!-- YAML
added: v10.5.0
-->

* {null|stream.Writable}

如果 `stdin: true` 被传递给 [`Worker`][] 构造函数，这是一个可写流。写入此流的数据将在工作线程中以 [`process.stdin`][] 的形式提供.

### `worker.stdout`

<!-- YAML
added: v10.5.0
-->

* {stream.Readable}

这是一个可读流，其中包含写入工作线程内 [`process.stdout`][] 的数据。如果 `stdout: true` 未传递给 [`Worker`][] 构造函数，则数据通过管道传输到父线程的 [`process.stdout`][] 流.

### `worker.terminate()`

<!-- YAML
added: v10.5.0
changes:
  - version: v12.5.0
    pr-url: https://github.com/nodejs/node/pull/28021
    description: This function now returns a Promise.
                 Passing a callback is deprecated, and was useless up to this
                 version, as the Worker was actually terminated synchronously.
                 Terminating is now a fully asynchronous operation.
-->

* Returns: {Promise}

尽快停止工作线程中的所有 JavaScript 执行.
为发出 [`'exit'` 事件][] 时完成的退出代码返回一个 Promise.

### `worker.threadId`

<!-- YAML
added: v10.5.0
-->

* {integer}

被引用线程的整数标识符。在工作线程内部，它可以作为 [`require('node:worker_threads').threadId`][].
该值对于单个进程中的每个 `Worker` 实例都是唯一的.

### `worker.unref()`

<!-- YAML
added: v10.5.0
-->

如果这是事件系统中唯一的活动句柄，则在工作线程上调用 `unref()` 允许线程退出。如果工人已经 `unref()`ed 再次调用 `unref()` 无效.

## Notes

### Synchronous blocking of stdio

`Worker`s 利用通过 {MessagePort} 的消息传递来实现与`stdio` 的交互。这意味着来自 `Worker` 的 `stdio` 输出可能会被阻塞 Node.js 事件循环的接收端的同步代码阻塞.

```mjs
import {
  Worker,
  isMainThread,
} from 'worker_threads';

if (isMainThread) {
  new Worker(new URL(import.meta.url));
  for (let n = 0; n < 1e10; n++) {
    // Looping to simulate work.
  }
} else {
  // This output will be blocked by the for loop in the main thread.
  console.log('foo');
}
```

```cjs
'use strict';

const {
  Worker,
  isMainThread,
} = require('node:worker_threads');

if (isMainThread) {
  new Worker(__filename);
  for (let n = 0; n < 1e10; n++) {
    // Looping to simulate work.
  }
} else {
  // This output will be blocked by the for loop in the main thread.
  console.log('foo');
}
```

### Launching worker threads from preload scripts

从预加载脚本（使用 `-r` 命令行标志加载和运行的脚本）启动工作线程时要小心。除非明确设置了 `execArgv` 选项，否则新的 Worker 线程会自动从正在运行的进程继承命令行标志，并将预加载与主线程相同的预加载脚本。如果预加载脚本无条件地启动一个工作线程，每个产生的线程都会产生另一个，直到应用程序崩溃.

[Addons worker support]: addons.md#worker-support
[ECMAScript module loader]: esm.md#data-imports
[HTML structured clone algorithm]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm
[Signals events]: process.md#signal-events
[Web Workers]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API
[`'close'` event]: #event-close
[`'exit'` event]: #event-exit
[`'online'` event]: #event-online
[`ArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer
[`AsyncResource`]: async_hooks.md#class-asyncresource
[`Buffer.allocUnsafe()`]: buffer.md#static-method-bufferallocunsafesize
[`Buffer`]: buffer.md
[`ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST`]: errors.md#err_missing_message_port_in_transfer_list
[`ERR_WORKER_NOT_RUNNING`]: errors.md#err_worker_not_running
[`EventTarget`]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget
[`FileHandle`]: fs.md#class-filehandle
[`MessagePort`]: #class-messageport
[`SharedArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer
[`Uint8Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array
[`WebAssembly.Module`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Module
[`Worker constructor options`]: #new-workerfilename-options
[`Worker`]: #class-worker
[`data:` URL]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs
[`fs.close()`]: fs.md#fsclosefd-callback
[`fs.open()`]: fs.md#fsopenpath-flags-mode-callback
[`markAsUntransferable()`]: #workermarkasuntransferableobject
[`node:cluster` module]: cluster.md
[`perf_hooks.performance`]: perf_hooks.md#perf_hooksperformance
[`perf_hooks` `eventLoopUtilization()`]: perf_hooks.md#performanceeventlooputilizationutilization1-utilization2
[`port.on('message')`]: #event-message
[`port.onmessage()`]: https://developer.mozilla.org/en-US/docs/Web/API/MessagePort/onmessage
[`port.postMessage()`]: #portpostmessagevalue-transferlist
[`process.abort()`]: process.md#processabort
[`process.chdir()`]: process.md#processchdirdirectory
[`process.env`]: process.md#processenv
[`process.execArgv`]: process.md#processexecargv
[`process.exit()`]: process.md#processexitcode
[`process.stderr`]: process.md#processstderr
[`process.stdin`]: process.md#processstdin
[`process.stdout`]: process.md#processstdout
[`process.title`]: process.md#processtitle
[`require('node:worker_threads').isMainThread`]: #workerismainthread
[`require('node:worker_threads').parentPort.on('message')`]: #event-message
[`require('node:worker_threads').parentPort.postMessage()`]: #workerpostmessagevalue-transferlist
[`require('node:worker_threads').parentPort`]: #workerparentport
[`require('node:worker_threads').threadId`]: #workerthreadid
[`require('node:worker_threads').workerData`]: #workerworkerdata
[`trace_events`]: tracing.md
[`v8.getHeapSnapshot()`]: v8.md#v8getheapsnapshot
[`vm`]: vm.md
[`worker.SHARE_ENV`]: #workershare_env
[`worker.on('message')`]: #event-message_1
[`worker.postMessage()`]: #workerpostmessagevalue-transferlist
[`worker.terminate()`]: #workerterminate
[`worker.threadId`]: #workerthreadid_1
[async-resource-worker-pool]: async_context.md#using-asyncresource-for-a-worker-thread-pool
[browser `MessagePort`]: https://developer.mozilla.org/en-US/docs/Web/API/MessagePort
[child processes]: child_process.md
[contextified]: vm.md#what-does-it-mean-to-contextify-an-object
[v8.serdes]: v8.md#serialization-api
