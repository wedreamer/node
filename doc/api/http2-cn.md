# HTTP/2

<!-- YAML
added: v8.4.0
changes:
  - version:
      - v15.3.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/36070
    description: It is possible to abort a request with an AbortSignal.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/34664
    description: Requests with the `host` header (with or without
                 `:authority`) can now be sent/received.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22466
    description: HTTP/2 is now Stable. Previously, it had been Experimental.
-->

<!--introduced_in=v8.4.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/http2.js -->

这`node:http2`模块提供了[HTTP/2][]协议。
可以使用以下命令访问它：

```js
const http2 = require('node:http2');
```

## 确定加密支持是否不可用

.js 可以在不包含对
`node:crypto`模块。在这种情况下，尝试`import`从`node:http2`或
叫`require('node:http2')`将导致引发错误。

使用 CommonJS 时，可以使用 try/catch 捕获引发的错误：

```cjs
let http2;
try {
  http2 = require('node:http2');
} catch (err) {
  console.log('http2 support is disabled!');
}
```

使用词法 ESM 时`import`关键字，错误只能是
如果处理程序`process.on('uncaughtException')`已注册
*以前*进行任何加载模块的尝试（例如，使用
预加载模块）。

使用 ESM 时，如果代码有可能在构建版本上运行
的节点.js未启用加密支持，请考虑使用
[`import()`][import()]函数而不是词法`import`关键词：

```mjs
let http2;
try {
  http2 = await import('node:http2');
} catch (err) {
  console.log('http2 support is disabled!');
}
```

## 核心接口

核心 API 提供了一个专门针对以下方面设计的低级接口：
支持 HTTP/2 协议功能。它特别*不*设计用于
与现有兼容性[HTTP/1][]模块接口。然而
这[兼容性接口][Compatibility API]是。

这`http2`核心 API 在客户端和服务器之间比
`http`应用程序接口。例如，大多数事件，如`'error'`,`'connect'`和
`'stream'`，可以由客户端代码或服务器端代码发出。

### 服务器端示例

下面说明了使用核心 API 的简单 HTTP/2 服务器。
由于没有已知的浏览器支持
[未加密的 HTTP/2][HTTP/2 Unencrypted]、用途
[`http2.createSecureServer()`][http2.createSecureServer()]在通信时是必需的
使用浏览器客户端。

```js
const http2 = require('node:http2');
const fs = require('node:fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('localhost-privkey.pem'),
  cert: fs.readFileSync('localhost-cert.pem')
});
server.on('error', (err) => console.error(err));

server.on('stream', (stream, headers) => {
  // stream is a Duplex
  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(8443);
```

若要为此示例生成证书和密钥，请运行：

```bash
openssl req -x509 -newkey rsa:2048 -nodes -sha256 -subj '/CN=localhost' \
  -keyout localhost-privkey.pem -out localhost-cert.pem
```

### 客户端示例

下面说明了 HTTP/2 客户端：

```js
const http2 = require('node:http2');
const fs = require('node:fs');
const client = http2.connect('https://localhost:8443', {
  ca: fs.readFileSync('localhost-cert.pem')
});
client.on('error', (err) => console.error(err));

const req = client.request({ ':path': '/' });

req.on('response', (headers, flags) => {
  for (const name in headers) {
    console.log(`${name}: ${headers[name]}`);
  }
});

req.setEncoding('utf8');
let data = '';
req.on('data', (chunk) => { data += chunk; });
req.on('end', () => {
  console.log(`\n${data}`);
  client.close();
});
req.end();
```

### 类：`Http2Session`

<!-- YAML
added: v8.4.0
-->

*   扩展：{事件发射器}

的实例`http2.Http2Session`类表示活动通信
HTTP/2 客户端和服务器之间的会话。此类的实例是*不*
旨在由用户代码直接构造。

每`Http2Session`实例将表现出略有不同的行为
取决于它是作为服务器还是客户端运行。这
`http2session.type`属性可用于确定
`Http2Session`正在运行。在服务器端，用户代码应该很少
有机会与`Http2Session`对象直接，最多
通常通过与`Http2Server`或
`Http2Stream`对象。

用户代码不会创建`Http2Session`直接实例。服务器端
`Http2Session`实例由`Http2Server`实例当
收到新的 HTTP/2 连接。客户端`Http2Session`实例是
使用`http2.connect()`方法。

#### `Http2Session`和套接字

每`Http2Session`实例仅与一个实例关联[`net.Socket`][net.Socket]或
[`tls.TLSSocket`][tls.TLSSocket]创建时。当`Socket`或
`Http2Session`都被摧毁了，两者都会被摧毁。

由于施加了特定的序列化和处理要求
通过HTTP / 2协议，不建议用户代码从中读取数据
或将数据写入`Socket`绑定到的实例`Http2Session`.这样做可以
将 HTTP/2 会话置于导致会话的不确定状态，并且
套接字变得不可用。

曾经`Socket`已绑定到`Http2Session`，用户代码应依赖
仅在`Http2Session`.

#### 事件：`'close'`

<!-- YAML
added: v8.4.0
-->

这`'close'`事件在`Http2Session`已销毁。其
侦听器不需要任何参数。

#### 事件：`'connect'`

<!-- YAML
added: v8.4.0
-->

*   `session`{Http2Session}
*   `socket`{网.插座}

这`'connect'`事件在`Http2Session`已成功
连接到远程对等体，通信可能会开始。

用户代码通常不会直接侦听此事件。

#### 事件：`'error'`

<!-- YAML
added: v8.4.0
-->

*   `error`{错误}

这`'error'`当在处理期间发生错误时发出事件
一`Http2Session`.

#### 事件：`'frameError'`

<!-- YAML
added: v8.4.0
-->

*   `type`{整数}框架类型。
*   `code`{整数}错误代码。
*   `id`{整数}流 ID（或`0`如果框架未与
    流）。

这`'frameError'`当尝试执行以下操作时发生错误时发出事件
在会话上发送帧。如果无法发送的帧已关联
具有特定的`Http2Stream`，尝试发出`'frameError'`事件
`Http2Stream`已制作。

如果`'frameError'`事件与流相关联，该流将是
关闭并立即销毁`'frameError'`事件。如果
事件未与流关联，`Http2Session`将被关闭
紧跟在`'frameError'`事件。

#### 事件：`'goaway'`

<!-- YAML
added: v8.4.0
-->

*   `errorCode`{数字}在`GOAWAY`框架。
*   `lastStreamID`{数字}远程对等体成功上一个流的 ID
    已处理（或`0`如果未指定 ID）。
*   `opaqueData`{缓冲区}如果`GOAWAY`
    框架，`Buffer`将传递包含该数据的实例。

这`'goaway'`事件在`GOAWAY`帧被接收。

这`Http2Session`实例将在`'goaway'`
发出事件。

#### 事件：`'localSettings'`

<!-- YAML
added: v8.4.0
-->

*   `settings`{HTTP/2 设置对象}的副本`SETTINGS`已接收帧。

这`'localSettings'`事件在确认时发出`SETTINGS`框架
已收到。

使用时`http2session.settings()`提交新设置，修改
设置在`'localSettings'`发出事件。

```js
session.settings({ enablePush: false });

session.on('localSettings', (settings) => {
  /* Use the new settings */
});
```

#### 事件：`'ping'`

<!-- YAML
added: v10.12.0
-->

*   `payload`{缓冲区}这`PING`帧 8 字节有效负载

这`'ping'`事件在`PING`帧从 接收
连接的对等体。

#### 事件：`'remoteSettings'`

<!-- YAML
added: v8.4.0
-->

*   `settings`{HTTP/2 设置对象}的副本`SETTINGS`已接收帧。

这`'remoteSettings'`事件在`SETTINGS`帧被接收
从连接的对等体。

```js
session.on('remoteSettings', (settings) => {
  /* Use the new settings */
});
```

#### 事件：`'stream'`

<!-- YAML
added: v8.4.0
-->

*   `stream`{Http2Stream}对流的引用
*   `headers`{HTTP/2 Headers Object}描述标头的对象
*   `flags`{数字}关联的数字标志
*   `rawHeaders`{数组}一个数组，其中包含原始标头名称，后跟
    它们各自的值。

这`'stream'`事件在`Http2Stream`已创建。

```js
const http2 = require('node:http2');
session.on('stream', (stream, headers, flags) => {
  const method = headers[':method'];
  const path = headers[':path'];
  // ...
  stream.respond({
    ':status': 200,
    'content-type': 'text/plain; charset=utf-8'
  });
  stream.write('hello ');
  stream.end('world');
});
```

在服务器端，用户代码通常不会直接侦听此事件，
并改为为`'stream'`发出的事件
`net.Server`或`tls.Server`返回的实例`http2.createServer()`和
`http2.createSecureServer()`，如下面的示例所示：

```js
const http2 = require('node:http2');

// Create an unencrypted HTTP/2 server
const server = http2.createServer();

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200
  });
  stream.on('error', (error) => console.error(error));
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

即使 HTTP/2 流和网络套接字不是 1：1 对应关系，
网络错误将破坏每个单独的流，并且必须在
流级别，如上所示。

#### 事件：`'timeout'`

<!-- YAML
added: v8.4.0
-->

在`http2session.setTimeout()`方法用于设置超时期限
为此`Http2Session`这`'timeout'`如果没有，则发出事件
活动`Http2Session`在配置的毫秒数之后。
它的侦听器不需要任何参数。

```js
session.setTimeout(2000);
session.on('timeout', () => { /* .. */ });
```

#### `http2session.alpnProtocol`

<!-- YAML
added: v9.4.0
-->

*   {字符串|未定义}

值将为`undefined`如果`Http2Session`尚未连接到
插座`h2c`如果`Http2Session`未连接到`TLSSocket`或
将返回已连接的值`TLSSocket`自己的`alpnProtocol`
财产。

#### `http2session.close([callback])`

<!-- YAML
added: v9.4.0
-->

*   `callback`{函数}

优雅地关闭`Http2Session`，允许任何现有流
自行完成并防止新的`Http2Stream`实例
创建。一旦关闭，`http2session.destroy()` *可能*如果有的话，被叫
没有打开`Http2Stream`实例。

如果指定，则`callback`函数被注册为
`'close'`事件。

#### `http2session.closed`

<!-- YAML
added: v9.4.0
-->

*   {布尔值}

将是`true`如果这是`Http2Session`实例已关闭，否则
`false`.

#### `http2session.connecting`

<!-- YAML
added: v10.0.0
-->

*   {布尔值}

将是`true`如果这是`Http2Session`实例仍在连接，将设置
自`false`发射前`connect`事件和/或调用`http2.connect`
回调。

#### `http2session.destroy([error][, code])`

<!-- YAML
added: v8.4.0
-->

*   `error`{错误}一`Error`对象，如果`Http2Session`正在被摧毁
    由于错误。
*   `code`{数字}要在最终版本中发送的 HTTP/2 错误代码`GOAWAY`框架。
    如果未指定，并且`error`不是未定义的，默认值为`INTERNAL_ERROR`,
    否则默认为`NO_ERROR`.

立即终止`Http2Session`和关联的`net.Socket`或
`tls.TLSSocket`.

一旦被摧毁，`Http2Session`将发出`'close'`事件。如果`error`
不是未定义的，一个`'error'`事件将在 紧接在
`'close'`事件。

如果有任何剩余的未结状态`Http2Streams`与
`Http2Session`，那些也将被摧毁。

#### `http2session.destroyed`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

将是`true`如果这是`Http2Session`实例已被销毁，不得
使用时间更长，否则`false`.

#### `http2session.encrypted`

<!-- YAML
added: v9.4.0
-->

*   {布尔值|未定义}

值为`undefined`如果`Http2Session`会话套接字尚未
连接`true`如果`Http2Session`与`TLSSocket`,
和`false`如果`Http2Session`连接到任何其他类型的插座
或流。

#### `http2session.goaway([code[, lastStreamID[, opaqueData]]])`

<!-- YAML
added: v9.4.0
-->

*   `code`{数字}HTTP/2 错误代码
*   `lastStreamID`{数字}上次处理的数字 ID`Http2Stream`
*   `opaqueData`{缓冲区|TypedArray|DataView} A`TypedArray`或`DataView`
    包含要在`GOAWAY`框架。

传输`GOAWAY`帧到连接的对等体*没有*关闭
`Http2Session`.

#### `http2session.localSettings`

<!-- YAML
added: v8.4.0
-->

*   {HTTP/2 设置对象}

描述此对象的当前本地设置的无原型对象
`Http2Session`.本地设置是本地的*这* `Http2Session`实例。

#### `http2session.originSet`

<!-- YAML
added: v9.4.0
-->

*   {string\[]|undefined}

如果`Http2Session`连接到`TLSSocket`这`originSet`财产
将返回一个`Array`的起源`Http2Session`可能
被认为是权威的。

这`originSet`属性仅在使用安全 TLS 连接时可用。

#### `http2session.pendingSettingsAck`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

指示`Http2Session`当前正在等待 确认
已发送`SETTINGS`框架。将是`true`在调用
`http2session.settings()`方法。将是`false`一旦全部发送`SETTINGS`
帧已得到确认。

#### `http2session.ping([payload, ]callback)`

<!-- YAML
added: v8.9.3
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `payload`{缓冲区|TypedArray|DataView} 可选的 ping 负载。
*   `callback`{函数}
*   返回：{布尔值}

发送`PING`帧到连接的 HTTP/2 对等体。一个`callback`函数必须
提供。该方法将返回`true`如果`PING`已发送，`false`
否则。

未完成（未确认）ping 的最大数量由
`maxOutstandingPings`配置选项。默认最大值为 10。

如果提供，则`payload`必须是`Buffer`,`TypedArray`或`DataView`
包含 8 个字节的数据，这些数据将与`PING`和
返回ping确认。

回调将使用三个参数调用：一个错误参数，该参数将
是`null`如果`PING`已成功确认，`duration`论点
报告自发送 ping 以来经过的毫秒数，并且
已收到确认，并且`Buffer`包含 8 字节`PING`
有效载荷。

```js
session.ping(Buffer.from('abcdefgh'), (err, duration, payload) => {
  if (!err) {
    console.log(`Ping acknowledged in ${duration} milliseconds`);
    console.log(`With payload '${payload.toString()}'`);
  }
});
```

如果`payload`参数未指定，默认负载将为
64 位时间戳（小字节序）标记`PING`期间。

#### `http2session.ref()`

<!-- YAML
added: v9.4.0
-->

调用[`ref()`][`net.Socket.prototype.ref()`]关于这个`Http2Session`
实例的底层[`net.Socket`][net.Socket].

#### `http2session.remoteSettings`

<!-- YAML
added: v8.4.0
-->

*   {HTTP/2 设置对象}

描述此对象的当前远程设置的无原型对象
`Http2Session`.远程设置由*连接*HTTP/2 对等体。

#### `http2session.setLocalWindowSize(windowSize)`

<!-- YAML
added:
  - v15.3.0
  - v14.18.0
-->

*   `windowSize`{数字}

设置本地终结点的窗口大小。
这`windowSize`是要设置的总窗口大小，而不是
三角洲。

```js
const http2 = require('node:http2');

