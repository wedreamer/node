# V8

<!--introduced_in=v4.0.0-->

<!-- source_link=lib/v8.js -->

`node:v8` 模块公开了特定于 Node.js 二进制文件中 [V8][] 版本的 API。它可以使用:

```js
const v8 = require('node:v8');
```

## `v8.cachedDataVersionTag()`

<!-- YAML
added: v8.0.0
-->

* Returns: {integer}

返回一个整数，表示从 V8 版本、命令行标志和检测到的 CPU 功能派生的版本标记。这对于确定 [`vm.Script`][] `cachedData` 缓冲区是否与此 V8 实例兼容很有用.

```js
console.log(v8.cachedDataVersionTag()); // 3947234607
// The value returned by v8.cachedDataVersionTag() is derived from the V8
// version, command-line flags, and detected CPU features. Test that the value
// does indeed update when flags are toggled.
v8.setFlagsFromString('--allow_natives_syntax');
console.log(v8.cachedDataVersionTag()); // 183726201
```

## `v8.getHeapCodeStatistics()`

<!-- YAML
added: v12.8.0
-->

* Returns: {Object}

获取堆中代码及其元数据的统计信息，请参阅 V8 [`GetHeapCodeAndMetadataStatistics`][] API。返回具有以下属性的对象:

* `code_and_metadata_size` {number}
* `bytecode_and_metadata_size` {number}
* `external_script_source_size` {number}
* `cpu_profiler_metadata_size` {number}

<!-- eslint-skip -->

```js
{
  code_and_metadata_size: 212208,
  bytecode_and_metadata_size: 161368,
  external_script_source_size: 1410794,
  cpu_profiler_metadata_size: 0,
}
```

## `v8.getHeapSnapshot()`

<!-- YAML
added: v11.13.0
-->

* Returns: {stream.Readable} A Readable Stream containing the V8 heap snapshot

生成当前 V8 堆的快照并返回可用于读取 JSON 序列化表示的可读流.
此 JSON 流格式旨在与 Chrome DevTools 等工具一起使用。 JSON 模式未记录且特定于 V8 引擎. 因此，模式可能会从 V8 的一个版本更改为下一个版本.

创建堆快照需要的内存大约是创建快照时堆大小的两倍。这会导致 OOM 杀手终止进程的风险.

生成快照是一种同步操作，它会根据堆大小在一段时间内阻塞事件循环.

```js
// Print heap snapshot to the console
const v8 = require('node:v8');
const stream = v8.getHeapSnapshot();
stream.pipe(process.stdout);
```

## `v8.getHeapSpaceStatistics()`

<!-- YAML
added: v6.0.0
changes:
  - version: v7.5.0
    pr-url: https://github.com/nodejs/node/pull/10186
    description: Support values exceeding the 32-bit unsigned integer range.
-->

* Returns: {Object\[]}

返回有关 V8 堆空间的统计信息，即组成 V8 堆的段。由于统计信息是通过 V8 [`GetHeapSpaceStatistics`][] 函数提供的，并且可能会从一个 V8 版本更改为下一个版本，因此既不能保证堆空间的顺序，也不能保证堆空间的可用性.

返回的值是一个包含以下属性的对象数组:

* `space_name` {string}
* `space_size` {number}
* `space_used_size` {number}
* `space_available_size` {number}
* `physical_space_size` {number}

```json
[
  {
    "space_name": "new_space",
    "space_size": 2063872,
    "space_used_size": 951112,
    "space_available_size": 80824,
    "physical_space_size": 2063872
  },
  {
    "space_name": "old_space",
    "space_size": 3090560,
    "space_used_size": 2493792,
    "space_available_size": 0,
    "physical_space_size": 3090560
  },
  {
    "space_name": "code_space",
    "space_size": 1260160,
    "space_used_size": 644256,
    "space_available_size": 960,
    "physical_space_size": 1260160
  },
  {
    "space_name": "map_space",
    "space_size": 1094160,
    "space_used_size": 201608,
    "space_available_size": 0,
    "physical_space_size": 1094160
  },
  {
    "space_name": "large_object_space",
    "space_size": 0,
    "space_used_size": 0,
    "space_available_size": 1490980608,
    "physical_space_size": 0
  }
]
```

