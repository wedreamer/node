# 断续器

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/http.js -->

要使用HTTP服务器和客户端，必须`require('node:http')`.

Node.js中的HTTP接口旨在支持许多功能
传统上难以使用的协议。
特别是大型（可能是块编码）消息。接口是
注意从不缓冲整个请求或响应，因此
用户能够流式传输数据。

HTTP 消息头由如下对象表示：

<!-- eslint-skip -->

```js
{ 'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'example.com',
  'accept': '*/*' }
```

键是小写的。不修改值。

为了支持所有可能的HTTP应用程序，Node.js
HTTP API是非常低级的。它处理流处理和消息
仅解析。它将消息解析为标头和正文，但不
解析实际的标头或正文。

看[`message.headers`][message.headers]，详细了解如何处理重复的标头。

收到的原始标头将保留在`rawHeaders`
属性，它是`[key, value, key2, value2, ...]`.为
例如，上一个邮件头对象可能具有`rawHeaders`
列表如下所示：

<!-- eslint-disable semi -->

```js
[ 'ConTent-Length', '123456',
  'content-LENGTH', '123',
  'content-type', 'text/plain',
  'CONNECTION', 'keep-alive',
  'Host', 'example.com',
  'accepT', '*/*' ]
```

## 类：`http.Agent`

<!-- YAML
added: v0.3.4
-->

一`Agent`负责管理连接持久性
并重用 HTTP 客户端。它维护挂起请求的队列
对于给定的主机和端口，为每个主机和端口重用单个套接字连接
直到队列为空，此时套接字被销毁
或放入池中，在那里保留它以再次用于对
相同的主机和端口。它是被销毁还是被池化取决于
`keepAlive` [选择](#new-agentoptions).

池连接为其启用了 TCP 保持活动状态，但服务器可能会
仍然关闭空闲连接，在这种情况下，它们将从
池，当对
该主机和端口。服务器也可能拒绝允许多个请求
在同一连接上，在这种情况下，连接必须是
为每个请求重新制作，并且无法池化。这`Agent`仍然会制作
对该服务器的请求，但每个请求都将通过新连接发生。

当客户端或服务器关闭连接时，将删除该连接
从游泳池。池中任何未使用的套接字都将被取消引用，因此不会
以在没有未完成请求时保持节点.js进程运行。
（请参见[`socket.unref()`][socket.unref()]).

这是很好的做法，[`destroy()`][destroy()]一`Agent`实例为“否”
使用时间更长，因为未使用的套接字会消耗操作系统资源。

当套接字发出任一项时，将从代理中删除套接字
一个`'close'`事件或`'agentRemove'`事件。当打算保留一个
HTTP请求打开很长时间而没有将其保留在代理中，什么
可以像下面这样完成：

```js
http.get(options, (res) => {
  // Do stuff
}).on('socket', (socket) => {
  socket.emit('agentRemove');
});
```

代理也可以用于单个请求。通过提供
`{agent: false}`作为`http.get()`或`http.request()`
函数，一次性使用`Agent`将使用默认选项
用于客户端连接。

`agent:false`:

```js
http.get({
  hostname: 'localhost',
  port: 80,
  path: '/',
  agent: false  // Create a new agent just for this one request
}, (res) => {
  // Do stuff with response
});
```

### `new Agent([options])`

<!-- YAML
added: v0.3.4
changes:
  - version:
      - v15.6.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/36685
    description: Change the default scheduling from 'fifo' to 'lifo'.
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33617
    description: Add `maxTotalSockets` option to agent constructor.
  - version:
      - v14.5.0
      - v12.20.0
    pr-url: https://github.com/nodejs/node/pull/33278
    description: Add `scheduling` option to specify the free socket
                 scheduling strategy.
-->

*   `options`{对象}要在代理上设置的可配置选项集。
    可以具有以下字段：
    *   `keepAlive`{布尔值}即使没有插座，也要随身携带插座
        未完成的请求，因此它们可以用于将来的请求，而无需
        必须重新建立 TCP 连接。不要与
        `keep-alive`的值`Connection`页眉。这`Connection: keep-alive`
        标头在使用代理时始终发送，除非`Connection`
        标头被显式指定，或者当`keepAlive`和`maxSockets`
        选项分别设置为`false`和`Infinity`，在这种情况下
        `Connection: close`将使用。**违约：** `false`.
    *   `keepAliveMsecs`{数字}使用`keepAlive`选项，指定
        这[初始延迟][initial delay]
        用于 TCP 保持活动状态数据包。忽略时
        `keepAlive`选项是`false`或`undefined`.**违约：** `1000`.
    *   `maxSockets`{数字}每个主机允许的最大套接字数。
        如果同一主机打开多个并发连接，则每个请求
        将使用新的套接字，直到`maxSockets`已达到值。
        如果主机尝试打开的连接数超过`maxSockets`,
        其他请求将进入挂起的请求队列，并且
        当现有连接终止时，将进入活动连接状态。
        这确保了最多有`maxSockets`活动连接位于
        从给定主机的任何时间点。
        **违约：** `Infinity`.
    *   `maxTotalSockets`{数字}允许的最大套接字数
        所有主机总计。每个请求将使用一个新的套接字
        直到达到最大值。
        **违约：** `Infinity`.
    *   `maxFreeSockets`{数字}每个主机保持打开状态的最大套接字数
        在自由状态下。仅在以下情况下相关`keepAlive`设置为`true`.
        **违约：** `256`.
    *   `scheduling`{字符串}拣选时要应用的调度策略
        下一个要使用的自由插槽。它可以是`'fifo'`或`'lifo'`.
        这两种调度策略之间的主要区别在于`'lifo'`
        选择最近使用的套接字，而`'fifo'`选择
        最近最少使用的套接字。
        如果每秒请求速率较低，则`'lifo'`调度
        将降低选择可能已关闭的插座的风险
        由服务器由于不活动。
        如果每秒请求速率较高，
        这`'fifo'`调度将最大化打开套接字的数量，
        而`'lifo'`调度将使其尽可能低。
        **违约：** `'lifo'`.
    *   `timeout`{数字}套接字超时（以毫秒为单位）。
        这将在创建套接字时设置超时。

`options`在[`socket.connect()`][socket.connect()]也受支持。

默认值[`http.globalAgent`][http.globalAgent]由[`http.request()`][http.request()]具有全部
这些值设置为其各自的默认值。

要配置其中任何一个，自定义[`http.Agent`][http.Agent]必须创建实例。

```js
const http = require('node:http');
const keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
```

### `agent.createConnection(options[, callback])`

<!-- YAML
added: v0.11.4
-->

*   `options`{对象}包含连接详细信息的选项。检查
    [`net.createConnection()`][net.createConnection()]对于选项的格式
*   `callback`{函数}接收所创建套接字的回调函数
*   返回：{流。双工}

生成用于 HTTP 请求的套接字/流。

默认情况下，此函数与[`net.createConnection()`][net.createConnection()].然而
如果需要更大的灵活性，自定义代理可能会覆盖此方法。

可以通过以下两种方式之一提供套接字/流：通过返回
套接字/流从此函数，或通过将套接字/流传递给`callback`.

此方法保证返回 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

`callback`具有`(err, stream)`.

### `agent.keepSocketAlive(socket)`

<!-- YAML
added: v8.1.0
-->

*   `socket`{流。双工}

调用时间`socket`与请求分离，并且可以由
`Agent`.默认行为是：

```js
socket.setKeepAlive(true, this.keepAliveMsecs);
socket.unref();
return true;
```

此方法可以被特定方法覆盖`Agent`亚纲。如果
方法返回一个 falsy 值，套接字将被销毁而不是持久化
它用于下一个请求。

这`socket`参数可以是 {net 的一个实例。套接字}，的子类
{流。双工}。

### `agent.reuseSocket(socket, request)`

<!-- YAML
added: v8.1.0
-->

*   `socket`{流。双工}
*   `request`{http.客户端请求}

调用时间`socket`附加到`request`由于以下原因而坚持后
保持活动状态选项。默认行为是：

```js
socket.ref();
```

此方法可以被特定方法覆盖`Agent`亚纲。

这`socket`参数可以是 {net 的一个实例。套接字}，的子类
{流。双工}。

### `agent.destroy()`

<!-- YAML
added: v0.11.4
-->

销毁代理当前正在使用的任何套接字。

通常没有必要这样做。但是，如果使用
代理与`keepAlive`启用，则最好显式关闭
不再需要代理时。否则
套接字可能会在服务器出现之前保持打开状态相当长的时间
终止它们。

### `agent.freeSockets`

<!-- YAML
added: v0.11.4
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36409
    description: The property now has a `null` prototype.
-->

*   {对象}

一个对象，其中包含当前等待使用的套接字数组
代理在以下情况下`keepAlive`已启用。不要修改。

套接字`freeSockets`列表将自动销毁，并且
已从 阵列中移除`'timeout'`.

### `agent.getName([options])`

<!-- YAML
added: v0.11.4
changes:
  - version:
    - v17.7.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41906
    description: The `options` parameter is now optional.
-->

*   `options`{对象}一组选项，为名称生成提供信息
    *   `host`{字符串}要颁发的服务器的域名或 IP 地址
        请求
    *   `port`{数字}远程服务器的端口
    *   `localAddress`{字符串}用于绑定网络连接的本地接口
        发出请求时
    *   `family`{整数}如果不等于 4 或 6，则必须为 4 或 6`undefined`.
*   返回：{字符串}

获取一组请求选项的唯一名称，以确定
连接可以重复使用。对于 HTTP 代理，这将返回
`host:port:localAddress`或`host:port:localAddress:family`.对于 HTTPS 代理，
该名称包括 CA、证书、密码和其他特定于 HTTPS/TLS 的选项
它们决定了套接字的可重用性。

### `agent.maxFreeSockets`

<!-- YAML
added: v0.11.7
-->

*   {数字}

默认设置为 256。对于具有以下特点的代理`keepAlive`已启用，此
设置将在空闲置空间中保持打开状态的最大套接字数
州。

### `agent.maxSockets`

<!-- YAML
added: v0.3.6
-->

*   {数字}

默认情况下设置为`Infinity`.确定代理的并发套接字数
可以按源打开。原点是[`agent.getName()`][agent.getName()].

### `agent.maxTotalSockets`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

*   {数字}

默认情况下设置为`Infinity`.确定代理的并发套接字数
可以有打开。与`maxSockets`，则此参数适用于所有源。

### `agent.requests`

<!-- YAML
added: v0.5.9
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36409
    description: The property now has a `null` prototype.
-->

*   {对象}

一个对象，其中包含尚未分配给 的请求队列
插座。不要修改。

### `agent.sockets`

<!-- YAML
added: v0.3.6
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/36409
    description: The property now has a `null` prototype.
-->

*   {对象}

一个对象，其中包含
代理。不要修改。

## 类：`http.ClientRequest`

<!-- YAML
added: v0.1.17
-->

*   扩展：{http.传出消息}

此对象在内部创建，并从[`http.request()`][http.request()].它
表示*进行中*其标头已排队的请求。这
标头仍可使用[`setHeader(name, value)`][setHeader(name, value)],
[`getHeader(name)`][getHeader(name)],[`removeHeader(name)`][removeHeader(name)]应用程序接口。实际标头将
与第一个数据块一起发送或在调用时发送[`request.end()`][request.end()].

若要获取响应，请为 添加侦听器[`'response'`]['response']添加到请求对象。
[`'response'`]['response']将在响应时从请求对象发出
已收到标头。这[`'response'`]['response']事件使用一个执行
参数，它是 的实例[`http.IncomingMessage`][http.IncomingMessage].

在[`'response'`]['response']事件，可以将侦听器添加到
响应对象;特别要听`'data'`事件。

如果不是[`'response'`]['response']添加处理程序，则响应将为
完全丢弃。但是，如果[`'response'`]['response']添加了事件处理程序，
，然后来自响应对象的数据**必须**被消费，要么被
叫`response.read()`每当有`'readable'`事件，或
通过添加`'data'`处理程序，或通过调用`.resume()`方法。
在数据被消耗之前，`'end'`事件不会触发。此外，直到
数据被读取它将消耗内存，最终可能导致
“进程内存不足”错误。

