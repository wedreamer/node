# 过程

<!-- introduced_in=v0.10.0 -->

<!-- type=global -->

<!-- source_link=lib/process.js -->

这`process`对象提供有关当前当前的信息并对其进行控制
节点.js进程。

```mjs
import process from 'node:process';
```

```cjs
const process = require('node:process');
```

## 流程事件

这`process`对象是 的实例[`EventEmitter`][EventEmitter].

### 事件：`'beforeExit'`

<!-- YAML
added: v0.11.12
-->

这`'beforeExit'`当 Node.js 清空其事件循环并具有
无需安排其他工作。通常，节点.js进程将在以下情况下退出：
没有安排任何工作，但有一个侦听器在`'beforeExit'`
事件可以进行异步调用，从而导致 Node.js 进程
继续。

侦听器回调函数的值为
[`process.exitCode`][process.exitCode]作为唯一参数传递。

这`'beforeExit'`事件是*不*针对导致显式的条件发出
终止，例如呼叫[`process.exit()`][process.exit()]或未捕获的异常。

这`'beforeExit'`应该*不*用作`'exit'`事件
除非目的是安排额外的工作。

```mjs
import process from 'node:process';

process.on('beforeExit', (code) => {
  console.log('Process beforeExit event with code: ', code);
});

process.on('exit', (code) => {
  console.log('Process exit event with code: ', code);
});

console.log('This message is displayed first.');

// Prints:
// This message is displayed first.
// Process beforeExit event with code: 0
// Process exit event with code: 0
```

```cjs
const process = require('node:process');

process.on('beforeExit', (code) => {
  console.log('Process beforeExit event with code: ', code);
});

process.on('exit', (code) => {
  console.log('Process exit event with code: ', code);
});

console.log('This message is displayed first.');

// Prints:
// This message is displayed first.
// Process beforeExit event with code: 0
// Process exit event with code: 0
```

### 事件：`'disconnect'`

<!-- YAML
added: v0.7.7
-->

如果使用 IPC 通道生成 Node.js 进程（请参阅[子进程][Child Process]
和[簇][Cluster]文档），`'disconnect'`事件将在以下情况下发出
IPC 通道已关闭。

### 事件：`'exit'`

<!-- YAML
added: v0.1.7
-->

*   `code`{整数}

这`'exit'`事件是在 Node.js 进程即将作为
结果之一：

*   这`process.exit()`显式调用方法;
*   Node.js事件循环不再有任何其他工作要执行。

没有办法阻止此时事件循环的退出，并且一次
都`'exit'`侦听器已完成节点的运行.js进程将终止。

调用侦听器回调函数时，将退出代码指定
由[`process.exitCode`][process.exitCode]属性，或`exitCode`参数传递给
[`process.exit()`][process.exit()]方法。

```mjs
import process from 'node:process';

process.on('exit', (code) => {
  console.log(`About to exit with code: ${code}`);
});
```

```cjs
const process = require('node:process');

process.on('exit', (code) => {
  console.log(`About to exit with code: ${code}`);
});
```

侦听器函数**必须**仅执行**同步**操作。节点.js
进程将在调用 后立即退出`'exit'`事件侦听器
导致放弃仍在事件循环中排队的任何其他工作。
例如，在以下示例中，超时永远不会发生：

```mjs
import process from 'node:process';

process.on('exit', (code) => {
  setTimeout(() => {
    console.log('This will not run');
  }, 0);
});
```

```cjs
const process = require('node:process');

process.on('exit', (code) => {
  setTimeout(() => {
    console.log('This will not run');
  }, 0);
});
```

### 事件：`'message'`

<!-- YAML
added: v0.5.10
-->

*   `message`{ 对象 | 布尔|号 | 字符串 | null } 解析的 JSON 对象
    或可序列化的基元值。
*   `sendHandle`{网.服务器|网。套接字} a[`net.Server`][net.Server]或[`net.Socket`][net.Socket]
    对象，或未定义。

如果使用 IPC 通道生成 Node.js 进程（请参阅[子进程][Child Process]
和[簇][Cluster]文档），`'message'`事件在
父进程使用[`childprocess.send()`][childprocess.send()]接收者
子进程。

消息经过序列化和分析。生成的消息可能
与最初发送的内容不同。

如果`serialization`选项设置为`advanced`生成时使用
过程，`message`参数可以包含 JSON 无法包含的数据
来表示。
看[的高级序列化`child_process`][Advanced serialization for child_process]了解更多详情。

### 事件：`'multipleResolves'`

<!-- YAML
added: v10.12.0
deprecated:
  - v17.6.0
  - v16.15.0
-->

> 稳定性：0 - 已弃用

*   `type`{字符串}分辨率类型。其中之一`'resolve'`或`'reject'`.
*   `promise`{承诺}多次解决或拒绝的承诺。
*   `value`{任何}用于解决承诺的值，或者
    在原始解析后被拒绝。

这`'multipleResolves'`事件在`Promise`已被：

*   已解决多次。
*   多次被拒绝。
*   解决后被拒绝。
*   拒绝后已解决。

这对于在使用
`Promise`构造函数，因为多个分辨率被默默地吞噬。然而
此事件的发生并不一定表示存在错误。为
例[`Promise.race()`][Promise.race()]可以触发`'multipleResolves'`事件。

由于事件在以下情况下的不可靠性
[`Promise.race()`][Promise.race()]上面的例子它已被弃用。

```mjs
import process from 'node:process';

process.on('multipleResolves', (type, promise, reason) => {
  console.error(type, promise, reason);
  setImmediate(() => process.exit(1));
});

async function main() {
  try {
    return await new Promise((resolve, reject) => {
      resolve('First call');
      resolve('Swallowed resolve');
      reject(new Error('Swallowed reject'));
    });
  } catch {
    throw new Error('Failed');
  }
}

main().then(console.log);
// resolve: Promise { 'First call' } 'Swallowed resolve'
// reject: Promise { 'First call' } Error: Swallowed reject
//     at Promise (*)
//     at new Promise (<anonymous>)
//     at main (*)
// First call
```

```cjs
const process = require('node:process');

process.on('multipleResolves', (type, promise, reason) => {
  console.error(type, promise, reason);
  setImmediate(() => process.exit(1));
});

async function main() {
  try {
    return await new Promise((resolve, reject) => {
      resolve('First call');
      resolve('Swallowed resolve');
      reject(new Error('Swallowed reject'));
    });
  } catch {
    throw new Error('Failed');
  }
}

main().then(console.log);
// resolve: Promise { 'First call' } 'Swallowed resolve'
// reject: Promise { 'First call' } Error: Swallowed reject
//     at Promise (*)
//     at new Promise (<anonymous>)
//     at main (*)
// First call
```

### 事件：`'rejectionHandled'`

<!-- YAML
added: v1.4.1
-->

*   `promise`{承诺}迟到的处理承诺。

这`'rejectionHandled'`事件在`Promise`已被拒绝
并附加了一个错误处理程序（使用[`promise.catch()`][promise.catch()]为
example） 晚于 Node.js 事件循环的一轮。

这`Promise`对象以前会在
`'unhandledRejection'`事件，但在处理过程中获得了
拒绝处理程序。

没有顶级的概念`Promise`拒绝的链条
总是被处理。本质上是异步的，`Promise`
拒绝可以在将来的某个时间点处理，可能晚于
事件循环轮次`'unhandledRejection'`要发出的事件。

另一种说法是，与同步代码不同，在同步代码中存在
不断增长的未处理的异常列表，承诺可能会有
不断增长和缩小的未处理的拒绝列表。

在同步代码中，`'uncaughtException'`事件在列表时发出
未处理的异常增长。

在异步代码中，`'unhandledRejection'`当列表发出事件时
未经处理的拒绝增加，并且`'rejectionHandled'`发出事件
当未处理的拒绝列表缩小时。

```mjs
import process from 'node:process';

const unhandledRejections = new Map();
process.on('unhandledRejection', (reason, promise) => {
  unhandledRejections.set(promise, reason);
});
process.on('rejectionHandled', (promise) => {
  unhandledRejections.delete(promise);
});
```

```cjs
const process = require('node:process');

const unhandledRejections = new Map();
process.on('unhandledRejection', (reason, promise) => {
  unhandledRejections.set(promise, reason);
});
process.on('rejectionHandled', (promise) => {
  unhandledRejections.delete(promise);
});
```

在此示例中，`unhandledRejections` `Map`会随着时间的推移而增长和萎缩，
反映拒绝，这些拒绝开始未处理，然后被处理。是的
可以定期在错误日志中记录此类错误（这是
可能最适合长时间运行的应用程序）或进程退出时（这可能是
对于脚本来说最方便）。

### 事件：`'uncaughtException'`

<!-- YAML
added: v0.1.18
changes:
  - version:
     - v12.0.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/26599
    description: Added the `origin` argument.
-->

*   `err`{错误}未捕获的异常。
*   `origin`{字符串}指示异常是否源自未处理的
    拒绝或来自同步错误。可以是`'uncaughtException'`或
    `'unhandledRejection'`.后者在
    `Promise`基于异步上下文（或者如果`Promise`被拒绝）和
    [`--unhandled-rejections`][--unhandled-rejections]标志设置为`strict`或`throw`（这是
    默认值），并且不处理拒绝，或者当拒绝在
    命令行入口点的 ES 模块静态加载阶段。

这`'uncaughtException'`事件是在未捕获的 JavaScript 时发出的
异常气泡一直返回到事件循环。默认情况下，节点.js
通过将堆栈跟踪打印到`stderr`和退出
使用代码 1，覆盖任何先前设置的内容[`process.exitCode`][process.exitCode].
为`'uncaughtException'`事件覆盖此默认值
行为。或者，将[`process.exitCode`][process.exitCode]在
`'uncaughtException'`处理程序，这将导致进程退出
提供退出代码。否则，在存在此类处理程序的情况下，该进程将
以 0 退出。

```mjs
import process from 'node:process';

process.on('uncaughtException', (err, origin) => {
  fs.writeSync(
    process.stderr.fd,
    `Caught exception: ${err}\n` +
    `Exception origin: ${origin}`
  );
});

setTimeout(() => {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

```cjs
const process = require('node:process');

process.on('uncaughtException', (err, origin) => {
  fs.writeSync(
    process.stderr.fd,
    `Caught exception: ${err}\n` +
    `Exception origin: ${origin}`
  );
});

