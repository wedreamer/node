# 已弃用的 API

<!--introduced_in=v7.7.0-->

<!-- type=misc -->

节点.js API 可能由于以下任一原因而被弃用：

*   使用 API 是不安全的。
*   提供了改进的替代 API。
*   预计在未来的主要版本中将对 API 进行重大更改。

Node.js 使用三种类型的 Deprecation：

*   仅文档
*   运行
*   生命周期结束

仅文档弃用是指仅在
Node.js API 文档。这些在运行Node.js时不会产生副作用。
某些仅文档的弃用在启动时会触发运行时警告
跟[`--pending-deprecation`][--pending-deprecation]标志（或其替代方案，
`NODE_PENDING_DEPRECATION=1`环境变量），类似于运行时
下面的弃用。支持该标志的仅文档弃用
在
[已弃用的 API 列表](#list-of-deprecated-apis).

默认情况下，运行时弃用将生成进程警告，该警告将
打印到`stderr`首次使用已弃用的 API 时。当
[`--throw-deprecation`][--throw-deprecation]使用命令行标志，运行时弃用将
导致引发错误。

当功能被删除或即将被删除时，将使用生命周期终止弃用
从节点.js。

## 撤销弃用

有时，API 的弃用可能会被逆转。在这种情况下，
本文件将更新与决定有关的信息。
但是，不会修改弃用标识符。

## 已弃用的 API 列表

### DEP0001：`http.OutgoingMessage.prototype.flush`

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31164
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v1.6.0
    pr-url: https://github.com/nodejs/node/pull/1156
    description: Runtime deprecation.
-->

类型：报废

`OutgoingMessage.prototype.flush()`已被删除。用
`OutgoingMessage.prototype.flushHeaders()`相反。

### DEP0002：`require('_linklist')`

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12113
    description: End-of-Life.
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3078
    description: Runtime deprecation.
-->

类型：报废

这`_linklist`模块已弃用。请使用用户空间替代方案。

### DEP0003：`_writableState.buffer`

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31165
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.15
    pr-url: https://github.com/nodejs/node-v0.x-archive/pull/8826
    description: Runtime deprecation.
-->

类型：报废

这`_writableState.buffer`已被删除。用`_writableState.getBuffer()`
相反。

### DEP0004：`CryptoStream.prototype.readyState`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17882
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.4.0
    commit: 9c7f89bf56abd37a796fea621ad2e47dd33d2b82
    description: Documentation-only deprecation.
-->

类型：报废

这`CryptoStream.prototype.readyState`属性已被删除。

### DEP0005：`Buffer()`构造 函数

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19524
    description: Runtime deprecation.
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4682
    description: Documentation-only deprecation.
-->

类型：运行时（支持[`--pending-deprecation`][--pending-deprecation])

这`Buffer()`功能和`new Buffer()`构造函数由于
可能导致意外安全问题的 API 可用性问题。

或者，使用以下构造方法之一`Buffer`
对象：

*   [`Buffer.alloc(size[, fill[, encoding]])`][alloc]：创建一个`Buffer`跟
    *初始 化*记忆。
*   [`Buffer.allocUnsafe(size)`][alloc_unsafe_size]：创建一个`Buffer`跟
    *初始 化*记忆。
*   [`Buffer.allocUnsafeSlow(size)`][Buffer.allocUnsafeSlow(size)]：创建一个`Buffer`跟*初始 化*
    记忆。
*   [`Buffer.from(array)`][Buffer.from(array)]：创建一个`Buffer`与副本`array`
*   [`Buffer.from(arrayBuffer[, byteOffset[, length]])`][from_arraybuffer]-
    创建一个`Buffer`包装给定的`arrayBuffer`.
*   [`Buffer.from(buffer)`][Buffer.from(buffer)]：创建一个`Buffer`复制`buffer`.
*   [`Buffer.from(string[, encoding])`][from_string_encoding]：创建一个`Buffer`
    复制`string`.

没有`--pending-deprecation`，运行时警告仅对不在 中的代码发生
`node_modules`.这意味着 不会有弃用警告
`Buffer()`在依赖项中的用法。跟`--pending-deprecation`，运行时
无论在何处，都会出现警告结果`Buffer()`发生使用。

### DEP0006：`child_process` `options.customFds`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25279
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.14
    description: Runtime deprecation.
  - version: v0.5.10
    description: Documentation-only deprecation.
-->

类型：报废

在[`child_process`][child_process]模块的`spawn()`,`fork()`和`exec()`
方法，`options.customFds`选项已弃用。这`options.stdio`
应改用选项。

### DEP0007： 替换`cluster` `worker.suicide`跟`worker.exitedAfterDisconnect`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/13702
    description: End-of-Life.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/3747
    description: Runtime deprecation.
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/3743
    description: Documentation-only deprecation.
-->

类型：报废

在节点的早期版本中.js`cluster`，具有名称的布尔属性
`suicide`已添加到`Worker`对象。该物业的意图是
提供有关如何以及为何的指示`Worker`实例已退出。在节点中.js
在 6.0.0 中，旧属性被弃用并替换为新的
[`worker.exitedAfterDisconnect`][worker.exitedAfterDisconnect]财产。旧物业名称没有
精确地描述了实际的语义，并且不必要地充满了情感。

### DEP0008：`require('node:constants')`

<!-- YAML
changes:
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6534
    description: Documentation-only deprecation.
-->

类型：仅文档

这`node:constants`模块已弃用。当需要访问常量时
与特定 Node 相关.js内置模块，开发人员应改为参考
到`constants`属性由相关模块公开。例如
`require('node:fs').constants`和`require('node:os').constants`.

### DEP0009：`crypto.pbkdf2`没有消化

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31166
    description: End-of-Life (for `digest === null`).
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22861
    description: Runtime deprecation (for `digest === null`).
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11305
    description: End-of-Life (for `digest === undefined`).
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4047
    description: Runtime deprecation (for `digest === undefined`).
-->

类型：报废

使用[`crypto.pbkdf2()`][crypto.pbkdf2()]未指定摘要的 API 已弃用
在 Node.js 6.0 中，因为该方法默认使用不推荐的
`'SHA1'`消化。以前，打印了弃用警告。起价
节点.js 8.0.0，调用`crypto.pbkdf2()`或`crypto.pbkdf2Sync()`跟
`digest`设置为`undefined`将抛出一个`TypeError`.

从 Node.js v11.0.0 开始，调用这些函数`digest`设置为
`null`将打印弃用警告，以便在以下情况下与行为保持一致`digest`
是`undefined`.

然而，现在，通过`undefined`或`null`将抛出一个`TypeError`.

### DEP0010：`crypto.createCredentials`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/21153
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.13
    pr-url: https://github.com/nodejs/node-v0.x-archive/pull/7265
    description: Runtime deprecation.
-->

类型：报废

这`crypto.createCredentials()`已删除 API。请使用
[`tls.createSecureContext()`][tls.createSecureContext()]相反。

### DEP0011：`crypto.Credentials`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/21153
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.13
    pr-url: https://github.com/nodejs/node-v0.x-archive/pull/7265
    description: Runtime deprecation.
-->

类型：报废

这`crypto.Credentials`类已被删除。请使用[`tls.SecureContext`][tls.SecureContext]
相反。

### DEP0012：`Domain.dispose`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15412
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.7
    pr-url: https://github.com/nodejs/node-v0.x-archive/pull/5021
    description: Runtime deprecation.
-->

类型：报废

`Domain.dispose()`已被删除。从失败的 I/O 操作中恢复
通过域上设置的错误事件处理程序显式地进行。

### DEP0013：`fs`不带回调的异步函数

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18668
    description: End-of-Life.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: Runtime deprecation.
-->

类型：报废

在没有回调的情况下调用异步函数会引发`TypeError`
在节点.js 10.0.0 及更高版本中。看<https://github.com/nodejs/node/pull/12562>.

### DEP0014：`fs.read`旧字符串接口

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9683
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4525
    description: Runtime deprecation.
  - version: v0.1.96
    commit: c93e0aaf062081db3ec40ac45b3e2c979d5759d6
    description: Documentation-only deprecation.
-->

类型：报废

这[`fs.read()`][fs.read()]遗产`String`接口已弃用。使用`Buffer`
改为文档中提到的 API。

### DEP0015：`fs.readSync`旧字符串接口

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9683
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4525
    description: Runtime deprecation.
  - version: v0.1.96
    commit: c93e0aaf062081db3ec40ac45b3e2c979d5759d6
    description: Documentation-only deprecation.
-->

类型：报废

这[`fs.readSync()`][fs.readSync()]遗产`String`接口已弃用。使用
`Buffer`改为文档中提到的 API。

### DEP0016：`GLOBAL`/`root`

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31167
    description: End-of-Life.
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/1838
    description: Runtime deprecation.
-->

类型：报废

这`GLOBAL`和`root`的别名`global`属性已弃用
在 Node.js 6.0.0 中，此后已被删除。

### DEP0017：`Intl.v8BreakIterator`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15238
    description: End-of-Life.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8908
    description: Runtime deprecation.
-->

类型：报废