为了向后兼容，`res`只会发出`'error'`如果存在
`'error'`侦听器已注册。

节点.js不检查内容长度和长度
已传输的主体是否相等。

### 事件：`'abort'`

<!-- YAML
added: v1.4.1
deprecated:
  - v17.0.0
  - v16.12.0
-->

> 稳定性：0 - 已弃用。收听`'close'`事件。

在客户端中止请求时发出。此事件仅
在第一次调用时发出`abort()`.

### 事件：`'close'`

<!-- YAML
added: v0.5.4
-->

指示请求已完成，或者其基础连接为
提前终止（在响应完成之前）。

### 事件：`'connect'`

<!-- YAML
added: v0.7.0
-->

*   `response`{http.传入消息}
*   `socket`{流。双工}
*   `head`{缓冲区}

每次服务器使用`CONNECT`方法。如果
此事件未被侦听，客户端收到`CONNECT`方法将
关闭其连接。

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

演示如何侦听`'connect'`事件：

```js
const http = require('node:http');
const net = require('node:net');
const { URL } = require('node:url');

// Create an HTTP tunneling proxy
const proxy = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('okay');
});
proxy.on('connect', (req, clientSocket, head) => {
  // Connect to an origin server
  const { port, hostname } = new URL(`http://${req.url}`);
  const serverSocket = net.connect(port || 80, hostname, () => {
    clientSocket.write('HTTP/1.1 200 Connection Established\r\n' +
                    'Proxy-agent: Node.js-Proxy\r\n' +
                    '\r\n');
    serverSocket.write(head);
    serverSocket.pipe(clientSocket);
    clientSocket.pipe(serverSocket);
  });
});

// Now that proxy is running
proxy.listen(1337, '127.0.0.1', () => {

  // Make a request to a tunneling proxy
  const options = {
    port: 1337,
    host: '127.0.0.1',
    method: 'CONNECT',
    path: 'www.google.com:80'
  };

  const req = http.request(options);
  req.end();

  req.on('connect', (res, socket, head) => {
    console.log('got connected!');

    // Make a request over an HTTP tunnel
    socket.write('GET / HTTP/1.1\r\n' +
                 'Host: www.google.com:80\r\n' +
                 'Connection: close\r\n' +
                 '\r\n');
    socket.on('data', (chunk) => {
      console.log(chunk.toString());
    });
    socket.on('end', () => {
      proxy.close();
    });
  });
});
```

### 事件：`'continue'`

<!-- YAML
added: v0.3.2
-->

当服务器发送“100 Continue”HTTP 响应时发出，通常是因为
请求包含“预期：100-继续”。这是一个指令
客户端应发送请求正文。

### 事件：`'finish'`

<!-- YAML
added: v0.3.6
-->

发送请求时发出。更具体地说，此事件是发出的
当响应标头和正文的最后一段已传递给
用于通过网络传输的操作系统。这并不意味着
服务器已收到任何内容。

### 事件：`'information'`

<!-- YAML
added: v10.0.0
-->

*   `info`{对象}
    *   `httpVersion`{字符串}
    *   `httpVersionMajor`{整数}
    *   `httpVersionMinor`{整数}
    *   `statusCode`{整数}
    *   `statusMessage`{字符串}
    *   `headers`{对象}
    *   `rawHeaders`{字符串\[]}

当服务器发送 1xx 中间响应（不包括 101）时发出
升级）。此事件的侦听器将收到一个对象，其中包含
HTTP 版本、状态代码、状态消息、键值标头对象、
和数组，其中包含原始标头名称，后跟其各自的值。

```js
const http = require('node:http');

const options = {
  host: '127.0.0.1',
  port: 8080,
  path: '/length_request'
};

// Make a request
const req = http.request(options);
req.end();

req.on('information', (info) => {
  console.log(`Got information prior to main response: ${info.statusCode}`);
});
```

101 升级状态不会触发此事件，因为它们从
传统的 HTTP 请求/响应链，例如 Web 套接字、就地 TLS
升级，或 HTTP 2.0。要收到 101 升级通知的通知，请收听
[`'upgrade'`]['upgrade']事件。

### 事件：`'response'`

<!-- YAML
added: v0.1.0
-->

*   `response`{http.传入消息}

在收到对此请求的响应时发出。仅发出此事件
一次。

### 事件：`'socket'`

<!-- YAML
added: v0.5.3
-->

*   `socket`{流。双工}

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

### 事件：`'timeout'`

<!-- YAML
added: v0.7.8
-->

当基础套接字因不活动超时时发出。这只会通知
套接字已空闲。必须手动销毁请求。

另请参阅：[`request.setTimeout()`][request.setTimeout()].

### 事件：`'upgrade'`

<!-- YAML
added: v0.1.94
-->

*   `response`{http.传入消息}
*   `socket`{流。双工}
*   `head`{缓冲区}

每次服务器响应升级请求时发出。如果
事件未被侦听，响应状态代码为 101 切换
协议，接收升级标头的客户端将具有其连接
闭。

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

演示如何侦听`'upgrade'`事件。

```js
const http = require('node:http');

// Create an HTTP server
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('okay');
});
server.on('upgrade', (req, socket, head) => {
  socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
               'Upgrade: WebSocket\r\n' +
               'Connection: Upgrade\r\n' +
               '\r\n');

  socket.pipe(socket); // echo back
});

// Now that server is running
server.listen(1337, '127.0.0.1', () => {

  // make a request
  const options = {
    port: 1337,
    host: '127.0.0.1',
    headers: {
      'Connection': 'Upgrade',
      'Upgrade': 'websocket'
    }
  };

  const req = http.request(options);
  req.end();

  req.on('upgrade', (res, socket, upgradeHead) => {
    console.log('got upgraded!');
    socket.end();
    process.exit(0);
  });
});
```

### `request.abort()`

<!-- YAML
added: v0.3.8
deprecated:
  - v14.1.0
  - v13.14.0
-->

> 稳定性：0 - 已弃用：使用[`request.destroy()`][request.destroy()]相反。

将请求标记为中止。调用此将导致剩余数据
在要丢弃的响应和要销毁的套接字中。

### `request.aborted`

<!-- YAML
added: v0.11.14
deprecated:
  - v17.0.0
  - v16.12.0
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/20230
    description: The `aborted` property is no longer a timestamp number.
-->

> 稳定性：0 - 已弃用。检查[`request.destroyed`][request.destroyed]相反。

*   {布尔值}

这`request.aborted`属性将是`true`如果请求有
已中止。

### `request.connection`

<!-- YAML
added: v0.3.0
deprecated: v13.0.0
-->

> 稳定性：0 - 已弃用。用[`request.socket`][request.socket].

*   {流。双工}

看[`request.socket`][request.socket].

### `request.cork()`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

看[`writable.cork()`][writable.cork()].

### `request.end([data[, encoding]][, callback])`

<!-- YAML
added: v0.1.90
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18780
    description: This method now returns a reference to `ClientRequest`.
-->

*   `data`{字符串|缓冲区}
*   `encoding`{字符串}
*   `callback`{函数}
*   返回值：{this}

完成发送请求。如果身体的任何部位是
取消发送，它会将它们刷新到流中。如果请求是
分块，这将发送终止`'0\r\n\r\n'`.

如果`data`已指定，它等效于调用
[`request.write(data, encoding)`][request.write(data, encoding)]其次`request.end(callback)`.

如果`callback`指定，它将在请求流时调用
已完成。

### `request.destroy([error])`

<!-- YAML
added: v0.3.0
changes:
  - version: v14.5.0
    pr-url: https://github.com/nodejs/node/pull/32789
    description: The function returns `this` for consistency with other Readable
                 streams.
-->

*   `error`{错误}可选，要发出的错误`'error'`事件。
*   返回值：{this}

销毁请求。（可选）发出`'error'`事件
并发出`'close'`事件。调用此将导致剩余数据
在要丢弃的响应和要销毁的套接字中。

看[`writable.destroy()`][writable.destroy()]了解更多详情。

#### `request.destroyed`

<!-- YAML
added:
  - v14.1.0
  - v13.14.0
-->

*   {布尔值}

是`true`后[`request.destroy()`][request.destroy()]已被调用。

看[`writable.destroyed`][writable.destroyed]了解更多详情。

### `request.finished`

<!-- YAML
added: v0.0.1
deprecated:
 - v13.4.0
 - v12.16.0
-->

> 稳定性：0 - 已弃用。用[`request.writableEnded`][request.writableEnded].

*   {布尔值}

这`request.finished`属性将是`true`如果[`request.end()`][request.end()]
已被调用。`request.end()`如果
请求通过以下方式启动[`http.get()`][http.get()].

### `request.flushHeaders()`

<!-- YAML
added: v1.6.0
-->

刷新请求标头。

出于效率原因，Node.js 通常会缓冲请求标头，直到
`request.end()`或写入第一个请求数据块。它
然后尝试将请求标头和数据打包到单个 TCP 数据包中。

这通常是需要的（它保存TCP往返），但不是当第一个
数据直到很久以后才会发送。`request.flushHeaders()`绕过
优化并启动请求。

### `request.getHeader(name)`

<!-- YAML
added: v1.6.0
-->

*   `name`{字符串}
*   返回值：{任意}

读出请求上的标头。该名称不区分大小写。
返回值的类型取决于提供给
[`request.setHeader()`][request.setHeader()].

```js
request.setHeader('content-type', 'text/html');
request.setHeader('Content-Length', Buffer.byteLength(body));
request.setHeader('Cookie', ['type=ninja', 'language=javascript']);
const contentType = request.getHeader('Content-Type');
// 'contentType' is 'text/html'
const contentLength = request.getHeader('Content-Length');
// 'contentLength' is of type number
const cookie = request.getHeader('Cookie');
// 'cookie' is of type string[]
```

### `request.getHeaderNames()`

<!-- YAML
added: v7.7.0
-->

*   返回：{字符串\[]}

返回一个数组，其中包含当前传出标头的唯一名称。
所有标头名称均为小写。

```js
request.setHeader('Foo', 'bar');
request.setHeader('Cookie', ['foo=bar', 'bar=baz']);

const headerNames = request.getHeaderNames();
// headerNames === ['foo', 'cookie']
```

### `request.getHeaders()`

<!-- YAML
added: v7.7.0
-->

*   返回： {对象}

返回当前传出标头的浅层副本。由于浅层复制
，数组值可以发生突变，而无需对各种进行其他调用
与标头相关的 http 模块方法。返回对象的键是
标头名称和值是各自的标头值。所有标头名称
为小写。

返回的对象`response.getHeaders()`方法*不*
原型继承自 JavaScript`Object`.这意味着
`Object`方法，例如`obj.toString()`,`obj.hasOwnProperty()`和其他
未定义且*将不起作用*.

```js
request.setHeader('Foo', 'bar');
request.setHeader('Cookie', ['foo=bar', 'bar=baz']);

const headers = response.getHeaders();
// headers === { foo: 'bar', 'cookie': ['foo=bar', 'bar=baz'] }
```

### `request.getRawHeaderNames()`

<!-- YAML
added:
  - v15.13.0
  - v14.17.0
-->

*   返回：{字符串\[]}

返回一个数组，其中包含当前传出原始数据的唯一名称
头。返回标头名称时，并设置了其确切的大小写。

```js
request.setHeader('Foo', 'bar');
request.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headerNames = request.getRawHeaderNames();
// headerNames === ['Foo', 'Set-Cookie']
```

### `request.hasHeader(name)`

<!-- YAML
added: v7.7.0
-->

*   `name`{字符串}
*   返回：{布尔值}

返回`true`如果标头由`name`当前设置在
传出标头。标头名称匹配不区分大小写。

```js
const hasContentType = request.hasHeader('content-type');
```

### `request.maxHeadersCount`

*   {数字}**违约：** `2000`

限制最大响应标头计数。如果设置为 0，则不会应用任何限制。

### `request.path`

<!-- YAML
added: v0.4.0
-->

*   {字符串}请求路径。

### `request.method`

<!-- YAML
added: v0.1.97
-->

*   {字符串}请求方法。

### `request.host`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

*   {字符串}请求主机。

### `request.protocol`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

*   {字符串}请求协议。

### `request.removeHeader(name)`

<!-- YAML
added: v1.6.0
-->

*   `name`{字符串}

删除已定义到标头对象中的标头。

```js
request.removeHeader('Content-Type');
```

### `request.reusedSocket`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
-->

*   {布尔值}请求是否通过重用的套接字发送。

通过启用了保持活动状态的代理发送请求时，底层套接字
可能会被重复使用。但是，如果服务器在不幸的时间关闭连接，则客户端
可能会遇到“ECONNRESET”错误。

```js
const http = require('node:http');

