# TTY

<!--introduced_in=v0.10.0-->

> Stability: 2 - Stable

<!-- source_link=lib/tty.js -->

`node:tty` 模块提供了 `tty.ReadStream` 和 `tty.WriteStream` 类。在大多数情况下，没有必要或不可能直接使用此模块。但是，它可以使用:

```js
const tty = require('node:tty');
```

当 Node.js 检测到它正在使用附加的文本终端（“TTY”）运行时，默认情况下，[`process.stdin`][] 将被初始化为 `tty.ReadStream` 的实例，并且 [` process.stdout`][] 和 [`process.stderr`][] 默认是 `tty.WriteStream` 的实例。确定 Node.js 是否在 TTY 上下文中运行的首选方法是检查 `process.stdout.isTTY` 属性的值是否为 `true`:

```console
$ node -p -e "Boolean(process.stdout.isTTY)"
true
$ node -p -e "Boolean(process.stdout.isTTY)" | cat
false
```

在大多数情况下，应用程序几乎没有理由手动创建 tty.ReadStream 和 tty.WriteStream 类的实例.

## Class: `tty.ReadStream`

<!-- YAML
added: v0.5.8
-->

* Extends: {net.Socket}

表示 TTY 的可读面。在正常情况下 [`process.stdin`][] 将是 Node.js 进程中唯一的 `tty.ReadStream` 实例，应该没有理由创建额外的实例.

### `readStream.isRaw`

<!-- YAML
added: v0.7.7
-->

如果 TTY 当前配置为作为原始设备运行，则为 `true` 的`boolean`。默认为“假”.

### `readStream.isTTY`

<!-- YAML
added: v0.5.8
-->

对于 `tty.ReadStream` 实例，`boolean` 始终为 `true`.

### `readStream.setRawMode(mode)`

<!-- YAML
added: v0.7.7
-->

* `mode` {boolean} 如果为 `true`，则将 `tty.ReadStream` 配置为作为原始设备运行。如果为 `false`，则将 `tty.ReadStream` 配置为在其默认模式下运行。 `readStream.isRaw` 属性将设置为结果模式.
* Returns: {this} The read stream instance.

允许配置 `tty.ReadStream` 使其作为原始设备运行.

在原始模式下，输入始终是逐个字符可用的，不包括修饰符。此外，终端对字符的所有特殊处理都被禁用，包括回显输入字符. <kbd>Ctrl</kbd>+<kbd>C</kbd> 在此模式下将不再导致“SIGINT”.

## Class: `tty.WriteStream`

<!-- YAML
added: v0.5.8
-->

* Extends: {net.Socket}

表示 TTY 的可写侧。在正常情况下，[`process.stdout`][] 和 [`process.stderr`][] 将是为 Node.js 进程创建的唯一 `tty.WriteStream` 实例，应该没有理由创建其他实例.

### Event: `'resize'`

<!-- YAML
added: v0.7.7
-->

