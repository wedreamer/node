# 性能测量接口

<!--introduced_in=v8.5.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/perf_hooks.js -->

此模块提供 W3C 子集的实现
[网络性能接口][Web Performance APIs]以及用于
特定于节点.js性能度量。

节点.js支持以下[网络性能接口][Web Performance APIs]:

*   [高分辨率时间][High Resolution Time]
*   [性能时间表][Performance Timeline]
*   [用户计时][User Timing]
*   [资源计时][Resource Timing]

```js
const { PerformanceObserver, performance } = require('node:perf_hooks');

const obs = new PerformanceObserver((items) => {
  console.log(items.getEntries()[0].duration);
  performance.clearMarks();
});
obs.observe({ type: 'measure' });
performance.measure('Start to Now');

performance.mark('A');
doSomeLongRunningProcess(() => {
  performance.measure('A to Now', 'A');

  performance.mark('B');
  performance.measure('A to B', 'A', 'B');
});
```

## `perf_hooks.performance`

<!-- YAML
added: v8.5.0
-->

可用于从当前收集性能指标的对象
节点.js实例。它类似于[`window.performance`][window.performance]在浏览器中。

### `performance.clearMarks([name])`

<!-- YAML
added: v8.5.0
-->

*   `name`{字符串}

如果`name`未提供，则全部删除`PerformanceMark`对象从
性能时间表。如果`name`，仅删除命名标记。

### `performance.clearMeasures([name])`

<!-- YAML
added: v16.7.0
-->

*   `name`{字符串}

如果`name`未提供，则全部删除`PerformanceMeasure`对象从
性能时间表。如果`name`，仅删除命名度量值。

### `performance.clearResourceTimings([name])`

<!-- YAML
added: v18.2.0
-->

*   `name`{字符串}

如果`name`未提供，则全部删除`PerformanceResourceTiming`对象来自
资源时间线。如果`name`，仅删除命名资源。

### `performance.eventLoopUtilization([utilization1[, utilization2]])`

<!-- YAML
added:
 - v14.10.0
 - v12.19.0
-->

*   `utilization1`{对象}上一次调用的结果
    `eventLoopUtilization()`.
*   `utilization2`{对象}上一次调用的结果
    `eventLoopUtilization()`之前`utilization1`.
*   返回 {对象}
    *   `idle`{数字}
    *   `active`{数字}
    *   `utilization`{数字}

这`eventLoopUtilization()`方法返回一个对象，该对象包含
事件循环作为
高分辨率毫秒计时器。这`utilization`值是计算出的
事件循环利用率 （ELU）。

如果引导尚未在主线程上完成，则属性具有
的价值`0`.ELU 立即在以下位置可用[工作线程][Worker threads]因为
引导发生在事件循环中。

双`utilization1`和`utilization2`是可选参数。

如果`utilization1`被传递，则当前调用之间的增量`active`
和`idle`时间，以及相应的`utilization`值为
计算并返回（类似于[`process.hrtime()`][process.hrtime()]).

如果`utilization1`和`utilization2`都通过了，那么 delta 是
在两个参数之间计算。这是一个方便的选择，因为，
与[`process.hrtime()`][process.hrtime()]，计算 ELU 比
单个减法。

ELU 类似于 CPU 利用率，只是它只测量事件循环
统计信息，而不是 CPU 使用率。它表示事件的时间百分比
循环已在事件循环的事件提供程序之外花费（例如`epoll_wait`).
不考虑其他 CPU 空闲时间。下面是一个示例
一个大部分空闲的进程将如何具有高 ELU。

```js
'use strict';
const { eventLoopUtilization } = require('node:perf_hooks').performance;
const { spawnSync } = require('node:child_process');

setImmediate(() => {
  const elu = eventLoopUtilization();
  spawnSync('sleep', ['5']);
  console.log(eventLoopUtilization(elu).utilization);
});
```

尽管 CPU 在运行此脚本时大部分处于空闲状态，但
`utilization`是`1`.这是因为调用
[`child_process.spawnSync()`][child_process.spawnSync()]阻止事件循环继续。

传入用户定义对象，而不是上一次调用的结果
`eventLoopUtilization()`将导致未定义的行为。返回值
不保证反映事件循环的任何正确状态。

### `performance.getEntries()`

<!-- YAML
added: v16.7.0
-->

*   返回： {PerformanceEntry\[]}

返回`PerformanceEntry`按时间顺序排列的对象
尊重`performanceEntry.startTime`.如果您只对以下内容感兴趣
某些类型或具有特定名称的性能条目，请参阅
`performance.getEntriesByType()`和`performance.getEntriesByName()`.

### `performance.getEntriesByName(name[, type])`

<!-- YAML
added: v16.7.0
-->