// Server has a 5 seconds keep-alive timeout by default
http
  .createServer((req, res) => {
    res.write('hello\n');
    res.end();
  })
  .listen(3000);

setInterval(() => {
  // Adapting a keep-alive agent
  http.get('http://localhost:3000', { agent }, (res) => {
    res.on('data', (data) => {
      // Do nothing
    });
  });
}, 5000); // Sending request on 5s interval so it's easy to hit idle timeout
```

通过标记请求是否重用套接字，我们可以做
基于它自动错误重试。

```js
const http = require('node:http');
const agent = new http.Agent({ keepAlive: true });

function retriableRequest() {
  const req = http
    .get('http://localhost:3000', { agent }, (res) => {
      // ...
    })
    .on('error', (err) => {
      // Check if retry is needed
      if (req.reusedSocket && err.code === 'ECONNRESET') {
        retriableRequest();
      }
    });
}

retriableRequest();
```

### `request.setHeader(name, value)`

<!-- YAML
added: v1.6.0
-->

*   `name`{字符串}
*   `value`{任何}

设置标头对象的单个标头值。如果此标头已存在于
要发送的标头，其值将被替换。使用字符串数组
在这里发送多个具有相同名称的标头。非字符串值将为
存储时无需修改。因此[`request.getHeader()`][request.getHeader()]可能返回
非字符串值。但是，非字符串值将转换为字符串
用于网络传输。

```js
request.setHeader('Content-Type', 'application/json');
```

或

```js
request.setHeader('Cookie', ['type=ninja', 'language=javascript']);
```

当值为字符串时，如果包含
字符外部`latin1`编码。

如果您需要在值中传递 UTF-8 字符，请对值进行编码
使用[RFC 8187][]标准。

```js
const filename = 'Rock 🎵.txt';
request.setHeader('Content-Disposition', `attachment; filename*=utf-8''${encodeURIComponent(filename)}`);
```

### `request.setNoDelay([noDelay])`

<!-- YAML
added: v0.5.9
-->

*   `noDelay`{布尔值}

将套接字分配给此请求并连接后
[`socket.setNoDelay()`][socket.setNoDelay()]将被调用。

### `request.setSocketKeepAlive([enable][, initialDelay])`

<!-- YAML
added: v0.5.9
-->

*   `enable`{布尔值}
*   `initialDelay`{数字}

将套接字分配给此请求并连接后
[`socket.setKeepAlive()`][socket.setKeepAlive()]将被调用。

### `request.setTimeout(timeout[, callback])`

<!-- YAML
added: v0.5.9
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/8895
    description: Consistently set socket timeout only when the socket connects.
-->

*   `timeout`{数字}请求超时前的毫秒。
*   `callback`{函数}发生超时时要调用的可选函数。
    与绑定到 相同`'timeout'`事件。
*   返回值：{http。客户端请求}

将套接字分配给此请求并连接后
[`socket.setTimeout()`][socket.setTimeout()]将被调用。

### `request.socket`

<!-- YAML
added: v0.3.0
-->

*   {流。双工}

对基础套接字的引用。通常用户不希望访问
此属性。特别是，套接字不会发出`'readable'`事件
因为协议解析器如何连接到套接字。

```js
const http = require('node:http');
const options = {
  host: 'www.google.com',
};
const req = http.get(options);
req.end();
req.once('response', (res) => {
  const ip = req.socket.localAddress;
  const port = req.socket.localPort;
  console.log(`Your IP address is ${ip} and your source port is ${port}.`);
  // Consume response object
});
```

此属性保证是 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

### `request.uncork()`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

看[`writable.uncork()`][writable.uncork()].

### `request.writableEnded`

<!-- YAML
added: v12.9.0
-->

*   {布尔值}

是`true`后[`request.end()`][request.end()]已被调用。此属性
不指示数据是否已刷新，用于此用途
[`request.writableFinished`][request.writableFinished]相反。

### `request.writableFinished`

<!-- YAML
added: v12.7.0
-->

*   {布尔值}

是`true`如果所有数据都已刷新到底层系统，请立即
在[`'finish'`]['finish']发出事件。

### `request.write(chunk[, encoding][, callback])`

<!-- YAML
added: v0.1.29
-->

*   `chunk`{字符串|缓冲区}
*   `encoding`{字符串}
*   `callback`{函数}
*   返回：{布尔值}

发送正文的一块。可以多次调用此方法。如果不是
`Content-Length`设置后，数据将自动以 HTTP 分块形式进行编码
传输编码，以便服务器知道数据何时结束。这
`Transfer-Encoding: chunked`已添加标头。叫[`request.end()`][request.end()]
是完成发送请求所必需的。

这`encoding`参数是可选的，仅在以下情况下适用`chunk`是一个字符串。
默认值为`'utf8'`.

这`callback`参数是可选的，当此数据块时将调用
已刷新，但仅当块为非空时。

返回`true`如果整个数据已成功刷新到内核
缓冲区。返回`false`如果全部或部分数据在用户内存中排队。
`'drain'`当缓冲区再次释放时，将发出。

什么时候`write`函数是用空字符串或缓冲区调用的，它
没有，等待更多的输入。

## 类：`http.Server`

<!-- YAML
added: v0.1.17
-->

*   扩展：{net.服务器}

### 事件：`'checkContinue'`

<!-- YAML
added: v0.3.0
-->

*   `request`{http.传入消息}
*   `response`{http.ServerResponse}

每次使用 HTTP 发出请求时发出`Expect: 100-continue`已接收。
如果未侦听此事件，服务器将自动响应
与`100 Continue`视情况而定。

处理此事件涉及调用[`response.writeContinue()`][response.writeContinue()]如果
客户端应继续发送请求正文，或生成适当的
HTTP 响应（例如 400 错误请求），如果客户端不应继续发送
请求正文。

当发出并处理此事件时，[`'request'`]['request']事件将
不发出。

### 事件：`'checkExpectation'`

<!-- YAML
added: v5.5.0
-->

*   `request`{http.传入消息}
*   `response`{http.ServerResponse}

每次使用 HTTP 发出请求时发出`Expect`接收标头，其中
值不是`100-continue`.如果未侦听此事件，则服务器将
自动响应`417 Expectation Failed`视情况而定。

当发出并处理此事件时，[`'request'`]['request']事件将
不发出。

### 事件：`'clientError'`

<!-- YAML
added: v0.1.94
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25605
    description: The default behavior will return a 431 Request Header
                 Fields Too Large if a HPE_HEADER_OVERFLOW error occurs.
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/17672
    description: The `rawPacket` is the current buffer that just parsed. Adding
                 this buffer to the error object of `'clientError'` event is to
                 make it possible that developers can log the broken packet.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4557
    description: The default action of calling `.destroy()` on the `socket`
                 will no longer take place if there are listeners attached
                 for `'clientError'`.
-->

*   `exception`{错误}
*   `socket`{流。双工}

如果客户端连接发出`'error'`事件，它将被转发到这里。
此事件的侦听器负责关闭/销毁底层
插座。例如，人们可能希望更优雅地关闭套接字
自定义 HTTP 响应，而不是突然断开连接。

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

默认行为是尝试用 HTTP“400 错误请求”关闭套接字，
或 HTTP “431 请求标头字段太大” （在
[`HPE_HEADER_OVERFLOW`][HPE_HEADER_OVERFLOW]错误。如果套接字不可写或已经
写入的数据会立即被销毁。

`socket`是[`net.Socket`][net.Socket]源自错误的对象。

```js
const http = require('node:http');

