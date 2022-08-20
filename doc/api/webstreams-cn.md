# Web Streams API

<!--introduced_in=v16.5.0-->

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: Use of this API no longer emit a runtime warning.
-->

> 稳定性： 1 - 实验性。

实现[WHATWG Streams Standard][].

## 概述

这[WHATWG Streams Standard][]（或“Web流”）定义了用于处理的API
流数据。它类似于节点.js[流][Streams]API，但后来出现
并已成为跨许多JavaScript流式传输数据的“标准”API
环境。

有三种主要类型的对象：

*   `ReadableStream`- 表示流数据源。
*   `WritableStream`- 表示流数据的目标。
*   `TransformStream`- 表示用于转换流数据的算法。

### 例`ReadableStream`

此示例创建一个简单的`ReadableStream`推动电流
`performance.now()`时间戳每秒一次，永远。异步可迭代
用于从流中读取数据。

```mjs
import {
  ReadableStream
} from 'node:stream/web';

import {
  setInterval as every
} from 'node:timers/promises';

import {
  performance
} from 'node:perf_hooks';

const SECOND = 1000;

const stream = new ReadableStream({
  async start(controller) {
    for await (const _ of every(SECOND))
      controller.enqueue(performance.now());
  }
});

for await (const value of stream)
  console.log(value);
```

```cjs
const {
  ReadableStream
} = require('node:stream/web');

const {
  setInterval: every
} = require('node:timers/promises');

const {
  performance
} = require('node:perf_hooks');

const SECOND = 1000;

const stream = new ReadableStream({
  async start(controller) {
    for await (const _ of every(SECOND))
      controller.enqueue(performance.now());
  }
});

(async () => {
  for await (const value of stream)
    console.log(value);
})();
```

## 应用程序接口

### 类：`ReadableStream`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new ReadableStream([underlyingSource [, strategy]])`

<!-- YAML
added: v16.5.0
-->

<!--lint disable maximum-line-length remark-lint-->

*   `underlyingSource`{对象}
    *   `start`{函数}在以下情况下立即调用的用户定义函数
        这`ReadableStream`已创建。
        *   `controller`{ReadableStreamDefaultController|ReadableByteStreamController}
        *   返回：`undefined`或履行承诺`undefined`.
    *   `pull`{函数}一个用户定义函数，当
        `ReadableStream`内部队列未满。操作可以是同步的，也可以是
        异步。如果异步，则在之前不会再次调用该函数
        返回的承诺已实现。
        *   `controller`{ReadableStreamDefaultController|ReadableByteStreamController}
        *   回报：兑现承诺`undefined`.
    *   `cancel`{函数}一个用户定义函数，当
        `ReadableStream`被取消。
        *   `reason`{任何}
        *   回报：兑现承诺`undefined`.
    *   `type`{字符串}必须是`'bytes'`或`undefined`.
    *   `autoAllocateChunkSize`{数字}仅在以下情况下使用`type`等于
        `'bytes'`.
*   `strategy`{对象}
    *   `highWaterMark`{数字}背压前的最大内部队列大小
        已应用。
    *   `size`{函数}用于标识每个函数大小的用户定义函数
        数据块。
        *   `chunk`{任何}
        *   返回值：{数字}

<!--lint enable maximum-line-length remark-lint-->

#### `readableStream.locked`

<!-- YAML
added: v16.5.0
-->

*   类型：{布尔值} 设置为`true`如果有活跃的读者
    {可读流}.

这`readableStream.locked`属性是`false`默认情况下，并且是
已切换到`true`当有一个活跃的读者消费
流的数据。

#### `readableStream.cancel([reason])`

<!-- YAML
added: v16.5.0
-->

*   `reason`{任何}
*   回报：兑现承诺`undefined`一旦取消
    已完成。

#### `readableStream.getReader([options])`

<!-- YAML
added: v16.5.0
-->

*   `options`{对象}
    *   `mode`{字符串}`'byob'`或`undefined`
*   返回：{ReadableStreamDefaultReader|ReadableStreamBYOBReader}

