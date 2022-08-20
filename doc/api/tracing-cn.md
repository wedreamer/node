# Trace events

<!--introduced_in=v7.7.0-->

> Stability: 1 - Experimental

<!-- source_link=lib/trace_events.js -->

`node:trace_events` 模块提供了一种机制来集中 V8、Node.js 核心和用户空间代码生成的跟踪信息.

可以使用 `--trace-event-categories` 命令行标志或使用 `node:trace_events` 模块启用跟踪。 `--trace-event-categories` 标志接受逗号分隔的类别名称列表.

可用的类别是:

* `node`: An empty placeholder.
* `node.async_hooks`: Enables capture of detailed [`async_hooks`][] trace data.
  The [`async_hooks`][] events have a unique `asyncId` and a special `triggerId`
  `triggerAsyncId` property.
* `node.bootstrap`: Enables capture of Node.js bootstrap milestones.
* `node.console`: Enables capture of `console.time()` and `console.count()`
  output.
* `node.dns.native`: Enables capture of trace data for DNS queries.
* `node.net.native`: Enables capture of trace data for network.
* `node.environment`: Enables capture of Node.js Environment milestones.
* `node.fs.sync`: Enables capture of trace data for file system sync methods.
* `node.perf`: Enables capture of [Performance API][] measurements.
  * `node.perf.usertiming`: Enables capture of only Performance API User Timing
    measures and marks.
  * `node.perf.timerify`: Enables capture of only Performance API timerify
    measurements.
* `node.promises.rejections`: Enables capture of trace data tracking the number
  of unhandled Promise rejections and handled-after-rejections.
* `node.vm.script`: Enables capture of trace data for the `node:vm` module's
  `runInNewContext()`, `runInContext()`, and `runInThisContext()` methods.
* `v8`: The [V8][] events are GC, compiling, and execution related.
* `node.http`: Enables capture of trace data for http request / response.

默认情况下，启用了 `node`、`node.async_hooks` 和 `v8` 类别.

```bash
node --trace-event-categories v8,node,node.async_hooks server.js
```

Node.js 的早期版本需要使用 `--trace-events-enabled` 标志来启用跟踪事件。此要求已被删除。但是，`--trace-events-enabled` 标志_may_ 仍然可以使用，默认情况下将启用 `node`、`node.async_hooks` 和 `v8` 跟踪事件类别.

```bash
node --trace-events-enabled

# is equivalent to

node --trace-event-categories v8,node,node.async_hooks
```

或者，可以使用 `node:trace_events` 模块启用跟踪事件:

```js
const trace_events = require('node:trace_events');
const tracing = trace_events.createTracing({ categories: ['node.perf'] });
tracing.enable();  // Enable trace event capture for the 'node.perf' category

// do work

tracing.disable();  // Disable trace event capture for the 'node.perf' category
```

在启用跟踪的情况下运行 Node.js 将生成可以在 [`chrome://tracing`](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool) 中打开的日志文件) Chrome 的标签.

默认情况下，日志文件名为 `node_trace.${rotation}.log`，其中 `${rotation}` 是递增的日志轮换 ID。文件路径模式可以用 `--trace-event-file-pattern` 指定，它接受支持 `${rotation}` 和 `${pid}` 的模板字符串:

```bash
node --trace-event-categories v8 --trace-event-file-pattern '${pid}-${rotation}.log' server.js
```

为确保在 SIGINT、SIGTERM 或 SIGBREAK 等信号事件之后正确生成日志文件，请确保在代码中包含适当的处理程序，例如:

```js
process.on('SIGINT', function onSigint() {
  console.info('Received SIGINT.');
  process.exit(130);  // Or applicable exit code depending on OS and signal
});
```

跟踪系统使用与 `process.hrtime()` 使用的时间源相同的时间源.
然而，跟踪事件时间戳以微秒表示，与返回纳秒的 `process.hrtime()` 不同.

此模块的功能在 [`Worker`][] 线程中不可用.

## The `node:trace_events` module

<!-- YAML
added: v10.0.0
-->

### `Tracing` object

<!-- YAML
added: v10.0.0
-->

`Tracing` 对象用于启用或禁用类别集的跟踪。使用 `trace_events.createTracing()` 方法创建实例.

创建时，“跟踪”对象被禁用。调用 `tracing.enable()` 方法将类别添加到启用的跟踪事件类别集中.调用 `tracing.disable()` 将从启用的跟踪事件类别集中删除类别.

#### `tracing.categories`

<!-- YAML
added: v10.0.0
-->

* {string}

此“跟踪”对象涵盖的跟踪事件类别的逗号分隔列表.

#### `tracing.disable()`

<!-- YAML
added: v10.0.0
-->

禁用此“跟踪”对象.

只有被其他启用的 `Tracing` 对象和 _not_ 由 `--trace-event-categories` 标志指定的跟踪事件类别_not_ 将被禁用.

```js
const trace_events = require('node:trace_events');
const t1 = trace_events.createTracing({ categories: ['node', 'v8'] });
const t2 = trace_events.createTracing({ categories: ['node.perf', 'node'] });
t1.enable();
t2.enable();

// Prints 'node,node.perf,v8'
console.log(trace_events.getEnabledCategories());

t2.disable(); // Will only disable emission of the 'node.perf' category

// Prints 'node,v8'
console.log(trace_events.getEnabledCategories());
```

#### `tracing.enable()`

<!-- YAML
added: v10.0.0
-->

为 `Tracing` 对象所涵盖的类别集启用此 `Tracing` 对象.

#### `tracing.enabled`

<!-- YAML
added: v10.0.0
-->

* {boolean} `true` only if the `Tracing` object has been enabled.

### `trace_events.createTracing(options)`

<!-- YAML
added: v10.0.0
-->

* `options` {Object}
  * `categories` {string\[]} An array of trace category names. Values included
    in the array are coerced to a string when possible. An error will be
    thrown if the value cannot be coerced.
* Returns: {Tracing}.

为给定的 `categories` 集创建并返回一个 `Tracing` 对象.

```js
const trace_events = require('node:trace_events');
const categories = ['node.perf', 'node.async_hooks'];
const tracing = trace_events.createTracing({ categories });
tracing.enable();
// do stuff
tracing.disable();
```

### `trace_events.getEnabledCategories()`

<!-- YAML
added: v10.0.0
-->

* Returns: {string}

返回所有当前启用的跟踪事件类别的逗号分隔列表。当前启用的跟踪事件类别集由所有当前启用的“跟踪”对象和使用“--trace-event-categories”标志启用的任何类别的_union_确定.

给定下面的文件 `test.js`，命令 `node --trace-event-categories node.perf test.js` 将打印 `'node.async_hooks,node.perf'` 到控制台.

```js
const trace_events = require('node:trace_events');
const t1 = trace_events.createTracing({ categories: ['node.async_hooks'] });
const t2 = trace_events.createTracing({ categories: ['node.perf'] });
const t3 = trace_events.createTracing({ categories: ['v8'] });

t1.enable();
t2.enable();

console.log(trace_events.getEnabledCategories());
```

[Performance API]: perf_hooks.md
[V8]: v8.md
[`Worker`]: worker_threads.md#class-worker
[`async_hooks`]: async_hooks.md