## `v8.getHeapStatistics()`

<!-- YAML
added: v1.0.0
changes:
  - version: v7.5.0
    pr-url: https://github.com/nodejs/node/pull/10186
    description: Support values exceeding the 32-bit unsigned integer range.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/8610
    description: Added `malloced_memory`, `peak_malloced_memory`,
                 and `does_zap_garbage`.
-->

* Returns: {Object}

返回具有以下属性的对象:

* `total_heap_size` {number}
* `total_heap_size_executable` {number}
* `total_physical_size` {number}
* `total_available_size` {number}
* `used_heap_size` {number}
* `heap_size_limit` {number}
* `malloced_memory` {number}
* `peak_malloced_memory` {number}
* `does_zap_garbage` {number}
* `number_of_native_contexts` {number}
* `number_of_detached_contexts` {number}
* `total_global_handles_size` {number}
* `used_global_handles_size` {number}
* `external_memory` {number}

`does_zap_garbage` 是一个 0/​​1 布尔值，表示是否启用了 `--zap_code_space` 选项。这使得 V8 使用位模式覆盖堆垃圾。 RSS 占用空间（驻留集大小）变得更大，因为它不断接触所有堆页面，这使得它们不太可能被操作系统换出.

`number_of_native_contexts` native\_context 的值是当前活动的顶级上下文的数量。此数字随时间增加表示内存泄漏.

`number_of_detached_contexts` detached\_context 的值是已分离但尚未被垃圾回收的上下文的数量。此数字非零表示潜在的内存泄漏.

`total_global_handles_size` total\_global\_handles\_size 的值是 V8 全局句柄的总内存大小.

`used_global_handles_size` used\_global\_handles\_size的值是V8全局句柄的已用内存大小.

`external_memory` external\_memory 的值是数组缓冲区和外部字符串的内存大小.

<!-- eslint-skip -->

```js
{
  total_heap_size: 7326976,
  total_heap_size_executable: 4194304,
  total_physical_size: 7326976,
  total_available_size: 1152656,
  used_heap_size: 3476208,
  heap_size_limit: 1535115264,
  malloced_memory: 16384,
  peak_malloced_memory: 1127496,
  does_zap_garbage: 0,
  number_of_native_contexts: 1,
  number_of_detached_contexts: 0,
  total_global_handles_size: 8192,
  used_global_handles_size: 3296,
  external_memory: 318824
}
```

## `v8.setFlagsFromString(flags)`

<!-- YAML
added: v1.0.0
-->

* `flags` {string}

`v8.setFlagsFromString()` 方法可用于以编程方式设置 V8 命令行标志。应谨慎使用此方法。在 VM 启动后更改设置可能会导致不可预知的行为，包括崩溃和数据丢失；或者它可能什么都不做.

可用于 Node.js 版本的 V8 选项可以通过运行 `node --v8-options` 来确定.

Usage:

```js
// Print GC events to stdout for one minute.
const v8 = require('node:v8');
v8.setFlagsFromString('--trace_gc');
setTimeout(() => { v8.setFlagsFromString('--notrace_gc'); }, 60e3);
```

## `v8.stopCoverage()`

<!-- YAML
added:
  - v15.1.0
  - v14.18.0
  - v12.22.0
-->

`v8.stopCoverage()` 方法允许用户停止由 [`NODE_V8_COVERAGE`][] 启动的覆盖率收集，以便 V8 可以释放执行计数记录并优化代码。如果用户想要按需收集覆盖率，这可以与 [`v8.takeCoverage()`][] 结合使用.

## `v8.takeCoverage()`

