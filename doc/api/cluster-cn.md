# Cluster

<!--introduced_in=v0.10.0-->

> Stability: 2 - Stable

<!-- source_link=lib/cluster.js -->

Node.js 进程集群可用于运行多个 Node.js 实例，这些实例可以在其应用程序线程之间分配工作负载。当不需要进程隔离时，请改用 [`worker_threads`][] 模块，它允许在单个 Node.js 实例中运行多个应用程序线程.

集群模块允许轻松创建所有共享服务器端口的子进程.

```mjs
import cluster from 'node:cluster';
import http from 'node:http';
import { cpus } from 'node:os';
import process from 'node:process';

const numCPUs = cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

```cjs
const cluster = require('node:cluster');
const http = require('node:http');
const numCPUs = require('node:os').cpus().length;
const process = require('node:process');

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

运行 Node.js 现在将在工作人员之间共享端口 8000:

```console
$ node server.js
Primary 3596 is running
Worker 4324 started
Worker 4520 started
Worker 6056 started
Worker 5644 started
```

在 Windows 上，尚无法在工作人员中设置命名管道服务器.

## How it works

<!--type=misc-->

工作进程是使用 [`child_process.fork()`][] 方法生成的，因此它们可以通过 IPC 与父进程通信并来回传递服务器句柄.

cluster 模块支持两种分配传入连接的方法.

第一个（以及除 Windows 之外的所有平台上的默认方法）是循环方法，其中主进程侦听端口，接受新连接并以循环方式将它们分发给工作人员，其中一些内置 -聪明地避免工作进程超载.

第二种方法是主进程创建侦听套接字并将其发送给感兴趣的工作人员。然后工作人员直接接受传入的连接.

理论上，第二种方法应该提供最佳性能.
然而，在实践中，由于操作系统调度程序的变幻莫测，分布往往非常不平衡。已观察到负载超过 70% 的连接仅在两个进程中结束,
在总共八个.

因为 `server.listen()` 将大部分工作交给主进程，所以在三种情况下，普通 Node.js 进程和集群工作者之间的行为会有所不同:

1. `server.listen({fd: 7})` 因为消息是传给主节点的，所以会监听父节点**中的文件描述符7，并将句柄传递给worker，而不是监听worker对7号文件描述符引用什么的想法.
2. `server.listen(handle)` 显式侦听句柄将导致工作人员使用提供的句柄，而不是与主进程对话.
3. `server.listen(0)` 通常，这将导致服务器监听随机端口。但是，在集群中，每个工作人员每次执行“listen(0)”时都会收到相同的“随机”端口。本质上，端口第一次是随机的，但之后是可预测的。要侦听唯一端口，请根据集群工作人员 ID 生成端口号.

Node.js 不提供路由逻辑。因此，重要的是设计一个应用程序，使其不会过于依赖内存中的数据对象来处理会话和登录等事情.

因为工人都是独立的进程，它们可以根据程序的需要被杀死或重新生成，而不会影响其他工人。只要还有一些工作人员还活着，服务器就会继续接受连接。如果没有工人活着，现有的连接将被丢弃，新的连接将被拒绝。然而，Node.js 不会自动管理工作人员的数量。根据自己的需要管理工作池是应用程序的责任.

尽管 `node:cluster` 模块的主要用例是网络，但它也可以用于需要工作进程的其他用例.

## Class: `Worker`

<!-- YAML
added: v0.7.0
-->

* Extends: {EventEmitter}

`Worker` 对象包含有关工作人员的所有公共信息和方法.
在主节点中，它可以使用 `cluster.workers` 获得。在工作人员中，可以使用 `cluster.worker` 获得.

### Event: `'disconnect'`

<!-- YAML
added: v0.7.7
-->

类似于 `cluster.on('disconnect')` 事件，但特定于该工作人员.

```js
cluster.fork().on('disconnect', () => {
  // Worker has disconnected
});
```

### Event: `'error'`

<!-- YAML
added: v0.7.3
-->

此事件与 [`child_process.fork()`][] 提供的事件相同.

在worker中，也可以使用`process.on('error')`.

### Event: `'exit'`

<!-- YAML
added: v0.11.2
-->

* `code` {number} 退出代码，如果正常退出.
* `signal` {string} 导致进程被杀死的信号名称（例如“SIGHUP”）.

类似于 `cluster.on('exit')` 事件，但特定于该工作人员.