*   `name`{字符串}
*   `type`{字符串}
*   返回： {PerformanceEntry\[]}

返回`PerformanceEntry`按时间顺序排列的对象
关于`performanceEntry.startTime`谁的`performanceEntry.name`是
等于`name`，以及（可选）其`performanceEntry.entryType`等于
`type`.

### `performance.getEntriesByType(type)`

<!-- YAML
added: v16.7.0
-->

*   `type`{字符串}
*   返回： {PerformanceEntry\[]}

返回`PerformanceEntry`按时间顺序排列的对象
关于`performanceEntry.startTime`谁的`performanceEntry.entryType`
等于`type`.

### `performance.mark([name[, options]])`

<!-- YAML
added: v8.5.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Updated to conform to the User Timing Level 3 specification.
-->

*   `name`{字符串}
*   `options`{对象}
    *   `detail`{任何}要包含在标记中的其他可选详细信息。
    *   `startTime`{数字}用作标记时间的可选时间戳。
        **违约**:`performance.now()`.

创建新的`PerformanceMark`条目。一个
`PerformanceMark`是 的子类`PerformanceEntry`谁的
`performanceEntry.entryType`始终`'mark'`，以及其
`performanceEntry.duration`始终`0`.使用性能标记
以标记性能时间轴中的特定重要时刻。

被创造的`PerformanceMark`条目被放入全球性能时间表
并可以查询`performance.getEntries`,
`performance.getEntriesByName`和`performance.getEntriesByType`.当
执行观察，应从全局清除条目
性能时间表手动与`performance.clearMarks`.

### `performance.markResourceTiming(timingInfo, requestedUrl, initiatorType, global, cacheMode)`

<!-- YAML
added: v18.2.0
-->

*   `timingInfo`{对象}[获取计时信息][Fetch Timing Info]
*   `requestedUrl`{字符串}资源网址
*   `initiatorType`{字符串}启动器名称，例如：“fetch”
*   `global`{对象}
*   `cacheMode`{字符串}缓存模式必须是空字符串 （''） 或'local'

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

创建新的`PerformanceResourceTiming`条目。一个
`PerformanceResourceTiming`是 的子类`PerformanceEntry`谁的
`performanceEntry.entryType`始终`'resource'`.性能资源
用于在资源时间轴中标记时刻。

被创造的`PerformanceMark`条目被放入全局资源时间轴中
并可以查询`performance.getEntries`,
`performance.getEntriesByName`和`performance.getEntriesByType`.当
执行观察，应从全局清除条目
性能时间表手动与`performance.clearResourceTimings`.

### `performance.measure(name[, startMarkOrOptions[, endMark]])`

<!-- YAML
added: v8.5.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Updated to conform to the User Timing Level 3 specification.
  - version:
      - v13.13.0
      - v12.16.3
    pr-url: https://github.com/nodejs/node/pull/32651
    description: Make `startMark` and `endMark` parameters optional.
-->

*   `name`{字符串}
*   `startMarkOrOptions`{字符串|对象} 可选。
    *   `detail`{任何}要包含在度量值中的其他可选详细信息。
    *   `duration`{数字}开始时间和结束时间之间的持续时间。
    *   `end`{数字|字符串}用作结束时间的时间戳或字符串
        识别以前记录的标记。
    *   `start`{数字|字符串}用作开始时间的时间戳或字符串
        识别以前记录的标记。
*   `endMark`{字符串}自选。在以下情况下必须省略`startMarkOrOptions`是一个
    {对象}。

创建新的`PerformanceMeasure`条目。一个
`PerformanceMeasure`是 的子类`PerformanceEntry`谁的
`performanceEntry.entryType`始终`'measure'`，以及其
`performanceEntry.duration`测量自
`startMark`和`endMark`.

这`startMark`参数可以标识任何*现存* `PerformanceMark`在
性能时间线，或*五月*标识任何时间戳属性
由`PerformanceNodeTiming`类。如果`startMark`做
不存在，则引发错误。

可选`endMark`参数必须标识任何*现存* `PerformanceMark`
在性能时间轴或
`PerformanceNodeTiming`类。`endMark`将是`performance.now()`
如果未传递任何参数，否则如果`endMark`不存在，
将引发错误。

被创造的`PerformanceMeasure`条目被放入全球性能时间表
并可以查询`performance.getEntries`,
`performance.getEntriesByName`和`performance.getEntriesByType`.当
执行观察，应从全局清除条目
性能时间表手动与`performance.clearMeasures`.

### `performance.nodeTiming`

<!-- YAML
added: v8.5.0
-->

*   {性能节点优化}

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

的实例`PerformanceNodeTiming`提供性能的类
特定节点.js操作里程碑的指标。

### `performance.now()`

<!-- YAML
added: v8.5.0
-->

