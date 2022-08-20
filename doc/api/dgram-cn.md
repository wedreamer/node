# UDP/数据报套接字

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- name=dgram -->

<!-- source_link=lib/dgram.js -->

这`node:dgram`模块提供了 UDP 数据报套接字的实现。

```mjs
import dgram from 'node:dgram';

const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`server error:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`server listening ${address.address}:${address.port}`);
});

server.bind(41234);
// Prints: server listening 0.0.0.0:41234
```

```cjs
const dgram = require('node:dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`server error:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`server listening ${address.address}:${address.port}`);
});

server.bind(41234);
// Prints: server listening 0.0.0.0:41234
```

## 类：`dgram.Socket`

<!-- YAML
added: v0.1.99
-->

*   扩展：{事件发射器}

封装数据报功能。

的新实例`dgram.Socket`创建于[`dgram.createSocket()`][dgram.createSocket()].
这`new`关键字不用于创建`dgram.Socket`实例。

### 事件：`'close'`

<!-- YAML
added: v0.1.99
-->

这`'close'`事件是在套接字关闭后发出的[`close()`][close()].
一旦触发，就没有新的`'message'`事件将在此套接字上发出。

### 事件：`'connect'`

<!-- YAML
added: v12.0.0
-->

这`'connect'`事件在套接字与远程数据库关联后发出
地址作为成功的结果[`connect()`][connect()]叫。

### 事件：`'error'`

<!-- YAML
added: v0.1.99
-->

*   `exception`{错误}

这`'error'`每当发生任何错误时，都会发出事件。事件处理程序
函数传递单个`Error`对象。

### 事件：`'listening'`

<!-- YAML
added: v0.1.99
-->

这`'listening'`事件在`dgram.Socket`是可寻址的，并且
可以接收数据。这种情况可以显式地通过`socket.bind()`或
隐式地首次发送数据时使用`socket.send()`.
直到`dgram.Socket`正在侦听，底层系统资源不
存在和调用，例如`socket.address()`和`socket.setTTL()`将失败。

### 事件：`'message'`

<!-- YAML
added: v0.1.99
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43054
    description: The `family` property now returns a string instead of a number.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41431
    description: The `family` property now returns a number instead of a string.
-->

这`'message'`当套接字上有可用的新数据报时，将发出事件。
事件处理程序函数传递两个参数：`msg`和`rinfo`.

*   `msg`{缓冲区}消息。
*   `rinfo`{对象}远程地址信息。
    *   `address`{字符串}发件人地址。
    *   `family`{字符串}地址系列 （`'IPv4'`或`'IPv6'`).
    *   `port`{数字}发送方端口。
    *   `size`{数字}邮件大小。

如果传入数据包的源地址是 IPv6 链路本地
地址，接口名称将添加到`address`.为
例如，在`en0`接口可能具有
地址字段设置为`'fe80::2618:1234:ab11:3b9c%en0'`哪里`'%en0'`
是作为区域 ID 后缀的接口名称。

### `socket.addMembership(multicastAddress[, multicastInterface])`

<!-- YAML
added: v0.6.9
-->

*   `multicastAddress`{字符串}
*   `multicastInterface`{字符串}

告诉内核在给定位置加入多播组`multicastAddress`和
`multicastInterface`使用`IP_ADD_MEMBERSHIP`套接字选项。如果
`multicastInterface`参数未指定，操作系统将选择
一个接口，并将向其添加成员身份。向“每个”添加成员身份
可用接口，调用`addMembership`多次，每个接口一次。

在未绑定套接字上调用时，此方法将隐式绑定到随机
端口，侦听所有接口。

在多个 UDP 套接字之间共享时`cluster`工人，
`socket.addMembership()`函数必须只调用一次或
`EADDRINUSE`将发生错误：

```mjs
import cluster from 'node:cluster';
import dgram from 'node:dgram';

if (cluster.isPrimary) {
  cluster.fork(); // Works ok.
  cluster.fork(); // Fails with EADDRINUSE.
} else {
  const s = dgram.createSocket('udp4');
  s.bind(1234, () => {
    s.addMembership('224.0.0.114');
  });
}
```

```cjs
const cluster = require('node:cluster');
const dgram = require('node:dgram');