const server = http.createServer((req, res) => {
  res.end();
});
server.on('clientError', (err, socket) => {
  socket.end('HTTP/1.1 400 Bad Request\r\n\r\n');
});
server.listen(8000);
```

当`'clientError'`事件发生，没有`request`或`response`
对象，因此发送的任何 HTTP 响应，包括响应标头和有效负载，
*必须*直接写入`socket`对象。必须注意
确保响应是格式正确的 HTTP 响应消息。

`err`是 的实例`Error`包含两个额外的列：

*   `bytesParsed`：Node.js可能已解析的请求数据包的字节数
    正确;
*   `rawPacket`：当前请求的原始数据包。

在某些情况下，客户端已经收到响应和/或套接字
已经被摧毁，就像在`ECONNRESET`错误。以前
尝试将数据发送到套接字，最好检查它是否仍然
写。

```js
server.on('clientError', (err, socket) => {
  if (err.code === 'ECONNRESET' || !socket.writable) {
    return;
  }

  socket.end('HTTP/1.1 400 Bad Request\r\n\r\n');
});
```

### 事件：`'close'`

<!-- YAML
added: v0.1.4
-->

服务器关闭时发出。

### 事件：`'connect'`

<!-- YAML
added: v0.7.0
-->

*   `request`{http.IncomingMessage} HTTP 请求的参数，如
    这[`'request'`]['request']事件
*   `socket`{流。双工} 服务器和客户端之间的网络套接字
*   `head`{缓冲区}隧道流的第一个数据包（可能为空）

每次客户端请求 HTTP 时发出`CONNECT`方法。如果此事件是
未侦听，则客户端请求`CONNECT`方法将有他们的
连接已关闭。

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

发出此事件后，请求的套接字将没有`'data'`
事件侦听器，这意味着它需要绑定才能处理数据
发送到该套接字上的服务器。

### 事件：`'connection'`

<!-- YAML
added: v0.1.0
-->

*   `socket`{流。双工}

建立新的 TCP 流时，将发出此事件。`socket`是
通常是类型的对象[`net.Socket`][net.Socket].通常用户不会想要
访问此事件。特别是，套接字不会发出`'readable'`事件
因为协议解析器如何连接到套接字。这`socket`能
也可在以下位置访问`request.socket`.

用户还可以显式发出此事件以注入连接
进入 HTTP 服务器。在这种情况下，任何[`Duplex`][Duplex]可以传递流。

如果`socket.setTimeout()`在这里调用，超时将替换为
`server.keepAliveTimeout`当套接字已提供请求时（如果
`server.keepAliveTimeout`为非零）。

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

### 事件：`'dropRequest'`

<!-- YAML
added: v18.7.0
-->

*   `request`{http.IncomingMessage} HTTP 请求的参数，如
    这[`'request'`]['request']事件
*   `socket`{流。双工} 服务器和客户端之间的网络套接字

当套接字上的请求数达到阈值
`server.maxRequestsPerSocket`，服务器将丢弃新请求
并发出`'dropRequest'`事件，然后发送`503`到客户端。

### 事件：`'request'`

<!-- YAML
added: v0.1.0
-->

*   `request`{http.传入消息}
*   `response`{http.ServerResponse}

每次有请求时发出。可能有多个请求
每个连接（在 HTTP 保持活动状态连接的情况下）。

### 事件：`'upgrade'`

<!-- YAML
added: v0.1.94
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19981
    description: Not listening to this event no longer causes the socket
                 to be destroyed if a client sends an Upgrade header.
-->

*   `request`{http.IncomingMessage} HTTP 请求的参数，如
    这[`'request'`]['request']事件
*   `socket`{流。双工} 服务器和客户端之间的网络套接字
*   `head`{缓冲区}升级流的第一个数据包（可能为空）

每次客户端请求 HTTP 升级时发出。侦听此事件
是可选的，客户端不能坚持协议更改。

发出此事件后，请求的套接字将没有`'data'`
事件侦听器，这意味着它需要绑定才能处理数据
发送到该套接字上的服务器。

此事件保证传递给 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

### `server.close([callback])`

<!-- YAML
added: v0.1.90
changes:
  - version:
      - REPLACEME
    pr-url: https://github.com/nodejs/node/pull/43522
    description: The method closes idle connections before returning.

-->

*   `callback`{函数}

阻止服务器接受新连接并关闭所有连接
连接到此服务器，这些服务器未发送请求或正在等待
响应。
看[`net.Server.close()`][net.Server.close()].

### `server.closeAllConnections()`

<!-- YAML
added: v18.2.0
-->

关闭连接到此服务器的所有连接。

### `server.closeIdleConnections()`

<!-- YAML
added: v18.2.0
-->

关闭连接到此服务器且未发送请求的所有连接
或等待响应。

### `server.headersTimeout`

<!-- YAML
added:
 - v11.3.0
 - v10.14.0
-->

*   {数字}**违约：** `60000`

限制解析器等待接收完整 HTTP 的时间
头。

如果超时过期，服务器将以状态 408 进行响应，而不带
将请求转发到请求侦听器，然后关闭连接。

必须将其设置为非零值（例如 120 秒）以防止
在部署服务器时出现潜在的拒绝服务攻击
前面的反向代理。

### `server.listen()`

启动侦听连接的 HTTP 服务器。
此方法与[`server.listen()`][server.listen()]从[`net.Server`][net.Server].

### `server.listening`

<!-- YAML
added: v5.7.0
-->

*   {布尔值}指示服务器是否正在侦听连接。

### `server.maxHeadersCount`

<!-- YAML
added: v0.7.0
-->

*   {数字}**违约：** `2000`

限制最大传入标头计数。如果设置为 0，则不会应用任何限制。

### `server.requestTimeout`

<!-- YAML
added: v14.11.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41263
    description: The default request timeout changed
                 from no timeout to 300s (5 minutes).
-->

*   {数字}**违约：** `300000`

设置从 接收整个请求的超时值（以毫秒为单位）
客户端。

如果超时过期，服务器将以状态 408 进行响应，而不带
将请求转发到请求侦听器，然后关闭连接。

必须将其设置为非零值（例如 120 秒）以防止
在部署服务器时出现潜在的拒绝服务攻击
前面的反向代理。

### `server.setTimeout([msecs][, callback])`

<!-- YAML
added: v0.9.12
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27558
    description: The default timeout changed from 120s to 0 (no timeout).
-->

*   `msecs`{数字}**违约：**0（无超时）
*   `callback`{函数}
*   返回值：{http。服务器}

设置套接字的超时值，并发出`'timeout'`事件
服务器对象，如果超时，将套接字作为参数传递
发生。

如果有`'timeout'`服务器对象上的事件侦听器，然后是
将使用超时套接字作为参数进行调用。

默认情况下，服务器不会使套接字超时。但是，如果回调
分配给服务器的`'timeout'`事件，必须处理超时
明确地。

### `server.maxRequestsPerSocket`

<!-- YAML
added: v16.10.0
-->

*   {数字}每个套接字的请求数。**违约：**0（无限制）

套接字可以处理的最大请求数
在关闭之前保持活动状态连接。

值`0`将禁用限制。

当达到限制时，它将设置`Connection`标头值`close`,
但实际上不会关闭连接，后续请求发送
达到限制后将获得`503 Service Unavailable`作为回应。

### `server.timeout`

<!-- YAML
added: v0.9.12
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27558
    description: The default timeout changed from 120s to 0 (no timeout).
-->

*   {数字}超时（以毫秒为单位）。**违约：**0（无超时）

假定套接字之前处于非活动状态的毫秒数
已超时。

值`0`将禁用传入连接的超时行为。

套接字超时逻辑是在连接时设置的，因此请更改此设置
值仅影响与服务器的新连接，而不影响任何现有连接。

### `server.keepAliveTimeout`

<!-- YAML
added: v8.0.0
-->

*   {数字}超时（以毫秒为单位）。**违约：** `5000`（5 秒）。

服务器需要等待的不活动毫秒数
传入数据，在完成写入最后一个响应之后，在套接字之前
将被摧毁。如果服务器在保持活动状态之前收到新数据
超时已触发，它将重置常规不活动超时，即
[`server.timeout`][server.timeout].

值`0`将禁用传入时的保持活动超时行为
连接。
值`0`使 http 服务器的行为类似于之前的 Node.js 版本
到 8.0.0，它没有保持活动超时。

套接字超时逻辑是在连接时设置的，因此仅更改此值
影响到服务器的新连接，而不是任何现有连接。

## 类：`http.ServerResponse`

<!-- YAML
added: v0.1.17
-->

*   扩展：{http.传出消息}

此对象由 HTTP 服务器在内部创建，而不是由用户创建。是的
作为第二个参数传递给[`'request'`]['request']事件。

### 事件：`'close'`

<!-- YAML
added: v0.6.7
-->

指示响应已完成，或者其基础连接为
提前终止（在响应完成之前）。

### 事件：`'finish'`

<!-- YAML
added: v0.3.6
-->

发送响应时发出。更具体地说，此事件是
当响应标头和正文的最后一段为
移交给操作系统通过网络传输。它
并不意味着客户已经收到任何东西。

### `response.addTrailers(headers)`

<!-- YAML
added: v0.3.0
-->

*   `headers`{对象}

此方法添加 HTTP 尾随标头（一个标头，但在
消息）到响应。

预告片将**只**如果分块编码用于
响应;如果不是（例如，如果请求是HTTP / 1.0），他们将
被默默地丢弃。

HTTP 需要`Trailer`要发送的标头，以便
发出尾部，其值中包含标头字段的列表。例如，

```js
response.writeHead(200, { 'Content-Type': 'text/plain',
                          'Trailer': 'Content-MD5' });
response.write(fileData);
response.addTrailers({ 'Content-MD5': '7895bf4b8828b55ceaf47747b4bca667' });
response.end();
```

尝试设置包含无效字符的标头字段名称或值
将导致[`TypeError`][TypeError]被抛出。

### `response.connection`

<!-- YAML
added: v0.3.0
deprecated: v13.0.0
-->

> 稳定性：0 - 已弃用。用[`response.socket`][response.socket].

*   {流。双工}

看[`response.socket`][response.socket].

### `response.cork()`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

看[`writable.cork()`][writable.cork()].

### `response.end([data[, encoding]][, callback])`

<!-- YAML
added: v0.1.90
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18780
    description: This method now returns a reference to `ServerResponse`.
-->

*   `data`{字符串|缓冲区}
*   `encoding`{字符串}
*   `callback`{函数}
*   返回值：{this}

此方法向服务器发出所有响应标头和正文的信号
已发送;该服务器应将此消息视为已完成。
该方法，`response.end()`，则必须在每个响应上调用。

如果`data`指定，其效果类似于调用
[`response.write(data, encoding)`][response.write(data, encoding)]其次`response.end(callback)`.

如果`callback`指定，它将在响应流时调用
已完成。

### `response.finished`

<!-- YAML
added: v0.0.2
deprecated:
 - v13.4.0
 - v12.16.0
-->

> 稳定性：0 - 已弃用。用[`response.writableEnded`][response.writableEnded].

*   {布尔值}

这`response.finished`属性将是`true`如果[`response.end()`][response.end()]
已被调用。

### `response.flushHeaders()`

<!-- YAML
added: v1.6.0
-->

刷新响应标头。另请参阅：[`request.flushHeaders()`][request.flushHeaders()].

### `response.getHeader(name)`

<!-- YAML
added: v0.4.0
-->

*   `name`{字符串}
*   返回值：{任意}

读出已排队但未发送到客户端的标头。
该名称不区分大小写。返回值的类型取决于
关于提供给[`response.setHeader()`][response.setHeader()].

```js
response.setHeader('Content-Type', 'text/html');
response.setHeader('Content-Length', Buffer.byteLength(body));
response.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
const contentType = response.getHeader('content-type');
// contentType is 'text/html'
const contentLength = response.getHeader('Content-Length');
// contentLength is of type number
const setCookie = response.getHeader('set-cookie');
// setCookie is of type string[]
```

### `response.getHeaderNames()`

<!-- YAML
added: v7.7.0
-->

*   返回：{字符串\[]}

返回一个数组，其中包含当前传出标头的唯一名称。
所有标头名称均为小写。

```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headerNames = response.getHeaderNames();
// headerNames === ['foo', 'set-cookie']
```

### `response.getHeaders()`

<!-- YAML
added: v7.7.0
-->

*   返回： {对象}

返回当前传出标头的浅层副本。由于浅层复制
，数组值可以发生突变，而无需对各种进行其他调用
与标头相关的 http 模块方法。返回对象的键是
标头名称和值是各自的标头值。所有标头名称
为小写。

返回的对象`response.getHeaders()`方法*不*
原型继承自 JavaScript`Object`.这意味着
`Object`方法，例如`obj.toString()`,`obj.hasOwnProperty()`和其他
未定义且*将不起作用*.

```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headers = response.getHeaders();
// headers === { foo: 'bar', 'set-cookie': ['foo=bar', 'bar=baz'] }
```

### `response.hasHeader(name)`

<!-- YAML
added: v7.7.0
-->

*   `name`{字符串}
*   返回：{布尔值}

返回`true`如果标头由`name`当前设置在
传出标头。标头名称匹配不区分大小写。

```js
const hasContentType = response.hasHeader('content-type');
```

### `response.headersSent`

<!-- YAML
added: v0.9.3
-->

*   {布尔值}

布尔值（只读）。如果发送了标头，则为 true，否则为 false。

### `response.removeHeader(name)`

<!-- YAML
added: v0.4.0
-->

*   `name`{字符串}

删除排队等待隐式发送的标头。

```js
response.removeHeader('Content-Encoding');
```

### `response.req`

<!-- YAML
added: v15.7.0
-->

*   {http.传入消息}

对原始 HTTP 的引用`request`对象。

### `response.sendDate`

<!-- YAML
added: v0.7.5
-->

*   {布尔值}

如果为 true，则将自动生成日期标头并发送
如果标头中尚不存在响应，则为响应。默认值为 true。

这应该只在测试时禁用;HTTP 需要日期标头
在回应中。

### `response.setHeader(name, value)`

<!-- YAML
added: v0.4.0
-->

*   `name`{字符串}
*   `value`{任何}
*   返回值：{http。ServerResponse}

返回响应对象。

为隐式标头设置单个标头值。如果此标头已存在
在要发送的标头中，其值将被替换。使用字符串数组
在这里发送多个具有相同名称的标头。非字符串值将为
存储时无需修改。因此[`response.getHeader()`][response.getHeader()]可能返回
非字符串值。但是，非字符串值将转换为字符串
用于网络传输。将相同的响应对象返回给调用方，
以启用呼叫链接。

```js
response.setHeader('Content-Type', 'text/html');
```

或

```js
response.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
```

尝试设置包含无效字符的标头字段名称或值
将导致[`TypeError`][TypeError]被抛出。

当标头已设置为[`response.setHeader()`][response.setHeader()]，它们将被合并
将任何标头传递给[`response.writeHead()`][response.writeHead()]，并传递标头
自[`response.writeHead()`][response.writeHead()]给定优先级。

```js
// Returns content-type = text/plain
const server = http.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

如果[`response.writeHead()`][response.writeHead()]方法被调用，但此方法尚未
调用，它将直接将提供的标头值写入网络
不进行内部缓存的通道，以及[`response.getHeader()`][response.getHeader()]在
标头将不会产生预期的结果。如果逐行填充标头
是需要与潜在的未来检索和修改， 使用
[`response.setHeader()`][response.setHeader()]而不是[`response.writeHead()`][response.writeHead()].

### `response.setTimeout(msecs[, callback])`

<!-- YAML
added: v0.9.12
-->

*   `msecs`{数字}
*   `callback`{函数}
*   返回值：{http。ServerResponse}

将套接字的超时值设置为`msecs`.如果回调是
提供，然后将其添加为侦听器`'timeout'`事件
响应对象。

如果不是`'timeout'`侦听器添加到请求、响应或
服务器，然后套接字在超时时被销毁。如果处理程序是
分配给请求、响应或服务器的`'timeout'`事件
必须显式处理超时套接字。

### `response.socket`

<!-- YAML
added: v0.3.0
-->

*   {流。双工}

对基础套接字的引用。通常用户不希望访问
此属性。特别是，套接字不会发出`'readable'`事件
因为协议解析器如何连接到套接字。后
`response.end()`，则该属性为空。

```js
const http = require('node:http');
const server = http.createServer((req, res) => {
  const ip = res.socket.remoteAddress;
  const port = res.socket.remotePort;
  res.end(`Your IP address is ${ip} and your source port is ${port}.`);
}).listen(3000);
```