<!-- YAML
added:
  - v15.1.0
  - v14.18.0
  - v12.22.0
-->

`v8.takeCoverage()` 方法允许用户按需将 [`NODE_V8_COVERAGE`][] 开始的覆盖写入磁盘。该方法可以在进程的生命周期内多次调用。每次执行计数器将被重置并且新的覆盖率报告将被写入 [`NODE_V8_COVERAGE`][] 指定的目录.

当进程即将退出时，最后一次覆盖仍将写入磁盘，除非在进程退出之前调用 [`v8.stopCoverage()`][].

## `v8.writeHeapSnapshot([filename])`

<!-- YAML
added: v11.13.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41373
    description: An exception will now be thrown if the file could not be written.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42577
    description: Make the returned error codes consistent across all platforms.
-->

* `filename` {string} The file path where the V8 heap snapshot is to be
  saved. If not specified, a file name with the pattern
  `'Heap-${yyyymmdd}-${hhmmss}-${pid}-${thread_id}.heapsnapshot'` will be
  generated, where `{pid}` will be the PID of the Node.js process,
  `{thread_id}` will be `0` when `writeHeapSnapshot()` is called from
  the main Node.js thread or the id of a worker thread.
* Returns: {string} The filename where the snapshot was saved.

生成当前 V8 堆的快照并将其写入 JSON 文件。该文件旨在与 Chrome DevTools 等工具一起使用。 JSON 模式未记录且特定于 V8 引擎，并且可能会从 V8 的一个版本更改为下一个版本.

堆快照特定于单个 V8 隔离。使用 [worker threads][] 时，从主线程生成的堆快照将不包含任何有关工作线程的信息，反之亦然.

创建堆快照需要的内存大约是创建快照时堆大小的两倍。这会导致 OOM 杀手终止进程的风险.

生成快照是一种同步操作，它会根据堆大小在一段时间内阻塞事件循环.

```js
const { writeHeapSnapshot } = require('node:v8');
const {
  Worker,
  isMainThread,
  parentPort
} = require('node:worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);

  worker.once('message', (filename) => {
    console.log(`worker heapdump: ${filename}`);
    // Now get a heapdump for the main thread.
    console.log(`main thread heapdump: ${writeHeapSnapshot()}`);
  });

  // Tell the worker to create a heapdump.
  worker.postMessage('heapdump');
} else {
  parentPort.once('message', (message) => {
    if (message === 'heapdump') {
      // Generate a heapdump for the worker
      // and return the filename to the parent.
      parentPort.postMessage(writeHeapSnapshot());
    }
  });
}
```

## Serialization API

序列化 API 提供了以与 [HTML 结构化克隆算法][] 兼容的方式序列化 JavaScript 值的方法.

该格式是向后兼容的（即可以安全地存储到磁盘）.
相同的 JavaScript 值可能会导致不同的序列化输出.

### `v8.serialize(value)`

<!-- YAML
added: v8.0.0
-->

* `value` {any}
* Returns: {Buffer}

使用 [`DefaultSerializer`][] 将 `value` 序列化到缓冲区中.

[`ERR_BUFFER_TOO_LARGE`][] 将在尝试序列化需要缓冲区大于 [`buffer.constants.MAX_LENGTH`][] 的巨大对象时抛出.

### `v8.deserialize(buffer)`

<!-- YAML
added: v8.0.0
-->

* `buffer` {Buffer|TypedArray|DataView} A buffer returned by [`serialize()`][].

使用带有默认选项的 [`DefaultDeserializer`][] 从缓冲区中读取 JS 值.

### Class: `v8.Serializer`

<!-- YAML
added: v8.0.0
-->

#### `new Serializer()`

Creates a new `Serializer` object.

#### `serializer.writeHeader()`

写出一个标头，其中包括序列化格式版本.

#### `serializer.writeValue(value)`

* `value` {any}

序列化一个 JavaScript 值并将序列化的表示添加到内部缓冲区.

如果 `value` 不能被序列化，这会引发错误.