*   返回值：{数字}

返回当前高分辨率毫秒时间戳，其中 0 表示
电流的开始`node`过程。

### `performance.timeOrigin`

<!-- YAML
added: v8.5.0
-->

*   {数字}

这[`timeOrigin`][timeOrigin]指定高分辨率毫秒时间戳
其中电流`node`过程开始，以Unix时间测量。

### `performance.timerify(fn[, options])`

<!-- YAML
added: v8.5.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37475
    description: Added the histogram option.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Re-implemented to use pure-JavaScript and the ability
                 to time async functions.
-->

*   `fn`{函数}
*   `options`{对象}
    *   `histogram`{可记录的历史记录}使用 创建的直方图对象
        `perf_hooks.createHistogram()`将记录运行时持续时间
        纳 秒。

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

将函数包装在一个新函数中，该函数测量
包装函数。一个`PerformanceObserver`必须订阅`'function'`
事件类型，以便访问计时详细信息。

```js
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

function someFunction() {
  console.log('hello world');
}

const wrapped = performance.timerify(someFunction);

const obs = new PerformanceObserver((list) => {
  console.log(list.getEntries()[0].duration);

  performance.clearMarks();
  performance.clearMeasures();
  obs.disconnect();
});
obs.observe({ entryTypes: ['function'] });

// A performance timeline entry will be created
wrapped();
```

如果包装的函数返回一个 promise，则将附加一个 finally 处理程序
到承诺，一旦最终处理程序
调用。

### `performance.toJSON()`

<!-- YAML
added: v16.1.0
-->

一个对象，它是`performance`对象。它
类似于[`window.performance.toJSON`][window.performance.toJSON]在浏览器中。

## 类：`PerformanceEntry`

<!-- YAML
added: v8.5.0
-->

### `performanceEntry.detail`

<!-- YAML
added: v16.0.0
-->

*   {任何}

特定于`entryType`.

### `performanceEntry.duration`

<!-- YAML
added: v8.5.0
-->

*   {数字}

此条目经过的总毫秒数。此值不会
对所有性能条目类型都有意义。

### `performanceEntry.entryType`

<!-- YAML
added: v8.5.0
-->

*   {字符串}

性能条目的类型。它可能是以下情况之一：

*   `'node'`（仅限节点.js）
*   `'mark'`（可在网站上找到）
*   `'measure'`（可在网站上找到）
*   `'gc'`（仅限节点.js）
*   `'function'`（仅限节点.js）
*   `'http2'`（仅限节点.js）
*   `'http'`（仅限节点.js）

### `performanceEntry.flags`

<!-- YAML
added:
 - v13.9.0
 - v12.17.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Runtime deprecated. Now moved to the detail property
                 when entryType is 'gc'.
-->

*   {数字}

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

什么时候`performanceEntry.entryType`等于`'gc'`这`performance.flags`
属性包含有关垃圾回收操作的其他信息。
该值可以是以下值之一：