const server = http2.createServer();
const expectedWindowSize = 2 ** 20;
server.on('connect', (session) => {

  // Set local window size to be 2 ** 20
  session.setLocalWindowSize(expectedWindowSize);
});
```

#### `http2session.setTimeout(msecs, callback)`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `msecs`{数字}
*   `callback`{函数}

用于设置在 上没有活动时调用的回调函数
这`Http2Session`后`msecs`毫秒。给定的`callback`是
已注册为`'timeout'`事件。

#### `http2session.socket`

<!-- YAML
added: v8.4.0
-->

*   {网.套接字|tls.TLSSocket}

返回`Proxy`充当对象的对象`net.Socket`（或`tls.TLSSocket`） 但是
将可用方法限制为可安全与 HTTP/2 一起使用的方法。

`destroy`,`emit`,`end`,`pause`,`read`,`resume`和`write`将抛出
带代码的错误`ERR_HTTP2_NO_SOCKET_MANIPULATION`.看
[`Http2Session`和套接字][Http2Session and Sockets]了解更多信息。

`setTimeout`方法将在此调用`Http2Session`.

所有其他交互将直接路由到套接字。

#### `http2session.state`

<!-- YAML
added: v8.4.0
-->

提供有关
`Http2Session`.

*   {对象}
    *   `effectiveLocalWindowSize`{数字}当前本地（接收）
        的流控制窗口大小`Http2Session`.
    *   `effectiveRecvDataLength`{数字}当前字节数
        自上次流控制以来已接收的`WINDOW_UPDATE`.
    *   `nextStreamID`{数字}要使用的数字标识符
        下次再来一次`Http2Stream`由此创建`Http2Session`.
    *   `localWindowSize`{数字}远程对等体可以读取的字节数
        发送而不接收`WINDOW_UPDATE`.
    *   `lastProcStreamID`{数字}的数字 ID`Http2Stream`
        对于其中`HEADERS`或`DATA`最近收到帧。
    *   `remoteWindowSize`{数字}此的字节数`Http2Session`
        可以在不接收的情况下发送`WINDOW_UPDATE`.
    *   `outboundQueueSize`{数字}当前在
        此项的出站队列`Http2Session`.
    *   `deflateDynamicTableSize`{数字}的当前大小（以字节为单位）
        出站标头压缩状态表。
    *   `inflateDynamicTableSize`{数字}的当前大小（以字节为单位）
        入站标头压缩状态表。

描述此当前状态的对象`Http2Session`.

#### `http2session.settings([settings][, callback])`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `settings`{HTTP/2 设置对象}
*   `callback`{函数}连接会话后调用的回调，或者
    如果会话已连接，则立即连接。
    *   `err`{错误|空}
    *   `settings`{HTTP/2 设置对象}更新的`settings`对象。
    *   `duration`{整数}

更新此的当前本地设置`Http2Session`并发送新的
`SETTINGS`帧到连接的 HTTP/2 对等体。

一旦被调用，`http2session.pendingSettingsAck`属性将是`true`
当会话正在等待远程对等方确认新的
设置。

新设置在`SETTINGS`确认
已收到，并且`'localSettings'`发出事件。可以发送
倍数`SETTINGS`帧，而确认仍处于挂起状态。

#### `http2session.type`

<!-- YAML
added: v8.4.0
-->

*   {数字}

这`http2session.type`将等于
`http2.constants.NGHTTP2_SESSION_SERVER`如果这是`Http2Session`实例是
服务器，以及`http2.constants.NGHTTP2_SESSION_CLIENT`如果实例是
客户。

#### `http2session.unref()`

<!-- YAML
added: v9.4.0
-->

调用[`unref()`][`net.Socket.prototype.unref()`]关于这个`Http2Session`
实例的底层[`net.Socket`][net.Socket].

### 类：`ServerHttp2Session`

<!-- YAML
added: v8.4.0
-->

*   Extends： {Http2Session}

#### `serverhttp2session.altsvc(alt, originOrStream)`

<!-- YAML
added: v9.4.0
-->

*   `alt`{字符串}替代服务配置的说明为
    定义者[RFC 7838][].
*   `originOrStream`{数字|字符串|网址|对象} 指定
    原点（或`Object`与`origin`属性）或数字
    活动活动的标识符`Http2Stream`如`http2stream.id`
    财产。

提交`ALTSVC`框架（定义为[RFC 7838][]） 添加到连接的客户端。

```js
const http2 = require('node:http2');

const server = http2.createServer();
server.on('session', (session) => {
  // Set altsvc for origin https://example.org:80
  session.altsvc('h2=":8000"', 'https://example.org:80');
});

server.on('stream', (stream) => {
  // Set altsvc for a specific stream
  stream.session.altsvc('h2=":8000"', stream.id);
});
```

发送`ALTSVC`具有特定流 ID 的帧指示备用
服务与给定的源相关联`Http2Stream`.

这`alt`和原点字符串*必须*仅包含 ASCII 字节，并且
严格解释为 ASCII 字节序列。特殊值`'clear'`
可以传递以清除给定的任何先前设置的替代服务
域。

当为`originOrStream`参数，它将被解析为
将派生一个 URL 和源。例如，原点
网址`'https://example.org/foo/bar'`是 ASCII 字符串
`'https://example.org'`.如果给定字符串中的任何一个，将引发错误
无法解析为 URL，或者无法派生有效源。

一个`URL`对象，或任何具有`origin`属性，可以传递为
`originOrStream`，在这种情况下，值`origin`属性将是
使用。的值`origin`财产*必须*是正确序列化的
ASCII 起源。

#### 指定替代服务

的格式`alt`参数由[RFC 7838][]作为
ASCII 字符串，包含以逗号分隔的“替代”协议列表
与特定主机和端口相关联。

例如，值`'h2="example.org:81"'`表示 HTTP/2
协议在主机上可用`'example.org'`在 TCP/IP 端口 81 上。这
主机和端口*必须*包含在报价中 （`"`） 字符。

可以指定多种替代方案，例如：`'h2="example.org:81",
h2=":82"'`.

协议标识符 （`'h2'`在示例中）可以是任何有效的
[ALPN 协议 ID][ALPN Protocol ID].

这些值的语法不由 Node 验证.js实现和
由用户提供或从对等方接收。

#### `serverhttp2session.origin(...origins)`

<!-- YAML
added: v10.12.0
-->

*   `origins`{ 字符串|网址|对象 } 作为传递的一个或多个 URL 字符串
    单独的参数。

提交`ORIGIN`框架（定义为[RFC 8336][]） 到连接的客户端
以公布服务器能够为其提供的源集
权威回应。

```js
const http2 = require('node:http2');
const options = getSecureOptionsSomehow();
const server = http2.createSecureServer(options);
server.on('stream', (stream) => {
  stream.respond();
  stream.end('ok');
});
server.on('session', (session) => {
  session.origin('https://example.com', 'https://example.org');
});
```

当字符串作为`origin`，它将被解析为 URL 和
将派生原点。例如，HTTP URL 的源
`'https://example.org/foo/bar'`是 ASCII 字符串
`'https://example.org'`.如果给定字符串中的任何一个，将引发错误
无法解析为 URL，或者无法派生有效源。

一个`URL`对象，或任何具有`origin`属性，可以传递为
一`origin`，在这种情况下，值`origin`属性将是
使用。的值`origin`财产*必须*是正确序列化的
ASCII 起源。

或者，`origins`选项可在创建新的 HTTP/2 时使用
服务器使用`http2.createSecureServer()`方法：

```js
const http2 = require('node:http2');
const options = getSecureOptionsSomehow();
options.origins = ['https://example.com', 'https://example.org'];
const server = http2.createSecureServer(options);
server.on('stream', (stream) => {
  stream.respond();
  stream.end('ok');
});
```

### 类：`ClientHttp2Session`

<!-- YAML
added: v8.4.0
-->

*   Extends： {Http2Session}

#### 事件：`'altsvc'`

<!-- YAML
added: v9.4.0
-->

*   `alt`{字符串}
*   `origin`{字符串}
*   `streamId`{数字}

这`'altsvc'`事件在`ALTSVC`帧由 接收
客户端。事件随`ALTSVC`值、源和流
ID。如果不是`origin`在`ALTSVC`框架`origin`将
为空字符串。

```js
const http2 = require('node:http2');
const client = http2.connect('https://example.org');

client.on('altsvc', (alt, origin, streamId) => {
  console.log(alt);
  console.log(origin);
  console.log(streamId);
});
```

#### 事件：`'origin'`

<!-- YAML
added: v10.12.0
-->

*   `origins`{字符串\[]}

这`'origin'`事件在`ORIGIN`帧由 接收
客户端。该事件由`origin`字符串。这
`http2session.originSet`将更新以包括收到的
起源。

```js
const http2 = require('node:http2');
const client = http2.connect('https://example.org');

client.on('origin', (origins) => {
  for (let n = 0; n < origins.length; n++)
    console.log(origins[n]);
});
```

这`'origin'`仅当使用安全的 TLS 连接时才会发出事件。

#### `clienthttp2session.request(headers[, options])`

<!-- YAML
added: v8.4.0
-->

*   `headers`{HTTP/2 Headers Object}

*   `options`{对象}
    *   `endStream`{布尔值}`true`如果`Http2Stream` *写*侧面应该
        最初关闭，例如在发送`GET`不应该的请求
        期望有效载荷主体。
    *   `exclusive`{布尔值}什么时候`true`和`parent`标识父流，
        创建的流成为父级的唯一直接依赖项，具有
        所有其他现有依赖项都成为新创建的流的依赖项。
        **违约：** `false`.
    *   `parent`{数字}指定新河流的数字标识符
        创建的流是依赖于的。
    *   `weight`{数字}指定流在关系中的相对依赖关系
        到具有相同内容的其他流`parent`.该值是介于`1`
        和`256`（包括）。
    *   `waitForTrailers`{布尔值}什么时候`true`这`Http2Stream`将发出
        `'wantTrailers'`决赛后的活动`DATA`帧已发送。
    *   `signal`{中止信号}中止可用于中止正在进行的信号
        请求。

*   返回： {ClientHttp2Stream}

对于 HTTP/2 客户端`Http2Session`仅限实例，`http2session.request()`
创建并返回`Http2Stream`可用于发送
对连接的服务器的 HTTP/2 请求。

当`ClientHttp2Session`首次创建，套接字可能尚未创建
连接。如果`clienthttp2session.request()`在此期间调用，
实际请求将被推迟，直到套接字准备就绪。
如果`session`在执行实际请求之前关闭，
`ERR_HTTP2_GOAWAY_SESSION`被抛出。

此方法仅在以下情况下可用：`http2session.type`等于
`http2.constants.NGHTTP2_SESSION_CLIENT`.

```js
const http2 = require('node:http2');
const clientSession = http2.connect('https://localhost:1234');
const {
  HTTP2_HEADER_PATH,
  HTTP2_HEADER_STATUS
} = http2.constants;

const req = clientSession.request({ [HTTP2_HEADER_PATH]: '/' });
req.on('response', (headers) => {
  console.log(headers[HTTP2_HEADER_STATUS]);
  req.on('data', (chunk) => { /* .. */ });
  req.on('end', () => { /* .. */ });
});
```

当`options.waitForTrailers`选项已设置，`'wantTrailers'`事件
在对要发送的最后一个有效负载数据块进行排队后立即发出。
这`http2stream.sendTrailers()`然后可以调用方法发送尾随
标头到对等方。

什么时候`options.waitForTrailers`已设置，`Http2Stream`不会自动
当决赛时关闭`DATA`帧被传输。用户代码必须调用
`http2stream.sendTrailers()`或`http2stream.close()`以关闭
`Http2Stream`.

什么时候`options.signal`设置为`AbortSignal`然后`abort`在
相应`AbortController`被调用，请求将发出一个`'error'`
具有`AbortError`错误。

这`:method`和`:path`其中未指定伪标头`headers`,
它们分别默认为：

*   `:method`=`'GET'`
*   `:path`=`/`

### 类：`Http2Stream`

<!-- YAML
added: v8.4.0
-->

*   扩展：{流。双工}

的每个实例`Http2Stream`类表示双向 HTTP/2
通信流通过`Http2Session`实例。任何单一`Http2Session`
最多可能有 2 个<sup>31</sup>-1`Http2Stream`在其生存期内的实例。

用户代码不会构造`Http2Stream`直接实例。相反，这些
通过`Http2Session`
实例。在服务器上，`Http2Stream`实例是在响应中创建的
到传入的 HTTP 请求（并通过`'stream'`
事件），或响应对`http2stream.pushStream()`方法。
在客户端上，`Http2Stream`实例在以下任一情况下创建并返回
`http2session.request()`方法被调用，或响应传入
`'push'`事件。

这`Http2Stream`类是[`ServerHttp2Stream`][ServerHttp2Stream]和
[`ClientHttp2Stream`][ClientHttp2Stream]类，每个类都由以下任一项专门使用
分别是服务器端或客户端。

都`Http2Stream`实例是[`Duplex`][Duplex]流。这`Writable`侧面的
`Duplex`用于将数据发送到连接的对等体，而`Readable`边
用于接收由连接的对等体发送的数据。

的默认文本字符编码`Http2Stream`是 UTF-8。使用
`Http2Stream`要发送文本，请使用`'content-type'`标头以设置字符
编码。

```js
stream.respond({
  'content-type': 'text/html; charset=utf-8',
  ':status': 200
});
```

#### `Http2Stream`生命周期

##### 创造

在服务器端，实例[`ServerHttp2Stream`][ServerHttp2Stream]创建
什么时候：

*   一个新的 HTTP/2`HEADERS`接收具有以前未使用的流ID的帧;
*   这`http2stream.pushStream()`调用方法。

在客户端，实例[`ClientHttp2Stream`][ClientHttp2Stream]在以下情况下创建
`http2session.request()`调用方法。

在客户端上，`Http2Stream`实例返回者`http2session.request()`
如果父母`Http2Session`尚未
已完全建立。在这种情况下，操作调用`Http2Stream`
将被缓冲，直到`'ready'`发出事件。用户代码应该很少，
如果有的话，需要处理`'ready'`直接事件。的就绪状态
`Http2Stream`可以通过检查的值来确定`http2stream.id`.如果
值为`undefined`，则流尚未准备好使用。

##### 破坏

都[`Http2Stream`][Http2Stream]在以下任一情况下，实例将被销毁：

*   一`RST_STREAM`流的帧由连接的对等体接收，
    并且（仅适用于客户端流）已读取挂起的数据。
