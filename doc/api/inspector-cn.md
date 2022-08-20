# 检查员

<!--introduced_in=v8.0.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/inspector.js -->

这`node:inspector`模块提供了一个用于与 V8 交互的 API
检查员。

可以使用以下命令访问它：

```js
const inspector = require('node:inspector');
```

## `inspector.close()`

停用检查器。阻止，直到没有活动连接。

此功能在 中不可用[工作线程][worker threads].

## `inspector.console`

*   {对象}用于将消息发送到远程检查器控制台的对象。

```js
require('node:inspector').console.log('a message');
```

检查器控制台没有与 Node 的 API 奇偶校验.js
安慰。

## `inspector.open([port[, host[, wait]]])`

*   `port`{数字}用于侦听检查器连接的端口。自选。
    **违约：**在 CLI 上指定的内容。
*   `host`{字符串}主机以侦听检查器连接。自选。
    **违约：**在 CLI 上指定的内容。
*   `wait`{布尔值}在客户端连接之前阻止。自选。
    **违约：** `false`.

激活主机和端口上的检查器。相当于
`node --inspect=[[host:]port]`，但可以在节点具有
开始。

如果等待是`true`，将阻塞，直到客户端连接到检查端口
并且流控制已传递到调试器客户端。

查看[安全警告][security warning]关于`host`
参数用法。

## `inspector.url()`

*   返回：{字符串|未定义}

返回活动检查器的网址，或`undefined`如果没有。

```console
$ node --inspect -p 'inspector.url()'
Debugger listening on ws://127.0.0.1:9229/166e272e-7a30-4d09-97ce-f1c012b43c34
For help, see: https://nodejs.org/en/docs/inspector
ws://127.0.0.1:9229/166e272e-7a30-4d09-97ce-f1c012b43c34

$ node --inspect=localhost:3000 -p 'inspector.url()'
Debugger listening on ws://localhost:3000/51cf8d0e-3c36-4c59-8efd-54519839e56a
For help, see: https://nodejs.org/en/docs/inspector
ws://localhost:3000/51cf8d0e-3c36-4c59-8efd-54519839e56a

$ node -p 'inspector.url()'
undefined
```

## `inspector.waitForDebugger()`

<!-- YAML
added: v12.7.0
-->

阻止，直到客户端（现有或稍后连接）发送
`Runtime.runIfWaitingForDebugger`命令。

如果没有活动的检查器，将引发异常。

## 类：`inspector.Session`

*   扩展：{事件发射器}

这`inspector.Session`用于将消息分派给 V8 检查器
后端和接收消息响应和通知。

### `new inspector.Session()`

<!-- YAML
added: v8.0.0
-->

创建一个新的实例`inspector.Session`类。检查器会话
需要通过以下方式连接[`session.connect()`][session.connect()]在消息之前
可以调度到检查器后端。

### 事件：`'inspectorNotification'`

<!-- YAML
added: v8.0.0
-->

*   {对象}通知消息对象

在收到来自 V8 检查器的任何通知时发出。

```js
session.on('inspectorNotification', (message) => console.log(message.method));
// Debugger.paused
// Debugger.resumed
```

也可以使用特定方法仅订阅通知：

### 事件：`<inspector-protocol-method>`;

<!-- YAML
added: v8.0.0
-->

*   {对象}通知消息对象

在收到设置了方法字段的检查器通知时发出
到`<inspector-protocol-method>`价值。

以下代码段在[`'Debugger.paused'`]['Debugger.paused']
事件，并在程序每次运行时打印程序挂起的原因
执行被暂停（例如，通过断点）：

```js
session.on('Debugger.paused', ({ params }) => {
  console.log(params.hitBreakpoints);
});
// [ '/the/file/that/has/the/breakpoint.js:11:0' ]
```

### `session.connect()`

<!-- YAML
added: v8.0.0
-->

将会话连接到检查器后端。

### `session.connectToMainThread()`

<!-- YAML
added: v12.11.0
-->

将会话连接到主线程检查器后端。例外情况将
如果未在工作线程上调用此 API，则抛出。

### `session.disconnect()`

<!-- YAML
added: v8.0.0
-->

立即关闭会话。将调用所有挂起的消息回调
出现错误。[`session.connect()`][session.connect()]将需要被调用才能发送
再次发送消息。重新连接的会话将丢失所有检查器状态，例如
已启用的代理或配置的断点。

### `session.post(method[, params][, callback])`

<!-- YAML
added: v8.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `method`{字符串}
*   `params`{对象}
*   `callback`{函数}

将消息发布到检查器后端。`callback`将在以下情况下收到通知
收到响应。`callback`是一个接受两个可选的函数
参数：错误和特定于消息的结果。

```js
session.post('Runtime.evaluate', { expression: '2 + 2' },
             (error, { result }) => console.log(result));
// Output: { type: 'number', value: 4, description: '4' }
```

最新版本的 V8 检查器协议发布在
[Chrome DevTools Protocol Viewer][].

Node.js检查器支持所有已声明的 Chrome DevTools 协议域
由 V8.Chrome DevTools Protocol 域提供了一个用于交互的接口
使用其中一个运行时代理程序检查应用程序状态并侦听
到运行时事件。

## 用法示例

除了调试器之外，还可以通过 DevTools 获得各种 V8 Profilers。
协议。

### CPU 分析器

以下示例显示了如何使用[CPU 探查器][CPU Profiler]:

```js
const inspector = require('node:inspector');
const fs = require('node:fs');
const session = new inspector.Session();
session.connect();

session.post('Profiler.enable', () => {
  session.post('Profiler.start', () => {
    // Invoke business logic under measurement here...

    // some time later...
    session.post('Profiler.stop', (err, { profile }) => {
      // Write profile to disk, upload, etc.
      if (!err) {
        fs.writeFileSync('./profile.cpuprofile', JSON.stringify(profile));
      }
    });
  });
});
```

### 堆探查器

以下示例显示了如何使用[堆探查器][Heap Profiler]:

```js
const inspector = require('node:inspector');
const fs = require('node:fs');
const session = new inspector.Session();

const fd = fs.openSync('profile.heapsnapshot', 'w');

session.connect();

session.on('HeapProfiler.addHeapSnapshotChunk', (m) => {
  fs.writeSync(fd, m.params.chunk);
});

session.post('HeapProfiler.takeHeapSnapshot', null, (err, r) => {
  console.log('HeapProfiler.takeHeapSnapshot done:', err, r);
  session.disconnect();
  fs.closeSync(fd);
});
```

[CPU Profiler]: https://chromedevtools.github.io/devtools-protocol/v8/Profiler

[Chrome DevTools Protocol Viewer]: https://chromedevtools.github.io/devtools-protocol/v8/

[Heap Profiler]: https://chromedevtools.github.io/devtools-protocol/v8/HeapProfiler

[`'Debugger.paused'`]: https://chromedevtools.github.io/devtools-protocol/v8/Debugger#event-paused

[`session.connect()`]: #sessionconnect

[security warning]: cli.md#warning-binding-inspector-to-a-public-ipport-combination-is-insecure

[worker threads]: worker_threads.md