```mjs
import { ReadableStream } from 'node:stream/web';

const stream = new ReadableStream();

const reader = stream.getReader();

console.log(await reader.read());
```

```cjs
const { ReadableStream } = require('node:stream/web');

const stream = new ReadableStream();

const reader = stream.getReader();

reader.read().then(console.log);
```

导致`readableStream.locked`成为`true`.

#### `readableStream.pipeThrough(transform[, options])`

<!-- YAML
added: v16.5.0
-->

*   `transform`{对象}
    *   `readable`{可读流}这`ReadableStream`到哪个
        `transform.writable`将推送可能修改的数据
        是从这里接收的`ReadableStream`.
    *   `writable`{可写流}这`WritableStream`对此
        `ReadableStream`的数据将被写入。
*   `options`{对象}
    *   `preventAbort`{布尔值}什么时候`true`，则此错误`ReadableStream`
        不会造成`transform.writable`要中止。
    *   `preventCancel`{布尔值}什么时候`true`、目标中的错误
        `transform.writable`不要导致这种情况`ReadableStream`成为
        取消。
    *   `preventClose`{布尔值}什么时候`true`，关闭此`ReadableStream`
        不会导致`transform.writable`关闭。
    *   `signal`{中止信号}允许取消数据传输
        使用 {AbortController}。
*   返回：{可读流} 从`transform.readable`.

将此 {ReadableStream} 连接到 {ReadableStream} 对，然后
{可写流} 在`transform`参数使得
来自此 {ReadableStream} 的数据被写入`transform.writable`,
可能被转换，然后被推到`transform.readable`.一旦
管道已配置，`transform.readable`返回。

导致`readableStream.locked`成为`true`管道操作时
处于活动状态。

```mjs
import {
  ReadableStream,
  TransformStream,
} from 'node:stream/web';

const stream = new ReadableStream({
  start(controller) {
    controller.enqueue('a');
  },
});

const transform = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});

const transformedStream = stream.pipeThrough(transform);

for await (const chunk of transformedStream)
  console.log(chunk);
```

```cjs
const {
  ReadableStream,
  TransformStream,
} = require('node:stream/web');

const stream = new ReadableStream({
  start(controller) {
    controller.enqueue('a');
  },
});

const transform = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});

const transformedStream = stream.pipeThrough(transform);

(async () => {
  for await (const chunk of transformedStream)
    console.log(chunk);
})();
```

#### `readableStream.pipeTo(destination, options)`

<!-- YAML
added: v16.5.0
-->

*   `destination`{可写流}一个 {可写流}
    `ReadableStream`的数据将被写入。
*   `options`{对象}
    *   `preventAbort`{布尔值}什么时候`true`，则此错误`ReadableStream`
        不会造成`destination`要中止。
    *   `preventCancel`{布尔值}什么时候`true`、中的错误`destination`
        不会造成这种情况`ReadableStream`将被取消。
    *   `preventClose`{布尔值}什么时候`true`，关闭此`ReadableStream`
        不会导致`destination`关闭。
    *   `signal`{中止信号}允许取消数据传输
        使用 {AbortController}。
*   回报：兑现承诺`undefined`

导致`readableStream.locked`成为`true`管道操作时
处于活动状态。

#### `readableStream.tee()`

<!-- YAML
added: v16.5.0
-->

*   返回： {可读流\[]}

返回一对新的 {ReadableStream} 实例，这些实例
`ReadableStream`的数据将被转发。每个人都将收到
相同的数据。

导致`readableStream.locked`成为`true`.

#### `readableStream.values([options])`

<!-- YAML
added: v16.5.0
-->

*   `options`{对象}
    *   `preventCancel`{布尔值}什么时候`true`，阻止 {可读流}
        当异步迭代器突然终止时关闭。
        **违约**:`false`.

创建并返回可用于使用此函数的异步迭代器
`ReadableStream`的数据。

导致`readableStream.locked`成为`true`而异步迭代器
处于活动状态。

```mjs
import { Buffer } from 'node:buffer';

const stream = new ReadableStream(getSomeSource());

for await (const chunk of stream.values({ preventCancel: true }))
  console.log(Buffer.from(chunk).toString());
```

