# Command-line API

<!--introduced_in=v5.9.1-->

<!--type=misc-->

Node.js 带有多种 CLI 选项。这些选项公开了内置调试、执行脚本的多种方式以及其他有用的运行时选项.

要在终端中将此文档作为手册页查看，请运行“man node”.

## Synopsis

`node [options] [V8 options] [<program-entry-point> | -e "script" | -] [--] [arguments]`

`node inspect [<program-entry-point> | -e "script" | <host>:<port>] …`

`node --v8-options`

不带参数执行以启动 [REPL][].

有关 `node inspect` 的更多信息，请参阅 [debugger][] 文档.

## Program entry point

程序入口点是一个类似说明符的字符串。如果字符串不是绝对路径，则将其解析为当前工作目录的相对路径。然后由 [CommonJS][] 模块加载器解析该路径。如果没有找到对应的文件，则抛出错误.

如果找到一个文件，它的路径将在以下任何条件下传递给[ECMAScript module loader][]:

* 该程序以命令行标志启动，该标志强制使用 ECMAScript 模块加载器加载入口点.
* 该文件具有 `.mjs` 扩展名.
* 该文件没有 `.cjs` 扩展名，并且最近的父 `package.json` 文件包含顶级 [`"type"`][] 字段，其值为 `"module"`.

否则，使用 CommonJS 模块加载器加载文件。请参阅 [模块加载器][] 了解更多详细信息.

### ECMAScript modules loader entry point caveat

当加载 [ECMAScript module loader][] 加载程序入口点时，`node` 命令将仅接受具有 `.js`、`.mjs` 或 `.cjs` 扩展名的文件作为输入；并在启用 [`--experimental-wasm-modules`][] 时使用 `.wasm` 扩展.

## Options

<!-- YAML
changes:
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/23020
    description: Underscores instead of dashes are now allowed for
                 Node.js options as well, in addition to V8 options.
-->

所有选项，包括 V8 选项，都允许用破折号 (`-`) 或下划线 (`_`) 分隔单词。例如，`--pending-deprecation` 等价于 `--pending_deprecation`.

如果采用单个值的选项（例如 `--max-http-header-size`）多次传递，则使用最后传递的值。命令行中的选项优先于通过 [`NODE_OPTIONS`][] 环境变量传递的选项.

### `-`

<!-- YAML
added: v8.0.0
-->

标准输入的别名。类似于在其他命令行实用程序中使用 `-`，这意味着脚本是从标准输入读取的，其余选项传递给该脚本.

### `--`

<!-- YAML
added: v6.11.0
-->

指示节点选项的结束。将其余参数传递给脚本.
如果在此之前没有提供脚本文件名或 eval/print 脚本，则下一个参数用作脚本文件名.

### `--abort-on-uncaught-exception`

<!-- YAML
added: v0.10.8
-->

中止而不是退出会导致生成核心文件以使用调试器进行事后分析（例如 `lldb`、`gdb` 和 `mdb`）.

如果传递了这个标志，行为仍然可以通过 [`process.setUncaughtExceptionCaptureCallback()`][] 设置为不中止（并通过使用使用它的 `node:domain` 模块）.

### `--build-snapshot`

<!-- YAML
added: REPLACEME
-->

> Stability: 1 - Experimental

Generates a snapshot blob when the process exits and writes it to disk, which can be loaded later with `--snapshot-blob`.

构建快照时，如果不指定 --snapshot-blob ，则默认将生成的 blob 写入当前工作目录的 snapshot.blob 中。否则会被写入`--snapshot-blob`指定的路径.

```console
$ echo "globalThis.foo = 'I am from the snapshot'" > snapshot.js

# Run snapshot.js to intialize the application and snapshot the
# state of it into snapshot.blob.
$ node --snapshot-blob snapshot.blob --build-snapshot snapshot.js

$ echo "console.log(globalThis.foo)" > index.js

# Load the generated snapshot and start the application from index.js.
$ node --snapshot-blob snapshot.blob index.js
I am from the snapshot
```

[`v8.startupSnapshot` API][] 可用于在快照构建时指定入口点，从而避免在反序列化时需要额外的入口脚本:

```console
$ echo "require('v8').startupSnapshot.setDeserializeMainFunction(() => console.log('I am from the snapshot'))" > snapshot.js
$ node --snapshot-blob snapshot.blob --build-snapshot snapshot.js
$ node --snapshot-blob snapshot.blob
I am from the snapshot
```

有关更多信息，请查看 [`v8.startupSnapshot` API][] 文档.

目前对运行时快照的支持是实验性的:

1. 快照尚不支持用户级模块，因此只能对一个文件进行快照。用户可以在构建快照之前使用他们选择的捆绑器将他们的应用程序捆绑到一个脚本中，但是.
2. 只有一部分内置模块在快照中工作，尽管 Node.js 核心测试套件会检查一些相当复杂的应用程序是否可以被快照。正在添加对更多模块的支持。如果在构建快照时发生任何崩溃或错误行为，请在 [Node.js 问题跟踪器][] 中提交报告，并在 [用户空间快照的跟踪问题][] 中链接到它.

### `--completion-bash`

<!-- YAML
added: v10.12.0
-->

为 Node.js 打印可使用源代码的 bash 完成脚本.

```console
$ node --completion-bash > node_bash_completion
$ source node_bash_completion
```

### `-C=condition`, `--conditions=condition`

<!-- YAML
added:
  - v14.9.0
  - v12.19.0
-->

> Stability: 1 - Experimental

启用对自定义 [条件导出][] 解析条件的实验性支持.

允许任意数量的自定义字符串条件名称.

`"node"`、`"default"`、`"import"` 和 `"require"` 的默认 Node.js 条件将始终按照定义应用.

For example, to run a module with "development" resolutions:

```console
$ node -C=development app.js
```

### `--cpu-prof`

<!-- YAML
added: v12.0.0
-->

> Stability: 1 - Experimental

Starts the V8 CPU profiler on start up, and writes the CPU profile to disk before exit.

如果 `--cpu-prof-dir` 未指定，则生成的配置文件放在当前工作目录中.

如果 `--cpu-prof-name` 未指定，则生成的配置文件名为 `CPU.${yyyymmdd}.${hhmmss}.${pid}.${tid}.${seq}.cpuprofile`.

```console
$ node --cpu-prof index.js
$ ls *.cpuprofile
CPU.20190409.202950.15293.0.0.cpuprofile
```

### `--cpu-prof-dir`

<!-- YAML
added: v12.0.0
-->

> Stability: 1 - Experimental

指定由`--cpu-prof`生成的CPU配置文件将放置的目录.

默认值由 [`--diagnostic-dir`][] 命令行选项控制.

### `--cpu-prof-interval`

<!-- YAML
added: v12.2.0
-->

> Stability: 1 - Experimental

指定由 `--cpu-prof` 生成的 CPU 配置文件的采样间隔（以微秒为单位）。默认值为 1000 微秒.

### `--cpu-prof-name`

<!-- YAML
added: v12.0.0
-->

> Stability: 1 - Experimental

指定 `--cpu-prof` 生成的 CPU profile 的文件名.

### `--diagnostic-dir=directory`

设置所有诊断输出文件的写入目录.
默认为当前工作目录.

影响默认输出目录:

* [`--cpu-prof-dir`][]
* [`--heap-prof-dir`][]
* [`--redirect-warnings`][]

### `--disable-proto=mode`

<!-- YAML
added:
 - v13.12.0
 - v12.17.0
-->

禁用 `Object.prototype.__proto__` 属性。如果 `mode` 是 `delete`，则该属性将被完全删除。如果 `mode` 是 `throw`，对属性的访问会引发异常，代码为 `ERR_PROTO_ACCESS`.