*   这`http2stream.close()`方法被调用，并且（仅适用于客户端流）
    已读取挂起的数据。
*   这`http2stream.destroy()`或`http2session.destroy()`调用方法。

当`Http2Stream`实例被销毁，将尝试发送
`RST_STREAM`帧到连接的对等体。

当`Http2Stream`实例被销毁，`'close'`事件将
发出。因为`Http2Stream`是 的实例`stream.Duplex`这
`'end'`如果流数据当前正在流动，则还将发出事件。
这`'error'`在以下情况下，也可能发出事件`http2stream.destroy()`已调用
与`Error`作为第一个参数传递。

在`Http2Stream`已被摧毁，`http2stream.destroyed`
属性将是`true`和`http2stream.rstCode`属性将指定
`RST_STREAM`错误代码。这`Http2Stream`实例一次不再可用
摧毁。

#### 事件：`'aborted'`

<!-- YAML
added: v8.4.0
-->

这`'aborted'`事件在`Http2Stream`实例为
在通信中途异常中止。
它的侦听器不需要任何参数。

这`'aborted'`仅当`Http2Stream`可写面
尚未结束。

#### 事件：`'close'`

<!-- YAML
added: v8.4.0
-->

这`'close'`事件在`Http2Stream`被摧毁。一次
此事件被发出，`Http2Stream`实例不再可用。

关闭流时使用的 HTTP/2 错误代码可以使用以下命令检索
这`http2stream.rstCode`财产。如果代码是除以下值以外的任何值
`NGHTTP2_NO_ERROR`(`0`），一个`'error'`事件也将发出。

#### 事件：`'error'`

<!-- YAML
added: v8.4.0
-->

*   `error`{错误}

这`'error'`当在处理期间发生错误时发出事件
一`Http2Stream`.

#### 事件：`'frameError'`

<!-- YAML
added: v8.4.0
-->

*   `type`{整数}框架类型。
*   `code`{整数}错误代码。
*   `id`{整数}流 ID（或`0`如果框架未与
    流）。

这`'frameError'`当尝试执行以下操作时发生错误时发出事件
发送帧。调用时，处理程序函数将接收一个整数
参数标识帧类型，以及标识
错误代码。这`Http2Stream`实例将在
`'frameError'`发出事件。

#### 事件：`'ready'`

<!-- YAML
added: v8.4.0
-->

这`'ready'`事件在`Http2Stream`已打开，已
已分配一个`id`，并且可以使用。侦听器不期望任何
参数。

#### 事件：`'timeout'`

<!-- YAML
added: v8.4.0
-->

这`'timeout'`在未收到任何活动后发出事件
`Http2Stream`在设置的毫秒数内使用
`http2stream.setTimeout()`.
它的侦听器不需要任何参数。

#### 事件：`'trailers'`

<!-- YAML
added: v8.4.0
-->

*   `headers`{HTTP/2 Headers Object}描述标头的对象
*   `flags`{数字}关联的数字标志

这`'trailers'`当与 关联的标头块时发出事件
接收尾随标头字段。侦听器回调传递
[HTTP/2 标头对象][HTTP/2 Headers Object]和与标头关联的标志。

在以下情况下，可能不会发出此事件`http2stream.end()`称为
在收到预告片并且未读取传入数据之前，或者
听了。

```js
stream.on('trailers', (headers, flags) => {
  console.log(headers);
});
```

#### 事件：`'wantTrailers'`

<!-- YAML
added: v10.0.0
-->

这`'wantTrailers'`事件在`Http2Stream`已将
最后`DATA`要在帧上发送的帧和`Http2Stream`已准备好发送
尾随标头。发起请求或响应时，`waitForTrailers`
必须设置选项才能发出此事件。

#### `http2stream.aborted`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

设置为`true`如果`Http2Stream`实例异常中止。设置后，
这`'aborted'`事件将已发出。

#### `http2stream.bufferSize`

<!-- YAML
added:
 - v11.2.0
 - v10.16.0
-->

*   {数字}

此属性显示当前缓冲要写入的字符数。
看[`net.Socket.bufferSize`][net.Socket.bufferSize]了解详情。

#### `http2stream.close(code[, callback])`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `code`{数字}标识错误代码的无符号 32 位整数。
    **违约：** `http2.constants.NGHTTP2_NO_ERROR`(`0x00`).
*   `callback`{函数}注册的可选函数以侦听
    `'close'`事件。

关闭`Http2Stream`实例，方法是发送`RST_STREAM`帧到
连接的 HTTP/2 对等体。

#### `http2stream.closed`

<!-- YAML
added: v9.4.0
-->

*   {布尔值}

设置为`true`如果`Http2Stream`实例已关闭。

#### `http2stream.destroyed`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

设置为`true`如果`Http2Stream`实例已被销毁，不再存在
可用。

#### `http2stream.endAfterHeaders`

<!-- YAML
added: v10.11.0
-->

*   {布尔值}

设置为`true`如果`END_STREAM`在请求或响应中设置了标志
标头帧已接收，指示不应接收任何其他数据
和可读的一面`Http2Stream`将关闭。

#### `http2stream.id`

<!-- YAML
added: v8.4.0
-->

*   {数字|未定义}

此的数字流标识符`Http2Stream`实例。设置为`undefined`
如果尚未分配流标识符。

#### `http2stream.pending`

<!-- YAML
added: v9.4.0
-->

*   {布尔值}

设置为`true`如果`Http2Stream`尚未为实例分配
数字流标识符。

#### `http2stream.priority(options)`

<!-- YAML
added: v8.4.0
-->

*   `options`{对象}
    *   `exclusive`{布尔值}什么时候`true`和`parent`标识父流，
        此流成为父级的唯一直接依赖项，具有
        所有其他现有依赖项都成为此流的依赖项。**违约：**
        `false`.
    *   `parent`{数字}指定此流的数字标识符
        是依赖于的。
    *   `weight`{数字}指定流在关系中的相对依赖关系
        到具有相同内容的其他流`parent`.该值是介于`1`
        和`256`（包括）。
    *   `silent`{布尔值}什么时候`true`，在本地更改优先级，而不
        发送`PRIORITY`帧到连接的对等体。

更新了此项的优先级`Http2Stream`实例。

#### `http2stream.rstCode`

<!-- YAML
added: v8.4.0
-->

*   {数字}

设置为`RST_STREAM` [错误代码][error code]报告时`Http2Stream`是
在收到`RST_STREAM`来自连接的对等体的帧，
叫`http2stream.close()`或`http2stream.destroy()`.将是
`undefined`如果`Http2Stream`尚未关闭。

#### `http2stream.sentHeaders`

<!-- YAML
added: v9.5.0
-->

*   {HTTP/2 Headers Object}

包含为此发送的出站标头的对象`Http2Stream`.

#### `http2stream.sentInfoHeaders`

<!-- YAML
added: v9.5.0
-->

*   {HTTP/2 Headers Object\[]}

包含出站信息（附加）标头的对象数组
为此发送`Http2Stream`.

#### `http2stream.sentTrailers`

<!-- YAML
added: v9.5.0
-->

*   {HTTP/2 Headers Object}

包含为此发送的出站尾部的对象`HttpStream`.

#### `http2stream.session`

<!-- YAML
added: v8.4.0
-->

*   {Http2Session}

对`Http2Session`拥有此项的实例`Http2Stream`.这
值将为`undefined`在`Http2Stream`实例被销毁。

#### `http2stream.setTimeout(msecs, callback)`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `msecs`{数字}
*   `callback`{函数}

```js
const http2 = require('node:http2');
const client = http2.connect('http://example.org:8000');
const { NGHTTP2_CANCEL } = http2.constants;
const req = client.request({ ':path': '/' });

// Cancel the stream if there's no activity after 5 seconds
req.setTimeout(5000, () => req.close(NGHTTP2_CANCEL));
```

#### `http2stream.state`

<!-- YAML
added: v8.4.0
-->

提供有关
`Http2Stream`.

*   {对象}
    *   `localWindowSize`{数字}连接的对等体可以发送的字节数
        为此`Http2Stream`没有收到`WINDOW_UPDATE`.
    *   `state`{数字}指示 的低级当前状态的标志
        `Http2Stream`由`nghttp2`.
    *   `localClose`{数字}`1`如果这是`Http2Stream`已在当地关闭。
    *   `remoteClose`{数字}`1`如果这是`Http2Stream`已关闭
        远程。
    *   `sumDependencyWeight`{数字}所有的总权重`Http2Stream`
        依赖于此的实例`Http2Stream`指定使用
        `PRIORITY`框架。
    *   `weight`{数字}这一点的优先权重`Http2Stream`.

此的当前状态`Http2Stream`.

#### `http2stream.sendTrailers(headers)`

<!-- YAML
added: v10.0.0
-->

*   `headers`{HTTP/2 Headers Object}

发送尾随`HEADERS`帧到连接的 HTTP/2 对等体。此方法
将导致`Http2Stream`立即关闭，并且必须
在`'wantTrailers'`事件已发出。发送
请求或发送响应，`options.waitForTrailers`必须设置选项
为了保持`Http2Stream`决赛后开放`DATA`框架，以便
可以发送拖车。

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond(undefined, { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ xyz: 'abc' });
  });
  stream.end('Hello World');
});
```

HTTP/1 规范禁止预告片包含 HTTP/2 伪标头
字段（例如`':method'`,`':path'`等）。

### 类：`ClientHttp2Stream`

<!-- YAML
added: v8.4.0
-->

*   Extends {Http2Stream}

这`ClientHttp2Stream`类是 的扩展`Http2Stream`那是
仅在 HTTP/2 客户端上使用。`Http2Stream`客户端上的实例
提供诸如`'response'`和`'push'`仅与
客户端。

#### 事件：`'continue'`

<!-- YAML
added: v8.5.0
-->

当服务器发送`100 Continue`状态，通常是因为
包含的请求`Expect: 100-continue`.这是一个指令
客户端应发送请求正文。

#### 事件：`'headers'`

<!-- YAML
added: v8.4.0
-->

*   `headers`{HTTP/2 Headers Object}
*   `flags`{数字}

这`'headers'`当收到额外的标头块时发出事件
对于流，例如当块`1xx`收到信息标头。
侦听器回调传递[HTTP/2 标头对象][HTTP/2 Headers Object]和标志
与标头相关联。

```js
stream.on('headers', (headers, flags) => {
  console.log(headers);
});
```

#### 事件：`'push'`

<!-- YAML
added: v8.4.0
-->

*   `headers`{HTTP/2 Headers Object}
*   `flags`{数字}

这`'push'`当服务器推送流的响应标头时发出事件
已收到。侦听器回调传递[HTTP/2 标头对象][HTTP/2 Headers Object]和
与标头关联的标志。

```js
stream.on('push', (headers, flags) => {
  console.log(headers);
});
```

#### 事件：`'response'`

<!-- YAML
added: v8.4.0
-->

*   `headers`{HTTP/2 Headers Object}
*   `flags`{数字}

这`'response'`响应时发出事件`HEADERS`框架已
从连接的 HTTP/2 服务器接收此流。侦听器是
用两个参数调用：一个`Object`包含收到的
[HTTP/2 标头对象][HTTP/2 Headers Object]，以及与标头关联的标志。

```js
const http2 = require('node:http2');
const client = http2.connect('https://localhost');
const req = client.request({ ':path': '/' });
req.on('response', (headers, flags) => {
  console.log(headers[':status']);
});
```

### 类：`ServerHttp2Stream`

<!-- YAML
added: v8.4.0
-->

*   扩展： {Http2Stream}

这`ServerHttp2Stream`类是 的扩展[`Http2Stream`][Http2Stream]那是
仅在 HTTP/2 服务器上使用。`Http2Stream`服务器上的实例
提供其他方法，例如`http2stream.pushStream()`和
`http2stream.respond()`仅在服务器上相关。

#### `http2stream.additionalHeaders(headers)`

<!-- YAML
added: v8.4.0
-->

*   `headers`{HTTP/2 Headers Object}

发送其他信息`HEADERS`帧到连接的 HTTP/2 对等体。

#### `http2stream.headersSent`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

如果发送了标头，则为 true，否则为 false（只读）。

#### `http2stream.pushAllowed`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

映射到 的只读属性`SETTINGS_ENABLE_PUSH`遥控器的标志
客户的最新`SETTINGS`框架。将是`true`如果远程对等体
接受推送流，`false`否则。每个设置都是相同的
`Http2Stream`在同一`Http2Session`.

#### `http2stream.pushStream(headers[, options], callback)`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `headers`{HTTP/2 Headers Object}
*   `options`{对象}
    *   `exclusive`{布尔值}什么时候`true`和`parent`标识父流，
        创建的流成为父级的唯一直接依赖项，具有
        所有其他现有依赖项都成为新创建的流的依赖项。
        **违约：** `false`.
    *   `parent`{数字}指定新河流的数字标识符
        创建的流是依赖于的。
*   `callback`{函数}推送流被调用后的回调
    发起。
    *   `err`{错误}
    *   `pushStream`{ServerHttp2Stream}返回的`pushStream`对象。
    *   `headers`{HTTP/2 Headers Object}标头对象`pushStream`是
        启动于。

启动推送流。回调是使用新的`Http2Stream`
为作为第二个参数传递的推送流创建的实例，或
`Error`作为第一个参数传递。

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond({ ':status': 200 });
  stream.pushStream({ ':path': '/' }, (err, pushStream, headers) => {
    if (err) throw err;
    pushStream.respond({ ':status': 200 });
    pushStream.end('some pushed data');
  });
  stream.end('some data');
});
```

不允许在`HEADERS`框架。通过
一个`weight`值`http2stream.priority`与`silent`选项设置为
`true`以在并发流之间启用服务器端带宽平衡。

叫`http2stream.pushStream()`不允许从推送流中
并将引发错误。

#### `http2stream.respond([headers[, options]])`

<!-- YAML
added: v8.4.0
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33160
    description: Allow explicitly setting date headers.
-->

*   `headers`{HTTP/2 Headers Object}
*   `options`{对象}
    *   `endStream`{布尔值}设置为`true`以指示响应不会
        包括有效负载数据。
    *   `waitForTrailers`{布尔值}什么时候`true`这`Http2Stream`将发出
        `'wantTrailers'`决赛后的活动`DATA`帧已发送。

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond({ ':status': 200 });
  stream.end('some data');
});
```

启动响应。当`options.waitForTrailers`选项已设置，
`'wantTrailers'`事件将在排队最后一个区块后立即发出
要发送的有效负载数据。这`http2stream.sendTrailers()`然后方法可以是
用于将尾随标头字段发送到对等方。

什么时候`options.waitForTrailers`已设置，`Http2Stream`不会自动
当决赛时关闭`DATA`帧被传输。用户代码必须调用
`http2stream.sendTrailers()`或`http2stream.close()`以关闭
`Http2Stream`.

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond({ ':status': 200 }, { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ ABC: 'some value to send' });
  });
  stream.end('some data');
});
```