`Intl.v8BreakIterator`是非标准扩展，已被删除。
看[`Intl.Segmenter`](https://github.com/tc39/proposal-intl-segmenter).

### DEP0018：未处理的承诺拒绝

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35316
    description: End-of-Life.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8217
    description: Runtime deprecation.
-->

类型：报废

未处理的应许拒绝已弃用。默认情况下，承诺被拒绝
未处理的终止节点.js进程，并带有非零出口
法典。要更改 Node.js 处理未处理的拒绝的方式，请使用
[`--unhandled-rejections`][--unhandled-rejections]命令行选项。

### DEP0019：`require('.')`已解析的外部目录

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26973
    description: Removed functionality.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v1.8.1
    pr-url: https://github.com/nodejs/node/pull/1363
    description: Runtime deprecation.
-->

类型：报废

在某些情况下，`require('.')`可以在包目录之外解析。
此行为已删除。

### DEP0020：`Server.connections`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33647
    description: Server.connections has been removed.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.9.7
    pr-url: https://github.com/nodejs/node-v0.x-archive/pull/4595
    description: Runtime deprecation.
-->

类型：报废

这`Server.connections`属性在 Node.js v0.9.7 中已弃用，并且具有
已删除。请使用[`Server.getConnections()`][Server.getConnections()]方法代替。

### DEP0021：`Server.listenFD`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27127
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.7.12
    commit: 41421ff9da1288aa241a5e9dcf915b685ade1c23
    description: Runtime deprecation.
-->

类型：报废

这`Server.listenFD()`方法已弃用并删除。请使用
[`Server.listen({fd: <number>})`][Server.listen({fd: <number>})]相反。

### DEP0022：`os.tmpDir()`

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31169
    description: End-of-Life.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/6739
    description: Runtime deprecation.
-->

类型：报废

这`os.tmpDir()`API 在 Node.js 7.0.0 中已弃用，此后一直处于
删除。请使用[`os.tmpdir()`][os.tmpdir()]相反。

### DEP0023：`os.getNetworkInterfaces()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25280
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.6.0
    commit: 37bb37d151fb6ee4696730e63ff28bb7a4924f97
    description: Runtime deprecation.
-->

类型：报废

这`os.getNetworkInterfaces()`方法已弃用。请使用
[`os.networkInterfaces()`][os.networkInterfaces()]方法代替。

### DEP0024：`REPLServer.prototype.convertToContext()`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/13434
    description: End-of-Life.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7829
    description: Runtime deprecation.
-->

类型：报废

这`REPLServer.prototype.convertToContext()`API 已被删除。

### DEP0025：`require('node:sys')`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v1.0.0
    pr-url: https://github.com/nodejs/node/pull/317
    description: Runtime deprecation.
-->

类型：运行时

这`node:sys`模块已弃用。请使用[`util`][util]模块。

### DEP0026：`util.print()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25377
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.3
    commit: 896b2aa7074fc886efd7dd0a397d694763cac7ce
    description: Runtime deprecation.
-->

类型：报废

`util.print()`已被删除。请使用[`console.log()`][console.log()]相反。

### DEP0027：`util.puts()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25377
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.3
    commit: 896b2aa7074fc886efd7dd0a397d694763cac7ce
    description: Runtime deprecation.
-->

类型：报废

`util.puts()`已被删除。请使用[`console.log()`][console.log()]相反。

### DEP0028：`util.debug()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25377
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.3
    commit: 896b2aa7074fc886efd7dd0a397d694763cac7ce
    description: Runtime deprecation.
-->

类型：报废

`util.debug()`已被删除。请使用[`console.error()`][console.error()]相反。

### DEP0029：`util.error()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25377
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.3
    commit: 896b2aa7074fc886efd7dd0a397d694763cac7ce
    description: Runtime deprecation.
-->

类型：报废

`util.error()`已被删除。请使用[`console.error()`][console.error()]相反。

### DEP0030：`SlowBuffer`

<!-- YAML
changes:
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5833
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`SlowBuffer`][SlowBuffer]类已弃用。请使用
[`Buffer.allocUnsafeSlow(size)`][Buffer.allocUnsafeSlow(size)]相反。

### DEP0031：`ecdh.setPublicKey()`

<!-- YAML
changes:
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v5.2.0
    pr-url: https://github.com/nodejs/node/pull/3511
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`ecdh.setPublicKey()`][ecdh.setPublicKey()]方法现在已被弃用，因为它包含在
API 没有用。

### DEP0032：`node:domain`模块

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v1.4.2
    pr-url: https://github.com/nodejs/node/pull/943
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`domain`][domain]模块已弃用，不应使用。

### DEP0033：`EventEmitter.listenerCount()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v3.2.0
    pr-url: https://github.com/nodejs/node/pull/2349
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`events.listenerCount(emitter, eventName)`][events.listenerCount(emitter, eventName)]原料药是
荒废的。请使用[`emitter.listenerCount(eventName)`][emitter.listenerCount(eventName)]相反。

### DEP0034：`fs.exists(path, callback)`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v1.0.0
    pr-url: https://github.com/nodejs/node/pull/166
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`fs.exists(path, callback)`][fs.exists(path, callback)]API 已弃用。请使用
[`fs.stat()`][fs.stat()]或[`fs.access()`][fs.access()]相反。

### DEP0035：`fs.lchmod(path, mode, callback)`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.4.7
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`fs.lchmod(path, mode, callback)`][fs.lchmod(path, mode, callback)]API 已弃用。

### DEP0036：`fs.lchmodSync(path, mode)`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.4.7
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`fs.lchmodSync(path, mode)`][fs.lchmodSync(path, mode)]API 已弃用。

### DEP0037：`fs.lchown(path, uid, gid, callback)`

<!-- YAML
changes:
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/21498
    description: Deprecation revoked.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.4.7
    description: Documentation-only deprecation.
-->

类型：已弃用已吊销

这[`fs.lchown(path, uid, gid, callback)`][fs.lchown(path, uid, gid, callback)]API 已弃用。这
弃用已被撤销，因为必要的支持 API 已添加到
libuv.

### DEP0038：`fs.lchownSync(path, uid, gid)`

<!-- YAML
changes:
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/21498
    description: Deprecation revoked.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.4.7
    description: Documentation-only deprecation.
-->

类型：已弃用已吊销

这[`fs.lchownSync(path, uid, gid)`][fs.lchownSync(path, uid, gid)]API 已弃用。弃用是
已撤销，因为在 libuv 中添加了必要的支持 API。

### DEP0039：`require.extensions`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.10.6
    commit: 7bd8a5a2a60b75266f89f9a32877d55294a3881c
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`require.extensions`][require.extensions]属性已弃用。

### DEP0040：`node:punycode`模块

<!-- YAML
changes:
  - version: v16.6.0
    pr-url: https://github.com/nodejs/node/pull/38444
    description: Added support for `--pending-deprecation`.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7941
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

这[`punycode`][punycode]模块已弃用。请使用用户空间替代方案
相反。

### DEP0041：`NODE_REPL_HISTORY_FILE`环境变量

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/13876
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v3.0.0
    pr-url: https://github.com/nodejs/node/pull/2224
    description: Documentation-only deprecation.
-->

类型：报废

这`NODE_REPL_HISTORY_FILE`环境变量已被删除。请使用
`NODE_REPL_HISTORY`相反。

### DEP0042：`tls.CryptoStream`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17882
    description: End-of-Life.
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v0.11.3
    commit: af80e7bc6e6f33c582eb1f7d37c7f5bbe9f910f7
    description: Documentation-only deprecation.
-->

类型：报废

这[`tls.CryptoStream`][tls.CryptoStream]类已被删除。请使用
[`tls.TLSSocket`][tls.TLSSocket]相反。

### DEP0043：`tls.SecurePair`

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11349
    description: Runtime deprecation.
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6063
    description: Documentation-only deprecation.
  - version: v0.11.15
    pr-url:
      - https://github.com/nodejs/node-v0.x-archive/pull/8695
      - https://github.com/nodejs/node-v0.x-archive/pull/8700
    description: Deprecation revoked.
  - version: v0.11.3
    commit: af80e7bc6e6f33c582eb1f7d37c7f5bbe9f910f7
    description: Runtime deprecation.
-->

类型：仅文档

这[`tls.SecurePair`][tls.SecurePair]类已弃用。请使用
[`tls.TLSSocket`][tls.TLSSocket]相反。

### DEP0044：`util.isArray()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isArray()`][util.isArray()]API 已弃用。请使用`Array.isArray()`
相反。

### DEP0045：`util.isBoolean()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isBoolean()`][util.isBoolean()]API 已弃用。

### DEP0046：`util.isBuffer()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isBuffer()`][util.isBuffer()]API 已弃用。请使用
[`Buffer.isBuffer()`][Buffer.isBuffer()]相反。

### DEP0047：`util.isDate()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isDate()`][util.isDate()]API 已弃用。

### DEP0048：`util.isError()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isError()`][util.isError()]API 已弃用。

### DEP0049：`util.isFunction()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isFunction()`][util.isFunction()]API 已弃用。

### DEP0050：`util.isNull()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isNull()`][util.isNull()]API 已弃用。

### DEP0051：`util.isNullOrUndefined()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isNullOrUndefined()`][util.isNullOrUndefined()]API 已弃用。

### DEP0052：`util.isNumber()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isNumber()`][util.isNumber()]API 已弃用。