*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_NO`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_CONSTRUCT_RETAINED`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_FORCED`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_SYNCHRONOUS_PHANTOM_PROCESSING`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_ALL_AVAILABLE_GARBAGE`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_ALL_EXTERNAL_MEMORY`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_SCHEDULE_IDLE`

### `performanceEntry.name`

<!-- YAML
added: v8.5.0
-->

*   {字符串}

性能条目的名称。

### `performanceEntry.kind`

<!-- YAML
added: v8.5.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Runtime deprecated. Now moved to the detail property
                 when entryType is 'gc'.
-->

*   {数字}

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

什么时候`performanceEntry.entryType`等于`'gc'`这`performance.kind`
属性标识已发生的垃圾回收操作的类型。
该值可以是以下值之一：

*   `perf_hooks.constants.NODE_PERFORMANCE_GC_MAJOR`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_MINOR`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_INCREMENTAL`
*   `perf_hooks.constants.NODE_PERFORMANCE_GC_WEAKCB`

### `performanceEntry.startTime`

<!-- YAML
added: v8.5.0
-->

*   {数字}

高分辨率毫秒时间戳标记的开始时间
性能条目。

### 垃圾回收 （“gc”） 详细信息

什么时候`performanceEntry.type`等于`'gc'`这`performanceEntry.detail`
属性将是具有两个属性的 {对象}：

*   `kind`{数字}其中之一：
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_MAJOR`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_MINOR`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_INCREMENTAL`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_WEAKCB`
*   `flags`{数字}其中之一：
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_NO`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_CONSTRUCT_RETAINED`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_FORCED`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_SYNCHRONOUS_PHANTOM_PROCESSING`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_ALL_AVAILABLE_GARBAGE`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_ALL_EXTERNAL_MEMORY`
    *   `perf_hooks.constants.NODE_PERFORMANCE_GC_FLAGS_SCHEDULE_IDLE`

### HTTP （'http'） Details

什么时候`performanceEntry.type`等于`'http'`这`performanceEntry.detail`
属性将是包含附加信息的 {对象}。

如果`performanceEntry.name`等于`HttpClient`这`detail`
将包含以下属性：`req`,`res`.而`req`财产
将是一个 {对象} 包含`method`,`url`,`headers`这`res`财产
将是一个 {对象} 包含`statusCode`,`statusMessage`,`headers`.

如果`performanceEntry.name`等于`HttpRequest`这`detail`
将包含以下属性：`req`,`res`.而`req`财产
将是一个 {对象} 包含`method`,`url`,`headers`这`res`财产
将是一个 {对象} 包含`statusCode`,`statusMessage`,`headers`.

这可能会增加额外的内存开销，并且只应用于
出于诊断目的，默认情况下不会在生产环境中保持打开状态。

### HTTP/2 （'http2'） Details

什么时候`performanceEntry.type`等于`'http2'`这
`performanceEntry.detail`属性将是一个 {对象} 包含
其他性能信息。

如果`performanceEntry.name`等于`Http2Stream`这`detail`
将包含以下属性：

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

如果`performanceEntry.name`等于`Http2Session`这`detail`将
包含以下属性：

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

### 定时（“函数”）详细信息

什么时候`performanceEntry.type`等于`'function'`这
`performanceEntry.detail`属性将是 {Array} 列表
时标函数的输入参数。

### 净（“净”）详细信息

什么时候`performanceEntry.type`等于`'net'`这
`performanceEntry.detail`属性将是一个 {对象} 包含
其他信息。

如果`performanceEntry.name`等于`connect`这`detail`
将包含以下属性：`host`,`port`.

### DNS （'dns'） Details

什么时候`performanceEntry.type`等于`'dns'`这
`performanceEntry.detail`属性将是一个 {对象} 包含
其他信息。

如果`performanceEntry.name`等于`lookup`这`detail`
将包含以下属性：`hostname`,`family`,`hints`,`verbatim`,
`addresses`.

如果`performanceEntry.name`等于`lookupService`这`detail`将
包含以下属性：`host`,`port`,`hostname`,`service`.

如果`performanceEntry.name`等于`queryxxx`或`getHostByAddr`这`detail`将
包含以下属性：`host`,`ttl`,`result`.的价值`result`是
与结果相同`queryxxx`或`getHostByAddr`.

## 类：`PerformanceNodeTiming`

<!-- YAML
added: v8.5.0
-->

*   扩展：{PerformanceEntry}

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

提供节点本身.js计时详细信息。此类的构造函数
不向用户公开。

### `performanceNodeTiming.bootstrapComplete`

<!-- YAML
added: v8.5.0
-->

*   {数字}

节点.js处理的高分辨率毫秒时间戳
完成引导。如果引导尚未完成，则属性
值为 -1。

### `performanceNodeTiming.environment`

<!-- YAML
added: v8.5.0
-->

*   {数字}

节点.js环境所在的高分辨率毫秒时间戳
初始 化。

### `performanceNodeTiming.idleTime`

<!-- YAML
added:
  - v14.10.0
  - v12.19.0
-->

*   {数字}

事件循环时间量的高分辨率毫秒时间戳
在事件循环的事件提供程序中一直处于空闲状态（例如`epoll_wait`).这
不考虑 CPU 使用率。如果事件循环尚未
已启动（例如，在主脚本的第一个刻度线中），该属性具有
值为 0。

### `performanceNodeTiming.loopExit`

<!-- YAML
added: v8.5.0
-->

*   {数字}

节点.js事件循环的高分辨率毫秒时间戳
退出。如果事件循环尚未退出，则该属性的值为 -1。
它只能在 的处理程序中具有不为 -1 的值[`'exit'`]['exit']事件。

### `performanceNodeTiming.loopStart`

<!-- YAML
added: v8.5.0
-->

*   {数字}

节点.js事件循环的高分辨率毫秒时间戳
开始。如果事件循环尚未开始（例如，在
main 脚本），则该属性的值为 -1。

### `performanceNodeTiming.nodeStart`

<!-- YAML
added: v8.5.0
-->

*   {数字}

节点进程的高分辨率毫秒时间戳.js
初始 化。

### `performanceNodeTiming.v8Start`

<!-- YAML
added: v8.5.0
-->

*   {数字}

V8 平台所在的高分辨率毫秒时间戳
初始 化。

## 类：`PerformanceResourceTiming`

<!-- YAML
added: v18.2.0
-->

*   扩展：{PerformanceEntry}

提供有关加载应用程序的详细网络计时数据
资源。

此类的构造函数不直接向用户公开。

### `performanceResourceTiming.workerStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