#### `serializer.releaseBuffer()`

* Returns: {Buffer}

返回存储的内部缓冲区。释放缓冲区后，不应使用此序列化程序。如果先前的写入失败，则调用此方法会导致未定义的行为.

#### `serializer.transferArrayBuffer(id, arrayBuffer)`

* `id` {integer} A 32-bit unsigned integer.
* `arrayBuffer` {ArrayBuffer} An `ArrayBuffer` instance.

将 `ArrayBuffer` 标记为将其内容传输到带外.
将反序列化上下文中对应的 `ArrayBuffer` 传递给 [`deserializer.transferArrayBuffer()`][].

#### `serializer.writeUint32(value)`

* `value` {integer}

写一个原始的 32 位无符号整数.
用于自定义 [`serializer._writeHostObject()`][] 内部.

#### `serializer.writeUint64(hi, lo)`

* `hi` {integer}
* `lo` {integer}

写一个原始的 64 位无符号整数，分成高 32 位和低 32 位部分.
用于自定义 [`serializer._writeHostObject()`][] 内部.

#### `serializer.writeDouble(value)`

* `value` {number}

写一个 JS `number` 值.
用于自定义 [`serializer._writeHostObject()`][] 内部.

#### `serializer.writeRawBytes(buffer)`

* `buffer` {Buffer|TypedArray|DataView}

将原始字节写入序列化器的内部缓冲区。解串器将需要一种计算缓冲区长度的方法.
用于自定义 [`serializer._writeHostObject()`][] 内部.

#### `serializer._writeHostObject(object)`

* `object` {Object}

调用此方法来编写某种宿主对象，即由本机 C++ 绑定创建的对象。如果不能序列化`object`，应该抛出一个合适的异常.

`Serializer` 类本身不存在此方法，但可以由子类提供.

#### `serializer._getDataCloneError(message)`

* `message` {string}

调用此方法生成无法克隆对象时将抛出的错误对象.

此方法默认为 [`Error`][] 构造函数，并且可以在子类上覆盖.

#### `serializer._getSharedArrayBufferId(sharedArrayBuffer)`

* `sharedArrayBuffer` {SharedArrayBuffer}

当序列化程序要序列化 ​​`SharedArrayBuffer` 对象时，将调用此方法。它必须为对象返回一个无符号的 32 位整数 ID，如果此 `SharedArrayBuffer` 已被序列化，则使用相同的 ID。反序列化的时候，这个ID会传给[`deserializer.transferArrayBuffer()`][].

如果对象不能被序列化，应该抛出异常.

`Serializer` 类本身不存在此方法，但可以由子类提供.

#### `serializer._setTreatArrayBufferViewsAsHostObjects(flag)`

* `flag` {boolean} **Default:** `false`

指示是否将 `TypedArray` 和 `DataView` 对象视为宿主对象，即将它们传递给 [`serializer._writeHostObject()`][].

### Class: `v8.Deserializer`

<!-- YAML
added: v8.0.0
-->

#### `new Deserializer(buffer)`

* `buffer` {Buffer|TypedArray|DataView} A buffer returned by
  [`serializer.releaseBuffer()`][].

创建一个新的 `Deserializer` 对象.

#### `deserializer.readHeader()`

读取并验证标头（包括格式版本）.
例如，可能会拒绝无效或不受支持的电汇格式。在这种情况下，会抛出一个“错误”.

#### `deserializer.readValue()`

从缓冲区反序列化一个 JavaScript 值并返回它.

#### `deserializer.transferArrayBuffer(id, arrayBuffer)`

* `id` {integer} A 32-bit unsigned integer.
* `arrayBuffer` {ArrayBuffer|SharedArrayBuffer} An `ArrayBuffer` instance.

将 `ArrayBuffer` 标记为将其内容传输到带外.
将序列化上下文中的相应 `ArrayBuffer` 传递给 [`serializer.transferArrayBuffer()`][] （或在 `SharedArrayBuffer`s 的情况下从 [`serializer._getSharedArrayBufferId()`][] 返回 `id`）.

