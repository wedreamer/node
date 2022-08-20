# 全局对象

<!--introduced_in=v0.10.0-->

<!-- type=misc -->

这些对象在所有模块中都可用。可能会出现以下变量
全球化，但不是。它们仅存在于模块的范围内，请参阅
[模块系统文档][module system documentation]:

*   [`__dirname`][__dirname]
*   [`__filename`][__filename]
*   [`exports`][exports]
*   [`module`][module]
*   [`require()`][require()]

此处列出的对象特定于 Node.js。有[内置对象][built-in objects]
它们是JavaScript语言本身的一部分，这些语言也是全球性的。
访问。

## 类：`AbortController`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
changes:
  - version: v15.4.0
    pr-url: https://github.com/nodejs/node/pull/35949
    description: No longer experimental.
-->

<!-- type=global -->

用于在选定中发出取消信号的实用程序类`Promise`基于 API。
该 API 基于 Web API[`AbortController`][AbortController].

```js
const ac = new AbortController();

ac.signal.addEventListener('abort', () => console.log('Aborted!'),
                           { once: true });

ac.abort();

console.log(ac.signal.aborted);  // Prints True
```

### `abortController.abort([reason])`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
changes:
  - version:
      - v17.2.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/40807
    description: Added the new optional reason argument.
-->

*   `reason`{任何}可选原因，可在`AbortSignal`的
    `reason`财产。

触发中止信号，导致`abortController.signal`发出
这`'abort'`事件。

### `abortController.signal`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
-->

*   类型： {中止信号}

### 类：`AbortSignal`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
-->

*   扩展：{事件目标}

这`AbortSignal`用于在
`abortController.abort()`调用方法。

#### 静态方法：`AbortSignal.abort([reason])`

<!-- YAML
added:
  - v15.12.0
  - v14.17.0
changes:
  - version:
      - v17.2.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/40807
    description: Added the new optional reason argument.
-->

*   `reason`： {任意}
*   返回： {中止信号}

返回新的已中止`AbortSignal`.

#### 静态方法：`AbortSignal.timeout(delay)`

<!-- YAML
added:
  - v17.3.0
  - v16.14.0
-->

*   `delay`{数字}触发前要等待的毫秒数
    中止信号。

返回新的`AbortSignal`这将在 中止`delay`毫秒。

#### 事件：`'abort'`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
-->

这`'abort'`事件在`abortController.abort()`方法
被调用。回调由单个对象参数调用，并带有
单`type`属性设置为`'abort'`:

```js
const ac = new AbortController();

// Use either the onabort property...
ac.signal.onabort = () => console.log('aborted!');

// Or the EventTarget API...
ac.signal.addEventListener('abort', (event) => {
  console.log(event.type);  // Prints 'abort'
}, { once: true });

ac.abort();
```

这`AbortController`与哪个`AbortSignal`仅关联
曾经触发`'abort'`事件一次。我们建议进行代码检查
该`abortSignal.aborted`属性为`false`在添加之前`'abort'`
事件侦听器。

附加到 的任何事件侦听器`AbortSignal`应使用
`{ once: true }`选项（或者，如果使用`EventEmitter`要附加的 API
侦听器，请使用`once()`方法）以确保事件侦听器是
一旦`'abort'`事件已处理。如果不这样做，可能会
导致内存泄漏。

#### `abortSignal.aborted`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
-->

*   类型：{布尔值} True 在`AbortController`已中止。

#### `abortSignal.onabort`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
-->

*   类型： {函数}

可由用户代码设置为接收通知的可选回调函数
当`abortController.abort()`函数已被调用。

#### `abortSignal.reason`

<!-- YAML
added:
  - v17.2.0
  - v16.14.0
-->

*   类型： {任意}

当`AbortSignal`已触发。

```js
const ac = new AbortController();
ac.abort(new Error('boom!'));
console.log(ac.signal.reason);  // Error('boom!');
```