if (cluster.isPrimary) {
  cluster.fork(); // Works ok.
  cluster.fork(); // Fails with EADDRINUSE.
} else {
  const s = dgram.createSocket('udp4');
  s.bind(1234, () => {
    s.addMembership('224.0.0.114');
  });
}
```

### `socket.addSourceSpecificMembership(sourceAddress, groupAddress[, multicastInterface])`

<!-- YAML
added:
 - v13.1.0
 - v12.16.0
-->

*   `sourceAddress`{字符串}
*   `groupAddress`{字符串}
*   `multicastInterface`{字符串}

告诉内核在给定位置加入特定于源的多播通道
`sourceAddress`和`groupAddress`，使用`multicastInterface`与
`IP_ADD_SOURCE_MEMBERSHIP`套接字选项。如果`multicastInterface`论点
未指定，操作系统将选择一个接口并添加
它的成员资格。要将成员资格添加到每个可用接口，请调用
`socket.addSourceSpecificMembership()`多次，每个接口一次。

在未绑定套接字上调用时，此方法将隐式绑定到随机
端口，侦听所有接口。

### `socket.address()`

<!-- YAML
added: v0.1.99
-->

*   返回： {对象}

返回一个对象，其中包含套接字的地址信息。
对于 UDP 套接字，此对象将包含`address`,`family`和`port`
性能。

此方法抛出`EBADF`如果在未绑定套接字上调用。

### `socket.bind([port][, address][, callback])`

<!-- YAML
added: v0.1.99
changes:
  - version: v0.9.1
    commit: 332fea5ac1816e498030109c4211bca24a7fa667
    description: The method was changed to an asynchronous execution model.
                 Legacy code would need to be changed to pass a callback
                 function to the method call.
-->

*   `port`{整数}
*   `address`{字符串}
*   `callback`{函数} 没有参数。绑定完成时调用。

对于 UDP 套接字，导致`dgram.Socket`侦听数据报
命名的邮件`port`和可选`address`.如果`port`莫
指定 或 是`0`，操作系统将尝试绑定到
随机端口。如果`address`未指定，操作系统将
尝试侦听所有地址。绑定完成后，一个
`'listening'`发出事件，并且可选`callback`函数是
叫。

指定`'listening'`事件侦听器和传递
`callback`到`socket.bind()`方法无害，但不是很
有用。

绑定的数据报套接字使 Node.js进程保持运行以接收
数据报消息。

如果绑定失败，则`'error'`将生成事件。在极少数情况下（例如
尝试与封闭的套接字绑定），一个[`Error`][Error]可能会被抛出。

在端口 41234 上侦听的 UDP 服务器的示例：

```mjs
import dgram from 'node:dgram';

const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`server error:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`server listening ${address.address}:${address.port}`);
});

server.bind(41234);
// Prints: server listening 0.0.0.0:41234
```

```cjs
const dgram = require('node:dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`server error:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`server listening ${address.address}:${address.port}`);
});