#### `deserializer.getWireFormatVersion()`

* Returns: {integer}

读取基础线格式版本。可能主要对读取旧线格式版本的遗留代码有用。不能在 `.readHeader()` 之前调用.

#### `deserializer.readUint32()`

* Returns: {integer}

读取一个原始的 32 位无符号整数并返回它.
用于自定义 [`deserializer._readHostObject()`][] 内部.

#### `deserializer.readUint64()`

* Returns: {integer\[]}

读取一个原始的 64 位无符号整数并将其作为数组返回，其中包含两个 32 位无符号整数条目.
用于自定义 [`deserializer._readHostObject()`][] 内部.

#### `deserializer.readDouble()`

* Returns: {number}

读取一个 JS `number` 值.
用于自定义 [`deserializer._readHostObject()`][] 内部.

#### `deserializer.readRawBytes(length)`

* `length` {integer}
* Returns: {Buffer}

从解串器的内部缓冲区读取原始字节。 `length` 参数必须对应于传递给 [`serializer.writeRawBytes()`][] 的缓冲区的长度.
用于自定义 [`deserializer._readHostObject()`][] 内部.

#### `deserializer._readHostObject()`

调用此方法来读取某种宿主对象，即由本机 C++ 绑定创建的对象。如果无法反序列化数据，则应抛出适当的异常.

`Deserializer` 类本身不存在此方法，但可以由子类提供.

### Class: `v8.DefaultSerializer`

<!-- YAML
added: v8.0.0
-->

[`Serializer`][] 的子类，将 `TypedArray`（特别是 [`Buffer`][]）和 `DataView` 对象序列化为宿主对象，并且仅存储它们的底层 `ArrayBuffer` 的一部分指.

### Class: `v8.DefaultDeserializer`

<!-- YAML
added: v8.0.0
-->

[`Deserializer`][] 的子类，对应 [`DefaultSerializer`][] 所写的格式.

## Promise hooks

`promiseHooks` 接口可用于跟踪 Promise 生命周期事件.
要跟踪_所有_异步活动，请参阅 [`async_hooks`][]，它在内部使用此模块生成承诺生命周期事件以及其他异步资源的事件。有关请求上下文管理，请参阅 [`AsyncLocalStorage`][].

```mjs
import { promiseHooks } from 'node:v8';

// There are four lifecycle events produced by promises:

// The `init` event represents the creation of a promise. This could be a
// direct creation such as with `new Promise(...)` or a continuation such
// as `then()` or `catch()`. It also happens whenever an async function is
// called or does an `await`. If a continuation promise is created, the
// `parent` will be the promise it is a continuation from.
function init(promise, parent) {
  console.log('a promise was created', { promise, parent });
}

// The `settled` event happens when a promise receives a resolution or
// rejection value. This may happen synchronously such as when using
// `Promise.resolve()` on non-promise input.
function settled(promise) {
  console.log('a promise resolved or rejected', { promise });
}

// The `before` event runs immediately before a `then()` or `catch()` handler
// runs or an `await` resumes execution.
function before(promise) {
  console.log('a promise is about to call a then handler', { promise });
}

// The `after` event runs immediately after a `then()` handler runs or when
// an `await` begins after resuming from another.
function after(promise) {
  console.log('a promise is done calling a then handler', { promise });
}

// Lifecycle hooks may be started and stopped individually
const stopWatchingInits = promiseHooks.onInit(init);
const stopWatchingSettleds = promiseHooks.onSettled(settled);
const stopWatchingBefores = promiseHooks.onBefore(before);
const stopWatchingAfters = promiseHooks.onAfter(after);

// Or they may be started and stopped in groups
const stopHookSet = promiseHooks.createHook({
  init,
  settled,
  before,
  after
});

// To stop a hook, call the function returned at its creation.
stopWatchingInits();
stopWatchingSettleds();
stopWatchingBefores();
stopWatchingAfters();
stopHookSet();
```