### DEP0053：`util.isObject()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isObject()`][util.isObject()]API 已弃用。

### DEP0054：`util.isPrimitive()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isPrimitive()`][util.isPrimitive()]API 已弃用。

### DEP0055：`util.isRegExp()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isRegExp()`][util.isRegExp()]API 已弃用。

### DEP0056：`util.isString()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isString()`][util.isString()]API 已弃用。

### DEP0057：`util.isSymbol()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isSymbol()`][util.isSymbol()]API 已弃用。

### DEP0058：`util.isUndefined()`

<!-- YAML
changes:
  - version:
    - v6.12.0
    - v4.8.6
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version:
    - v4.0.0
    - v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2447
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.isUndefined()`][util.isUndefined()]API 已弃用。

### DEP0059：`util.log()`

<!-- YAML
changes:
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6161
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util.log()`][util.log()]API 已弃用。

### DEP0060：`util._extend()`

<!-- YAML
changes:
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4903
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`util._extend()`][util._extend()]API 已弃用。

### DEP0061：`fs.SyncWriteStream`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/20735
    description: End-of-Life.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10467
    description: Runtime deprecation.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/6749
    description: Documentation-only deprecation.
-->

类型：报废

这`fs.SyncWriteStream`类从未打算成为可公开访问的
API 和 已被删除。没有可用的替代 API。请使用用户空间
另类。

### DEP0062：`node --debug`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25828
    description: End-of-Life.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10970
    description: Runtime deprecation.
-->

类型：报废

`--debug`激活旧版 V8 调试器接口，该接口已被删除为
的 V8 5.8。它被检查器取代，检查器被激活`--inspect`
相反。

### DEP0063：`ServerResponse.prototype.writeHeader()`

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11355
    description: Documentation-only deprecation.
-->

类型：仅文档

这`node:http`模块`ServerResponse.prototype.writeHeader()`原料药是
荒废的。请使用`ServerResponse.prototype.writeHead()`相反。

这`ServerResponse.prototype.writeHeader()`方法从未记录为
官方支持的 API。

### DEP0064：`tls.createSecurePair()`

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11349
    description: Runtime deprecation.
  - version: v6.12.0
    pr-url: https://github.com/nodejs/node/pull/10116
    description: A deprecation code has been assigned.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6063
    description: Documentation-only deprecation.
  - version: v0.11.15
    pr-url:
      - https://github.com/nodejs/node-v0.x-archive/pull/8695
      - https://github.com/nodejs/node-v0.x-archive/pull/8700
    description: Deprecation revoked.
  - version: v0.11.3
    commit: af80e7bc6e6f33c582eb1f7d37c7f5bbe9f910f7
    description: Runtime deprecation.
-->

类型：运行时

这`tls.createSecurePair()`API 在 Node 的文档中已被弃用.js
0.11.3. 用户应使用`tls.Socket`相反。

### DEP0065：`repl.REPL_MODE_MAGIC`和`NODE_REPL_MODE=magic`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19187
    description: End-of-Life.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11599
    description: Documentation-only deprecation.
-->

类型：报废

这`node:repl`模块的`REPL_MODE_MAGIC`常量，用于`replMode`选择
已被删除。其行为在功能上与
`REPL_MODE_SLOPPY`自 Node.js 6.0.0 起，导入 V8 5.0 时。请使用
`REPL_MODE_SLOPPY`相反。

这`NODE_REPL_MODE`环境变量用于设置底层
`replMode`的交互式`node`会期。它的价值，`magic`，也是
删除。请使用`sloppy`相反。

### DEP0066：`OutgoingMessage.prototype._headers, OutgoingMessage.prototype._headerNames`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/24167
    description: Runtime deprecation.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10941
    description: Documentation-only deprecation.
-->

类型：运行时

这`node:http`模块`OutgoingMessage.prototype._headers`和
`OutgoingMessage.prototype._headerNames`属性已弃用。使用其中之一
公共方法（例如`OutgoingMessage.prototype.getHeader()`,
`OutgoingMessage.prototype.getHeaders()`,
`OutgoingMessage.prototype.getHeaderNames()`,
`OutgoingMessage.prototype.getRawHeaderNames()`,
`OutgoingMessage.prototype.hasHeader()`,
`OutgoingMessage.prototype.removeHeader()`,
`OutgoingMessage.prototype.setHeader()`） 用于处理传出标头。

这`OutgoingMessage.prototype._headers`和
`OutgoingMessage.prototype._headerNames`属性从未记录为
官方支持的属性。

### DEP0067：`OutgoingMessage.prototype._renderHeaders`

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10941
    description: Documentation-only deprecation.
-->

类型：仅文档

这`node:http`模块`OutgoingMessage.prototype._renderHeaders()`原料药是
荒废的。

这`OutgoingMessage.prototype._renderHeaders`属性从未记录为
官方支持的 API。

### DEP0068：`node debug`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33648
    description: The legacy `node debug` command was removed.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11441
    description: Runtime deprecation.
-->

类型：报废

`node debug`对应于旧版 CLI 调试器，该调试器已替换为
基于 V8 检查器的 CLI 调试器可通过`node inspect`.

### DEP0069：`vm.runInDebugContext(string)`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/13295
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/12815
    description: Runtime deprecation.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12243
    description: Documentation-only deprecation.
-->

类型：报废

DebugContext 在 V8 中已被删除，在 Node.js 10+ 中不可用。

DebugContext是一个实验性的API。

### DEP0070：`async_hooks.currentId()`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14414
    description: End-of-Life.
  - version: v8.2.0
    pr-url: https://github.com/nodejs/node/pull/13490
    description: Runtime deprecation.
-->

类型：报废

`async_hooks.currentId()`已重命名为`async_hooks.executionAsyncId()`为
清晰。

此更改是在以下情况下进行的`async_hooks`是一个实验性的 API。

### DEP0071：`async_hooks.triggerId()`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14414
    description: End-of-Life.
  - version: v8.2.0
    pr-url: https://github.com/nodejs/node/pull/13490
    description: Runtime deprecation.
-->

类型：报废

`async_hooks.triggerId()`已重命名为`async_hooks.triggerAsyncId()`为
清晰。

此更改是在以下情况下进行的`async_hooks`是一个实验性的 API。

### DEP0072：`async_hooks.AsyncResource.triggerId()`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14414
    description: End-of-Life.
  - version: v8.2.0
    pr-url: https://github.com/nodejs/node/pull/13490
    description: Runtime deprecation.
-->

类型：报废

`async_hooks.AsyncResource.triggerId()`已重命名为
`async_hooks.AsyncResource.triggerAsyncId()`为清楚起见。

此更改是在以下情况下进行的`async_hooks`是一个实验性的 API。

### DEP0073： 几个内部属性`net.Server`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17141
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14449
    description: Runtime deprecation.
-->

类型：报废

访问 多个未记录的内部属性`net.Server`实例
不推荐使用不适当的名称。

由于原始API是未记录的，通常对非内部没有用处
代码中，未提供替换 API。

### DEP0074：`REPLServer.bufferedCommand`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33286
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/13687
    description: Runtime deprecation.
-->

类型：报废

这`REPLServer.bufferedCommand`属性已被弃用，转而支持
[`REPLServer.clearBufferedCommand()`][REPLServer.clearBufferedCommand()].

### DEP0075：`REPLServer.parseREPLKeyword()`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33286
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14223
    description: Runtime deprecation.
-->

类型：报废

`REPLServer.parseREPLKeyword()`已从用户空间可见性中删除。

### DEP0076：`tls.parseCertString()`

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41479
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14249
    description: Runtime deprecation.
  - version: v8.6.0
    pr-url: https://github.com/nodejs/node/pull/14245
    description: Documentation-only deprecation.
-->

类型：报废

`tls.parseCertString()`是一个微不足道的解析助手，由
错误。虽然它应该解析证书使用者和颁发者字符串，
它从未正确处理多值相对可分辨名称。

本文档的早期版本建议使用`querystring.parse()`作为
替代`tls.parseCertString()`.然而`querystring.parse()`也做
未正确处理所有证书使用者，因此不应使用。

### DEP0077：`Module._debug()`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/13948
    description: Runtime deprecation.
-->

类型：运行时

`Module._debug()`已弃用。

这`Module._debug()`函数从未被记录为正式的
支持的 API。

### DEP0078：`REPLServer.turnOffEditorMode()`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33286
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15136
    description: Runtime deprecation.
-->

类型：报废

`REPLServer.turnOffEditorMode()`已从用户空间可见性中删除。

### DEP0079： 通过`.inspect()`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/20722
    description: End-of-Life.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16393
    description: Runtime deprecation.
  - version: v8.7.0
    pr-url: https://github.com/nodejs/node/pull/15631
    description: Documentation-only deprecation.
-->

类型：报废

使用名为`inspect`在对象上指定自定义检查
函数[`util.inspect()`][util.inspect()]已弃用。用[`util.inspect.custom`][util.inspect.custom]
相反。为了向后兼容 Node.js 6.4.0 版之前，两者兼而有之
可以指定。

### DEP0080：`path._makeLong()`

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/14956
    description: Documentation-only deprecation.
-->

类型：仅文档