#### `abortSignal.throwIfAborted()`

<!-- YAML
added: v17.3.0
-->

如果`abortSignal.aborted`是`true`抛出`abortSignal.reason`.

## 类：`Blob`

<!-- YAML
added: v18.0.0
-->

<!-- type=global -->

请参阅 {Blob}。

## 类：`Buffer`

<!-- YAML
added: v0.1.103
-->

<!-- type=global -->

*   {函数}

用于处理二进制数据。查看[缓冲段][buffer section].

## 类：`ByteLengthQueuingStrategy`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ByteLengthQueuingStrategy`][ByteLengthQueuingStrategy].

## `__dirname`

此变量可能显示为全局变量，但不是全局变量。看[`__dirname`][__dirname].

## `__filename`

此变量可能显示为全局变量，但不是全局变量。看[`__filename`][__filename].

## `atob(data)`

<!-- YAML
added: v16.0.0
-->

> 稳定性： 3 - 旧版。用`Buffer.from(data, 'base64')`相反。

的全局别名[`buffer.atob()`][buffer.atob()].

## `BroadcastChannel`

<!-- YAML
added: v18.0.0
-->

请参阅{广播频道}。

## `btoa(data)`

<!-- YAML
added: v16.0.0
-->

> 稳定性： 3 - 旧版。用`buf.toString('base64')`相反。

的全局别名[`buffer.btoa()`][buffer.btoa()].

## `clearImmediate(immediateObject)`

<!-- YAML
added: v0.9.1
-->

<!--type=global-->

[`clearImmediate`][clearImmediate]在[定时器][timers]部分。

## `clearInterval(intervalObject)`

<!-- YAML
added: v0.0.1
-->

<!--type=global-->

[`clearInterval`][clearInterval]在[定时器][timers]部分。

## `clearTimeout(timeoutObject)`

<!-- YAML
added: v0.0.1
-->

<!--type=global-->

[`clearTimeout`][clearTimeout]在[定时器][timers]部分。

## 类：`CompressionStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`CompressionStream`][CompressionStream].

## `console`

<!-- YAML
added: v0.1.100
-->

<!-- type=global -->

*   {对象}

用于打印到标准和标准。查看[`console`][console]部分。

## 类：`CountQueuingStrategy`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`CountQueuingStrategy`][CountQueuingStrategy].

## `Crypto`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用
> [`--experimental-global-webcrypto`][--experimental-global-webcrypto]CLI 标志。

{Crypto} 的浏览器兼容实现。此全局可用
仅当编译了 Node.js 二进制文件，并包含对
`node:crypto`模块。

## `crypto`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用
> [`--experimental-global-webcrypto`][--experimental-global-webcrypto]CLI 标志。

与浏览器兼容的实现[Web Crypto API][].

## `CryptoKey`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用
> [`--experimental-global-webcrypto`][--experimental-global-webcrypto]CLI 标志。

{CryptoKey} 的浏览器兼容实现。此全局可用
仅当编译了 Node.js 二进制文件，并包含对
`node:crypto`模块。

## `CustomEvent`

<!-- YAML
added: v18.7.0
-->

> 稳定性： 1 - 实验性。使用
> [`--experimental-global-customevent`][--experimental-global-customevent]CLI 标志。

<!-- type=global -->

与浏览器兼容的实现[`CustomEvent`网络接口][CustomEvent Web API].

## 类：`DecompressionStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`DecompressionStream`][DecompressionStream].

## `Event`

<!-- YAML
added: v15.0.0
changes:
  - version: v15.4.0
    pr-url: https://github.com/nodejs/node/pull/35949
    description: No longer experimental.
-->

<!-- type=global -->

与浏览器兼容的实现`Event`类。看
[`EventTarget`和`Event`应用程序接口][EventTarget and Event API]了解更多详情。

## `EventTarget`