### `--disallow-code-generation-from-strings`

<!-- YAML
added: v9.8.0
-->

使从字符串生成代码的内置语言功能（例如 `eval` 和 `new Function`）抛出异常。这不会影响 Node.js `node:vm` 模块.

### `--dns-result-order=order`

<!-- YAML
added:
  - v16.4.0
  - v14.18.0
changes:
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/39987
    description: Changed default value to `verbatim`.
-->

在 [`dns.lookup()`][] 和 [`dnsPromises.lookup()`][] 中设置 `verbatim` 的默认值。该值可能是:

* `ipv4first`: sets default `verbatim` `false`.
* `verbatim`: sets default `verbatim` `true`.

默认值为 `verbatim` 并且 [`dns.setDefaultResultOrder()`][] 的优先级高于 `--dns-result-order`.

### `--enable-fips`

<!-- YAML
added: v6.0.0
-->

在启动时启用符合 FIPS 的加密。 （需要针对兼容 FIPS 的 OpenSSL 构建 Node.js。）

### `--enable-source-maps`

<!-- YAML
added: v12.12.0
changes:
  - version:
      - v15.11.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37362
    description: This API is no longer experimental.
-->

启用对堆栈跟踪的 [Source Map v3][Source Map] 支持.

使用转译器（例如 TypeScript）时，应用程序抛出的堆栈跟踪引用转译后的代码，而不是原始源位置.
`--enable-source-maps` 启用源映射缓存，并尽最大努力报告相对于原始源文件的堆栈跟踪.

覆盖 `Error.prepareStackTrace` 防止 `--enable-source-maps` 修改堆栈跟踪.

请注意，启用源映射可能会在访问“Error.stack”时给您的应用程序带来延迟。如果您在应用程序中频繁访问 `Error.stack`，请考虑 `--enable-source-maps` 的性能影响.

### `--experimental-global-customevent`

<!-- YAML
added: v18.7.0
-->

在全局范围内公开 [CustomEvent Web API][].

### `--experimental-global-webcrypto`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

在全局范围内公开 [Web Crypto API][].

### `--experimental-import-meta-resolve`

<!-- YAML
added:
  - v13.9.0
  - v12.16.2
-->

启用实验性的“import.meta.resolve()”支持.

### `--experimental-loader=module`

<!-- YAML
added: v8.8.0
changes:
  - version: v12.11.1
    pr-url: https://github.com/nodejs/node/pull/29752
    description: This flag was renamed from `--loader` to
                 `--experimental-loader`.
-->

指定自定义实验的“模块”[ECMAScript 模块加载器][].
`module` 可以是任何被接受为 [`import` 说明符][] 的字符串.

### `--experimental-network-imports`

<!-- YAML
added:
  - v17.6.0
  - v16.15.0
-->

> Stability: 1 - Experimental

在 `import` 说明符中启用对 `https:` 协议的实验性支持.

### `--experimental-policy`

<!-- YAML
added: v11.8.0
-->

使用指定文件作为安全策略.

### `--no-experimental-fetch`

<!-- YAML
added: v18.0.0
-->

禁用对 [Fetch API][] 的实验性支持.

### `--no-experimental-repl-await`

<!-- YAML
added: v16.6.0
-->

使用此标志禁用 REPL 中的顶级等待.

### `--experimental-shadow-realm`

<!-- YAML
added: REPLACEME
-->

使用此标志启用 [ShadowRealm][] 支持.

### `--experimental-specifier-resolution=mode`

<!-- YAML
added:
 - v13.4.0
 - v12.16.0
-->

设置解析 ES 模块说明符的解析算法。有效选项是 `explicit` 和 `node`.

默认值为 `explicit`，需要提供模块的完整路径。 `node` 模式支持可选的文件扩展名和导入具有索引文件的目录的能力.

参见 [customizing ESM specifier resolution][] 示例用法.

### `--experimental-vm-modules`

<!-- YAML
added: v9.6.0
-->

在 `node:vm` 模块中启用实验性 ES 模块支持.

### `--experimental-wasi-unstable-preview1`

<!-- YAML
added:
  - v13.3.0
  - v12.16.0
changes:
  - version: v13.6.0
    pr-url: https://github.com/nodejs/node/pull/30980
    description: changed from `--experimental-wasi-unstable-preview0` to
                 `--experimental-wasi-unstable-preview1`.
-->

启用实验性 WebAssembly 系统接口 (WASI) 支持.

### `--experimental-wasm-modules`

<!-- YAML
added: v12.3.0
-->

启用实验性 WebAssembly 模块支持.

### `--force-context-aware`

<!-- YAML
added: v12.12.0
-->

禁用加载非 [上下文感知][] 的本机插件.

### `--force-fips`

<!-- YAML
added: v6.0.0
-->

在启动时强制使用符合 FIPS 的加密。 （不能从脚本代码中禁用。）（与 `--enable-fips` 的要求相同。）

### `--frozen-intrinsics`

<!-- YAML
added: v11.12.0
-->

> Stability: 1 - Experimental

启用实验性冻结内在函数，如 `Array` 和 `Object`.

仅支持根上下文。不能保证 `globalThis.Array` 确实是默认的内部引用。代码可能会在此标志下中断.

允许添加 polyfill,
[`--require`][] 和 [`--import`][] 都在冻结内在函数之前运行.

### `--force-node-api-uncaught-exceptions-policy`

<!-- YAML
added: v18.3.0
-->

在 Node-API 异步回调上强制执行 `uncaughtException` 事件.

为了防止现有的附加组件使进程崩溃，默认情况下不启用此标志。将来，此标志将默认启用以强制执行正确的行为.

### `--heapsnapshot-near-heap-limit=max_count`

<!-- YAML
added:
  - v15.1.0
  - v14.18.0
-->

> Stability: 1 - Experimental

当 V8 堆使用量接近堆限制时，将 V8 堆快照写入磁盘。 `count` 应该是一个非负整数（在这种情况下 Node.js 将不超过 `max_count` 个快照写入磁盘）.

生成快照时，可能会触发垃圾收集并降低堆使用率。因此，在 Node.js 实例最终耗尽内存之前，可能会将多个快照写入磁盘。可以比较这些堆快照以确定在拍摄连续快照期间分配了哪些对象。不能保证 Node.js 会准确地将 `max_count` 快照写入磁盘，但在 Node.js 实例在 `max_count` 为大于“0”.

生成 V8 快照需要时间和内存（由 V8 堆管理的内存和 V8 堆外的本机内存）。堆越大，它需要的资源就越多。 Node.js 将调整 V8 堆以适应额外的 V8 堆内存开销，并尽量避免用完进程可用的所有内存。当进程使用的内存比系统认为合适的多时，进程可能会被系统突然终止，具体取决于系统配置.

```console
$ node --max-old-space-size=100 --heapsnapshot-near-heap-limit=3 index.js
Wrote snapshot to Heap.20200430.100036.49580.0.001.heapsnapshot
Wrote snapshot to Heap.20200430.100037.49580.0.002.heapsnapshot
Wrote snapshot to Heap.20200430.100038.49580.0.003.heapsnapshot

<--- Last few GCs --->

[49580:0x110000000]     4826 ms: Mark-sweep 130.6 (147.8) -> 130.5 (147.8) MB, 27.4 / 0.0 ms  (average mu = 0.126, current mu = 0.034) allocation failure scavenge might not succeed
[49580:0x110000000]     4845 ms: Mark-sweep 130.6 (147.8) -> 130.6 (147.8) MB, 18.8 / 0.0 ms  (average mu = 0.088, current mu = 0.031) allocation failure scavenge might not succeed


<--- JS stacktrace --->

FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
....
```

### `--heapsnapshot-signal=signal`

<!-- YAML
added: v12.0.0
-->