```mjs
import cluster from 'node:cluster';

if (cluster.isPrimary) {
  const worker = cluster.fork();
  worker.on('exit', (code, signal) => {
    if (signal) {
      console.log(`worker was killed by signal: ${signal}`);
    } else if (code !== 0) {
      console.log(`worker exited with error code: ${code}`);
    } else {
      console.log('worker success!');
    }
  });
}
```

```cjs
const cluster = require('node:cluster');

if (cluster.isPrimary) {
  const worker = cluster.fork();
  worker.on('exit', (code, signal) => {
    if (signal) {
      console.log(`worker was killed by signal: ${signal}`);
    } else if (code !== 0) {
      console.log(`worker exited with error code: ${code}`);
    } else {
      console.log('worker success!');
    }
  });
}
```

### Event: `'listening'`

<!-- YAML
added: v0.7.0
-->

* `address` {Object}

类似于 `cluster.on('listening')` 事件，但特定于该工作人员.

```mjs
cluster.fork().on('listening', (address) => {
  // Worker is listening
});
```

```cjs
cluster.fork().on('listening', (address) => {
  // Worker is listening
});
```

它不会在工作人员中发出.

### Event: `'message'`

<!-- YAML
added: v0.7.0
-->

* `message` {Object}
* `handle` {undefined|Object}

类似于 `cluster` 的 `'message'` 事件，但特定于这个 worker.

在worker中，也可以使用`process.on('message')`.

参见 [`process` 事件：`'message'`][].

这是使用消息系统的示例。它在主进程中记录工作人员收到的 HTTP 请求数:

```mjs
import cluster from 'node:cluster';
import http from 'node:http';
import { cpus } from 'node:os';
import process from 'node:process';

if (cluster.isPrimary) {

  // Keep track of http requests
  let numReqs = 0;
  setInterval(() => {
    console.log(`numReqs = ${numReqs}`);
  }, 1000);

  // Count requests
  function messageHandler(msg) {
    if (msg.cmd && msg.cmd === 'notifyRequest') {
      numReqs += 1;
    }
  }

  // Start workers and listen for messages containing notifyRequest
  const numCPUs = cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  for (const id in cluster.workers) {
    cluster.workers[id].on('message', messageHandler);
  }

} else {

  // Worker processes have a http server.
  http.Server((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');

    // Notify primary about the request
    process.send({ cmd: 'notifyRequest' });
  }).listen(8000);
}
```

```cjs
const cluster = require('node:cluster');
const http = require('node:http');
const process = require('node:process');

if (cluster.isPrimary) {

  // Keep track of http requests
  let numReqs = 0;
  setInterval(() => {
    console.log(`numReqs = ${numReqs}`);
  }, 1000);

  // Count requests
  function messageHandler(msg) {
    if (msg.cmd && msg.cmd === 'notifyRequest') {
      numReqs += 1;
    }
  }

  // Start workers and listen for messages containing notifyRequest
  const numCPUs = require('node:os').cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  for (const id in cluster.workers) {
    cluster.workers[id].on('message', messageHandler);
  }

} else {

  // Worker processes have a http server.
  http.Server((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');

    // Notify primary about the request
    process.send({ cmd: 'notifyRequest' });
  }).listen(8000);
}
```

### Event: `'online'`

<!-- YAML
added: v0.7.0
-->

类似于 `cluster.on('online')` 事件，但特定于该工作人员.

```js
cluster.fork().on('online', () => {
  // Worker is online
});
```

它不会在工作人员中发出.

### `worker.disconnect()`

<!-- YAML
added: v0.7.7
changes:
  - version: v7.3.0
    pr-url: https://github.com/nodejs/node/pull/10019
    description: This method now returns a reference to `worker`.
-->

* 返回：{cluster.Worker} 对 `worker` 的引用.

在 worker 中，这个函数会关闭所有服务器，等待这些服务器上的 `'close'` 事件，然后断开 IPC 通道.

在主节点中，一个内部消息被发送到工作线程，导致它自己调用`.disconnect()`.

导致设置“.exitedAfterDisconnect”.

服务器关闭后，它将不再接受新的连接，但任何其他侦听工作人员都可以接受连接。现有的连接将被允许照常关闭。当不再存在连接时，请参阅 [`server.close()`][]，到 worker 的 IPC 通道将关闭，使其优雅地死去.

以上_only_适用于服务器连接，客户端连接不会被worker自动关闭，disconnect也不会等他们关闭才退出.

在worker中，存在`process.disconnect`，但不是这个函数;
它是 [`disconnect()`][].

