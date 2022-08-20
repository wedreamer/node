# 诊断通道

<!--introduced_in=v15.1.0-->

> 稳定性： 1 - 实验

<!-- source_link=lib/diagnostics_channel.js -->

这`node:diagnostics_channel`模块提供了一个 API 来创建命名通道
报告任意消息数据以进行诊断。

可以使用以下命令访问它：

```mjs
import diagnostics_channel from 'node:diagnostics_channel';
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');
```

它旨在使模块编写器想要报告诊断消息
将创建一个或多个顶级通道来报告消息。
通道也可以在运行时获取，但不鼓励这样做
由于这样做会产生额外的开销。频道可以导出
方便，但只要知道这个名字，就可以在任何地方获得。

如果您打算让您的模块生成诊断数据供其他人使用
消费 建议您包括名称的文档
通道与消息数据的形状一起使用。频道名称
通常应包含模块名称，以避免与以下位置的数据发生冲突
其他模块。

## 公共接口

### 概述

以下是公共 API 的简单概述。

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

// Get a reusable channel object
const channel = diagnostics_channel.channel('my-channel');

function onMessage(message, name) {
  // Received data
}

// Subscribe to the channel
diagnostics_channel.subscribe('my-channel', onMessage);

// Check if the channel has an active subscriber
if (channel.hasSubscribers) {
  // Publish data to the channel
  channel.publish({
    some: 'data'
  });
}

// Unsubscribe from the channel
diagnostics_channel.unsubscribe('my-channel', onMessage);
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

// Get a reusable channel object
const channel = diagnostics_channel.channel('my-channel');

function onMessage(message, name) {
  // Received data
}

// Subscribe to the channel
diagnostics_channel.subscribe('my-channel', onMessage);

// Check if the channel has an active subscriber
if (channel.hasSubscribers) {
  // Publish data to the channel
  channel.publish({
    some: 'data'
  });
}

// Unsubscribe from the channel
diagnostics_channel.unsubscribe('my-channel', onMessage);
```

#### `diagnostics_channel.hasSubscribers(name)`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
-->

*   `name`{字符串|符号}频道名称
*   返回：{布尔值} 如果有活动的订阅者

检查指定频道是否有活动订阅者。这在以下情况下很有帮助：
您要发送的消息可能准备成本高昂。

此 API 是可选的，但在尝试从非常
对性能敏感的代码。

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

if (diagnostics_channel.hasSubscribers('my-channel')) {
  // There are subscribers, prepare and publish message
}
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

if (diagnostics_channel.hasSubscribers('my-channel')) {
  // There are subscribers, prepare and publish message
}
```

#### `diagnostics_channel.channel(name)`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
-->

*   `name`{字符串|符号}频道名称
*   返回：{通道} 命名的通道对象

这是任何想要发布到指定
渠道。它生成一个通道对象，该对象经过优化，可减少
尽可能多地发布时间。

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

const channel = diagnostics_channel.channel('my-channel');
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

const channel = diagnostics_channel.channel('my-channel');
```

#### `diagnostics_channel.subscribe(name, onMessage)`

<!-- YAML
added:
 - v18.7.0
-->

*   `name`{字符串|符号}频道名称
*   `onMessage`{函数}用于接收通道消息的处理程序
    *   `message`{任何}消息数据
    *   `name`{字符串|符号}频道的名称

注册消息处理程序以订阅此通道。此消息处理程序
每当将消息发布到通道时，将同步运行。任何
消息处理程序中引发的错误将触发[`'uncaughtException'`]['uncaughtException'].

```mjs
import diagnostics_channel from 'diagnostics_channel';

diagnostics_channel.subscribe('my-channel', (message, name) => {
  // Received data
});
```

```cjs
const diagnostics_channel = require('diagnostics_channel');

diagnostics_channel.subscribe('my-channel', (message, name) => {
  // Received data
});
```

#### `diagnostics_channel.unsubscribe(name, onMessage)`

<!-- YAML
added:
 - v18.7.0
-->

*   `name`{字符串|符号}频道名称
*   `onMessage`{函数}要删除的上一个已订阅处理程序
*   返回：{布尔值}`true`如果找到处理程序，`false`否则。

删除以前注册到此通道的消息处理程序
[`diagnostics_channel.subscribe(name, onMessage)`][diagnostics_channel.subscribe(name, onMessage)].

```mjs
import diagnostics_channel from 'diagnostics_channel';

function onMessage(message, name) {
  // Received data
}

diagnostics_channel.subscribe('my-channel', onMessage);

diagnostics_channel.unsubscribe('my-channel', onMessage);
```

```cjs
const diagnostics_channel = require('diagnostics_channel');

function onMessage(message, name) {
  // Received data
}

diagnostics_channel.subscribe('my-channel', onMessage);