setTimeout(() => {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

可以监控`'uncaughtException'`事件而不覆盖
通过安装
`'uncaughtExceptionMonitor'`听者。

#### 警告：使用`'uncaughtException'`正确

`'uncaughtException'`是异常处理的基本机制
仅用作最后手段。活动*不应该*用作
等效于`On Error Resume Next`.未处理的异常本质上意味着
应用程序处于未定义状态。尝试恢复应用程序
代码未从异常中正确恢复可能会导致额外的
不可预见和不可预测的问题。

不会捕获从事件处理程序中引发的异常。取而代之的是
进程将以非零退出代码退出，并且将打印堆栈跟踪。
这是为了避免无限递归。

在未捕获的异常后尝试正常恢复可能类似于
升级计算机时拔出电源线。十分之九
时代，什么都没发生。但第十次，系统变得腐败。

正确使用`'uncaughtException'`是执行同步清理
关闭前分配的资源（例如文件描述符、句柄等）
整个过程。**恢复正常操作后不安全
`'uncaughtException'`.**

要以更可靠的方式重新启动崩溃的应用程序，请：
`'uncaughtException'`发出与否，应使用外部监视器
在单独的进程中检测应用程序故障并恢复或重新启动
需要。

### 事件：`'uncaughtExceptionMonitor'`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
-->

*   `err`{错误}未捕获的异常。
*   `origin`{字符串}指示异常是否源自未处理的
    拒绝或来自同步错误。可以是`'uncaughtException'`或
    `'unhandledRejection'`.后者在
    `Promise`基于异步上下文（或者如果`Promise`被拒绝）和
    [`--unhandled-rejections`][--unhandled-rejections]标志设置为`strict`或`throw`（这是
    默认值），并且不处理拒绝，或者当拒绝在
    命令行入口点的 ES 模块静态加载阶段。

这`'uncaughtExceptionMonitor'`事件在
`'uncaughtException'`发出事件或通过安装挂接
[`process.setUncaughtExceptionCaptureCallback()`][process.setUncaughtExceptionCaptureCallback()]被调用。

安装`'uncaughtExceptionMonitor'`侦听器不更改行为
一次`'uncaughtException'`发出事件。该过程将
如果没有，仍然崩溃`'uncaughtException'`已安装侦听器。

```mjs
import process from 'node:process';

process.on('uncaughtExceptionMonitor', (err, origin) => {
  MyMonitoringTool.logSync(err, origin);
});

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
// Still crashes Node.js
```

```cjs
const process = require('node:process');

process.on('uncaughtExceptionMonitor', (err, origin) => {
  MyMonitoringTool.logSync(err, origin);
});

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
// Still crashes Node.js
```

### 事件：`'unhandledRejection'`

<!-- YAML
added: v1.4.1
changes:
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8217
    description: Not handling `Promise` rejections is deprecated.
  - version: v6.6.0
    pr-url: https://github.com/nodejs/node/pull/8223
    description: Unhandled `Promise` rejections will now emit
                 a process warning.
-->

*   `reason`{错误|任何}承诺被拒绝的对象
    （通常为[`Error`][Error]对象）。
*   `promise`{承诺}被拒绝的承诺。

这`'unhandledRejection'`事件在`Promise`被拒绝，并且
在事件循环的轮次中，没有错误处理程序附加到 promise。
当使用 Promises 进行编程时，异常被封装为“拒绝”
承诺”。可以使用以下命令捕获和处理拒绝[`promise.catch()`][promise.catch()]和
通过`Promise`链。这`'unhandledRejection'`事件是
可用于检测和跟踪被拒绝的承诺
拒绝尚未处理。

```mjs
import process from 'node:process';

process.on('unhandledRejection', (reason, promise) => {
  console.log('Unhandled Rejection at:', promise, 'reason:', reason);
  // Application specific logging, throwing an error, or other logic here
});

somePromise.then((res) => {
  return reportToUser(JSON.pasre(res)); // Note the typo (`pasre`)
}); // No `.catch()` or `.then()`
```

```cjs
const process = require('node:process');

process.on('unhandledRejection', (reason, promise) => {
  console.log('Unhandled Rejection at:', promise, 'reason:', reason);
  // Application specific logging, throwing an error, or other logic here
});

somePromise.then((res) => {
  return reportToUser(JSON.pasre(res)); // Note the typo (`pasre`)
}); // No `.catch()` or `.then()`
```

以下操作还将触发`'unhandledRejection'`事件
排放：

```mjs
import process from 'node:process';

function SomeResource() {
  // Initially set the loaded status to a rejected promise
  this.loaded = Promise.reject(new Error('Resource not yet loaded!'));
}

const resource = new SomeResource();
// no .catch or .then on resource.loaded for at least a turn
```

```cjs
const process = require('node:process');

function SomeResource() {
  // Initially set the loaded status to a rejected promise
  this.loaded = Promise.reject(new Error('Resource not yet loaded!'));
}

const resource = new SomeResource();
// no .catch or .then on resource.loaded for at least a turn
```

在此示例中，可以将拒绝作为开发人员错误进行跟踪
就像其他情况一样`'unhandledRejection'`事件。自
解决此类故障，非操作性
[`.catch(() => { })`][`promise.catch()`]处理程序可以附加到
`resource.loaded`，这将阻止`'unhandledRejection'`事件来自
正在发出。

### 事件：`'warning'`

<!-- YAML
added: v6.0.0
-->

*   `warning`{错误}警告的主要属性包括：
    *   `name`{字符串}警告的名称。**违约：** `'Warning'`.
    *   `message`{字符串}系统提供的警告说明。
    *   `stack`{字符串}堆栈跟踪到代码中警告所在的位置
        已发出。

这`'warning'`每当 Node.js 发出进程警告时，就会发出事件。

进程警告类似于错误，因为它描述了异常
引起用户注意的条件。但是，警告
不是正常 Node.js 和 JavaScript 错误处理流程的一部分。
Node.js可以在检测到可能不良的编码实践时发出警告
导致应用程序性能、错误或安全漏洞欠佳。

```mjs
import process from 'node:process';

process.on('warning', (warning) => {
  console.warn(warning.name);    // Print the warning name
  console.warn(warning.message); // Print the warning message
  console.warn(warning.stack);   // Print the stack trace
});
```

```cjs
const process = require('node:process');

process.on('warning', (warning) => {
  console.warn(warning.name);    // Print the warning name
  console.warn(warning.message); // Print the warning message
  console.warn(warning.stack);   // Print the stack trace
});
```

默认情况下，Node.js 会将进程警告打印到`stderr`.这`--no-warnings`
命令行选项可用于禁止显示默认控制台输出，但
`'warning'`事件仍将由`process`对象。

下面的示例阐释了打印到`stderr`什么时候
向事件中添加了太多侦听器：

```console
$ node
> events.defaultMaxListeners = 1;
> process.on('foo', () => {});
> process.on('foo', () => {});
> (node:38638) MaxListenersExceededWarning: Possible EventEmitter memory leak
detected. 2 foo listeners added. Use emitter.setMaxListeners() to increase limit
```

相反，以下示例关闭默认警告输出，然后
将自定义处理程序添加到`'warning'`事件：

```console
$ node --no-warnings
> const p = process.on('warning', (warning) => console.warn('Do not do that!'));
> events.defaultMaxListeners = 1;
> process.on('foo', () => {});
> process.on('foo', () => {});
> Do not do that!
```

这`--trace-warnings`命令行选项可用于具有默认值
警告的控制台输出包括警告的完整堆栈跟踪。

启动节点.js使用`--throw-deprecation`命令行标志将
导致自定义弃用警告作为异常引发。

使用`--trace-deprecation`命令行标志将导致自定义
弃用要打印到`stderr`以及堆栈跟踪。

使用`--no-deprecation`命令行标志将禁止显示所有报告
的自定义弃用。

这`*-deprecation`命令行标志仅影响使用该名称的警告
`'DeprecationWarning'`.

### 事件：`'worker'`

<!-- YAML
added:
  - v16.2.0
  - v14.18.0
-->

*   `worker`{工人}已创建的 {工作线程}。

这`'worker'`事件是在创建新的 {Worker} 线程后发出的。

#### 发出自定义警告

查看[`process.emitWarning()`][process_emit_warning]发行方法
自定义或特定于应用程序的警告。

#### 节点.js警告名称

对于警告类型没有严格的准则（由`name`
属性） 由 Node.js 发出。可以随时添加新的警告类型。
一些最常见的警告类型包括：

*   `'DeprecationWarning'`- 指示使用已弃用的 Node.js API 或功能。
    此类警告必须包括`'code'`属性标识
    [弃用代码][deprecation code].
*   `'ExperimentalWarning'`- 表示使用实验性节点.js API 或
    特征。必须谨慎使用此类功能，因为它们可能会在任何情况下更改
    时间，并且不受同样严格的语义版本控制和长期的约束
    支持策略作为受支持的功能。
*   `'MaxListenersExceededWarning'`- 表示侦听器过多
    给定事件已在`EventEmitter`或`EventTarget`.
    这通常表示内存泄漏。
*   `'TimeoutOverflowWarning'`- 指示无法拟合的数值
    在 32 位有符号整数内已提供给`setTimeout()`
    或`setInterval()`功能。
*   `'UnsupportedWarning'`- 指示使用不受支持的选项或功能
    这将被忽略，而不是被视为错误。一个例子是使用
    使用 HTTP/2 兼容性 API 时的 HTTP 响应状态消息。

### 信号事件

<!--type=event-->

<!--name=SIGINT, SIGHUP, etc.-->

当 Node.js 进程收到信号时，将发出信号事件。请
请参阅 signal（7） 以获取标准 POSIX 信号名称的列表，例如
`'SIGINT'`,`'SIGHUP'`等。

信号在 上不可用[`Worker`][Worker]线程。

信号处理程序将接收信号的名称 （`'SIGINT'`,
`'SIGTERM'`等）作为第一个参数。

每个事件的名称将是信号的大写通用名称（例如
`'SIGINT'`为`SIGINT`信号）。

```mjs
import process from 'node:process';

// Begin reading from stdin so the process does not exit.
process.stdin.resume();

process.on('SIGINT', () => {
  console.log('Received SIGINT. Press Control-D to exit.');
});

// Using a single function to handle multiple signals
function handle(signal) {
  console.log(`Received ${signal}`);
}

process.on('SIGINT', handle);
process.on('SIGTERM', handle);
```

```cjs
const process = require('node:process');

// Begin reading from stdin so the process does not exit.
process.stdin.resume();

process.on('SIGINT', () => {
  console.log('Received SIGINT. Press Control-D to exit.');
});

// Using a single function to handle multiple signals
function handle(signal) {
  console.log(`Received ${signal}`);
}

process.on('SIGINT', handle);
process.on('SIGTERM', handle);
```

*   `'SIGUSR1'`由 Node 保留.js以启动[调试器][debugger].有可能
    安装侦听器，但这样做可能会干扰调试器。
*   `'SIGTERM'`和`'SIGINT'`在非 Windows 平台上具有默认处理程序，这些处理程序
    在使用代码退出之前重置终端模式`128 + signal number`.如果有的话
    这些信号中安装了侦听器，其默认行为将为
    已删除（节点.js将不再退出）。
*   `'SIGPIPE'`默认情况下忽略。它可以安装侦听器。
*   `'SIGHUP'`在控制台窗口关闭时在 Windows 上生成，并在
    其他平台在各种类似条件下。参见信号（7）。它可以有一个
    已安装的侦听器，但 Node.js 将被无条件终止
    大约10秒后。在非 Windows 平台上，默认
    的行为`SIGHUP`是终止节点.js，但一旦侦听器已被
    已安装，其默认行为将被删除。
*   `'SIGTERM'`在Windows上不受支持，它可以被监听。
*   `'SIGINT'`从终端在所有平台上都受支持，并且通常可以是
    生成于<kbd>Ctrl</kbd>+<kbd>C</kbd>（尽管这可能是可配置的）。
    在以下情况下不会生成它[终端原始模式][terminal raw mode]已启用
    和<kbd>Ctrl</kbd>+<kbd>C</kbd>使用。
*   `'SIGBREAK'`在以下情况下在 Windows 上交付<kbd>Ctrl</kbd>+<kbd>破</kbd>是
    压。在非Windows平台上，它可以被监听，但没有办法
    以发送或生成它。
*   `'SIGWINCH'`在调整控制台大小时传送。在 Windows 上，此
    仅在移动光标时写入控制台时发生，或者
    在原始模式下使用可读 tty 时。
*   `'SIGKILL'`无法安装侦听器，它将无条件地
    在所有平台上终止节点.js。
*   `'SIGSTOP'`无法安装侦听器。
*   `'SIGBUS'`,`'SIGFPE'`,`'SIGSEGV'`和`'SIGILL'`，未引发时
    人为地使用 kill（2）， 固有地使过程处于
    调用 JS 侦听器是不安全的。这样做可能会导致该过程
    以停止响应。
*   `0`可以发送测试是否存在一个进程，如果
    该进程存在，但如果该进程不存在，则会引发错误。

Windows不支持信号，因此没有等效于信号终止，
但是Node.js提供了一些仿真[`process.kill()`][process.kill()]和
[`subprocess.kill()`][subprocess.kill()]:

*   发送`SIGINT`,`SIGTERM`和`SIGKILL`将导致无条件
    终止目标进程，之后，子进程将报告
    该过程被信号终止。
*   发送信号`0`可作为平台独立方式进行测试
    存在一个进程。

## `process.abort()`

<!-- YAML
added: v0.7.0
-->

这`process.abort()`方法导致 Node.js 进程立即退出，并且
生成核心文件。

此功能在 中不可用[`Worker`][Worker]线程。

## `process.allowedNodeEnvironmentFlags`

<!-- YAML
added: v10.10.0
-->

*   {设置}

这`process.allowedNodeEnvironmentFlags`物业是特别的，
只读`Set`允许的标志数量[`NODE_OPTIONS`][NODE_OPTIONS]
环境变量。

`process.allowedNodeEnvironmentFlags`延伸`Set`，但覆盖
`Set.prototype.has`识别几个不同的可能标志
交涉。`process.allowedNodeEnvironmentFlags.has()`将
返回`true`在以下情况下：

*   标志可以省略前导单 （`-`） 或双 （`--`） 破折号;例如，
    `inspect-brk`为`--inspect-brk`或`r`为`-r`.
*   传递到 V8 的标志（如`--v8-options`） 可替换
    一个或多个*非领先*下划线的破折号，反之亦然;
    例如，`--perf_basic_prof`,`--perf-basic-prof`,`--perf_basic-prof`,
    等。
*   标志可以包含一个或多个等于 （`=`） 字符;都
    字符之后和包含第一个等于将被忽略;
    例如，`--stack-trace-limit=100`.
*   标志*必须*允许在[`NODE_OPTIONS`][NODE_OPTIONS].

迭代时`process.allowedNodeEnvironmentFlags`，标志将
仅显示*一次*;每个都将以一个或多个破折号开头。标志
传递到 V8 将包含下划线而不是非前导
破折号：

```mjs
import { allowedNodeEnvironmentFlags } from 'node:process';

allowedNodeEnvironmentFlags.forEach((flag) => {
  // -r
  // --inspect-brk
  // --abort_on_uncaught_exception
  // ...
});
```

```cjs
const { allowedNodeEnvironmentFlags } = require('node:process');

allowedNodeEnvironmentFlags.forEach((flag) => {
  // -r
  // --inspect-brk
  // --abort_on_uncaught_exception
  // ...
});
```

方法`add()`,`clear()`和`delete()`之
`process.allowedNodeEnvironmentFlags`什么都不做，并且会失败
默默。

如果编译了节点.js*没有* [`NODE_OPTIONS`][NODE_OPTIONS]支持（如
[`process.config`][process.config]),`process.allowedNodeEnvironmentFlags`将
包含什么*会有*是允许的。

## `process.arch`

<!-- YAML
added: v0.5.0
-->

*   {字符串}

为其编译 Node.js 二进制文件的操作系统 CPU 体系结构。
可能的值包括：`'arm'`,`'arm64'`,`'ia32'`,`'mips'`,`'mipsel'`,`'ppc'`,
`'ppc64'`,`'s390'`,`'s390x'`和`'x64'`.

```mjs
import { arch } from 'node:process';

console.log(`This processor architecture is ${arch}`);
```

```cjs
const { arch } = require('node:process');

console.log(`This processor architecture is ${arch}`);
```

## `process.argv`

<!-- YAML
added: v0.1.27
-->

*   {字符串\[]}

这`process.argv`属性返回包含命令行的数组
启动 Node.js 进程时传递的参数。第一个元素将
是[`process.execPath`][process.execPath].看`process.argv0`如果访问原始值
之`argv[0]`是必需的。第二个元素将是JavaScript的路径
文件正在执行。其余元素将是任何其他命令行
参数。

例如，假设以下脚本`process-args.js`:

```mjs
import { argv } from 'node:process';

// print process.argv
argv.forEach((val, index) => {
  console.log(`${index}: ${val}`);
});
```

```cjs
const { argv } = require('node:process');

// print process.argv
argv.forEach((val, index) => {
  console.log(`${index}: ${val}`);
});
```

启动节点.js进程为：

```console
$ node process-args.js one two=three four
```

将生成输出：

```text
0: /usr/local/bin/node
1: /Users/mjr/work/node/process-args.js
2: one
3: two=three
4: four
```

## `process.argv0`

<!-- YAML
added: v6.4.0
-->

*   {字符串}

这`process.argv0`属性存储原始值的只读副本
`argv[0]`在 Node.js 启动时通过。

```console
$ bash -c 'exec -a customArgv0 ./node'
> process.argv[0]
'/Volumes/code/external/node/out/Release/node'
> process.argv0
'customArgv0'
```

## `process.channel`

<!-- YAML
added: v7.1.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/30165
    description: The object no longer accidentally exposes native C++ bindings.
-->

*   {对象}

如果节点.js进程是使用 IPC 通道生成的（请参阅
[子进程][Child Process]文档），`process.channel`
属性是对 IPC 通道的引用。如果不存在 IPC 通道，则
属性是`undefined`.

### `process.channel.ref()`

<!-- YAML
added: v7.1.0
-->

此方法使 IPC 通道保持进程的事件循环
在以下情况下运行：`.unref()`以前被叫过。

通常，这是通过`'disconnect'`和`'message'`
侦听器`process`对象。但是，此方法可用于
显式请求特定行为。

### `process.channel.unref()`

<!-- YAML
added: v7.1.0
-->

此方法使 IPC 通道不保留进程的事件循环
运行，并让它完成，即使在通道打开时也是如此。

通常，这是通过`'disconnect'`和`'message'`
侦听器`process`对象。但是，此方法可用于
显式请求特定行为。

## `process.chdir(directory)`

<!-- YAML
added: v0.1.17
-->

*   `directory`{字符串}

这`process.chdir()`方法更改 的当前工作目录
Node.js处理或引发异常（例如，如果这样做失败）
指定的`directory`不存在）。

```mjs
import { chdir, cwd } from 'node:process';

console.log(`Starting directory: ${cwd()}`);
try {
  chdir('/tmp');
  console.log(`New directory: ${cwd()}`);
} catch (err) {
  console.error(`chdir: ${err}`);
}
```

```cjs
const { chdir, cwd } = require('node:process');

console.log(`Starting directory: ${cwd()}`);
try {
  chdir('/tmp');
  console.log(`New directory: ${cwd()}`);
} catch (err) {
  console.error(`chdir: ${err}`);
}
```

此功能在 中不可用[`Worker`][Worker]线程。

## `process.config`

<!-- YAML
added: v0.7.7
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36902
    description: Modifying process.config has been deprecated.
-->

*   {对象}

这`process.config`属性返回`Object`包含 JavaScript
用于编译当前节点的配置选项的表示形式.js
可执行。这与`config.gypi`在以下时间生成的文件
运行`./configure`脚本。

可能输出的示例如下所示：

<!-- eslint-skip -->

```js
{
  target_defaults:
   { cflags: [],
     default_configuration: 'Release',
     defines: [],
     include_dirs: [],
     libraries: [] },
  variables:
   {
     host_arch: 'x64',
     napi_build_version: 5,
     node_install_npm: 'true',
     node_prefix: '',
     node_shared_cares: 'false',
     node_shared_http_parser: 'false',
     node_shared_libuv: 'false',
     node_shared_zlib: 'false',
     node_use_openssl: 'true',
     node_shared_openssl: 'false',
     strict_aliasing: 'true',
     target_arch: 'x64',
     v8_use_snapshot: 1
   }
}
```

这`process.config`属性是**不**只读，并且存在
生态系统中已知可以扩展、修改或完全替换的模块
的价值`process.config`.

修改`process.config`财产，或
`process.config`对象已被弃用。这`process.config`将制作
在将来的版本中是只读的。

## `process.connected`

<!-- YAML
added: v0.7.2
-->

*   {布尔值}

如果使用 IPC 通道生成 Node.js 进程（请参阅[子进程][Child Process]
和[簇][Cluster]文档），`process.connected`物业会回来
`true`只要IPC通道已连接并返回`false`后
`process.disconnect()`被调用。

一次`process.connected`是`false`，则无法再发送消息
通过 IPC 通道使用`process.send()`.

## `process.cpuUsage([previousValue])`

<!-- YAML
added: v6.1.0
-->

*   `previousValue`{对象}调用时返回的上一个值
    `process.cpuUsage()`
*   返回： {对象}
    *   `user`{整数}
    *   `system`{整数}

这`process.cpuUsage()`方法返回用户和系统 CPU 时间使用情况
当前进程，在具有属性的对象中`user`和`system`谁的
值是微秒值（百万分之一秒）。这些值测量时间
分别在用户和系统代码中花费，并且可能最终大于
如果多个 CPU 内核正在为此过程执行工作，则实际运行时间。

上一次调用的结果`process.cpuUsage()`可以作为
参数到函数，以获取差异读取。

```mjs
import { cpuUsage } from 'node:process';

const startUsage = cpuUsage();
// { user: 38579, system: 6986 }

// spin the CPU for 500 milliseconds
const now = Date.now();
while (Date.now() - now < 500);

console.log(cpuUsage(startUsage));
// { user: 514883, system: 11226 }
```

```cjs
const { cpuUsage } = require('node:process');

const startUsage = cpuUsage();
// { user: 38579, system: 6986 }

// spin the CPU for 500 milliseconds
const now = Date.now();
while (Date.now() - now < 500);

console.log(cpuUsage(startUsage));
// { user: 514883, system: 11226 }
```

## `process.cwd()`

<!-- YAML
added: v0.1.8
-->

*   返回：{字符串}

这`process.cwd()`方法返回节点的当前工作目录.js
过程。

```mjs
import { cwd } from 'node:process';

console.log(`Current directory: ${cwd()}`);
```

```cjs
const { cwd } = require('node:process');

console.log(`Current directory: ${cwd()}`);
```

## `process.debugPort`

<!-- YAML
added: v0.7.2
-->

*   {数字}

启用时 Node.js调试器使用的端口。

```mjs
import process from 'node:process';

process.debugPort = 5858;
```

```cjs
const process = require('node:process');

process.debugPort = 5858;
```

## `process.disconnect()`

<!-- YAML
added: v0.7.2
-->

如果使用 IPC 通道生成 Node.js 进程（请参阅[子进程][Child Process]
和[簇][Cluster]文档），`process.disconnect()`方法将关闭
IPC 通道到父进程，允许子进程正常退出
一旦没有其他联系使它活着。

通话效果`process.disconnect()`与呼叫相同
[`ChildProcess.disconnect()`][ChildProcess.disconnect()]从父进程。

如果节点.js进程未通过 IPC 通道生成，
`process.disconnect()`将是`undefined`.

## `process.dlopen(module, filename[, flags])`

<!-- YAML
added: v0.1.16
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/12794
    description: Added support for the `flags` argument.
-->

*   `module`{对象}
*   `filename`{字符串}
*   `flags`{os.constants.dlopen}**违约：** `os.constants.dlopen.RTLD_LAZY`

这`process.dlopen()`方法允许动态加载共享对象。是的
主要用于`require()`加载C++插件，不应使用
直接，特殊情况除外。换句话说，[`require()`][require()]应该是
首选`process.dlopen()`除非有具体原因，例如
自定义 dlopen 标志或从 ES 模块加载。

这`flags`参数是一个整数，允许指定 dlopen
行为。查看[`os.constants.dlopen`][os.constants.dlopen]文档以了解详细信息。

呼叫时的重要要求`process.dlopen()`是`module`
必须传递实例。然后，由C++插件导出的函数
可通过以下方式访问`module.exports`.

下面的示例演示如何加载C++插件，命名为`local.node`,
导出`foo`功能。所有符号在加载之前
调用返回，通过传递`RTLD_NOW`不断。在此示例中
假定该常量可用。

```mjs
import { dlopen } from 'node:process';
import { constants } from 'node:os';
import { fileURLToPath } from 'node:url';

const module = { exports: {} };
dlopen(module, fileURLToPath(new URL('local.node', import.meta.url)),
       constants.dlopen.RTLD_NOW);
module.exports.foo();
```

```cjs
const { dlopen } = require('node:process');
const { constants } = require('node:os');
const { join } = require('node:path');

const module = { exports: {} };
dlopen(module, join(__dirname, 'local.node'), constants.dlopen.RTLD_NOW);
module.exports.foo();
```

## `process.emitWarning(warning[, options])`

<!-- YAML
added: v8.0.0
-->

*   `warning`{字符串|错误} 要发出的警告。
*   `options`{对象}
    *   `type`{字符串}什么时候`warning`是一个`String`,`type`是要使用的名称
        对于*类型*发出警告。**违约：** `'Warning'`.
    *   `code`{字符串}发出的警告实例的唯一标识符。
    *   `ctor`{函数}什么时候`warning`是一个`String`,`ctor`是可选的
        用于限制生成的堆栈跟踪的函数。**违约：**
        `process.emitWarning`.
    *   `detail`{字符串}要包含在错误中的其他文本。

这`process.emitWarning()`方法可用于发出自定义或应用程序
特定进程警告。可以通过将处理程序添加到
[`'warning'`][process_warning]事件。

```mjs
import { emitWarning } from 'node:process';

// Emit a warning with a code and additional detail.
emitWarning('Something happened!', {
  code: 'MY_WARNING',
  detail: 'This is some additional information'
});
// Emits:
// (node:56338) [MY_WARNING] Warning: Something happened!
// This is some additional information
```

```cjs
const { emitWarning } = require('node:process');

// Emit a warning with a code and additional detail.
emitWarning('Something happened!', {
  code: 'MY_WARNING',
  detail: 'This is some additional information'
});
// Emits:
// (node:56338) [MY_WARNING] Warning: Something happened!
// This is some additional information
```

在此示例中，一个`Error`对象由内部生成
`process.emitWarning()`并传递到
[`'warning'`][process_warning]处理器。

```mjs
import process from 'node:process';

process.on('warning', (warning) => {
  console.warn(warning.name);    // 'Warning'
  console.warn(warning.message); // 'Something happened!'
  console.warn(warning.code);    // 'MY_WARNING'
  console.warn(warning.stack);   // Stack trace
  console.warn(warning.detail);  // 'This is some additional information'
});
```

```cjs
const process = require('node:process');

process.on('warning', (warning) => {
  console.warn(warning.name);    // 'Warning'
  console.warn(warning.message); // 'Something happened!'
  console.warn(warning.code);    // 'MY_WARNING'
  console.warn(warning.stack);   // Stack trace
  console.warn(warning.detail);  // 'This is some additional information'
});
```

如果`warning`作为传递`Error`对象，`options`参数被忽略。

## `process.emitWarning(warning[, type[, code]][, ctor])`

<!-- YAML
added: v6.0.0
-->

*   `warning`{字符串|错误} 要发出的警告。
*   `type`{字符串}什么时候`warning`是一个`String`,`type`是要使用的名称
    对于*类型*发出警告。**违约：** `'Warning'`.
*   `code`{字符串}发出的警告实例的唯一标识符。
*   `ctor`{函数}什么时候`warning`是一个`String`,`ctor`是可选的
    用于限制生成的堆栈跟踪的函数。**违约：**
    `process.emitWarning`.

这`process.emitWarning()`方法可用于发出自定义或应用程序
特定进程警告。可以通过将处理程序添加到
[`'warning'`][process_warning]事件。

```mjs
import { emitWarning } from 'node:process';

// Emit a warning using a string.
emitWarning('Something happened!');
// Emits: (node: 56338) Warning: Something happened!
```

```cjs
const { emitWarning } = require('node:process');

// Emit a warning using a string.
emitWarning('Something happened!');
// Emits: (node: 56338) Warning: Something happened!
```

```mjs
import { emitWarning } from 'node:process';

// Emit a warning using a string and a type.
emitWarning('Something Happened!', 'CustomWarning');
// Emits: (node:56338) CustomWarning: Something Happened!
```

```cjs
const { emitWarning } = require('node:process');

// Emit a warning using a string and a type.
emitWarning('Something Happened!', 'CustomWarning');
// Emits: (node:56338) CustomWarning: Something Happened!
```

```mjs
import { emitWarning } from 'node:process';

emitWarning('Something happened!', 'CustomWarning', 'WARN001');
// Emits: (node:56338) [WARN001] CustomWarning: Something happened!
```

```cjs
const { emitWarning } = require('node:process');

process.emitWarning('Something happened!', 'CustomWarning', 'WARN001');
// Emits: (node:56338) [WARN001] CustomWarning: Something happened!
```

在前面的每个示例中，一个`Error`对象由内部生成
`process.emitWarning()`并传递到[`'warning'`][process_warning]
处理器。

```mjs
import process from 'node:process';

process.on('warning', (warning) => {
  console.warn(warning.name);
  console.warn(warning.message);
  console.warn(warning.code);
  console.warn(warning.stack);
});
```

```cjs
const process = require('node:process');

process.on('warning', (warning) => {
  console.warn(warning.name);
  console.warn(warning.message);
  console.warn(warning.code);
  console.warn(warning.stack);
});
```

如果`warning`作为传递`Error`对象，它将传递到
`'warning'`未修改的事件处理程序（以及可选`type`,
`code`和`ctor`参数将被忽略）：

```mjs
import { emitWarning } from 'node:process';

// Emit a warning using an Error object.
const myWarning = new Error('Something happened!');
// Use the Error name property to specify the type name
myWarning.name = 'CustomWarning';
myWarning.code = 'WARN001';

emitWarning(myWarning);
// Emits: (node:56338) [WARN001] CustomWarning: Something happened!
```

```cjs
const { emitWarning } = require('node:process');

// Emit a warning using an Error object.
const myWarning = new Error('Something happened!');
// Use the Error name property to specify the type name
myWarning.name = 'CustomWarning';
myWarning.code = 'WARN001';

emitWarning(myWarning);
// Emits: (node:56338) [WARN001] CustomWarning: Something happened!
```

一个`TypeError`在以下情况下被抛出`warning`是字符串以外的任何内容，或者`Error`
对象。

当进程警告使用`Error`对象，进程警告
机制是**不**替代正常的错误处理机制。

如果出现警告，则实现以下附加处理`type`是
`'DeprecationWarning'`:

*   如果`--throw-deprecation`使用命令行标志，弃用
    警告作为异常引发，而不是作为事件发出。
*   如果`--no-deprecation`使用命令行标志，弃用
    警告被抑制。
*   如果`--trace-deprecation`使用命令行标志，弃用
    警告打印到`stderr`以及完整的堆栈跟踪。

### 避免重复警告

最佳做法是，每个进程只应发出一次警告。待办事项
所以，把`emitWarning()`在布尔值后面。

```mjs
import { emitWarning } from 'node:process';

function emitMyWarning() {
  if (!emitMyWarning.warned) {
    emitMyWarning.warned = true;
    emitWarning('Only warn once!');
  }
}
emitMyWarning();
// Emits: (node: 56339) Warning: Only warn once!
emitMyWarning();
// Emits nothing
```

```cjs
const { emitWarning } = require('node:process');

function emitMyWarning() {
  if (!emitMyWarning.warned) {
    emitMyWarning.warned = true;
    emitWarning('Only warn once!');
  }
}
emitMyWarning();
// Emits: (node: 56339) Warning: Only warn once!
emitMyWarning();
// Emits nothing
```

## `process.env`

<!-- YAML
added: v0.1.27
changes:
  - version: v11.14.0
    pr-url: https://github.com/nodejs/node/pull/26544
    description: Worker threads will now use a copy of the parent thread's
                 `process.env` by default, configurable through the `env`
                 option of the `Worker` constructor.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18990
    description: Implicit conversion of variable value to string is deprecated.
-->

*   {对象}

这`process.env`属性返回包含用户环境的对象。
参见环境（7）。

此对象的示例如下所示：

<!-- eslint-skip -->

```js
{
  TERM: 'xterm-256color',
  SHELL: '/usr/local/bin/bash',
  USER: 'maciej',
  PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
  PWD: '/Users/maciej',
  EDITOR: 'vim',
  SHLVL: '1',
  HOME: '/Users/maciej',
  LOGNAME: 'maciej',
  _: '/usr/local/bin/node'
}
```

可以修改此对象，但此类修改不会
反映在节点外部.js进程，或（除非明确请求）
到其他[`Worker`][Worker]线程。
换句话说，以下示例将不起作用：

```console
$ node -e 'process.env.foo = "bar"' && echo $foo
```

虽然以下将：

```mjs
import { env } from 'node:process';

env.foo = 'bar';
console.log(env.foo);
```

```cjs
const { env } = require('node:process');

env.foo = 'bar';
console.log(env.foo);
```

在 上分配属性`process.env`将隐式转换值
转换为字符串。**此行为已弃用。**Node.js的未来版本可能会
当值不是字符串、数字或布尔值时引发错误。

```mjs
import { env } from 'node:process';

env.test = null;
console.log(env.test);
// => 'null'
env.test = undefined;
console.log(env.test);
// => 'undefined'
```

```cjs
const { env } = require('node:process');

env.test = null;
console.log(env.test);
// => 'null'
env.test = undefined;
console.log(env.test);
// => 'undefined'
```

用`delete`从中删除属性`process.env`.

```mjs
import { env } from 'node:process';

env.TEST = 1;
delete env.TEST;
console.log(env.TEST);
// => undefined
```

```cjs
const { env } = require('node:process');

env.TEST = 1;
delete env.TEST;
console.log(env.TEST);
// => undefined
```

在 Windows 操作系统上，环境变量不区分大小写。

```mjs
import { env } from 'node:process';

env.TEST = 1;
console.log(env.test);
// => 1
```

```cjs
const { env } = require('node:process');

env.TEST = 1;
console.log(env.test);
// => 1
```

除非在创建[`Worker`][Worker]实例
每[`Worker`][Worker]线程有自己的副本`process.env`，基于其
父线程的`process.env`，或任何被指定为`env`选择
到[`Worker`][Worker]构造 函数。更改为`process.env`将不可见
横[`Worker`][Worker]线程，并且只有主线程才能进行更改
对操作系统或本机加载项可见。

## `process.execArgv`

<!-- YAML
added: v0.7.7
-->

*   {字符串\[]}

这`process.execArgv`属性返回特定于 Node.js 命令行的集合
启动 Node.js 进程时传递的选项。这些选项不
出现在 返回的数组中[`process.argv`][process.argv]属性，并且不
包括 Node.js可执行文件、脚本名称或以下任何选项
脚本名称。这些选项对于生成子进程非常有用
与父级相同的执行环境。

```console
$ node --harmony script.js --version
```

结果在`process.execArgv`:

<!-- eslint-disable semi -->

```js
['--harmony']
```

和`process.argv`:

<!-- eslint-disable semi -->

```js
['/usr/local/bin/node', 'script.js', '--version']
```

指[`Worker`构造 函数][Worker constructor]了解工人的详细行为
具有此属性的线程。

## `process.execPath`

<!-- YAML
added: v0.1.100
-->

*   {字符串}

这`process.execPath`属性返回可执行文件的绝对路径名
启动了 Node.js 进程。符号链接（如果有）已解析。

<!-- eslint-disable semi -->

```js
'/usr/local/bin/node'
```

## `process.exit([code])`

<!-- YAML
added: v0.1.13
-->

*   `code`{整数}退出代码。**违约：** `0`.

这`process.exit()`方法指示 Node.js 终止进程
与退出状态同步`code`.如果`code`省略，退出使用
“成功”代码`0`或`process.exitCode`如果它已经
设置。节点.js不会终止，直到所有[`'exit'`]['exit']事件侦听器是
叫。

要使用“失败”代码退出：

```mjs
import { exit } from 'node:process';

exit(1);
```

```cjs
const { exit } = require('node:process');

exit(1);
```

执行 Node.js 的 shell 应将退出代码视为`1`.

叫`process.exit()`将强制进程尽快退出
即使仍有尚未挂起的异步操作
完全完成，包括 I/O 操作`process.stdout`和
`process.stderr`.

在大多数情况下，实际上没有必要调用`process.exit()`
明确地。节点.js进程将自行退出*如果没有额外的
待定工作*在事件循环中。这`process.exitCode`属性可以设置为
告诉进程在进程正常退出时要使用的退出代码。

例如，以下示例说明了*滥用*的
`process.exit()`可能导致数据打印到标准输出的方法
截断并丢失：

```mjs
import { exit } from 'node:process';

// This is an example of what *not* to do:
if (someConditionNotMet()) {
  printUsageToStdout();
  exit(1);
}
```

```cjs
const { exit } = require('node:process');

// This is an example of what *not* to do:
if (someConditionNotMet()) {
  printUsageToStdout();
  exit(1);
}
```

这是有问题的原因是因为写入`process.stdout`在节点中.js
有时*异步*并且可能发生在节点的多个价格变动中.js
事件循环。叫`process.exit()`，但是，强制进程退出
*以前*那些额外的写到`stdout`可以执行。

而不是打电话`process.exit()`直接，代码*应该*设置
`process.exitCode`并允许该过程通过避免自然退出
为事件循环安排任何其他工作：

```mjs
import process from 'node:process';

// How to properly set the exit code while letting
// the process exit gracefully.
if (someConditionNotMet()) {
  printUsageToStdout();
  process.exitCode = 1;
}
```

```cjs
const process = require('node:process');

// How to properly set the exit code while letting
// the process exit gracefully.
if (someConditionNotMet()) {
  printUsageToStdout();
  process.exitCode = 1;
}
```

如果由于错误情况而需要终止节点.js进程，
抛出一个*未捕获*错误并允许进程相应地终止
比打电话更安全`process.exit()`.

在[`Worker`][Worker]线程，此函数停止当前线程
比当前进程。

## `process.exitCode`

<!-- YAML
added: v0.11.8
-->

*   {整数}

一个数字，它将是进程退出代码，当进程
优雅地退出，或通过以下方式退出[`process.exit()`][process.exit()]未指定
代码。

将代码指定为[`process.exit(code)`][`process.exit()`]将覆盖任何
以前的设置`process.exitCode`.

## `process.getActiveResourcesInfo()`

<!-- YAML
added:
  - v17.3.0
  - v16.14.0
-->

> 稳定性： 1 - 实验

*   返回：{字符串\[]}

这`process.getActiveResourcesInfo()`方法返回字符串数组
包含当前保留的活动资源的类型
事件循环处于活动状态。

```mjs
import { getActiveResourcesInfo } from 'node:process';
import { setTimeout } from 'node:timers';

console.log('Before:', getActiveResourcesInfo());
setTimeout(() => {}, 1000);
console.log('After:', getActiveResourcesInfo());
// Prints:
//   Before: [ 'CloseReq', 'TTYWrap', 'TTYWrap', 'TTYWrap' ]
//   After: [ 'CloseReq', 'TTYWrap', 'TTYWrap', 'TTYWrap', 'Timeout' ]
```

```cjs
const { getActiveResourcesInfo } = require('node:process');
const { setTimeout } = require('node:timers');

console.log('Before:', getActiveResourcesInfo());
setTimeout(() => {}, 1000);
console.log('After:', getActiveResourcesInfo());
// Prints:
//   Before: [ 'TTYWrap', 'TTYWrap', 'TTYWrap' ]
//   After: [ 'TTYWrap', 'TTYWrap', 'TTYWrap', 'Timeout' ]
```

## `process.getegid()`

<!-- YAML
added: v2.0.0
-->

这`process.getegid()`方法返回数字有效组标识
的节点.js进程。（参见 getegid（2）。

```mjs
import process from 'node:process';

if (process.getegid) {
  console.log(`Current gid: ${process.getegid()}`);
}
```

```cjs
const process = require('node:process');

if (process.getegid) {
  console.log(`Current gid: ${process.getegid()}`);
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。

## `process.geteuid()`

<!-- YAML
added: v2.0.0
-->

*   返回： {对象}

这`process.geteuid()`方法返回
该过程。（参见 geteuid（2）。

```mjs
import process from 'node:process';

if (process.geteuid) {
  console.log(`Current uid: ${process.geteuid()}`);
}
```

```cjs
const process = require('node:process');

if (process.geteuid) {
  console.log(`Current uid: ${process.geteuid()}`);
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。

## `process.getgid()`

<!-- YAML
added: v0.1.31
-->

*   返回： {对象}

这`process.getgid()`方法返回
过程。（参见 getgid（2）。

```mjs
import process from 'node:process';

if (process.getgid) {
  console.log(`Current gid: ${process.getgid()}`);
}
```

```cjs
const process = require('node:process');

if (process.getgid) {
  console.log(`Current gid: ${process.getgid()}`);
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。

## `process.getgroups()`

<!-- YAML
added: v0.9.4
-->

*   返回：{整数\[]}

这`process.getgroups()`方法返回一个包含补充组的数组
身份证。POSIX 未指定它是否包含有效的组 ID，但
Node.js确保它始终如此。

```mjs
import process from 'node:process';

if (process.getgroups) {
  console.log(process.getgroups()); // [ 16, 21, 297 ]
}
```

```cjs
const process = require('node:process');

if (process.getgroups) {
  console.log(process.getgroups()); // [ 16, 21, 297 ]
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。

## `process.getuid()`

<!-- YAML
added: v0.1.28
-->

*   返回：{整数}

这`process.getuid()`方法返回进程的数字用户标识。
（参见 getuid（2）。

```mjs
import process from 'node:process';

if (process.getuid) {
  console.log(`Current uid: ${process.getuid()}`);
}
```

```cjs
const process = require('node:process');

if (process.getuid) {
  console.log(`Current uid: ${process.getuid()}`);
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。

## `process.hasUncaughtExceptionCaptureCallback()`

<!-- YAML
added: v9.3.0
-->

*   返回：{布尔值}

指示是否已使用
[`process.setUncaughtExceptionCaptureCallback()`][process.setUncaughtExceptionCaptureCallback()].

## `process.hrtime([time])`

<!-- YAML
added: v0.7.6
-->

> 稳定性： 3 - 旧版。用[`process.hrtime.bigint()`][process.hrtime.bigint()]相反。

*   `time`{整数\[]}上一次调用的结果`process.hrtime()`
*   返回：{整数\[]}

这是 的旧版[`process.hrtime.bigint()`][process.hrtime.bigint()]
以前`bigint`在 JavaScript 中引入。

这`process.hrtime()`方法返回当前高分辨率实时
在`[seconds, nanoseconds]`元`Array`哪里`nanoseconds`是
无法以秒精度表示的实时的剩余部分。

`time`是一个可选参数，必须是前一个参数的结果
`process.hrtime()`调用以与当前时间比较。如果参数
传入不是元组`Array`一个`TypeError`将被抛出。在
用户定义的数组，而不是上一次调用的结果
`process.hrtime()`将导致未定义的行为。

这些时间是相对于
过去，与一天中的时间无关，因此不受时钟的影响
漂移。主要用途是测量间隔之间的性能：

```mjs
import { hrtime } from 'node:process';

const NS_PER_SEC = 1e9;
const time = hrtime();
// [ 1800216, 25 ]

setTimeout(() => {
  const diff = hrtime(time);
  // [ 1, 552 ]

  console.log(`Benchmark took ${diff[0] * NS_PER_SEC + diff[1]} nanoseconds`);
  // Benchmark took 1000000552 nanoseconds
}, 1000);
```

```cjs
const { hrtime } = require('node:process');

const NS_PER_SEC = 1e9;
const time = hrtime();
// [ 1800216, 25 ]

setTimeout(() => {
  const diff = hrtime(time);
  // [ 1, 552 ]

  console.log(`Benchmark took ${diff[0] * NS_PER_SEC + diff[1]} nanoseconds`);
  // Benchmark took 1000000552 nanoseconds
}, 1000);
```

## `process.hrtime.bigint()`

<!-- YAML
added: v10.7.0
-->

*   返回：{bigint}

这`bigint`的版本[`process.hrtime()`][process.hrtime()]返回的方法
当前高分辨率实时（以纳秒为单位）作为`bigint`.

与[`process.hrtime()`][process.hrtime()]，则不支持其他`time`
参数，因为差值可以直接计算
通过两者的减法`bigint`s.

```mjs
import { hrtime } from 'node:process';

const start = hrtime.bigint();
// 191051479007711n

setTimeout(() => {
  const end = hrtime.bigint();
  // 191052633396993n

  console.log(`Benchmark took ${end - start} nanoseconds`);
  // Benchmark took 1154389282 nanoseconds
}, 1000);
```

```cjs
const { hrtime } = require('node:process');

const start = hrtime.bigint();
// 191051479007711n

setTimeout(() => {
  const end = hrtime.bigint();
  // 191052633396993n

  console.log(`Benchmark took ${end - start} nanoseconds`);
  // Benchmark took 1154389282 nanoseconds
}, 1000);
```

## `process.initgroups(user, extraGroup)`

<!-- YAML
added: v0.9.4
-->

*   `user`{字符串|数字}用户名或数字标识符。
*   `extraGroup`{字符串|数字}组名或数字标识符。

这`process.initgroups()`方法读取`/etc/group`文件并初始化
组访问列表，使用用户所属的所有组。这是
要求 Node.js 进程具有特权操作`root`
访问或`CAP_SETGID`能力。

删除权限时要小心：

```mjs
import { getgroups, initgroups, setgid } from 'node:process';

console.log(getgroups());         // [ 0 ]
initgroups('nodeuser', 1000);     // switch user
console.log(getgroups());         // [ 27, 30, 46, 1000, 0 ]
setgid(1000);                     // drop root gid
console.log(getgroups());         // [ 27, 30, 46, 1000 ]
```

```cjs
const { getgroups, initgroups, setgid } = require('node:process');

console.log(getgroups());         // [ 0 ]
initgroups('nodeuser', 1000);     // switch user
console.log(getgroups());         // [ 27, 30, 46, 1000, 0 ]
setgid(1000);                     // drop root gid
console.log(getgroups());         // [ 27, 30, 46, 1000 ]
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。
此功能在 中不可用[`Worker`][Worker]线程。

## `process.kill(pid[, signal])`

<!-- YAML
added: v0.0.6
-->

*   `pid`{数字}进程标识
*   `signal`{字符串|数字}要发送的信号，可以是字符串或数字。
    **违约：** `'SIGTERM'`.

这`process.kill()`方法发送`signal`到由
`pid`.

信号名称是字符串，例如`'SIGINT'`或`'SIGHUP'`.看[信号事件][Signal Events]
和 kill（2） 以获取更多信息。

如果目标`pid`不存在。作为特别
案例，信号`0`可用于测试进程是否存在。
Windows 平台将引发错误，如果`pid`用于终止进程
群。

即使此函数的名称是`process.kill()`，它真的只是一个
信号发送器，如`kill`系统调用。发送的信号可能会执行某些操作
除了杀死目标进程。

```mjs
import process, { kill } from 'node:process';

process.on('SIGHUP', () => {
  console.log('Got SIGHUP signal.');
});

setTimeout(() => {
  console.log('Exiting.');
  process.exit(0);
}, 100);

kill(process.pid, 'SIGHUP');
```

```cjs
const process = require('node:process');

process.on('SIGHUP', () => {
  console.log('Got SIGHUP signal.');
});

setTimeout(() => {
  console.log('Exiting.');
  process.exit(0);
}, 100);

process.kill(process.pid, 'SIGHUP');
```

什么时候`SIGUSR1`由节点接收.js进程，节点.js将启动
调试器。看[信号事件][Signal Events].

## `process.mainModule`

<!-- YAML
added: v0.1.17
deprecated: v14.0.0
-->

> 稳定性：0 - 已弃用：使用[`require.main`][require.main]相反。

*   {对象}

这`process.mainModule`属性提供了另一种检索方式
[`require.main`][require.main].不同之处在于，如果主模块在
运行[`require.main`][require.main]仍可参考原主模块
发生更改之前所需的模块。一般来说，它是
可以安全地假设两者指的是同一个模块。

与[`require.main`][require.main],`process.mainModule`将是`undefined`如果有
是无条目脚本。

## `process.memoryUsage()`

<!-- YAML
added: v0.1.16
changes:
  - version:
     - v13.9.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31550
    description: Added `arrayBuffers` to the returned object.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9587
    description: Added `external` to the returned object.
-->

*   返回： {对象}
    *   `rss`{整数}
    *   `heapTotal`{整数}
    *   `heapUsed`{整数}
    *   `external`{整数}
    *   `arrayBuffers`{整数}

返回一个对象，该对象描述节点的内存使用情况.js进程的测量值
字节。

```mjs
import { memoryUsage } from 'node:process';

console.log(memoryUsage());
// Prints:
// {
//  rss: 4935680,
//  heapTotal: 1826816,
//  heapUsed: 650472,
//  external: 49879,
//  arrayBuffers: 9386
// }
```

```cjs
const { memoryUsage } = require('node:process');

console.log(memoryUsage());
// Prints:
// {
//  rss: 4935680,
//  heapTotal: 1826816,
//  heapUsed: 650472,
//  external: 49879,
//  arrayBuffers: 9386
// }
```

*   `heapTotal`和`heapUsed`请参阅 V8 的内存使用情况。
*   `external`指绑定到 JavaScript 的C++对象的内存使用情况
    由 V8 管理的对象。
*   `rss`，驻留集大小，是主要占用的空间量
    的内存设备（即总分配内存的子集）
    进程，包括所有C++和 JavaScript 对象和代码。
*   `arrayBuffers`指分配给 的内存`ArrayBuffer`s 和
    `SharedArrayBuffer`s，包括所有节点.js[`Buffer`][Buffer]s.
    这也包含在`external`价值。当 Node.js 用作
    嵌入式库，此值可能是`0`因为分配`ArrayBuffer`s
    在这种情况下，可能无法跟踪。

使用时[`Worker`][Worker]线程`rss`将是一个对
整个过程，而其他字段将仅引用当前线程。

这`process.memoryUsage()`方法循环访问每个页面以收集
有关内存使用情况的信息，根据
程序内存分配。

## `process.memoryUsage.rss()`

<!-- YAML
added:
  - v15.6.0
  - v14.18.0
-->

*   返回：{整数}

这`process.memoryUsage.rss()`方法返回一个整数，表示
驻留集大小 （RSS），以字节为单位。

驻留集大小，是主图中占用的空间量
的内存设备（即总分配内存的子集）
进程，包括所有C++和 JavaScript 对象和代码。

此值与`rss`属性由`process.memoryUsage()`
但`process.memoryUsage.rss()`更快。

```mjs
import { memoryUsage } from 'node:process';

console.log(memoryUsage.rss());
// 35655680
```

```cjs
const { rss } = require('node:process');

console.log(memoryUsage.rss());
// 35655680
```

## `process.nextTick(callback[, ...args])`

<!-- YAML
added: v0.1.26
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v1.8.1
    pr-url: https://github.com/nodejs/node/pull/1077
    description: Additional arguments after `callback` are now supported.
-->

*   `callback`{函数}
*   `...args`{任何}调用 时要传递的其他参数`callback`

`process.nextTick()`增加`callback`到“下一个价格变动队列”。此队列是
在 JavaScript 堆栈上的当前操作运行到
完成，并在允许事件循环继续之前。有可能
创建一个无限循环，如果一个人要递归调用`process.nextTick()`.
查看[事件循环][Event Loop]更多背景的指南。

```mjs
import { nextTick } from 'node:process';

console.log('start');
nextTick(() => {
  console.log('nextTick callback');
});
console.log('scheduled');
// Output:
// start
// scheduled
// nextTick callback
```

```cjs
const { nextTick } = require('node:process');

console.log('start');
nextTick(() => {
  console.log('nextTick callback');
});
console.log('scheduled');
// Output:
// start
// scheduled
// nextTick callback
```

在开发API时，这一点很重要，以便为用户提供机会
分配事件处理程序*后*对象已被构造，但之前
发生了 I/O：

```mjs
import { nextTick } from 'node:process';

function MyThing(options) {
  this.setupOptions(options);

  nextTick(() => {
    this.startDoingStuff();
  });
}

const thing = new MyThing();
thing.getReadyForStuff();

// thing.startDoingStuff() gets called now, not before.
```

```cjs
const { nextTick } = require('node:process');

function MyThing(options) {
  this.setupOptions(options);

  nextTick(() => {
    this.startDoingStuff();
  });
}

const thing = new MyThing();
thing.getReadyForStuff();

// thing.startDoingStuff() gets called now, not before.
```

对于 API 而言，100% 同步或 100% 非常重要。
异步。请考虑以下示例：

```js
// WARNING!  DO NOT USE!  BAD UNSAFE HAZARD!
function maybeSync(arg, cb) {
  if (arg) {
    cb();
    return;
  }

  fs.stat('file', cb);
}
```

此 API 是危险的，因为在以下情况下：

```js
const maybeTrue = Math.random() > 0.5;

maybeSync(maybeTrue, () => {
  foo();
});

bar();
```

目前尚不清楚是否`foo()`或`bar()`将首先调用。

以下方法要好得多：

```mjs
import { nextTick } from 'node:process';

function definitelyAsync(arg, cb) {
  if (arg) {
    nextTick(cb);
    return;
  }

  fs.stat('file', cb);
}
```

```cjs
const { nextTick } = require('node:process');

function definitelyAsync(arg, cb) {
  if (arg) {
    nextTick(cb);
    return;
  }

  fs.stat('file', cb);
}
```

### 何时使用`queueMicrotask()`与。`process.nextTick()`

这[`queueMicrotask()`][queueMicrotask()]API 是`process.nextTick()`那
还使用与
执行已解析承诺的后续处理程序、捕获处理程序和最后处理程序。在
Node.js，每次“下一个刻度队列”被排空时，微任务队列
立即排干。

```mjs
import { nextTick } from 'node:process';

Promise.resolve().then(() => console.log(2));
queueMicrotask(() => console.log(3));
nextTick(() => console.log(1));
// Output:
// 1
// 2
// 3
```

```cjs
const { nextTick } = require('node:process');

Promise.resolve().then(() => console.log(2));
queueMicrotask(() => console.log(3));
nextTick(() => console.log(1));
// Output:
// 1
// 2
// 3
```

为*最*用户空间用例，`queueMicrotask()`API 提供了一个可移植的
以及用于延迟执行的可靠机制，该机制可跨多个工作
JavaScript平台环境，应该比`process.nextTick()`.
在简单的情况下，`queueMicrotask()`可以直接替代
`process.nextTick()`.

```js
console.log('start');
queueMicrotask(() => {
  console.log('microtask callback');
});
console.log('scheduled');
// Output:
// start
// scheduled
// microtask callback
```

两个 API 之间值得注意的一个区别是`process.nextTick()`
允许指定将作为参数传递给
调用时的延迟函数。实现相同的结果
`queueMicrotask()`需要使用闭包或绑定函数：

```js
function deferred(a, b) {
  console.log('microtask', a + b);
}

console.log('start');
queueMicrotask(deferred.bind(undefined, 1, 2));
console.log('scheduled');
// Output:
// start
// scheduled
// microtask 3
```

在下一个价格变动中引发错误的方式略有不同
队列和微任务队列被处理。排队的微任务中引发的错误
如果可能，应在排队的回调中处理回调。如果他们是
不是，`process.on('uncaughtException')`事件处理程序可用于捕获
并处理错误。

当有疑问时，除非具体能力`process.nextTick()`是
需要，使用`queueMicrotask()`.

## `process.noDeprecation`

<!-- YAML
added: v0.8.0
-->

*   {布尔值}

这`process.noDeprecation`属性指示`--no-deprecation`
标志在当前节点.js进程上设置。请参阅文档
这[`'warning'`事件][process_warning]和
[`emitWarning()`方法][process_emit_warning]有关此内容的更多信息
标志的行为。

## `process.pid`

<!-- YAML
added: v0.1.15
-->

*   {整数}

这`process.pid`属性返回进程的 PID。

```mjs
import { pid } from 'node:process';

console.log(`This process is pid ${pid}`);
```

```cjs
const { pid } = require('node:process');

console.log(`This process is pid ${pid}`);
```

## `process.platform`

<!-- YAML
added: v0.1.16
-->

*   {字符串}

这`process.platform`属性返回标识操作的字符串
为其编译了 Node.js 二进制文件的系统平台。

当前可能的值包括：

*   `'aix'`
*   `'darwin'`
*   `'freebsd'`
*   `'linux'`
*   `'openbsd'`
*   `'sunos'`
*   `'win32'`

```mjs
import { platform } from 'node:process';

console.log(`This platform is ${platform}`);
```

```cjs
const { platform } = require('node:process');

console.log(`This platform is ${platform}`);
```

价值`'android'`如果 Node.js 构建在
安卓操作系统。但是，Android 在 Node 中支持.js
[是实验性的][Android building].

## `process.ppid`

<!-- YAML
added:
  - v9.2.0
  - v8.10.0
  - v6.13.0
-->

*   {整数}

这`process.ppid`属性返回
当前进程。

```mjs
import { ppid } from 'node:process';

console.log(`The parent process is pid ${ppid}`);
```

```cjs
const { ppid } = require('node:process');

console.log(`The parent process is pid ${ppid}`);
```

## `process.release`

<!-- YAML
added: v3.0.0
changes:
  - version: v4.2.0
    pr-url: https://github.com/nodejs/node/pull/3212
    description: The `lts` property is now supported.
-->

*   {对象}

这`process.release`属性返回`Object`包含相关的元数据
到当前版本，包括源压缩包和仅标头的 URL
压缩球。

`process.release`包含以下属性：

*   `name`{字符串}始终为`'node'`.
*   `sourceUrl`{字符串} 指向*`.tar.gz`*文件包含
    当前版本的源代码。
*   `headersUrl`{字符串} 指向*`.tar.gz`*文件包含
    仅当前版本的源头文件。此文件是
    明显小于完整的源文件，可用于编译
    节点.js本机加载项。
*   `libUrl`{字符串} 指向*`node.lib`*与 的文件匹配
    当前版本的体系结构和版本。此文件用于
    编译 Node.js本机加载项。*此属性仅存在于 Windows 上
    Node.js的构建版本，并且在所有其他平台上都将丢失。*
*   `lts`{字符串} 标识[断续器][LTS]标签。
    此属性仅适用于 LTS 版本，并且`undefined`对于所有其他
    发布类型，包括*当前*释放。
    有效值包括 LTS 发行版代码名称（包括那些
    不再受支持）。
    *   `'Dubnium'`对于以 10.13.0 开头的 10.x LTS 行。
    *   `'Erbium'`对于以 12.13.0 开头的 12.x LTS 行。
        有关其他 LTS 版本代码名称，请参阅[节点.js更新日志存档](https://github.com/nodejs/node/blob/HEAD/doc/changelogs/CHANGELOG_ARCHIVE.md)

<!-- eslint-skip -->

```js
{
  name: 'node',
  lts: 'Erbium',
  sourceUrl: 'https://nodejs.org/download/release/v12.18.1/node-v12.18.1.tar.gz',
  headersUrl: 'https://nodejs.org/download/release/v12.18.1/node-v12.18.1-headers.tar.gz',
  libUrl: 'https://nodejs.org/download/release/v12.18.1/win-x64/node.lib'
}
```

在从源代码树的非发布版本的自定义版本中，只有
`name`属性可能存在。附加属性不应为
依靠存在。

## `process.report`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   {对象}

`process.report`是其方法用于生成诊断的对象
当前进程的报告。其他文档可在
[报告文档][report documentation].

### `process.report.compact`

<!-- YAML
added:
 - v13.12.0
 - v12.17.0
-->

*   {布尔值}

以紧凑的格式编写报告，单行 JSON，更易于使用
通过日志处理系统比默认的多行格式设计
人类消费。

```mjs
import { report } from 'node:process';

console.log(`Reports are compact? ${report.compact}`);
```

```cjs
const { report } = require('node:process');

console.log(`Reports are compact? ${report.compact}`);
```

### `process.report.directory`

<!-- YAML
added: v11.12.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   {字符串}

写入报表的目录。默认值为空字符串，
指示报告已写入
节点.js进程。

```mjs
import { report } from 'node:process';

console.log(`Report directory is ${report.directory}`);
```

```cjs
const { report } = require('node:process');

console.log(`Report directory is ${report.directory}`);
```

### `process.report.filename`

<!-- YAML
added: v11.12.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   {字符串}

写入报表的文件名。如果设置为空字符串，则输出
文件名将由时间戳、PID 和序列号组成。默认值
值是空字符串。

```mjs
import { report } from 'node:process';

console.log(`Report filename is ${report.filename}`);
```

```cjs
const { report } = require('node:process');

console.log(`Report filename is ${report.filename}`);
```

### `process.report.getReport([err])`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   `err`{错误}用于报告 JavaScript 堆栈的自定义错误。
*   返回： {对象}

返回 的诊断报告的 JavaScript 对象表示形式
正在运行的进程。该报告的 JavaScript 堆栈跟踪取自`err`如果
目前。

```mjs
import { report } from 'node:process';

const data = report.getReport();
console.log(data.header.nodejsVersion);

// Similar to process.report.writeReport()
import fs from 'node:fs';
fs.writeFileSync('my-report.log', util.inspect(data), 'utf8');
```

```cjs
const { report } = require('node:process');

const data = report.getReport();
console.log(data.header.nodejsVersion);

// Similar to process.report.writeReport()
const fs = require('node:fs');
fs.writeFileSync('my-report.log', util.inspect(data), 'utf8');
```

其他文档可在[报告文档][report documentation].

### `process.report.reportOnFatalError`

<!-- YAML
added: v11.12.0
changes:
  - version:
     - v15.0.0
     - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/35654
    description: This API is no longer experimental.
-->

*   {布尔值}

如果`true`，则会在出现致命错误时生成诊断报告，例如
内存错误或C++断言失败。

```mjs
import { report } from 'node:process';

console.log(`Report on fatal error: ${report.reportOnFatalError}`);
```

```cjs
const { report } = require('node:process');

console.log(`Report on fatal error: ${report.reportOnFatalError}`);
```

### `process.report.reportOnSignal`

<!-- YAML
added: v11.12.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   {布尔值}

如果`true`，则当进程收到
信号由`process.report.signal`.

```mjs
import { report } from 'node:process';

console.log(`Report on signal: ${report.reportOnSignal}`);
```

```cjs
const { report } = require('node:process');

console.log(`Report on signal: ${report.reportOnSignal}`);
```

### `process.report.reportOnUncaughtException`

<!-- YAML
added: v11.12.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   {布尔值}

如果`true`，则在未捕获异常时生成诊断报告。

```mjs
import { report } from 'node:process';

console.log(`Report on exception: ${report.reportOnUncaughtException}`);
```

```cjs
const { report } = require('node:process');

console.log(`Report on exception: ${report.reportOnUncaughtException}`);
```

### `process.report.signal`

<!-- YAML
added: v11.12.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   {字符串}

用于触发诊断报告创建的信号。默认值为
`'SIGUSR2'`.

```mjs
import { report } from 'node:process';

console.log(`Report signal: ${report.signal}`);
```

```cjs
const { report } = require('node:process');

console.log(`Report signal: ${report.signal}`);
```

### `process.report.writeReport([filename][, err])`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This API is no longer experimental.
-->

*   `filename`{字符串}写入报表的文件的名称。这
    应该是一个相对路径，该路径将追加到
    `process.report.directory`或节点的当前工作目录.js
    进程（如果未指定）。

*   `err`{错误}用于报告 JavaScript 堆栈的自定义错误。

*   返回：{string} 返回生成的报告的文件名。

将诊断报告写入文件。如果`filename`未提供，默认
文件名包括日期、时间、PID 和序列号。报告的
JavaScript 堆栈跟踪取自`err`，如果存在。

```mjs
import { report } from 'node:process';

report.writeReport();
```

```cjs
const { report } = require('node:process');

report.writeReport();
```

其他文档可在[报告文档][report documentation].

## `process.resourceUsage()`

<!-- YAML
added: v12.6.0
-->

*   返回：{对象} 当前进程的资源使用情况。所有这些
    值来自`uv_getrusage`调用哪个返回
    一个[`uv_rusage_t`结构][uv_rusage_t].
    *   `userCPUTime`{整数} 映射到`ru_utime`以微秒为单位计算。
        它与[`process.cpuUsage().user`][process.cpuUsage].
    *   `systemCPUTime`{整数} 映射到`ru_stime`以微秒为单位计算。
        它与[`process.cpuUsage().system`][process.cpuUsage].
    *   `maxRSS`{整数} 映射到`ru_maxrss`这是最大常驻集
        大小（以千字节为单位）。
    *   `sharedMemorySize`{整数} 映射到`ru_ixrss`但不受 支持
        任何平台。
    *   `unsharedDataSize`{整数} 映射到`ru_idrss`但不受 支持
        任何平台。
    *   `unsharedStackSize`{整数} 映射到`ru_isrss`但不受 支持
        任何平台。
    *   `minorPageFault`{整数} 映射到`ru_minflt`这是数量
        该过程的次要页面错误，请参阅
        [本文了解更多详情][wikipedia_minor_fault].
    *   `majorPageFault`{整数} 映射到`ru_majflt`这是数量
        该过程的主要页面错误，请参阅
        [本文了解更多详情][wikipedia_major_fault].此字段不是
        在 Windows 上受支持。
    *   `swappedOut`{整数} 映射到`ru_nswap`但不受任何支持
        平台。
    *   `fsRead`{整数} 映射到`ru_inblock`这是
        文件系统必须执行输入。
    *   `fsWrite`{整数} 映射到`ru_oublock`这是
        文件系统必须执行输出。
    *   `ipcSent`{整数} 映射到`ru_msgsnd`但不受任何支持
        平台。
    *   `ipcReceived`{整数} 映射到`ru_msgrcv`但不受任何支持
        平台。
    *   `signalsCount`{整数} 映射到`ru_nsignals`但不受任何支持
        平台。
    *   `voluntaryContextSwitches`{整数} 映射到`ru_nvcsw`这是
        由于进程自愿导致 CPU 上下文切换的次数
        在处理器的时间片完成之前放弃处理器（通常
        等待资源的可用性）。此字段在 Windows 上不受支持。
    *   `involuntaryContextSwitches`{整数} 映射到`ru_nivcsw`这是
        由于优先级较高而导致 CPU 上下文切换的次数
        进程变得可运行或因为当前进程超过其
        时间片。此字段在 Windows 上不受支持。

```mjs
import { resourceUsage } from 'node:process';

console.log(resourceUsage());
/*
  Will output:
  {
    userCPUTime: 82872,
    systemCPUTime: 4143,
    maxRSS: 33164,
    sharedMemorySize: 0,
    unsharedDataSize: 0,
    unsharedStackSize: 0,
    minorPageFault: 2469,
    majorPageFault: 0,
    swappedOut: 0,
    fsRead: 0,
    fsWrite: 8,
    ipcSent: 0,
    ipcReceived: 0,
    signalsCount: 0,
    voluntaryContextSwitches: 79,
    involuntaryContextSwitches: 1
  }
*/
```

```cjs
const { resourceUsage } = require('node:process');

console.log(resourceUsage());
/*
  Will output:
  {
    userCPUTime: 82872,
    systemCPUTime: 4143,
    maxRSS: 33164,
    sharedMemorySize: 0,
    unsharedDataSize: 0,
    unsharedStackSize: 0,
    minorPageFault: 2469,
    majorPageFault: 0,
    swappedOut: 0,
    fsRead: 0,
    fsWrite: 8,
    ipcSent: 0,
    ipcReceived: 0,
    signalsCount: 0,
    voluntaryContextSwitches: 79,
    involuntaryContextSwitches: 1
  }
*/
```

## `process.send(message[, sendHandle[, options]][, callback])`

<!-- YAML
added: v0.5.9
-->

*   `message`{对象}
*   `sendHandle`{网.服务器|网。插座}
*   `options`{对象} 用于参数化某些类型的
    处理。`options`支持以下属性：
    *   `keepOpen`{布尔值}传递 实例时可以使用的值
        `net.Socket`.什么时候`true`，则套接字在发送过程中保持打开状态。
        **违约：** `false`.
*   `callback`{函数}
*   返回：{布尔值}

如果使用 IPC 通道生成 Node.js，则`process.send()`方法可以是
用于将消息发送到父进程。消息将作为
[`'message'`]['message']父母的事件[`ChildProcess`][ChildProcess]对象。

如果节点.js未通过 IPC 通道生成，`process.send`将是
`undefined`.

消息经过序列化和分析。生成的消息可能
与最初发送的内容不同。

## `process.setegid(id)`

<!-- YAML
added: v2.0.0
-->

*   `id`{字符串|数字}组名称或 ID

这`process.setegid()`方法设置进程的有效组标识。
（见 setegid（2）。这`id`可以作为数字 ID 或组传递
名称字符串。如果指定了组名，则此方法在解析时会阻塞
关联了一个数字 ID。

```mjs
import process from 'node:process';

if (process.getegid && process.setegid) {
  console.log(`Current gid: ${process.getegid()}`);
  try {
    process.setegid(501);
    console.log(`New gid: ${process.getegid()}`);
  } catch (err) {
    console.log(`Failed to set gid: ${err}`);
  }
}
```

```cjs
const process = require('node:process');

if (process.getegid && process.setegid) {
  console.log(`Current gid: ${process.getegid()}`);
  try {
    process.setegid(501);
    console.log(`New gid: ${process.getegid()}`);
  } catch (err) {
    console.log(`Failed to set gid: ${err}`);
  }
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。
此功能在 中不可用[`Worker`][Worker]线程。

## `process.seteuid(id)`

<!-- YAML
added: v2.0.0
-->

*   `id`{字符串|数字}用户名或 ID

这`process.seteuid()`方法设置进程的有效用户标识。
（参见 seteuid（2）。这`id`可以作为数字 ID 或用户名传递
字符串。如果指定了用户名，则在解析
关联的数字 ID。

```mjs
import process from 'node:process';

if (process.geteuid && process.seteuid) {
  console.log(`Current uid: ${process.geteuid()}`);
  try {
    process.seteuid(501);
    console.log(`New uid: ${process.geteuid()}`);
  } catch (err) {
    console.log(`Failed to set uid: ${err}`);
  }
}
```

```cjs
const process = require('node:process');

if (process.geteuid && process.seteuid) {
  console.log(`Current uid: ${process.geteuid()}`);
  try {
    process.seteuid(501);
    console.log(`New uid: ${process.geteuid()}`);
  } catch (err) {
    console.log(`Failed to set uid: ${err}`);
  }
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。
此功能在 中不可用[`Worker`][Worker]线程。

## `process.setgid(id)`

<!-- YAML
added: v0.1.31
-->

*   `id`{字符串|数字}组名称或 ID

这`process.setgid()`方法设置进程的组标识。（请参见
setgid（2）.）这`id`可以作为数字 ID 或组名传递
字符串。如果指定了组名，则此方法会在解析
关联的数字 ID。

```mjs
import process from 'node:process';

if (process.getgid && process.setgid) {
  console.log(`Current gid: ${process.getgid()}`);
  try {
    process.setgid(501);
    console.log(`New gid: ${process.getgid()}`);
  } catch (err) {
    console.log(`Failed to set gid: ${err}`);
  }
}
```

```cjs
const process = require('node:process');

if (process.getgid && process.setgid) {
  console.log(`Current gid: ${process.getgid()}`);
  try {
    process.setgid(501);
    console.log(`New gid: ${process.getgid()}`);
  } catch (err) {
    console.log(`Failed to set gid: ${err}`);
  }
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。
此功能在 中不可用[`Worker`][Worker]线程。

## `process.setgroups(groups)`

<!-- YAML
added: v0.9.4
-->

*   `groups`{整数\[]}

这`process.setgroups()`方法设置
节点.js进程。这是一个需要 Node 的特权操作.js
过程有`root`或`CAP_SETGID`能力。

这`groups`数组可以包含数字组 ID 和/或组名。

```mjs
import process from 'node:process';

if (process.getgroups && process.setgroups) {
  try {
    process.setgroups([501]);
    console.log(process.getgroups()); // new groups
  } catch (err) {
    console.log(`Failed to set groups: ${err}`);
  }
}
```

```cjs
const process = require('node:process');

if (process.getgroups && process.setgroups) {
  try {
    process.setgroups([501]);
    console.log(process.getgroups()); // new groups
  } catch (err) {
    console.log(`Failed to set groups: ${err}`);
  }
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。
此功能在 中不可用[`Worker`][Worker]线程。

## `process.setuid(id)`

<!-- YAML
added: v0.1.28
-->

*   `id`{整数|字符串}

这`process.setuid(id)`方法设置进程的用户标识。（请参见
setuid（2）.）这`id`可以作为数字 ID 或用户名字符串传递。
如果指定了用户名，则在解析关联的
数字 ID。

```mjs
import process from 'node:process';

if (process.getuid && process.setuid) {
  console.log(`Current uid: ${process.getuid()}`);
  try {
    process.setuid(501);
    console.log(`New uid: ${process.getuid()}`);
  } catch (err) {
    console.log(`Failed to set uid: ${err}`);
  }
}
```

```cjs
const process = require('node:process');

if (process.getuid && process.setuid) {
  console.log(`Current uid: ${process.getuid()}`);
  try {
    process.setuid(501);
    console.log(`New uid: ${process.getuid()}`);
  } catch (err) {
    console.log(`Failed to set uid: ${err}`);
  }
}
```

此功能仅在POSIX平台上可用（即不适用于Windows或
安卓）。
此功能在 中不可用[`Worker`][Worker]线程。

## `process.setSourceMapsEnabled(val)`

<!-- YAML
added:
  - v16.6.0
  - v14.18.0
-->

> 稳定性： 1 - 实验

*   `val`{布尔值}

此功能启用或禁用[源地图 v3][Source Map]支持
堆栈跟踪。

它提供了与启动 Node.js进程相同的功能，具有命令行选项
`--enable-source-maps`.

只有 JavaScript 文件中的源映射在源映射之后加载
启用将被解析和加载。

## `process.setUncaughtExceptionCaptureCallback(fn)`

<!-- YAML
added: v9.3.0
-->

*   `fn`{函数|空}

这`process.setUncaughtExceptionCaptureCallback()`函数设置函数
当发生未捕获的异常时将调用该异常，这将收到
异常值本身作为其第一个参数。

如果设置了这样的函数，则[`'uncaughtException'`]['uncaughtException']事件将
不发出。如果`--abort-on-uncaught-exception`从
命令行或设置通过[`v8.setFlagsFromString()`][v8.setFlagsFromString()]，该过程将
不中止。配置为对异常（如报告）执行的操作
几代人也会受到影响

要取消设置捕获功能，
`process.setUncaughtExceptionCaptureCallback(null)`可以使用。调用此
具有非 - 的方法`null`参数，而设置另一个捕获函数时将
引发错误。

使用此函数与使用已弃用的函数是互斥的
[`domain`][domain]内置模块。

## `process.stderr`

*   {流}

这`process.stderr`属性返回连接到
`stderr`（fd`2`).这是一个[`net.Socket`][net.Socket]（这是一个[双工][Duplex]
流），除非 fd`2`引用文件，在这种情况下，它是
一个[写][Writable]流。

`process.stderr`与其他 Node.js 流在重要方面有所不同。看
[关于进程 I/O 的说明][note on process I/O]了解更多信息。

### `process.stderr.fd`

*   {数字}

此属性引用基础文件描述符的值
`process.stderr`.该值固定在`2`.在[`Worker`][Worker]线程
此字段不存在。

## `process.stdin`

*   {流}

这`process.stdin`属性返回连接到
`stdin`（fd`0`).这是一个[`net.Socket`][net.Socket]（这是一个[双工][Duplex]
流），除非 fd`0`引用文件，在这种情况下，它是
一个[读][Readable]流。

有关如何阅读的详细信息`stdin`看[`readable.read()`][readable.read()].

作为[双工][Duplex]流`process.stdin`也可以在“旧”模式下使用
与 v0.10 之前为 Node.js 编写的脚本兼容。
有关详细信息，请参阅[流兼容性][Stream compatibility].

在“旧”流模式中`stdin`默认情况已暂停，因此有一个
必须致电`process.stdin.resume()`从中读取。另请注意，呼叫
`process.stdin.resume()`本身会将流切换到“旧”模式。

### `process.stdin.fd`

*   {数字}

此属性引用基础文件描述符的值
`process.stdin`.该值固定在`0`.在[`Worker`][Worker]线程
此字段不存在。

## `process.stdout`

*   {流}

这`process.stdout`属性返回连接到
`stdout`（fd`1`).这是一个[`net.Socket`][net.Socket]（这是一个[双工][Duplex]
流），除非 fd`1`引用文件，在这种情况下，它是
一个[写][Writable]流。

例如，复制`process.stdin`自`process.stdout`:

```mjs
import { stdin, stdout } from 'node:process';

stdin.pipe(stdout);
```

```cjs
const { stdin, stdout } = require('node:process');

stdin.pipe(stdout);
```

`process.stdout`与其他 Node.js 流在重要方面有所不同。看
[关于进程 I/O 的说明][note on process I/O]了解更多信息。

### `process.stdout.fd`

*   {数字}

此属性引用基础文件描述符的值
`process.stdout`.该值固定在`1`.在[`Worker`][Worker]线程
此字段不存在。

### 关于进程 I/O 的说明

`process.stdout`和`process.stderr`不同于 .js 中的其他节点.js流
重要方式：

1.  它们在内部由[`console.log()`][console.log()]和[`console.error()`][console.error()],
    分别。
2.  写入可能是同步的，具体取决于流连接到的内容
    以及系统是Windows还是POSIX：
    *   文件：*同步*在 Windows 和 POSIX 上
    *   终端：*异步*在 Windows 上，*同步*在 POSIX 上
    *   管道（和插座）：*同步*在 Windows 上，*异步*在 POSIX 上

这些行为部分是出于历史原因，因为改变它们会
造成向后不兼容，但一些用户也期望它们。

同步写入避免了诸如使用`console.log()`或
`console.error()`意外交错，或者根本不写，如果
`process.exit()`在异步写入完成之前调用。看
[`process.exit()`][process.exit()]了解更多信息。

***警告***：同步写入阻塞事件循环，直到写入
完成。在输出到文件的情况下，这可以几乎是即时的，但是
在高系统负载下，接收端未读取的管道，或
对于慢速终端或文件系统，事件循环可能是
足够频繁地阻塞，足够长的时间，以产生严重的负面性能
影响。写入交互式终端时，这可能不是问题
会话，但在执行生产日志记录时要特别小心这一点
过程输出流。

要检查流是否已连接到[断续器][TTY]上下文中，请检查`isTTY`
财产。

例如：

```console
$ node -p "Boolean(process.stdin.isTTY)"
true
$ echo "foo" | node -p "Boolean(process.stdin.isTTY)"
false
$ node -p "Boolean(process.stdout.isTTY)"
true
$ node -p "Boolean(process.stdout.isTTY)" | cat
false
```

查看[断续器][TTY]文档以获取更多信息。

## `process.throwDeprecation`

<!-- YAML
added: v0.9.12
-->

*   {布尔值}

的初始值`process.throwDeprecation`指示
`--throw-deprecation`标志在当前节点.js进程上设置。
`process.throwDeprecation`是可变的，所以是否弃用
警告导致错误可能会在运行时更改。查看
的文档[`'warning'`事件][process_warning]和
[`emitWarning()`方法][process_emit_warning]了解更多信息。

```console
$ node --throw-deprecation -p "process.throwDeprecation"
true
$ node -p "process.throwDeprecation"
undefined
$ node
> process.emitWarning('test', 'DeprecationWarning');
undefined
> (node:26598) DeprecationWarning: test
> process.throwDeprecation = true;
true
> process.emitWarning('test', 'DeprecationWarning');
Thrown:
[DeprecationWarning: test] { name: 'DeprecationWarning' }
```

## `process.title`

<!-- YAML
added: v0.1.104
-->

*   {字符串}

这`process.title`属性返回当前进程标题（即返回
的当前值`ps`).将新值分配给`process.title`修改
的当前值`ps`.

当分配新值时，不同的平台将施加不同的最大值
标题的长度限制。通常，此类限制非常有限。
例如，在Linux和macOS上，`process.title`被限制在
二进制名称加上命令行参数的长度，因为将
`process.title`覆盖`argv`进程的内存。节点.js v0.8
允许较长的进程标题字符串，同时覆盖`environ`
记忆，但在某些地方可能是不安全和令人困惑的（相当晦涩的）
例。

将值赋给`process.title`可能无法生成准确的标签
在进程管理器应用程序（如 macOS Activity Monitor 或 Windows）中
服务经理。

## `process.traceDeprecation`

<!-- YAML
added: v0.8.0
-->

*   {布尔值}

这`process.traceDeprecation`属性指示
`--trace-deprecation`标志在当前节点.js进程上设置。查看
的文档[`'warning'`事件][process_warning]和
[`emitWarning()`方法][process_emit_warning]有关此内容的更多信息
标志的行为。

## `process.umask()`

<!-- YAML
added: v0.1.19
changes:
  - version:
    - v14.0.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/32499
    description: Calling `process.umask()` with no arguments is deprecated.

-->

> 稳定性：0 - 已弃用。叫`process.umask()`没有参数原因
> 要写入两次的进程范围的掩码。这引入了争用条件
> 在线程之间，并且是一个潜在的安全漏洞。没有保险箱，
> 跨平台替代 API。

`process.umask()`返回 Node.js进程的文件模式创建掩码。孩子
进程从父进程继承掩码。

## `process.umask(mask)`

<!-- YAML
added: v0.1.19
-->

*   `mask`{string|integer}

`process.umask(mask)`设置 Node.js 进程的文件模式创建掩码。孩子
进程从父进程继承掩码。返回上一个掩码。

```mjs
import { umask } from 'node:process';

const newmask = 0o022;
const oldmask = umask(newmask);
console.log(
  `Changed umask from ${oldmask.toString(8)} to ${newmask.toString(8)}`
);
```

```cjs
const { umask } = require('node:process');

const newmask = 0o022;
const oldmask = umask(newmask);
console.log(
  `Changed umask from ${oldmask.toString(8)} to ${newmask.toString(8)}`
);
```

在[`Worker`][Worker]线程`process.umask(mask)`将引发异常。

## `process.uptime()`

<!-- YAML
added: v0.5.0
-->

*   返回值：{数字}

这`process.uptime()`方法返回当前节点的秒数.js
进程已运行。

返回值包括秒的几分之一部分。用`Math.floor()`获得完整
秒。

## `process.version`

<!-- YAML
added: v0.1.3
-->

*   {字符串}

这`process.version`属性包含节点.js版本字符串。

```mjs
import { version } from 'node:process';

console.log(`Version: ${version}`);
// Version: v14.8.0
```

```cjs
const { version } = require('node:process');

console.log(`Version: ${version}`);
// Version: v14.8.0
```

获取不带前缀的版本字符串*v*用
`process.versions.node`.

## `process.versions`

<!-- YAML
added: v0.2.0
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15785
    description: The `v8` property now includes a Node.js specific suffix.
  - version: v4.2.0
    pr-url: https://github.com/nodejs/node/pull/3102
    description: The `icu` property is now supported.
-->

*   {对象}

这`process.versions`属性返回一个对象，其中列出了
节点.js及其依赖项。`process.versions.modules`表示当前
ABI 版本，每当C++ API 更改时，该版本就会增加。节点.js将拒绝
以加载针对其他模块 ABI 版本编译的模块。

```mjs
import { versions } from 'node:process';

console.log(versions);
```

```cjs
const { versions } = require('node:process');

console.log(versions);
```

将生成一个类似于以下内容的对象：

```console
{ node: '11.13.0',
  v8: '7.0.276.38-node.18',
  uv: '1.27.0',
  zlib: '1.2.11',
  brotli: '1.0.7',
  ares: '1.15.0',
  modules: '67',
  nghttp2: '1.34.0',
  napi: '4',
  llhttp: '1.1.1',
  openssl: '1.1.1b',
  cldr: '34.0',
  icu: '63.1',
  tz: '2018e',
  unicode: '11.0' }
```

## 退出代码

节点.js通常会以`0`不再异步时的状态代码
操作处于挂起状态。以下状态代码用于其他
例：

*   `1` **未捕获的致命异常**：有一个未被发现的例外，
    并且它不是由域或[`'uncaughtException'`]['uncaughtException']事件
    处理器。
*   `2`：未使用（由 Bash 保留用于内置误用）
*   `3` **内部 JavaScript 解析错误**：JavaScript 源代码
    节点内部.js引导过程导致解析错误。这
    极为罕见，一般只能在开发过程中发生
    节点.js本身。
*   `4` **内部 JavaScript 评估失败**： JavaScript
    Node 内部源代码.js引导过程失败
    计算时返回函数值。这是非常罕见的，而且
    通常只能在Node.js本身的开发过程中发生。
*   `5` **致命错误**：V8 中存在致命的不可恢复错误。
    通常，一条消息将被打印到带有前缀的 stderr`FATAL
    ERROR`.
*   `6` **非函数内部异常处理程序**： 有一个
    未捕获的异常，但内部致命异常处理程序
    函数以某种方式设置为非函数，并且无法调用。
*   `7` **内部异常处理程序运行时故障**： 有一个
    未捕获的异常，以及内部致命异常处理程序
    函数本身在尝试处理错误时抛出错误。这
    例如，如果[`'uncaughtException'`]['uncaughtException']或
    `domain.on('error')`处理程序引发错误。
*   `8`：未使用。在以前版本的 Node.js 中，有时退出代码 8
    表示未捕获的异常。
*   `9` **无效参数**：指定了未知选项，
    或者提供了一个需要值的选项，但没有一个值。
*   `10` **内部 JavaScript 运行时故障**： JavaScript
    Node 内部源代码.js引导过程引发错误
    调用引导函数的时间。这是非常罕见的，
    并且通常只能在Node.js本身的开发过程中发生。
*   `12` **无效的调试参数**：`--inspect`和/或`--inspect-brk`
    设置了选项，但所选的端口号无效或不可用。
*   `13` **未完成的顶层等待**:`await`在函数外部使用
    在顶级代码中，但已通过`Promise`从未解决。
*   `14` **快照失败**：节点.js开始构建V8启动
    快照和它失败了，因为某些要求的状态
    未满足申请。
*   `>128` **信号出口**：如果 Node.js收到致命信号，例如
    `SIGKILL`或`SIGHUP`，则其退出代码将为`128`加上
    信号代码的值。这是一种标准的 POSIX 实践，因为
    退出代码定义为 7 位整数，并设置信号退出
    高阶位，然后包含信号码的值。
    例如，信号`SIGABRT`具有价值`6`，因此预期退出
    代码将是`128`+`6`或`134`.

[Advanced serialization for `child_process`]: child_process.md#advanced-serialization

[Android building]: https://github.com/nodejs/node/blob/HEAD/BUILDING.md#androidandroid-based-devices-eg-firefox-os

[Child Process]: child_process.md

[Cluster]: cluster.md

[Duplex]: stream.md#duplex-and-transform-streams

[Event Loop]: https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#process-nexttick

[LTS]: https://github.com/nodejs/Release

[Readable]: stream.md#readable-streams

[Signal Events]: #signal-events

[Source Map]: https://sourcemaps.info/spec.html

[Stream compatibility]: stream.md#compatibility-with-older-nodejs-versions

[TTY]: tty.md#tty

[Writable]: stream.md#writable-streams

[`'exit'`]: #event-exit

[`'message'`]: child_process.md#event-message

[`'uncaughtException'`]: #event-uncaughtexception

[`--unhandled-rejections`]: cli.md#--unhandled-rejectionsmode

[`Buffer`]: buffer.md

[`ChildProcess.disconnect()`]: child_process.md#subprocessdisconnect

[`ChildProcess.send()`]: child_process.md#subprocesssendmessage-sendhandle-options-callback

[`ChildProcess`]: child_process.md#class-childprocess

[`Error`]: errors.md#class-error

[`EventEmitter`]: events.md#class-eventemitter

[`NODE_OPTIONS`]: cli.md#node_optionsoptions

[`Promise.race()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race

[`Worker`]: worker_threads.md#class-worker

[`Worker` constructor]: worker_threads.md#new-workerfilename-options

[`console.error()`]: console.md#consoleerrordata-args

[`console.log()`]: console.md#consolelogdata-args

[`domain`]: domain.md

[`net.Server`]: net.md#class-netserver

[`net.Socket`]: net.md#class-netsocket

[`os.constants.dlopen`]: os.md#dlopen-constants

[`process.argv`]: #processargv

[`process.config`]: #processconfig

[`process.execPath`]: #processexecpath

[`process.exit()`]: #processexitcode

[`process.exitCode`]: #processexitcode

[`process.hrtime()`]: #processhrtimetime

[`process.hrtime.bigint()`]: #processhrtimebigint

[`process.kill()`]: #processkillpid-signal

[`process.setUncaughtExceptionCaptureCallback()`]: #processsetuncaughtexceptioncapturecallbackfn

[`promise.catch()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch

[`queueMicrotask()`]: globals.md#queuemicrotaskcallback

[`readable.read()`]: stream.md#readablereadsize

[`require()`]: globals.md#require

[`require.main`]: modules.md#accessing-the-main-module

[`subprocess.kill()`]: child_process.md#subprocesskillsignal

[`v8.setFlagsFromString()`]: v8.md#v8setflagsfromstringflags

[debugger]: debugger.md

[deprecation code]: deprecations.md

[note on process I/O]: #a-note-on-process-io

[process.cpuUsage]: #processcpuusagepreviousvalue

[process_emit_warning]: #processemitwarningwarning-type-code-ctor

[process_warning]: #event-warning

[report documentation]: report.md

[terminal raw mode]: tty.md#readstreamsetrawmodemode

[uv_rusage_t]: https://docs.libuv.org/en/v1.x/misc.html#c.uv_rusage_t

[wikipedia_major_fault]: https://en.wikipedia.org/wiki/Page_fault#Major

[wikipedia_minor_fault]: https://en.wikipedia.org/wiki/Page_fault#Minor
