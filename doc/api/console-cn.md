# Console

<!--introduced_in=v0.10.13-->

> Stability: 2 - Stable

<!-- source_link=lib/console.js -->

`node:console` 模块提供了一个简单的调试控制台，类似于 Web 浏览器提供的 JavaScript 控制台机制。

该模块导出两个特定组件:

* 一个具有诸如 `console.log()`、`console.error()` 和 `console.warn()` 等方法的`Console` 类，可用于写入任何 Node.js 流.
* 配置为写入 [`process.stdout`][] 和 [`process.stderr`][] 的全局 `console` 实例。无需调用 `require('node:console')` 即可使用全局 `console`.

_**警告**_：全局控制台对象的方法既不像浏览器 API 那样始终保持同步，也不像所有其他 Node.js 流那样始终保持异步。有关更多信息，请参阅 [关于进程 I/O 的说明][].

使用全局“控制台”的示例:

```js
console.log('hello world');
// Prints: hello world, to stdout
console.log('hello %s', 'world');
// Prints: hello world, to stdout
console.error(new Error('Whoops, something bad happened'));
// Prints error message and stack trace to stderr:
//   Error: Whoops, something bad happened
//     at [eval]:5:15
//     at Script.runInThisContext (node:vm:132:18)
//     at Object.runInThisContext (node:vm:309:38)
//     at node:internal/process/execution:77:19
//     at [eval]-wrapper:6:22
//     at evalScript (node:internal/process/execution:76:60)
//     at node:internal/main/eval_string:23:3

const name = 'Will Robinson';
console.warn(`Danger ${name}! Danger!`);
// Prints: Danger Will Robinson! Danger!, to stderr
```

使用 `Console` 类的示例:

```js
const out = getStreamSomehow();
const err = getStreamSomehow();
const myConsole = new console.Console(out, err);

myConsole.log('hello world');
// Prints: hello world, to out
myConsole.log('hello %s', 'world');
// Prints: hello world, to out
myConsole.error(new Error('Whoops, something bad happened'));
// Prints: [Error: Whoops, something bad happened], to err

const name = 'Will Robinson';
myConsole.warn(`Danger ${name}! Danger!`);
// Prints: Danger Will Robinson! Danger!, to err
```

## Class: `Console`

<!-- YAML
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9744
    description: Errors that occur while writing to the underlying streams
                 will now be ignored by default.
-->

<!--type=class-->

`Console` 类可用于创建具有可配置输出流的简单记录器，并且可以使用 `require('node:console').Console` 或 `console.Console` （或它们的解构对应物）访问:

```js
const { Console } = require('node:console');
```

```js
const { Console } = console;
```

### `new Console(stdout[, stderr][, ignoreErrors])`

### `new Console(options)`

<!-- YAML
changes:
  - version:
     - v14.2.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32964
    description: The `groupIndentation` option was introduced.
  - version: v11.7.0
    pr-url: https://github.com/nodejs/node/pull/24978
    description: The `inspectOptions` option is introduced.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19372
    description: The `Console` constructor now supports an `options` argument,
                 and the `colorMode` option was introduced.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9744
    description: The `ignoreErrors` option was introduced.
-->

* `options` {Object}
  * `stdout` {stream.Writable}
  * `stderr` {stream.Writable}
  * `ignoreErrors` {boolean} Ignore errors when writing to the underlying
    streams. **Default:** `true`.
  * `colorMode` {boolean|string} Set color support for this `Console` instance.
    Setting to `true` enables coloring while inspecting values. Setting to
    `false` disables coloring while inspecting values. Setting to
    `'auto'` makes color support depend on the value of the `isTTY` property
    and the value returned by `getColorDepth()` on the respective stream. This
    option can not be used, if `inspectOptions.colors` is set as well.
    **Default:** `'auto'`.
  * `inspectOptions` {Object} Specifies options that are passed along to
    [`util.inspect()`][].
  * `groupIndentation` {number} Set group indentation.
    **Default:** `2`.

使用一个或两个可写流实例创建一个新的“控制台”。 `stdout` 是用于打印日志或信息输出的可写流。 `stderr` 用于警告或错误输出。如果未提供 `stderr`，则 `stdout` 用于 `stderr`.