因为长期存在的服务器连接可能会阻止工作人员断开连接，所以发送消息可能很有用，因此可以采取特定于应用程序的操作来关闭它们。实现超时也可能很有用，如果在一段时间后没有发出“断开连接”事件，则杀死工作人员.

```js
if (cluster.isPrimary) {
  const worker = cluster.fork();
  let timeout;

  worker.on('listening', (address) => {
    worker.send('shutdown');
    worker.disconnect();
    timeout = setTimeout(() => {
      worker.kill();
    }, 2000);
  });

  worker.on('disconnect', () => {
    clearTimeout(timeout);
  });

} else if (cluster.isWorker) {
  const net = require('node:net');
  const server = net.createServer((socket) => {
    // Connections never end
  });

  server.listen(8000);

  process.on('message', (msg) => {
    if (msg === 'shutdown') {
      // Initiate graceful close of any connections to server
    }
  });
}
```

### `worker.exitedAfterDisconnect`

<!-- YAML
added: v6.0.0
-->

* {boolean}

如果工作人员因 `.disconnect()` 退出，则此属性为 `true`.
如果工人以任何其他方式退出，则为“假”。如果worker没有退出，则为`undefined`.

布尔值 [`worker.exitedAfterDisconnect`][] 允许区分自愿退出和意外退出，主节点可以根据此值选择不重生工人.

```js
cluster.on('exit', (worker, code, signal) => {
  if (worker.exitedAfterDisconnect === true) {
    console.log('Oh, it was just voluntary – no need to worry');
  }
});

// kill worker
worker.kill();
```

### `worker.id`

<!-- YAML
added: v0.8.0
-->

* {integer}

每个新工人都有自己唯一的 id，这个 id 存储在 `id`.

当工作人员还活着时，这是在 `cluster.workers` 中索引它的键.

### `worker.isConnected()`

<!-- YAML
added: v0.11.14
-->

如果 worker 通过其 IPC 通道连接到其主节点，此函数返回“true”，否则返回“false”。工作人员在创建后连接到其主服务器。发出 `'disconnect'` 事件后断开连接.

### `worker.isDead()`

<!-- YAML
added: v0.11.14
-->

如果工作进程已终止（由于退出或被发出信号），此函数将返回 `true`。否则，它返回 `false`.