调度前的高分辨率毫秒时间戳
这`fetch`请求。如果资源未被辅助角色截获，则该属性
将始终返回 0。

### `performanceResourceTiming.redirectStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

表示开始时间的高分辨率毫秒时间戳
启动重定向的抓取。

### `performanceResourceTiming.redirectEnd`

<!-- YAML
added: v18.2.0
-->

*   {数字}

将在紧接其后创建的高分辨率毫秒时间戳
接收上次重定向响应的最后一个字节。

### `performanceResourceTiming.fetchStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

紧挨着节点.js开始前的高分辨率毫秒时间戳
以获取资源。

### `performanceResourceTiming.domainLookupStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

紧挨着节点.js开始前的高分辨率毫秒时间戳
资源的域名查找。

### `performanceResourceTiming.domainLookupEnd`

<!-- YAML
added: v18.2.0
-->

*   {数字}

高分辨率毫秒时间戳，表示立即的时间
在节点之后.js完成对资源的域名查找。

### `performanceResourceTiming.connectStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

高分辨率毫秒时间戳，表示立即的时间
在 Node.js 开始建立与服务器以检索的连接之前
资源。

### `performanceResourceTiming.connectEnd`

<!-- YAML
added: v18.2.0
-->

*   {数字}

高分辨率毫秒时间戳，表示立即的时间
在 Node.js 完成与服务器建立连接以检索
资源。

### `performanceResourceTiming.secureConnectionStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

高分辨率毫秒时间戳，表示立即的时间
在 Node.js启动握手过程以保护当前连接之前。

### `performanceResourceTiming.requestStart`

<!-- YAML
added: v18.2.0
-->

*   {数字}

高分辨率毫秒时间戳，表示立即的时间
在 Node.js 之前，从服务器接收响应的第一个字节。

### `performanceResourceTiming.responseEnd`

<!-- YAML
added: v18.2.0
-->

*   {数字}

高分辨率毫秒时间戳，表示立即的时间
在 Node 之后.js接收资源的最后一个字节或紧挨着之前
传输连接已关闭，以先到者为准。

### `performanceResourceTiming.transferSize`

<!-- YAML
added: v18.2.0
-->

*   {数字}

表示提取资源的大小（以八位字节为单位）的数字。尺寸
包括响应标头字段以及响应负载正文。

### `performanceResourceTiming.encodedBodySize`

<!-- YAML
added: v18.2.0
-->

*   {数字}

表示从读取接收的大小（以八位字节为单位）的数字
（HTTP 或缓存），在删除任何应用之前，有效负载正文
内容编码。

### `performanceResourceTiming.decodedBodySize`

<!-- YAML
added: v18.2.0
-->

*   {数字}

表示从读取接收的大小（以八位字节为单位）的数字
（HTTP 或缓存），在删除任何应用的消息正文后
内容编码。

### `performanceResourceTiming.toJSON()`

<!-- YAML
added: v18.2.0
-->

返回`object`这是
`PerformanceResourceTiming`对象

## 类：`perf_hooks.PerformanceObserver`

### `new PerformanceObserver(callback)`

<!-- YAML
added: v8.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `callback`{函数}
    *   `list`{PerformanceObserverEntryList}
    *   `observer`{PerformanceObserver}

`PerformanceObserver`对象在新的
`PerformanceEntry`实例已添加到性能时间轴。

```js
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

const obs = new PerformanceObserver((list, observer) => {
  console.log(list.getEntries());

  performance.clearMarks();
  performance.clearMeasures();
  observer.disconnect();
});
obs.observe({ entryTypes: ['mark'], buffered: true });

performance.mark('test');
```

因为`PerformanceObserver`实例引入自己的附加
性能开销，不应让实例订阅通知
无限期。用户应在观察者没有时立即断开连接
需要更长的时间。

这`callback`在以下情况下调用`PerformanceObserver`是
通知有关新`PerformanceEntry`实例。回调接收
`PerformanceObserverEntryList`实例和对
`PerformanceObserver`.

### `performanceObserver.disconnect()`

<!-- YAML
added: v8.5.0
-->

断开`PerformanceObserver`实例来自所有通知。

### `performanceObserver.observe(options)`

<!-- YAML
added: v8.5.0
changes:
  - version: v16.7.0
    pr-url: https://github.com/nodejs/node/pull/39297
    description: Updated to conform to Performance Timeline Level 2. The
                 buffered option has been added back.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37136
    description: Updated to conform to User Timing Level 3. The
                 buffered option has been removed.
-->