server.bind(41234);
// Prints: server listening 0.0.0.0:41234
```

### `socket.bind(options[, callback])`

<!-- YAML
added: v0.11.14
-->

*   `options`{对象}必填。支持以下属性：
    *   `port`{整数}
    *   `address`{字符串}
    *   `exclusive`{布尔值}
    *   `fd`{整数}
*   `callback`{函数}

对于 UDP 套接字，导致`dgram.Socket`侦听数据报
命名的邮件`port`和可选`address`作为传递
的属性`options`对象作为第一个参数传递。如果
`port`未指定或`0`，操作系统将尝试
以绑定到随机端口。如果`address`未指定，操作
系统将尝试侦听所有地址。一旦绑定是
完成， 一个`'listening'`发出事件，并且可选`callback`
函数被调用。

这`options`对象可以包含`fd`财产。当`fd`大
比`0`设置，它将用给定的
文件描述符。在这种情况下，属性`port`和`address`
将被忽略。

指定`'listening'`事件侦听器和传递
`callback`到`socket.bind()`方法无害，但不是很
有用。

这`options`对象可以包含附加`exclusive`属性
使用时使用`dgram.Socket`对象[`cluster`][cluster]模块。什么时候
`exclusive`设置为`false`（默认值），群集辅助角色将使用相同的
底层套接字句柄允许共享连接处理职责。
什么时候`exclusive`是`true`，但是，句柄未共享和尝试
端口共享导致错误。

绑定的数据报套接字使 Node.js进程保持运行以接收
数据报消息。

如果绑定失败，则`'error'`将生成事件。在极少数情况下（例如
尝试与封闭的套接字绑定），一个[`Error`][Error]可能会被抛出。

下面显示了侦听独占端口的套接字示例。

```js
socket.bind({
  address: 'localhost',
  port: 8000,
  exclusive: true
});
```

### `socket.close([callback])`

<!-- YAML
added: v0.1.99
-->

*   `callback`{函数}在套接字关闭时调用。

关闭基础套接字并停止侦听其上的数据。如果回调是
提供，它被添加为[`'close'`]['close']事件。

### `socket.connect(port[, address][, callback])`

<!-- YAML
added: v12.0.0
-->

*   `port`{整数}
*   `address`{字符串}
*   `callback`{函数}在连接完成或出错时调用。

关联`dgram.Socket`到远程地址和端口。每
此句柄发送的消息将自动发送到该目标。也
套接字将仅接收来自该远程对等体的消息。
尝试致电`connect()`在已连接的套接字上将产生
在[`ERR_SOCKET_DGRAM_IS_CONNECTED`][ERR_SOCKET_DGRAM_IS_CONNECTED]例外。如果`address`莫
提供`'127.0.0.1'`（对于`udp4`套接字）或`'::1'`（对于`udp6`套接字）
默认情况下将使用。连接完成后，一个`'connect'`事件
发出，并且可选`callback`函数被调用。如果发生故障，
这`callback`被调用，或者，如果调用，则`'error'`发出事件。

### `socket.disconnect()`

<!-- YAML
added: v12.0.0
-->

取消关联连接的同步函数`dgram.Socket`从
其远程地址。尝试致电`disconnect()`未绑定或已绑定
断开连接的套接字将导致[`ERR_SOCKET_DGRAM_NOT_CONNECTED`][ERR_SOCKET_DGRAM_NOT_CONNECTED]
例外。

### `socket.dropMembership(multicastAddress[, multicastInterface])`

<!-- YAML
added: v0.6.9
-->

*   `multicastAddress`{字符串}
*   `multicastInterface`{字符串}

指示内核将多播组保留在`multicastAddress`使用
`IP_DROP_MEMBERSHIP`套接字选项。此方法由
内核，当套接字关闭或进程终止时，大多数应用程序将
永远没有理由这样称呼。

如果`multicastInterface`未指定，操作系统将尝试
删除所有有效接口上的成员身份。

### `socket.dropSourceSpecificMembership(sourceAddress, groupAddress[, multicastInterface])`

<!-- YAML
added:
 - v13.1.0
 - v12.16.0
-->

*   `sourceAddress`{字符串}
*   `groupAddress`{字符串}
*   `multicastInterface`{字符串}

指示内核在给定位置保留特定于源的多播通道
`sourceAddress`和`groupAddress`使用`IP_DROP_SOURCE_MEMBERSHIP`
套接字选项。此方法由内核自动调用，当
socket 已关闭或进程终止，因此大多数应用永远不会有
有理由这样称呼。

如果`multicastInterface`未指定，操作系统将尝试
删除所有有效接口上的成员身份。

### `socket.getRecvBufferSize()`

<!-- YAML
added: v8.7.0
-->

*   返回：{数字}`SO_RCVBUF`套接字接收缓冲区大小（以字节为单位）。

此方法抛出[`ERR_SOCKET_BUFFER_SIZE`][ERR_SOCKET_BUFFER_SIZE]如果在未绑定套接字上调用。

### `socket.getSendBufferSize()`

<!-- YAML
added: v8.7.0
-->

*   返回：{数字}`SO_SNDBUF`套接字发送缓冲区大小（以字节为单位）。

此方法抛出[`ERR_SOCKET_BUFFER_SIZE`][ERR_SOCKET_BUFFER_SIZE]如果在未绑定套接字上调用。

### `socket.ref()`

<!-- YAML
added: v0.9.1
-->

*   返回：{dgram。插座}

默认情况下，绑定套接字将导致它阻止 Node.js 进程
退出，只要套接字处于打开状态。这`socket.unref()`可以使用的方法
从保留节点的引用计数中排除套接字.js
进程处于活动状态。这`socket.ref()`方法将套接字添加回引用
计数并恢复默认行为。

叫`socket.ref()`多次不会产生额外的影响。

这`socket.ref()`方法返回对套接字的引用，以便调用可以
链。

### `socket.remoteAddress()`

<!-- YAML
added: v12.0.0
-->

*   返回： {对象}

返回一个对象，其中包含`address`,`family`和`port`遥控器
端点。此方法抛出一个[`ERR_SOCKET_DGRAM_NOT_CONNECTED`][ERR_SOCKET_DGRAM_NOT_CONNECTED]例外
如果套接字未连接。

### `socket.send(msg[, offset, length][, port][, address][, callback])`

<!-- YAML
added: v0.1.99
changes:
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/39190
    description: The `address` parameter now only accepts a `string`, `null`
                 or `undefined`.
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/22413
    description: The `msg` parameter can now be any `TypedArray` or `DataView`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26871
    description: Added support for sending data on connected sockets.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11985
    description: The `msg` parameter can be an `Uint8Array` now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10473
    description: The `address` parameter is always optional now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5929
    description: On success, `callback` will now be called with an `error`
                 argument of `null` rather than `0`.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4374
    description: The `msg` parameter can be an array now. Also, the `offset`
                 and `length` parameters are optional now.
-->

*   `msg`{缓冲区|TypedArray|数据视图|字符串|数组} 要发送的消息。
*   `offset`{整数}消息开始的缓冲区中的偏移量。
*   `length`{整数}消息中的字节数。
*   `port`{整数}目标端口。
*   `address`{字符串}目标主机名或 IP 地址。
*   `callback`{函数}在发送消息时调用。

在套接字上广播数据报。
对于无连接套接字，目标`port`和`address`必须是
指定。另一方面，连接的套接字将使用其关联的套接字
远程端点，因此`port`和`address`不得设置参数。

这`msg`参数包含要发送的消息。
根据其类型，可以应用不同的行为。如果`msg`是一个`Buffer`,
任何`TypedArray`或`DataView`,
这`offset`和`length`指定`Buffer`其中
消息开始和消息中的字节数分别为。
如果`msg`是一个`String`，则它将自动转换为`Buffer`
跟`'utf8'`编码。包含以下消息
包含多字节字符，`offset`和`length`将计算
尊重[字节长度][byte length]而不是字符位置。
如果`msg`是一个数组，`offset`和`length`不得指定。

这`address`参数是一个字符串。如果`address`是主机名，
DNS 将用于解析主机的地址。如果`address`莫
提供或以其他方式无效，`'127.0.0.1'`（对于`udp4`套接字）或`'::1'`
（对于`udp6`套接字） 将默认使用。

如果套接字以前未通过调用绑定到`bind`、套接字
被分配了一个随机端口号，并绑定到“所有接口”地址
(`'0.0.0.0'`为`udp4`插座`'::0'`为`udp6`套接字。

可选`callback`函数可以指定为报告方式
DNS 错误或用于确定何时可以安全地重用`buf`对象。
DNS 查找会延迟发送至少一个刻度
节点.js事件循环。

确定数据报是否已发送的唯一方法是使用
`callback`.如果发生错误，并且`callback`给出，错误将为
作为第一个参数传递给`callback`.如果`callback`未给出，
错误以`'error'`事件`socket`对象。

偏移和长度是可选的，但两者兼而有之*必须*如果使用其中任何一个，则进行设置。
仅当第一个参数是`Buffer`一个`TypedArray`,
或`DataView`.

此方法抛出[`ERR_SOCKET_BAD_PORT`][ERR_SOCKET_BAD_PORT]如果在未绑定套接字上调用。

将 UDP 数据包发送到 上的端口的示例`localhost`;

```mjs
import dgram from 'node:dgram';
import { Buffer } from 'node:buffer';

