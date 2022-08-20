# 网

<!--introduced_in=v0.10.0-->

<!--lint disable maximum-line-length-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/net.js -->

这`node:net`模块提供了一个异步网络 API，用于创建基于流的
TCP 或[工控机][IPC]服务器 （[`net.createServer()`][net.createServer()]） 和客户端
([`net.createConnection()`][net.createConnection()]).

可以使用以下命令访问它：

```js
const net = require('node:net');
```

## 工控机支持

这`node:net`模块支持在 Windows 和 Unix 域上使用命名管道的 IPC
其他操作系统上的套接字。

### 标识 IPC 连接的路径

[`net.connect()`][net.connect()],[`net.createConnection()`][net.createConnection()],[`server.listen()`][server.listen()]和
[`socket.connect()`][socket.connect()]采取`path`参数来标识 IPC 端点。

在 Unix 上，本地域也称为 Unix 域。路径是
文件系统路径名。它被截断为与操作系统相关的长度
`sizeof(sockaddr_un.sun_path) - 1`.在 Linux 上，典型值为 107 字节，并且
macOS 上的 103 字节。如果一个 Node.js API 抽象创建了 Unix 域套接字，
它还将取消链接Unix域套接字。例如
[`net.createServer()`][net.createServer()]可以创建一个 Unix 域套接字和
[`server.close()`][server.close()]将取消链接它。但是，如果用户创建了Unix域
套接字在这些抽象之外，用户将需要删除它。一样
当 Node.js API 创建 Unix 域套接字但程序随后
崩溃。简而言之，Unix域套接字将在文件系统中可见，并且
将一直持续到取消链接。

在 Windows 上，本地域是使用命名管道实现的。路径*必须*
引用`\\?\pipe\`或`\\.\pipe\`.允许使用任何字符，
但后者可能会对管道名称进行一些处理，例如解析`..`
序列。无论它看起来如何，管道命名空间都是平面的。管道将
*不持久*.当关闭对它们的最后一个引用时，它们将被删除。
与Unix域套接字不同，Windows将在以下情况下关闭并删除管道：
拥有进程退出。

JavaScript 字符串转义需要使用额外的反斜杠指定路径
转义，例如：

```js
net.createServer().listen(
  path.join('\\\\?\\pipe', process.cwd(), 'myctl'));
```

## 类：`net.BlockList`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

这`BlockList`对象可以与某些网络 API 一起使用，以指定规则
禁用对特定 IP 地址、IP 范围的入站或出站访问，或
IP 子网。

### `blockList.addAddress(address[, type])`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

*   `address`{字符串|网.套接字地址} IPv4 或 IPv6 地址。
*   `type`{字符串}也`'ipv4'`或`'ipv6'`.**违约：** `'ipv4'`.

添加规则以阻止给定的 IP 地址。

### `blockList.addRange(start, end[, type])`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

*   `start`{字符串|网.套接字地址}
    范围。
*   `end`{字符串|网.套接字地址} 范围中的结束 IPv4 或 IPv6 地址。
*   `type`{字符串}也`'ipv4'`或`'ipv6'`.**违约：** `'ipv4'`.

添加规则以阻止来自`start`（含）至
`end`（包括）。

### `blockList.addSubnet(net, prefix[, type])`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

*   `net`{字符串|网.套接字地址} 网络 IPv4 或 IPv6 地址。
*   `prefix`{数字}CIDR 前缀位的数量。对于 IPv4，此
    必须是介于`0`和`32`.对于 IPv6，这必须在
    `0`和`128`.
*   `type`{字符串}也`'ipv4'`或`'ipv6'`.**违约：** `'ipv4'`.

添加规则以阻止指定为子网掩码的 IP 地址范围。

### `blockList.check(address[, type])`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

*   `address`{字符串|网.套接字地址} 要检查的 IP 地址
*   `type`{字符串}也`'ipv4'`或`'ipv6'`.**违约：** `'ipv4'`.
*   返回：{布尔值}

返回`true`如果给定的 IP 地址与添加到
`BlockList`.

```js
const blockList = new net.BlockList();
blockList.addAddress('123.123.123.123');
blockList.addRange('10.0.0.1', '10.0.0.10');
blockList.addSubnet('8592:757c:efae:4e45::', 64, 'ipv6');

console.log(blockList.check('123.123.123.123'));  // Prints: true
console.log(blockList.check('10.0.0.3'));  // Prints: true
console.log(blockList.check('222.111.111.222'));  // Prints: false