#### 异步迭代

{ReadableStream} 对象支持异步迭代器协议，使用
`for await`语法。

```mjs
import { Buffer } from 'node:buffer';

const stream = new ReadableStream(getSomeSource());

for await (const chunk of stream)
  console.log(Buffer.from(chunk).toString());
```

异步迭代器将使用 {ReadableStream}，直到它终止。

默认情况下，如果异步迭代器提前退出（通过`break`,
`return`，或`throw`），则 {ReadableStream} 将关闭。防止
自动关闭 {可读流}，使用`readableStream.values()`
获取异步迭代器并设置`preventCancel`选项
`true`.

{可读流} 不得被锁定（即，它不得具有现有的
活动读取器）。在异步迭代期间，{ReadableStream} 将被锁定。

#### 转移`postMessage()`

{ReadableStream} 实例可以使用 {MessagePort} 进行传输。

```js
const stream = new ReadableStream(getReadableSourceSomehow());

const { port1, port2 } = new MessageChannel();

port1.onmessage = ({ data }) => {
  data.getReader().read().then((chunk) => {
    console.log(chunk);
  });
};

port2.postMessage(stream, [stream]);
```

### 类：`ReadableStreamDefaultReader`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

默认情况下，呼叫`readableStream.getReader()`没有参数
将返回一个实例`ReadableStreamDefaultReader`.默认值
读取器将通过流传递的数据块视为不透明
值，它允许 {ReadableStream} 通常使用任何值
JavaScript 值。

#### `new ReadableStreamDefaultReader(stream)`

<!-- YAML
added: v16.5.0
-->

*   `stream`{可读流}

创建一个新的 {ReadableStreamDefaultReader}，该读取器被锁定到
给定 {ReadableStream}.

#### `readableStreamDefaultReader.cancel([reason])`

<!-- YAML
added: v16.5.0
-->

*   `reason`{任何}
*   回报：兑现承诺`undefined`.

取消 {ReadableStream} 并返回已实现的承诺
当基础流已取消时。

#### `readableStreamDefaultReader.closed`

<!-- YAML
added: v16.5.0
-->

*   类型： {承诺} 履行`undefined`当关联
    {ReadableStream} 被关闭或拒绝，如果流错误或读取器的
    锁定在流完成关闭之前释放。

#### `readableStreamDefaultReader.read()`

<!-- YAML
added: v16.5.0
-->

*   返回：使用对象实现的承诺：
    *   `value`{ArrayBuffer}
    *   `done`{布尔值}

从底层 {ReadableStream} 请求下一个数据块
并返回一个承诺，该承诺在数据实现后即已实现
可用。

#### `readableStreamDefaultReader.releaseLock()`

<!-- YAML
added: v16.5.0
-->

在底层 {ReadableStream} 上释放此读取器锁。

### 类：`ReadableStreamBYOBReader`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

这`ReadableStreamBYOBReader`是的替代消费者
面向字节的 {ReadableStream}s（使用
`underlyingSource.type`设置为等于`'bytes'`当
`ReadableStream`已创建）。

这`BYOB`是“自带缓冲区”的缩写。这是一个
允许更有效地读取面向字节的模式
避免无关复制的数据。

```mjs
import {
  open
} from 'node:fs/promises';

import {
  ReadableStream
} from 'node:stream/web';

import { Buffer } from 'node:buffer';

class Source {
  type = 'bytes';
  autoAllocateChunkSize = 1024;

  async start(controller) {
    this.file = await open(new URL(import.meta.url));
    this.controller = controller;
  }

  async pull(controller) {
    const view = controller.byobRequest?.view;
    const {
      bytesRead,
    } = await this.file.read({
      buffer: view,
      offset: view.byteOffset,
      length: view.byteLength
    });

    if (bytesRead === 0) {
      await this.file.close();
      this.controller.close();
    }
    controller.byobRequest.respond(bytesRead);
  }
}

const stream = new ReadableStream(new Source());

async function read(stream) {
  const reader = stream.getReader({ mode: 'byob' });

  const chunks = [];
  let result;
  do {
    result = await reader.read(Buffer.alloc(100));
    if (result.value !== undefined)
      chunks.push(Buffer.from(result.value));
  } while (!result.done);

  return Buffer.concat(chunks);
}

const data = await read(stream);
console.log(Buffer.from(data).toString());
```