启用一个信号处理程序，使 Node.js 进程在收到指定信号时写入堆转储。 `signal` 必须是有效的信号名称.
默认禁用.

```console
$ node --heapsnapshot-signal=SIGUSR2 index.js &
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
node         1  5.5  6.1 787252 247004 ?       Ssl  16:43   0:02 node --heapsnapshot-signal=SIGUSR2 index.js
$ kill -USR2 1
$ ls
Heap.20190718.133405.15554.0.001.heapsnapshot
```

### `--heap-prof`

<!-- YAML
added: v12.4.0
-->

> Stability: 1 - Experimental

在启动时启动 V8 堆分析器，并在退出前将堆分析器写入磁盘.

如果 `--heap-prof-dir` 未指定，则生成的配置文件放在当前工作目录中.

如果未指定 `--heap-prof-name`，则生成的配置文件名为 `Heap.${yyyymmdd}.${hhmmss}.${pid}.${tid}.${seq}.heapprofile`.

```console
$ node --heap-prof index.js
$ ls *.heapprofile
Heap.20190409.202950.15293.0.001.heapprofile
```

### `--heap-prof-dir`

<!-- YAML
added: v12.4.0
-->

> Stability: 1 - Experimental

指定将放置由 `--heap-prof` 生成的堆配置文件的目录.

默认值由 [`--diagnostic-dir`][] 命令行选项控制.

### `--heap-prof-interval`

<!-- YAML
added: v12.4.0
-->

> Stability: 1 - Experimental

指定由 `--heap-prof` 生成的堆配置文件的平均采样间隔（以字节为单位）。默认为 512 \* 1024 字节.

### `--heap-prof-name`

<!-- YAML
added: v12.4.0
-->

> Stability: 1 - Experimental

指定 `--heap-prof` 生成的堆配置文件的文件名.

### `--icu-data-dir=file`

<!-- YAML
added: v0.11.15
-->

指定 ICU 数据加载路径。 （覆盖`NODE_ICU_DATA`。）

### `--import=module`

<!-- YAML
added: REPLACEME
-->

> Stability: 1 - Experimental

启动时预加载指定模块.

遵循 [ECMAScript 模块][] 解析规则.
使用 [`--require`][] 加载 [CommonJS 模块][].
使用 `--require` 预加载的模块将在使用 `--import` 预加载的模块之前运行.

### `--input-type=type`

<!-- YAML
added: v12.0.0
-->

这将 Node.js 配置为将字符串输入解释为 CommonJS 或 ES 模块。字符串输入通过 `--eval`、`--print` 或 `STDIN` 输入.

有效值为 `"commonjs"` 和 `"module"`。默认是`"commonjs"`.

REPL 不支持此选项.

### `--inspect-brk[=[host:]port]`

<!-- YAML
added: v7.6.0
-->

在 `host:port` 上激活检查器并在用户脚本开始时中断.
默认 `host:port` 是 `127.0.0.1:9229`.

### `--inspect-port=[host:]port`

<!-- YAML
added: v7.6.0
-->

设置检查器激活时使用的 `host:port`.
通过发送“SIGUSR1”信号激活检查器时很有用.

默认主机是`127.0.0.1`.

有关 `host` 参数用法，请参阅下面的 [安全警告][].

### `--inspect[=[host:]port]`

<!-- YAML
added: v6.3.0
-->

在 `host:port` 上激活检查器。默认为“127.0.0.1:9229”.

V8 检查器集成允许 Chrome DevTools 和 IDE 等工具调试和分析 Node.js 实例。这些工具通过 tcp 端口附加到 Node.js 实例并使用 [Chrome DevTools 协议][] 进行通信.

<!-- Anchor to make sure old links find a target -->

<a id="inspector_security"></a>

#### Warning: binding inspector to a public IP:port combination is insecure

将检查器绑定到具有开放端口的公共 IP（包括 `0.0.0.0`）是不安全的，因为它允许外部主机连接到检查器并执行 [远程代码执行][] 攻击.

如果指定主机，请确保:

* 无法从公共网络访问主机.
* 防火墙不允许端口上不需要的连接.

**更具体地说，如果端口（默认为“9229”）不受防火墙保护，则“--inspect=0.0.0.0”是不安全的.**

有关详细信息，请参阅 [调试安全影响][] 部分.

### `--inspect-publish-uid=stderr,http`

指定inspector web socket url暴露方式.

默认情况下，检查器 websocket url 在 stderr 和 `http://host:port/json/list` 上的 `/json/list` 端点下可用.

### `--insecure-http-parser`

<!-- YAML
added:
 - v13.4.0
 - v12.15.0
 - v10.19.0
-->

使用接受无效 HTTP 标头的不安全 HTTP 解析器。这可能允许与不符合标准的 HTTP 实现的互操作性。它还可能允许请求走私和其他依赖于接受无效标头的 HTTP 攻击。避免使用此选项.

### `--jitless`

<!-- YAML
added: v12.0.0
-->

禁用[可执行内存的运行时分配][jitless]。出于安全原因，在某些平台上可能需要这样做。它还可以减少其他平台上的攻击面，但性能影响可能很严重.

此标志继承自 V8，可能会在上游发生更改。它可能会在非 semver-major 版本中消失.

### `--max-http-header-size=size`

<!-- YAML
added:
 - v11.6.0
 - v10.15.0
changes:
  - version: v13.13.0
    pr-url: https://github.com/nodejs/node/pull/32520
    description: Change maximum default size of HTTP headers from 8 KiB to 16 KiB.
-->

指定 HTTP 标头的最大大小（以字节为单位）。默认为 16 KiB.

### `--napi-modules`

<!-- YAML
added: v7.10.0
-->

此选项是无操作的。它是为了兼容性而保留的.

### `--no-addons`

<!-- YAML
added:
  - v16.10.0
  - v14.19.0
-->

禁用“node-addons”导出条件以及禁用加载本机插件。当指定 `--no-addons` 时，调用 `process.dlopen` 或需要原生 C++ 插件将失败并抛出异常.

### `--no-deprecation`

<!-- YAML
added: v0.8.0
-->

静默弃用警告.

### `--no-extra-info-on-fatal-exception`

<!-- YAML
added: v17.0.0
-->

隐藏导致退出的致命异常的额外信息.

### `--no-force-async-hooks-checks`

<!-- YAML
added: v9.0.0
-->

禁用 `async_hooks` 的运行时检查。当启用 `async_hooks` 时，这些仍将动态启用.

### `--no-global-search-paths`

<!-- YAML
added: v16.10.0
-->

不要从全局路径中搜索模块，例如 `$HOME/.node_modules` 和 `$NODE_PATH`.

### `--no-warnings`

<!-- YAML
added: v6.0.0
-->

静音所有进程警告（包括弃用）.

### `--node-memory-debug`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

在 Node.js 内部启用额外的内存泄漏调试检查。这通常只对调试 Node.js 本身的开发人员有用.

### `--openssl-config=file`

<!-- YAML
added: v6.9.0
-->

在启动时加载 OpenSSL 配置文件。除其他用途外，如果 Node.js 是针对支持 FIPS 的 OpenSSL 构建的，则这可用于启用符合 FIPS 的加密.

### `--openssl-shared-config`

<!-- YAML
added: v18.5.0
-->

启用从 OpenSSL 配置文件中读取的 OpenSSL 默认配置部分“openssl_conf”。默认配置文件名为 `openssl.cnf`，但可以使用环境变量 `OPENSSL_CONF` 或使用命令行选项 `--openssl-config` 进行更改.
默认 OpenSSL 配置文件的位置取决于 OpenSSL 如何链接到 Node.js。共享 OpenSSL 配置可能会产生不必要的影响，建议使用特定于 Node.js 的配置部分，即“nodejs_conf”，并且在不使用此选项时是默认设置.