<!-- YAML
added: v15.0.0
changes:
  - version: v15.4.0
    pr-url: https://github.com/nodejs/node/pull/35949
    description: No longer experimental.
-->

<!-- type=global -->

与浏览器兼容的实现`EventTarget`类。看
[`EventTarget`和`Event`应用程序接口][EventTarget and Event API]了解更多详情。

## `exports`

此变量可能显示为全局变量，但不是全局变量。看[`exports`][exports].

## `fetch`

<!-- YAML
added:
  - v17.5.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用[`--no-experimental-fetch`][--no-experimental-fetch]
> CLI 标志。

与浏览器兼容的实现[`fetch()`][fetch()]功能。

## 类`FormData`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用[`--no-experimental-fetch`][--no-experimental-fetch]
> CLI 标志。

{FormData} 的浏览器兼容实现。

## `global`

<!-- YAML
added: v0.1.27
-->

<!-- type=global -->

*   {对象}全局命名空间对象。

在浏览器中，顶级作用域是全局作用域。这意味着
在浏览器中`var something`将定义一个新的全局变量。在
节点.js这是不同的。顶级范围不是全局范围;
`var something`在 Node 中.js模块将是该模块的本地模块。

## 类`Headers`

<!-- YAML
added:
  - v17.5.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用[`--no-experimental-fetch`][--no-experimental-fetch]
> CLI 标志。

{标头} 的浏览器兼容实现。

## `MessageChannel`

<!-- YAML
added: v15.0.0
-->

<!-- type=global -->

这`MessageChannel`类。看[`MessageChannel`][MessageChannel]了解更多详情。

## `MessageEvent`

<!-- YAML
added: v15.0.0
-->

<!-- type=global -->

这`MessageEvent`类。看[`MessageEvent`][MessageEvent]了解更多详情。

## `MessagePort`

<!-- YAML
added: v15.0.0
-->

<!-- type=global -->

这`MessagePort`类。看[`MessagePort`][MessagePort]了解更多详情。

## `module`

此变量可能显示为全局变量，但不是全局变量。看[`module`][module].

## `performance`

<!-- YAML
added: v16.0.0
-->

这[`perf_hooks.performance`][perf_hooks.performance]对象。

## `process`

<!-- YAML
added: v0.1.7
-->

<!-- type=global -->

*   {对象}

流程对象。查看[`process`对象][process object]部分。

## `queueMicrotask(callback)`

<!-- YAML
added: v11.0.0
-->

<!-- type=global -->

*   `callback`{函数}要排队的函数。

这`queueMicrotask()`方法将要调用的微任务排队`callback`.如果
`callback`引发异常，[`process`对象][process object] `'uncaughtException'`
将发出事件。

微任务队列由 V8 管理，可以采用类似于
这[`process.nextTick()`][process.nextTick()]队列，由 Node.js 管理。这
`process.nextTick()`队列始终在微任务队列之前处理
在 Node 的每个回合中.js事件循环。

```js
// Here, `queueMicrotask()` is used to ensure the 'load' event is always
// emitted asynchronously, and therefore consistently. Using
// `process.nextTick()` here would result in the 'load' event always emitting
// before any other promise jobs.

DataHandler.prototype.load = async function load(key) {
  const hit = this._cache.get(key);
  if (hit !== undefined) {
    queueMicrotask(() => {
      this.emit('load', hit);
    });
    return;
  }

  const data = await fetchData(key);
  this._cache.set(key, data);
  this.emit('load', data);
};
```

## 类：`ReadableByteStreamController`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ReadableByteStreamController`][ReadableByteStreamController].

## 类：`ReadableStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ReadableStream`][ReadableStream].

## 类：`ReadableStreamBYOBReader`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ReadableStreamBYOBReader`][ReadableStreamBYOBReader].

## 类：`ReadableStreamBYOBRequest`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ReadableStreamBYOBRequest`][ReadableStreamBYOBRequest].