此属性保证是 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字}。

### `response.statusCode`

<!-- YAML
added: v0.4.0
-->

*   {数字}**违约：** `200`

使用隐式标头时（不调用[`response.writeHead()`][response.writeHead()]明确），
此属性控制在以下情况下将发送到客户端的状态代码
标头被刷新。

```js
response.statusCode = 404;
```

将响应标头发送到客户端后，此属性指示
已发出的状态代码。

### `response.statusMessage`

<!-- YAML
added: v0.11.8
-->

*   {字符串}

使用隐式标头时（不调用[`response.writeHead()`][response.writeHead()]明确），
此属性控制在以下情况下将发送到客户端的状态消息
标头被刷新。如果将其保留为`undefined`然后是标准
将使用状态代码的消息。

```js
response.statusMessage = 'Not found';
```

将响应标头发送到客户端后，此属性指示
已发出的状态消息。

### `response.uncork()`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

看[`writable.uncork()`][writable.uncork()].

### `response.writableEnded`

<!-- YAML
added: v12.9.0
-->

*   {布尔值}

是`true`后[`response.end()`][response.end()]已被调用。此属性
不指示数据是否已刷新，用于此用途
[`response.writableFinished`][response.writableFinished]相反。

### `response.writableFinished`

<!-- YAML
added: v12.7.0
-->

*   {布尔值}

是`true`如果所有数据都已刷新到底层系统，请立即
在[`'finish'`]['finish']发出事件。

### `response.write(chunk[, encoding][, callback])`

<!-- YAML
added: v0.1.29
-->

*   `chunk`{字符串|缓冲区}
*   `encoding`{字符串}**违约：** `'utf8'`
*   `callback`{函数}
*   返回：{布尔值}

如果调用此方法，并且[`response.writeHead()`][response.writeHead()]尚未调用，
它将切换到隐式标头模式并刷新隐式标头。

这将发送响应正文的一部分。此方法可能
被多次调用以提供身体的连续部位。

在`node:http`模块，当
请求是一个 HEAD 请求。同样，`204`和`304`反应
*莫*包括邮件正文。

`chunk`可以是字符串或缓冲区。如果`chunk`是一个字符串，
第二个参数指定如何将其编码为字节流。
`callback`将在刷新此数据块时调用。

这是原始的HTTP体，与更高级别的多部分无关
可以使用的正文编码。

第一次[`response.write()`][response.write()]被调用，它将发送缓冲
标头信息和客户端的第一个正文块。第二个
时间[`response.write()`][response.write()]被调用，节点.js假设数据将被流式传输，
并单独发送新数据。也就是说，响应被缓冲到
身体的第一块。

返回`true`如果整个数据已成功刷新到内核
缓冲区。返回`false`如果全部或部分数据在用户内存中排队。
`'drain'`当缓冲区再次释放时，将发出。

### `response.writeContinue()`

<!-- YAML
added: v0.3.0
-->

向客户端发送 HTTP/1.1 100 Continue 消息，指示
应发送请求正文。查看[`'checkContinue'`]['checkContinue']事件
`Server`.

### `response.writeHead(statusCode[, statusMessage][, headers])`

<!-- YAML
added: v0.1.30
changes:
  - version: v14.14.0
    pr-url: https://github.com/nodejs/node/pull/35274
    description: Allow passing headers as an array.
  - version:
     - v11.10.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/25974
    description: Return `this` from `writeHead()` to allow chaining with
                 `end()`.
  - version:
    - v5.11.0
    - v4.4.5
    pr-url: https://github.com/nodejs/node/pull/6291
    description: A `RangeError` is thrown if `statusCode` is not a number in
                 the range `[100, 999]`.
-->

*   `statusCode`{数字}
*   `statusMessage`{字符串}
*   `headers`{对象|数组}
*   返回值：{http。ServerResponse}

将响应标头发送到请求。状态代码为 3 位 HTTP
状态代码，如`404`.最后一个论点，`headers`是响应标头。
（可选）可以给出人类可读的`statusMessage`作为第二个
论点。

`headers`可能是`Array`其中，键和值位于同一列表中。
是的*不*元组列表。因此，偶数偏移量是关键值，
奇数偏移量是关联的值。数组位于同一位置
格式为`request.rawHeaders`.

返回对`ServerResponse`，以便可以链接调用。

```js
const body = 'hello world';
response
  .writeHead(200, {
    'Content-Length': Buffer.byteLength(body),
    'Content-Type': 'text/plain'
  })
  .end(body);
```

此方法只能在消息上调用一次，并且必须
之前被调用[`response.end()`][response.end()]被调用。

如果[`response.write()`][response.write()]或[`response.end()`][response.end()]在呼叫之前被调用
这将计算隐式/可变标头并调用此函数。

当标头已设置为[`response.setHeader()`][response.setHeader()]，它们将被合并
将任何标头传递给[`response.writeHead()`][response.writeHead()]，并传递标头
自[`response.writeHead()`][response.writeHead()]给定优先级。

如果调用此方法，并且[`response.setHeader()`][response.setHeader()]尚未调用，
它将直接将提供的标头值写入网络通道
没有内部缓存，并且[`response.getHeader()`][response.getHeader()]在页眉上
不会产生预期的结果。如果标头的渐进式填充为
希望与潜在的未来检索和修改，使用
[`response.setHeader()`][response.setHeader()]相反。

```js
// Returns content-type = text/plain
const server = http.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

`Content-Length`以字节为单位，而不是以字符为单位。用
[`Buffer.byteLength()`][Buffer.byteLength()]以确定正文的长度（以字节为单位）。节点.js
不检查是否`Content-Length`以及具有以下特征的身体长度
已传输是否相等。

尝试设置包含无效字符的标头字段名称或值
将导致[`TypeError`][TypeError]被抛出。

### `response.writeProcessing()`

<!-- YAML
added: v10.0.0
-->

向客户端发送 HTTP/1.1 102 处理消息，指示
应发送请求正文。

## 类：`http.IncomingMessage`

<!-- YAML
added: v0.1.17
changes:
  - version: v15.5.0
    pr-url: https://github.com/nodejs/node/pull/33035
    description: The `destroyed` value returns `true` after the incoming data
                 is consumed.
  - version:
     - v13.1.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30135
    description: The `readableHighWaterMark` value mirrors that of the socket.
-->

*   扩展：{流。可读}

一`IncomingMessage`对象由以下人员创建[`http.Server`][http.Server]或
[`http.ClientRequest`][http.ClientRequest]并作为第一个参数传递给[`'request'`]['request']
和[`'response'`]['response']事件分别。它可用于访问响应
状态、标头和数据。

不同于它`socket`值，它是 {stream 的子类。双工}，
`IncomingMessage`本身扩展 {流。可读} 并单独创建到
解析并发出传入的 HTTP 标头和有效负载，作为底层套接字
在保持活动状态的情况下可以多次重复使用。

### 事件：`'aborted'`

<!-- YAML
added: v0.3.8
deprecated:
  - v17.0.0
  - v16.12.0
-->

> 稳定性：0 - 已弃用。收听`'close'`事件。

在请求中止时发出。

### 事件：`'close'`

<!-- YAML
added: v0.4.2
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/33035
    description: The close event is now emitted when the request has been completed and not when the
                 underlying socket is closed.
-->

请求完成时发出。

### `message.aborted`

<!-- YAML
added: v10.1.0
deprecated:
  - v17.0.0
  - v16.12.0
-->

> 稳定性：0 - 已弃用。检查`message.destroyed`从 {流.可读}。

*   {布尔值}

这`message.aborted`属性将是`true`如果请求有
已中止。

### `message.complete`

<!-- YAML
added: v0.3.0
-->

*   {布尔值}

这`message.complete`属性将是`true`如果完整的 HTTP 消息具有
已接收并成功解析。

此属性作为确定客户端或
服务器在连接终止之前完全传输了消息：

```js
const req = http.request({
  host: '127.0.0.1',
  port: 8080,
  method: 'POST'
}, (res) => {
  res.resume();
  res.on('end', () => {
    if (!res.complete)
      console.error(
        'The connection was terminated while the message was still being sent');
  });
});
```

### `message.connection`

<!-- YAML
added: v0.1.90
deprecated: v16.0.0
 -->

> 稳定性：0 - 已弃用。用[`message.socket`][message.socket].

的别名[`message.socket`][message.socket].

### `message.destroy([error])`

<!-- YAML
added: v0.3.0
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/32789
    description: The function returns `this` for consistency with other Readable
                 streams.
-->

*   `error`{错误}
*   返回值：{this}

调用`destroy()`在接收到`IncomingMessage`.如果`error`
提供，一个`'error'`事件在套接字上发出，并且`error`已通过
作为事件上任何侦听器的参数。

### `message.headers`

<!-- YAML
added: v0.1.5
changes:
  - version: v15.1.0
    pr-url: https://github.com/nodejs/node/pull/35281
    description: >-
      `message.headers` is now lazily computed using an accessor property
      on the prototype and is no longer enumerable.
-->

*   {对象}

请求/响应标头对象。

标头名称和值的键值对。标头名称为小写。

```js
// Prints something like:
//
// { 'user-agent': 'curl/7.22.0',
//   host: '127.0.0.1:8000',
//   accept: '*/*' }
console.log(request.getHeaders());
```

原始标头中的重复项按以下方式处理，具体取决于
标头名称：

*   的副本`age`,`authorization`,`content-length`,`content-type`,
    `etag`,`expires`,`from`,`host`,`if-modified-since`,`if-unmodified-since`,
    `last-modified`,`location`,`max-forwards`,`proxy-authorization`,`referer`,
    `retry-after`,`server`或`user-agent`被丢弃。
*   `set-cookie`始终是一个数组。重复项将添加到数组中。
*   对于重复`cookie`标头，值与` ;  `.
*   对于所有其他标头，这些值与` ,  `.

### `message.headersDistinct`

<!-- YAML
added: v18.3.0
-->

*   {对象}

似[`message.headers`][message.headers]，但没有连接逻辑，值为
始终是字符串数组，即使对于仅接收一次的标头也是如此。

```js
// Prints something like:
//
// { 'user-agent': ['curl/7.22.0'],
//   host: ['127.0.0.1:8000'],
//   accept: ['*/*'] }
console.log(request.headersDistinct);
```

### `message.httpVersion`

<!-- YAML
added: v0.1.1
-->

*   {字符串}

在服务器请求的情况下，由客户端发送HTTP版本。在以下情况下
客户端响应，连接到的服务器的 HTTP 版本。
可能要么`'1.1'`或`'1.0'`.

也`message.httpVersionMajor`是第一个整数，并且
`message.httpVersionMinor`是第二个。

### `message.method`

<!-- YAML
added: v0.1.1
-->

*   {字符串}

**仅对从以下位置获得的请求有效[`http.Server`][http.Server].**

字符串形式的请求方法。只读。例子：`'GET'`,`'DELETE'`.

### `message.rawHeaders`

<!-- YAML
added: v0.11.6
-->

*   {字符串\[]}

原始请求/响应标头与接收时完全相同。

键和值位于同一列表中。是的*不*一个
元组列表。因此，偶数偏移量是键值，并且
奇数编号的偏移量是关联的值。

标头名称不小写，并且不合并重复项。

```js
// Prints something like:
//
// [ 'user-agent',
//   'this is invalid because there can be only one',
//   'User-Agent',
//   'curl/7.22.0',
//   'Host',
//   '127.0.0.1:8000',
//   'ACCEPT',
//   '*/*' ]
console.log(request.rawHeaders);
```

### `message.rawTrailers`

<!-- YAML
added: v0.11.6
-->

*   {字符串\[]}

原始请求/响应拖车键和值与它们完全相同
收到。仅填充于`'end'`事件。

### `message.setTimeout(msecs[, callback])`

<!-- YAML
added: v0.5.9
-->

*   `msecs`{数字}
*   `callback`{函数}
*   返回值：{http。传入消息}

调用`message.socket.setTimeout(msecs, callback)`.

### `message.socket`

<!-- YAML
added: v0.3.0
-->

*   {流。双工}

这[`net.Socket`][net.Socket]与连接关联的对象。

在 HTTPS 支持下，使用[`request.socket.getPeerCertificate()`][request.socket.getPeerCertificate()]以获得
客户端的身份验证详细信息。

此属性保证是 {net 的实例。套接字} 类，
{流的子类。双工}，除非用户指定了套接字
{net.套接字} 或内部为空。

### `message.statusCode`

<!-- YAML
added: v0.1.1
-->

*   {数字}

**仅对从以下位置获得的响应有效[`http.ClientRequest`][http.ClientRequest].**

3 位 HTTP 响应状态代码。例如`404`.

### `message.statusMessage`

<!-- YAML
added: v0.11.10
-->

*   {字符串}

**仅对从以下位置获得的响应有效[`http.ClientRequest`][http.ClientRequest].**

HTTP 响应状态消息（原因短语）。例如`OK`或`Internal Server
Error`.

### `message.trailers`

<!-- YAML
added: v0.3.0
-->

*   {对象}

请求/响应尾部对象。仅填充于`'end'`事件。

### `message.trailersDistinct`

<!-- YAML
added: v18.3.0
-->

*   {对象}

似[`message.trailers`][message.trailers]，但没有连接逻辑，值为
始终是字符串数组，即使对于仅接收一次的标头也是如此。
仅填充于`'end'`事件。

### `message.url`

<!-- YAML
added: v0.1.90
-->

*   {字符串}

**仅对从以下位置获得的请求有效[`http.Server`][http.Server].**

请求 URL 字符串。这仅包含实际中存在的 URL
HTTP 请求。请提出以下请求：

```http
GET /status?name=ryan HTTP/1.1
Accept: text/plain
```

要将 URL 解析为其各个部分，请执行以下操作：

```js
new URL(request.url, `http://${request.getHeaders().host}`);
```

什么时候`request.url`是`'/status?name=ryan'`和
`request.getHeaders().host`是`'localhost:3000'`:

```console
$ node
> new URL(request.url, `http://${request.getHeaders().host}`)
URL {
  href: 'http://localhost:3000/status?name=ryan',
  origin: 'http://localhost:3000',
  protocol: 'http:',
  username: '',
  password: '',
  host: 'localhost:3000',
  hostname: 'localhost',
  port: '3000',
  pathname: '/status',
  search: '?name=ryan',
  searchParams: URLSearchParams { 'name' => 'ryan' },
  hash: ''
}
```

## 类：`http.OutgoingMessage`

<!-- YAML
added: v0.1.17
-->

*   扩展：{流}

此类充当 的父类[`http.ClientRequest`][http.ClientRequest]
和[`http.ServerResponse`][http.ServerResponse].它是一个抽象的传出消息，来自
HTTP 事务参与者的视角。

### 事件：`'drain'`

<!-- YAML
added: v0.3.6
-->

当消息的缓冲区再次释放时发出。

### 事件：`'finish'`

<!-- YAML
added: v0.1.17
-->

传输成功完成后发出。

### 事件：`'prefinish'`

<!-- YAML
added: v0.11.6
-->

发出后`outgoingMessage.end()`被调用。
发出事件时，所有数据都已处理，但不一定
完全冲洗。

### `outgoingMessage.addTrailers(headers)`

<!-- YAML
added: v0.3.0
-->

*   `headers`{对象}

将 HTTP 尾部（标头，但在消息末尾）添加到消息中。

预告片将**只**如果消息是分块编码的，则发出。如果没有，
拖车将被默默丢弃。

HTTP 需要`Trailer`要发送的标头以发出预告片，
在其值中包含标题字段名称的列表，例如

```js
message.writeHead(200, { 'Content-Type': 'text/plain',
                         'Trailer': 'Content-MD5' });