*   `options`{对象}
    *   `type`{字符串}单个 {PerformanceEntry} 类型。不得给予
        如果`entryTypes`已指定。
    *   `entryTypes`{字符串\[]}标识
        {PerformanceEntry} 观察者感兴趣的实例。如果不是
        前提是将引发错误。
    *   `buffered`{布尔值}如果为 true，则使用
        全局列表`PerformanceEntry`缓冲条目。如果为假，则仅
        `PerformanceEntry`s 在将时间点发送到
        观察者回调。**违约：** `false`.

将 {PerformanceObserver} 实例订阅到 new 的通知
{PerformanceEntry} 由`options.entryTypes`
或`options.type`:

```js
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

const obs = new PerformanceObserver((list, observer) => {
  // Called once asynchronously. `list` contains three items.
});
obs.observe({ type: 'mark' });

for (let n = 0; n < 3; n++)
  performance.mark(`test${n}`);
```

## 类：`PerformanceObserverEntryList`

<!-- YAML
added: v8.5.0
-->

这`PerformanceObserverEntryList`类用于提供对
`PerformanceEntry`传递给`PerformanceObserver`.
此类的构造函数不向用户公开。

### `performanceObserverEntryList.getEntries()`

<!-- YAML
added: v8.5.0
-->

*   返回： {PerformanceEntry\[]}

返回`PerformanceEntry`按时间顺序排列的对象
关于`performanceEntry.startTime`.

```js
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

const obs = new PerformanceObserver((perfObserverList, observer) => {
  console.log(perfObserverList.getEntries());
  /**
   * [
   *   PerformanceEntry {
   *     name: 'test',
   *     entryType: 'mark',
   *     startTime: 81.465639,
   *     duration: 0
   *   },
   *   PerformanceEntry {
   *     name: 'meow',
   *     entryType: 'mark',
   *     startTime: 81.860064,
   *     duration: 0
   *   }
   * ]
   */

  performance.clearMarks();
  performance.clearMeasures();
  observer.disconnect();
});
obs.observe({ type: 'mark' });

performance.mark('test');
performance.mark('meow');
```

### `performanceObserverEntryList.getEntriesByName(name[, type])`

<!-- YAML
added: v8.5.0
-->

*   `name`{字符串}
*   `type`{字符串}
*   返回： {PerformanceEntry\[]}

返回`PerformanceEntry`按时间顺序排列的对象
关于`performanceEntry.startTime`谁的`performanceEntry.name`是
等于`name`，以及（可选）其`performanceEntry.entryType`等于
`type`.

```js
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

const obs = new PerformanceObserver((perfObserverList, observer) => {
  console.log(perfObserverList.getEntriesByName('meow'));
  /**
   * [
   *   PerformanceEntry {
   *     name: 'meow',
   *     entryType: 'mark',
   *     startTime: 98.545991,
   *     duration: 0
   *   }
   * ]
   */
  console.log(perfObserverList.getEntriesByName('nope')); // []

  console.log(perfObserverList.getEntriesByName('test', 'mark'));
  /**
   * [
   *   PerformanceEntry {
   *     name: 'test',
   *     entryType: 'mark',
   *     startTime: 63.518931,
   *     duration: 0
   *   }
   * ]
   */
  console.log(perfObserverList.getEntriesByName('test', 'measure')); // []

  performance.clearMarks();
  performance.clearMeasures();
  observer.disconnect();
});
obs.observe({ entryTypes: ['mark', 'measure'] });

performance.mark('test');
performance.mark('meow');
```

### `performanceObserverEntryList.getEntriesByType(type)`

<!-- YAML
added: v8.5.0
-->

*   `type`{字符串}
*   返回： {PerformanceEntry\[]}

返回`PerformanceEntry`按时间顺序排列的对象
关于`performanceEntry.startTime`谁的`performanceEntry.entryType`
等于`type`.

```js
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

const obs = new PerformanceObserver((perfObserverList, observer) => {
  console.log(perfObserverList.getEntriesByType('mark'));
  /**
   * [
   *   PerformanceEntry {
   *     name: 'test',
   *     entryType: 'mark',
   *     startTime: 55.897834,
   *     duration: 0
   *   },
   *   PerformanceEntry {
   *     name: 'meow',
   *     entryType: 'mark',
   *     startTime: 56.350146,
   *     duration: 0
   *   }
   * ]
   */
  performance.clearMarks();
  performance.clearMeasures();
  observer.disconnect();
});
obs.observe({ type: 'mark' });

performance.mark('test');
performance.mark('meow');
```

## `perf_hooks.createHistogram([options])`

<!-- YAML
added:
  - v15.9.0
  - v14.18.0
-->