内部`path._makeLong()`不打算供公众使用。然而
用户空间模块发现它很有用。内部 API 已弃用
并替换为相同的公共`path.toNamespacedPath()`方法。

### DEP0081：`fs.truncate()`使用文件描述符

<!-- YAML
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15990
    description: Runtime deprecation.
-->

类型：运行时

`fs.truncate()` `fs.truncateSync()`文件描述符的用法是
荒废的。请使用`fs.ftruncate()`或`fs.ftruncateSync()`以使用
文件描述符。

### DEP0082：`REPLServer.prototype.memory()`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33286
    description: End-of-Life.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/16242
    description: Runtime deprecation.
-->

类型：报废

`REPLServer.prototype.memory()`仅对于内部力学是必需的
这`REPLServer`本身。请勿使用此功能。

### DEP0083：通过设置禁用 ECDH`ecdhCurve`自`false`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19794
    description: End-of-Life.
  - version: v9.2.0
    pr-url: https://github.com/nodejs/node/pull/16130
    description: Runtime deprecation.
-->

类型：生命周期结束。

这`ecdhCurve`选项`tls.createSecureContext()`和`tls.TLSSocket`能
设置为`false`以仅在服务器上完全禁用 ECDH。此模式是
在准备迁移到 OpenSSL 1.1.0 时已弃用，并且与
客户端，现在不受支持。使用`ciphers`参数。

### DEP0084：需要捆绑的内部依赖关系

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25138
    description: This functionality has been removed.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16392
    description: Runtime deprecation.
-->

类型：报废

由于 Node.js 版本 4.4.0 和 5.2.0，因此有几个模块仅用于
内部用法被错误地暴露给用户代码，通过`require()`.这些
模块包括：

*   `v8/tools/codemap`
*   `v8/tools/consarray`
*   `v8/tools/csvparser`
*   `v8/tools/logreader`
*   `v8/tools/profile_view`
*   `v8/tools/profile`
*   `v8/tools/SourceMap`
*   `v8/tools/splaytree`
*   `v8/tools/tickprocessor-driver`
*   `v8/tools/tickprocessor`
*   `node-inspect/lib/_inspect`（从 7.6.0 开始）
*   `node-inspect/lib/internal/inspect_client`（从 7.6.0 开始）
*   `node-inspect/lib/internal/inspect_repl`（从 7.6.0 开始）

这`v8/*`模块没有任何导出，如果未导入，则在特定
顺序实际上会引发错误。因此，几乎没有合法用途
通过以下方式导入它们的案例`require()`.

另一方面`node-inspect`可以通过软件包在本地安装
管理器，因为它以相同的名称发布在 npm 注册表上。无来源
如果完成此操作，则必须进行代码修改。

### DEP0085： 异步钩子敏感 API

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17147
    description: End-of-Life.
  - version:
    - v9.4.0
    - v8.10.0
    pr-url: https://github.com/nodejs/node/pull/16972
    description: Runtime deprecation.
-->

类型：报废

AsyncHooks 敏感 API 从未被记录下来，并且存在各种小问题。
使用`AsyncResource`改为 API。看
<https://github.com/nodejs/node/issues/15572>.

### DEP0086： 删除`runInAsyncIdScope`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17147
    description: End-of-Life.
  - version:
    - v9.4.0
    - v8.10.0
    pr-url: https://github.com/nodejs/node/pull/16972
    description: Runtime deprecation.
-->

类型：报废

`runInAsyncIdScope`不发出`'before'`或`'after'`事件，因此可以
导致很多问题。看<https://github.com/nodejs/node/issues/14328>.

<!-- md-lint skip-deprecation DEP0087 -->

<!-- md-lint skip-deprecation DEP0088 -->

### DEP0089：`require('node:assert')`

<!-- YAML
changes:
  - version: v12.8.0
    pr-url: https://github.com/nodejs/node/pull/28892
    description: Deprecation revoked.
  - version:
      - v9.9.0
      - v8.13.0
    pr-url: https://github.com/nodejs/node/pull/17002
    description: Documentation-only deprecation.
-->

类型：已弃用已吊销

不建议直接导入断言，因为公开的函数使用
松散的相等性检查。弃用被撤销，因为使用了
`node:assert`模块不气馁，并且弃用导致开发人员
混乱。

### DEP0090： 无效的 GCM 身份验证标记长度

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/17825
    description: End-of-Life.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18017
    description: Runtime deprecation.
-->

类型：报废

Node.js用于支持所有 GCM 身份验证标记长度，这些标记长度由
调用时打开 SSL[`decipher.setAuthTag()`][decipher.setAuthTag()].从节点开始.js
v11.0.0，仅身份验证标记长度为 128、120、112、104、96、64 和 32
允许位。其他长度的身份验证标记对于每个
[NIST SP 800-38D][].

### DEP0091：`crypto.DEFAULT_ENCODING`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18333
    description: Runtime deprecation.
-->

类型：运行时

这[`crypto.DEFAULT_ENCODING`][crypto.DEFAULT_ENCODING]属性已弃用。

### DEP0092： 顶级`this`绑定到`module.exports`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16878
    description: Documentation-only deprecation.
-->

类型：仅文档

将属性分配给顶级`this`作为替代方案
自`module.exports`已弃用。开发人员应使用`exports`
或`module.exports`相反。

### DEP0093：`crypto.fips`已弃用和替换

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18335
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`crypto.fips`][crypto.fips]属性已弃用。请使用`crypto.setFips()`
和`crypto.getFips()`相反。

### DEP0094： 使用`assert.fail()`具有多个参数

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18418
    description: Runtime deprecation.
-->

类型：运行时

用`assert.fail()`不推荐使用多个参数。用
`assert.fail()`仅使用一个参数或使用其他参数`node:assert`模块
方法。

### DEP0095：`timers.enroll()`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18066
    description: Runtime deprecation.
-->

类型：运行时

`timers.enroll()`已弃用。请使用公开记录的
[`setTimeout()`][setTimeout()]或[`setInterval()`][setInterval()]相反。

### DEP0096：`timers.unenroll()`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18066
    description: Runtime deprecation.
-->

类型：运行时

`timers.unenroll()`已弃用。请使用公开记录的
[`clearTimeout()`][clearTimeout()]或[`clearInterval()`][clearInterval()]相反。

### DEP0097：`MakeCallback`跟`domain`财产

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17417
    description: Runtime deprecation.
-->

类型：运行时

用户`MakeCallback`添加`domain`属性承载上下文，
应该开始使用`async_context`的变体`MakeCallback`或
`CallbackScope`，或高级别`AsyncResource`类。

### DEP0098： 异步胡克斯嵌入器`AsyncResource.emitBefore`和`AsyncResource.emitAfter`蜜蜂属

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26530
    description: End-of-Life.
  - version:
    - v10.0.0
    - v9.6.0
    - v8.12.0
    pr-url: https://github.com/nodejs/node/pull/18632
    description: Runtime deprecation.
-->

类型：报废

AsyncHooks 提供的嵌入式 API 公开`.emitBefore()`和
`.emitAfter()`非常容易使用的方法不正确，可能导致
到不可恢复的错误。

用[`asyncResource.runInAsyncScope()`][asyncResource.runInAsyncScope()]相反，API提供了很多
更安全，更方便的替代方案。看
<https://github.com/nodejs/node/pull/18513>.

### DEP0099：异步上下文识别`node::MakeCallback`C++接口

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18632
    description: Compile-time deprecation.
-->

类型：编译时

某些版本的`node::MakeCallback`可用于本机插件的 API 是
荒废的。请使用接受`async_context`
参数。

### DEP0100：`process.assert()`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18666
    description: Runtime deprecation.
  - version: v0.3.7
    description: Documentation-only deprecation.
-->

类型：运行时

`process.assert()`已弃用。请使用[`assert`][assert]模块。

这从来都不是记录在案的功能。

### DEP0101：`--with-lttng`

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18982
    description: End-of-Life.
-->

类型：报废

这`--with-lttng`编译时选项已被删除。

### DEP0102： 使用`noAssert`在`Buffer#(read|write)`操作

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: End-of-Life.
-->

类型：报废

使用`noAssert`参数不再具有任何功能。所有输入均为
无论`noAssert`.跳过验证
可能导致难以找到的错误和崩溃。

### DEP0103：`process.binding('util').is[...]`类型检查