#### `http2stream.respondWithFD(fd[, headers[, options]])`

<!-- YAML
added: v8.4.0
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33160
    description: Allow explicitly setting date headers.
  - version: v12.12.0
    pr-url: https://github.com/nodejs/node/pull/29876
    description: The `fd` option may now be a `FileHandle`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18936
    description: Any readable file descriptor, not necessarily for a
                 regular file, is supported now.
-->

*   `fd`{数字|FileHandle} 可读文件描述符。
*   `headers`{HTTP/2 Headers Object}
*   `options`{对象}
    *   `statCheck`{函数}
    *   `waitForTrailers`{布尔值}什么时候`true`这`Http2Stream`将发出
        `'wantTrailers'`决赛后的活动`DATA`帧已发送。
    *   `offset`{数字}开始读取的偏移位置。
    *   `length`{数字}从 fd 发送的数据量。

启动一个响应，其数据是从给定的文件描述符中读取的。不
对给定的文件描述符执行验证。如果在以下情况下发生错误
尝试使用文件描述符读取数据，`Http2Stream`将是
使用`RST_STREAM`框架使用标准`INTERNAL_ERROR`法典。

使用时，`Http2Stream`对象的`Duplex`接口将关闭
自然而然。

```js
const http2 = require('node:http2');
const fs = require('node:fs');

const server = http2.createServer();
server.on('stream', (stream) => {
  const fd = fs.openSync('/some/file', 'r');

  const stat = fs.fstatSync(fd);
  const headers = {
    'content-length': stat.size,
    'last-modified': stat.mtime.toUTCString(),
    'content-type': 'text/plain; charset=utf-8'
  };
  stream.respondWithFD(fd, headers);
  stream.on('close', () => fs.closeSync(fd));
});
```

可选`options.statCheck`可以指定函数以给用户代码
有机会根据`fs.Stat`详
的给定 fd。如果`statCheck`函数提供，
`http2stream.respondWithFD()`方法将执行`fs.fstat()`调用
收集有关提供的文件描述符的详细信息。

这`offset`和`length`选项可用于将响应限制为
特定范围子集。例如，这可用于支持HTTP范围
请求。

文件描述符或`FileHandle`当流关闭时不关闭，
因此，一旦不再需要，就需要手动关闭它。
对多个流同时使用相同的文件描述符
不受支持，并可能导致数据丢失。重用文件描述符
支持流完成后。

当`options.waitForTrailers`选项已设置，`'wantTrailers'`事件
将在将最后一块有效负载数据排队后立即发出
送。这`http2stream.sendTrailers()`然后可以使用方法发送尾随
标头字段到对等方。

什么时候`options.waitForTrailers`已设置，`Http2Stream`不会自动
当决赛时关闭`DATA`帧被传输。用户代码*必须*调用
`http2stream.sendTrailers()`或`http2stream.close()`以关闭
`Http2Stream`.

```js
const http2 = require('node:http2');
const fs = require('node:fs');

const server = http2.createServer();
server.on('stream', (stream) => {
  const fd = fs.openSync('/some/file', 'r');

  const stat = fs.fstatSync(fd);
  const headers = {
    'content-length': stat.size,
    'last-modified': stat.mtime.toUTCString(),
    'content-type': 'text/plain; charset=utf-8'
  };
  stream.respondWithFD(fd, headers, { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ ABC: 'some value to send' });
  });

  stream.on('close', () => fs.closeSync(fd));
});
```

#### `http2stream.respondWithFile(path[, headers[, options]])`

<!-- YAML
added: v8.4.0
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33160
    description: Allow explicitly setting date headers.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18936
    description: Any readable file, not necessarily a
                 regular file, is supported now.
-->

*   `path`{字符串|缓冲区|网址}
*   `headers`{HTTP/2 Headers Object}
*   `options`{对象}
    *   `statCheck`{函数}
    *   `onError`{函数}在
        发送前出错。
    *   `waitForTrailers`{布尔值}什么时候`true`这`Http2Stream`将发出
        `'wantTrailers'`决赛后的活动`DATA`帧已发送。
    *   `offset`{数字}开始读取的偏移位置。
    *   `length`{数字}从 fd 发送的数据量。

发送常规文件作为响应。这`path`必须指定常规文件
或`'error'`事件将在`Http2Stream`对象。

使用时，`Http2Stream`对象的`Duplex`接口将关闭
自然而然。

可选`options.statCheck`可以指定函数以给用户代码
有机会根据`fs.Stat`详
给定文件的数目：

如果在尝试读取文件数据时发生错误，则`Http2Stream`
将使用`RST_STREAM`框架使用标准`INTERNAL_ERROR`
法典。如果`onError`定义了回调，然后调用它。否则
流将被销毁。

使用文件路径的示例：

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  function statCheck(stat, headers) {
    headers['last-modified'] = stat.mtime.toUTCString();
  }

  function onError(err) {
    // stream.respond() can throw if the stream has been destroyed by
    // the other side.
    try {
      if (err.code === 'ENOENT') {
        stream.respond({ ':status': 404 });
      } else {
        stream.respond({ ':status': 500 });
      }
    } catch (err) {
      // Perform actual error handling.
      console.log(err);
    }
    stream.end();
  }

  stream.respondWithFile('/some/file',
                         { 'content-type': 'text/plain; charset=utf-8' },
                         { statCheck, onError });
});
```

这`options.statCheck`函数也可用于取消发送操作
通过返回`false`.例如，条件请求可能会检查统计信息
结果以确定文件是否已被修改以返回适当的
`304`响应：

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  function statCheck(stat, headers) {
    // Check the stat here...
    stream.respond({ ':status': 304 });
    return false; // Cancel the send operation
  }
  stream.respondWithFile('/some/file',
                         { 'content-type': 'text/plain; charset=utf-8' },
                         { statCheck });
});
```

这`content-length`标题字段将自动设置。

这`offset`和`length`选项可用于将响应限制为
特定范围子集。例如，这可用于支持HTTP范围
请求。

这`options.onError`函数也可用于处理所有错误
这可能在启动文件传递之前发生。这
默认行为是销毁流。

当`options.waitForTrailers`选项已设置，`'wantTrailers'`事件
将在将最后一块有效负载数据排队后立即发出
送。这`http2stream.sendTrailers()`然后可以使用方法发送尾随
标头字段到对等方。

什么时候`options.waitForTrailers`已设置，`Http2Stream`不会自动
当决赛时关闭`DATA`帧被传输。用户代码必须调用
`http2stream.sendTrailers()`或`http2stream.close()`以关闭
`Http2Stream`.

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respondWithFile('/some/file',
                         { 'content-type': 'text/plain; charset=utf-8' },
                         { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ ABC: 'some value to send' });
  });
});
```

### 类：`Http2Server`

<!-- YAML
added: v8.4.0
-->

*   扩展：{net.服务器}

的实例`Http2Server`使用`http2.createServer()`
功能。这`Http2Server`类不由 直接导出
`node:http2`模块。

#### 事件：`'checkContinue'`

<!-- YAML
added: v8.5.0
-->

*   `request`{http2.Http2ServerRequest}
*   `response`{http2.Http2ServerResponse}

如果[`'request'`]['request']侦听器已注册或[`http2.createServer()`][http2.createServer()]是
提供了一个回调函数，`'checkContinue'`每次都发出事件
带有 HTTP 的请求`Expect: 100-continue`已接收。如果此事件是
未侦听，服务器将自动响应状态
`100 Continue`视情况而定。

处理此事件涉及调用[`response.writeContinue()`][response.writeContinue()]如果
客户端应继续发送请求正文，或生成适当的
HTTP 响应（例如 400 错误请求），如果客户端不应继续发送
请求正文。

当发出并处理此事件时，[`'request'`]['request']事件将
不发出。

#### 事件：`'connection'`

<!-- YAML
added: v8.4.0
-->

*   `socket`{流。双工}

建立新的 TCP 流时，将发出此事件。`socket`是
通常是类型的对象[`net.Socket`][net.Socket].通常用户不会想要
访问此事件。

用户还可以显式发出此事件以注入连接
进入 HTTP 服务器。在这种情况下，任何[`Duplex`][Duplex]可以传递流。

#### 事件：`'request'`

<!-- YAML
added: v8.4.0
-->

*   `request`{http2.Http2ServerRequest}
*   `response`{http2.Http2ServerResponse}

每次有请求时发出。可能有多个请求
每个会话。查看[兼容性接口][Compatibility API].

#### 事件：`'session'`

<!-- YAML
added: v8.4.0
-->

*   `session`{ServerHttp2Session}

这`'session'`事件在`Http2Session`由
`Http2Server`.

#### 事件：`'sessionError'`

<!-- YAML
added: v8.4.0
-->

*   `error`{错误}
*   `session`{ServerHttp2Session}

这`'sessionError'`事件在`'error'`事件由 发出
一`Http2Session`与 关联的对象`Http2Server`.

#### 事件：`'stream'`

<!-- YAML
added: v8.4.0
-->

*   `stream`{Http2Stream}对流的引用
*   `headers`{HTTP/2 Headers Object}描述标头的对象
*   `flags`{数字}关联的数字标志
*   `rawHeaders`{数组}一个数组，其中包含原始标头名称，后跟
    它们各自的值。

这`'stream'`事件在`'stream'`事件已发出
一`Http2Session`与服务器关联。

另请参见[`Http2Session`的`'stream'`事件][Http2Session's 'stream' event].

```js
const http2 = require('node:http2');
const {
  HTTP2_HEADER_METHOD,
  HTTP2_HEADER_PATH,
  HTTP2_HEADER_STATUS,
  HTTP2_HEADER_CONTENT_TYPE
} = http2.constants;

const server = http2.createServer();
server.on('stream', (stream, headers, flags) => {
  const method = headers[HTTP2_HEADER_METHOD];
  const path = headers[HTTP2_HEADER_PATH];
  // ...
  stream.respond({
    [HTTP2_HEADER_STATUS]: 200,
    [HTTP2_HEADER_CONTENT_TYPE]: 'text/plain; charset=utf-8'
  });
  stream.write('hello ');
  stream.end('world');
});
```

#### 事件：`'timeout'`

<!-- YAML
added: v8.4.0
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27558
    description: The default timeout changed from 120s to 0 (no timeout).
-->

这`'timeout'`当 服务器上没有活动时发出事件
使用设置的给定毫秒数`http2server.setTimeout()`.
**违约：**0（无超时）

#### `server.close([callback])`

<!-- YAML
added: v8.4.0
-->

*   `callback`{函数}

阻止服务器建立新会话。这不会阻止新的
由于 HTTP/2 的持久性而无法创建请求流
会话。要正常关闭服务器，请调用[`http2session.close()`][http2session.close()]上
所有活动会话。

如果`callback`，则在所有活动会话都完成之前不会调用它
已关闭，但服务器已停止允许新会话。看
[`net.Server.close()`][net.Server.close()]了解更多详情。

#### `server.setTimeout([msecs][, callback])`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27558
    description: The default timeout changed from 120s to 0 (no timeout).
-->

*   `msecs`{数字}**违约：**0（无超时）
*   `callback`{函数}
*   返回： {http2Server}

用于设置 http2 服务器请求的超时值，
并设置一个在没有活动时调用的回调函数
在`Http2Server`后`msecs`毫秒。

给定的回调被注册为`'timeout'`事件。

如果`callback`不是函数，而是新`ERR_INVALID_ARG_TYPE`
将引发错误。

#### `server.timeout`

<!-- YAML
added: v8.4.0
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

#### `server.updateSettings([settings])`

<!-- YAML
added:
  - v15.1.0
  - v14.17.0
-->

*   `settings`{HTTP/2 设置对象}

用于使用提供的设置更新服务器。

抛出`ERR_HTTP2_INVALID_SETTING_VALUE`无效`settings`值。

抛出`ERR_INVALID_ARG_TYPE`无效`settings`论点。

### 类：`Http2SecureServer`

<!-- YAML
added: v8.4.0
-->

*   扩展：{tls。服务器}

的实例`Http2SecureServer`使用
`http2.createSecureServer()`功能。这`Http2SecureServer`类不是
直接导出`node:http2`模块。

#### 事件：`'checkContinue'`

<!-- YAML
added: v8.5.0
-->

*   `request`{http2.Http2ServerRequest}
*   `response`{http2.Http2ServerResponse}

如果[`'request'`]['request']侦听器已注册或[`http2.createSecureServer()`][http2.createSecureServer()]
提供了一个回调函数，`'checkContinue'`每个发出的事件
使用 HTTP 为请求计时`Expect: 100-continue`已接收。如果此事件
未侦听，服务器将自动响应状态
`100 Continue`视情况而定。

处理此事件涉及调用[`response.writeContinue()`][response.writeContinue()]如果
客户端应继续发送请求正文，或生成适当的
HTTP 响应（例如 400 错误请求），如果客户端不应继续发送
请求正文。

当发出并处理此事件时，[`'request'`]['request']事件将
不发出。

#### 事件：`'connection'`

<!-- YAML
added: v8.4.0
-->

*   `socket`{流。双工}

当在 TLS 之前建立新的 TCP 流时，将发出此事件
开始握手。`socket`通常是类型的对象[`net.Socket`][net.Socket].
通常，用户不希望访问此事件。

用户还可以显式发出此事件以注入连接
进入 HTTP 服务器。在这种情况下，任何[`Duplex`][Duplex]可以传递流。

#### 事件：`'request'`

<!-- YAML
added: v8.4.0
-->

*   `request`{http2.Http2ServerRequest}
*   `response`{http2.Http2ServerResponse}

每次有请求时发出。可能有多个请求
每个会话。查看[兼容性接口][Compatibility API].

#### 事件：`'session'`

<!-- YAML
added: v8.4.0
-->

*   `session`{ServerHttp2Session}

这`'session'`事件在`Http2Session`由
`Http2SecureServer`.

#### 事件：`'sessionError'`

<!-- YAML
added: v8.4.0
-->

*   `error`{错误}
*   `session`{ServerHttp2Session}

这`'sessionError'`事件在`'error'`事件由 发出
一`Http2Session`与 关联的对象`Http2SecureServer`.

#### 事件：`'stream'`

<!-- YAML
added: v8.4.0
-->

*   `stream`{Http2Stream}对流的引用
*   `headers`{HTTP/2 Headers Object}描述标头的对象
*   `flags`{数字}关联的数字标志
*   `rawHeaders`{数组}一个数组，其中包含原始标头名称，后跟
    它们各自的值。

这`'stream'`事件在`'stream'`事件已发出
一`Http2Session`与服务器关联。

另请参见[`Http2Session`的`'stream'`事件][Http2Session's 'stream' event].

```js
const http2 = require('node:http2');
const {
  HTTP2_HEADER_METHOD,
  HTTP2_HEADER_PATH,
  HTTP2_HEADER_STATUS,
  HTTP2_HEADER_CONTENT_TYPE
} = http2.constants;