## 类：`ReadableStreamDefaultController`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ReadableStreamDefaultController`][ReadableStreamDefaultController].

## 类：`ReadableStreamDefaultReader`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`ReadableStreamDefaultReader`][ReadableStreamDefaultReader].

## `require()`

此变量可能显示为全局变量，但不是全局变量。看[`require()`][require()].

## `Response`

<!-- YAML
added:
  - v17.5.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用[`--no-experimental-fetch`][--no-experimental-fetch]
> CLI 标志。

{响应} 的浏览器兼容实现。

## `Request`

<!-- YAML
added:
  - v17.5.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用[`--no-experimental-fetch`][--no-experimental-fetch]
> CLI 标志。

{Request} 的浏览器兼容实现。

## `setImmediate(callback[, ...args])`

<!-- YAML
added: v0.9.1
-->

<!-- type=global -->

[`setImmediate`][setImmediate]在[定时器][timers]部分。

## `setInterval(callback, delay[, ...args])`

<!-- YAML
added: v0.0.1
-->

<!-- type=global -->

[`setInterval`][setInterval]在[定时器][timers]部分。

## `setTimeout(callback, delay[, ...args])`

<!-- YAML
added: v0.0.1
-->

<!-- type=global -->

[`setTimeout`][setTimeout]在[定时器][timers]部分。

## `structuredClone(value[, options])`

<!-- YAML
added: v17.0.0
-->

<!-- type=global -->

The WHATWG[`structuredClone`][structuredClone]方法。

## `SubtleCrypto`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

> 稳定性： 1 - 实验性。使用
> [`--experimental-global-webcrypto`][--experimental-global-webcrypto]CLI 标志。

{SubtleCrypto} 的浏览器兼容实现。此全局可用
仅当编译了 Node.js 二进制文件，并包含对
`node:crypto`模块。

## `DOMException`

<!-- YAML
added: v17.0.0
-->

<!-- type=global -->

The WHATWG`DOMException`类。看[`DOMException`][DOMException]了解更多详情。

## `TextDecoder`

<!-- YAML
added: v11.0.0
-->

<!-- type=global -->

The WHATWG`TextDecoder`类。查看[`TextDecoder`][TextDecoder]部分。

## 类：`TextDecoderStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`TextDecoderStream`][TextDecoderStream].

## `TextEncoder`

<!-- YAML
added: v11.0.0
-->

<!-- type=global -->

The WHATWG`TextEncoder`类。查看[`TextEncoder`][TextEncoder]部分。

## 类：`TextEncoderStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`TextEncoderStream`][TextEncoderStream].

## 类：`TransformStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`TransformStream`][TransformStream].

## 类：`TransformStreamDefaultController`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`TransformStreamDefaultController`][TransformStreamDefaultController].

## `URL`

<!-- YAML
added: v10.0.0
-->

<!-- type=global -->

The WHATWG`URL`类。查看[`URL`][URL]部分。

## `URLSearchParams`

<!-- YAML
added: v10.0.0
-->

<!-- type=global -->

The WHATWG`URLSearchParams`类。查看[`URLSearchParams`][URLSearchParams]部分。

## `WebAssembly`

<!-- YAML
added: v8.0.0
-->

<!-- type=global -->

*   {对象}

充当所有 W3C 的命名空间的对象
[WebAssembly][webassembly-org]相关功能。查看
[Mozilla 开发者网络][webassembly-mdn]用于和兼容性。

## 类：`WritableStream`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`WritableStream`][WritableStream].

## 类：`WritableStreamDefaultController`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`WritableStreamDefaultController`][WritableStreamDefaultController].

## 类：`WritableStreamDefaultWriter`

<!-- YAML
added: v18.0.0
-->

> 稳定性： 1 - 实验性。

与浏览器兼容的实现[`WritableStreamDefaultWriter`][WritableStreamDefaultWriter].