*   `options`{对象}
    *   `lowest`{数字|比金}最低的可识别值。必须是整数
        值大于 0。**违约：** `1`.
    *   `highest`{数字|比金}最高可记录值。必须是整数
        等于或大于两倍的值`lowest`.
        **违约：** `Number.MAX_SAFE_INTEGER`.
    *   `figures`{数字}精度位数。必须是介于
        `1`和`5`.**违约：** `3`.
*   返回 {RecordableHistogram}

返回 {RecordableHistogram}。

## `perf_hooks.monitorEventLoopDelay([options])`

<!-- YAML
added: v11.10.0
-->

*   `options`{对象}
    *   `resolution`{数字}采样率（以毫秒为单位）。必须更大
        比零。**违约：** `10`.
*   返回：{间隔时间轴}

*此属性是 Node.js 的扩展。它在 Web 浏览器中不可用。*

创建`IntervalHistogram`对事件循环进行采样和报告的对象
随时间推移而延迟。延迟将以纳秒为单位进行报告。

使用计时器检测近似事件循环延迟的工作原理，因为
计时器的执行与 libuv 的生命周期密切相关
事件循环。也就是说，循环中的延迟将导致执行延迟
的计时器，这些延迟是此 API 的具体目的
检测。

```js
const { monitorEventLoopDelay } = require('node:perf_hooks');
const h = monitorEventLoopDelay({ resolution: 20 });
h.enable();
// Do something.
h.disable();
console.log(h.min);
console.log(h.max);
console.log(h.mean);
console.log(h.stddev);
console.log(h.percentiles);
console.log(h.percentile(50));
console.log(h.percentile(99));
```

## 类：`Histogram`

<!-- YAML
added: v11.10.0
-->

### `histogram.count`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   {数字}

直方图记录的样本数。

### `histogram.countBigInt`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   {bigint}

直方图记录的样本数。

### `histogram.exceeds`

<!-- YAML
added: v11.10.0
-->

*   {数字}

事件循环延迟超过最大 1 小时事件的次数
环路延迟阈值。

### `histogram.exceedsBigInt`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   {bigint}

事件循环延迟超过最大 1 小时事件的次数
环路延迟阈值。

### `histogram.max`

<!-- YAML
added: v11.10.0
-->

*   {数字}

记录的最大事件循环延迟。

### `histogram.maxBigInt`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   {bigint}

记录的最大事件循环延迟。

### `histogram.mean`

<!-- YAML
added: v11.10.0
-->

*   {数字}

记录的事件循环延迟的平均值。

### `histogram.min`

<!-- YAML
added: v11.10.0
-->

*   {数字}

记录的最小事件循环延迟。

### `histogram.minBigInt`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   {bigint}

记录的最小事件循环延迟。

### `histogram.percentile(percentile)`

<!-- YAML
added: v11.10.0
-->

*   `percentile`{数字}范围 （0， 100] 中的百分位值。
*   返回值：{数字}

返回给定百分位数处的值。

### `histogram.percentileBigInt(percentile)`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   `percentile`{数字}范围 （0， 100] 中的百分位值。
*   返回：{bigint}

返回给定百分位数处的值。

### `histogram.percentiles`

<!-- YAML
added: v11.10.0
-->

*   {地图}

返回`Map`详细描述累积百分位数分布的对象。

### `histogram.percentilesBigInt`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   {地图}

返回`Map`详细描述累积百分位数分布的对象。

### `histogram.reset()`

<!-- YAML
added: v11.10.0
-->

重置收集的直方图数据。

### `histogram.stddev`

<!-- YAML
added: v11.10.0
-->

*   {数字}

记录的事件循环延迟的标准偏差。

## 类：`IntervalHistogram extends Histogram`

一个`Histogram`在给定的时间间隔内定期更新。

### `histogram.disable()`

<!-- YAML
added: v11.10.0
-->

*   返回：{布尔值}

禁用更新间隔计时器。返回`true`如果计时器是
停止`false`如果它已经停止。

### `histogram.enable()`

<!-- YAML
added: v11.10.0
-->

*   返回：{布尔值}

启用更新间隔计时器。返回`true`如果计时器是
开始`false`如果它已经启动。

### 克隆`IntervalHistogram`

{IntervalHistogram} 实例可以通过 {MessagePort} 进行克隆。在接收
最后，直方图被克隆为一个普通的{直方图}对象，它没有
实现`enable()`和`disable()`方法。

## 类：`RecordableHistogram extends Histogram`

<!-- YAML
added:
  - v15.9.0
  - v14.18.0
-->

### `histogram.add(other)`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

*   `other`{可记录的历史记录}

将值从`other`到此直方图。

### `histogram.record(val)`

<!-- YAML
added:
  - v15.9.0
  - v14.18.0
-->

*   `val`{数字|比金}要在直方图中记录的金额。

### `histogram.recordDelta()`

<!-- YAML
added:
  - v15.9.0
  - v14.18.0
-->