### `promiseHooks.onInit(init)`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

* `init` {Function} The [`init` callback][] to call when a promise is created.
* Returns: {Function} Call to stop the hook.

**`init` 钩子必须是一个普通函数。提供异步函数会抛出，因为它会产生无限的微任务循环.**

```mjs
import { promiseHooks } from 'node:v8';

const stop = promiseHooks.onInit((promise, parent) => {});
```

```cjs
const { promiseHooks } = require('node:v8');

const stop = promiseHooks.onInit((promise, parent) => {});
```

### `promiseHooks.onSettled(settled)`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

* `settled` {Function} The [`settled` callback][] to call when a promise
  is resolved or rejected.
* Returns: {Function} Call to stop the hook.

**`settle` 钩子必须是一个普通函数。提供异步函数会抛出，因为它会产生无限的微任务循环.**

```mjs
import { promiseHooks } from 'node:v8';

const stop = promiseHooks.onSettled((promise) => {});
```

```cjs
const { promiseHooks } = require('node:v8');

const stop = promiseHooks.onSettled((promise) => {});
```

### `promiseHooks.onBefore(before)`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

* `before` {Function} The [`before` callback][] to call before a promise
  continuation executes.
* Returns: {Function} Call to stop the hook.

**`before` 钩子必须是一个普通函数。提供异步函数会抛出，因为它会产生无限的微任务循环.**

```mjs
import { promiseHooks } from 'node:v8';

const stop = promiseHooks.onBefore((promise) => {});
```

```cjs
const { promiseHooks } = require('node:v8');

const stop = promiseHooks.onBefore((promise) => {});
```

### `promiseHooks.onAfter(after)`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

* `after` {Function} The [`after` callback][] to call after a promise
  continuation executes.
* Returns: {Function} Call to stop the hook.

**`after` 钩子必须是一个普通函数。提供异步函数会抛出，因为它会产生无限的微任务循环.**

```mjs
import { promiseHooks } from 'node:v8';

const stop = promiseHooks.onAfter((promise) => {});
```

```cjs
const { promiseHooks } = require('node:v8');

const stop = promiseHooks.onAfter((promise) => {});
```

### `promiseHooks.createHook(callbacks)`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

* `callbacks` {Object} The [Hook Callbacks][] to register
  * `init` {Function} The [`init` callback][].
  * `before` {Function} The [`before` callback][].
  * `after` {Function} The [`after` callback][].
  * `settled` {Function} The [`settled` callback][].
* Returns: {Function} Used for disabling hooks

**挂钩回调必须是普通函数。提供异步函数会抛出，因为它会产生无限的微任务循环.**

注册要为每个 Promise 的不同生命周期事件调用的函数.

回调 `init()`/`before()`/`after()`/`settled()` 在 Promise 的生命周期内为各个事件调用.

所有回调都是可选的。例如，如果只需要跟踪 Promise 的创建，那么只需要传递 `init` 回调。可以传递给 `callbacks` 的所有函数的细节在 [Hook Callbacks][] 部分.

```mjs
import { promiseHooks } from 'node:v8';

const stopAll = promiseHooks.createHook({
  init(promise, parent) {}
});
```

```cjs
const { promiseHooks } = require('node:v8');

const stopAll = promiseHooks.createHook({
  init(promise, parent) {}
});
```

### Hook callbacks

Key events in the lifetime of a promise have been categorized into four areas:
在调用延续处理程序之前/之后或在 await 前后，以及当 Promise 解决或拒绝时创建 Promise.

虽然这些钩子与 [`async_hooks`][] 的钩子相似，但它们缺少 `destroy` 钩子。其他类型的异步资源通常表示套接字或文件描述符，它们具有不同的“关闭”状态来表示“销毁”生命周期事件，而只要代码仍然可以访问它们，promise 就仍然可用。垃圾收集跟踪用于使 Promise 适合 `async_hooks` 事件模型，但是这种跟踪非常昂贵，它们甚至不一定会被垃圾收集.