const options = getOptionsSomehow();

const server = http2.createSecureServer(options);
server.on('stream', (stream, headers, flags) => {
  const method = headers[HTTP2_HEADER_METHOD];
  const path = headers[HTTP2_HEADER_PATH];
  // ...
  stream.respond({
    [HTTP2_HEADER_STATUS]: 200,
    [HTTP2_HEADER_CONTENT_TYPE]: 'text/plain; charset=utf-8'
  });
  stream.write('hello ');
  stream.end('world');
});
```

#### 事件：`'timeout'`

<!-- YAML
added: v8.4.0
-->

这`'timeout'`当 服务器上没有活动时发出事件
使用设置的给定毫秒数`http2secureServer.setTimeout()`.
**违约：**2 分钟。

#### 事件：`'unknownProtocol'`

<!-- YAML
added: v8.4.0
-->

*   `socket`{流。双工}

这`'unknownProtocol'`当连接客户端无法
协商允许的协议（即 HTTP/2 或 HTTP/1.1）。事件处理程序
接收用于处理的套接字。如果没有为此事件注册侦听器，
连接已终止。可以使用
`'unknownProtocolTimeout'`选项传递给[`http2.createSecureServer()`][http2.createSecureServer()].
查看[兼容性接口][Compatibility API].

#### `server.close([callback])`

<!-- YAML
added: v8.4.0
-->

*   `callback`{函数}

阻止服务器建立新会话。这不会阻止新的
由于 HTTP/2 的持久性而无法创建请求流
会话。要正常关闭服务器，请调用[`http2session.close()`][http2session.close()]上
所有活动会话。

如果`callback`，则在所有活动会话都完成之前不会调用它
已关闭，但服务器已停止允许新会话。看
[`tls.Server.close()`][tls.Server.close()]了解更多详情。

#### `server.setTimeout([msecs][, callback])`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `msecs`{数字}**违约：** `120000`（2分钟）
*   `callback`{函数}
*   返回： {Http2SecureServer}

用于设置 http2 安全服务器请求的超时值，
并设置一个在没有活动时调用的回调函数
在`Http2SecureServer`后`msecs`毫秒。

给定的回调被注册为`'timeout'`事件。

如果`callback`不是函数，而是新`ERR_INVALID_ARG_TYPE`
将引发错误。

#### `server.timeout`

<!-- YAML
added: v8.4.0
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

#### `server.updateSettings([settings])`

<!-- YAML
added:
  - v15.1.0
  - v14.17.0
-->

*   `settings`{HTTP/2 设置对象}

用于使用提供的设置更新服务器。

抛出`ERR_HTTP2_INVALID_SETTING_VALUE`无效`settings`值。

抛出`ERR_INVALID_ARG_TYPE`无效`settings`论点。

### `http2.createServer([options][, onRequestHandler])`

<!-- YAML
added: v8.4.0
changes:
  - version:
      - v15.10.0
      - v14.16.0
      - v12.21.0
      - v10.24.0
    pr-url: https://github.com/nodejs-private/node-private/pull/246
    description: Added `unknownProtocolTimeout` option with a default of 10000.
  - version:
     - v14.4.0
     - v12.18.0
     - v10.21.0
    commit: 3948830ce6408be620b09a70bf66158623022af0
    pr-url: https://github.com/nodejs-private/node-private/pull/204
    description: Added `maxSettings` option with a default of 32.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30534
    description: Added `maxSessionRejectedStreams` option with a default of 100.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30534
    description: Added `maxSessionInvalidFrames` option with a default of 1000.
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/29144
    description: The `PADDING_STRATEGY_CALLBACK` has been made equivalent to
                 providing `PADDING_STRATEGY_ALIGNED` and `selectPadding`
                 has been removed.
  - version: v12.4.0
    pr-url: https://github.com/nodejs/node/pull/27782
    description: The `options` parameter now supports `net.createServer()`
                 options.
  - version: v9.6.0
    pr-url: https://github.com/nodejs/node/pull/15752
    description: Added the `Http1IncomingMessage` and `Http1ServerResponse`
                 option.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
-->

*   `options`{对象}
    *   `maxDeflateDynamicTableSize`{数字}设置最大动态表大小
        用于使标头字段充气。**违约：** `4Kib`.
    *   `maxSettings`{数字}设置每个设置项的最大数目
        `SETTINGS`框架。允许的最小值为`1`.**违约：** `32`.
    *   `maxSessionMemory`{数字}设置`Http2Session`
        允许使用。该值以兆字节数表示，
        例如：`1`等于 1 兆字节。允许的最小值为`1`.
        这是一个基于信用的限额，现有`Http2Stream`s 可能会导致此问题
        要超出的限制，但新`Http2Stream`实例将被拒绝
        超出此限制时。当前数量`Http2Stream`会话
        当前内存使用的标题压缩表，当前数据
        排队等待发送，并且未确认`PING`和`SETTINGS`帧全部
        计入当前限制。**违约：** `10`.
    *   `maxHeaderListPairs`{数字}设置标头条目的最大数目。
        这类似于[`server.maxHeadersCount`][server.maxHeadersCount]或
        [`request.maxHeadersCount`][request.maxHeadersCount]在`node:http`模块。最小值
        是`4`.**违约：** `128`.
    *   `maxOutstandingPings`{数字}设置最大未完成项数，
        未确认的 ping。**违约：** `10`.
    *   `maxSendHeaderBlockLength`{数字}设置
        序列化的压缩标头块。尝试发送以下标头：
        超过此限制将导致`'frameError'`正在发出的事件
        以及被关闭和摧毁的溪流。
        虽然这会将最大允许大小设置为整个标头块，
        `nghttp2`（内部 http2 库）的限制为`65536`
        对于每个解压缩的键/值对。
    *   `paddingStrategy`{数字}用于确定
        要使用的填充`HEADERS`和`DATA`框架。**违约：**
        `http2.constants.PADDING_STRATEGY_NONE`.值可以是以下值之一：
        *   `http2.constants.PADDING_STRATEGY_NONE`：不添加填充物。
        *   `http2.constants.PADDING_STRATEGY_MAX`：填充的最大量，
            由内部实现确定，应用。
        *   `http2.constants.PADDING_STRATEGY_ALIGNED`：尝试应用足够多的应用
            填充以确保总帧长度，包括 9 字节
            标头，是 8 的倍数。对于每个帧，有一个允许的最大值
            由当前流控制状态确定的填充字节数
            和设置。如果此最大值小于计算出的所需金额，则
            确保对齐，使用最大值，并且不使用总帧长度
            必须以 8 个字节对齐。
    *   `peerMaxConcurrentStreams`{数字}设置最大并发数
        远程对等体的流，就好像`SETTINGS`已收到帧。将
        如果远程对等体设置了自己的值，则被覆盖
        `maxConcurrentStreams`.**违约：** `100`.
    *   `maxSessionInvalidFrames`{整数}设置最大无效数
        在会话关闭之前将允许的帧。
        **违约：** `1000`.
    *   `maxSessionRejectedStreams`{整数}设置最大拒绝数
        创建时，在会话关闭之前将允许的流。
        每个拒绝都与`NGHTTP2_ENHANCE_YOUR_CALM`
        错误，应告诉对等方不再打开任何流，继续
        因此，打开流被视为行为不端的对等体的标志。
        **违约：** `100`.
    *   `settings`{HTTP/2 设置对象}要发送到 的初始设置
        连接时的远程对等体。
    *   `Http1IncomingMessage`{http.传入消息} 指定
        `IncomingMessage`用于 HTTP/1 回退的类。有助于扩展
        原件`http.IncomingMessage`.**违约：** `http.IncomingMessage`.
    *   `Http1ServerResponse`{http.ServerResponse} 指定`ServerResponse`
        用于 HTTP/1 回退的类。有助于扩展原始文件
        `http.ServerResponse`.**违约：** `http.ServerResponse`.
    *   `Http2ServerRequest`{http2.Http2ServerRequest} 指定
        `Http2ServerRequest`要使用的类。
        有助于扩展原始文件`Http2ServerRequest`.
        **违约：** `Http2ServerRequest`.
    *   `Http2ServerResponse`{http2.Http2ServerResponse} 指定
        `Http2ServerResponse`要使用的类。
        有助于扩展原始文件`Http2ServerResponse`.
        **违约：** `Http2ServerResponse`.
    *   `unknownProtocolTimeout`{数字}指定超时（以毫秒为单位），
        服务器应该在[`'unknownProtocol'`]['unknownProtocol']发出。如果
        套接字尚未被销毁，此时服务器将销毁它。
        **违约：** `10000`.
    *   ...： 任何[`net.createServer()`][net.createServer()]可以提供选项。
*   `onRequestHandler`{函数}看[兼容性接口][Compatibility API]
*   返回： {http2Server}

返回`net.Server`创建和管理的实例`Http2Session`
实例。

由于没有已知的浏览器支持
[未加密的 HTTP/2][HTTP/2 Unencrypted]、用途
[`http2.createSecureServer()`][http2.createSecureServer()]在通信时是必需的
使用浏览器客户端。

```js
const http2 = require('node:http2');

// Create an unencrypted HTTP/2 server.
// Since there are no browsers known that support
// unencrypted HTTP/2, the use of `http2.createSecureServer()`
// is necessary when communicating with browser clients.
const server = http2.createServer();

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

### `http2.createSecureServer(options[, onRequestHandler])`

<!-- YAML
added: v8.4.0
changes:
  - version:
      - v15.10.0
      - v14.16.0
      - v12.21.0
      - v10.24.0
    pr-url: https://github.com/nodejs-private/node-private/pull/246
    description: Added `unknownProtocolTimeout` option with a default of 10000.
  - version:
     - v14.4.0
     - v12.18.0
     - v10.21.0
    commit: 3948830ce6408be620b09a70bf66158623022af0
    pr-url: https://github.com/nodejs-private/node-private/pull/204
    description: Added `maxSettings` option with a default of 32.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30534
    description: Added `maxSessionRejectedStreams` option with a default of 100.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30534
    description: Added `maxSessionInvalidFrames` option with a default of 1000.
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/29144
    description: The `PADDING_STRATEGY_CALLBACK` has been made equivalent to
                 providing `PADDING_STRATEGY_ALIGNED` and `selectPadding`
                 has been removed.
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/22956
    description: Added the `origins` option to automatically send an `ORIGIN`
                 frame on `Http2Session` startup.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
-->

*   `options`{对象}
    *   `allowHTTP1`{布尔值}不支持的传入客户端连接
        当设置为 HTTP/1.x 时，HTTP/2 将降级为 HTTP/1.x`true`.
        查看[`'unknownProtocol'`]['unknownProtocol']事件。看[ALPN 谈判][ALPN negotiation].
        **违约：** `false`.
    *   `maxDeflateDynamicTableSize`{数字}设置最大动态表大小
        用于使标头字段充气。**违约：** `4Kib`.
    *   `maxSettings`{数字}设置每个设置项的最大数目
        `SETTINGS`框架。允许的最小值为`1`.**违约：** `32`.
    *   `maxSessionMemory`{数字}设置`Http2Session`
        允许使用。该值以兆字节数表示，
        例如：`1`等于 1 兆字节。允许的最小值为`1`.这是一个
        基于信用的限额，现有`Http2Stream`s 可能会导致此问题
        要超出的限制，但新`Http2Stream`实例将被拒绝
        超出此限制时。当前数量`Http2Stream`会话
        当前内存使用的标题压缩表，当前数据
        排队等待发送，并且未确认`PING`和`SETTINGS`帧全部
        计入当前限制。**违约：** `10`.
    *   `maxHeaderListPairs`{数字}设置标头条目的最大数目。
        这类似于[`server.maxHeadersCount`][server.maxHeadersCount]或
        [`request.maxHeadersCount`][request.maxHeadersCount]在`node:http`模块。最小值
        是`4`.**违约：** `128`.
    *   `maxOutstandingPings`{数字}设置最大未完成项数，
        未确认的 ping。**违约：** `10`.
    *   `maxSendHeaderBlockLength`{数字}设置
        序列化的压缩标头块。尝试发送以下标头：
        超过此限制将导致`'frameError'`正在发出的事件
        以及被关闭和摧毁的溪流。
    *   `paddingStrategy`{数字}用于确定
        要使用的填充`HEADERS`和`DATA`框架。**违约：**
        `http2.constants.PADDING_STRATEGY_NONE`.值可以是以下值之一：
        *   `http2.constants.PADDING_STRATEGY_NONE`：不添加填充物。
        *   `http2.constants.PADDING_STRATEGY_MAX`：填充的最大量，
            由内部实现确定，应用。
        *   `http2.constants.PADDING_STRATEGY_ALIGNED`：尝试应用足够多的应用
            填充以确保总帧长度，包括
            9 字节标头，是 8 的倍数。对于每个帧，有一个最大值
            由电流控制确定的允许填充字节数
            状态和设置。如果此最大值小于计算的金额
            需要确保对齐，使用最大值和总帧长度
            不一定在 8 个字节处对齐。
    *   `peerMaxConcurrentStreams`{数字}设置最大并发数
        远程对等体的流，就好像`SETTINGS`已收到帧。将
        如果远程对等体设置了自己的值，则被覆盖
        `maxConcurrentStreams`.**违约：** `100`.
    *   `maxSessionInvalidFrames`{整数}设置最大无效数
        在会话关闭之前将允许的帧。
        **违约：** `1000`.
    *   `maxSessionRejectedStreams`{整数}设置最大拒绝数
        创建时，在会话关闭之前将允许的流。
        每个拒绝都与`NGHTTP2_ENHANCE_YOUR_CALM`
        错误，应告诉对等方不再打开任何流，继续
        因此，打开流被视为行为不端的对等体的标志。
        **违约：** `100`.
    *   `settings`{HTTP/2 设置对象}要发送到 的初始设置
        连接时的远程对等体。
    *   ...： 任何[`tls.createServer()`][tls.createServer()]可以提供选项。为
        服务器，标识选项 （`pfx`或`key`/`cert`） 通常是必需的。
    *   `origins`{字符串\[]}要在`ORIGIN`
        在创建新服务器后立即添加框架`Http2Session`.
    *   `unknownProtocolTimeout`{数字}指定超时（以毫秒为单位），
        服务器应该在[`'unknownProtocol'`]['unknownProtocol']发出事件。如果
        套接字尚未被销毁，此时服务器将销毁它。
        **违约：** `10000`.
*   `onRequestHandler`{函数}看[兼容性接口][Compatibility API]
*   返回： {Http2SecureServer}

返回`tls.Server`创建和管理的实例`Http2Session`
实例。