每当 `writeStream.columns` 或 `writeStream.rows` 属性发生变化时，都会触发 `resize' 事件。调用时没有参数传递给侦听器回调.

```js
process.stdout.on('resize', () => {
  console.log('screen size has changed!');
  console.log(`${process.stdout.columns}x${process.stdout.rows}`);
});
```

### `writeStream.clearLine(dir[, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28721
    description: The stream's write() callback and return value are exposed.
-->

* `dir` {number}
  * `-1`: to the left from cursor
  * `1`: to the right from cursor
  * `0`: the entire line
* `callback` {Function} Invoked once the operation completes.
* Returns: {boolean} `false` if the stream wishes for the calling code to wait
  for the `'drain'` event to be emitted before continuing to write additional
  data; otherwise `true`.

`writeStream.clearLine()` 在 `dir` 标识的方向上清除此 `WriteStream` 的当前行.

### `writeStream.clearScreenDown([callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28721
    description: The stream's write() callback and return value are exposed.
-->

* `callback` {Function} Invoked once the operation completes.
* Returns: {boolean} `false` if the stream wishes for the calling code to wait
  for the `'drain'` event to be emitted before continuing to write additional
  data; otherwise `true`.

`writeStream.clearScreenDown()` 从当前光标向下清除此 `WriteStream`.

### `writeStream.columns`

<!-- YAML
added: v0.7.7
-->

一个 `number` 指定 TTY 当前拥有的列数。每当发出 `'resize'` 事件时，都会更新此属性.

### `writeStream.cursorTo(x[, y][, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28721
    description: The stream's write() callback and return value are exposed.
-->

* `x` {number}
* `y` {number}
* `callback` {Function} Invoked once the operation completes.
* Returns: {boolean} `false` if the stream wishes for the calling code to wait
  for the `'drain'` event to be emitted before continuing to write additional
  data; otherwise `true`.

`writeStream.cursorTo()` 将此 `WriteStream` 的光标移动到指定位置.

### `writeStream.getColorDepth([env])`

<!-- YAML
added: v9.9.0
-->

* `env` {Object} An object containing the environment variables to check. This
  enables simulating the usage of a specific terminal. **Default:**
  `process.env`.
* Returns: {number}

Returns:

* `1` for 2,
* `4` for 16,
* `8` for 256,
* `24` for 16,777,216 colors supported.

使用它来确定终端支持的颜色。由于终端颜色的性质，可能出现误报或误报。这取决于进程信息和环境变量，这些变量可能与使用的终端有关.
可以传入一个 `env` 对象来模拟特定终端的使用。这对于检查特定环境设置的行为方式很有用.

要强制执行特定颜色支持，请使用以下环境设置之一.

* 2 colors: `FORCE_COLOR = 0` (Disables colors)
* 16 colors: `FORCE_COLOR = 1`
* 256 colors: `FORCE_COLOR = 2`
* 16,777,216 colors: `FORCE_COLOR = 3`

通过使用 `NO_COLOR` 和 `NODE_DISABLE_COLORS` 环境变量也可以禁用颜色支持.

### `writeStream.getWindowSize()`

<!-- YAML
added: v0.7.7
-->

* Returns: {number\[]}

`writeStream.getWindowSize()` 返回与此 `WriteStream` 对应的 TTY 的大小。数组的类型为“[numColumns, numRows]”，其中“numColumns”和“numRows”表示相应 TTY 中的列数和行数.

### `writeStream.hasColors([count][, env])`

<!-- YAML
added:
 - v11.13.0
 - v10.16.0
-->

* `count` {integer} The number of colors that are requested (minimum 2).
  **Default:** 16.
* `env` {Object} An object containing the environment variables to check. This
  enables simulating the usage of a specific terminal. **Default:**
  `process.env`.
* Returns: {boolean}

如果 `writeStream` 支持的颜色至少与 `count` 中提供的颜色一样多，则返回 `true`。最小支持为 2（黑白）.

这具有与 [`writeStream.getColorDepth()`][] 中所述相同的误报和误报.

```js
process.stdout.hasColors();
// Returns true or false depending on if `stdout` supports at least 16 colors.
process.stdout.hasColors(256);
// Returns true or false depending on if `stdout` supports at least 256 colors.
process.stdout.hasColors({ TMUX: '1' });
// Returns true.
process.stdout.hasColors(2 ** 24, { TMUX: '1' });
// Returns false (the environment setting pretends to support 2 ** 8 colors).
```

### `writeStream.isTTY`

<!-- YAML
added: v0.5.8
-->

A `boolean` that is always `true`.

### `writeStream.moveCursor(dx, dy[, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28721
    description: The stream's write() callback and return value are exposed.
-->

* `dx` {number}
* `dy` {number}
* `callback` {Function} Invoked once the operation completes.
* Returns: {boolean} `false` if the stream wishes for the calling code to wait
  for the `'drain'` event to be emitted before continuing to write additional
  data; otherwise `true`.

`writeStream.moveCursor()` 将这个 `WriteStream` 的光标 _relative_ 移动到其当前位置.

### `writeStream.rows`

<!-- YAML
added: v0.7.7
-->

一个 `number` 指定 TTY 当前拥有的行数。每当发出 `'resize'` 事件时，都会更新此属性.

## `tty.isatty(fd)`

<!-- YAML
added: v0.5.8
-->

* `fd` {number} A numeric file descriptor
* Returns: {boolean}

如果给定的 `fd` 与 TTY 相关联，则 `tty.isatty()` 方法返回 `true`，否则返回 `false`，包括任何时候 `fd` 不是非负整数.

[`process.stderr`]: process.md#processstderr
[`process.stdin`]: process.md#processstdin
[`process.stdout`]: process.md#processstdout
[`writeStream.getColorDepth()`]: #writestreamgetcolordepthenv