因为 Promise 是异步资源，其生命周期通过 Promise 钩子机制进行跟踪，所以 `init()`、`before()`、`after()` 和 `settled()` 回调_不能_是异步函数，因为它们会创建更多会产生无限循环的承诺.

虽然此 API 用于将 Promise 事件提供给 [`async_hooks`][]，但两者之间的顺序是未定义的。两个 API 都是多租户的，因此可以以相对于彼此的任何顺序产生事件.

#### `init(promise, parent)`

* `promise` {Promise} The promise being created.
* `parent` {Promise} The promise continued from, if applicable.

在构造承诺时调用。这_并不_意味着相应的`before`/`after`事件会发生，只是这种可能性存在。如果在没有获得延续的情况下创建了一个承诺，就会发生这种情况.

#### `before(promise)`

* `promise` {Promise}

在 Promise continuation 执行之前调用。这可以是 `then()`、`catch()` 或 `finally()` 处理程序或 `await` 恢复的形式.

`before` 回调将被调用 0 到 N 次。如果承诺没有继续进行，那么 `before` 回调通常会被调用 0 次。 `before` 回调可能会在同一个 Promise 进行了许多 continuation 的情况下被多次调用.

#### `after(promise)`

* `promise` {Promise}

在 Promise continuation 执行后立即调用。这可能在 `then()`、`catch()` 或 `finally()` 处理程序之后或在另一个 `await` 之后的 `await` 之前.

#### `settled(promise)`

* `promise` {Promise}

当 Promise 收到解决或拒绝值时调用。这可能在 `Promise.resolve()` 或 `Promise.reject()` 的情况下同步发生.

## Startup Snapshot API

<!-- YAML
added: v18.6.0
-->

> Stability: 1 - Experimental

`v8.startupSnapshot` 接口可用于为自定义启动快照添加序列化和反序列化挂钩。目前启动快照只能从源代码构建到 Node.js 二进制文件中.

```console
$ cd /path/to/node
$ ./configure --node-snapshot-main=entry.js
$ make node
# This binary contains the result of the execution of entry.js
$ out/Release/node
```

在上面的示例中，`entry.js` 可以使用来自 `v8.startupSnapshot` 接口的方法来指定如何在序列化过程中将自定义对象的信息保存在快照中，以及如何在反序列化过程中使用这些信息来同步这些对象。快照。例如，如果 `entry.js` 包含以下脚本:

```cjs
'use strict';

const fs = require('fs');
const zlib = require('zlib');
const path = require('path');
const assert = require('assert');

const {
  isBuildingSnapshot,
  addSerializeCallback,
  addDeserializeCallback,
  setDeserializeMainFunction
} = require('v8').startupSnapshot;

const filePath = path.resolve(__dirname, '../x1024.txt');
const storage = {};

assert(isBuildingSnapshot());

addSerializeCallback(({ filePath }) => {
  storage[filePath] = zlib.gzipSync(fs.readFileSync(filePath));
}, { filePath });

addDeserializeCallback(({ filePath }) => {
  storage[filePath] = zlib.gunzipSync(storage[filePath]);
}, { filePath });

setDeserializeMainFunction(({ filePath }) => {
  console.log(storage[filePath].toString());
}, { filePath });
```

生成的二进制文件将在启动期间简单地打印从快照反序列化的数据:

```console
$ out/Release/node
# Prints content of ./test/fixtures/x1024.txt
```

目前该 API 仅适用于从默认快照启动的 Node.js 实例，即从用户端快照反序列化的应用程序无法再次使用这些 API.

### `v8.startupSnapshot.addSerializeCallback(callback[, data])`

<!-- YAML
added: v18.6.0
-->

* `callback` {Function} Callback to be invoked before serialization.
* `data` {any} Optional data that will be passed to the `callback` when it
  gets called.