message.write(fileData);
message.addTrailers({ 'Content-MD5': '7895bf4b8828b55ceaf47747b4bca667' });
message.end();
```

尝试设置包含无效字符的标头字段名称或值
将导致`TypeError`被抛出。

### `outgoingMessage.appendHeader(name, value)`

<!-- YAML
added: v18.3.0
-->

*   `name`{字符串}标头名称
*   `value`{字符串|字符串\[]}标头值
*   返回值：{this}

为标头对象追加单个标头值。

如果值是数组，则等效于调用此方法多个
次。

如果标头之前没有值，则这等效于调用
[`outgoingMessage.setHeader(name, value)`][outgoingMessage.setHeader(name, value)].

取决于`options.uniqueHeaders`当客户端请求或
服务器已创建，这将最终在标头中多次发送或
单个时间，其中的值连接使用` ;  `.

### `outgoingMessage.connection`

<!-- YAML
added: v0.3.0
deprecated:
  - v15.12.0
  - v14.17.1
-->

> 稳定性：0 - 已弃用：使用[`outgoingMessage.socket`][outgoingMessage.socket]相反。

的别名[`outgoingMessage.socket`][outgoingMessage.socket].

### `outgoingMessage.cork()`

<!-- YAML
added:
  - v13.2.0
  - v12.16.0
-->

看[`writable.cork()`][writable.cork()].

### `outgoingMessage.destroy([error])`

<!-- YAML
added: v0.3.0
-->

*   `error`{错误}可选，要发出的错误`error`事件
*   返回值：{this}

销毁邮件。一旦套接字与消息相关联
并且连接在内，该套接字也将被摧毁。

### `outgoingMessage.end(chunk[, encoding][, callback])`

<!-- YAML
added: v0.1.90
changes:
  - version: v0.11.6
    description: add `callback` argument.
-->

*   `chunk`{字符串|缓冲区}
*   `encoding`{字符串}自选**违约**:`utf8`
*   `callback`{函数}自选
*   返回值：{this}

完成传出消息。如果身体的任何部位未被发送，它将
将它们刷新到基础系统。如果消息被分块，它将
发送终止块`0\r\n\r\n`，然后发送预告片（如果有）。

如果`chunk`已指定，它等效于调用
`outgoingMessage.write(chunk, encoding)`其次
`outgoingMessage.end(callback)`.

如果`callback`提供，消息完成后将调用它
（相当于`'finish'`事件）。

### `outgoingMessage.flushHeaders()`

<!-- YAML
added: v1.6.0
-->

刷新邮件头。

出于效率原因，Node.js 通常会缓冲消息头
直到`outgoingMessage.end()`被调用或消息数据的第一个块
是写的。然后，它尝试将标头和数据打包到单个 TCP 中。
包。

它通常是需要的（它保存TCP往返），但不是当第一个
数据直到很久以后才会发送。`outgoingMessage.flushHeaders()`
绕过优化并启动消息。

### `outgoingMessage.getHeader(name)`

<!-- YAML
added: v0.4.0
-->

*   `name`{字符串}标头的名称
*   返回 {字符串 | 未定义}

获取具有给定名称的 HTTP 标头的值。如果该标头不是
设置，返回值将为`undefined`.

### `outgoingMessage.getHeaderNames()`

<!-- YAML
added: v7.7.0
-->

*   返回 {字符串\[]}

返回一个数组，其中包含当前传出标头的唯一名称。
所有名称均为小写。

### `outgoingMessage.getHeaders()`

<!-- YAML
added: v7.7.0
-->

*   返回： {对象}

返回当前传出标头的浅层副本。由于浅层
使用 copy，数组值可以发生突变，而无需额外调用
各种与标头相关的 HTTP 模块方法。返回的密钥
对象是标头名称，值是相应的标头
值。所有标头名称均为小写。

返回的对象`outgoingMessage.getHeaders()`方法
不是从 JavaScript 继承的原型`Object`.这意味着
典型`Object`方法，例如`obj.toString()`,`obj.hasOwnProperty()`,
其他的没有定义，也不会起作用。

```js
outgoingMessage.setHeader('Foo', 'bar');
outgoingMessage.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headers = outgoingMessage.getHeaders();
// headers === { foo: 'bar', 'set-cookie': ['foo=bar', 'bar=baz'] }
```

### `outgoingMessage.hasHeader(name)`

<!-- YAML
added: v7.7.0
-->

*   `name`{字符串}
*   返回 {布尔值}

返回`true`如果标头由`name`当前设置在
传出标头。标头名称不区分大小写。

```js
const hasContentType = outgoingMessage.hasHeader('content-type');
```

### `outgoingMessage.headersSent`

<!-- YAML
added: v0.9.3
-->

*   {布尔值}

只读。`true`如果标头已发送，否则`false`.

### `outgoingMessage.pipe()`

<!-- YAML
added: v9.0.0
-->

覆盖`stream.pipe()`从旧版继承的方法`Stream`类
这是 的父类`http.OutgoingMessage`.

调用此方法将抛出一个`Error`因为`outgoingMessage`是一个
只写流。

### `outgoingMessage.removeHeader(name)`

<!-- YAML
added: v0.4.0
-->

*   `name`{字符串}标头名称

删除排队等待隐式发送的标头。

```js
outgoingMessage.removeHeader('Content-Encoding');
```

### `outgoingMessage.setHeader(name, value)`

<!-- YAML
added: v0.4.0
-->

*   `name`{字符串}标头名称
*   `value`{任何}标头值
*   返回值：{this}

设置单个标头值。如果标头已存在于要发送的
标头，其值将被替换。使用字符串数组发送多个
具有相同名称的标头。

### `outgoingMessage.setTimeout(msesc[, callback])`

<!-- YAML
added: v0.9.12
-->

*   `msesc`{数字}
*   `callback`{函数}超时时时要调用的可选函数
    发生。与绑定到 相同`timeout`事件。
*   返回值：{this}

一旦套接字与消息关联并已连接，
[`socket.setTimeout()`][socket.setTimeout()]将调用`msecs`作为第一个参数。

### `outgoingMessage.socket`

<!-- YAML
added: v0.3.0
-->

*   {流。双工}

对基础套接字的引用。通常，用户将不希望访问
此属性。

呼叫后`outgoingMessage.end()`，则此属性将为空。

### `outgoingMessage.uncork()`

<!-- YAML
added:
  - v13.2.0
  - v12.16.0
-->

看[`writable.uncork()`][writable.uncork()]

### `outgoingMessage.writableCorked`

<!-- YAML
added:
  - v13.2.0
  - v12.16.0
-->

*   {数字}

次数`outgoingMessage.cork()`已被调用。

### `outgoingMessage.writableEnded`

<!-- YAML
added: v12.9.0
-->

*   {布尔值}

是`true`如果`outgoingMessage.end()`已被调用。此属性
不指示数据是否已刷新。为此，请使用
`message.writableFinished`相反。

### `outgoingMessage.writableFinished`

<!-- YAML
added: v12.7.0
-->

*   {布尔值}

是`true`如果所有数据都已刷新到基础系统。

### `outgoingMessage.writableHighWaterMark`

<!-- YAML
added: v12.9.0
-->

*   {数字}

这`highWaterMark`的基础套接字（如果已分配）。否则，默认值
缓冲区级别在[`writable.write()`][writable.write()]开始返回 false （`16384`).

### `outgoingMessage.writableLength`

<!-- YAML
added: v12.9.0
-->

*   {数字}

缓冲字节数。

### `outgoingMessage.writableObjectMode`

<!-- YAML
added: v12.9.0
-->

*   {布尔值}

总是`false`.

### `outgoingMessage.write(chunk[, encoding][, callback])`

<!-- YAML
added: v0.1.29
changes:
  - version: v0.11.6
    description: The `callback` argument was added.
-->

*   `chunk`{字符串|缓冲区}
*   `encoding`{字符串}**违约**:`utf8`
*   `callback`{函数}
*   返回 {布尔值}

发送正文的一块。可以多次调用此方法。

这`encoding`参数仅在以下情况下相关`chunk`是一个字符串。默认值为
`'utf8'`.

这`callback`参数是可选的，当此数据块时将调用
已刷新。

返回`true`如果整个数据已成功刷新到内核
缓冲区。返回`false`如果全部或部分数据在用户中排队
记忆。这`'drain'`当缓冲区再次可用时，将发出事件。

## `http.METHODS`

<!-- YAML
added: v0.11.8
-->

*   {字符串\[]}

分析器支持的 HTTP 方法的列表。

## `http.STATUS_CODES`

<!-- YAML
added: v0.1.22
-->

*   {对象}

所有标准 HTTP 响应状态代码的集合，以及
每个的简短描述。例如`http.STATUS_CODES[404] === 'Not
Found'`.