### `--openssl-legacy-provider`

<!-- YAML
added: v17.0.0
-->

启用 OpenSSL 3.0 旧提供程序。有关详细信息，请参阅 [OSSL\_PROVIDER-legacy][OSSL_PROVIDER-legacy].

### `--pending-deprecation`

<!-- YAML
added: v8.0.0
-->

发出未决弃用警告.

未决弃用通常与运行时弃用相同，但值得注意的例外是它们默认关闭并且不会发出，除非 `--pending-deprecation` 命令行标志或 `NODE_PENDING_DEPRECATION=1` 环境变量，已设置。未决弃用用于提供一种选择性的“早期警告”机制，开发人员可以利用该机制检测弃用的 API 使用情况.

### `--policy-integrity=sri`

<!-- YAML
added: v12.7.0
-->

> Stability: 1 - Experimental

如果策略不具有指定的完整性，则指示 Node.js 在运行任何代码之前出错。它需要一个 [Subresource Integrity][] 字符串作为参数.

### `--preserve-symlinks`

<!-- YAML
added: v6.3.0
-->

指示模块加载器在解析和缓存模块时保留符号链接.

默认情况下，当 Node.js 从符号链接到不同磁盘位置的路径加载模块时，Node.js 将取消引用该链接并使用模块的实际磁盘“真实路径”作为标识符并作为定位其他依赖模块的根路径。在大多数情况下，这种默认行为是可以接受的。但是，当使用符号链接的对等依赖项时，如下例所示，如果 `moduleA` 尝试将 `moduleB` 作为对等依赖项，默认行为会引发异常:

```text
{appDir}
 ├── app
 │   ├── index.js
 │   └── node_modules
 │       ├── moduleA -> {appDir}/moduleA
 │       └── moduleB
 │           ├── index.js
 │           └── package.json
 └── moduleA
     ├── index.js
     └── package.json
```

`--preserve-symlinks` 命令行标志指示 Node.js 使用模块的符号链接路径而不是真实路径，从而允许找到符号链接的对等依赖项.

但是请注意，使用 `--preserve-symlinks` 可能会产生其他副作用.
具体来说，如果符号链接的 _native_ 模块从依赖关系树中的多个位置链接，则可能无法加载（Node.js 会将它们视为两个单独的模块，并会尝试多次加载该模块，从而引发异常).

`--preserve-symlinks` 标志不适用于主模块，它允许 `node --preserve-symlinks node_module/.bin/<foo>` 工作。要对主模块应用相同的行为，还可以使用 `--preserve-symlinks-main`.

### `--preserve-symlinks-main`

<!-- YAML
added: v10.2.0
-->

指示模块加载器在解析和缓存主模块（`require.main`）时保留符号链接.

这个标志的存在是为了让主模块可以选择加入 --preserve-symlinks 给所有其他导入的相同行为；但是，它们是单独的标志，用于向后兼容旧的 Node.js 版本.

`--preserve-symlinks-main` does not imply `--preserve-symlinks`; use
`--preserve-symlinks-main` in addition to
`--preserve-symlinks` when it is not desirable to follow symlinks before
resolving relative paths.

有关更多信息，请参阅`--preserve-symlinks`.

### `--prof`

<!-- YAML
added: v2.0.0
-->

生成 V8 分析器输出.

### `--prof-process`

<!-- YAML
added: v5.2.0
-->

处理使用 V8 选项 `--prof` 生成的 V8 分析器输出.

### `--redirect-warnings=file`

<!-- YAML
added: v8.0.0
-->

将进程警告写入给定文件，而不是打印到 stderr。如果文件不存在，将创建该文件，如果存在，将附加到该文件.
如果在尝试将警告写入文件时发生错误，则警告将改为写入 stderr.

`file` 名称可以是绝对路径。如果不是，它将被写入的默认目录由 [`--diagnostic-dir`][] 命令行选项控制.

### `--report-compact`

<!-- YAML
added:
 - v13.12.0
 - v12.17.0
-->

以紧凑的格式编写报告，单行 JSON，比为人类消费而设计的默认多行格式更容易被日志处理系统使用.

### `--report-dir=directory`, `report-directory=directory`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This option is no longer experimental.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: Changed from `--diagnostic-report-directory` to
                 `--report-directory`.
-->

生成报告的位置.

### `--report-filename=filename`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This option is no longer experimental.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-filename` to
                 `--report-filename`.
-->

将要写入报告的文件的名称.

### `--report-on-fatalerror`

<!-- YAML
added: v11.8.0
changes:
  - version:
    - v14.0.0
    - v13.14.0
    - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32496
    description: This option is no longer experimental.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-on-fatalerror` to
                 `--report-on-fatalerror`.
-->

允许在导致应用程序终止的致命错误（Node.js 运行时中的内部错误，例如内存不足）上触发报告。有助于检查各种诊断数据元素，例如堆、堆栈、事件循环状态、资源消耗等，以推断致命错误.

### `--report-on-signal`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This option is no longer experimental.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-on-signal` to
                 `--report-on-signal`.
-->

允许在接收到正在运行的 Node.js 进程的指定（或预定义）信号时生成报告。触发报告的信号通过`--report-signal`指定.

### `--report-signal=signal`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This option is no longer experimental.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-signal` to
                 `--report-signal`.
-->

设置或重置报告生成信号（Windows 不支持）.
默认信号是`SIGUSR2`.

### `--report-uncaught-exception`