```mjs
import cluster from 'node:cluster';
import http from 'node:http';
import { cpus } from 'node:os';
import process from 'node:process';

const numCPUs = cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('fork', (worker) => {
    console.log('worker is dead:', worker.isDead());
  });

  cluster.on('exit', (worker, code, signal) => {
    console.log('worker is dead:', worker.isDead());
  });
} else {
  // Workers can share any TCP connection. In this case, it is an HTTP server.
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Current process\n ${process.pid}`);
    process.kill(process.pid);
  }).listen(8000);
}
```

```cjs
const cluster = require('node:cluster');
const http = require('node:http');
const numCPUs = require('node:os').cpus().length;
const process = require('node:process');

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('fork', (worker) => {
    console.log('worker is dead:', worker.isDead());
  });

  cluster.on('exit', (worker, code, signal) => {
    console.log('worker is dead:', worker.isDead());
  });
} else {
  // Workers can share any TCP connection. In this case, it is an HTTP server.
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Current process\n ${process.pid}`);
    process.kill(process.pid);
  }).listen(8000);
}
```

### `worker.kill([signal])`

<!-- YAML
added: v0.9.12
-->

* `signal` {string} 要发送到工作进程的终止信号的名称。 **默认值：** `'SIGTERM'`

此功能将杀死工人。在主要工作人员中，它通过断开“worker.process”来完成此操作，一旦断开连接，就会使用“信号”终止。在工作人员中，它通过使用“信号”终止进程来实现.

`kill()` 函数在不等待正常断开连接的情况下杀死工作进程，它与`worker.process.kill()` 具有相同的行为.

为了向后兼容，此方法别名为 `worker.destroy()`.

在worker中，存在`process.kill()`，但不是这个函数;
它是 [`kill()`][].

### `worker.process`

<!-- YAML
added: v0.7.0
-->

* {ChildProcess}

所有工作程序都是使用 [`child_process.fork()`][] 创建的，此函数返回的对象存储为 `.process`。在工作人员中，存储了全局“进程”.

请参阅：[子进程模块][].

如果 `process` 上发生 `'disconnect'` 事件并且 `.exitedAfterDisconnect` 不是 `true`，则工作人员将调用 `process.exit(0)`。这可以防止意外断开连接.

### `worker.send(message[, sendHandle[, options]][, callback])`

<!-- YAML
added: v0.7.0
changes:
  - version: v4.0.0
    pr-url: https://github.com/nodejs/node/pull/2620
    description: The `callback` parameter is supported now.
-->

* `message` {Object}
* `sendHandle` {Handle}
* `options` {Object} The `options` argument, if present, is an object used to
  parameterize the sending of certain types of handles. `options` supports
  the following properties:
  * `keepOpen` {boolean} A value that can be used when passing instances of
    `net.Socket`. When `true`, the socket is kept open in the sending process.
    **Default:** `false`.
* `callback` {Function}
* Returns: {boolean}

向工作人员或主节点发送消息，可选择使用句柄.

在主节点中，这会向特定的工作人员发送消息。它等同于 [`ChildProcess.send()`][].

在工作者中，这会向主节点发送消息。它与 `process.send()` 相同.

此示例将回显来自主节点的所有消息:

```js
if (cluster.isPrimary) {
  const worker = cluster.fork();
  worker.send('hi there');

} else if (cluster.isWorker) {
  process.on('message', (msg) => {
    process.send(msg);
  });
}
```

## Event: `'disconnect'`

<!-- YAML
added: v0.7.9
-->

* `worker` {cluster.Worker}

在工作人员 IPC 通道断开连接后发出。当工作人员正常退出、被杀死或手动断开连接（例如使用`worker.disconnect()`）时，可能会发生这种情况.

`'disconnect'` 和 `'exit'` 事件之间可能存在延迟。这些事件可用于检测进程是否卡在清理中或是否存在长期连接.

```js
cluster.on('disconnect', (worker) => {
  console.log(`The worker #${worker.id} has disconnected`);
});
```

## Event: `'exit'`

<!-- YAML
added: v0.7.9
-->

* `worker` {cluster.Worker}
* `code` {number} The exit code, if it exited normally.
* `signal` {string} The name of the signal (e.g. `'SIGHUP'`) that caused
  the process to be killed.

当任何工作人员死亡时，集群模块将发出“退出”事件.

这可用于通过再次调用 [`.fork()`][] 来重新启动工作程序.

```js
cluster.on('exit', (worker, code, signal) => {
  console.log('worker %d died (%s). restarting...',
              worker.process.pid, signal || code);
  cluster.fork();
});
```

参见 [`child_process` 事件：`'exit'`][].

## Event: `'fork'`

<!-- YAML
added: v0.7.0
-->

* `worker` {cluster.Worker}

当一个新的工人被分叉时，集群模块将发出一个 `'fork'` 事件.
这可用于记录工作人员活动，并创建自定义超时.

```js
const timeouts = [];
function errorMsg() {
  console.error('Something must be wrong with the connection ...');
}