## `http.createServer([options][, requestListener])`

<!-- YAML
added: v0.1.13
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41263
    description: The `requestTimeout`, `headersTimeout`, `keepAliveTimeout`, and
                 `connectionsCheckingInterval` options are supported now.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/42163
    description: The `noDelay` option now defaults to `true`.
  - version:
    - v17.7.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41310
    description: The `noDelay`, `keepAlive` and `keepAliveInitialDelay`
                 options are supported now.
  - version:
     - v13.8.0
     - v12.15.0
     - v10.19.0
    pr-url: https://github.com/nodejs/node/pull/31448
    description: The `insecureHTTPParser` option is supported now.
  - version: v13.3.0
    pr-url: https://github.com/nodejs/node/pull/30570
    description: The `maxHeaderSize` option is supported now.
  - version:
    - v9.6.0
    - v8.12.0
    pr-url: https://github.com/nodejs/node/pull/15752
    description: The `options` argument is supported now.
-->

*   `options`{对象}
    *   `IncomingMessage`{http.传入消息} 指定`IncomingMessage`
        要使用的类。有助于扩展原始文件`IncomingMessage`.
        **违约：** `IncomingMessage`.
    *   `ServerResponse`{http.ServerResponse} 指定`ServerResponse`类
        待使用。有助于扩展原始文件`ServerResponse`.**违约：**
        `ServerResponse`.
    *   `requestTimeout`：设置接收的超时值（以毫秒为单位）
        来自客户端的整个请求。
        看[`server.requestTimeout`][server.requestTimeout]了解更多信息。
        **违约：** `300000`.
    *   `headersTimeout`：设置接收的超时值（以毫秒为单位）
        来自客户端的完整 HTTP 标头。
        看[`server.headersTimeout`][server.headersTimeout]了解更多信息。
        **违约：** `60000`.
    *   `keepAliveTimeout`：服务器处于非活动状态的毫秒数
        需要等待其他传入数据，在它完成写入后
        最后一个响应，在套接字将被销毁之前。
        看[`server.keepAliveTimeout`][server.keepAliveTimeout]了解更多信息。
        **违约：** `5000`.
    *   `connectionsCheckingInterval`：将间隔值（以毫秒为单位）设置为
        检查不完整请求中的请求和标头超时。
        **违约：** `30000`.
    *   `insecureHTTPParser`{布尔值}使用不安全的 HTTP 解析器，该解析器接受
        在以下情况下存在无效的 HTTP 标头`true`.使用不安全的解析器应该是
        避免。看[`--insecure-http-parser`][--insecure-http-parser]了解更多信息。
        **违约：** `false`
    *   `maxHeaderSize`{数字}（可选）覆盖
        [`--max-http-header-size`][--max-http-header-size]对于此服务器收到的请求，即
        请求标头的最大长度（以字节为单位）。
        **违约：**16384 （16 KiB）.
    *   `noDelay`{布尔值}如果设置为`true`，它禁用了Nagle的
        在收到新的传入连接后立即执行算法。
        **违约：** `true`.
    *   `keepAlive`{布尔值}如果设置为`true`，它支持保持活动状态功能
        在收到新的传入连接后立即在套接字上，
        类似地，在 \[`socket.setKeepAlive([enable][, initialDelay])`]\[`socket.setKeepAlive(enable, initialDelay)`].
        **违约：** `false`.
    *   `keepAliveInitialDelay`{数字}如果设置为正数，则将
        在空闲套接字上发送第一个保持活动状态探测之前的初始延迟。
        **违约：** `0`.
    *   `uniqueHeaders`{数组}应仅发送的响应标头的列表
        一次。如果标头的值是数组，则将联接项目
        用` ;  `.

*   `requestListener`{函数}

*   返回值：{http。服务器}

返回 的新实例[`http.Server`][http.Server].

这`requestListener`是一个自动的函数
添加到[`'request'`]['request']事件。

```cjs
const http = require('node:http');

// Create a local server to receive data from
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!'
  }));
});

server.listen(8000);
```

```cjs
const http = require('node:http');

// Create a local server to receive data from
const server = http.createServer();

// Listen to the request event
server.on('request', (request, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!'
  }));
});

server.listen(8000);
```

## `http.get(options[, callback])`

## `http.get(url[, options][, callback])`

<!-- YAML
added: v0.3.6
changes:
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/21616
    description: The `url` parameter can now be passed along with a separate
                 `options` object.
  - version: v7.5.0
    pr-url: https://github.com/nodejs/node/pull/10638
    description: The `options` parameter can be a WHATWG `URL` object.
-->

*   `url`{字符串|网址}
*   `options`{对象}接受相同的`options`如
    [`http.request()`][http.request()]，其中`method`始终设置为`GET`.
    从原型继承的属性将被忽略。
*   `callback`{函数}
*   返回值：{http。客户端请求}

由于大多数请求都是没有正文的 GET 请求，因此 Node.js 提供了此功能
方便的方法。此方法与
[`http.request()`][http.request()]是它将方法设置为GET并调用`req.end()`
自然而然。回调必须注意使用响应
数据原因载于[`http.ClientRequest`][http.ClientRequest]部分。

这`callback`使用单个参数调用，该参数是
[`http.IncomingMessage`][http.IncomingMessage].

JSON 抓取示例：

```js
http.get('http://localhost:8000/', (res) => {
  const { statusCode } = res;
  const contentType = res.headers['content-type'];

  let error;
  // Any 2xx status code signals a successful response but
  // here we're only checking for 200.
  if (statusCode !== 200) {
    error = new Error('Request Failed.\n' +
                      `Status Code: ${statusCode}`);
  } else if (!/^application\/json/.test(contentType)) {
    error = new Error('Invalid content-type.\n' +
                      `Expected application/json but received ${contentType}`);
  }
  if (error) {
    console.error(error.message);
    // Consume response data to free up memory
    res.resume();
    return;
  }

  res.setEncoding('utf8');
  let rawData = '';
  res.on('data', (chunk) => { rawData += chunk; });
  res.on('end', () => {
    try {
      const parsedData = JSON.parse(rawData);
      console.log(parsedData);
    } catch (e) {
      console.error(e.message);
    }
  });
}).on('error', (e) => {
  console.error(`Got error: ${e.message}`);
});

// Create a local server to receive data from
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!'
  }));
});

server.listen(8000);
```

## `http.globalAgent`

<!-- YAML
added: v0.5.9
changes:
  - version:
      - REPLACEME
    pr-url: https://github.com/nodejs/node/pull/43522
    description: The agent now uses HTTP Keep-Alive by default.
-->

*   {http.代理}

的全局实例`Agent`它被用作所有HTTP客户端的默认值
请求。

## `http.maxHeaderSize`

<!-- YAML
added:
 - v11.6.0
 - v10.15.0
-->

*   {数字}

只读属性，指定 HTTP 标头允许的最大大小（以字节为单位）。
默认值为 16 KiB。可使用[`--max-http-header-size`][--max-http-header-size]命令行界面
选择。

对于服务器和客户端请求，可以通过传递
`maxHeaderSize`选择。

## `http.request(options[, callback])`

## `http.request(url[, options][, callback])`

<!-- YAML
added: v0.3.6
changes:
  - version:
      - v16.7.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/39310
    description: When using a `URL` object parsed username and
                 password will now be properly URI decoded.
  - version:
      - v15.3.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/36048
    description: It is possible to abort a request with an AbortSignal.
  - version:
     - v13.8.0
     - v12.15.0
     - v10.19.0
    pr-url: https://github.com/nodejs/node/pull/31448
    description: The `insecureHTTPParser` option is supported now.
  - version: v13.3.0
    pr-url: https://github.com/nodejs/node/pull/30570
    description: The `maxHeaderSize` option is supported now.
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/21616
    description: The `url` parameter can now be passed along with a separate
                 `options` object.
  - version: v7.5.0
    pr-url: https://github.com/nodejs/node/pull/10638
    description: The `options` parameter can be a WHATWG `URL` object.
-->

*   `url`{字符串|网址}
*   `options`{对象}
    *   `agent`{http.代理|布尔值} 控件[`Agent`][Agent]行为。可能
        值：
        *   `undefined`（默认值）：使用[`http.globalAgent`][http.globalAgent]为此主机和端口。
        *   `Agent`对象：显式使用传入`Agent`.
        *   `false`：导致新的`Agent`具有要使用的默认值。
    *   `auth`{字符串}基本身份验证 （`'user:password'`） 来计算
        授权标头。
    *   `createConnection`{函数}生成套接字/流的函数
        用于请求时`agent`未使用选项。这可用于
        避免创建自定义`Agent`类只是为了覆盖默认值
        `createConnection`功能。看[`agent.createConnection()`][agent.createConnection()]了解更多
        详。任何[`Duplex`][Duplex]流是有效的返回值。
    *   `defaultPort`{数字}协议的默认端口。**违约：**
        `agent.defaultPort`如果`Agent`已使用，否则`undefined`.
    *   `family`{数字}解析时要使用的 IP 地址系列`host`或
        `hostname`.有效值为`4`或`6`.如果未指定，则 IP v4 和
        将使用 v6。
    *   `headers`{对象}包含请求标头的对象。
    *   `hints`{数字}自选[`dns.lookup()`提示][dns.lookup() hints].
    *   `host`{字符串}要颁发的服务器的域名或 IP 地址
        请求。**违约：** `'localhost'`.
    *   `hostname`{字符串}的别名`host`.支持[`url.parse()`][url.parse()],
        `hostname`如果两者都使用，则将使用`host`和`hostname`已指定。
    *   `insecureHTTPParser`{布尔值}使用不安全的 HTTP 解析器，该解析器接受
        在以下情况下存在无效的 HTTP 标头`true`.使用不安全的解析器应该是
        避免。看[`--insecure-http-parser`][--insecure-http-parser]了解更多信息。
        **违约：** `false`
    *   `localAddress`{字符串}用于绑定网络连接的本地接口。
    *   `localPort`{数字}要从中进行连接的本地端口。
    *   `lookup`{函数}自定义查找功能。**违约：** [`dns.lookup()`][dns.lookup()].
    *   `maxHeaderSize`{数字}（可选）覆盖
        [`--max-http-header-size`][--max-http-header-size]（响应标头的最大长度
        字节），表示从服务器接收的响应。
        **违约：**16384 （16 KiB）.
    *   `method`{字符串}指定 HTTP 请求方法的字符串。**违约：**
        `'GET'`.
    *   `path`{字符串}请求路径。应包括查询字符串（如果有）。
        例如`'/index.html?page=12'`.当请求路径时引发异常
        包含非法字符。目前，只有空格被拒绝，但
        将来可能会改变。**违约：** `'/'`.
    *   `port`{数字}远程服务器的端口。**违约：** `defaultPort`如果设置，
        还`80`.
    *   `protocol`{字符串}要使用的协议。**违约：** `'http:'`.
    *   `setHost`{布尔值}：指定是否自动添加
        `Host`页眉。默认值为`true`.
    *   `signal`{中止信号}：可用于中止正在进行的中止的中止信号
        请求。
    *   `socketPath`{字符串}Unix 域套接字。如果`host`
        或`port`，因为它们指定 TCP 套接字。
    *   `timeout`{number}：一个数字，用于指定套接字超时（以毫秒为单位）。
        这将在连接套接字之前设置超时。
    *   `uniqueHeaders`{数组}应发送的请求标头的列表
        只有一次。如果标头的值是数组，则将联接项目
        用` ;  `.
*   `callback`{函数}
*   返回值：{http。客户端请求}

`options`在[`socket.connect()`][socket.connect()]也受支持。

Node.js为每个服务器维护多个连接以发出 HTTP 请求。
此功能允许透明地发出请求。

`url`可以是字符串或[`URL`][URL]对象。如果`url`是一个
字符串，它会自动解析为[`new URL()`][new URL()].如果是[`URL`][URL]
对象，它会自动转换为普通对象`options`对象。

如果两者兼而有之`url`和`options`指定，对象与
`options`属性优先。

可选`callback`参数将添加为 一次性侦听器
这[`'response'`]['response']事件。