#### `new ReadableStreamBYOBReader(stream)`

<!-- YAML
added: v16.5.0
-->

*   `stream`{可读流}

创建新的`ReadableStreamBYOBReader`被锁定到
给定 {ReadableStream}.

#### `readableStreamBYOBReader.cancel([reason])`

<!-- YAML
added: v16.5.0
-->

*   `reason`{任何}
*   回报：兑现承诺`undefined`.

取消 {ReadableStream} 并返回已实现的承诺
当基础流已取消时。

#### `readableStreamBYOBReader.closed`

<!-- YAML
added: v16.5.0
-->

*   类型： {承诺} 履行`undefined`当关联
    {ReadableStream} 被关闭或拒绝，如果流错误或读取器的
    锁定在流完成关闭之前释放。

#### `readableStreamBYOBReader.read(view)`

<!-- YAML
added: v16.5.0
-->

*   `view`{缓冲区|TypedArray|数据视图}
*   返回：使用对象实现的承诺：
    *   `value`{ArrayBuffer}
    *   `done`{布尔值}

从底层 {ReadableStream} 请求下一个数据块
并返回一个承诺，该承诺在数据实现后即已实现
可用。

不要将池化 {Buffer} 对象实例传递给此方法。
汇集`Buffer`对象是使用`Buffer.allocUnsafe()`,
或`Buffer.from()`，或经常被各种返回`node:fs`模块
回调。这些类型的`Buffer`使用共享的基础
{ArrayBuffer} 对象，其中包含来自
池化`Buffer`实例。当`Buffer`， {TypedArray}，
或 {DataView} 传入到`readableStreamBYOBReader.read()`,
视图的基础`ArrayBuffer`是*超然*，使
可能存在的所有现有视图`ArrayBuffer`.这
可能会对您的应用程序产生灾难性的后果。

#### `readableStreamBYOBReader.releaseLock()`

<!-- YAML
added: v16.5.0
-->

在底层 {ReadableStream} 上释放此读取器锁。

### 类：`ReadableStreamDefaultController`

<!-- YAML
added: v16.5.0
-->

每个 {ReadableStream} 都有一个控制器，负责
流队列的内部状态和管理。这
`ReadableStreamDefaultController`是默认控制器
的实现`ReadableStream`不面向字节的。

#### `readableStreamDefaultController.close()`

<!-- YAML
added: v16.5.0
-->

关闭与此控制器关联的 {ReadableStream}。

#### `readableStreamDefaultController.desiredSize`

<!-- YAML
added: v16.5.0
-->

*   类型： {数字}

返回要填充 {ReadableStream} 的剩余数据量
队列。

#### `readableStreamDefaultController.enqueue(chunk)`

<!-- YAML
added: v16.5.0
-->

*   `chunk`{任何}

将新的数据块追加到 {ReadableStream} 的队列中。

#### `readableStreamDefaultController.error(error)`

<!-- YAML
added: v16.5.0
-->

*   `error`{任何}

发出错误信号，导致 {ReadableStream} 出错并关闭。

### 类：`ReadableByteStreamController`

<!-- YAML
added: v16.5.0
-->

每个 {ReadableStream} 都有一个控制器，负责
流队列的内部状态和管理。这
`ReadableByteStreamController`用于面向字节`ReadableStream`s.

#### `readableByteStreamController.byobRequest`

<!-- YAML
added: v16.5.0
-->

*   Type： {ReadableStreamBYOBRequest}

#### `readableByteStreamController.close()`

<!-- YAML
added: v16.5.0
-->

关闭与此控制器关联的 {ReadableStream}。

#### `readableByteStreamController.desiredSize`

<!-- YAML
added: v16.5.0
-->

*   类型： {数字}

返回要填充 {ReadableStream} 的剩余数据量
队列。

#### `readableByteStreamController.enqueue(chunk)`

<!-- YAML
added: v16.5.0
-->