<!-- YAML
changes:
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/22004
    description: Superseded by [DEP0111](#DEP0111).
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18415
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

用`process.binding()`一般应避免。类型检查
特别是方法可以通过使用[`util.types`][util.types].

此弃用已被 弃用
`process.binding()`原料药 （[DEP0111](#DEP0111)).

### DEP0104：`process.env`字符串强制

<!-- YAML
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18990
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

将非字符串属性赋给[`process.env`][process.env]，则分配的值为
隐式转换为字符串。如果分配了
value 不是字符串、布尔值或数字。今后，这种转让可能
导致引发错误。请在之前将属性转换为字符串
将其分配给`process.env`.

### DEP0105：`decipher.finaltol`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/19941
    description: End-of-Life.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19353
    description: Runtime deprecation.
-->

类型：报废

`decipher.finaltol()`从未被记录过，并且是
[`decipher.final()`][decipher.final()].此 API 已被删除，建议使用
[`decipher.final()`][decipher.final()]相反。

### DEP0106：`crypto.createCipher`和`crypto.createDecipher`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22089
    description: Runtime deprecation.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19343
    description: Documentation-only deprecation.
-->

类型：运行时

用[`crypto.createCipher()`][crypto.createCipher()]和[`crypto.createDecipher()`][crypto.createDecipher()]应该是
避免使用弱键派生函数（MD5无盐）和静态
初始化向量。建议使用
[`crypto.pbkdf2()`][crypto.pbkdf2()]或[`crypto.scrypt()`][crypto.scrypt()]并使用
[`crypto.createCipheriv()`][crypto.createCipheriv()]和[`crypto.createDecipheriv()`][crypto.createDecipheriv()]以获得
[`Cipher`][Cipher]和[`Decipher`][Decipher]分别是对象。

### DEP0107：`tls.convertNPNProtocols()`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/20736
    description: End-of-Life.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19403
    description: Runtime deprecation.
-->

类型：报废

这是一个未记录的帮助程序函数，不适合在 Node 外部使用.js
核心和过时，由于删除了NPN（下一个协议协商）支持。

### DEP0108：`zlib.bytesRead`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/23308
    description: Runtime deprecation.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19414
    description: Documentation-only deprecation.
-->

类型：运行时

已弃用的别名[`zlib.bytesWritten`][zlib.bytesWritten].选择此原始名称
因为将值解释为字节数也是有意义的
由引擎读取，但与 Node 中的其他流不一致.js
公开这些名称下的值。

### DEP0109：`http`,`https`和`tls`支持无效网址

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36853
    description: End-of-Life.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/20270
    description: Runtime deprecation.
-->

类型：报废

一些以前支持（但完全无效）的 URL 通过
[`http.request()`][http.request()],[`http.get()`][http.get()],[`https.request()`][https.request()],
[`https.get()`][https.get()]和[`tls.checkServerIdentity()`][tls.checkServerIdentity()]API，因为它们是
被遗产所接受`url.parse()`应用程序接口。上述 API 现在使用 WHATWG
需要严格有效 URL 的 URL 解析器。传递无效的 URL 是
已弃用，并且将来将删除支持。

### DEP0110：`vm.Script`缓存数据

<!-- YAML
changes:
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/20300
    description: Documentation-only deprecation.
-->

类型：仅文档

这`produceCachedData`选项已弃用。用
[`script.createCachedData()`][script.createCachedData()]相反。

### DEP0111：`process.binding()`

<!-- YAML
changes:
  - version: v11.12.0
    pr-url: https://github.com/nodejs/node/pull/26500
    description: Added support for `--pending-deprecation`.
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/22004
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

`process.binding()`仅供 Node.js内部代码使用。

### DEP0112：`dgram`私有 API

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22011
    description: Runtime deprecation.
-->

类型：运行时

这`node:dgram`模块以前包含几个从未有过的 API
在 Node.js 核心之外访问：`Socket.prototype._handle`,
`Socket.prototype._receiving`,`Socket.prototype._bindState`,
`Socket.prototype._queue`,`Socket.prototype._reuseAddr`,
`Socket.prototype._healthCheck()`,`Socket.prototype._stopReceiving()`和
`dgram._createSocketHandle()`.

### DEP0113：`Cipher.setAuthTag()`,`Decipher.getAuthTag()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26249
    description: End-of-Life.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22126
    description: Runtime deprecation.
-->

类型：报废

`Cipher.setAuthTag()`和`Decipher.getAuthTag()`不再可用。他们
从未被记录下来，并且在被调用时会抛出。

### DEP0114：`crypto._toBuf()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25338
    description: End-of-Life.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22501
    description: Runtime deprecation.
-->

类型：报废

这`crypto._toBuf()`函数未设计为由外部模块使用
的节点.js核心，并已被删除。

<!--lint disable nodejs-yaml-comments -->

### DEP0115：`crypto.prng()`,`crypto.pseudoRandomBytes()`,`crypto.rng()`

<!-- YAML
changes:
  - version: v11.0.0
    pr-url:
      - https://github.com/nodejs/node/pull/22519
      - https://github.com/nodejs/node/pull/23017
    description: Added documentation-only deprecation
                 with `--pending-deprecation` support.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

<!--lint enable nodejs-yaml-comments -->

在最新版本的 Node.js 中，两者之间没有区别
[`crypto.randomBytes()`][crypto.randomBytes()]和`crypto.pseudoRandomBytes()`.后者是
与未记录的别名一起弃用`crypto.prng()`和
`crypto.rng()`赞成[`crypto.randomBytes()`][crypto.randomBytes()]并可能在
未来版本。

### DEP0116： 旧版 URL API

<!-- YAML
changes:
  - version:
      - v15.13.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37784
    description: Deprecation revoked. Status changed to "Legacy".
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22715
    description: Documentation-only deprecation.
-->

类型：已弃用已吊销

这[旧版网址 API][Legacy URL API]已弃用。这包括[`url.format()`][url.format()],
[`url.parse()`][url.parse()],[`url.resolve()`][url.resolve()]，以及[遗产`urlObject`][legacy urlObject].请
使用[WHATWG URL API][]相反。

### DEP0117：本机加密句柄

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27011
    description: End-of-Life.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22747
    description: Runtime deprecation.
-->

类型：报废

以前版本的 Node.js通过以下方式向内部本机对象公开句柄
这`_handle`的属性`Cipher`,`Decipher`,`DiffieHellman`,
`DiffieHellmanGroup`,`ECDH`,`Hash`,`Hmac`,`Sign`和`Verify`类。
这`_handle`属性已被删除，因为不正确地使用了本机
对象可能导致应用程序崩溃。

### DEP0118：`dns.lookup()`支持虚假的主机名

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/23173
    description: Runtime deprecation.
-->

类型：运行时

以前版本的 Node.js 受支持`dns.lookup()`使用虚假的主机名
喜欢`dns.lookup(false)`由于向后兼容性。
此行为未记录，并且被认为在实际应用中未使用。
在 Node.js 的未来版本中，它将成为一个错误。

### DEP0119：`process.binding('uv').errname()`私有接口

<!-- YAML
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/23597
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

`process.binding('uv').errname()`已弃用。请使用
[`util.getSystemErrorName()`][util.getSystemErrorName()]相反。

### DEP0120： 视窗性能计数器支持

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/24862
    description: End-of-Life.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22485
    description: Runtime deprecation.
-->

类型：报废

Windows 性能计数器支持已从 Node.js 中删除。这
非法`COUNTER_NET_SERVER_CONNECTION()`,
`COUNTER_NET_SERVER_CONNECTION_CLOSE()`,`COUNTER_HTTP_SERVER_REQUEST()`,
`COUNTER_HTTP_SERVER_RESPONSE()`,`COUNTER_HTTP_CLIENT_REQUEST()`和
`COUNTER_HTTP_CLIENT_RESPONSE()`函数已被弃用。

### DEP0121：`net._setSimultaneousAccepts()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23760
    description: Runtime deprecation.
-->

类型：运行时

无证件`net._setSimultaneousAccepts()`函数最初是
用于在使用
`node:child_process`和`node:cluster`模块在 Windows 上。该函数不是
通常有用，并且正在被删除。请参阅此处的讨论：
<https://github.com/nodejs/node/issues/18391>

### DEP0122：`tls` `Server.prototype.setOptions()`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23820
    description: Runtime deprecation.
-->

类型：运行时

请使用`Server.prototype.setSecureContext()`相反。

### DEP0123：将 TLS 服务器名称设置为 IP 地址

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23329
    description: Runtime deprecation.
-->

类型：运行时

不允许将 TLS 服务器名称设置为 IP 地址
[RFC 6066][].这将在将来的版本中被忽略。

### DEP0124： 使用`REPLServer.rli`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33286
    description: End-of-Life.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26260
    description: Runtime deprecation.
-->

类型：报废

此属性是对实例本身的引用。

### DEP0125：`require('node:_stream_wrap')`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26245
    description: Runtime deprecation.
-->

类型：运行时

这`node:_stream_wrap`模块已弃用。

### DEP0126：`timers.active()`

<!-- YAML
changes:
  - version: v11.14.0
    pr-url: https://github.com/nodejs/node/pull/26760
    description: Runtime deprecation.
-->

类型：运行时

以前未记录的`timers.active()`已弃用。
请使用公开记录的[`timeout.refresh()`][timeout.refresh()]相反。
如果需要重新引用超时，[`timeout.ref()`][timeout.ref()]可以使用
自 Node.js 10 以来没有性能影响。

### DEP0127：`timers._unrefActive()`

<!-- YAML
changes:
  - version: v11.14.0
    pr-url: https://github.com/nodejs/node/pull/26760
    description: Runtime deprecation.
-->

类型：运行时

以前未记录的“私人”`timers._unrefActive()`已弃用。
请使用公开记录的[`timeout.refresh()`][timeout.refresh()]相反。
如果取消引用超时是必需的，[`timeout.unref()`][timeout.unref()]可以使用
自 Node.js 10 以来没有性能影响。

### DEP0128：模块无效`main`条目和`index.js`文件

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37204
    description: Runtime deprecation.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26823
    description: Documentation-only.
-->

类型：运行时

具有无效的模块`main`进入（例如，`./does-not-exist.js`） 和
还有一个`index.js`顶级目录中的文件将解析
`index.js`文件。这已被弃用，并且将来会引发错误
节点.js版本。

### DEP0129：`ChildProcess._channel`

<!-- YAML
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27949
    description: Runtime deprecation.
  - version: v11.14.0
    pr-url: https://github.com/nodejs/node/pull/26982
    description: Documentation-only.
-->

类型：运行时

这`_channel`返回的子进程对象的属性`spawn()`和
类似的功能不供公众使用。用`ChildProcess.channel`
相反。

### DEP0130：`Module.createRequireFromPath()`

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37201
    description: End-of-life.
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27951
    description: Runtime deprecation.
  - version: v12.2.0
    pr-url: https://github.com/nodejs/node/pull/27405
    description: Documentation-only.
-->

类型：报废

用[`module.createRequire()`][module.createRequire()]相反。

### DEP0131： 旧版 HTTP 解析器

<!-- YAML
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/29589
    description: This feature has been removed.
  - version: v12.22.0
    pr-url: https://github.com/nodejs/node/pull/37603
    description: Runtime deprecation.
  - version: v12.3.0
    pr-url: https://github.com/nodejs/node/pull/27498
    description: Documentation-only.
-->

类型：报废

旧版 HTTP 解析器，默认在 Node 版本中使用.js 12.0.0 之前的版本，
已弃用，并已在 v13.0.0 中删除。在 v13.0.0 之前，
`--http-parser=legacy`命令行标志可用于恢复为使用
旧版解析器。

### DEP0132：`worker.terminate()`带回调

<!-- YAML
changes:
  - version: v12.5.0
    pr-url: https://github.com/nodejs/node/pull/28021
    description: Runtime deprecation.
-->

类型：运行时

将回调传递给[`worker.terminate()`][worker.terminate()]已弃用。使用返回的
`Promise`相反，或者是工人的听众`'exit'`事件。

### DEP0133：`http` `connection`

<!-- YAML
changes:
  - version: v12.12.0
    pr-url: https://github.com/nodejs/node/pull/29015
    description: Documentation-only deprecation.
-->

类型：仅文档

喜欢[`response.socket`][response.socket]多[`response.connection`][response.connection]和
[`request.socket`][request.socket]多[`request.connection`][request.connection].

### DEP0134：`process._tickCallback`

<!-- YAML
changes:
  - version: v12.12.0
    pr-url: https://github.com/nodejs/node/pull/29781
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

这`process._tickCallback`属性从未记录为
官方支持的 API。

### DEP0135：`WriteStream.open()`和`ReadStream.open()`是内部的

<!-- YAML
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/29061
    description: Runtime deprecation.
-->

类型：运行时

[`WriteStream.open()`][WriteStream.open()]和[`ReadStream.open()`][ReadStream.open()]是未记录的内部
在用户空间中使用没有意义的 API。文件流应始终
通过相应的工厂方法开业[`fs.createWriteStream()`][fs.createWriteStream()]
和[`fs.createReadStream()`][fs.createReadStream()]） 或通过在选项中传递文件描述符。

### DEP0136：`http` `finished`

<!-- YAML
changes:
  - version:
     - v13.4.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/28679
    description: Documentation-only deprecation.
-->

类型：仅文档

[`response.finished`][response.finished]指示是否[`response.end()`][response.end()]已经
已调用，而不是是否`'finish'`已发出，并且基础数据
已刷新。

用[`response.writableFinished`][response.writableFinished]或[`response.writableEnded`][response.writableEnded]
因此，为了避免歧义。

维护现有行为`response.finished`应替换为
`response.writableEnded`.

### DEP0137： 闭幕 fs.垃圾回收上的文件处理

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/28396
    description: Runtime deprecation.
-->

类型：运行时

允许[`fs.FileHandle`][fs.FileHandle]在垃圾回收时要关闭的对象是
荒废的。将来，这样做可能会导致抛出错误，该错误将
终止进程。

请确保所有`fs.FileHandle`对象使用显式关闭
`FileHandle.prototype.close()`当`fs.FileHandle`不再需要：

```js
const fsPromises = require('node:fs').promises;
async function openAndClose() {
  let filehandle;
  try {
    filehandle = await fsPromises.open('thefile.txt', 'r');
  } finally {
    if (filehandle !== undefined)
      await filehandle.close();
  }
}
```

### DEP0138：`process.mainModule`

<!-- YAML
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/32232
    description: Documentation-only deprecation.
-->

类型：仅文档

[`process.mainModule`][process.mainModule]是一个仅限通用 JS 的功能，而`process`全球
对象与非 CommonJS 环境共享。它在 ECMAScript 中的使用
模块不受支持。

它被弃用，以支持[`require.main`][require.main]，因为它具有相同的服务
目的，并且仅在 CommonJS 环境中可用。

### DEP0139：`process.umask()`没有参数

<!-- YAML
changes:
  - version:
    - v14.0.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/32499
    description: Documentation-only deprecation.
-->

类型：仅文档

叫`process.umask()`没有参数导致进程范围的掩码
写两次。这会在线程之间引入争用条件，并且是
潜在的安全漏洞。没有安全的跨平台替代方案
应用程序接口。

### DEP0140： 使用`request.destroy()`而不是`request.abort()`

<!-- YAML
changes:
  - version:
    - v14.1.0
    - v13.14.0
    pr-url: https://github.com/nodejs/node/pull/32807
    description: Documentation-only deprecation.
-->

类型：仅文档

用[`request.destroy()`][request.destroy()]而不是[`request.abort()`][request.abort()].

### DEP0141：`repl.inputStream`和`repl.outputStream`

<!-- YAML
changes:
  - version: v14.3.0
    pr-url: https://github.com/nodejs/node/pull/33294
    description: Documentation-only (supports [`--pending-deprecation`][]).
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

这`node:repl`模块导出输入和输出流两次。用`.input`
而不是`.inputStream`和`.output`而不是`.outputStream`.

### DEP0142：`repl._builtinLibs`

<!-- YAML
changes:
  - version: v14.3.0
    pr-url: https://github.com/nodejs/node/pull/33294
    description: Documentation-only (supports [`--pending-deprecation`][]).
-->

类型：仅文档

这`node:repl`模块导出一个`_builtinLibs`包含数组的属性
内置模块。到目前为止，它不完整，相反，最好依靠
后`require('node:module').builtinModules`.

### DEP0143：`Transform._transformState`

<!-- YAML
changes:
  - version: v14.5.0
    pr-url: https://github.com/nodejs/node/pull/33126
    description: Runtime deprecation.
-->

类型：运行时
`Transform._transformState`将在未来的版本中将其删除
由于简化了实施，不再需要。

### DEP0144：`module.parent`

<!-- YAML
changes:
  - version:
    - v14.6.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/32217
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

CommonJS 模块可以使用以下命令访问需要它的第一个模块
`module.parent`.此功能已弃用，因为它不起作用
在 ECMAScript 模块存在的情况下始终如一，因为它给出了
CommonJS 模块图的表示不准确。

一些模块使用它来检查它们是否是当前进程的入口点。
相反，建议进行比较`require.main`和`module`:

```js
if (require.main === module) {
  // Code section that will run only if current file is the entry point.
}
```

当查找需要当前模块的 CommonJS 模块时，
`require.cache`和`module.children`可以使用：

```js
const moduleParents = Object.values(require.cache)
  .filter((m) => m.children.includes(module));
```

### DEP0145：`socket.bufferSize`

<!-- YAML
changes:
  - version: v14.6.0
    pr-url: https://github.com/nodejs/node/pull/34088
    description: Documentation-only deprecation.
-->

类型：仅文档

[`socket.bufferSize`][socket.bufferSize]只是[`writable.writableLength`][writable.writableLength].

### DEP0146：`new crypto.Certificate()`

<!-- YAML
changes:
  - version: v14.9.0
    pr-url: https://github.com/nodejs/node/pull/34697
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`crypto.Certificate()`构造 函数][crypto.Certificate() constructor]已弃用。用
[静态方法`crypto.Certificate()`][static methods of crypto.Certificate()]相反。

### DEP0147：`fs.rmdir(path, { recursive: true })`

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37302
    description: Runtime deprecation.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35562
    description: Runtime deprecation for permissive behavior.
  - version: v14.14.0
    pr-url: https://github.com/nodejs/node/pull/35579
    description: Documentation-only deprecation.
-->

类型：运行时

在 Node.js 的未来版本中，`recursive`选项将被忽略
`fs.rmdir`,`fs.rmdirSync`和`fs.promises.rmdir`.

用`fs.rm(path, { recursive: true, force: true })`,
`fs.rmSync(path, { recursive: true, force: true })`或
`fs.promises.rm(path, { recursive: true, force: true })`相反。

### DEP0148： 中的文件夹映射`"exports"`（尾随`"/"`)

<!-- YAML
changes:
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/40121
    description: End-of-Life.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37215
    description: Runtime deprecation.
  - version: v15.1.0
    pr-url: https://github.com/nodejs/node/pull/35747
    description: Runtime deprecation for self-referencing imports.
  - version: v14.13.0
    pr-url: https://github.com/nodejs/node/pull/34718
    description: Documentation-only deprecation.
-->

类型：运行时

使用尾随`"/"`以在
[子路径导出][subpath exports]或[子路径导入][subpath imports]字段已弃用。用
[子路径模式][subpath patterns]相反。

### DEP0149：`http.IncomingMessage#connection`

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/33768
    description: Documentation-only deprecation.
 -->

类型：仅文档。

喜欢[`message.socket`][message.socket]多[`message.connection`][message.connection].

### DEP0150： 更改 的值`process.config`

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36902
    description: Runtime deprecation.
-->

类型：运行时

这`process.config`属性提供对 Node.js编译时设置的访问。
但是，该属性是可变的，因此可能会被篡改。能力
若要更改的值将在 Node.js 的未来版本中删除。

### DEP0151：主索引查找和扩展搜索

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37206
    description: Runtime deprecation.
  - version:
      - v15.8.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/36918
    description: Documentation-only deprecation
                 with `--pending-deprecation` support.
-->

类型：运行时

以前`index.js`和扩展搜索查找将应用于
`import 'pkg'`主入口点分辨率，即使在解析 ES 模块时也是如此。

通过此弃用，所有ES模块主入口点分辨率都需要
显式[`"exports"`或`"main"`进入]["exports" or "main" entry]具有确切的文件扩展名。

### DEP0152：扩展性能输入属性

<!-- YAML
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Runtime deprecation.
-->

类型：运行时

这`'gc'`,`'http2'`和`'http'`{PerformanceEntry} 对象类型具有
分配给他们的其他属性，用于提供其他信息。
这些属性现在在标准中可用`detail`财产
的`PerformanceEntry`对象。现有的访问器已
已弃用，不应再使用。

### DEP0153：`dns.lookup`和`dnsPromises.lookup`选项类型强制

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41431
    description: End-of-Life.
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/39793
    description: Runtime deprecation.
  - version: v16.8.0
    pr-url: https://github.com/nodejs/node/pull/38906
    description: Documentation-only deprecation.
-->

类型：报废

使用非空非整数值`family`选项，非空值
的非数字值`hints`选项，非空值非布尔值`all`
选项，或 非空非布尔值`verbatim`选项
[`dns.lookup()`][dns.lookup()]和[`dnsPromises.lookup()`][dnsPromises.lookup()]抛出一个
`ERR_INVALID_ARG_TYPE`错误。

### DEP0154： RSA-PSS 生成密钥对选项

<!-- YAML
changes:
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/39927
    description: Documentation-only deprecation.
-->

类型：仅文档（支持[`--pending-deprecation`][--pending-deprecation])