[Web Crypto API]: webcrypto.md

[`--experimental-global-customevent`]: cli.md#--experimental-global-customevent

[`--experimental-global-webcrypto`]: cli.md#--experimental-global-webcrypto

[`--no-experimental-fetch`]: cli.md#--no-experimental-fetch

[`AbortController`]: https://developer.mozilla.org/en-US/docs/Web/API/AbortController

[`ByteLengthQueuingStrategy`]: webstreams.md#class-bytelengthqueuingstrategy

[`CompressionStream`]: webstreams.md#class-compressionstream

[`CountQueuingStrategy`]: webstreams.md#class-countqueuingstrategy

[`CustomEvent` Web API]: https://dom.spec.whatwg.org/#customevent

[`DOMException`]: https://developer.mozilla.org/en-US/docs/Web/API/DOMException

[`DecompressionStream`]: webstreams.md#class-decompressionstream

[`EventTarget` and `Event` API]: events.md#eventtarget-and-event-api

[`MessageChannel`]: worker_threads.md#class-messagechannel

[`MessageEvent`]: https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent/MessageEvent

[`MessagePort`]: worker_threads.md#class-messageport

[`ReadableByteStreamController`]: webstreams.md#class-readablebytestreamcontroller

[`ReadableStreamBYOBReader`]: webstreams.md#class-readablestreambyobreader

[`ReadableStreamBYOBRequest`]: webstreams.md#class-readablestreambyobrequest

[`ReadableStreamDefaultController`]: webstreams.md#class-readablestreamdefaultcontroller

[`ReadableStreamDefaultReader`]: webstreams.md#class-readablestreamdefaultreader

[`ReadableStream`]: webstreams.md#class-readablestream

[`TextDecoderStream`]: webstreams.md#class-textdecoderstream

[`TextDecoder`]: util.md#class-utiltextdecoder

[`TextEncoderStream`]: webstreams.md#class-textencoderstream

[`TextEncoder`]: util.md#class-utiltextencoder

[`TransformStreamDefaultController`]: webstreams.md#class-transformstreamdefaultcontroller

[`TransformStream`]: webstreams.md#class-transformstream

[`URLSearchParams`]: url.md#class-urlsearchparams

[`URL`]: url.md#class-url

[`WritableStreamDefaultController`]: webstreams.md#class-writablestreamdefaultcontroller

[`WritableStreamDefaultWriter`]: webstreams.md#class-writablestreamdefaultwriter

[`WritableStream`]: webstreams.md#class-writablestream

[`__dirname`]: modules.md#__dirname

[`__filename`]: modules.md#__filename

[`buffer.atob()`]: buffer.md#bufferatobdata

[`buffer.btoa()`]: buffer.md#bufferbtoadata

[`clearImmediate`]: timers.md#clearimmediateimmediate

[`clearInterval`]: timers.md#clearintervaltimeout

[`clearTimeout`]: timers.md#cleartimeouttimeout

[`console`]: console.md

[`exports`]: modules.md#exports

[`fetch()`]: https://developer.mozilla.org/en-US/docs/Web/API/fetch

[`module`]: modules.md#module

[`perf_hooks.performance`]: perf_hooks.md#perf_hooksperformance

[`process.nextTick()`]: process.md#processnexttickcallback-args

[`process` object]: process.md#process

[`require()`]: modules.md#requireid

[`setImmediate`]: timers.md#setimmediatecallback-args

[`setInterval`]: timers.md#setintervalcallback-delay-args

[`setTimeout`]: timers.md#settimeoutcallback-delay-args

[`structuredClone`]: https://developer.mozilla.org/en-US/docs/Web/API/structuredClone

[buffer section]: buffer.md

[built-in objects]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects

[module system documentation]: modules.md

[timers]: timers.md

[webassembly-mdn]: https://developer.mozilla.org/en-US/docs/WebAssembly

[webassembly-org]: https://webassembly.org