*   `chunk`： {缓冲区|TypedArray|数据视图}

将新的数据块追加到 {ReadableStream} 的队列中。

#### `readableByteStreamController.error(error)`

<!-- YAML
added: v16.5.0
-->

*   `error`{任何}

发出错误信号，导致 {ReadableStream} 出错并关闭。

### 类：`ReadableStreamBYOBRequest`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

使用时`ReadableByteStreamController`以面向字节
流，以及使用`ReadableStreamBYOBReader`,
这`readableByteStreamController.byobRequest`财产
提供对`ReadableStreamBYOBRequest`实例
表示当前读取请求。对象
用于访问`ArrayBuffer`/`TypedArray`
已提供供读取请求填写，
并提供用于向数据发出信号的方法
已提供。

#### `readableStreamBYOBRequest.respond(bytesWritten)`

<!-- YAML
added: v16.5.0
-->

*   `bytesWritten`{数字}

表明`bytesWritten`已写入的字节数
自`readableStreamBYOBRequest.view`.

#### `readableStreamBYOBRequest.respondWithNewView(view)`

<!-- YAML
added: v16.5.0
-->

*   `view`{缓冲区|TypedArray|数据视图}

请求已通过写入字节完成
到一个新的`Buffer`,`TypedArray`或`DataView`.

#### `readableStreamBYOBRequest.view`

<!-- YAML
added: v16.5.0
-->

*   类型： {缓冲区|TypedArray|数据视图}

### 类：`WritableStream`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

这`WritableStream`是流数据发送到的目标。

```mjs
import {
  WritableStream
} from 'node:stream/web';

const stream = new WritableStream({
  write(chunk) {
    console.log(chunk);
  }
});

await stream.getWriter().write('Hello World');
```

#### `new WritableStream([underlyingSink[, strategy]])`

<!-- YAML
added: v16.5.0
-->

*   `underlyingSink`{对象}
    *   `start`{函数}在以下情况下立即调用的用户定义函数
        这`WritableStream`已创建。
        *   `controller`{WritableStreamDefaultController}
        *   返回：`undefined`或履行承诺`undefined`.
    *   `write`{函数}一个用户定义函数，当
        数据已写入`WritableStream`.
        *   `chunk`{任何}
        *   `controller`{WritableStreamDefaultController}
        *   回报：兑现承诺`undefined`.
    *   `close`{函数}一个用户定义函数，当
        `WritableStream`已关闭。
        *   回报：兑现承诺`undefined`.
    *   `abort`{函数}调用以突然关闭的用户定义函数
        这`WritableStream`.
        *   `reason`{任何}
        *   回报：兑现承诺`undefined`.
    *   `type`{任何}这`type`选项保留供将来使用，并且*必须*是
        定义。
*   `strategy`{对象}
    *   `highWaterMark`{数字}背压前的最大内部队列大小
        已应用。
    *   `size`{函数}用于标识每个函数大小的用户定义函数
        数据块。
        *   `chunk`{任何}
        *   返回值：{数字}

#### `writableStream.abort([reason])`

<!-- YAML
added: v16.5.0
-->

*   `reason`{任何}
*   回报：兑现承诺`undefined`.

突然终止`WritableStream`.所有排队的写入将为
取消，相关承诺被拒绝。

#### `writableStream.close()`

<!-- YAML
added: v16.5.0
-->

*   回报：兑现承诺`undefined`.

关闭`WritableStream`当不需要其他写入时。

#### `writableStream.getWriter()`

<!-- YAML
added: v16.5.0
-->

*   返回： {WriteableStreamDefaultWriter}

创建并创建可用于写入的新编写器实例
数据进入`WritableStream`.

#### `writableStream.locked`

<!-- YAML
added: v16.5.0
-->

*   类型： {布尔值}

这`writableStream.locked`属性是`false`默认情况下，并且是
已切换到`true`虽然有一个活跃的作家附加到这个
`WritableStream`.

#### 使用 postMessage（） 传输

可以使用 {MessagePort} 传输 {WritableStream} 实例。