```js
const output = fs.createWriteStream('./stdout.log');
const errorOutput = fs.createWriteStream('./stderr.log');
// Custom simple logger
const logger = new Console({ stdout: output, stderr: errorOutput });
// use it like console
const count = 5;
logger.log('count: %d', count);
// In stdout.log: count 5
```

全局 `console` 是一个特殊的 `Console`，其输出被发送到 [`process.stdout`][] 和 [`process.stderr`][]。相当于调用:

```js
new Console({ stdout: process.stdout, stderr: process.stderr });
```

### `console.assert(value[, ...message])`

<!-- YAML
added: v0.1.101
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17706
    description: The implementation is now spec compliant and does not throw
                 anymore.
-->

* `value` {any} The value tested for being truthy.
* `...message` {any} All arguments besides `value` are used as error message.

如果 `value` 为 [falsy][] 或省略，`console.assert()` 会写入一条消息。它只写一条消息，不会影响执行。输出总是以“断言失败”开始。如果提供，`message` 使用 [`util.format()`][] 格式化.

如果 `value` 是 [truthy][]，则什么也不会发生.

```js
console.assert(true, 'does nothing');

console.assert(false, 'Whoops %s work', 'didn\'t');
// Assertion failed: Whoops didn't work

console.assert();
// Assertion failed
```

### `console.clear()`

<!-- YAML
added: v8.3.0
-->

当 `stdout` 是 TTY 时，调用 `console.clear()` 将尝试清除 TTY。当 `stdout` 不是 TTY 时，此方法不执行任何操作.

`console.clear()` 的具体操作可能因操作系统和终端类型而异。对于大多数 Linux 操作系统，`console.clear()` 的操作类似于 `clear` shell 命令。在 Windows 上，`console.clear()` 将仅清除 Node.js 二进制文件的当前终端视口中的输出.

### `console.count([label])`

<!-- YAML
added: v8.3.0
-->

* `label` {string} The display label for the counter. **Default:** `'default'`.

维护一个特定于`label`的内部计数器，并将`console.count()`使用给定`label`调用的次数输出到`stdout`.

<!-- eslint-skip -->

```js
> console.count()
default: 1
undefined
> console.count('default')
default: 2
undefined
> console.count('abc')
abc: 1
undefined
> console.count('xyz')
xyz: 1
undefined
> console.count('abc')
abc: 2
undefined
> console.count()
default: 3
undefined
>
```

### `console.countReset([label])`

<!-- YAML
added: v8.3.0
-->

* `label` {string} The display label for the counter. **Default:** `'default'`.

重置特定于“标签”的内部计数器.

<!-- eslint-skip -->

```js
> console.count('abc');
abc: 1
undefined
> console.countReset('abc');
undefined
> console.count('abc');
abc: 1
undefined
>
```

### `console.debug(data[, ...args])`

<!-- YAML
added: v8.0.0
changes:
  - version: v8.10.0
    pr-url: https://github.com/nodejs/node/pull/17033
    description: "`console.debug` is now an alias for `console.log`."
-->

* `data` {any}
* `...args` {any}

`console.debug()` 函数是 [`console.log()`][] 的别名.

### `console.dir(obj[, options])`

<!-- YAML
added: v0.1.101
-->

* `obj` {any}
* `options` {Object}
  * `showHidden` {boolean} If `true` then the object's non-enumerable and symbol
    properties will be shown too. **Default:** `false`.
  * `depth` {number} Tells [`util.inspect()`][] how many times to recurse while
    formatting the object. This is useful for inspecting large complicated
    objects. To make it recurse indefinitely, pass `null`. **Default:** `2`.
  * `colors` {boolean} If `true`, then the output will be styled with ANSI color
    codes. Colors are customizable;
    see [customizing `util.inspect()` colors][]. **Default:** `false`.

在 `obj` 上使用 [`util.inspect()`][] 并将结果字符串打印到 `stdout`.
此函数绕过在 `obj` 上定义的任何自定义 `inspect()` 函数.

### `console.dirxml(...data)`