这`'hash'`和`'mgf1Hash'`选项将替换为`'hashAlgorithm'`
和`'mgf1HashAlgorithm'`.

### DEP0155：模式说明符分辨率中的尾部斜杠

<!-- YAML
changes:
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/40117
    description: Runtime deprecation.
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/40039
    description: Documentation-only deprecation
                 with `--pending-deprecation` support.
-->

类型：运行时

以 结尾的说明符的重新映射`"/"`喜欢`import 'pkg/x/'`已弃用
用于包装`"exports"`和`"imports"`图案分辨率。

### DEP0156：`.aborted`财产和`'abort'`,`'aborted'`事件`http`

<!-- YAML
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/36670
    description: Documentation-only deprecation.
-->

类型：仅文档

改为移动到 {Stream} API，因为[`http.ClientRequest`][http.ClientRequest],
[`http.ServerResponse`][http.ServerResponse]和[`http.IncomingMessage`][http.IncomingMessage]都是基于流的。
检查`stream.destroyed`而不是`.aborted`属性，并侦听
`'close'`而不是`'abort'`,`'aborted'`事件。

这`.aborted`财产和`'abort'`事件仅用于检测
`.abort()`调用。要提前关闭请求，请使用流
`.destroy([error])`然后检查`.destroyed`财产和`'close'`事件
应该具有相同的效果。接收端还应检查
[`readable.readableEnded`][readable.readableEnded]值[`http.IncomingMessage`][http.IncomingMessage]以获取是否
这是一个流产或优雅的破坏。