<!-- YAML
added: v11.8.0
changes:
  - version:
     - v13.12.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32242
    description: This option is no longer experimental.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-uncaught-exception` to
                 `--report-uncaught-exception`.
-->

允许在未捕获的异常上生成报告。结合本机堆栈和其他运行时环境数据检查 JavaScript 堆栈时很有用.

### `--secure-heap=n`

<!-- YAML
added: v15.6.0
-->

初始化 `n` 字节的 OpenSSL 安全堆。初始化时，安全堆用于在密钥生成和其他操作期间在 OpenSSL 中选择的分配类型。例如，这对于防止敏感信息因指针溢出或欠载而泄漏很有用.

安全堆是固定大小的，不能在运行时调整大小，因此，如果使用，选择足够大的堆以覆盖所有应用程序使用非常重要.

给定的堆大小必须是 2 的幂。任何小于 2 的值都将禁用安全堆.

默认情况下禁用安全堆.

安全堆在 Windows 上不可用.

有关更多详细信息，请参阅 [`CRYPTO_secure_malloc_init`][].

### `--secure-heap-min=n`

<!-- YAML
added: v15.6.0
-->

使用 `--secure-heap` 时，`--secure-heap-min` 标志指定安全堆的最小分配。最小值为“2”.
最大值是 `--secure-heap` 或 `2147483647` 中的较小者.
给定的值必须是 2 的幂.

### `--snapshot-blob=path`

<!-- YAML
added: REPLACEME
-->

> Stability: 1 - Experimental

当与 `--build-snapshot` 一起使用时，`--snapshot-blob` 指定生成的快照 blob 将被写入的路径。如果未指定，默认情况下，生成的 blob 将写入当前工作目录中的 `snapshot.blob`.

当不使用 `--build-snapshot` 时，`--snapshot-blob` 指定将用于恢复应用程序状态的 blob 的路径.

### `--test`

<!-- YAML
added: v18.1.0
-->

启动 Node.js 命令行测试运行程序。此标志不能与 `--check`、`--eval`、`--interactive` 或检查器结合使用。有关详细信息，请参阅有关 [从命令行运行测试][] 的文档.

### `--test-only`

<!-- YAML
added: v18.0.0
-->

将测试运行器配置为仅执行设置了 `only` 选项的顶级测试.

### `--throw-deprecation`

<!-- YAML
added: v0.11.14
-->

为弃用抛出错误.

### `--title=title`

<!-- YAML
added: v10.7.0
-->

在启动时设置 `process.title`.

### `--tls-cipher-list=list`

<!-- YAML
added: v4.0.0
-->

指定一个替代的默认 TLS 密码列表。需要使用加密支持构建 Node.js（默认）.

### `--tls-keylog=file`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

将 TLS 密钥材料记录到文件中。密钥材料为 NSS `SSLKEYLOGFILE` 格式，可被软件（如 Wireshark）用于解密 TLS 流量.

### `--tls-max-v1.2`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

将 [`tls.DEFAULT_MAX_VERSION`][] 设置为“TLSv1.2”。用于禁用对 TLSv1.3 的支持.

### `--tls-max-v1.3`

<!-- YAML
added: v12.0.0
-->

将默认 [`tls.DEFAULT_MAX_VERSION`][] 设置为“TLSv1.3”。用于启用对 TLSv1.3 的支持.

### `--tls-min-v1.0`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

将默认 [`tls.DEFAULT_MIN_VERSION`][] 设置为“TLSv1”。用于与旧的 TLS 客户端或服务器兼容.

### `--tls-min-v1.1`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

将默认 [`tls.DEFAULT_MIN_VERSION`][] 设置为“TLSv1.1”。用于与旧的 TLS 客户端或服务器兼容.

### `--tls-min-v1.2`

<!-- YAML
added:
 - v12.2.0
 - v10.20.0
-->

将默认 [`tls.DEFAULT_MIN_VERSION`][] 设置为“TLSv1.2”。这是 12.x 及更高版本的默认设置，但支持该选项以与较旧的 Node.js 版本兼容.

### `--tls-min-v1.3`

<!-- YAML
added: v12.0.0
-->

将默认 [`tls.DEFAULT_MIN_VERSION`][] 设置为“TLSv1.3”。用于禁用对 TLSv1.2 的支持，它不如 TLSv1.3 安全.

### `--trace-atomics-wait`

<!-- YAML
added: v14.3.0
deprecated: REPLACEME
-->

> Stability: 0 - Deprecated

将调用 [`Atomics.wait()`][] 的简短摘要打印到 stderr.
输出可能如下所示:

```text
(node:15701) [Thread 0] Atomics.wait(&lt;address> + 0, 1, inf) started
(node:15701) [Thread 0] Atomics.wait(&lt;address> + 0, 1, inf) did not wait because the values mismatched
(node:15701) [Thread 0] Atomics.wait(&lt;address> + 0, 0, 10) started
(node:15701) [Thread 0] Atomics.wait(&lt;address> + 0, 0, 10) timed out
(node:15701) [Thread 0] Atomics.wait(&lt;address> + 4, 0, inf) started
(node:15701) [Thread 1] Atomics.wait(&lt;address> + 4, -1, inf) started
(node:15701) [Thread 0] Atomics.wait(&lt;address> + 4, 0, inf) was woken up by another thread
(node:15701) [Thread 1] Atomics.wait(&lt;address> + 4, -1, inf) was woken up by another thread
```

这里的字段对应于:

* [`worker_threads.threadId`][] 给出的线程 ID
* 有问题的“SharedArrayBuffer”的基地址，以及与传递给“Atomics.wait()”的索引对应的字节偏移量
* 传递给“Atomics.wait()”的期望值
* 超时传递给“Atomics.wait”

### `--trace-deprecation`

<!-- YAML
added: v0.8.0
-->

打印弃用的堆栈跟踪.

### `--trace-event-categories`

<!-- YAML
added: v7.7.0
-->

使用 `--trace-events-enabled` 启用跟踪事件跟踪时应跟踪的类别的逗号分隔列表.

### `--trace-event-file-pattern`

<!-- YAML
added: v9.8.0
-->

指定跟踪事件数据的文件路径的模板字符串，它支持 `${rotation}` 和 `${pid}`.

### `--trace-events-enabled`

<!-- YAML
added: v7.7.0
-->

启用跟踪事件跟踪信息的收集.

### `--trace-exit`

<!-- YAML
added:
 - v13.5.0
 - v12.16.0
-->

每当主动退出环境时打印堆栈跟踪，即调用 `process.exit()`.

### `--trace-sigint`

<!-- YAML
added:
 - v13.9.0
 - v12.17.0
-->

在 SIGINT 上打印堆栈跟踪.

### `--trace-sync-io`

<!-- YAML
added: v2.1.0
-->

每当在事件循环的第一轮后检测到同步 I/O 时打印堆栈跟踪.

### `--trace-tls`

<!-- YAML
added: v12.2.0
-->

将 TLS 数据包跟踪信息打印到“stderr”。这可用于调试 TLS 连接问题.

### `--trace-uncaught`

<!-- YAML
added: v13.1.0
-->

打印未捕获异常的堆栈跟踪；通常，会打印与创建 `Error` 相关的堆栈跟踪，而这使得 Node.js 也会打印与抛出值相关的堆栈跟踪（不需要是 `Error` 实例）.

启用此选项可能会对垃圾收集行为产生负面影响.

### `--trace-warnings`

<!-- YAML
added: v6.0.0
-->

打印进程警告的堆栈跟踪（包括弃用）.

### `--track-heap-objects`

<!-- YAML
added: v2.4.0
-->

跟踪堆快照的堆对象分配.

### `--unhandled-rejections=mode`

<!-- YAML
added:
  - v12.0.0
  - v10.17.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33021
    description: Changed default mode to `throw`. Previously, a warning was
                 emitted.
-->

使用此标志可以更改发生未处理拒绝时应该发生的情况。可以选择以下模式之一:

* `throw`: 发出 [`unhandledRejection`][]。如果未设置此挂钩，则将未处理的拒绝作为未捕获的异常引发。这是默认设置.
* `strict`: 将未处理的拒绝提升为未捕获的异常。如果处理了异常，则发出 [`unhandledRejection`][].
* `warn`: 始终触发警告，无论是否设置了 [`unhandledRejection`][] 钩子，但不打印弃用警告.
* `warn-with-error-code`: 发出 [`unhandledRejection`][]。如果未设置此钩子，则触发警告，并将进程退出代码设置为 1.
* `none`: 静音所有警告.

如果在命令行入口点的 ES 模块静态加载阶段发生拒绝，它将始终将其作为未捕获的异常引发.

### `--use-bundled-ca`, `--use-openssl-ca`

<!-- YAML
added: v6.11.0
-->

使用当前 Node.js 版本提供的捆绑 Mozilla CA 存储或使用 OpenSSL 的默认 CA 存储。默认存储可在构建时选择.

由 Node.js 提供的捆绑 CA 存储是 Mozilla CA 存储的快照，在发布时已修复。在所有支持的平台上都是相同的.

使用 OpenSSL 存储允许对存储进行外部修改。对于大多数 Linux 和 BSD 发行版，该商店由发行版维护者和系统管理员维护。 OpenSSL CA 存储位置取决于 OpenSSL 库的配置，但这可以在运行时使用环境变量进行更改.

请参阅“SSL_CERT_DIR”和“SSL_CERT_FILE”.

### `--use-largepages=mode`

<!-- YAML
added:
 - v13.6.0
 - v12.17.0
-->

在启动时将 Node.js 静态代码重新映射到大内存页面。如果目标系统支持，这将导致 Node.js 静态代码移动到 2 MiB 页面而不是 4 KiB 页面.

以下值对 `mode` 有效:

* `off`: 不会尝试映射。这是默认设置.
* `on`: 如果操作系统支持，将尝试映射。映射失败将被忽略，一条消息将打印到标准错误.
* `silent`: 如果操作系统支持，将尝试映射。映射失败将被忽略，不会被报告.

### `--v8-options`

<!-- YAML
added: v0.1.3
-->

打印 V8 命令行选项.

### `--v8-pool-size=num`

<!-- YAML
added: v5.10.0
-->

设置 V8 的线程池大小，用于分配后台作业.

如果设置为 0 则 V8 将根据在线处理器的数量选择适当大小的线程池.

如果提供的值大于 V8 的最大值，则选择最大值.

### `--zero-fill-buffers`

<!-- YAML
added: v6.0.0
-->

自动零填充所有新分配的 [`Buffer`][] 和 [`SlowBuffer`][] 实例.

### `-c`, `--check`

<!-- YAML
added:
  - v5.0.0
  - v4.2.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19600
    description: The `--require` option is now supported when checking a file.
-->

语法检查脚本而不执行.

### `-e`, `--eval "script"`

<!-- YAML
added: v0.5.2
changes:
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Built-in libraries are now available as predefined variables.
-->

将以下参数评估为 JavaScript。在 REPL 中预定义的模块也可以在 `script` 中使用.

在 Windows 上，使用 `cmd.exe` 单引号将无法正常工作，因为它只能识别双引号 `"`。在 Powershell 或 Git bash 中，`'` 和 `"` 都可以使用.

### `-h`, `--help`

<!-- YAML
added: v0.1.3
-->

打印节点命令行选项.
此选项的输出没有本文档详细.

### `-i`, `--interactive`

<!-- YAML
added: v0.7.7
-->

即使标准输入看起来不是终端也打开 REPL.

### `-p`, `--print "script"`

<!-- YAML
added: v0.6.4
changes:
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Built-in libraries are now available as predefined variables.
-->

与 `-e` 相同，但打印结果.

### `-r`, `--require module`

<!-- YAML
added: v1.6.0
-->

启动时预加载指定模块.

遵循 `require()` 的模块解析规则。 `module` 可以是文件的路径，也可以是节点模块名称.

仅支持 CommonJS 模块.
使用 [`--import`][] 预加载 [ECMAScript 模块][].
使用 `--require` 预加载的模块将在使用 `--import` 预加载的模块之前运行.

### `-v`, `--version`

<!-- YAML
added: v0.1.3
-->

打印节点的版本.

## Environment variables

### `FORCE_COLOR=[1, 2, 3]`

`FORCE_COLOR` 环境变量用于启用 ANSI 彩色输出。该值可能是:

* `1`, `true` 或空字符串 `''` 表示支持 16 色,
* `2` 表示支持 256 色，或
* `3` 表示支持1600万色.

当使用 `FORCE_COLOR` 并将其设置为支持的值时，`NO_COLOR` 和 `NODE_DISABLE_COLORS` 环境变量都将被忽略.

任何其他值都将导致彩色输出被禁用.

### `NODE_DEBUG=module[,…]`

<!-- YAML
added: v0.1.32
-->

`','` - 应打印调试信息的核心模块的分隔列表.

### `NODE_DEBUG_NATIVE=module[,…]`

`','` - 应打印调试信息的核心 C++ 模块的分隔列表.

### `NODE_DISABLE_COLORS=1`

<!-- YAML
added: v0.3.0
-->

设置后，REPL 中不会使用颜色.

### `NODE_EXTRA_CA_CERTS=file`

<!-- YAML
added: v7.3.0
-->

设置后，众所周知的“根”CA（如 VeriSign）将使用 `file` 中的额外证书进行扩展。该文件应包含一个或多个 PEM 格式的受信任证书。如果文件丢失或格式错误，将使用 [`process.emitWarning()`][emit_warning] 发出（一次）消息，但任何错误都将被忽略.

当为 TLS 或 HTTPS 客户端或服务器显式指定“ca”选项属性时，既不使用众所周知的证书，也不使用额外的证书.

当 `node` 作为 setuid root 运行或设置了 Linux 文件功能时，此环境变量将被忽略.

`NODE_EXTRA_CA_CERTS` 环境变量仅在 Node.js 进程首次启动时读取。使用 `process.env.NODE_EXTRA_CA_CERTS` 在运行时更改值对当前进程没有影响.

### `NODE_ICU_DATA=file`

<!-- YAML
added: v0.11.15
-->

ICU（`Intl` 对象）数据的数据路径。使用 small-icu 支持编译时将扩展链接数据.

### `NODE_NO_WARNINGS=1`

<!-- YAML
added: v6.11.0
-->

当设置为“1”时，进程警告被静音.

### `NODE_OPTIONS=options...`

<!-- YAML
added: v8.0.0
-->

以空格分隔的命令行选项列表。 `options...` 在命令行选项之前解释，因此命令行选项将覆盖或复合在 `options...` 中的任何内容之后。如果使用了环境中不允许的选项，例如 `-p` 或脚本文件.

如果选项值包含空格，则可以使用双引号对其进行转义:

```bash
NODE_OPTIONS='--require "./my path/file.js"'
```

作为命令行选项传递的单例标志将覆盖传递给“NODE_OPTIONS”的相同标志:

```bash
# The inspector will be available on port 5555
NODE_OPTIONS='--inspect=localhost:4444' node --inspect=localhost:5555
```

可以多次传递的标志将被视为先传递其“NODE_OPTIONS”实例，然后再传递其命令行实例:

```bash
NODE_OPTIONS='--require "./a.js"' node --require "./b.js"
# is equivalent to:
node --require "./a.js" --require "./b.js"
```

Node.js options that are allowed are:

<!-- node-options-node start -->

* `--conditions`, `-C`
* `--diagnostic-dir`
* `--disable-proto`
* `--dns-result-order`
* `--enable-fips`
* `--enable-source-maps`
* `--experimental-abortcontroller`
* `--experimental-global-customevent`
* `--experimental-global-webcrypto`
* `--experimental-import-meta-resolve`
* `--experimental-json-modules`
* `--experimental-loader`
* `--experimental-modules`
* `--experimental-network-imports`
* `--experimental-policy`
* `--experimental-shadow-realm`
* `--experimental-specifier-resolution`
* `--experimental-top-level-await`
* `--experimental-vm-modules`
* `--experimental-wasi-unstable-preview1`
* `--experimental-wasm-modules`
* `--force-context-aware`
* `--force-fips`
* `--force-node-api-uncaught-exceptions-policy`
* `--frozen-intrinsics`
* `--heapsnapshot-near-heap-limit`
* `--heapsnapshot-signal`
* `--http-parser`
* `--icu-data-dir`
* `--import`
* `--input-type`
* `--insecure-http-parser`
* `--inspect-brk`
* `--inspect-port`, `--debug-port`
* `--inspect-publish-uid`
* `--inspect`
* `--max-http-header-size`
* `--napi-modules`
* `--no-addons`
* `--no-deprecation`
* `--no-experimental-fetch`
* `--no-experimental-repl-await`
* `--no-extra-info-on-fatal-exception`
* `--no-force-async-hooks-checks`
* `--no-global-search-paths`
* `--no-warnings`
* `--node-memory-debug`
* `--openssl-config`
* `--openssl-legacy-provider`
* `--openssl-shared-config`
* `--pending-deprecation`
* `--policy-integrity`
* `--preserve-symlinks-main`
* `--preserve-symlinks`
* `--prof-process`
* `--redirect-warnings`
* `--report-compact`
* `--report-dir`, `--report-directory`
* `--report-filename`
* `--report-on-fatalerror`
* `--report-on-signal`
* `--report-signal`
* `--report-uncaught-exception`
* `--require`, `-r`
* `--secure-heap-min`
* `--secure-heap`
* `--snapshot-blob`
* `--test-only`
* `--throw-deprecation`
* `--title`
* `--tls-cipher-list`
* `--tls-keylog`
* `--tls-max-v1.2`
* `--tls-max-v1.3`
* `--tls-min-v1.0`
* `--tls-min-v1.1`
* `--tls-min-v1.2`
* `--tls-min-v1.3`
* `--trace-atomics-wait`
* `--trace-deprecation`
* `--trace-event-categories`
* `--trace-event-file-pattern`
* `--trace-events-enabled`
* `--trace-exit`
* `--trace-sigint`
* `--trace-sync-io`
* `--trace-tls`
* `--trace-uncaught`
* `--trace-warnings`
* `--track-heap-objects`
* `--unhandled-rejections`
* `--use-bundled-ca`
* `--use-largepages`
* `--use-openssl-ca`
* `--v8-pool-size`
* `--zero-fill-buffers`

<!-- node-options-node end -->

V8 options that are allowed are:

<!-- node-options-v8 start -->

* `--abort-on-uncaught-exception`
* `--disallow-code-generation-from-strings`
* `--huge-max-old-generation-size`
* `--interpreted-frames-native-stack`
* `--jitless`
* `--max-old-space-size`
* `--perf-basic-prof-only-functions`
* `--perf-basic-prof`
* `--perf-prof-unwinding-info`
* `--perf-prof`
* `--stack-trace-limit`

<!-- node-options-v8 end -->

`--perf-basic-prof-only-functions`, `--perf-basic-prof`,
`--perf-prof-unwinding-info` 和 `--perf-prof` 仅在 Linux 上可用.

### `NODE_PATH=path[:…]`

<!-- YAML
added: v0.1.32
-->

`':'` - 以模块搜索路径为前缀的分隔目录列表.

在 Windows 上，这是一个 `';'` 分隔的列表.

### `NODE_PENDING_DEPRECATION=1`

<!-- YAML
added: v8.0.0
-->

当设置为 `1` 时，发出挂起的弃用警告.

未决弃用通常与运行时弃用相同，但值得注意的例外是它们默认关闭并且不会发出，除非 `--pending-deprecation` 命令行标志或 `NODE_PENDING_DEPRECATION=1` 环境变量，已设置。未决弃用用于提供一种选择性的“早期警告”机制，开发人员可以利用该机制检测弃用的 API 使用情况.

### `NODE_PENDING_PIPE_INSTANCES=instances`

设置管道服务器等待连接时挂起的管道实例句柄数。此设置仅适用于 Windows.

### `NODE_PRESERVE_SYMLINKS=1`

<!-- YAML
added: v7.1.0
-->

当设置为 `1` 时，指示模块加载器在解析和缓存模块时保留符号链接.

### `NODE_REDIRECT_WARNINGS=file`

<!-- YAML
added: v8.0.0
-->

设置后，将向给定文件发出进程警告，而不是打印到 stderr。如果文件不存在，将创建该文件，如果存在，将附加到该文件。如果在尝试将警告写入文件时发生错误，则警告将改为写入 stderr。这等效于使用 `--redirect-warnings=file` 命令行标志.

### `NODE_REPL_HISTORY=file`

<!-- YAML
added: v3.0.0
-->

用于存储持久 REPL 历史记录的文件的路径。默认路径是`~/.node_repl_history`，被这个变量覆盖。将值设置为空字符串（`''` 或 `' '`）会禁用持久性 REPL 历史记录.

### `NODE_REPL_EXTERNAL_MODULE=file`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
-->

Path to a Node.js module which will be loaded in place of the built-in REPL.
将此值覆盖为空字符串 (`''`) 将使用内置 REPL.

### `NODE_SKIP_PLATFORM_CHECK=value`

<!-- YAML
added: v14.5.0
-->

如果 `value` 等于 `'1'`，则在 Node.js 启动期间跳过对受支持平台的检查。 Node.js 可能无法正确执行。在不受支持的平台上遇到的任何问题都不会得到修复.

### `NODE_TLS_REJECT_UNAUTHORIZED=value`

如果 `value` 等于 `'0'`，则禁用 TLS 连接的证书验证.
这使得 TLS 和扩展的 HTTPS 不安全。强烈建议不要使用此环境变量.

### `NODE_V8_COVERAGE=dir`

设置后，Node.js 将开始将 [V8 JavaScript 代码覆盖率][] 和 [Source Map][] 数据输出到作为参数提供的目录（覆盖率信息以 JSON 格式写入带有 `coverage` 前缀的文件）.

`NODE_V8_COVERAGE` 将自动传播到子进程，从而更容易检测调用 `child_process.spawn()` 系列的应用程序
的功能。 `NODE_V8_COVERAGE` 可以设置为空字符串，以防止传播.

#### Coverage output

Coverage 作为顶级键 `result` 上的 [ScriptCoverage][] 对象数组输出:

```json
{
  "result": [
    {
      "scriptId": "67",
      "url": "internal/tty.js",
      "functions": []
    }
  ]
}
```

#### Source map cache

> Stability: 1 - Experimental

如果找到，源地图数据将附加到 JSON 覆盖对象上的顶级键 `source-map-cache`.

`source-map-cache` 是一个对象，其键表示从中提取源映射的文件，其值包括原始源映射 URL（在键 `url` 中）、已解析的 Source Map v3 信息（在键中`data`）和源文件的行长（在键`lineLengths`中）.

```json
{
  "result": [
    {
      "scriptId": "68",
      "url": "file:///absolute/path/to/source.js",
      "functions": []
    }
  ],
  "source-map-cache": {
    "file:///absolute/path/to/source.js": {
      "url": "./path-to-map.json",
      "data": {
        "version": 3,
        "sources": [
          "file:///absolute/path/to/original.js"
        ],
        "names": [
          "Foo",
          "console",
          "info"
        ],
        "mappings": "MAAMA,IACJC,YAAaC",
        "sourceRoot": "./"
      },
      "lineLengths": [
        13,
        62,
        38,
        27
      ]
    }
  }
}
```

### `NO_COLOR=<any>`

[`NO_COLOR`][] 是 `NODE_DISABLE_COLORS` 的别名。环境变量的值是任意的.

### `OPENSSL_CONF=file`

<!-- YAML
added: v6.11.0
-->

在启动时加载 OpenSSL 配置文件。除其他用途外，如果 Node.js 是使用 `./configure --openssl-fips` 构建的，这可用于启用符合 FIPS 的加密。

如果使用 [`--openssl-config`][] 命令行选项，则忽略环境变量.

### `SSL_CERT_DIR=dir`

<!-- YAML
added: v7.7.0
-->

如果启用了 `--use-openssl-ca`，这将覆盖并设置包含受信任证书的 OpenSSL 目录.

请注意，除非显式设置子环境，否则此环境变量将被任何子进程继承，如果它们使用 OpenSSL，可能会导致它们信任与节点相同的 CA.

### `SSL_CERT_FILE=file`

<!-- YAML
added: v7.7.0
-->

如果启用了 `--use-openssl-ca`，这将覆盖并设置包含受信任证书的 OpenSSL 文件.

请注意，除非显式设置子环境，否则此环境变量将被任何子进程继承，如果它们使用 OpenSSL，可能会导致它们信任与节点相同的 CA.

### `TZ`

<!-- YAML
added: v0.0.1
changes:
  - version:
     - v16.2.0
    pr-url: https://github.com/nodejs/node/pull/38642
    description:
      Changing the TZ variable using process.env.TZ = changes the timezone
      on Windows as well.
  - version:
     - v13.0.0
    pr-url: https://github.com/nodejs/node/pull/20026
    description:
      Changing the TZ variable using process.env.TZ = changes the timezone
      on POSIX systems.
-->

`TZ` 环境变量用于指定时区配置.

虽然 Node.js 不支持所有各种 [在其他环境中处理 `TZ` 的方式][]，但它确实支持基本的 [时区 ID][]（例如
`'Etc/UTC'`、`'Europe/Paris'` 或 `'America/New_York'`）.
它可能支持一些其他缩写或别名，但强烈建议不要使用这些缩写或别名，并且不能保证.

```console
$ TZ=Europe/Dublin node -pe "new Date().toString()"
Wed May 12 2021 20:30:48 GMT+0100 (Irish Standard Time)
```

### `UV_THREADPOOL_SIZE=size`

将 libuv 的线程池中使用的线程数设置为 `size` 线程.

Node.js 尽可能使用异步系统 API，但在它们不存在的地方，libuv 的线程池用于基于同步系统 API 创建异步节点 API。使用线程池的 Node.js API 是:

* 所有 `fs` API，除了文件观察器 API 和显式同步的 API
* 异步加密 API，例如 `crypto.pbkdf2()`、`crypto.scrypt()`、`crypto.randomBytes()`、`crypto.randomFill()`、`crypto.generateKeyPair()`
* `dns.lookup()`
* 所有 `zlib` API，除了那些显式同步的 API

因为 libuv 的线程池具有固定大小，这意味着如果出于某种原因，这些 API 中的任何一个需要很长时间，则在 libuv 的线程池中运行的其他（看似无关的）API 将体验到性能下降。为了缓解这个问题，一种可能的解决方案是通过将“UV_THREADPOOL_SIZE”环境变量设置为大于“4”（其当前默认值）的值来增加 libuv 线程池的大小。有关详细信息，请参阅 [libuv 线程池文档][].

## Useful V8 options

V8 有自己的一组 CLI 选项。提供给“node”的任何 V8 CLI 选项都将传递给 V8 进行处理。 V8 的选项有_没有稳定性保证_.
V8 团队本身并不认为它们是其正式 API 的一部分，并保留随时更改它们的权利。同样，它们不在 Node.js 稳定性保证范围内。许多 V8 选项只有 V8 开发人员才感兴趣。尽管如此，仍有一小部分 V8 选项可广泛适用于 Node.js，并在此处记录它们:

### `--max-old-space-size=SIZE` (in megabytes)

设置 V8 旧内存部分的最大内存大小。随着内存消耗接近极限，V8 将花费更多时间进行垃圾收集以释放未使用的内存.

在具有 2 GiB 内存的机器上，考虑将其设置为 1536 (1.5 GiB) 以留出一些内存用于其他用途并避免交换.

```console
$ node --max-old-space-size=1536 index.js
```

### `--max-semi-space-size=SIZE` (in megabytes)

设置 V8 的 [scavenge 垃圾收集器][] 的最大 [semi-space][] 大小，以 MiB 为单位（兆字节）.
增加半空间的最大大小可能会提高 Node.js 的吞吐量，但会消耗更多内存.

由于 V8 堆的年轻代大小是半空间大小的三倍（参见 V8 中的 [`YoungGenerationSizeFromSemiSpaceSize`][]），
半空间增加 1 MiB 适用于三个单独的半空间中的每一个，并导致堆大小增加 3 MiB。吞吐量的提高取决于您的工作量（参见 [#42511][]）.

对于 64 位系统，默认值为 16 MiB，对于 32 位系统，默认值为 8 MiB。要为您的应用程序获得最佳配置，您应该在为您的应用程序运行基准测试时尝试不同的 max-semi-space-size 值.

例如，在 64 位系统上进行基准测试:

```bash
for MiB in 16 32 64 128; do
    node --max-semi-space-size=$MiB index.js