<!-- YAML
added: v8.0.0
changes:
  - version: v9.3.0
    pr-url: https://github.com/nodejs/node/pull/17152
    description: "`console.dirxml` now calls `console.log` for its arguments."
-->

* `...data` {any}

此方法调用 `console.log()` 将接收到的参数传递给它.
此方法不产生任何 XML 格式.

### `console.error([data][, ...args])`

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

使用换行符打印到 `stderr`。可以传递多个参数，第一个用作主要消息，所有附加的用作类似于 printf(3) 的替换值（参数都传递给 [`util.format()`][]）.

```js
const code = 5;
console.error('error #%d', code);
// Prints: error #5, to stderr
console.error('error', code);
// Prints: error 5, to stderr
```

如果在第一个字符串中找不到格式化元素（例如 `%d`），则在每个参数上调用 [`util.inspect()`][] 并将结果字符串值连接起来。有关详细信息，请参阅 [`util.format()`][].

### `console.group([...label])`

<!-- YAML
added: v8.5.0
-->

* `...label` {any}

通过空格增加后续行的缩进以达到 `groupIndentation` 长度.

如果提供了一个或多个“标签”，则首先打印这些标签，而无需额外的缩进.

### `console.groupCollapsed()`

<!-- YAML
  added: v8.5.0
-->

[`console.group()`][] 的别名.

### `console.groupEnd()`

<!-- YAML
added: v8.5.0
-->

通过空格减少后续行的缩进以达到 `groupIndentation` 长度.

### `console.info([data][, ...args])`

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

`console.info()` 函数是 [`console.log()`][] 的别名.

### `console.log([data][, ...args])`

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

使用换行符打印到 `stdout`。可以传递多个参数，第一个用作主要消息，所有附加的用作类似于 printf(3) 的替换值（参数都传递给 [`util.format()`][]）.

```js
const count = 5;
console.log('count: %d', count);
// Prints: count: 5, to stdout
console.log('count:', count);
// Prints: count: 5, to stdout
```

See [`util.format()`][] for more information.

### `console.table(tabularData[, properties])`

<!-- YAML
added: v10.0.0
-->

* `tabularData` {any}
* `properties` {string\[]} Alternate properties for constructing the table.

尝试用 `tabularData` 的属性列（或使用 `properties`）和 `tabularData` 的行构建一个表并记录它。如果无法将其解析为表格，则回退到仅记录参数.

```js
// These can't be parsed as tabular data
console.table(Symbol());
// Symbol()

console.table(undefined);
// undefined

console.table([{ a: 1, b: 'Y' }, { a: 'Z', b: 2 }]);
// ┌─────────┬─────┬─────┐
// │ (index) │  a  │  b  │
// ├─────────┼─────┼─────┤
// │    0    │  1  │ 'Y' │
// │    1    │ 'Z' │  2  │
// └─────────┴─────┴─────┘

console.table([{ a: 1, b: 'Y' }, { a: 'Z', b: 2 }], ['a']);
// ┌─────────┬─────┐
// │ (index) │  a  │
// ├─────────┼─────┤
// │    0    │  1  │
// │    1    │ 'Z' │
// └─────────┴─────┘
```

### `console.time([label])`

<!-- YAML
added: v0.1.104
-->

* `label` {string} **Default:** `'default'`

启动可用于计算操作持续时间的计时器。计时器由唯一的“标签”标识。在调用 [`console.timeEnd()`][] 时使用相同的 `label` 来停止计时器并将经过的时间以合适的时间单位输出到 `stdout`。例如，如果经过的时间是 3869 毫秒，则 `console.timeEnd()` 显示“3.869s”.

### `console.timeEnd([label])`

<!-- YAML
added: v0.1.104
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/29251
    description: The elapsed time is displayed with a suitable time unit.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5901
    description: This method no longer supports multiple calls that don't map
                 to individual `console.time()` calls; see below for details.
-->

* `label` {string} **Default:** `'default'`

停止之前通过调用 [`console.time()`][] 启动的计时器并将结果打印到 `stdout`:

```js
console.time('bunch-of-stuff');
// Do a bunch of stuff.
console.timeEnd('bunch-of-stuff');
// Prints: bunch-of-stuff: 225.438ms
```

### `console.timeLog([label][, ...data])`

<!-- YAML
added: v10.7.0
-->

* `label` {string} **Default:** `'default'`
* `...data` {any}

对于之前通过调用 [`console.time()`][] 启动的计时器，将经过的时间和其他 `data` 参数打印到 `stdout`:

```js
console.time('process');
const value = expensiveProcess1(); // Returns 42
console.timeLog('process', value);
// Prints "process: 365.227ms 42".
doExpensiveProcess2(value);
console.timeEnd('process');
```

### `console.trace([message][, ...args])`

<!-- YAML
added: v0.1.104
-->

* `message` {any}
* `...args` {any}

将字符串 `'Trace: '` 打印到 `stderr`，然后是 [`util.format()`][] 格式化消息和堆栈跟踪到代码中的当前位置.

```js
console.trace('Show me');
// Prints: (stack trace will vary based on where trace is called)
//  Trace: Show me
//    at repl:2:9
//    at REPLServer.defaultEval (repl.js:248:27)
//    at bound (domain.js:287:14)
//    at REPLServer.runBound [as eval] (domain.js:300:12)
//    at REPLServer.<anonymous> (repl.js:412:12)
//    at emitOne (events.js:82:20)
//    at REPLServer.emit (events.js:169:7)
//    at REPLServer.Interface._onLine (readline.js:210:10)
//    at REPLServer.Interface._line (readline.js:549:8)
//    at REPLServer.Interface._ttyWrite (readline.js:826:14)
```

### `console.warn([data][, ...args])`

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

`console.warn()` 函数是 [`console.error()`][] 的别名.

## Inspector only methods

以下方法由 V8 引擎在通用 API 中公开，但除非与 [inspector][]（`--inspect` 标志）一起使用，否则不会显示任何内容.

### `console.profile([label])`

<!-- YAML
added: v8.0.0
-->

* `label` {string}

除非在检查器中使用，否则此方法不会显示任何内容。 `console.profile()` 方法使用可选标签启动 JavaScript CPU 配置文件，直到调用 [`console.profileEnd()`][]。然后将配置文件添加到检查器的 **Profile** 面板.

```js
console.profile('MyLabel');
// Some code
console.profileEnd('MyLabel');
// Adds the profile 'MyLabel' to the Profiles panel of the inspector.
```

### `console.profileEnd([label])`

<!-- YAML
added: v8.0.0
-->

* `label` {string}

除非在检查器中使用，否则此方法不会显示任何内容。如果已启动一个 JavaScript CPU 分析会话，则停止当前的 JavaScript CPU 分析会话，并将报告打印到检查器的 **Profiles** 面板。有关示例，请参见 [`console.profile()`][].

如果在没有标签的情况下调用此方法，则会停止最近启动的配置文件.

### `console.timeStamp([label])`

<!-- YAML
added: v8.0.0
-->

* `label` {string}

除非在检查器中使用，否则此方法不会显示任何内容。 `console.timeStamp()` 方法将带有标签 `'label'` 的事件添加到检查器的 **Timeline** 面板.

[`console.error()`]: #consoleerrordata-args
[`console.group()`]: #consolegrouplabel
[`console.log()`]: #consolelogdata-args
[`console.profile()`]: #consoleprofilelabel
[`console.profileEnd()`]: #consoleprofileendlabel
[`console.time()`]: #consoletimelabel
[`console.timeEnd()`]: #consoletimeendlabel
[`process.stderr`]: process.md#processstderr
[`process.stdout`]: process.md#processstdout
[`util.format()`]: util.md#utilformatformat-args
[`util.inspect()`]: util.md#utilinspectobject-options
[customizing `util.inspect()` colors]: util.md#customizing-utilinspect-colors
[falsy]: https://developer.mozilla.org/en-US/docs/Glossary/Falsy
[inspector]: debugger.md
[note on process I/O]: process.md#a-note-on-process-io
[truthy]: https://developer.mozilla.org/en-US/docs/Glossary/Truthy