```js
const http2 = require('node:http2');
const fs = require('node:fs');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem')
};

// Create a secure HTTP/2 server
const server = http2.createSecureServer(options);

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

### `http2.connect(authority[, options][, listener])`

<!-- YAML
added: v8.4.0
changes:
  - version:
      - v15.10.0
      - v14.16.0
      - v12.21.0
      - v10.24.0
    pr-url: https://github.com/nodejs-private/node-private/pull/246
    description: Added `unknownProtocolTimeout` option with a default of 10000.
  - version:
     - v14.4.0
     - v12.18.0
     - v10.21.0
    commit: 3948830ce6408be620b09a70bf66158623022af0
    pr-url: https://github.com/nodejs-private/node-private/pull/204
    description: Added `maxSettings` option with a default of 32.
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/29144
    description: The `PADDING_STRATEGY_CALLBACK` has been made equivalent to
                 providing `PADDING_STRATEGY_ALIGNED` and `selectPadding`
                 has been removed.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
-->

*   `authority`{字符串|URL} 要连接到的远程 HTTP/2 服务器。这必须
    采用最小有效 URL 的形式，并带有`http://`或`https://`
    前缀、主机名和 IP 端口（如果使用非默认端口）。用户信息
    （用户 ID 和密码）、路径、查询字符串和片段详细信息
    网址将被忽略。
*   `options`{对象}
    *   `maxDeflateDynamicTableSize`{数字}设置最大动态表大小
        用于使标头字段充气。**违约：** `4Kib`.
    *   `maxSettings`{数字}设置每个设置项的最大数目
        `SETTINGS`框架。允许的最小值为`1`.**违约：** `32`.
    *   `maxSessionMemory`{数字}设置`Http2Session`
        允许使用。该值以兆字节数表示，
        例如：`1`等于 1 兆字节。允许的最小值为`1`.
        这是一个基于信用的限额，现有`Http2Stream`s 可能会导致此问题
        要超出的限制，但新`Http2Stream`实例将被拒绝
        超出此限制时。当前数量`Http2Stream`会话
        当前内存使用的标题压缩表，当前数据
        排队等待发送，并且未确认`PING`和`SETTINGS`帧全部
        计入当前限制。**违约：** `10`.
    *   `maxHeaderListPairs`{数字}设置标头条目的最大数目。
        这类似于[`server.maxHeadersCount`][server.maxHeadersCount]或
        [`request.maxHeadersCount`][request.maxHeadersCount]在`node:http`模块。最小值
        是`1`.**违约：** `128`.
    *   `maxOutstandingPings`{数字}设置最大未完成项数，
        未确认的 ping。**违约：** `10`.
    *   `maxReservedRemoteStreams`{数字}设置保留推送的最大数量
        客户端将在任何给定时间接受的流。一旦当前数量
        当前预留的推送流超过此限制，新的推送流
        服务器发送的将被自动拒绝。允许的最小值
        为 0。允许的最大值为 2<sup>32</sup>-1.设置负值
        此选项为允许的最大值。**违约：** `200`.
    *   `maxSendHeaderBlockLength`{数字}设置
        序列化的压缩标头块。尝试发送以下标头：
        超过此限制将导致`'frameError'`正在发出的事件
        以及被关闭和摧毁的溪流。
    *   `paddingStrategy`{数字}用于确定
        要使用的填充`HEADERS`和`DATA`框架。**违约：**
        `http2.constants.PADDING_STRATEGY_NONE`.值可以是以下值之一：
        *   `http2.constants.PADDING_STRATEGY_NONE`：不添加填充物。
        *   `http2.constants.PADDING_STRATEGY_MAX`：填充的最大量，
            由内部实现确定，应用。
        *   `http2.constants.PADDING_STRATEGY_ALIGNED`：尝试应用足够多的应用
            填充以确保总帧长度，包括
            9 字节标头，是 8 的倍数。对于每个帧，有一个最大值
            由电流控制确定的允许填充字节数
            状态和设置。如果此最大值小于计算的金额
            需要确保对齐，使用最大值和总帧长度
            不一定在 8 个字节处对齐。
    *   `peerMaxConcurrentStreams`{数字}设置最大并发数
        远程对等体的流，就好像`SETTINGS`已收到帧。将
        如果远程对等体设置了自己的值，则被覆盖
        `maxConcurrentStreams`.**违约：** `100`.
    *   `protocol`{字符串}要连接的协议，如果未在
        `authority`.值可以是`'http:'`或`'https:'`.**违约：**
        `'https:'`
    *   `settings`{HTTP/2 设置对象}要发送到 的初始设置
        连接时的远程对等体。
    *   `createConnection`{函数}一个可选的回调，用于接收`URL`
        实例传递到`connect`和`options`对象，并返回
        [`Duplex`][Duplex]要用作此会话的连接的流。
    *   ...： 任何[`net.connect()`][net.connect()]或[`tls.connect()`][tls.connect()]可以提供选项。
    *   `unknownProtocolTimeout`{数字}指定超时（以毫秒为单位），
        服务器应该在[`'unknownProtocol'`]['unknownProtocol']发出事件。如果
        套接字尚未被销毁，此时服务器将销毁它。
        **违约：** `10000`.
*   `listener`{函数}将注册为
    [`'connect'`]['connect']事件。
*   返回： {ClientHttp2Session}

返回`ClientHttp2Session`实例。

```js
const http2 = require('node:http2');
const client = http2.connect('https://localhost:1234');

/* Use the client */

client.close();
```

### `http2.constants`

<!-- YAML
added: v8.4.0
-->

#### 的错误代码`RST_STREAM`和`GOAWAY`

|价值|名称|恒定|
|------ |------------------- |--------------------------------------------- |
|`0x00`|无错误|`http2.constants.NGHTTP2_NO_ERROR`|
|`0x01`|协议错误|`http2.constants.NGHTTP2_PROTOCOL_ERROR`|
|`0x02`|内部错误|`http2.constants.NGHTTP2_INTERNAL_ERROR`|
|`0x03`|流量控制误差|`http2.constants.NGHTTP2_FLOW_CONTROL_ERROR`|
|`0x04`|设置超时|`http2.constants.NGHTTP2_SETTINGS_TIMEOUT`|
|`0x05`|流已关闭|`http2.constants.NGHTTP2_STREAM_CLOSED`|
|`0x06`|帧大小误差|`http2.constants.NGHTTP2_FRAME_SIZE_ERROR`|
|`0x07`|拒绝流|`http2.constants.NGHTTP2_REFUSED_STREAM`|
|`0x08`|取消|`http2.constants.NGHTTP2_CANCEL`|
|`0x09`|压缩错误|`http2.constants.NGHTTP2_COMPRESSION_ERROR`|
|`0x0a`|连接错误|`http2.constants.NGHTTP2_CONNECT_ERROR`|
|`0x0b`|增强您的平静|`http2.constants.NGHTTP2_ENHANCE_YOUR_CALM`|
|`0x0c`|安全|不足`http2.constants.NGHTTP2_INADEQUATE_SECURITY`|
|`0x0d`|HTTP/1.1 必需|`http2.constants.NGHTTP2_HTTP_1_1_REQUIRED`|

这`'timeout'`当 服务器上没有活动时发出事件
使用设置的给定毫秒数`http2server.setTimeout()`.

### `http2.getDefaultSettings()`

<!-- YAML
added: v8.4.0
-->

*   返回：{HTTP/2 设置对象}

返回一个对象，其中包含`Http2Session`
实例。此方法在每次调用时返回一个新的对象实例
因此，返回的实例可以安全地修改以供使用。

### `http2.getPackedSettings([settings])`

<!-- YAML
added: v8.4.0
-->

*   `settings`{HTTP/2 设置对象}
*   返回：{缓冲区}

返回`Buffer`包含给定的序列化表示形式的实例
HTTP/2 设置，如[HTTP/2][]规范。这是有意为之
用于`HTTP2-Settings`标头字段。

```js
const http2 = require('node:http2');

const packed = http2.getPackedSettings({ enablePush: false });

console.log(packed.toString('base64'));
// Prints: AAIAAAAA
```

### `http2.getUnpackedSettings(buf)`

<!-- YAML
added: v8.4.0
-->

*   `buf`{缓冲区|TypedArray} 打包的设置。
*   返回：{HTTP/2 设置对象}

返回[HTTP/2 设置对象][HTTP/2 Settings Object]包含来自
给定的`Buffer`生成者`http2.getPackedSettings()`.

### `http2.sensitiveHeaders`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

*   {符号}

此符号可以设置为具有数组的 HTTP/2 标头对象上的属性
值，以便提供被视为敏感的标头的列表。
看[敏感标头][Sensitive headers]了解更多详情。

### 标头对象

标头在 JavaScript 对象上表示为 own 属性。房产
键将序列化为小写。属性值应为字符串（如果
他们不是他们会被强迫串）或`Array`字符串数（按顺序排列）
以为每个标头字段发送多个值）。

```js
const headers = {
  ':status': '200',
  'content-type': 'text-plain',
  'ABC': ['has', 'more', 'than', 'one', 'value']
};

stream.respond(headers);
```

传递给回调函数的标头对象将具有`null`原型。这
意味着普通的JavaScript对象方法，例如
`Object.prototype.toString()`和`Object.prototype.hasOwnProperty()`将
不工作。

对于传入标头：

*   这`:status`标头转换为`number`.
*   的副本`:status`,`:method`,`:authority`,`:scheme`,`:path`,
    `:protocol`,`age`,`authorization`,`access-control-allow-credentials`,
    `access-control-max-age`,`access-control-request-method`,`content-encoding`,
    `content-language`,`content-length`,`content-location`,`content-md5`,
    `content-range`,`content-type`,`date`,`dnt`,`etag`,`expires`,`from`,
    `host`,`if-match`,`if-modified-since`,`if-none-match`,`if-range`,
    `if-unmodified-since`,`last-modified`,`location`,`max-forwards`,
    `proxy-authorization`,`range`,`referer`,`retry-after`,`tk`,
    `upgrade-insecure-requests`,`user-agent`或`x-content-type-options`是
    丢弃。
*   `set-cookie`始终是一个数组。重复项将添加到数组中。
*   对于重复`cookie`标头中，值与 ';'.
*   对于所有其他标头，这些值与 '， ' 连接在一起。

```js
const http2 = require('node:http2');
const server = http2.createServer();
server.on('stream', (stream, headers) => {
  console.log(headers[':path']);
  console.log(headers.ABC);
});
```

#### 敏感标头

HTTP2 标头可以标记为敏感，这意味着 HTTP/2
标头压缩算法永远不会为它们编制索引。这对于以下情况有意义：
具有低熵的标头值，并且可能被认为对
攻击者，例如`Cookie`或`Authorization`.为此，请添加
标题名称`[http2.sensitiveHeaders]`属性作为数组：

```js
const headers = {
  ':status': '200',
  'content-type': 'text-plain',
  'cookie': 'some-cookie',
  'other-sensitive-header': 'very secret data',
  [http2.sensitiveHeaders]: ['cookie', 'other-sensitive-header']
};

stream.respond(headers);
```

对于某些标头，例如`Authorization`和短`Cookie`头
此标志是自动设置的。

此属性也为收到的标头设置。它将包含
所有标记为敏感的标头，包括自动以这种方式标记的标头。

### 设置对象

<!-- YAML
added: v8.4.0
changes:
  - version: v12.12.0
    pr-url: https://github.com/nodejs/node/pull/29833
    description: The `maxConcurrentStreams` setting is stricter.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: The `maxHeaderListSize` setting is now strictly enforced.
-->

这`http2.getDefaultSettings()`,`http2.getPackedSettings()`,
`http2.createServer()`,`http2.createSecureServer()`,
`http2session.settings()`,`http2session.localSettings`和
`http2session.remoteSettings`API 要么返回，要么作为输入接收
对象，用于定义 的配置设置`Http2Session`对象。
这些对象是包含以下内容的普通 JavaScript 对象
性能。

*   `headerTableSize`{数字}指定用于 的最大字节数
    标头压缩。允许的最小值为 0。允许的最大值
    为 2<sup>32</sup>-1.**违约：** `4096`.
*   `enablePush`{布尔值}指定`true`如果 HTTP/2 推送流是
    允许在`Http2Session`实例。**违约：** `true`.
*   `initialWindowSize`{数字}指定*发件人的*初始窗口大小
    用于流级流控制的字节。允许的最小值为 0。这
    最大允许值为 2<sup>32</sup>-1.**违约：** `65535`.
*   `maxFrameSize`{数字}指定最大帧的大小（以字节为单位）
    有效载荷。允许的最小值为 16，384。允许的最大值为
    2<sup>24</sup>-1.**违约：** `16384`.
*   `maxConcurrentStreams`{数字}指定最大并发数
    允许在`Http2Session`.没有默认值
    至少在理论上意味着 2<sup>32</sup>-1 流可能已打开
    在任何给定时间同时在`Http2Session`.最小值
    为 0。允许的最大值为 2<sup>32</sup>-1.**违约：**
    `4294967295`.
*   `maxHeaderListSize`{数字}指定最大大小（未压缩的八位字节）
    将被接受的标题列表。允许的最小值为 0。这
    最大允许值为 2<sup>32</sup>-1.**违约：** `65535`.
*   `maxHeaderSize`{数字}的别名`maxHeaderListSize`.
*   `enableConnectProtocol`{布尔值}指定`true`如果“扩展连接”
    协议“定义者[RFC 8441][]是要启用的。此设置仅
    如果由服务器发送，则有意义。一旦`enableConnectProtocol`设置
    已为给定`Http2Session`，则无法禁用它。
    **违约：** `false`.

设置对象上的所有其他属性都将被忽略。

### 错误处理

使用
`node:http2`模块：

当不正确的参数、选项或设置值
转会了。这些将始终由同步报告`throw`.

当在不正确的时间尝试操作时，会发生状态错误（对于
实例，尝试在流关闭后在流上发送数据）。这些将
使用同步报告`throw`或通过`'error'`事件
这`Http2Stream`,`Http2Session`或 HTTP/2 服务器对象，具体取决于位置
以及错误发生时。

当 HTTP/2 会话意外失败时，会发生内部错误。这些将是
通过以下方式报告`'error'`事件`Http2Session`或 HTTP/2 服务器对象。

当违反各种 HTTP/2 协议约束时，会发生协议错误。
这些将使用同步报告`throw`或通过`'error'`
事件`Http2Stream`,`Http2Session`或 HTTP/2 服务器对象，具体取决于
关于错误发生的地点和时间。

### 标头名称和值中的字符处理无效

HTTP/2 实现对以下位置中的无效字符进行更严格的处理：
HTTP 标头名称和值比 HTTP/1 实现。

标头字段名称为*不区分大小写*并通过线路传输
严格地作为小写字符串。Node.js提供的 API 允许标头
要设置为大小写混合字符串的名称（例如`Content-Type`），但将转换
那些小写（例如`content-type`） 时。