```js
const stream = new WritableStream(getWritableSinkSomehow());

const { port1, port2 } = new MessageChannel();

port1.onmessage = ({ data }) => {
  data.getWriter().write('hello');
};

port2.postMessage(stream, [stream]);
```

### 类：`WritableStreamDefaultWriter`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new WritableStreamDefaultWriter(stream)`

<!-- YAML
added: v16.5.0
-->

*   `stream`{可写流}

创建新的`WritableStreamDefaultWriter`被锁定到给定
`WritableStream`.

#### `writableStreamDefaultWriter.abort([reason])`

<!-- YAML
added: v16.5.0
-->

*   `reason`{任何}
*   回报：兑现承诺`undefined`.

突然终止`WritableStream`.所有排队的写入将为
取消，相关承诺被拒绝。

#### `writableStreamDefaultWriter.close()`

<!-- YAML
added: v16.5.0
-->

*   回报：兑现承诺`undefined`.

关闭`WritableStream`当不需要其他写入时。

#### `writableStreamDefaultWriter.closed`

<!-- YAML
added: v16.5.0
-->

*   类型： {承诺} 履行`undefined`当关联
    {WritableStream} 被关闭或拒绝，如果流错误或编写器的
    锁定在流完成关闭之前释放。

#### `writableStreamDefaultWriter.desiredSize`

<!-- YAML
added: v16.5.0
-->

*   类型： {数字}

填充 {WritableStream} 的队列所需的数据量。

#### `writableStreamDefaultWriter.ready`

<!-- YAML
added: v16.5.0
-->

*   类型：通过`undefined`当
    编写器已准备好使用。

#### `writableStreamDefaultWriter.releaseLock()`

<!-- YAML
added: v16.5.0
-->

释放此编写器对底层 {ReadableStream} 的锁定。

#### `writableStreamDefaultWriter.write([chunk])`

<!-- YAML
added: v16.5.0
-->

*   `chunk`： {任意}
*   回报：兑现承诺`undefined`.

将新的数据块追加到 {WritableStream} 的队列中。

### 类：`WritableStreamDefaultController`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

这`WritableStreamDefaultController`管理是 {WritableStream} 的
内部状态。

#### `writableStreamDefaultController.abortReason`

*   类型： {任意}`reason`传递给的值`writableStream.abort()`.

#### `writableStreamDefaultController.error(error)`

<!-- YAML
added: v16.5.0
-->

*   `error`{任何}

由用户代码调用，以指示处理过程中发生了错误
这`WritableStream`数据。调用时，{WritableStream} 将被中止，
取消当前挂起的写入。

#### `writableStreamDefaultController.signal`

*   类型： {中止信号} an`AbortSignal`可用于取消挂起
    在 {WritableStream} 中止时写入或关闭操作。

### 类：`TransformStream`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

一个`TransformStream`由一个 {ReadableStream} 和一个 {WritableStream} 组成，它们
连接起来，使得数据写入`WritableStream`已收到，
并可能被转化，然后被推入`ReadableStream`的
队列。

```mjs
import {
  TransformStream
} from 'node:stream/web';

const transform = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});

await Promise.all([
  transform.writable.getWriter().write('A'),
  transform.readable.getReader().read(),
]);
```

#### `new TransformStream([transformer[, writableStrategy[, readableStrategy]]])`

<!-- YAML
added: v16.5.0
-->

*   `transformer`{对象}
    *   `start`{函数}在以下情况下立即调用的用户定义函数
        这`TransformStream`已创建。
        *   `controller`{TransformStreamDefaultController}
        *   返回：`undefined`或履行承诺`undefined`
    *   `transform`{函数}一个用户定义的函数，它接收和
        可能修改，写入`transformStream.writable`,
        在将其转发到`transformStream.readable`.
        *   `chunk`{任何}
        *   `controller`{TransformStreamDefaultController}
        *   回报：兑现承诺`undefined`.
    *   `flush`{函数}紧挨着前面调用的用户定义函数
        可写侧的`TransformStream`已关闭，表示结束
        转换过程。
        *   `controller`{TransformStreamDefaultController}
        *   回报：兑现承诺`undefined`.
    *   `readableType`{任何}`readableType`保留选项供将来使用
        和*必须*是`undefined`.
    *   `writableType`{任何}`writableType`保留选项供将来使用
        和*必须*是`undefined`.