diagnostics_channel.unsubscribe('my-channel', onMessage);
```

### 类：`Channel`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
-->

班级`Channel`表示数据中的单个命名通道
管道。它用于跟踪订阅者并在有订阅者时发布消息
是订阅者在场。它作为一个单独的对象存在，以避免通道
在发布时查找，实现非常快的发布速度并允许
适合大量使用，同时产生非常低的成本。频道创建方式为
[`diagnostics_channel.channel(name)`][diagnostics_channel.channel(name)]，直接构造通道
跟`new Channel(name)`不受支持。

#### `channel.hasSubscribers`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
-->

*   返回：{布尔值} 如果有活动的订阅者

检查此频道是否有活跃的订阅者。这在以下情况下很有帮助：
您要发送的消息可能准备成本高昂。

此 API 是可选的，但在尝试从非常
对性能敏感的代码。

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

const channel = diagnostics_channel.channel('my-channel');

if (channel.hasSubscribers) {
  // There are subscribers, prepare and publish message
}
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

const channel = diagnostics_channel.channel('my-channel');

if (channel.hasSubscribers) {
  // There are subscribers, prepare and publish message
}
```

#### `channel.publish(message)`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
-->

*   `message`{任何}要发送给频道订阅者的消息

向频道的任何订阅者发布消息。这将触发
消息处理程序同步，以便它们将在同一上下文中执行。

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

const channel = diagnostics_channel.channel('my-channel');

channel.publish({
  some: 'message'
});
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

const channel = diagnostics_channel.channel('my-channel');

channel.publish({
  some: 'message'
});
```

#### `channel.subscribe(onMessage)`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
deprecated: v18.7.0
-->

> 稳定性：0 - 已弃用：使用[`diagnostics_channel.subscribe(name, onMessage)`][diagnostics_channel.subscribe(name, onMessage)]

*   `onMessage`{函数}用于接收通道消息的处理程序
    *   `message`{任何}消息数据
    *   `name`{字符串|符号}频道的名称

注册消息处理程序以订阅此通道。此消息处理程序
每当将消息发布到通道时，将同步运行。任何
消息处理程序中引发的错误将触发[`'uncaughtException'`]['uncaughtException'].

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

const channel = diagnostics_channel.channel('my-channel');

channel.subscribe((message, name) => {
  // Received data
});
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

const channel = diagnostics_channel.channel('my-channel');

channel.subscribe((message, name) => {
  // Received data
});
```

#### `channel.unsubscribe(onMessage)`

<!-- YAML
added:
 - v15.1.0
 - v14.17.0
deprecated: v18.7.0
changes:
  - version:
    - v17.1.0
    - v16.14.0
    - v14.19.0
    pr-url: https://github.com/nodejs/node/pull/40433
    description: Added return value. Added to channels without subscribers.
-->

> 稳定性：0 - 已弃用：使用[`diagnostics_channel.unsubscribe(name, onMessage)`][diagnostics_channel.unsubscribe(name, onMessage)]

*   `onMessage`{函数}要删除的上一个已订阅处理程序
*   返回：{布尔值}`true`如果找到处理程序，`false`否则。

删除以前注册到此通道的消息处理程序
[`channel.subscribe(onMessage)`][channel.subscribe(onMessage)].

```mjs
import diagnostics_channel from 'node:diagnostics_channel';

const channel = diagnostics_channel.channel('my-channel');

function onMessage(message, name) {
  // Received data
}

channel.subscribe(onMessage);

channel.unsubscribe(onMessage);
```

```cjs
const diagnostics_channel = require('node:diagnostics_channel');

const channel = diagnostics_channel.channel('my-channel');

function onMessage(message, name) {
  // Received data
}

channel.subscribe(onMessage);

channel.unsubscribe(onMessage);
```

### 内置通道

#### 断续器

`http.client.request.start`

*   `request`{http.客户端请求}

在客户端启动请求时发出。

`http.client.response.finish`

*   `request`{http.客户端请求}
*   `response`{http.传入消息}

在客户端收到响应时发出。

`http.server.request.start`

*   `request`{http.传入消息}
*   `response`{http.ServerResponse}
*   `socket`{网.插座}
*   `server`{http.服务器}

在服务器收到请求时发出。

`http.server.response.finish`

*   `request`{http.传入消息}
*   `response`{http.ServerResponse}
*   `socket`{网.插座}
*   `server`{http.服务器}

在服务器发送响应时发出。

`net.client.socket`

*   `socket`{网.插座}

在创建新的 TCP 或管道客户端套接字时发出。

`net.server.socket`

*   `socket`{网.插座}

在收到新的 TCP 或管道连接时发出。

`udp.socket`

*   `socket`{dgram.插座}

在创建新的 UDP 套接字时发出。

[`'uncaughtException'`]: process.md#event-uncaughtexception

[`channel.subscribe(onMessage)`]: #channelsubscribeonmessage

[`diagnostics_channel.channel(name)`]: #diagnostics_channelchannelname

[`diagnostics_channel.subscribe(name, onMessage)`]: #diagnostics_channelsubscribename-onmessage

[`diagnostics_channel.unsubscribe(name, onMessage)`]: #diagnostics_channelunsubscribename-onmessage