标头字段名称*必须*包含以下一个或多个 ASCII
字符：`a`-`z`,`A`-`Z`,`0`-`9`,`!`,`#`,`$`,`%`,`&`,`'`,`*`,`+`,
`-`,`.`,`^`,`_`,`` ` ``（反引号），`|`和`~`.

在 HTTP 标头字段名称中使用无效字符将导致
要关闭的流，并报告协议错误。

标头字段值的处理方式更加宽大，但*应该*不包含
换行符或回车符和*应该*仅限于 US-ASCII
字符，根据 HTTP 规范的要求。

### 在客户端上推送流

要在客户端上接收推送的流，请为`'stream'`
事件`ClientHttp2Session`:

```js
const http2 = require('node:http2');

const client = http2.connect('http://localhost');

client.on('stream', (pushedStream, requestHeaders) => {
  pushedStream.on('push', (responseHeaders) => {
    // Process response headers
  });
  pushedStream.on('data', (chunk) => { /* handle pushed data */ });
});

const req = client.request({ ':path': '/' });
```

### 支持`CONNECT`方法

这`CONNECT`方法用于允许将 HTTP/2 服务器用作代理
用于 TCP/IP 连接。

一个简单的 TCP 服务器：

```js
const net = require('node:net');

const server = net.createServer((socket) => {
  let name = '';
  socket.setEncoding('utf8');
  socket.on('data', (chunk) => name += chunk);
  socket.on('end', () => socket.end(`hello ${name}`));
});

server.listen(8000);
```

HTTP/2 连接代理：

```js
const http2 = require('node:http2');
const { NGHTTP2_REFUSED_STREAM } = http2.constants;
const net = require('node:net');

const proxy = http2.createServer();
proxy.on('stream', (stream, headers) => {
  if (headers[':method'] !== 'CONNECT') {
    // Only accept CONNECT requests
    stream.close(NGHTTP2_REFUSED_STREAM);
    return;
  }
  const auth = new URL(`tcp://${headers[':authority']}`);
  // It's a very good idea to verify that hostname and port are
  // things this proxy should be connecting to.
  const socket = net.connect(auth.port, auth.hostname, () => {
    stream.respond();
    socket.pipe(stream);
    stream.pipe(socket);
  });
  socket.on('error', (error) => {
    stream.close(http2.constants.NGHTTP2_CONNECT_ERROR);
  });
});

proxy.listen(8001);
```

HTTP/2 CONNECT 客户端：

```js
const http2 = require('node:http2');

const client = http2.connect('http://localhost:8001');

// Must not specify the ':path' and ':scheme' headers
// for CONNECT requests or an error will be thrown.
const req = client.request({
  ':method': 'CONNECT',
  ':authority': `localhost:${port}`
});

req.on('response', (headers) => {
  console.log(headers[http2.constants.HTTP2_HEADER_STATUS]);
});
let data = '';
req.setEncoding('utf8');
req.on('data', (chunk) => data += chunk);
req.on('end', () => {
  console.log(`The server says: ${data}`);
  client.close();
});
req.end('Jane');
```

### 扩展`CONNECT`协议

[RFC 8441][]将“扩展连接协议”扩展到 HTTP/2，该扩展
可用于引导使用`Http2Stream`使用`CONNECT`
方法作为其他通信协议（如 WebSockets）的隧道。

HTTP/2 服务器通过使用
这`enableConnectProtocol`设置：

```js
const http2 = require('node:http2');
const settings = { enableConnectProtocol: true };
const server = http2.createServer({ settings });
```

一旦客户端收到`SETTINGS`来自服务器的框架，指示
可以使用扩展的 CONNECT，也可以发送`CONNECT`使用
`':protocol'`HTTP/2 伪标头：

```js
const http2 = require('node:http2');
const client = http2.connect('http://localhost:8080');
client.on('remoteSettings', (settings) => {
  if (settings.enableConnectProtocol) {
    const req = client.request({ ':method': 'CONNECT', ':protocol': 'foo' });
    // ...
  }
});
```

## 兼容性接口

兼容性 API 的目标是提供类似的开发人员体验
使用 HTTP/2 时的 HTTP/1，使得开发应用程序成为可能
支持两者[HTTP/1][]和 HTTP/2。此 API 仅面向
**公共接口**的[HTTP/1][].但是，许多模块使用内部
方法或状态，以及那些*不支持*因为它是一个完全
不同的实现。

下面的示例使用兼容性创建 HTTP/2 服务器
应用程序接口：

```js
const http2 = require('node:http2');
const server = http2.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain; charset=utf-8' });
  res.end('ok');
});
```

为了创建混合[断续器][HTTPS]和 HTTP/2 服务器，请参阅
[ALPN 谈判][ALPN negotiation]部分。
不支持从非 tls HTTP/1 服务器升级。

HTTP/2 兼容性 API 由以下部分组成：[`Http2ServerRequest`][Http2ServerRequest]和
[`Http2ServerResponse`][Http2ServerResponse].它们的目标是API与HTTP / 1的兼容性，但是
它们不会隐藏协议之间的差异。例如，
HTTP 代码的状态消息将被忽略。

### ALPN 谈判

ALPN 协商允许同时支持两者[断续器][HTTPS]和 HTTP/2 over
相同的套接字。这`req`和`res`对象可以是 HTTP/1 或
HTTP/2 和应用程序**必须**将自己限制为公共 API
[HTTP/1][]，并检测是否可以使用更高级的
HTTP/2 的功能。

下面的示例创建一个同时支持这两种协议的服务器：

```js
const { createSecureServer } = require('node:http2');
const { readFileSync } = require('node:fs');

const cert = readFileSync('./cert.pem');
const key = readFileSync('./key.pem');

const server = createSecureServer(
  { cert, key, allowHTTP1: true },
  onRequest
).listen(4443);

function onRequest(req, res) {
  // Detects if it is a HTTPS request or HTTP/2
  const { socket: { alpnProtocol } } = req.httpVersion === '2.0' ?
    req.stream.session : req;
  res.writeHead(200, { 'content-type': 'application/json' });
  res.end(JSON.stringify({
    alpnProtocol,
    httpVersion: req.httpVersion
  }));
}
```

这`'request'`事件在两者上的工作方式相同[断续器][HTTPS]和
HTTP/2.

### 类：`http2.Http2ServerRequest`

<!-- YAML
added: v8.4.0
-->

*   扩展：{流。可读}

一个`Http2ServerRequest`对象由以下人员创建[`http2.Server`][http2.Server]或
[`http2.SecureServer`][http2.SecureServer]并作为第一个参数传递给
[`'request'`]['request']事件。它可用于访问请求状态、标头和
数据。

#### 事件：`'aborted'`

<!-- YAML
added: v8.4.0
-->

这`'aborted'`事件在`Http2ServerRequest`实例为
在通信中途异常中止。

这`'aborted'`仅当`Http2ServerRequest`写
一方尚未结束。

#### 事件：`'close'`

<!-- YAML
added: v8.4.0
-->

指示基础[`Http2Stream`][Http2Stream]已关闭。
就像`'end'`，则每个响应仅发生一次此事件。

#### `request.aborted`

<!-- YAML
added: v10.1.0
-->

*   {布尔值}

这`request.aborted`属性将是`true`如果请求有
已中止。

#### `request.authority`

<!-- YAML
added: v8.4.0
-->

*   {字符串}

请求机构伪标头字段。因为 HTTP/2 允许请求
以设置`:authority`或`host`，此值派生自
`req.headers[':authority']`如果存在。否则，它派生自
`req.headers['host']`.

#### `request.complete`

<!-- YAML
added: v12.10.0
-->

*   {布尔值}

这`request.complete`属性将是`true`如果请求有
已完成、中止或已销毁。

#### `request.connection`

<!-- YAML
added: v8.4.0
deprecated: v13.0.0
-->

> 稳定性：0 - 已弃用。用[`request.socket`][request.socket].

*   {网.套接字|tls.TLSSocket}

看[`request.socket`][request.socket].

#### `request.destroy([error])`

<!-- YAML
added: v8.4.0
-->

*   `error`{错误}

调用`destroy()`在[`Http2Stream`][Http2Stream]收到
这[`Http2ServerRequest`][Http2ServerRequest].如果`error`提供，一个`'error'`事件
发出和`error`作为参数传递给事件上的任何侦听器。

如果流已被销毁，则它不执行任何操作。

#### `request.headers`

<!-- YAML
added: v8.4.0
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
console.log(request.headers);
```

看[HTTP/2 标头对象][HTTP/2 Headers Object].

在 HTTP/2 中，请求路径、主机名、协议和方法表示为
以`:`字符（例如`':path'`).这些特别的
标头将包含在`request.headers`对象。必须注意不要
无意中修改了这些特殊标头，或者可能会发生错误。例如
从请求中删除所有标头将导致发生错误：

```js
removeAllHeaders(request.headers);
assert(request.url);   // Fails because the :path header has been removed
```

#### `request.httpVersion`

<!-- YAML
added: v8.4.0
-->

*   {字符串}

在服务器请求的情况下，由客户端发送HTTP版本。在以下情况下
客户端响应，连接到的服务器的 HTTP 版本。返回
`'2.0'`.

也`message.httpVersionMajor`是第一个整数，并且
`message.httpVersionMinor`是第二个。

#### `request.method`

<!-- YAML
added: v8.4.0
-->

*   {字符串}

字符串形式的请求方法。只读。例子：`'GET'`,`'DELETE'`.

#### `request.rawHeaders`

<!-- YAML
added: v8.4.0
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

#### `request.rawTrailers`

<!-- YAML
added: v8.4.0
-->

*   {字符串\[]}

原始请求/响应拖车键和值与它们完全相同
收到。仅填充于`'end'`事件。

#### `request.scheme`

<!-- YAML
added: v8.4.0
-->

*   {字符串}

指示方案的请求方案伪标头字段
目标网址的一部分。

#### `request.setTimeout(msecs, callback)`

<!-- YAML
added: v8.4.0
-->

*   `msecs`{数字}
*   `callback`{函数}
*   返回值：{http2。Http2ServerRequest}

设置[`Http2Stream`][Http2Stream]的超时值`msecs`.如果回调是
提供，然后将其添加为侦听器`'timeout'`事件
响应对象。

如果不是`'timeout'`侦听器添加到请求、响应或
服务器，然后[`Http2Stream`][Http2Stream]当超时时，它们将被销毁。如果
处理程序分配给请求、响应或服务器的`'timeout'`
事件，必须显式处理超时套接字。

#### `request.socket`

<!-- YAML
added: v8.4.0
-->

*   {网.套接字|tls.TLSSocket}

返回`Proxy`充当对象的对象`net.Socket`（或`tls.TLSSocket`） 但是
应用基于 HTTP/2 逻辑的 getter、setter 和方法。

`destroyed`,`readable`和`writable`属性将从 和 中检索
设置于`request.stream`.

`destroy`,`emit`,`end`,`on`和`once`方法将被调用
`request.stream`.

`setTimeout`方法将被调用`request.stream.session`.

`pause`,`read`,`resume`和`write`将抛出带有代码的错误
`ERR_HTTP2_NO_SOCKET_MANIPULATION`.看[`Http2Session`和套接字][Http2Session and Sockets]为
更多信息。

所有其他交互将直接路由到套接字。借助 TLS 支持，
用[`request.socket.getPeerCertificate()`][request.socket.getPeerCertificate()]获取客户的
身份验证详细信息。

#### `request.stream`

<!-- YAML
added: v8.4.0
-->

*   {Http2Stream}

这[`Http2Stream`][Http2Stream]对象支持请求。

#### `request.trailers`

<!-- YAML
added: v8.4.0
-->

*   {对象}

请求/响应尾部对象。仅填充于`'end'`事件。

#### `request.url`

<!-- YAML
added: v8.4.0
-->

*   {字符串}

请求 URL 字符串。这仅包含实际中存在的 URL
HTTP 请求。如果请求是：

```http
GET /status?name=ryan HTTP/1.1
Accept: text/plain
```

然后`request.url`将是：

<!-- eslint-disable semi -->

```js
'/status?name=ryan'
```

要将 url 解析为其各个部分，`new URL()`可以使用：

```console
$ node
> new URL('/status?name=ryan', 'http://example.com')
URL {
  href: 'http://example.com/status?name=ryan',
  origin: 'http://example.com',
  protocol: 'http:',
  username: '',
  password: '',
  host: 'example.com',
  hostname: 'example.com',
  port: '',
  pathname: '/status',
  search: '?name=ryan',
  searchParams: URLSearchParams { 'name' => 'ryan' },
  hash: ''
}
```

### 类：`http2.Http2ServerResponse`

<!-- YAML
added: v8.4.0
-->

*   扩展：{流}

此对象由 HTTP 服务器在内部创建，而不是由用户创建。是的
作为第二个参数传递给[`'request'`]['request']事件。

#### 事件：`'close'`

<!-- YAML
added: v8.4.0
-->

指示基础[`Http2Stream`][Http2Stream]在之前终止
[`response.end()`][response.end()]被调用或能够冲洗。

#### 事件：`'finish'`

<!-- YAML
added: v8.4.0
-->

发送响应时发出。更具体地说，此事件是
当响应标头和正文的最后一段为
移交给 HTTP/2 多路复用，以便通过网络传输。它
并不意味着客户已经收到任何东西。

在此事件之后，将不再对响应对象发出任何事件。

#### `response.addTrailers(headers)`

<!-- YAML
added: v8.4.0
-->

*   `headers`{对象}

此方法添加 HTTP 尾随标头（一个标头，但在
消息）到响应。

尝试设置包含无效字符的标头字段名称或值
将导致[`TypeError`][TypeError]被抛出。

#### `response.connection`

<!-- YAML
added: v8.4.0
deprecated: v13.0.0
-->

> 稳定性：0 - 已弃用。用[`response.socket`][response.socket].

*   {网.套接字|tls.TLSSocket}

看[`response.socket`][response.socket].

#### `response.createPushResponse(headers, callback)`

<!-- YAML
added: v8.4.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `headers`{HTTP/2 Headers Object}描述标头的对象
*   `callback`{函数}调用一次`http2stream.pushStream()`已完成，
    或尝试创建推送`Http2Stream`已失败或
    已被拒绝，或状态`Http2ServerRequest`在
    调用`http2stream.pushStream()`方法
    *   `err`{错误}
    *   `res`{http2.Http2ServerResponse} 新创建的`Http2ServerResponse`
        对象

叫[`http2stream.pushStream()`][http2stream.pushStream()]使用给定的标头，并包装
鉴于[`Http2Stream`][Http2Stream]在新创建的`Http2ServerResponse`作为回调
参数（如果成功）。什么时候`Http2ServerRequest`已关闭，回调为
调用时出现错误`ERR_HTTP2_INVALID_STREAM`.

#### `response.end([data[, encoding]][, callback])`