const message = Buffer.from('Some bytes');
const client = dgram.createSocket('udp4');
client.send(message, 41234, 'localhost', (err) => {
  client.close();
});
```

```cjs
const dgram = require('node:dgram');
const { Buffer } = require('node:buffer');

const message = Buffer.from('Some bytes');
const client = dgram.createSocket('udp4');
client.send(message, 41234, 'localhost', (err) => {
  client.close();
});
```

将由多个缓冲区组成的 UDP 数据包发送到 上的端口的示例
`127.0.0.1`;

```mjs
import dgram from 'node:dgram';
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from('Some ');
const buf2 = Buffer.from('bytes');
const client = dgram.createSocket('udp4');
client.send([buf1, buf2], 41234, (err) => {
  client.close();
});
```

```cjs
const dgram = require('node:dgram');
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from('Some ');
const buf2 = Buffer.from('bytes');
const client = dgram.createSocket('udp4');
client.send([buf1, buf2], 41234, (err) => {
  client.close();
});
```

发送多个缓冲区可能会更快或更慢，具体取决于
应用程序和操作系统。将基准测试运行到
根据具体情况确定最佳策略。一般而言
但是，发送多个缓冲区更快。

使用连接到 端口的套接字发送 UDP 数据包的示例
`localhost`:

```mjs
import dgram from 'node:dgram';
import { Buffer } from 'node:buffer';