cluster.on('fork', (worker) => {
  timeouts[worker.id] = setTimeout(errorMsg, 2000);
});
cluster.on('listening', (worker, address) => {
  clearTimeout(timeouts[worker.id]);
});
cluster.on('exit', (worker, code, signal) => {
  clearTimeout(timeouts[worker.id]);
  errorMsg();
});
```

## Event: `'listening'`

<!-- YAML
added: v0.7.0
-->

* `worker` {cluster.Worker}
* `address` {Object}

从 worker 调用 `listen()` 后，当服务器上发出 `'listening'` 事件时，主节点的 `cluster` 上也会发出 `'listening'` 事件.

事件处理程序使用两个参数执行，`worker` 包含工作对象，`address` 对象包含以下连接属性：`address`、`port` 和 `addressType`。如果工作人员正在侦听多个地址，这将非常有用.

```js
cluster.on('listening', (worker, address) => {
  console.log(
    `A worker is now connected to ${address.address}:${address.port}`);
});
```

`addressType` 是其中之一:

* `4` (TCPv4)
* `6` (TCPv6)
* `-1` (Unix domain socket)
* `'udp4'` or `'udp6'` (UDPv4 or UDPv6)

## Event: `'message'`

<!-- YAML
added: v2.5.0
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5361
    description: The `worker` parameter is passed now; see below for details.
-->

* `worker` {cluster.Worker}
* `message` {Object}
* `handle` {undefined|Object}

当集群主节点收到来自任何工作人员的消息时发出.

请参阅 [`child_process` 事件：`'message'`][].

## Event: `'online'`

<!-- YAML
added: v0.7.0
-->

* `worker` {cluster.Worker}

在 fork 一个新的 worker 后，worker 应该回复一条在线消息.
当主节点收到在线消息时，它将发出此事件.
`'fork'` 和 `'online'` 的区别在于，当主分叉一个 worker 时会发出 fork，而 `'online'` 会在 worker 运行时发出.

```js
cluster.on('online', (worker) => {
  console.log('Yay, the worker responded after it was forked');
});
```

## Event: `'setup'`

<!-- YAML
added: v0.7.1
-->

* `settings` {Object}

每次调用 [`.setupPrimary()`][] 时发出.

`settings` 对象是调用 [`.setupPrimary()`][] 时的 `cluster.settings` 对象，并且只是建议性的，因为可以对 [`.setupPrimary()`][] 进行多次调用在一个滴答声中.

如果准确性很重要，请使用`cluster.settings`.

## `cluster.disconnect([callback])`

<!-- YAML
added: v0.7.7
-->

* `callback` {Function} 当所有工作人员断开连接并且句柄关闭时调用.

在 `cluster.workers` 中的每个工作人员上调用 `.disconnect()`.

当它们断开连接时，所有内部句柄都将关闭，如果没有其他事件在等待，则允许主进程正常终止.

该方法接受一个可选的回调参数，该参数将在完成时调用.

这只能从主进程调用.

## `cluster.fork([env])`

<!-- YAML
added: v0.6.0
-->

* `env` {Object} Key/value pairs to add to worker process environment.
* Returns: {cluster.Worker}

产生一个新的工作进程.

这只能从主进程调用.

## `cluster.isMaster`

<!-- YAML
added: v0.8.1
deprecated: v16.0.0
-->

[`cluster.isPrimary`][] 的已弃用别名.

## `cluster.isPrimary`

<!-- YAML
added: v16.0.0
-->

* {boolean}

如果进程是主进程，则为真。这是由 `process.env.NODE_UNIQUE_ID` 决定的。如果 `process.env.NODE_UNIQUE_ID` 未定义，则 `isPrimary` 为 `true`.

## `cluster.isWorker`

<!-- YAML
added: v0.6.0
-->

* {boolean}

如果进程不是主进程，则为真（它是 `cluster.isPrimary` 的否定）.

## `cluster.schedulingPolicy`

<!-- YAML
added: v0.11.2
-->

调度策略，“cluster.SCHED_RR”用于循环或“cluster.SCHED_NONE”留给操作系统。这是一个全局设置，一旦第一个工作程序被生成，或者 [`.setupPrimary()`][] 被调用，以先到者为准.

`SCHED_RR` 是除 Windows 以外的所有操作系统的默认值.
一旦 libuv 能够有效地分发 IOCP 句柄而不会对性能造成很大影响，Windows 将更改为“SCHED_RR”.

`cluster.schedulingPolicy` 也可以通过`NODE_CLUSTER_SCHED_POLICY` 环境变量来设置。有效值为 `'rr'` 和 `'none'`.

## `cluster.settings`

<!-- YAML
added: v0.7.1
changes:
  - version:
     - v13.2.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30162
    description: The `serialization` option is supported now.
  - version: v9.5.0
    pr-url: https://github.com/nodejs/node/pull/18399
    description: The `cwd` option is supported now.
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/17412
    description: The `windowsHide` option is supported now.
  - version: v8.2.0
    pr-url: https://github.com/nodejs/node/pull/14140
    description: The `inspectPort` option is supported now.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7838
    description: The `stdio` option is supported now.
-->

* {Object}
  * `execArgv` {string\[]} List of string arguments passed to the Node.js
    executable. **Default:** `process.execArgv`.
  * `exec` {string} File path to worker file. **Default:** `process.argv[1]`.
  * `args` {string\[]} String arguments passed to worker.
    **Default:** `process.argv.slice(2)`.
  * `cwd` {string} Current working directory of the worker process. **Default:**
    `undefined` (inherits from parent process).
  * `serialization` {string} Specify the kind of serialization used for sending
    messages between processes. Possible values are `'json'` and `'advanced'`.
    See [Advanced serialization for `child_process`][] for more details.
    **Default:** `false`.
  * `silent` {boolean} Whether or not to send output to parent's stdio.
    **Default:** `false`.
  * `stdio` {Array} Configures the stdio of forked processes. Because the
    cluster module relies on IPC to function, this configuration must contain an
    `'ipc'` entry. When this option is provided, it overrides `silent`.
  * `uid` {number} Sets the user identity of the process. (See setuid(2).)
  * `gid` {number} Sets the group identity of the process. (See setgid(2).)
  * `inspectPort` {number|Function} Sets inspector port of worker.
    This can be a number, or a function that takes no arguments and returns a
    number. By default each worker gets its own port, incremented from the
    primary's `process.debugPort`.
  * `windowsHide` {boolean} Hide the forked processes console window that would
    normally be created on Windows systems. **Default:** `false`.

调用 [`.setupPrimary()`][]（或 [`.fork()`][]）后，此设置对象将包含设置，包括默认值.

此对象不打算手动更改或设置.

## `cluster.setupMaster([settings])`

<!-- YAML
added: v0.7.1
deprecated: v16.0.0
changes:
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7838
    description: The `stdio` option is supported now.
-->

[`.setupPrimary()`][] 的已弃用别名.

## `cluster.setupPrimary([settings])`

<!-- YAML
added: v16.0.0
-->

* `settings` {Object} See [`cluster.settings`][].

`setupPrimary` 用于更改默认的 'fork' 行为。调用后，设置将出现在 `cluster.settings` 中.

任何设置更改只会影响未来对 [`.fork()`][] 的调用，并且对已经运行的工作人员没有影响.

无法通过 `.setupPrimary()` 设置的 worker 的唯一属性是传递给 [`.fork()`][] 的 `env`.

上述默认值仅适用于第一次调用；以后调用的默认值是调用 `cluster.setupPrimary()` 时的当前值.

```mjs
import cluster from 'node:cluster';