<!-- YAML
added: v8.4.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18780
    description: This method now returns a reference to `ServerResponse`.
-->

*   `data`{字符串|缓冲区|Uint8Array}
*   `encoding`{字符串}
*   `callback`{函数}
*   返回值：{this}

此方法向服务器发出所有响应标头和正文的信号
已发送;该服务器应将此消息视为已完成。
该方法，`response.end()`，则必须在每个响应上调用。

如果`data`已指定，它等效于调用
[`response.write(data, encoding)`][response.write(data, encoding)]其次`response.end(callback)`.

如果`callback`指定，它将在响应流时调用
已完成。

#### `response.finished`

<!-- YAML
added: v8.4.0
deprecated:
 - v13.4.0
 - v12.16.0
-->

> 稳定性：0 - 已弃用。用[`response.writableEnded`][response.writableEnded].

*   {布尔值}

指示响应是否已完成的布尔值。开始
如`false`.后[`response.end()`][response.end()]执行，该值将为`true`.

#### `response.getHeader(name)`

<!-- YAML
added: v8.4.0
-->

*   `name`{字符串}
*   返回：{字符串}

读出已排队但未发送到客户端的标头。
该名称不区分大小写。

```js
const contentType = response.getHeader('content-type');
```

#### `response.getHeaderNames()`

<!-- YAML
added: v8.4.0
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

#### `response.getHeaders()`

<!-- YAML
added: v8.4.0
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

#### `response.hasHeader(name)`

<!-- YAML
added: v8.4.0
-->

*   `name`{字符串}
*   返回：{布尔值}

返回`true`如果标头由`name`当前设置在
传出标头。标头名称匹配不区分大小写。

```js
const hasContentType = response.hasHeader('content-type');
```

#### `response.headersSent`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

如果发送了标头，则为 true，否则为 false（只读）。

#### `response.removeHeader(name)`

<!-- YAML
added: v8.4.0
-->

*   `name`{字符串}

删除已排队等待隐式发送的标头。

```js
response.removeHeader('Content-Encoding');
```

### `response.req`

<!-- YAML
added: v15.7.0
-->

*   {http2.Http2ServerRequest}

对原始 HTTP2 的引用`request`对象。

#### `response.sendDate`

<!-- YAML
added: v8.4.0
-->

*   {布尔值}

如果为 true，则将自动生成日期标头并发送
如果标头中尚不存在响应，则为响应。默认值为 true。

这应该只在测试时禁用;HTTP 需要日期标头
在回应中。

#### `response.setHeader(name, value)`

<!-- YAML
added: v8.4.0
-->

*   `name`{字符串}
*   `value`{字符串|字符串\[]}

为隐式标头设置单个标头值。如果此标头已存在
在要发送的标头中，其值将被替换。使用字符串数组
在这里发送多个具有相同名称的标头。

```js
response.setHeader('Content-Type', 'text/html; charset=utf-8');
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
const server = http2.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html; charset=utf-8');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain; charset=utf-8' });
  res.end('ok');
});
```

#### `response.setTimeout(msecs[, callback])`

<!-- YAML
added: v8.4.0
-->

*   `msecs`{数字}
*   `callback`{函数}
*   返回值：{http2。Http2ServerResponse}

设置[`Http2Stream`][Http2Stream]的超时值`msecs`.如果回调是
提供，然后将其添加为侦听器`'timeout'`事件
响应对象。

如果不是`'timeout'`侦听器添加到请求、响应或
服务器，然后[`Http2Stream`][Http2Stream]当超时时，它们将被销毁。如果
处理程序分配给请求、响应或服务器的`'timeout'`
事件，必须显式处理超时套接字。

#### `response.socket`

<!-- YAML
added: v8.4.0
-->

*   {网.套接字|tls.TLSSocket}

返回`Proxy`充当对象的对象`net.Socket`（或`tls.TLSSocket`） 但是
应用基于 HTTP/2 逻辑的 getter、setter 和方法。

`destroyed`,`readable`和`writable`属性将从 和 中检索
设置于`response.stream`.

`destroy`,`emit`,`end`,`on`和`once`方法将被调用
`response.stream`.

`setTimeout`方法将被调用`response.stream.session`.

`pause`,`read`,`resume`和`write`将抛出带有代码的错误
`ERR_HTTP2_NO_SOCKET_MANIPULATION`.看[`Http2Session`和套接字][Http2Session and Sockets]为
更多信息。

所有其他交互将直接路由到套接字。

```js
const http2 = require('node:http2');
const server = http2.createServer((req, res) => {
  const ip = req.socket.remoteAddress;
  const port = req.socket.remotePort;
  res.end(`Your IP address is ${ip} and your source port is ${port}.`);
}).listen(3000);
```

#### `response.statusCode`

<!-- YAML
added: v8.4.0
-->

*   {数字}

使用隐式标头时（不调用[`response.writeHead()`][response.writeHead()]明确），
此属性控制在以下情况下将发送到客户端的状态代码
标头被刷新。

```js
response.statusCode = 404;
```

将响应标头发送到客户端后，此属性指示
已发出的状态代码。

#### `response.statusMessage`

<!-- YAML
added: v8.4.0
-->

*   {字符串}

HTTP/2 不支持状态消息 （RFC 7540 8.1.2.4）。它返回
空字符串。

#### `response.stream`

<!-- YAML
added: v8.4.0
-->

*   {Http2Stream}

这[`Http2Stream`][Http2Stream]对象支持响应。

#### `response.writableEnded`

<!-- YAML
added: v12.9.0
-->

*   {布尔值}

是`true`后[`response.end()`][response.end()]已被调用。此属性
不指示数据是否已刷新，用于此用途
[`writable.writableFinished`][writable.writableFinished]相反。

#### `response.write(chunk[, encoding][, callback])`

<!-- YAML
added: v8.4.0
-->

*   `chunk`{字符串|缓冲区|Uint8Array}
*   `encoding`{字符串}
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
默认情况下，`encoding`是`'utf8'`.`callback`将调用此块时
的数据被刷新。

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

#### `response.writeContinue()`

<!-- YAML
added: v8.4.0
-->

发送状态`100 Continue`到客户端，指示请求正文
应发送。查看[`'checkContinue'`]['checkContinue']事件`Http2Server`和
`Http2SecureServer`.

#### `response.writeHead(statusCode[, statusMessage][, headers])`

<!-- YAML
added: v8.4.0
changes:
  - version:
     - v11.10.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/25974
    description: Return `this` from `writeHead()` to allow chaining with
                 `end()`.
-->

*   `statusCode`{数字}
*   `statusMessage`{字符串}
*   `headers`{对象|数组}
*   返回值：{http2。Http2ServerResponse}

将响应标头发送到请求。状态代码为 3 位 HTTP
状态代码，如`404`.最后一个论点，`headers`是响应标头。

返回对`Http2ServerResponse`，以便可以链接调用。

为了兼容[HTTP/1][]，人类可读`statusMessage`可能
作为第二个参数传递。但是，因为`statusMessage`没有
这意味着在HTTP / 2中，参数将不起作用并且进程警告
将发出。

```js
const body = 'hello world';
response.writeHead(200, {
  'Content-Length': Buffer.byteLength(body),
  'Content-Type': 'text/plain; charset=utf-8',
});
```

`Content-Length`以字节而不是字符为单位。这
`Buffer.byteLength()`API 可用于确定
给定编码。在出站消息上，Node.js 不检查内容长度
并且被传输的身体的长度是否相等。但是，当
接收消息，节点.js将自动拒绝消息时
`Content-Length`与实际负载大小不匹配。

此方法最多可以在消息上调用一次，然后再
[`response.end()`][response.end()]被调用。

如果[`response.write()`][response.write()]或[`response.end()`][response.end()]在呼叫之前被调用
这将计算隐式/可变标头并调用此函数。

当标头已设置为[`response.setHeader()`][response.setHeader()]，它们将被合并
将任何标头传递给[`response.writeHead()`][response.writeHead()]，并传递标头
自[`response.writeHead()`][response.writeHead()]给定优先级。

```js
// Returns content-type = text/plain
const server = http2.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html; charset=utf-8');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain; charset=utf-8' });
  res.end('ok');
});
```

尝试设置包含无效字符的标头字段名称或值
将导致[`TypeError`][TypeError]被抛出。

## 收集 HTTP/2 性能指标

这[性能观察者][Performance Observer]API 可用于收集基本性能
每个指标`Http2Session`和`Http2Stream`实例。

```js
const { PerformanceObserver } = require('node:perf_hooks');

const obs = new PerformanceObserver((items) => {
  const entry = items.getEntries()[0];
  console.log(entry.entryType);  // prints 'http2'
  if (entry.name === 'Http2Session') {
    // Entry contains statistics about the Http2Session
  } else if (entry.name === 'Http2Stream') {
    // Entry contains statistics about the Http2Stream
  }
});
obs.observe({ entryTypes: ['http2'] });
```

这`entryType`的属性`PerformanceEntry`将等于`'http2'`.

这`name`的属性`PerformanceEntry`将等于
`'Http2Stream'`或`'Http2Session'`.

如果`name`等于`Http2Stream`这`PerformanceEntry`将包含
以下附加属性：

*   `bytesRead`{数字}数量`DATA`为此接收的帧字节数
    `Http2Stream`.
*   `bytesWritten`{数字}数量`DATA`为此发送的帧字节
    `Http2Stream`.
*   `id`{数字}关联的标识符`Http2Stream`
*   `timeToFirstByte`{数字}中间经过的毫秒数
    `PerformanceEntry` `startTime`和接待第一`DATA`框架。
*   `timeToFirstByteSent`{数字}之间经过的毫秒数
    这`PerformanceEntry` `startTime`并发送第一个`DATA`框架。
*   `timeToFirstHeader`{数字}中间经过的毫秒数
    `PerformanceEntry` `startTime`以及第一个标头的接收。

如果`name`等于`Http2Session`这`PerformanceEntry`将包含
以下附加属性：

*   `bytesRead`{数字}为此接收的字节数`Http2Session`.
*   `bytesWritten`{数字}为此发送的字节数`Http2Session`.
*   `framesReceived`{数字}接收的 HTTP/2 帧数
    `Http2Session`.
*   `framesSent`{数字}发送的 HTTP/2 帧数`Http2Session`.
*   `maxConcurrentStreams`{数字}并发的最大流数
    在`Http2Session`.
*   `pingRTT`{数字}自传输以来经过的毫秒数
    的`PING`框架和接受其确认。仅当出现以下情况时才存在
    一个`PING`帧已发送到`Http2Session`.
*   `streamAverageDuration`{数字}的平均持续时间（以毫秒为单位）
    都`Http2Stream`实例。
*   `streamCount`{数字}数量`Http2Stream`处理条件的实例
    这`Http2Session`.
*   `type`{字符串}也`'server'`或`'client'`以识别
    `Http2Session`.

## 关于`:authority`和`host`

HTTP/2 要求请求具有`:authority`伪标头
或`host`页眉。喜欢`:authority`在构建 HTTP/2 时
直接请求，以及`host`当从HTTP / 1转换时（在代理中，
例如）。

兼容性 API 回退到`host`如果`:authority`莫
目前。看[`request.authority`][request.authority]了解更多信息。然而
如果不使用兼容性 API（或使用`req.headers`直接），
您需要自己实现任何回退行为。

[ALPN Protocol ID]: https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml#alpn-protocol-ids

[ALPN negotiation]: #alpn-negotiation

[Compatibility API]: #compatibility-api

[HTTP/1]: http.md

[HTTP/2]: https://tools.ietf.org/html/rfc7540

[HTTP/2 Headers Object]: #headers-object

[HTTP/2 Settings Object]: #settings-object

[HTTP/2 Unencrypted]: https://http2.github.io/faq/#does-http2-require-encryption

[HTTPS]: https.md

[Performance Observer]: perf_hooks.md

[RFC 7838]: https://tools.ietf.org/html/rfc7838

[RFC 8336]: https://tools.ietf.org/html/rfc8336

[RFC 8441]: https://tools.ietf.org/html/rfc8441

[Sensitive headers]: #sensitive-headers

[`'checkContinue'`]: #event-checkcontinue

[`'connect'`]: #event-connect

[`'request'`]: #event-request

[`'unknownProtocol'`]: #event-unknownprotocol

[`ClientHttp2Stream`]: #class-clienthttp2stream

[`Duplex`]: stream.md#class-streamduplex

[`Http2ServerRequest`]: #class-http2http2serverrequest

[`Http2ServerResponse`]: #class-http2http2serverresponse

[`Http2Session` and Sockets]: #http2session-and-sockets

[`Http2Session`'s `'stream'` event]: #event-stream

[`Http2Stream`]: #class-http2stream

[`ServerHttp2Stream`]: #class-serverhttp2stream

[`TypeError`]: errors.md#class-typeerror

[`http2.SecureServer`]: #class-http2secureserver

[`http2.Server`]: #class-http2server

[`http2.createSecureServer()`]: #http2createsecureserveroptions-onrequesthandler

[`http2.createServer()`]: #http2createserveroptions-onrequesthandler

[`http2session.close()`]: #http2sessionclosecallback

[`http2stream.pushStream()`]: #http2streampushstreamheaders-options-callback

[`import()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import

[`net.Server.close()`]: net.md#serverclosecallback

[`net.Socket.bufferSize`]: net.md#socketbuffersize

[`net.Socket.prototype.ref()`]: net.md#socketref

[`net.Socket.prototype.unref()`]: net.md#socketunref

[`net.Socket`]: net.md#class-netsocket

[`net.connect()`]: net.md#netconnect

[`net.createServer()`]: net.md#netcreateserveroptions-connectionlistener

[`request.authority`]: #requestauthority

[`request.maxHeadersCount`]: http.md#requestmaxheaderscount

[`request.socket.getPeerCertificate()`]: tls.md#tlssocketgetpeercertificatedetailed

[`request.socket`]: #requestsocket

[`response.end()`]: #responseenddata-encoding-callback

[`response.setHeader()`]: #responsesetheadername-value

[`response.socket`]: #responsesocket

[`response.writableEnded`]: #responsewritableended

[`response.write()`]: #responsewritechunk-encoding-callback

[`response.write(data, encoding)`]: http.md#responsewritechunk-encoding-callback

[`response.writeContinue()`]: #responsewritecontinue

[`response.writeHead()`]: #responsewriteheadstatuscode-statusmessage-headers

[`server.maxHeadersCount`]: http.md#servermaxheaderscount

[`tls.Server.close()`]: tls.md#serverclosecallback

[`tls.TLSSocket`]: tls.md#class-tlstlssocket

[`tls.connect()`]: tls.md#tlsconnectoptions-callback

[`tls.createServer()`]: tls.md#tlscreateserveroptions-secureconnectionlistener

[`writable.writableFinished`]: stream.md#writablewritablefinished

[error code]: #error-codes-for-rst_stream-and-goaway