// IPv6 notation for IPv4 addresses works:
console.log(blockList.check('::ffff:7b7b:7b7b', 'ipv6')); // Prints: true
console.log(blockList.check('::ffff:123.123.123.123', 'ipv6')); // Prints: true
```

### `blockList.rules`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

*   类型： {字符串\[]}

添加到黑名单的规则列表。

## 类：`net.SocketAddress`

<!-- YAML
added:
  - v15.14.0
  - v14.18.0
-->

### `new net.SocketAddress([options])`

<!-- YAML
added:
  - v15.14.0
  - v14.18.0
-->

*   `options`{对象}
    *   `address`{字符串}以 IPv4 或 IPv6 字符串表示的网络地址。
        **违约**:`'127.0.0.1'`如果`family`是`'ipv4'`;`'::'`如果`family`是
        `'ipv6'`.
    *   `family`{字符串}其中之一`'ipv4'`或`'ipv6'`.
        **违约**:`'ipv4'`.
    *   `flowlabel`{数字}仅在以下情况下使用的 IPv6 流标签`family`是`'ipv6'`.
    *   `port`{数字}一个 IP 端口。

### `socketaddress.address`

<!-- YAML
added:
  - v15.14.0
  - v14.18.0
-->

*   键入 {字符串}

### `socketaddress.family`

<!-- YAML
added:
  - v15.14.0
  - v14.18.0
-->

*   键入 {字符串} 任一`'ipv4'`或`'ipv6'`.

### `socketaddress.flowlabel`

<!-- YAML
added:
  - v15.14.0
  - v14.18.0
-->

*   键入 {数字}

### `socketaddress.port`

<!-- YAML
added:
  - v15.14.0
  - v14.18.0
-->

*   键入 {数字}

## 类：`net.Server`

<!-- YAML
added: v0.1.90
-->

*   扩展：{事件发射器}

此类用于创建 TCP 或[工控机][IPC]服务器。

### `new net.Server([options][, connectionListener])`

*   `options`{对象}看
    [`net.createServer([options][, connectionListener])`][`net.createServer()`].
*   `connectionListener`{函数}自动设置为
    [`'connection'`]['connection']事件。
*   返回：{净。服务器}

`net.Server`是一个[`EventEmitter`][EventEmitter]出现以下事件：

### 事件：`'close'`

<!-- YAML
added: v0.5.0
-->

服务器关闭时发出。如果存在连接，则
在所有连接结束之前，不会发出事件。

### 事件：`'connection'`

<!-- YAML
added: v0.1.90
-->

*   {网.套接字} 连接对象

建立新连接时发出。`socket`是 的实例
`net.Socket`.

### 事件：`'error'`

<!-- YAML
added: v0.1.90
-->

*   {错误}

发生错误时发出。与[`net.Socket`][net.Socket]这[`'close'`]['close']
事件将**不**在此事件之后直接发出，除非
[`server.close()`][server.close()]被手动调用。请参阅讨论中的示例
[`server.listen()`][server.listen()].

### 事件：`'listening'`

<!-- YAML
added: v0.1.90
-->

调用后绑定服务器时发出[`server.listen()`][server.listen()].

### 事件：`'drop'`

<!-- YAML
added: v18.6.0
-->

当连接数达到阈值`server.maxConnections`,
服务器将断开新连接并发出`'drop'`事件。如果是
TCP 服务器，参数如下所示，否则参数为`undefined`.

*   `data`{对象}传递给事件侦听器的参数。
    *   `localAddress`{字符串} 本地地址。
    *   `localPort`{数字}本地端口。
    *   `localFamily`{字符串}当地家庭。
    *   `remoteAddress`{字符串}远程地址。
    *   `remotePort`{数字}远程端口。
    *   `remoteFamily`{字符串}远程 IP 系列。`'IPv4'`或`'IPv6'`.

### `server.address()`

<!-- YAML
added: v0.1.90
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43054
    description: The `family` property now returns a string instead of a number.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41431
    description: The `family` property now returns a number instead of a string.
-->

*   返回：{对象|字符串|null}

返回绑定`address`，地址`family`名称，以及`port`的服务器
如果侦听 IP 套接字，则由操作系统报告
（用于查找在获取操作系统分配的地址时分配的端口）：
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`.

对于侦听管道或 Unix 域套接字的服务器，将返回名称
作为字符串。

```js
const server = net.createServer((socket) => {
  socket.end('goodbye\n');
}).on('error', (err) => {
  // Handle errors here.
  throw err;
});

// Grab an arbitrary unused port.
server.listen(() => {
  console.log('opened server on', server.address());
});
```

`server.address()`返回`null`在`'listening'`事件已
发出或呼叫后`server.close()`.

### `server.close([callback])`

<!-- YAML
added: v0.1.90
-->

*   `callback`{函数}在服务器关闭时调用。
*   返回：{净。服务器}

阻止服务器接受新连接并保持现有
连接。此函数是异步的，服务器最终关闭
当所有连接都结束并且服务器发出[`'close'`]['close']事件。
可选`callback`将被调用一旦`'close'`事件发生。与
该事件，它将被调用`Error`作为服务器的唯一参数
关闭时未打开。

### `server.getConnections(callback)`

<!-- YAML
added: v0.9.7
-->

*   `callback`{函数}
*   返回：{净。服务器}

异步获取服务器上的并发连接数。工程
当套接字被发送到分叉时。

回调应采用两个参数`err`和`count`.

### `server.listen()`

启动服务器侦听连接。一个`net.Server`可以是 TCP 或
一[工控机][IPC]服务器取决于它侦听的内容。

可能的签名：

*   [`server.listen(handle[, backlog][, callback])`][`server.listen(handle)`]
*   [`server.listen(options[, callback])`][`server.listen(options)`]
*   [`server.listen(path[, backlog][, callback])`][`server.listen(path)`]
    为[工控机][IPC]服务器
*   [`server.listen([port[, host[, backlog]]][, callback])`][`server.listen(port)`]
    对于 TCP 服务器

此函数是异步的。当服务器开始侦听时，
[`'listening'`]['listening']将发出事件。最后一个参数`callback`
将被添加为[`'listening'`]['listening']事件。

都`listen()`方法可以采取`backlog`参数以指定最大值
挂起连接队列的长度。实际长度将确定
通过操作系统通过系统设置，例如`tcp_max_syn_backlog`和`somaxconn`
在 Linux 上。此参数的默认值为 511（而不是 512）。

都[`net.Socket`][net.Socket]设置为`SO_REUSEADDR`（请参见[`socket(7)`][socket(7)]为
详细信息）。

这`server.listen()`当且仅当存在
第一个期间的错误`server.listen()`呼叫或`server.close()`已经
叫。否则，一个`ERR_SERVER_ALREADY_LISTEN`将引发错误。

收听时最常见的错误之一是`EADDRINUSE`.
当另一台服务器已经在侦听请求的服务器时，就会发生这种情况。
`port`/`path`/`handle`.处理此问题的一种方法是重试
经过一定时间后：

```js
server.on('error', (e) => {
  if (e.code === 'EADDRINUSE') {
    console.log('Address in use, retrying...');
    setTimeout(() => {
      server.close();
      server.listen(PORT, HOST);
    }, 1000);
  }
});
```

#### `server.listen(handle[, backlog][, callback])`

<!-- YAML
added: v0.5.10
-->

*   `handle`{对象}
*   `backlog`{数字}常用参数[`server.listen()`][server.listen()]功能
*   `callback`{函数}
*   返回：{净。服务器}

启动服务器侦听给定的连接`handle`有
已绑定到端口、Unix 域套接字或 Windows 命名管道。

这`handle`对象可以是服务器，套接字（任何带有
底层`_handle`成员），或具有`fd`成员是
有效的文件描述符。

在 Windows 上不支持侦听文件描述符。

#### `server.listen(options[, callback])`

<!-- YAML
added: v0.11.14
changes:
  - version: v15.6.0
    pr-url: https://github.com/nodejs/node/pull/36623
    description: AbortSignal support was added.
  - version: v11.4.0
    pr-url: https://github.com/nodejs/node/pull/23798
    description: The `ipv6Only` option is supported.
-->