cluster.setupPrimary({
  exec: 'worker.js',
  args: ['--use', 'https'],
  silent: true
});
cluster.fork(); // https worker
cluster.setupPrimary({
  exec: 'worker.js',
  args: ['--use', 'http']
});
cluster.fork(); // http worker
```

```cjs
const cluster = require('node:cluster');

cluster.setupPrimary({
  exec: 'worker.js',
  args: ['--use', 'https'],
  silent: true
});
cluster.fork(); // https worker
cluster.setupPrimary({
  exec: 'worker.js',
  args: ['--use', 'http']
});
cluster.fork(); // http worker
```

这只能从主进程调用.

## `cluster.worker`

<!-- YAML
added: v0.7.0
-->

* {Object}

对当前工作对象的引用。在主进程中不可用.

```mjs
import cluster from 'node:cluster';

if (cluster.isPrimary) {
  console.log('I am primary');
  cluster.fork();
  cluster.fork();
} else if (cluster.isWorker) {
  console.log(`I am worker #${cluster.worker.id}`);
}
```

```cjs
const cluster = require('node:cluster');

if (cluster.isPrimary) {
  console.log('I am primary');
  cluster.fork();
  cluster.fork();
} else if (cluster.isWorker) {
  console.log(`I am worker #${cluster.worker.id}`);
}
```

## `cluster.workers`

<!-- YAML
added: v0.7.0
-->

* {Object}

存储活动工作对象的哈希，由“id”字段键控。这使得遍历所有工作人员变得容易。仅在主进程中可用.

在工作人员断开连接并退出后，工作人员将从 `cluster.workers` 中删除。这两个事件之间的顺序无法提前确定。但是，可以保证从 `cluster.workers` 列表中删除发生在最后一个 `'disconnect'` 或 `'exit'` 事件发出之前.

```mjs
import cluster from 'node:cluster';

for (const worker of Object.values(cluster.workers)) {
  worker.send('big announcement to all workers');
}
```

```cjs
const cluster = require('node:cluster');

for (const worker of Object.values(cluster.workers)) {
  worker.send('big announcement to all workers');
}
```

[Advanced serialization for `child_process`]: child_process.md#advanced-serialization
[Child Process module]: child_process.md#child_processforkmodulepath-args-options
[`.fork()`]: #clusterforkenv
[`.setupPrimary()`]: #clustersetupprimarysettings
[`ChildProcess.send()`]: child_process.md#subprocesssendmessage-sendhandle-options-callback
[`child_process.fork()`]: child_process.md#child_processforkmodulepath-args-options
[`child_process` event: `'exit'`]: child_process.md#event-exit
[`child_process` event: `'message'`]: child_process.md#event-message
[`cluster.isPrimary`]: #clusterisprimary
[`cluster.settings`]: #clustersettings
[`disconnect()`]: child_process.md#subprocessdisconnect
[`kill()`]: process.md#processkillpid-signal
[`process` event: `'message'`]: process.md#event-message
[`server.close()`]: net.md#event-close
[`worker.exitedAfterDisconnect`]: #workerexitedafterdisconnect
[`worker_threads`]: worker_threads.md