### DEP0157：流中的可再转移支持

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/40773
    description: End-of-life.
  - version:
      - v17.2.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/40860
    description: Documentation-only deprecation.
-->

类型：报废

Node.js流的一个未记录的功能是支持
实现方法。这现已弃用，请改用回调并避免
将异步函数用于流实现方法。

此功能导致用户遇到意外问题，其中用户
以回调样式实现函数，但使用例如异步方法
会导致错误，因为混合承诺和回调语义是无效的。

```js
const w = new Writable({
  async final(callback) {
    await someOp();
    callback();
  }
});
```

### DEP0158：`buffer.slice(start, end)`

<!-- YAML
changes:
  - version:
    - v17.5.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41596
    description: Documentation-only deprecation.
-->

类型：仅文档

此方法已弃用，因为它与
`Uint8Array.prototype.slice()`，这是`Buffer`.

用[`buffer.subarray`][buffer.subarray]它做同样的事情。

### DEP0159：`ERR_INVALID_CALLBACK`

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: End-of-Life.
-->

类型：报废

此错误代码已被删除，因为增加了更多的混淆
用于值类型验证的错误。

### DEP0160：`process.on('multipleResolves', handler)`

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41896
    description: Runtime deprecation.
  - version:
    - v17.6.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41872
    description: Documentation-only deprecation.
-->

类型：运行时。

此事件已被弃用，因为它不适用于 V8 承诺组合器
这降低了它的有用性。

### DEP0161：`process._getActiveRequests()`和`process._getActiveHandles()`

<!-- YAML
changes:
  - version:
    - v17.6.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41587
    description: Documentation-only deprecation.
-->

类型：仅文档

这`process._getActiveHandles()`和`process._getActiveRequests()`
函数不供公众使用，将来可能会被删除
释放。

用[`process.getActiveResourcesInfo()`][process.getActiveResourcesInfo()]以获取活动类型的列表
资源，而不是实际的引用。

### DEP0162：`fs.write()`,`fs.writeFileSync()`强制到字符串

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42607
    description: Runtime deprecation.
  - version:
    - v17.8.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/42149
    description: Documentation-only deprecation.
-->

类型：运行时

具有自己的对象的隐式强制`toString`属性，作为第二个传递
参数[`fs.write()`][fs.write()],[`fs.writeFile()`][fs.writeFile()],[`fs.appendFile()`][fs.appendFile()],
[`fs.writeFileSync()`][fs.writeFileSync()]和[`fs.appendFileSync()`][fs.appendFileSync()]已弃用。
将它们转换为基元字符串。

### DEP0163：`channel.subscribe(onMessage)`,`channel.unsubscribe(onMessage)`

<!-- YAML
changes:
  - version: v18.7.0
    pr-url: https://github.com/nodejs/node/pull/42714
    description: Documentation-only deprecation.
-->

类型：仅文档

这些方法已被弃用，因为它们可以以不这样做的方式使用。
使通道引用保持活动状态足够长的时间以接收事件。

用[`diagnostics_channel.subscribe(name, onMessage)`][diagnostics_channel.subscribe(name, onMessage)]或
[`diagnostics_channel.unsubscribe(name, onMessage)`][diagnostics_channel.unsubscribe(name, onMessage)]这同样
相反。

### DEP0164：`process.exit([code])`强制到整数

<!-- YAML
changes:
  - version: v18.7.0
    pr-url: https://github.com/nodejs/node/pull/43738
    description: Documentation-only deprecation.
-->

类型：仅文档

`code`值`undefined`,`null`、整数和整数
字符串（例如，'1'）被弃用为参数[`process.exit()`][process.exit()].

### DEP0165：`--trace-atomics-wait`

<!-- YAML
changes:
  - version: REPLACEME
    pr-url: https://github.com/nodejs/node/pull/44093
    description: Documentation-only deprecation.
-->

类型：仅文档

这[`--trace-atomics-wait`][--trace-atomics-wait]标志已弃用。

[Legacy URL API]: url.md#legacy-url-api

[NIST SP 800-38D]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf

[RFC 6066]: https://tools.ietf.org/html/rfc6066#section-3

[WHATWG URL API]: url.md#the-whatwg-url-api

[`"exports"` or `"main"` entry]: packages.md#main-entry-point-export

[`--pending-deprecation`]: cli.md#--pending-deprecation

[`--throw-deprecation`]: cli.md#--throw-deprecation

[`--trace-atomics-wait`]: cli.md#--trace-atomics-wait

[`--unhandled-rejections`]: cli.md#--unhandled-rejectionsmode

[`Buffer.allocUnsafeSlow(size)`]: buffer.md#static-method-bufferallocunsafeslowsize

[`Buffer.from(array)`]: buffer.md#static-method-bufferfromarray

[`Buffer.from(buffer)`]: buffer.md#static-method-bufferfrombuffer

[`Buffer.isBuffer()`]: buffer.md#static-method-bufferisbufferobj

[`Cipher`]: crypto.md#class-cipher

[`Decipher`]: crypto.md#class-decipher

[`REPLServer.clearBufferedCommand()`]: repl.md#replserverclearbufferedcommand

[`ReadStream.open()`]: fs.md#class-fsreadstream

[`Server.getConnections()`]: net.md#servergetconnectionscallback