计算自
上一次调用`recordDelta()`并在直方图中记录该金额。

## 例子

### 测量异步操作的持续时间

下面的示例使用[异步挂钩][Async Hooks]和要测量的性能 API
超时操作的实际持续时间（包括所花费的时间
以执行回调）。

```js
'use strict';
const async_hooks = require('node:async_hooks');
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');

const set = new Set();
const hook = async_hooks.createHook({
  init(id, type) {
    if (type === 'Timeout') {
      performance.mark(`Timeout-${id}-Init`);
      set.add(id);
    }
  },
  destroy(id) {
    if (set.has(id)) {
      set.delete(id);
      performance.mark(`Timeout-${id}-Destroy`);
      performance.measure(`Timeout-${id}`,
                          `Timeout-${id}-Init`,
                          `Timeout-${id}-Destroy`);
    }
  }
});
hook.enable();

const obs = new PerformanceObserver((list, observer) => {
  console.log(list.getEntries()[0]);
  performance.clearMarks();
  performance.clearMeasures();
  observer.disconnect();
});
obs.observe({ entryTypes: ['measure'], buffered: true });

setTimeout(() => {}, 1000);
```

### 测量加载依赖项所需的时间

以下示例测量`require()`要加载的操作
依赖：

<!-- eslint-disable no-global-assign -->

```js
'use strict';
const {
  performance,
  PerformanceObserver
} = require('node:perf_hooks');
const mod = require('node:module');

// Monkey patch the require function
mod.Module.prototype.require =
  performance.timerify(mod.Module.prototype.require);
require = performance.timerify(require);

// Activate the observer
const obs = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  entries.forEach((entry) => {
    console.log(`require('${entry[0]}')`, entry.duration);
  });
  performance.clearMarks();
  performance.clearMeasures();
  obs.disconnect();
});
obs.observe({ entryTypes: ['function'], buffered: true });

require('some-module');
```

### 测量一次 HTTP 往返需要多长时间

以下示例用于跟踪 HTTP 客户端花费的时间
(`OutgoingMessage`） 和 HTTP 请求 （`IncomingMessage`).对于 HTTP 客户端，
它表示启动请求和接收请求之间的时间间隔
响应，对于HTTP请求，它表示接收之间的时间间隔
请求并发送响应：

```js
'use strict';
const { PerformanceObserver } = require('node:perf_hooks');
const http = require('node:http');

const obs = new PerformanceObserver((items) => {
  items.getEntries().forEach((item) => {
    console.log(item);
  });
});

obs.observe({ entryTypes: ['http'] });

const PORT = 8080;

http.createServer((req, res) => {
  res.end('ok');
}).listen(PORT, () => {
  http.get(`http://127.0.0.1:${PORT}`);
});
```

### 测量多长时间`net.connect`（仅适用于 TCP）在连接成功时采用

```js
'use strict';
const { PerformanceObserver } = require('node:perf_hooks');
const net = require('node:net');
const obs = new PerformanceObserver((items) => {
  items.getEntries().forEach((item) => {
    console.log(item);
  });
});
obs.observe({ entryTypes: ['net'] });
const PORT = 8080;
net.createServer((socket) => {
  socket.destroy();
}).listen(PORT, () => {
  net.connect(PORT);
});
```

### 测量请求成功时 DNS 需要多长时间

```js
'use strict';
const { PerformanceObserver } = require('node:perf_hooks');
const dns = require('node:dns');
const obs = new PerformanceObserver((items) => {
  items.getEntries().forEach((item) => {
    console.log(item);
  });
});
obs.observe({ entryTypes: ['dns'] });
dns.lookup('localhost', () => {});
dns.promises.resolve('localhost');
```

[Async Hooks]: async_hooks.md

[Fetch Timing Info]: https://fetch.spec.whatwg.org/#fetch-timing-info

[High Resolution Time]: https://www.w3.org/TR/hr-time-2

[Performance Timeline]: https://w3c.github.io/performance-timeline/

[Resource Timing]: https://www.w3.org/TR/resource-timing-2/

[User Timing]: https://www.w3.org/TR/user-timing/

[Web Performance APIs]: https://w3c.github.io/perf-timing-primer/

[Worker threads]: worker_threads.md#worker-threads

[`'exit'`]: process.md#event-exit

[`child_process.spawnSync()`]: child_process.md#child_processspawnsynccommand-args-options

[`process.hrtime()`]: process.md#processhrtimetime

[`timeOrigin`]: https://w3c.github.io/hr-time/#dom-performance-timeorigin

[`window.performance.toJSON`]: https://developer.mozilla.org/en-US/docs/Web/API/Performance/toJSON

[`window.performance`]: https://developer.mozilla.org/en-US/docs/Web/API/Window/performance