*   `options`{对象}必填。支持以下属性：
    *   `port`{数字}
    *   `host`{字符串}
    *   `path`{字符串}如果出现以下情况，将被忽略`port`已指定。看
        [标识 IPC 连接的路径][Identifying paths for IPC connections].
    *   `backlog`{数字}常用参数[`server.listen()`][server.listen()]
        功能。
    *   `exclusive`{布尔值}**违约：** `false`
    *   `readableAll`{布尔值}对于IPC服务器，使管道可读
        面向所有用户。**违约：** `false`.
    *   `writableAll`{布尔值}对于IPC服务器，使管道可写
        面向所有用户。**违约：** `false`.
    *   `ipv6Only`{布尔值}对于 TCP 服务器，设置`ipv6Only`自`true`将
        禁用双栈支持，即绑定到主机`::`不会使
        `0.0.0.0`被束缚。**违约：** `false`.
    *   `signal`{中止信号}可用于关闭侦听服务器的中止信号。
*   `callback`{函数}
    功能。
*   返回：{净。服务器}

如果`port`，其行为与
[`server.listen([port[, host[, backlog]]][, callback])`][`server.listen(port)`].
否则，如果`path`，其行为与
[`server.listen(path[, backlog][, callback])`][`server.listen(path)`].
如果未指定任何一个，则将引发错误。

如果`exclusive`是`false`（默认值），则群集辅助角色将使用相同的
基础句柄，允许共享连接处理职责。什么时候
`exclusive`是`true`，句柄未共享，并尝试端口共享
导致错误。侦听独占端口的示例是
所 示。

```js
server.listen({
  host: 'localhost',
  port: 80,
  exclusive: true
});
```

什么时候`exclusive`是`true`并且底层句柄是共享的，它是
可能多个工作线程查询具有不同积压工作的句柄。
在这种情况下，第一个`backlog`传递给主进程将被使用。

以 root 用户身份启动 IPC 服务器可能会导致 服务器路径无法访问
非特权用户。用`readableAll`和`writableAll`将使服务器
可供所有用户访问。

如果`signal`选项已启用，正在调用`.abort()`在相应的
`AbortController`类似于呼叫`.close()`在服务器上：

```js
const controller = new AbortController();
server.listen({
  host: 'localhost',
  port: 80,
  signal: controller.signal
});
// Later, when you want to close the server.
controller.abort();
```

#### `server.listen(path[, backlog][, callback])`

<!-- YAML
added: v0.1.90
-->

*   `path`{字符串}服务器应侦听的路径。看
    [标识 IPC 连接的路径][Identifying paths for IPC connections].
*   `backlog`{数字}常用参数[`server.listen()`][server.listen()]功能。
*   `callback`{函数}.
*   返回：{净。服务器}

启动[工控机][IPC]服务器侦听给定的连接`path`.

#### `server.listen([port[, host[, backlog]]][, callback])`

<!-- YAML
added: v0.1.90
-->

*   `port`{数字}
*   `host`{字符串}
*   `backlog`{数字}常用参数[`server.listen()`][server.listen()]功能。
*   `callback`{函数}.
*   返回：{净。服务器}

启动 TCP 服务器侦听给定服务器上的连接`port`和`host`.

如果`port`省略或为 0，操作系统将分配任意
未使用的端口，可以使用`server.address().port`
在[`'listening'`]['listening']事件已发出。