*   `writableStrategy`{对象}
    *   `highWaterMark`{数字}背压前的最大内部队列大小
        已应用。
    *   `size`{函数}用于标识每个函数大小的用户定义函数
        数据块。
        *   `chunk`{任何}
        *   返回值：{数字}
*   `readableStrategy`{对象}
    *   `highWaterMark`{数字}背压前的最大内部队列大小
        已应用。
    *   `size`{函数}用于标识每个函数大小的用户定义函数
        数据块。
        *   `chunk`{任何}
        *   返回值：{数字}

#### `transformStream.readable`

<!-- YAML
added: v16.5.0
-->

*   类型： {可读流}

#### `transformStream.writable`

<!-- YAML
added: v16.5.0
-->

*   类型： {可写流}

#### 使用 postMessage（） 传输

可以使用 {MessagePort} 传输 {TransformStream} 实例。

```js
const stream = new TransformStream();

const { port1, port2 } = new MessageChannel();

port1.onmessage = ({ data }) => {
  const { writable, readable } = data;
  // ...
};

port2.postMessage(stream, [stream]);
```

### 类：`TransformStreamDefaultController`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

这`TransformStreamDefaultController`管理内部状态
的`TransformStream`.

#### `transformStreamDefaultController.desiredSize`

<!-- YAML
added: v16.5.0
-->

*   类型： {数字}

填充可读端队列所需的数据量。

#### `transformStreamDefaultController.enqueue([chunk])`

<!-- YAML
added: v16.5.0
-->

*   `chunk`{任何}

将数据块追加到可读端的队列中。

#### `transformStreamDefaultController.error([reason])`

<!-- YAML
added: v16.5.0
-->

*   `reason`{任何}

向可读和可写侧发出发生错误的信号
在处理转换数据时，导致双方突然
闭。

#### `transformStreamDefaultController.terminate()`

<!-- YAML
added: v16.5.0
-->

关闭传输的可读侧并导致可写侧
突然因错误而关闭。

### 类：`ByteLengthQueuingStrategy`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new ByteLengthQueuingStrategy(options)`

<!-- YAML
added: v16.5.0
-->

*   `options`{对象}
    *   `highWaterMark`{数字}

#### `byteLengthQueuingStrategy.highWaterMark`

<!-- YAML
added: v16.5.0
-->

*   类型： {数字}

#### `byteLengthQueuingStrategy.size`

<!-- YAML
added: v16.5.0
-->

*   类型： {函数}
    *   `chunk`{任何}
    *   返回值：{数字}

### 类：`CountQueuingStrategy`

<!-- YAML
added: v16.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new CountQueuingStrategy(options)`

<!-- YAML
added: v16.5.0
-->

*   `options`{对象}
    *   `highWaterMark`{数字}

#### `countQueuingStrategy.highWaterMark`

<!-- YAML
added: v16.5.0
-->

*   类型： {数字}

#### `countQueuingStrategy.size`

<!-- YAML
added: v16.5.0
-->

*   类型： {函数}
    *   `chunk`{任何}
    *   返回值：{数字}

### 类：`TextEncoderStream`

<!-- YAML
added: v16.6.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new TextEncoderStream()`

<!-- YAML
added: v16.6.0
-->

创建新的`TextEncoderStream`实例。

#### `textEncoderStream.encoding`

<!-- YAML
added: v16.6.0
-->

*   类型： {字符串}

支持的编码`TextEncoderStream`实例。

#### `textEncoderStream.readable`

<!-- YAML
added: v16.6.0
-->

*   类型： {可读流}

#### `textEncoderStream.writable`

<!-- YAML
added: v16.6.0
-->

*   类型： {可写流}

### 类：`TextDecoderStream`