`http.request()`返回[`http.ClientRequest`][http.ClientRequest]
类。这`ClientRequest`实例是可写流。如果需要
上传带有 POST 请求的文件，然后写入`ClientRequest`对象。

```js
const http = require('node:http');

const postData = JSON.stringify({
  'msg': 'Hello World!'
});

const options = {
  hostname: 'www.google.com',
  port: 80,
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Content-Length': Buffer.byteLength(postData)
  }
};

const req = http.request(options, (res) => {
  console.log(`STATUS: ${res.statusCode}`);
  console.log(`HEADERS: ${JSON.stringify(res.headers)}`);
  res.setEncoding('utf8');
  res.on('data', (chunk) => {
    console.log(`BODY: ${chunk}`);
  });
  res.on('end', () => {
    console.log('No more data in response.');
  });
});

req.on('error', (e) => {
  console.error(`problem with request: ${e.message}`);
});

// Write data to request body
req.write(postData);
req.end();
```

在示例中`req.end()`被调用。跟`http.request()`一
必须始终致电`req.end()`表示请求结束 -
即使没有数据写入请求正文。

如果在请求期间遇到任何错误（无论是使用 DNS 解析，
TCP 级别错误，或实际的 HTTP 解析错误）`'error'`发出事件
在返回的请求对象上。与所有人一样`'error'`事件（如果没有侦听器）
注册后，将抛出错误。

有一些特殊的标头应该注意。

*   发送“连接：保持活动状态”将通知 Node.js该连接
    服务器应保留到下一个请求。

*   发送“Content-Length”标头将禁用默认的分块编码。

*   发送“Expect”标头将立即发送请求标头。
    通常，在发送“预期：100-continue”时，既有超时又有侦听器
    对于`'continue'`应设置事件。有关详细信息，请参阅 RFC 2616 第 8.2.3 节
    信息。

*   发送授权标头将使用`auth`选择
    以计算基本身份验证。

使用的示例[`URL`][URL]如`options`:

```js
const options = new URL('http://abc:xyz@example.com');

const req = http.request(options, (res) => {
  // ...
});
```

在成功的请求中，将在以下事件中发出以下事件
次序：

*   `'socket'`
*   `'response'`
    *   `'data'`任意次数，在`res`对象
        (`'data'`如果响应正文为空，则根本不会发出，对于
        实例，在大多数重定向中）
    *   `'end'`在`res`对象
*   `'close'`

如果出现连接错误，将发出以下事件：

*   `'socket'`
*   `'error'`
*   `'close'`

如果在收到响应之前过早关闭连接，
将按以下顺序发出以下事件：

*   `'socket'`
*   `'error'`带有错误和消息`'Error: socket hang up'`和代码
    `'ECONNRESET'`
*   `'close'`

如果在收到响应后过早关闭连接，
将按以下顺序发出以下事件：

*   `'socket'`
*   `'response'`
    *   `'data'`任意次数，在`res`对象
*   （此处连接已关闭）
*   `'aborted'`在`res`对象
*   `'error'`在`res`带有错误且带有消息的对象
    `'Error: aborted'`和代码`'ECONNRESET'`.
*   `'close'`
*   `'close'`在`res`对象

如果`req.destroy()`在分配套接字之前调用，如下所示
事件将按以下顺序发出：

*   (`req.destroy()`调用此处）
*   `'error'`带有错误和消息`'Error: socket hang up'`和代码
    `'ECONNRESET'`
*   `'close'`

如果`req.destroy()`在连接成功之前调用，如下所示
事件将按以下顺序发出：

*   `'socket'`
*   (`req.destroy()`调用此处）
*   `'error'`带有错误和消息`'Error: socket hang up'`和代码
    `'ECONNRESET'`
*   `'close'`

如果`req.destroy()`在收到响应后调用，如下所示
事件将按以下顺序发出：

*   `'socket'`
*   `'response'`
    *   `'data'`任意次数，在`res`对象
*   (`req.destroy()`调用此处）
*   `'aborted'`在`res`对象
*   `'error'`在`res`带有错误且带有消息的对象
    `'Error: aborted'`和代码`'ECONNRESET'`.
*   `'close'`
*   `'close'`在`res`对象

如果`req.abort()`在分配套接字之前调用，如下所示
事件将按以下顺序发出：

*   (`req.abort()`调用此处）
*   `'abort'`
*   `'close'`

如果`req.abort()`在连接成功之前调用，如下所示
事件将按以下顺序发出：

*   `'socket'`
*   (`req.abort()`调用此处）
*   `'abort'`
*   `'error'`带有错误和消息`'Error: socket hang up'`和代码
    `'ECONNRESET'`
*   `'close'`

如果`req.abort()`在收到响应后调用，如下所示
事件将按以下顺序发出：

*   `'socket'`
*   `'response'`
    *   `'data'`任意次数，在`res`对象
*   (`req.abort()`调用此处）
*   `'abort'`
*   `'aborted'`在`res`对象
*   `'error'`在`res`带有错误且带有消息的对象
    `'Error: aborted'`和代码`'ECONNRESET'`.
*   `'close'`
*   `'close'`在`res`对象

设置`timeout`选项或使用`setTimeout()`函数将
不中止请求或执行除添加`'timeout'`事件。

通过`AbortSignal`，然后呼叫`abort`在相应的
`AbortController`行为方式与呼叫方式相同`.destroy()`在
请求本身。

## `http.validateHeaderName(name)`

<!-- YAML
added: v14.3.0
-->

*   `name`{字符串}

对提供的`name`在以下情况下完成
`res.setHeader(name, value)`被调用。

将非法值传递为`name`将导致[`TypeError`][TypeError]被抛出，
识别者`code: 'ERR_INVALID_HTTP_TOKEN'`.

在将标头传递给 HTTP 请求之前，不必使用此方法
或响应。HTTP 模块将自动验证此类标头。
例子：

例：

```js
const { validateHeaderName } = require('node:http');

try {
  validateHeaderName('');
} catch (err) {
  err instanceof TypeError; // --> true
  err.code; // --> 'ERR_INVALID_HTTP_TOKEN'
  err.message; // --> 'Header name must be a valid HTTP token [""]'
}
```

## `http.validateHeaderValue(name, value)`

<!-- YAML
added: v14.3.0
-->

*   `name`{字符串}
*   `value`{任何}

对提供的`value`在以下情况下完成
`res.setHeader(name, value)`被调用。

将非法值传递为`value`将导致[`TypeError`][TypeError]被抛出。

*   未定义的值错误由以下方式标识：`code: 'ERR_HTTP_INVALID_HEADER_VALUE'`.
*   无效值字符错误由`code: 'ERR_INVALID_CHAR'`.

在将标头传递给 HTTP 请求之前，不必使用此方法
或响应。HTTP 模块将自动验证此类标头。

例子：

```js
const { validateHeaderValue } = require('node:http');

try {
  validateHeaderValue('x-my-header', undefined);
} catch (err) {
  err instanceof TypeError; // --> true
  err.code === 'ERR_HTTP_INVALID_HEADER_VALUE'; // --> true
  err.message; // --> 'Invalid value "undefined" for header "x-my-header"'
}

try {
  validateHeaderValue('x-my-header', 'oʊmɪɡə');
} catch (err) {
  err instanceof TypeError; // --> true
  err.code === 'ERR_INVALID_CHAR'; // --> true
  err.message; // --> 'Invalid character in header content ["x-my-header"]'
}
```

## `http.setMaxIdleHTTPParsers`

<!-- YAML
added: REPLACEME
-->

*   {数字}

设置空闲 HTTP 解析器的最大数量。**违约：** `1000`.

[RFC 8187]: https://www.rfc-editor.org/rfc/rfc8187.txt

[`'checkContinue'`]: #event-checkcontinue

[`'finish'`]: #event-finish

[`'request'`]: #event-request

[`'response'`]: #event-response

[`'upgrade'`]: #event-upgrade

[`--insecure-http-parser`]: cli.md#--insecure-http-parser

[`--max-http-header-size`]: cli.md#--max-http-header-sizesize

[`Agent`]: #class-httpagent

[`Buffer.byteLength()`]: buffer.md#static-method-bufferbytelengthstring-encoding

[`Duplex`]: stream.md#class-streamduplex

[`HPE_HEADER_OVERFLOW`]: errors.md#hpe_header_overflow

[`TypeError`]: errors.md#class-typeerror

[`URL`]: url.md#the-whatwg-url-api

[`agent.createConnection()`]: #agentcreateconnectionoptions-callback

[`agent.getName()`]: #agentgetnameoptions

[`destroy()`]: #agentdestroy

[`dns.lookup()`]: dns.md#dnslookuphostname-options-callback

[`dns.lookup()` hints]: dns.md#supported-getaddrinfo-flags

[`getHeader(name)`]: #requestgetheadername

[`http.Agent`]: #class-httpagent

[`http.ClientRequest`]: #class-httpclientrequest

[`http.IncomingMessage`]: #class-httpincomingmessage

[`http.ServerResponse`]: #class-httpserverresponse

[`http.Server`]: #class-httpserver

[`http.get()`]: #httpgetoptions-callback

[`http.globalAgent`]: #httpglobalagent

[`http.request()`]: #httprequestoptions-callback

[`message.headers`]: #messageheaders

[`message.socket`]: #messagesocket

[`message.trailers`]: #messagetrailers

[`net.Server.close()`]: net.md#serverclosecallback

[`net.Server`]: net.md#class-netserver

[`net.Socket`]: net.md#class-netsocket

[`net.createConnection()`]: net.md#netcreateconnectionoptions-connectlistener

[`new URL()`]: url.md#new-urlinput-base

[`outgoingMessage.setHeader(name, value)`]: #outgoingmessagesetheadername-value

[`outgoingMessage.socket`]: #outgoingmessagesocket

[`removeHeader(name)`]: #requestremoveheadername

[`request.destroy()`]: #requestdestroyerror

[`request.destroyed`]: #requestdestroyed

[`request.end()`]: #requestenddata-encoding-callback

[`request.flushHeaders()`]: #requestflushheaders

[`request.getHeader()`]: #requestgetheadername

[`request.setHeader()`]: #requestsetheadername-value

[`request.setTimeout()`]: #requestsettimeouttimeout-callback

[`request.socket.getPeerCertificate()`]: tls.md#tlssocketgetpeercertificatedetailed

[`request.socket`]: #requestsocket

[`request.writableEnded`]: #requestwritableended

[`request.writableFinished`]: #requestwritablefinished

[`request.write(data, encoding)`]: #requestwritechunk-encoding-callback

[`response.end()`]: #responseenddata-encoding-callback

[`response.getHeader()`]: #responsegetheadername

[`response.setHeader()`]: #responsesetheadername-value

[`response.socket`]: #responsesocket

[`response.writableEnded`]: #responsewritableended

[`response.writableFinished`]: #responsewritablefinished

[`response.write()`]: #responsewritechunk-encoding-callback

[`response.write(data, encoding)`]: #responsewritechunk-encoding-callback

[`response.writeContinue()`]: #responsewritecontinue

[`response.writeHead()`]: #responsewriteheadstatuscode-statusmessage-headers

[`server.headersTimeout`]: #serverheaderstimeout

[`server.keepAliveTimeout`]: #serverkeepalivetimeout

[`server.listen()`]: net.md#serverlisten

[`server.requestTimeout`]: #serverrequesttimeout

[`server.timeout`]: #servertimeout

[`setHeader(name, value)`]: #requestsetheadername-value

[`socket.connect()`]: net.md#socketconnectoptions-connectlistener

[`socket.setKeepAlive()`]: net.md#socketsetkeepaliveenable-initialdelay

[`socket.setNoDelay()`]: net.md#socketsetnodelaynodelay

[`socket.setTimeout()`]: net.md#socketsettimeouttimeout-callback

[`socket.unref()`]: net.md#socketunref

[`url.parse()`]: url.md#urlparseurlstring-parsequerystring-slashesdenotehost

[`writable.cork()`]: stream.md#writablecork

[`writable.destroy()`]: stream.md#writabledestroyerror

[`writable.destroyed`]: stream.md#writabledestroyed

[`writable.uncork()`]: stream.md#writableuncork

[`writable.write()`]: stream.md#writablewritechunk-encoding-callback

[initial delay]: net.md#socketsetkeepaliveenable-initialdelay