如果`host`省略，服务器将接受
[未指定的 IPv6 地址][unspecified IPv6 address]\(`::`） 时，或
[未指定的 IPv4 地址][unspecified IPv4 address]\(`0.0.0.0`），否则。

在大多数操作系统中，侦听[未指定的 IPv6 地址][unspecified IPv6 address]\(`::`)
可能导致`net.Server`也要收听[未指定的 IPv4 地址][unspecified IPv4 address]
(`0.0.0.0`).

### `server.listening`

<!-- YAML
added: v5.7.0
-->

*   {布尔值}指示服务器是否正在侦听连接。

### `server.maxConnections`

<!-- YAML
added: v0.2.0
-->

*   {整数}

将此属性设置为在服务器的连接计数获得时拒绝连接
高。

不建议在将套接字发送给孩子后使用此选项
跟[`child_process.fork()`][child_process.fork()].

### `server.ref()`

<!-- YAML
added: v0.9.1
-->

*   返回：{净。服务器}

相反`unref()`叫`ref()`在以前的`unref`ed 服务器将
*不*如果程序是唯一剩下的服务器，则让程序退出（默认行为）。
如果服务器是`ref`ed 呼叫`ref()`再也不会有效果。

### `server.unref()`

<!-- YAML
added: v0.9.1
-->

*   返回：{净。服务器}

叫`unref()`将允许程序退出，如果这是唯一的
事件系统中的活动服务器。如果服务器已经`unref`ed 呼叫
`unref()`再也不会有效果。

## 类：`net.Socket`

<!-- YAML
added: v0.3.4
-->

*   扩展：{流。双工}

此类是 TCP 套接字或流式处理的抽象[工控机][IPC]端点
（在 Windows 上使用命名管道，否则使用 Unix 域套接字）。它也是
一[`EventEmitter`][EventEmitter].

一个`net.Socket`可以由用户创建并直接用于交互
服务器。例如，它由[`net.createConnection()`][net.createConnection()],
因此，用户可以使用它与服务器通信。

它也可以由Node创建.js并在连接时传递给用户
已接收。例如，它被传递给
[`'connection'`]['connection']在 上发出的事件[`net.Server`][net.Server]，以便用户可以使用
它与客户互动。

### `new net.Socket([options])`

<!-- YAML
added: v0.3.4
changes:
  - version: v15.14.0
    pr-url: https://github.com/nodejs/node/pull/37735
    description: AbortSignal support was added.
-->

*   `options`{对象}可用选项包括：
    *   `fd`{数字}如果指定，请用
        给定的文件描述符，否则将创建一个新的套接字。
    *   `allowHalfOpen`{布尔值}如果设置为`false`，则套接字将
        当可读侧结束时，自动结束可写侧。看
        [`net.createServer()`][net.createServer()]和[`'end'`]['end']事件以了解详细信息。**违约：**
        `false`.
    *   `readable`{布尔值}允许在套接字上读取`fd`已通过，
        否则将被忽略。**违约：** `false`.
    *   `writable`{布尔值}允许在套接字上写入`fd`已通过，
        否则将被忽略。**违约：** `false`.
    *   `signal`{中止信号}可用于销毁
        插座。
*   返回：{净。插座}

创建新的套接字对象。

新创建的套接字可以是 TCP 套接字，也可以是流式处理[工控机][IPC]
端点，取决于它是什么[`connect()`][`socket.connect()`]自。

### 事件：`'close'`

<!-- YAML
added: v0.1.90
-->

*   `hadError`{布尔值}`true`如果套接字出现传输错误。

插座完全关闭后发出。参数`hadError`是一个布尔值
它表示插座是否由于传输错误而关闭。

### 事件：`'connect'`

<!-- YAML
added: v0.1.90
-->

成功建立套接字连接时发出。
看[`net.createConnection()`][net.createConnection()].

### 事件：`'data'`

<!-- YAML
added: v0.1.90
-->

*   {Buffer|string}

收到数据时发出。参数`data`将是一个`Buffer`或
`String`.数据的编码由下式设置[`socket.setEncoding()`][socket.setEncoding()].

如果没有侦听器，则数据将丢失，当`Socket`
发出`'data'`事件。

### 事件：`'drain'`

<!-- YAML
added: v0.1.90
-->

当写入缓冲区变为空时发出。可用于限制上载。

另请参见：返回值`socket.write()`.

### 事件：`'end'`

<!-- YAML
added: v0.1.90
-->

当插座的另一端发出信号时发出传输结束信号，因此
结束套接字的可读侧。

默认情况下（`allowHalfOpen`是`false`） 套接字将发送结束
将数据包传回，并在写出后销毁其文件描述符
其挂起的写入队列。但是，如果`allowHalfOpen`设置为`true`这
套接字不会自动[`end()`][`socket.end()`]其可写面，
允许用户写入任意数量的数据。用户必须调用
[`end()`][`socket.end()`]显式关闭连接（即发送
FIN 数据包返回）。

### 事件：`'error'`

<!-- YAML
added: v0.1.90
-->

*   {错误}

发生错误时发出。这`'close'`事件将直接调用
在此事件之后。

### 事件：`'lookup'`

<!-- YAML
added: v0.11.3
changes:
  - version: v5.10.0
    pr-url: https://github.com/nodejs/node/pull/5598
    description: The `host` parameter is supported now.
-->

在解析主机名后但在连接之前发出。
不适用于 Unix 套接字。

*   `err`{错误|空}错误对象。看[`dns.lookup()`][dns.lookup()].
*   `address`{字符串}IP 地址。
*   `family`{数字|空}地址类型。看[`dns.lookup()`][dns.lookup()].
*   `host`{字符串}主机名。

### 事件：`'ready'`

<!-- YAML
added: v9.11.0
-->

当套接字准备好使用时发出。

紧接后触发`'connect'`.

### 事件：`'timeout'`

<!-- YAML
added: v0.1.90
-->

如果套接字因不活动而超时，则发出。这只是为了通知
套接字已空闲。用户必须手动关闭连接。

另请参阅：[`socket.setTimeout()`][socket.setTimeout()].

### `socket.address()`

<!-- YAML
added: v0.1.90
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43054
    description: The `family` property now returns a string instead of a number.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41431
    description: The `family` property now returns a number instead of a string.
-->

*   返回： {对象}

返回绑定`address`，地址`family`名称和`port`的
操作系统报告的套接字：
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

### `socket.bufferSize`

<!-- YAML
added: v0.3.8
deprecated:
  - v14.6.0
-->

> 稳定性：0 - 已弃用：使用[`writable.writableLength`][writable.writableLength]相反。

*   {整数}

此属性显示缓冲以进行写入的字符数。缓冲区
可能包含编码后长度尚不得而知的字符串。所以这个数字
只是缓冲区中字节数的近似值。

`net.Socket`具有以下特性：`socket.write()`总是有效。这是为了
帮助用户快速启动和运行。计算机无法始终跟上
与写入套接字的数据量。网络连接
只是可能太慢了。节点.js将在内部将写入的数据排队到
插座，并在可能的情况下通过电线将其发送出去。

这种内部缓冲的结果是内存可能会增长。
体验大型或不断增长的用户`bufferSize`应该尝试
“限制”其程序中的数据流
[`socket.pause()`][socket.pause()]和[`socket.resume()`][socket.resume()].

### `socket.bytesRead`

<!-- YAML
added: v0.5.3
-->

*   {整数}

接收的字节数。

### `socket.bytesWritten`

<!-- YAML
added: v0.5.3
-->

*   {整数}

发送的字节数。

### `socket.connect()`

在给定套接字上启动连接。

可能的签名：

*   [`socket.connect(options[, connectListener])`][`socket.connect(options)`]
*   [`socket.connect(path[, connectListener])`][`socket.connect(path)`]
    为[工控机][IPC]连接。
*   [`socket.connect(port[, host][, connectListener])`][`socket.connect(port)`]
    用于 TCP 连接。
*   返回：{净。套接字} 套接字本身。

此函数是异步的。建立连接后，
[`'connect'`]['connect']将发出事件。如果连接有问题，
而不是[`'connect'`]['connect']事件，[`'error'`]['error']事件将发出
传递给[`'error'`]['error']听者。
最后一个参数`connectListener`，如果提供，将被添加为侦听器
对于[`'connect'`]['connect']事件**一次**.

此功能应仅用于在以下情况下重新连接套接字
`'close'`已发出或以其他方式可能导致未定义
行为。

#### `socket.connect(options[, connectListener])`

<!-- YAML
added: v0.1.90
changes:
  - version:
    - v17.7.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41310
    description: The `noDelay`, `keepAlive` and `keepAliveInitialDelay`
                 options are supported now.
  - version: v12.10.0
    pr-url: https://github.com/nodejs/node/pull/25436
    description: Added `onread` option.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6021
    description: The `hints` option defaults to `0` in all cases now.
                 Previously, in the absence of the `family` option it would
                 default to `dns.ADDRCONFIG | dns.V4MAPPED`.
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/6000
    description: The `hints` option is supported now.
-->

*   `options`{对象}
*   `connectListener`{函数}常用参数[`socket.connect()`][socket.connect()]
    方法。将被添加为 的侦听器[`'connect'`]['connect']事件一次。
*   返回：{净。套接字} 套接字本身。

在给定套接字上启动连接。通常不需要这种方法，
套接字应创建并打开[`net.createConnection()`][net.createConnection()].用
这仅在实现自定义套接字时。

对于 TCP 连接，可用`options`是：

*   `port`{数字}必填。套接字应连接到的端口。
*   `host`{字符串}套接字应连接到的主机。**违约：** `'localhost'`.
*   `localAddress`{字符串}套接字应从本地地址进行连接。
*   `localPort`{数字}套接字应从本地端口进行连接。
*   `family`{数字}：IP 堆栈的版本。必须是`4`,`6`或`0`.价值
    `0`表示同时允许 IPv4 和 IPv6 地址。**违约：** `0`.
*   `hints`{数字}自选[`dns.lookup()`提示][dns.lookup() hints].
*   `lookup`{函数}自定义查找功能。**违约：** [`dns.lookup()`][dns.lookup()].
*   `noDelay`{布尔值}如果设置为`true`，它立即禁用了Nagle算法的使用
    建立套接字后。**违约：** `false`.
*   `keepAlive`{布尔值}如果设置为`true`，它支持套接字上的保持活动状态功能
    在建立连接后立即，类似地在
    [`socket.setKeepAlive([enable][, initialDelay])`][`socket.setKeepAlive(enable, initialDelay)`].
    **违约：** `false`.
*   `keepAliveInitialDelay`{数字}如果设置为正数，则将初始延迟设置为
    第一个保持活动探测器在空闲套接字上发送。**违约：** `0`.

为[工控机][IPC]连接， 可用`options`是：

*   `path`{字符串}必填。客户端应连接到的路径。
    看[标识 IPC 连接的路径][Identifying paths for IPC connections].如果提供，则特定于 TCP
    上面的选项将被忽略。

对于这两种类型，都可用`options`包括：

*   `onread`{对象}如果指定，传入数据存储在单个中`buffer`
    并传递给提供的`callback`当数据到达套接字时。
    这将导致流式处理功能不提供任何数据。
    套接字将发出类似`'error'`,`'end'`和`'close'`
    照常。像这样的方法`pause()`和`resume()`也会表现为
    预期。
    *   `buffer`{缓冲区|Uint8Array|函数} 可重用的内存块
        用于存储传入数据或返回此类数据的函数。
    *   `callback`{函数}为每个传入块调用此函数
        数据。向它传递两个参数：写入的字节数
        `buffer`和引用`buffer`.返回`false`从此函数到
        隐 式`pause()`套接字。此函数将在
        全球背景。

下面是一个客户端使用`onread`选择：

```js
const net = require('node:net');
net.connect({
  port: 80,
  onread: {
    // Reuses a 4KiB Buffer for every read from the socket.
    buffer: Buffer.alloc(4 * 1024),
    callback: function(nread, buf) {
      // Received data is available in `buf` from 0 to `nread`.
      console.log(buf.toString('utf8', 0, nread));
    }
  }
});
```

#### `socket.connect(path[, connectListener])`

*   `path`{字符串}客户端应连接到的路径。看
    [标识 IPC 连接的路径][Identifying paths for IPC connections].
*   `connectListener`{函数}常用参数[`socket.connect()`][socket.connect()]
    方法。将被添加为 的侦听器[`'connect'`]['connect']事件一次。
*   返回：{净。套接字} 套接字本身。

启动[工控机][IPC]给定套接字上的连接。

别名
[`socket.connect(options[, connectListener])`][`socket.connect(options)`]
调用`{ path: path }`如`options`.

#### `socket.connect(port[, host][, connectListener])`

<!-- YAML
added: v0.1.90
-->

*   `port`{数字}客户端应连接到的端口。
*   `host`{字符串}客户端应连接到的主机。
*   `connectListener`{函数}常用参数[`socket.connect()`][socket.connect()]
    方法。将被添加为 的侦听器[`'connect'`]['connect']事件一次。
*   返回：{净。套接字} 套接字本身。

在给定套接字上启动 TCP 连接。

别名
[`socket.connect(options[, connectListener])`][`socket.connect(options)`]
调用`{port: port, host: host}`如`options`.

### `socket.connecting`

<!-- YAML
added: v6.1.0
-->

*   {布尔值}

如果`true`,
[`socket.connect(options[, connectListener])`][`socket.connect(options)`]是
已调用，但尚未完成。它会留下来`true`直到套接字变为
已连接，然后将其设置为`false`和`'connect'`发出事件。注意
该
[`socket.connect(options[, connectListener])`][`socket.connect(options)`]
回调是`'connect'`事件。

### `socket.destroy([error])`

<!-- YAML
added: v0.1.90
-->

*   `error`{对象}
*   返回：{净。插座}

确保此套接字上不再发生 I/O 活动。
销毁流并关闭连接。

看[`writable.destroy()`][writable.destroy()]了解更多详情。

### `socket.destroyed`

*   {布尔值}指示连接是否被破坏。曾经
    连接被破坏，不能再使用它传输数据。

看[`writable.destroyed`][writable.destroyed]了解更多详情。

### `socket.end([data[, encoding]][, callback])`

<!-- YAML
added: v0.1.90
-->

*   `data`{字符串|缓冲区|Uint8Array}
*   `encoding`{字符串}仅当数据`string`.**违约：** `'utf8'`.
*   `callback`{函数}套接字完成时的可选回调。
*   返回：{净。套接字} 套接字本身。

半关闭插槽。即，它发送 FIN 数据包。有可能
服务器仍将发送一些数据。

看[`writable.end()`][writable.end()]了解更多详情。

### `socket.localAddress`

<!-- YAML
added: v0.9.6
-->

*   {字符串}

远程客户端的本地 IP 地址的字符串表示形式
连接。例如，在服务器上侦听`'0.0.0.0'`，如果客户端
连接于`'192.168.1.1'`，的值`socket.localAddress`将
`'192.168.1.1'`.

### `socket.localPort`

<!-- YAML
added: v0.9.6
-->

*   {整数}

本地端口的数字表示形式。例如`80`或`21`.

### `socket.localFamily`

<!-- YAML
added: REPLACEME
-->

*   {字符串}

本地 IP 系列的字符串表示形式。`'IPv4'`或`'IPv6'`.

### `socket.pause()`

*   返回：{净。套接字} 套接字本身。

暂停读取数据。那是[`'data'`]['data']事件将不会发出。
用于限制上传。

### `socket.pending`

<!-- YAML
added:
 - v11.2.0
 - v10.16.0
-->

*   {布尔值}

这是`true`如果套接字尚未连接，则因为`.connect()`
尚未被调用或因为它仍在连接过程中
（请参见[`socket.connecting`][socket.connecting]).

### `socket.ref()`

<!-- YAML
added: v0.9.1
-->

*   返回：{净。套接字} 套接字本身。

相反`unref()`叫`ref()`在以前的`unref`ed 插座将
*不*如果程序是唯一剩下的套接字（默认行为），则让程序退出。
如果套接字是`ref`ed 呼叫`ref`再也不会有效果。

### `socket.remoteAddress`

<!-- YAML
added: v0.5.10
-->

*   {字符串}

远程 IP 地址的字符串表示形式。例如
`'74.125.127.100'`或`'2001:4860:a005::68'`.值可能是`undefined`如果
套接字被销毁（例如，如果客户端断开连接）。

### `socket.remoteFamily`

<!-- YAML
added: v0.11.14
-->

*   {字符串}

远程 IP 系列的字符串表示形式。`'IPv4'`或`'IPv6'`.

### `socket.remotePort`

<!-- YAML
added: v0.5.10
-->

*   {整数}

远程端口的数字表示形式。例如`80`或`21`.

### `socket.resetAndDestroy()`

<!-- YAML
added: v18.3.0
-->

*   返回：{净。插座}

通过发送 RST 数据包关闭 TCP 连接并销毁流。
如果此 TCP 套接字处于连接状态，它将发送一个 RST 数据包，并在连接后销毁此 TCP 套接字。
否则，它将调用`socket.destroy`与`ERR_SOCKET_CLOSED`错误。
如果这不是 TCP 套接字（例如，管道），则调用此方法将立即抛出`ERR_INVALID_HANDLE_TYPE`错误。

### `socket.resume()`

*   返回：{净。套接字} 套接字本身。

呼叫后恢复阅读[`socket.pause()`][socket.pause()].

### `socket.setEncoding([encoding])`

<!-- YAML
added: v0.1.90
-->

*   `encoding`{字符串}
*   返回：{净。套接字} 套接字本身。

将套接字的编码设置为[可读流][Readable Stream].看
[`readable.setEncoding()`][readable.setEncoding()]了解更多信息。

### `socket.setKeepAlive([enable][, initialDelay])`

<!-- YAML
added: v0.1.92
changes:
  - version:
    - v13.12.0
    - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32204
    description: New defaults for `TCP_KEEPCNT` and `TCP_KEEPINTVL` socket options were added.
-->

*   `enable`{布尔值}**违约：** `false`
*   `initialDelay`{数字}**违约：** `0`
*   返回：{净。套接字} 套接字本身。

启用/禁用保持活动状态功能，并可选择设置初始
在空闲套接字上发送第一个保持活动探测器之前的延迟。

设置`initialDelay`（以毫秒为单位）以设置最后一个之间的延迟
接收到的数据包和第一个保持活动状态的探测。设置`0`为
`initialDelay`将保持值与默认值不变
（或上一个）设置。

启用保持活动状态功能将设置以下套接字选项：

*   `SO_KEEPALIVE=1`
*   `TCP_KEEPIDLE=initialDelay`
*   `TCP_KEEPCNT=10`
*   `TCP_KEEPINTVL=1`

### `socket.setNoDelay([noDelay])`

<!-- YAML
added: v0.1.90
-->

*   `noDelay`{布尔值}**违约：** `true`
*   返回：{净。套接字} 套接字本身。

启用/禁用 Nagle 算法的使用。

创建TCP连接时，它将启用Nagle的算法。

Nagle的算法在通过网络发送数据之前延迟数据。它尝试
以延迟为代价优化吞吐量。

通过`true`为`noDelay`或者不传递参数将禁用 Nagle 的
套接字的算法。通过`false`为`noDelay`将启用内格尔
算法。

### `socket.setTimeout(timeout[, callback])`

<!-- YAML
added: v0.1.90
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `timeout`{数字}
*   `callback`{函数}
*   返回：{净。套接字} 套接字本身。

将套接字设置为在以下时间后超时`timeout`毫秒级的不活动状态
套接字。默认情况下`net.Socket`没有超时。

当触发空闲超时时，套接字将收到[`'timeout'`]['timeout']
事件，但连接不会被切断。用户必须手动调用
[`socket.end()`][socket.end()]或[`socket.destroy()`][socket.destroy()]以结束连接。

```js
socket.setTimeout(3000);
socket.on('timeout', () => {
  console.log('socket timeout');
  socket.end();
});
```

如果`timeout`为 0，则禁用现有的空闲超时。

可选`callback`参数将添加为
[`'timeout'`]['timeout']事件。

### `socket.timeout`

<!-- YAML
added: v10.7.0
-->

*   {数字|未定义}

套接字超时（以毫秒为单位），由[`socket.setTimeout()`][socket.setTimeout()].
是的`undefined`如果尚未设置超时。

### `socket.unref()`

<!-- YAML
added: v0.9.1
-->

*   返回：{净。套接字} 套接字本身。

叫`unref()`将允许程序退出，如果这是唯一的
事件系统中的活动套接字。如果套接字已经`unref`ed 呼叫
`unref()`再也不会有效果。

### `socket.write(data[, encoding][, callback])`

<!-- YAML
added: v0.1.90
-->

*   `data`{字符串|缓冲区|Uint8Array}
*   `encoding`{字符串}仅当数据`string`.**违约：** `utf8`.
*   `callback`{函数}
*   返回：{布尔值}

在套接字上发送数据。第二个参数指定
字符串的大小写。它默认为 UTF8 编码。

返回`true`如果整个数据已成功刷新到内核
缓冲区。返回`false`如果全部或部分数据在用户内存中排队。
[`'drain'`]['drain']当缓冲区再次释放时，将发出。

可选`callback`参数将在数据最终执行时执行
写出来，可能不会立即写出来。

看`Writable`流[`write()`][stream_writable_write]方法更多
信息。

### `socket.readyState`

<!-- YAML
added: v0.5.0
-->

*   {字符串}

此属性以字符串形式表示连接的状态。

*   如果流正在连接`socket.readyState`是`opening`.
*   如果流是可读写的，则它是`open`.
*   如果流是可读的且不可写，则它是`readOnly`.
*   如果流不可读写，则它是`writeOnly`.

## `net.connect()`

别名
[`net.createConnection()`][`net.createConnection()`].

可能的签名：

*   [`net.connect(options[, connectListener])`][`net.connect(options)`]
*   [`net.connect(path[, connectListener])`][`net.connect(path)`]为[工控机][IPC]
    连接。
*   [`net.connect(port[, host][, connectListener])`][`net.connect(port, host)`]
    用于 TCP 连接。

### `net.connect(options[, connectListener])`

<!-- YAML
added: v0.7.0
-->

*   `options`{对象}
*   `connectListener`{函数}
*   返回：{净。插座}

别名
[`net.createConnection(options[, connectListener])`][`net.createConnection(options)`].

### `net.connect(path[, connectListener])`

<!-- YAML
added: v0.1.90
-->

*   `path`{字符串}
*   `connectListener`{函数}
*   返回：{净。插座}

别名
[`net.createConnection(path[, connectListener])`][`net.createConnection(path)`].

### `net.connect(port[, host][, connectListener])`

<!-- YAML
added: v0.1.90
-->

*   `port`{数字}
*   `host`{字符串}
*   `connectListener`{函数}
*   返回：{净。插座}

别名
[`net.createConnection(port[, host][, connectListener])`][`net.createConnection(port, host)`].

## `net.createConnection()`

工厂函数，用于创建新的[`net.Socket`][net.Socket],
立即启动与[`socket.connect()`][socket.connect()],
然后返回`net.Socket`这将启动连接。

建立连接后，[`'connect'`]['connect']将发出事件
在返回的套接字上。最后一个参数`connectListener`，如果提供，
将被添加为[`'connect'`]['connect']事件**一次**.

可能的签名：

*   [`net.createConnection(options[, connectListener])`][`net.createConnection(options)`]
*   [`net.createConnection(path[, connectListener])`][`net.createConnection(path)`]
    为[工控机][IPC]连接。
*   [`net.createConnection(port[, host][, connectListener])`][`net.createConnection(port, host)`]
    用于 TCP 连接。

这[`net.connect()`][net.connect()]函数是此函数的别名。

### `net.createConnection(options[, connectListener])`

<!-- YAML
added: v0.1.90
-->

*   `options`{对象}必填。将传递给两个
    [`new net.Socket([options])`][`new net.Socket(options)`]调用和
    [`socket.connect(options[, connectListener])`][`socket.connect(options)`]
    方法。
*   `connectListener`{函数}的通用参数
    [`net.createConnection()`][net.createConnection()]功能。如果提供，将添加为
    的监听器[`'connect'`]['connect']事件在返回的套接字上一次。
*   返回：{净。套接字} 用于启动连接的新创建的套接字。

有关可用选项，请参阅
[`new net.Socket([options])`][`new net.Socket(options)`]
和[`socket.connect(options[, connectListener])`][`socket.connect(options)`].

附加选项：

*   `timeout`{数字}如果设置，将用于调用
    [`socket.setTimeout(timeout)`][socket.setTimeout(timeout)]在创建套接字之后，但在之前
    它启动连接。

下面是描述的 echo 服务器的客户端示例
在[`net.createServer()`][net.createServer()]部分：

```js
const net = require('node:net');
const client = net.createConnection({ port: 8124 }, () => {
  // 'connect' listener.
  console.log('connected to server!');
  client.write('world!\r\n');
});
client.on('data', (data) => {
  console.log(data.toString());
  client.end();
});
client.on('end', () => {
  console.log('disconnected from server');
});
```

在套接字上进行连接`/tmp/echo.sock`:

```js
const client = net.createConnection({ path: '/tmp/echo.sock' });
```

### `net.createConnection(path[, connectListener])`

<!-- YAML
added: v0.1.90
-->

*   `path`{字符串}套接字应连接到的路径。将传递到
    [`socket.connect(path[, connectListener])`][`socket.connect(path)`].
    看[标识 IPC 连接的路径][Identifying paths for IPC connections].
*   `connectListener`{函数}的通用参数
    [`net.createConnection()`][net.createConnection()]函数，一个“一次”的侦听器
    `'connect'`事件。将传递到
    [`socket.connect(path[, connectListener])`][`socket.connect(path)`].
*   返回：{净。套接字} 用于启动连接的新创建的套接字。

启动[工控机][IPC]连接。

此函数创建一个新的[`net.Socket`][net.Socket]将所有选项设置为默认值，
立即启动与
[`socket.connect(path[, connectListener])`][`socket.connect(path)`],
然后返回`net.Socket`这将启动连接。

### `net.createConnection(port[, host][, connectListener])`

<!-- YAML
added: v0.1.90
-->

*   `port`{数字}套接字应连接到的端口。将传递到
    [`socket.connect(port[, host][, connectListener])`][`socket.connect(port)`].
*   `host`{字符串}套接字应连接到的主机。将传递到
    [`socket.connect(port[, host][, connectListener])`][`socket.connect(port)`].
    **违约：** `'localhost'`.
*   `connectListener`{函数}的通用参数
    [`net.createConnection()`][net.createConnection()]函数，一个“一次”的侦听器
    `'connect'`事件。将传递到
    [`socket.connect(port[, host][, connectListener])`][`socket.connect(port)`].
*   返回：{净。套接字} 用于启动连接的新创建的套接字。

启动 TCP 连接。

此函数创建一个新的[`net.Socket`][net.Socket]将所有选项设置为默认值，
立即启动与
[`socket.connect(port[, host][, connectListener])`][`socket.connect(port)`],
然后返回`net.Socket`这将启动连接。

## `net.createServer([options][, connectionListener])`

<!-- YAML
added: v0.5.0
-->

*   `options`{对象}
    *   `allowHalfOpen`{布尔值}如果设置为`false`，则套接字将
        当可读侧结束时，自动结束可写侧。
        **违约：** `false`.
    *   `pauseOnConnect`{布尔值}指示套接字是否应
        在传入连接时暂停。**违约：** `false`.
    *   `noDelay`{布尔值}如果设置为`true`，它立即禁用了Nagle算法的使用
        收到新的传入连接后。**违约：** `false`.
    *   `keepAlive`{布尔值}如果设置为`true`，它支持套接字上的保持活动状态功能
        在收到新的传入连接后立即，类似于
        [`socket.setKeepAlive([enable][, initialDelay])`][`socket.setKeepAlive(enable, initialDelay)`].
        **违约：** `false`.
    *   `keepAliveInitialDelay`{数字}如果设置为正数，则将初始延迟设置为
        第一个保持活动探测器在空闲套接字上发送。**违约：** `0`.

*   `connectionListener`{函数}自动设置为
    [`'connection'`]['connection']事件。

*   返回：{净。服务器}

创建新的 TCP 或[工控机][IPC]服务器。

如果`allowHalfOpen`设置为`true`，当插座的另一端
信号端传输，服务器只会发回端端
传输时[`socket.end()`][socket.end()]被显式调用。例如，在
TCP 的上下文，当收到 FIN 打包时，发送 FIN 打包
仅当[`socket.end()`][socket.end()]被显式调用。在此之前
连接是半闭合的（不可读，但仍然可写）。看[`'end'`]['end']
事件和[RFC 1122][half-closed]（第 4.2.2.13 节）了解更多信息。

如果`pauseOnConnect`设置为`true`，则与每个
传入连接将暂停，并且不会从其句柄中读取任何数据。
这允许在进程之间传递连接，而无需任何数据
由原始进程读取。要开始从暂停的套接字读取数据，请调用
[`socket.resume()`][socket.resume()].

服务器可以是 TCP 服务器，也可以是[工控机][IPC]服务器，取决于它是什么
[`listen()`][`server.listen()`]自。

下面是一个 TCP 回显服务器的示例，它侦听连接
在端口 8124 上：

```js
const net = require('node:net');
const server = net.createServer((c) => {
  // 'connection' listener.
  console.log('client connected');
  c.on('end', () => {
    console.log('client disconnected');
  });
  c.write('hello\r\n');
  c.pipe(c);
});
server.on('error', (err) => {
  throw err;
});
server.listen(8124, () => {
  console.log('server bound');
});
```

通过使用测试`telnet`:

```console
$ telnet localhost 8124
```

在套接字上侦听`/tmp/echo.sock`:

```js
server.listen('/tmp/echo.sock', () => {
  console.log('server bound');
});
```

用`nc`连接到 Unix 域套接字服务器：

```console
$ nc -U /tmp/echo.sock
```

## `net.isIP(input)`

<!-- YAML
added: v0.3.0
-->

*   `input`{字符串}
*   返回：{整数}

返回`6`如果`input`是一个 IPv6 地址。返回`4`如果`input`是 IPv4
地址[点十进制表示法][dot-decimal notation]没有前导零。否则，返回
`0`.

```js
net.isIP('::1'); // returns 6
net.isIP('127.0.0.1'); // returns 4
net.isIP('127.000.000.001'); // returns 0
net.isIP('127.0.0.1/24'); // returns 0
net.isIP('fhqwhgads'); // returns 0
```

## `net.isIPv4(input)`

<!-- YAML
added: v0.3.0
-->

*   `input`{字符串}
*   返回：{布尔值}

返回`true`如果`input`是[点十进制表示法][dot-decimal notation]无
前导零。否则，返回`false`.

```js
net.isIPv4('127.0.0.1'); // returns true
net.isIPv4('127.000.000.001'); // returns false
net.isIPv4('127.0.0.1/24'); // returns false
net.isIPv4('fhqwhgads'); // returns false
```

## `net.isIPv6(input)`

<!-- YAML
added: v0.3.0
-->

*   `input`{字符串}
*   返回：{布尔值}

返回`true`如果`input`是一个 IPv6 地址。否则，返回`false`.

```js
net.isIPv6('::1'); // returns true
net.isIPv6('fhqwhgads'); // returns false
```

[IPC]: #ipc-support

[Identifying paths for IPC connections]: #identifying-paths-for-ipc-connections

[Readable Stream]: stream.md#class-streamreadable

[`'close'`]: #event-close

[`'connect'`]: #event-connect

[`'connection'`]: #event-connection

[`'data'`]: #event-data

[`'drain'`]: #event-drain

[`'end'`]: #event-end

[`'error'`]: #event-error_1

[`'listening'`]: #event-listening

[`'timeout'`]: #event-timeout

[`EventEmitter`]: events.md#class-eventemitter

[`child_process.fork()`]: child_process.md#child_processforkmodulepath-args-options

[`dns.lookup()`]: dns.md#dnslookuphostname-options-callback

[`dns.lookup()` hints]: dns.md#supported-getaddrinfo-flags

[`net.Server`]: #class-netserver

[`net.Socket`]: #class-netsocket

[`net.connect()`]: #netconnect

[`net.connect(options)`]: #netconnectoptions-connectlistener

[`net.connect(path)`]: #netconnectpath-connectlistener

[`net.connect(port, host)`]: #netconnectport-host-connectlistener

[`net.createConnection()`]: #netcreateconnection

[`net.createConnection(options)`]: #netcreateconnectionoptions-connectlistener

[`net.createConnection(path)`]: #netcreateconnectionpath-connectlistener

[`net.createConnection(port, host)`]: #netcreateconnectionport-host-connectlistener

[`net.createServer()`]: #netcreateserveroptions-connectionlistener

[`new net.Socket(options)`]: #new-netsocketoptions

[`readable.setEncoding()`]: stream.md#readablesetencodingencoding

[`server.close()`]: #serverclosecallback

[`server.listen()`]: #serverlisten

[`server.listen(handle)`]: #serverlistenhandle-backlog-callback

[`server.listen(options)`]: #serverlistenoptions-callback

[`server.listen(path)`]: #serverlistenpath-backlog-callback

[`server.listen(port)`]: #serverlistenport-host-backlog-callback

[`socket(7)`]: https://man7.org/linux/man-pages/man7/socket.7.html

[`socket.connect()`]: #socketconnect

[`socket.connect(options)`]: #socketconnectoptions-connectlistener

[`socket.connect(path)`]: #socketconnectpath-connectlistener

[`socket.connect(port)`]: #socketconnectport-host-connectlistener

[`socket.connecting`]: #socketconnecting

[`socket.destroy()`]: #socketdestroyerror

[`socket.end()`]: #socketenddata-encoding-callback

[`socket.pause()`]: #socketpause

[`socket.resume()`]: #socketresume

[`socket.setEncoding()`]: #socketsetencodingencoding

[`socket.setKeepAlive(enable, initialDelay)`]: #socketsetkeepaliveenable-initialdelay

[`socket.setTimeout()`]: #socketsettimeouttimeout-callback

[`socket.setTimeout(timeout)`]: #socketsettimeouttimeout-callback

[`writable.destroy()`]: stream.md#writabledestroyerror

[`writable.destroyed`]: stream.md#writabledestroyed

[`writable.end()`]: stream.md#writableendchunk-encoding-callback

[`writable.writableLength`]: stream.md#writablewritablelength

[dot-decimal notation]: https://en.wikipedia.org/wiki/Dot-decimal_notation

[half-closed]: https://tools.ietf.org/html/rfc1122

[stream_writable_write]: stream.md#writablewritechunk-encoding-callback

[unspecified IPv4 address]: https://en.wikipedia.org/wiki/0.0.0.0

[unspecified IPv6 address]: https://en.wikipedia.org/wiki/IPv6_address#Unspecified_address