<!-- YAML
added: v16.6.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new TextDecoderStream([encoding[, options]])`

<!-- YAML
added: v16.6.0
-->

*   `encoding`{字符串}标识`encoding`那个`TextDecoder`实例
    支持。**违约：** `'utf-8'`.
*   `options`{对象}
    *   `fatal`{布尔值}`true`如果解码失败是致命的。
    *   `ignoreBOM`{布尔值}什么时候`true`这`TextDecoderStream`将包括
        解码结果中的字节顺序标记。什么时候`false`，字节顺序标记
        将从输出中删除。此选项仅在以下情况下使用：`encoding`是
        `'utf-8'`,`'utf-16be'`或`'utf-16le'`.**违约：** `false`.

创建新的`TextDecoderStream`实例。

#### `textDecoderStream.encoding`

<!-- YAML
added: v16.6.0
-->

*   类型： {字符串}

支持的编码`TextDecoderStream`实例。

#### `textDecoderStream.fatal`

<!-- YAML
added: v16.6.0
-->

*   类型： {布尔值}

该值将为`true`如果解码错误导致`TypeError`存在
扔。

#### `textDecoderStream.ignoreBOM`

<!-- YAML
added: v16.6.0
-->

*   类型： {布尔值}

该值将为`true`如果解码结果将包括字节顺序
马克。

#### `textDecoderStream.readable`

<!-- YAML
added: v16.6.0
-->

*   类型： {可读流}

#### `textDecoderStream.writable`

<!-- YAML
added: v16.6.0
-->

*   类型： {可写流}

### 类：`CompressionStream`

<!-- YAML
added: v17.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new CompressionStream(format)`

<!-- YAML
added: v17.0.0
-->

*   `format`{字符串}其中之一`'deflate'`或`'gzip'`.

#### `compressionStream.readable`

<!-- YAML
added: v17.0.0
-->

*   类型： {可读流}

#### `compressionStream.writable`

<!-- YAML
added: v17.0.0
-->

*   类型： {可写流}

### 类：`DecompressionStream`

<!-- YAML
added: v17.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42225
    description: This class is now exposed on the global object.
-->

#### `new DecompressionStream(format)`

<!-- YAML
added: v17.0.0
-->

*   `format`{字符串}其中之一`'deflate'`或`'gzip'`.

#### `decompressionStream.readable`

<!-- YAML
added: v17.0.0
-->

*   类型： {可读流}

#### `decompressionStream.writable`

<!-- YAML
added: v17.0.0
-->

*   类型： {可写流}

### 公用事业消费者

<!-- YAML
added: v16.7.0
-->

实用程序使用函数提供通用使用选项
流。

它们通过以下方式进行访问：

```mjs
import {
  arrayBuffer,
  blob,
  buffer,
  json,
  text,
} from 'node:stream/consumers';
```

```cjs
const {
  arrayBuffer,
  blob,
  buffer,
  json,
  text,
} = require('node:stream/consumers');
```

#### `streamConsumers.arrayBuffer(stream)`

<!-- YAML
added: v16.7.0
-->

*   `stream`{可读流|流。可读|AsyncIterator}
*   返回：{承诺} 通过`ArrayBuffer`包含完整的
    流的内容。

#### `streamConsumers.blob(stream)`

<!-- YAML
added: v16.7.0
-->

*   `stream`{可读流|流。可读|AsyncIterator}
*   返回：{Promise} 使用包含完整内容的 {Blob} 实现
    的流。

#### `streamConsumers.buffer(stream)`

<!-- YAML
added: v16.7.0
-->

*   `stream`{可读流|流。可读|AsyncIterator}
*   返回：{Promise} 使用包含完整内容的 {Buffer} 实现
    流的内容。

#### `streamConsumers.json(stream)`

<!-- YAML
added: v16.7.0
-->

*   `stream`{可读流|流。可读|AsyncIterator}
*   返回：{Promise} 使用解析为
    UTF-8 编码的字符串，然后通过`JSON.parse()`.

#### `streamConsumers.text(stream)`

<!-- YAML
added: v16.7.0
-->

*   `stream`{可读流|流。可读|AsyncIterator}
*   返回：{Promise} 使用解析为
    UTF-8 编码字符串。

[Streams]: stream.md

[WHATWG Streams Standard]: https://streams.spec.whatwg.org/