const message = Buffer.from('Some bytes');
const client = dgram.createSocket('udp4');
client.connect(41234, 'localhost', (err) => {
  client.send(message, (err) => {
    client.close();
  });
});
```

```cjs
const dgram = require('node:dgram');
const { Buffer } = require('node:buffer');

const message = Buffer.from('Some bytes');
const client = dgram.createSocket('udp4');
client.connect(41234, 'localhost', (err) => {
  client.send(message, (err) => {
    client.close();
  });
});
```

#### 有关 UDP 数据报大小的注意事项

IPv4/v6 数据报的最大大小取决于`MTU`
（最大传输单位）和`Payload Length`字段大小。

*   这`Payload Length`字段为 16 位宽，这意味着正常
    有效负载不能超过 64K 八位字节，包括互联网标头和数据
    （65，507 字节 = 65，535 − 8 字节 UDP 标头 − 20 字节 IP 标头）;
    这通常适用于环回接口，但如此长的数据报
    消息对于大多数主机和网络是不切实际的。

*   这`MTU`是给定链路层技术可以支持的最大尺寸
    数据报消息。对于任何链路，IPv4 要求最低要求`MTU`的 68
    八位字节，而推荐`MTU`对于 IPv4 为 576（通常建议使用）
    作为`MTU`对于拨号式应用），无论它们到达的是整个还是进入
    碎片。

    对于 IPv6，最低`MTU`是 1280 个八位字节。但是，强制性最低限度
    片段重组缓冲区大小为 1500 个八位字节。68 个八位字节的值为
    非常小，因为大多数当前的链路层技术，如以太网，都有一个
    最低`MTU`的 1500。

不可能事先知道每个链路的MTU通过
数据包可能会传输。发送大于接收方的数据报`MTU`将
不起作用，因为数据包将在不通知
数据未到达其预期接收者的来源。

### `socket.setBroadcast(flag)`

<!-- YAML
added: v0.6.9
-->

*   `flag`{布尔值}

设置或清除`SO_BROADCAST`套接字选项。设置为`true`、UDP
数据包可以发送到本地接口的广播地址。

此方法抛出`EBADF`如果在未绑定套接字上调用。

### `socket.setMulticastInterface(multicastInterface)`

<!-- YAML
added: v8.6.0
-->

*   `multicastInterface`{字符串}

*本节中对范围的所有引用都引用
[IPv6 区域指数][IPv6 Zone Indices]，其定义如下：[RFC 4007][].字符串形式，IP
具有作用域索引的索引写为`'IP%scope'`其中 scope 是接口名称
或接口编号。*

将套接字的默认传出多播接口设置为选定的
接口或返回系统接口选择。这`multicastInterface`必须
是套接字系列中 IP 的有效字符串表示形式。

对于 IPv4 套接字，这应该是为所需物理配置的 IP
接口。发送到套接字上的多播的所有数据包都将在
接口由最近成功使用此调用确定。

对于 IPv6 套接字，`multicastInterface`应包括一个范围来指示
接口，如以下示例所示。在 IPv6 中，个人`send`呼叫可以
在地址中也使用显式作用域，因此仅将数据包发送到多播
未指定显式作用域的地址受最新范围的影响
成功使用此调用。

此方法抛出`EBADF`如果在未绑定套接字上调用。

#### 示例：IPv6 传出多播接口

在大多数系统上，作用域格式使用接口名称：

```js
const socket = dgram.createSocket('udp6');

socket.bind(1234, () => {
  socket.setMulticastInterface('::%eth1');
});
```

在 Windows 上，其中作用域格式使用接口号：

```js
const socket = dgram.createSocket('udp6');