done
```

[#42511]: https://github.com/nodejs/node/issues/42511
[Chrome DevTools Protocol]: https://chromedevtools.github.io/devtools-protocol/
[CommonJS]: modules.md
[CommonJS module]: modules.md
[CustomEvent Web API]: https://dom.spec.whatwg.org/#customevent
[ECMAScript module]: esm.md#modules-ecmascript-modules
[ECMAScript module loader]: esm.md#loaders
[Fetch API]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
[Modules loaders]: packages.md#modules-loaders
[Node.js issue tracker]: https://github.com/nodejs/node/issues
[OSSL_PROVIDER-legacy]: https://www.openssl.org/docs/man3.0/man7/OSSL_PROVIDER-legacy.html
[REPL]: repl.md
[ScriptCoverage]: https://chromedevtools.github.io/devtools-protocol/tot/Profiler#type-ScriptCoverage
[ShadowRealm]: https://github.com/tc39/proposal-shadowrealm
[Source Map]: https://sourcemaps.info/spec.html
[Subresource Integrity]: https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity
[V8 JavaScript code coverage]: https://v8project.blogspot.com/2017/12/javascript-code-coverage.html
[Web Crypto API]: webcrypto.md
[`"type"`]: packages.md#type
[`--cpu-prof-dir`]: #--cpu-prof-dir
[`--diagnostic-dir`]: #--diagnostic-dirdirectory
[`--experimental-wasm-modules`]: #--experimental-wasm-modules
[`--heap-prof-dir`]: #--heap-prof-dir
[`--import`]: #--importmodule
[`--openssl-config`]: #--openssl-configfile
[`--redirect-warnings`]: #--redirect-warningsfile
[`--require`]: #-r---require-module
[`Atomics.wait()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics/wait
[`Buffer`]: buffer.md#class-buffer
[`CRYPTO_secure_malloc_init`]: https://www.openssl.org/docs/man1.1.0/man3/CRYPTO_secure_malloc_init.html
[`NODE_OPTIONS`]: #node_optionsoptions
[`NO_COLOR`]: https://no-color.org
[`SlowBuffer`]: buffer.md#class-slowbuffer
[`YoungGenerationSizeFromSemiSpaceSize`]: https://chromium.googlesource.com/v8/v8.git/+/refs/tags/10.3.129/src/heap/heap.cc#328
[`dns.lookup()`]: dns.md#dnslookuphostname-options-callback
[`dns.setDefaultResultOrder()`]: dns.md#dnssetdefaultresultorderorder
[`dnsPromises.lookup()`]: dns.md#dnspromiseslookuphostname-options
[`import` specifier]: esm.md#import-specifiers
[`process.setUncaughtExceptionCaptureCallback()`]: process.md#processsetuncaughtexceptioncapturecallbackfn
[`tls.DEFAULT_MAX_VERSION`]: tls.md#tlsdefault_max_version
[`tls.DEFAULT_MIN_VERSION`]: tls.md#tlsdefault_min_version
[`unhandledRejection`]: process.md#event-unhandledrejection
[`v8.startupSnapshot` API]: v8.md#startup-snapshot-api
[`worker_threads.threadId`]: worker_threads.md#workerthreadid
[conditional exports]: packages.md#conditional-exports
[context-aware]: addons.md#context-aware-addons
[customizing ESM specifier resolution]: esm.md#customizing-esm-specifier-resolution-algorithm
[debugger]: debugger.md
[debugging security implications]: https://nodejs.org/en/docs/guides/debugging-getting-started/#security-implications
[emit_warning]: process.md#processemitwarningwarning-options
[jitless]: https://v8.dev/blog/jitless
[libuv threadpool documentation]: https://docs.libuv.org/en/latest/threadpool.html
[remote code execution]: https://www.owasp.org/index.php/Code_Injection
[running tests from the command line]: test.md#running-tests-from-the-command-line
[scavenge garbage collector]: https://v8.dev/blog/orinoco-parallel-scavenger
[security warning]: #warning-binding-inspector-to-a-public-ipport-combination-is-insecure
[semi-space]: https://www.memorymanagement.org/glossary/s.html#semi.space
[timezone IDs]: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
[tracking issue for user-land snapshots]: https://github.com/nodejs/node/issues/44014
[ways that `TZ` is handled in other environments]: https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html