[`Server.listen({fd: <number>})`]: net.md#serverlistenhandle-backlog-callback

[`SlowBuffer`]: buffer.md#class-slowbuffer

[`WriteStream.open()`]: fs.md#class-fswritestream

[`assert`]: assert.md

[`asyncResource.runInAsyncScope()`]: async_context.md#asyncresourceruninasyncscopefn-thisarg-args

[`buffer.subarray`]: buffer.md#bufsubarraystart-end

[`child_process`]: child_process.md

[`clearInterval()`]: timers.md#clearintervaltimeout

[`clearTimeout()`]: timers.md#cleartimeouttimeout

[`console.error()`]: console.md#consoleerrordata-args

[`console.log()`]: console.md#consolelogdata-args

[`crypto.Certificate()` constructor]: crypto.md#legacy-api

[`crypto.DEFAULT_ENCODING`]: crypto.md#cryptodefault_encoding

[`crypto.createCipher()`]: crypto.md#cryptocreatecipheralgorithm-password-options

[`crypto.createCipheriv()`]: crypto.md#cryptocreatecipherivalgorithm-key-iv-options

[`crypto.createDecipher()`]: crypto.md#cryptocreatedecipheralgorithm-password-options

[`crypto.createDecipheriv()`]: crypto.md#cryptocreatedecipherivalgorithm-key-iv-options

[`crypto.fips`]: crypto.md#cryptofips

[`crypto.pbkdf2()`]: crypto.md#cryptopbkdf2password-salt-iterations-keylen-digest-callback

[`crypto.randomBytes()`]: crypto.md#cryptorandombytessize-callback

[`crypto.scrypt()`]: crypto.md#cryptoscryptpassword-salt-keylen-options-callback

[`decipher.final()`]: crypto.md#decipherfinaloutputencoding

[`decipher.setAuthTag()`]: crypto.md#deciphersetauthtagbuffer-encoding

[`diagnostics_channel.subscribe(name, onMessage)`]: diagnostics_channel.md#diagnostics_channelsubscribename-onmessage

[`diagnostics_channel.unsubscribe(name, onMessage)`]: diagnostics_channel.md#diagnostics_channelunsubscribename-onmessage

[`dns.lookup()`]: dns.md#dnslookuphostname-options-callback

[`dnsPromises.lookup()`]: dns.md#dnspromiseslookuphostname-options

[`domain`]: domain.md

[`ecdh.setPublicKey()`]: crypto.md#ecdhsetpublickeypublickey-encoding

[`emitter.listenerCount(eventName)`]: events.md#emitterlistenercounteventname

[`events.listenerCount(emitter, eventName)`]: events.md#eventslistenercountemitter-eventname

[`fs.FileHandle`]: fs.md#class-filehandle

[`fs.access()`]: fs.md#fsaccesspath-mode-callback

[`fs.appendFile()`]: fs.md#fsappendfilepath-data-options-callback

[`fs.appendFileSync()`]: fs.md#fsappendfilesyncpath-data-options

[`fs.createReadStream()`]: fs.md#fscreatereadstreampath-options

[`fs.createWriteStream()`]: fs.md#fscreatewritestreampath-options

[`fs.exists(path, callback)`]: fs.md#fsexistspath-callback

[`fs.lchmod(path, mode, callback)`]: fs.md#fslchmodpath-mode-callback

[`fs.lchmodSync(path, mode)`]: fs.md#fslchmodsyncpath-mode

[`fs.lchown(path, uid, gid, callback)`]: fs.md#fslchownpath-uid-gid-callback

[`fs.lchownSync(path, uid, gid)`]: fs.md#fslchownsyncpath-uid-gid

[`fs.read()`]: fs.md#fsreadfd-buffer-offset-length-position-callback

[`fs.readSync()`]: fs.md#fsreadsyncfd-buffer-offset-length-position

[`fs.stat()`]: fs.md#fsstatpath-options-callback

[`fs.write()`]: fs.md#fswritefd-buffer-offset-length-position-callback

[`fs.writeFile()`]: fs.md#fswritefilefile-data-options-callback

[`fs.writeFileSync()`]: fs.md#fswritefilesyncfile-data-options

[`http.ClientRequest`]: http.md#class-httpclientrequest

[`http.IncomingMessage`]: http.md#class-httpincomingmessage

[`http.ServerResponse`]: http.md#class-httpserverresponse

[`http.get()`]: http.md#httpgetoptions-callback

[`http.request()`]: http.md#httprequestoptions-callback

[`https.get()`]: https.md#httpsgetoptions-callback

[`https.request()`]: https.md#httpsrequestoptions-callback

[`message.connection`]: http.md#messageconnection

[`message.socket`]: http.md#messagesocket

[`module.createRequire()`]: module.md#modulecreaterequirefilename

[`os.networkInterfaces()`]: os.md#osnetworkinterfaces

[`os.tmpdir()`]: os.md#ostmpdir

[`process.env`]: process.md#processenv

[`process.exit()`]: process.md#processexitcode

[`process.getActiveResourcesInfo()`]: process.md#processgetactiveresourcesinfo

[`process.mainModule`]: process.md#processmainmodule

[`punycode`]: punycode.md

[`readable.readableEnded`]: stream.md#readablereadableended

[`request.abort()`]: http.md#requestabort

[`request.connection`]: http.md#requestconnection

[`request.destroy()`]: http.md#requestdestroyerror

[`request.socket`]: http.md#requestsocket

[`require.extensions`]: modules.md#requireextensions

[`require.main`]: modules.md#accessing-the-main-module

[`response.connection`]: http.md#responseconnection

[`response.end()`]: http.md#responseenddata-encoding-callback

[`response.finished`]: http.md#responsefinished

[`response.socket`]: http.md#responsesocket

[`response.writableEnded`]: http.md#responsewritableended

[`response.writableFinished`]: http.md#responsewritablefinished

[`script.createCachedData()`]: vm.md#scriptcreatecacheddata

[`setInterval()`]: timers.md#setintervalcallback-delay-args

[`setTimeout()`]: timers.md#settimeoutcallback-delay-args

[`socket.bufferSize`]: net.md#socketbuffersize

[`timeout.ref()`]: timers.md#timeoutref

[`timeout.refresh()`]: timers.md#timeoutrefresh

[`timeout.unref()`]: timers.md#timeoutunref

[`tls.CryptoStream`]: tls.md#class-tlscryptostream

[`tls.SecureContext`]: tls.md#tlscreatesecurecontextoptions

[`tls.SecurePair`]: tls.md#class-tlssecurepair

[`tls.TLSSocket`]: tls.md#class-tlstlssocket

[`tls.checkServerIdentity()`]: tls.md#tlscheckserveridentityhostname-cert

[`tls.createSecureContext()`]: tls.md#tlscreatesecurecontextoptions

[`url.format()`]: url.md#urlformaturlobject

[`url.parse()`]: url.md#urlparseurlstring-parsequerystring-slashesdenotehost

[`url.resolve()`]: url.md#urlresolvefrom-to

[`util._extend()`]: util.md#util_extendtarget-source

[`util.getSystemErrorName()`]: util.md#utilgetsystemerrornameerr

[`util.inspect()`]: util.md#utilinspectobject-options

[`util.inspect.custom`]: util.md#utilinspectcustom

[`util.isArray()`]: util.md#utilisarrayobject

[`util.isBoolean()`]: util.md#utilisbooleanobject

[`util.isBuffer()`]: util.md#utilisbufferobject

[`util.isDate()`]: util.md#utilisdateobject

[`util.isError()`]: util.md#utiliserrorobject

[`util.isFunction()`]: util.md#utilisfunctionobject

[`util.isNull()`]: util.md#utilisnullobject

[`util.isNullOrUndefined()`]: util.md#utilisnullorundefinedobject

[`util.isNumber()`]: util.md#utilisnumberobject

[`util.isObject()`]: util.md#utilisobjectobject

[`util.isPrimitive()`]: util.md#utilisprimitiveobject

[`util.isRegExp()`]: util.md#utilisregexpobject

[`util.isString()`]: util.md#utilisstringobject

[`util.isSymbol()`]: util.md#utilissymbolobject

[`util.isUndefined()`]: util.md#utilisundefinedobject

[`util.log()`]: util.md#utillogstring

[`util.types`]: util.md#utiltypes

[`util`]: util.md

[`worker.exitedAfterDisconnect`]: cluster.md#workerexitedafterdisconnect

[`worker.terminate()`]: worker_threads.md#workerterminate

[`writable.writableLength`]: stream.md#writablewritablelength

[`zlib.bytesWritten`]: zlib.md#zlibbyteswritten

[alloc]: buffer.md#static-method-bufferallocsize-fill-encoding

[alloc_unsafe_size]: buffer.md#static-method-bufferallocunsafesize

[from_arraybuffer]: buffer.md#static-method-bufferfromarraybuffer-byteoffset-length

[from_string_encoding]: buffer.md#static-method-bufferfromstring-encoding

[legacy `urlObject`]: url.md#legacy-urlobject

[static methods of `crypto.Certificate()`]: crypto.md#class-certificate

[subpath exports]: packages.md#subpath-exports

[subpath imports]: packages.md#subpath-imports

[subpath patterns]: packages.md#subpath-patterns