添加一个回调，当 Node.js 实例即将序列化为快照并退出时将调用该回调。这可以用来释放不应该或不能序列化的资源或将用户数据转换成更适合序列化的形式.

### `v8.startupSnapshot.addDeserializeCallback(callback[, data])`

<!-- YAML
added: v18.6.0
-->

* `callback` {Function} Callback to be invoked after the snapshot is
  deserialized.
* `data` {any} Optional data that will be passed to the `callback` when it
  gets called.

添加从快照反序列化 Node.js 实例时将调用的回调。 `callback` 和 `data`（如果提供）将被序列化到快照中，它们可用于重新初始化应用程序的状态或重新获取应用程序从应用程序重新启动时所需的资源快照.

### `v8.startupSnapshot.setDeserializeMainFunction(callback[, data])`

<!-- YAML
added: v18.6.0
-->

* `callback` {Function} Callback to be invoked as the entry point after the
  snapshot is deserialized.
* `data` {any} Optional data that will be passed to the `callback` when it
  gets called.

这会在从快照反序列化 Node.js 应用程序时设置它的入口点。这只能在快照构建脚本中调用一次。如果被调用，反序列化应用程序不再需要额外的入口点脚本来启动，并且将简单地调用回调以及反序列化数据（如果提供），否则仍然需要向反序列化应用程序提供入口点脚本.

### `v8.startupSnapshot.isBuildingSnapshot()`

<!-- YAML
added: v18.6.0
-->

* Returns: {boolean}

如果运行 Node.js 实例来构建快照，则返回 true.

[HTML structured clone algorithm]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm
[Hook Callbacks]: #hook-callbacks
[V8]: https://developers.google.com/v8/
[`AsyncLocalStorage`]: async_context.md#class-asynclocalstorage
[`Buffer`]: buffer.md
[`DefaultDeserializer`]: #class-v8defaultdeserializer
[`DefaultSerializer`]: #class-v8defaultserializer
[`Deserializer`]: #class-v8deserializer
[`ERR_BUFFER_TOO_LARGE`]: errors.md#err_buffer_too_large
[`Error`]: errors.md#class-error
[`GetHeapCodeAndMetadataStatistics`]: https://v8docs.nodesource.com/node-13.2/d5/dda/classv8_1_1_isolate.html#a6079122af17612ef54ef3348ce170866
[`GetHeapSpaceStatistics`]: https://v8docs.nodesource.com/node-13.2/d5/dda/classv8_1_1_isolate.html#ac673576f24fdc7a33378f8f57e1d13a4
[`NODE_V8_COVERAGE`]: cli.md#node_v8_coveragedir
[`Serializer`]: #class-v8serializer
[`after` callback]: #afterpromise
[`async_hooks`]: async_hooks.md
[`before` callback]: #beforepromise
[`buffer.constants.MAX_LENGTH`]: buffer.md#bufferconstantsmax_length
[`deserializer._readHostObject()`]: #deserializer_readhostobject
[`deserializer.transferArrayBuffer()`]: #deserializertransferarraybufferid-arraybuffer
[`init` callback]: #initpromise-parent
[`serialize()`]: #v8serializevalue
[`serializer._getSharedArrayBufferId()`]: #serializer_getsharedarraybufferidsharedarraybuffer
[`serializer._writeHostObject()`]: #serializer_writehostobjectobject
[`serializer.releaseBuffer()`]: #serializerreleasebuffer
[`serializer.transferArrayBuffer()`]: #serializertransferarraybufferid-arraybuffer
[`serializer.writeRawBytes()`]: #serializerwriterawbytesbuffer
[`settled` callback]: #settledpromise
[`v8.stopCoverage()`]: #v8stopcoverage
[`v8.takeCoverage()`]: #v8takecoverage
[`vm.Script`]: vm.md#new-vmscriptcode-options
[worker threads]: worker_threads.md