socket.bind(1234, () => {
  socket.setMulticastInterface('::%2');
});
```

#### 示例：IPv4 传出多播接口

所有系统都在所需的物理接口上使用主机的 IP：

```js
const socket = dgram.createSocket('udp4');

socket.bind(1234, () => {
  socket.setMulticastInterface('10.0.0.2');
});
```

#### 通话结果

对尚未准备好发送或不再打开的套接字上的调用可能会引发*不
运行* [`Error`][Error].

如果`multicastInterface`不能解析为IP，然后*埃因瓦尔*
[`System Error`][System Error]被抛出。

在 IPv4 上，如果`multicastInterface`是有效地址，但与任何地址都不匹配
接口，或者如果地址与系列不匹配，则
一个[`System Error`][System Error]如`EADDRNOTAVAIL`或`EPROTONOSUP`被抛出。

在 IPv6 上，大多数指定或省略作用域的错误都会导致套接字
继续使用（或返回到）系统的默认接口选择。

套接字的地址系列的任意地址 （IPv4`'0.0.0.0'`或 IPv6`'::'`） 可以是
用于将套接字默认传出接口的控制权返回给系统
用于将来的多播数据包。

### `socket.setMulticastLoopback(flag)`

<!-- YAML
added: v0.3.8
-->

*   `flag`{布尔值}

设置或清除`IP_MULTICAST_LOOP`套接字选项。设置为`true`,
多播数据包也将在本地接口上接收。

此方法抛出`EBADF`如果在未绑定套接字上调用。

### `socket.setMulticastTTL(ttl)`

<!-- YAML
added: v0.3.8
-->

*   `ttl`{整数}

设置`IP_MULTICAST_TTL`套接字选项。而TTL通常代表
“生存时间”，在此上下文中，它指定 IP 跃点数
允许数据包通过，特别是对于多播流量。每
转发数据包的路由器或网关会递减 TTL。如果 TTL 是
如果路由器递减到 0，则不会转发它。

这`ttl`参数可能介于 0 和 255 之间。大多数系统上的默认值为`1`.

此方法抛出`EBADF`如果在未绑定套接字上调用。

### `socket.setRecvBufferSize(size)`

<!-- YAML
added: v8.7.0
-->

*   `size`{整数}

设置`SO_RCVBUF`套接字选项。设置最大套接字接收缓冲区
以字节为单位。

此方法抛出[`ERR_SOCKET_BUFFER_SIZE`][ERR_SOCKET_BUFFER_SIZE]如果在未绑定套接字上调用。

### `socket.setSendBufferSize(size)`

<!-- YAML
added: v8.7.0
-->

*   `size`{整数}

设置`SO_SNDBUF`套接字选项。设置最大套接字发送缓冲区
以字节为单位。

此方法抛出[`ERR_SOCKET_BUFFER_SIZE`][ERR_SOCKET_BUFFER_SIZE]如果在未绑定套接字上调用。

### `socket.setTTL(ttl)`

<!-- YAML
added: v0.1.101
-->

*   `ttl`{整数}

设置`IP_TTL`套接字选项。虽然TTL通常代表“生存时间”，
在此上下文中，它指定允许数据包的 IP 跃点数
旅行通过。转发数据包的每个路由器或网关都会递减
腾讯网.如果路由器将 TTL 递减为 0，则不会转发 TTL。
更改 TTL 值通常用于网络探测或多播时。

这`ttl`参数可能介于 1 和 255 之间。大多数系统上的默认值
为 64。

此方法抛出`EBADF`如果在未绑定套接字上调用。

### `socket.unref()`

<!-- YAML
added: v0.9.1
-->

*   返回：{dgram。插座}

默认情况下，绑定套接字将导致它阻止 Node.js 进程
退出，只要套接字处于打开状态。这`socket.unref()`可以使用的方法
从保留节点的引用计数中排除套接字.js
进程处于活动状态，即使套接字仍为
注意的。

叫`socket.unref()`多次不会产生加法效应。

这`socket.unref()`方法返回对套接字的引用，以便调用可以
链。

## `node:dgram`模块函数

### `dgram.createSocket(options[, callback])`

<!-- YAML
added: v0.11.13
changes:
  - version: v15.8.0
    pr-url: https://github.com/nodejs/node/pull/37026
    description: AbortSignal support was added.
  - version: v11.4.0
    pr-url: https://github.com/nodejs/node/pull/23798
    description: The `ipv6Only` option is supported.
  - version: v8.7.0
    pr-url: https://github.com/nodejs/node/pull/13623
    description: The `recvBufferSize` and `sendBufferSize` options are
                 supported now.
  - version: v8.6.0
    pr-url: https://github.com/nodejs/node/pull/14560
    description: The `lookup` option is supported.
-->

*   `options`{对象}可用选项包括：
    *   `type`{字符串}插座家族。必须是`'udp4'`或`'udp6'`.
        必填。
    *   `reuseAddr`{布尔值}什么时候`true` [`socket.bind()`][socket.bind()]将重用
        address，即使另一个进程已经绑定了套接字。
        **违约：** `false`.
    *   `ipv6Only`{布尔值}设置`ipv6Only`自`true`将
        禁用双栈支持，即绑定到地址`::`不会使
        `0.0.0.0`被束缚。**违约：** `false`.
    *   `recvBufferSize`{数字}设置`SO_RCVBUF`套接字值。
    *   `sendBufferSize`{数字}设置`SO_SNDBUF`套接字值。
    *   `lookup`{函数}自定义查找功能。**违约：** [`dns.lookup()`][dns.lookup()].
    *   `signal`{中止信号}可用于关闭套接字的中止信号。
*   `callback`{函数}附加为`'message'`事件。自选。
*   返回：{dgram。插座}

创建`dgram.Socket`对象。创建套接字后，调用
[`socket.bind()`][socket.bind()]将指示套接字开始侦听数据报
消息。什么时候`address`和`port`未传递到[`socket.bind()`][socket.bind()]这
方法将套接字绑定到随机端口上的“所有接口”地址
（它为两者都做了正确的事情`udp4`和`udp6`套接字）。绑定地址
和端口可以使用[`socket.address().address`][socket.address().address]和
[`socket.address().port`][socket.address().port].

如果`signal`选项已启用，正在调用`.abort()`在相应的
`AbortController`类似于呼叫`.close()`在套接字上：

```js
const controller = new AbortController();
const { signal } = controller;
const server = dgram.createSocket({ type: 'udp4', signal });
server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});
// Later, when you want to close the server.
controller.abort();
```

### `dgram.createSocket(type[, callback])`

<!-- YAML
added: v0.1.99
-->

*   `type`{字符串}也`'udp4'`或`'udp6'`.
*   `callback`{函数}作为侦听器附加到`'message'`事件。
*   返回：{dgram。插座}

创建`dgram.Socket`指定对象`type`.

创建套接字后，调用[`socket.bind()`][socket.bind()]将指示
套接字以开始侦听数据报消息。什么时候`address`和`port`是
未传递到[`socket.bind()`][socket.bind()]该方法将套接字绑定到“all”
随机端口上的接口“地址（它对两者都做了正确的事情`udp4`
和`udp6`套接字）。可以使用以下命令检索绑定地址和端口
[`socket.address().address`][socket.address().address]和[`socket.address().port`][socket.address().port].

[IPv6 Zone Indices]: https://en.wikipedia.org/wiki/IPv6_address#Scoped_literal_IPv6_addresses

[RFC 4007]: https://tools.ietf.org/html/rfc4007

[`'close'`]: #event-close

[`ERR_SOCKET_BAD_PORT`]: errors.md#err_socket_bad_port

[`ERR_SOCKET_BUFFER_SIZE`]: errors.md#err_socket_buffer_size

[`ERR_SOCKET_DGRAM_IS_CONNECTED`]: errors.md#err_socket_dgram_is_connected

[`ERR_SOCKET_DGRAM_NOT_CONNECTED`]: errors.md#err_socket_dgram_not_connected

[`Error`]: errors.md#class-error

[`System Error`]: errors.md#class-systemerror

[`close()`]: #socketclosecallback

[`cluster`]: cluster.md

[`connect()`]: #socketconnectport-address-callback

[`dgram.createSocket()`]: #dgramcreatesocketoptions-callback

[`dns.lookup()`]: dns.md#dnslookuphostname-options-callback

[`socket.address().address`]: #socketaddress

[`socket.address().port`]: #socketaddress

[`socket.bind()`]: #socketbindport-address-callback

[byte length]: buffer.md#static-method-bufferbytelengthstring-encoding
