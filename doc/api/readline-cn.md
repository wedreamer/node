# 读线

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/readline.js -->

这`node:readline`模块提供了一个接口，用于从
[读][Readable]流（例如[`process.stdin`][process.stdin]） 一次一行。

要使用基于承诺的 API，请执行以下操作：

```mjs
import * as readline from 'node:readline/promises';
```

```cjs
const readline = require('node:readline/promises');
```

要使用回调和同步 API，请执行以下操作：

```mjs
import * as readline from 'node:readline';
```

```cjs
const readline = require('node:readline');
```

以下简单示例说明了`node:readline`
模块。

```mjs
import * as readline from 'node:readline/promises';
import { stdin as input, stdout as output } from 'node:process';

const rl = readline.createInterface({ input, output });

const answer = await rl.question('What do you think of Node.js? ');

console.log(`Thank you for your valuable feedback: ${answer}`);

rl.close();
```

```cjs
const readline = require('node:readline');
const { stdin: input, stdout: output } = require('node:process');

const rl = readline.createInterface({ input, output });

rl.question('What do you think of Node.js? ', (answer) => {
  // TODO: Log the answer in a database
  console.log(`Thank you for your valuable feedback: ${answer}`);

  rl.close();
});
```

调用此代码后，Node.js 应用程序将不会终止，直到
`readline.Interface`关闭，因为接口等待数据
在`input`流。

<a id='readline_class_interface'></a>

## 类：`InterfaceConstructor`

<!-- YAML
added: v0.1.104
-->

*   扩展：{事件发射器}

的实例`InterfaceConstructor`类是使用
`readlinePromises.createInterface()`或`readline.createInterface()`方法。
每个实例都与单个实例相关联`input` [读][Readable]流和
单`output` [写][Writable]流。
这`output`流用于打印到达的用户输入的提示，
并从中读取，`input`流。

### 事件：`'close'`

<!-- YAML
added: v0.1.98
-->

这`'close'`发生下列情况之一时，将发出事件：

*   这`rl.close()`调用方法，并且`InterfaceConstructor`实例具有
    放弃对`input`和`output`流;
*   这`input`流接收其`'end'`事件;
*   这`input`流接收<kbd>Ctrl</kbd>+<kbd>D</kbd>到信号
    传输末端（ EOT）;
*   这`input`流接收<kbd>Ctrl</kbd>+<kbd>C</kbd>到信号`SIGINT`
    并且没有`'SIGINT'`在 上注册的事件侦听器
    `InterfaceConstructor`实例。

调用侦听器函数时不传递任何参数。

这`InterfaceConstructor`实例完成一次`'close'`事件是
排放。

### 事件：`'line'`

<!-- YAML
added: v0.1.98
-->

这`'line'`每当`input`流接收
下线输入 （`\n`,`\r`或`\r\n`).这通常发生在用户
印刷机<kbd>进入</kbd>或<kbd>返回</kbd>.

这`'line'`如果已从流中读取新数据，则还会发出事件，并且
该流在没有最终行尾标记的情况下结束。

使用包含单行的字符串调用侦听器函数
接收的输入。

```js
rl.on('line', (input) => {
  console.log(`Received: ${input}`);
});
```

### 事件：`'history'`

<!-- YAML
added:
  - v15.8.0
  - v14.18.0
-->

这`'history'`每当历史记录数组发生更改时，就会发出事件。

侦听器函数是使用包含历史记录数组的数组调用的。
它将反映由于以下原因而添加的所有更改，添加的行和删除的行
`historySize`和`removeHistoryDuplicates`.

主要目的是允许侦听器保留历史记录。
侦听器也可以更改历史记录对象。这
可能有助于防止将某些行添加到历史记录中，例如
密码。

```js
rl.on('history', (history) => {
  console.log(`Received: ${history}`);
});
```

### 事件：`'pause'`

<!-- YAML
added: v0.7.5
-->

这`'pause'`发生下列情况之一时，将发出事件：

*   这`input`流已暂停。
*   这`input`流未暂停，并接收`'SIGCONT'`事件。（请参见
    事件[`'SIGTSTP'`]['SIGTSTP']和[`'SIGCONT'`]['SIGCONT'].)

调用侦听器函数时不传递任何参数。

```js
rl.on('pause', () => {
  console.log('Readline paused.');
});
```

### 事件：`'resume'`

<!-- YAML
added: v0.7.5
-->

这`'resume'`每当`input`流已恢复。

调用侦听器函数时不传递任何参数。

```js
rl.on('resume', () => {
  console.log('Readline resumed.');
});
```

### 事件：`'SIGCONT'`

<!-- YAML
added: v0.7.5
-->

这`'SIGCONT'`事件在节点.js进程先前移动到
背景使用<kbd>Ctrl</kbd>+<kbd>Z</kbd>（即`SIGTSTP`） 则
使用 fg（1p） 带回前台。

如果`input`流已暂停*以前*这`SIGTSTP`请求，此事件将
不发出。

调用侦听器函数时不传递任何参数。

```js
rl.on('SIGCONT', () => {
  // `prompt` will automatically resume the stream
  rl.prompt();
});
```

这`'SIGCONT'`事件是*不*在 Windows 上受支持。

### 事件：`'SIGINT'`

<!-- YAML
added: v0.3.0
-->

这`'SIGINT'`每当`input`流接收
一个<kbd>Ctrl+C</kbd>输入，通常称为`SIGINT`.如果没有
`'SIGINT'`事件侦听器注册时`input`流接收
`SIGINT`这`'pause'`将发出事件。

调用侦听器函数时不传递任何参数。

```js
rl.on('SIGINT', () => {
  rl.question('Are you sure you want to exit? ', (answer) => {
    if (answer.match(/^y(es)?$/i)) rl.pause();
  });
});
```

### 事件：`'SIGTSTP'`

<!-- YAML
added: v0.7.5
-->

这`'SIGTSTP'`事件在`input`流接收
一个<kbd>Ctrl</kbd>+<kbd>Z</kbd>输入，通常称为`SIGTSTP`.如果有
不`'SIGTSTP'`事件侦听器注册时`input`流接收
`SIGTSTP`，节点.js进程将被发送到后台。

当使用 fg（1p） 恢复程序时，`'pause'`和`'SIGCONT'`事件
将发出。这些可用于恢复`input`流。

这`'pause'`和`'SIGCONT'`如果`input`是
在将进程发送到后台之前暂停。

调用侦听器函数时不传递任何参数。

```js
rl.on('SIGTSTP', () => {
  // This will override SIGTSTP and prevent the program from going to the
  // background.
  console.log('Caught SIGTSTP.');
});
```

这`'SIGTSTP'`事件是*不*在 Windows 上受支持。

### `rl.close()`

<!-- YAML
added: v0.1.98
-->

这`rl.close()`方法关闭`InterfaceConstructor`实例和
放弃对`input`和`output`流。当被调用时，
这`'close'`将发出事件。

叫`rl.close()`不会立即停止其他事件（包括`'line'`)
从 发出的`InterfaceConstructor`实例。

### `rl.pause()`

<!-- YAML
added: v0.3.4
-->

这`rl.pause()`方法暂停`input`流，允许恢复
如有必要，稍后再使用。

叫`rl.pause()`不会立即暂停其他事件（包括
`'line'`） 从 发出`InterfaceConstructor`实例。

### `rl.prompt([preserveCursor])`

<!-- YAML
added: v0.1.98
-->

*   `preserveCursor`{布尔值}如果`true`，防止光标放置
    重置为`0`.

这`rl.prompt()`方法将`InterfaceConstructor`配置的实例
`prompt`到 新行`output`以便为用户提供新的
提供输入的位置。

当被调用时，`rl.prompt()`将恢复`input`流（如果已）
暂停。

如果`InterfaceConstructor`创建于`output`设置为`null`或
`undefined`未写入提示。

### `rl.question(query[, options], callback)`

<!-- YAML
added: v0.3.3
-->

*   `query`{字符串}要写入的语句或查询`output`，前面附加在
    提示。
*   `options`{对象}
    *   `signal`{中止信号}（可选）允许`question()`要取消
        使用`AbortController`.
*   `callback`{函数}使用用户的
    响应于`query`.

这`rl.question()`方法显示`query`通过将其写入`output`,
等待 提供用户输入`input`，然后调用`callback`
函数将提供的输入作为第一个参数传递。

当被调用时，`rl.question()`将恢复`input`流（如果已）
暂停。

如果`InterfaceConstructor`创建于`output`设置为`null`或
`undefined`这`query`未写入。

这`callback`函数传递给`rl.question()`不遵循典型
接受`Error`对象或`null`作为第一个参数。
这`callback`以提供的答案作为唯一参数进行调用。

如果调用`rl.question()`后`rl.close()`.

用法示例：

```js
rl.question('What is your favorite food? ', (answer) => {
  console.log(`Oh, so your favorite food is ${answer}`);
});
```

使用`AbortController`以取消问题。

```js
const ac = new AbortController();
const signal = ac.signal;

rl.question('What is your favorite food? ', { signal }, (answer) => {
  console.log(`Oh, so your favorite food is ${answer}`);
});

signal.addEventListener('abort', () => {
  console.log('The food question timed out');
}, { once: true });

setTimeout(() => ac.abort(), 10000);
```

### `rl.resume()`

<!-- YAML
added: v0.3.4
-->

这`rl.resume()`方法恢复`input`流（如果已暂停）。

### `rl.setPrompt(prompt)`

<!-- YAML
added: v0.1.98
-->

*   `prompt`{字符串}

这`rl.setPrompt()`方法设置将写入的提示`output`
每当`rl.prompt()`被调用。

### `rl.getPrompt()`

<!-- YAML
added:
  - v15.3.0
  - v14.17.0
-->

*   返回：{字符串} 当前提示字符串

这`rl.getPrompt()`方法返回`rl.prompt()`.

### `rl.write(data[, key])`

<!-- YAML
added: v0.1.98
-->

*   `data`{字符串}
*   `key`{对象}
    *   `ctrl`{布尔值}`true`以指示<kbd>Ctrl</kbd>钥匙。
    *   `meta`{布尔值}`true`以指示<kbd>元</kbd>钥匙。
    *   `shift`{布尔值}`true`以指示<kbd>转变</kbd>钥匙。
    *   `name`{字符串}密钥的名称。

这`rl.write()`方法将写入`data`或识别的关键序列
由`key`到`output`.这`key`参数仅在以下情况下受支持`output`是
一个[断续器][TTY]文本终端。看[TTY 键绑定][TTY keybindings]以获取键列表
组合。

如果`key`指定，`data`被忽略。

当被调用时，`rl.write()`将恢复`input`流（如果已）
暂停。

如果`InterfaceConstructor`创建于`output`设置为`null`或
`undefined`这`data`和`key`未写入。

```js
rl.write('Delete this!');
// Simulate Ctrl+U to delete the line written previously
rl.write(null, { ctrl: true, name: 'u' });
```

这`rl.write()`方法将数据写入`readline` `Interface`的
`input` *就好像它是由用户提供的*.

### `rl[Symbol.asyncIterator]()`

<!-- YAML
added:
 - v11.4.0
 - v10.16.0
changes:
  - version:
     - v11.14.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/26989
    description: Symbol.asyncIterator support is no longer experimental.
-->

*   返回： {AsyncIterator}

创建一个`AsyncIterator`循环访问输入中每一行的对象
以字符串形式流。此方法允许异步迭代
`InterfaceConstructor`对象通过`for await...of`循环。

不会转发输入流中的错误。

如果循环终止为`break`,`throw`或`return`,
[`rl.close()`][rl.close()]将被调用。换句话说，迭代
`InterfaceConstructor`将始终完全使用输入流。

性能无法与传统相提并论`'line'`事件 API。用`'line'`
而是用于对性能敏感的应用程序。

```js
async function processLineByLine() {
  const rl = readline.createInterface({
    // ...
  });

  for await (const line of rl) {
    // Each line in the readline input will be successively available here as
    // `line`.
  }
}
```

`readline.createInterface()`将开始使用一次输入流
调用。在接口创建和之间具有异步操作
异步迭代可能会导致缺少行。

### `rl.line`

<!-- YAML
added: v0.1.98
changes:
  - version:
      - v15.8.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/33676
    description: Value will always be a string, never undefined.
-->

*   {字符串}

节点正在处理的当前输入数据。

当从 TTY 流收集输入以检索
到目前为止已处理的当前值，在`line`事件
正在发出。一旦`line`事件已发出，此属性将
为空字符串。

请注意，在实例运行时修改该值可能会
如果出现意外后果`rl.cursor`不受控制。

**如果不使用 TTY 流进行输入，请使用[`'line'`]['line']事件。**

一个可能的用例如下：

```js
const values = ['lorem ipsum', 'dolor sit amet'];
const rl = readline.createInterface(process.stdin);
const showResults = debounce(() => {
  console.log(
    '\n',
    values.filter((val) => val.startsWith(rl.line)).join(' ')
  );
}, 300);
process.stdin.on('keypress', (c, k) => {
  showResults();
});
```

### `rl.cursor`

<!-- YAML
added: v0.1.98
-->

*   {数字|未定义}

光标相对于`rl.line`.

这将跟踪当前光标在输入字符串中的位置，当
从 TTY 流读取输入。游标的位置决定了
将在处理输入时修改的输入字符串部分，
以及将呈现终端脱字符号的列。

### `rl.getCursorPos()`

<!-- YAML
added:
 - v13.5.0
 - v12.16.0
-->

*   返回： {对象}
    *   `rows`{number} 光标当前落在提示符所在的行
    *   `cols`{数字} 光标当前落在屏幕上的列

返回游标相对于输入的实际位置
提示符 + 字符串。长输入（换行）字符串以及多个
行提示包含在计算中。

## Promises API

<!-- YAML
added: v17.0.0
-->

> 稳定性： 1 - 实验

### 类：`readlinePromises.Interface`

<!-- YAML
added: v17.0.0
-->

*   扩展：{读取行。接口构造函数}

的实例`readlinePromises.Interface`类是使用
`readlinePromises.createInterface()`方法。每个实例都与
单`input` [读][Readable]流和单个`output` [写][Writable]流。
这`output`流用于打印到达的用户输入的提示，
并从中读取，`input`流。

#### `rl.question(query[, options])`

<!-- YAML
added: v17.0.0
-->

*   `query`{字符串}要写入的语句或查询`output`，前面附加在
    提示。
*   `options`{对象}
    *   `signal`{中止信号}（可选）允许`question()`要取消
        使用`AbortSignal`.
*   返回：{承诺} 使用用户的
    响应于`query`.

这`rl.question()`方法显示`query`通过将其写入`output`,
等待 提供用户输入`input`，然后调用`callback`
函数将提供的输入作为第一个参数传递。

当被调用时，`rl.question()`将恢复`input`流（如果已）
暂停。

如果`readlinePromises.Interface`创建于`output`设置为`null`或
`undefined`这`query`未写入。

如果问题在之后被调用`rl.close()`，它将返回被拒绝的承诺。

用法示例：

```mjs
const answer = await rl.question('What is your favorite food? ');
console.log(`Oh, so your favorite food is ${answer}`);
```

使用`AbortSignal`以取消问题。

```mjs
const signal = AbortSignal.timeout(10_000);

signal.addEventListener('abort', () => {
  console.log('The food question timed out');
}, { once: true });

const answer = await rl.question('What is your favorite food? ', { signal });
console.log(`Oh, so your favorite food is ${answer}`);
```

### 类：`readlinePromises.Readline`

<!-- YAML
added: v17.0.0
-->

#### `new readlinePromises.Readline(stream[, options])`

<!-- YAML
added: v17.0.0
-->

*   `stream`{流。可写} A[断续器][TTY]流。
*   `options`{对象}
    *   `autoCommit`{布尔值}如果`true`，无需致电`rl.commit()`.

#### `rl.clearLine(dir)`

<!-- YAML
added: v17.0.0
-->

*   `dir`{整数}
    *   `-1`：从光标向左
    *   `1`：从光标向右
    *   `0`：整条生产线
*   返回：此

这`rl.clearLine()`方法添加到挂起操作的内部列表中
清除关联行的当前行的操作`stream`在指定的
方向由`dir`.
叫`rl.commit()`以查看此方法的效果，除非`autoCommit: true`
已传递给构造函数。

#### `rl.clearScreenDown()`

<!-- YAML
added: v17.0.0
-->

*   返回：此

这`rl.clearScreenDown()`方法添加到挂起操作的内部列表中
从 的当前位置清除关联流的操作
光标向下。
叫`rl.commit()`以查看此方法的效果，除非`autoCommit: true`
已传递给构造函数。

#### `rl.commit()`

<!-- YAML
added: v17.0.0
-->

*   返回： {承诺}

这`rl.commit()`方法将所有挂起的操作发送到关联的
`stream`并清除挂起操作的内部列表。

#### `rl.cursorTo(x[, y])`

<!-- YAML
added: v17.0.0
-->

*   `x`{整数}
*   `y`{整数}
*   返回：此

这`rl.cursorTo()`方法将操作添加到挂起操作的内部列表中
将光标移动到关联位置中的指定位置`stream`.
叫`rl.commit()`以查看此方法的效果，除非`autoCommit: true`
已传递给构造函数。

#### `rl.moveCursor(dx, dy)`

<!-- YAML
added: v17.0.0
-->

*   `dx`{整数}
*   `dy`{整数}
*   返回：此

这`rl.moveCursor()`方法添加到挂起操作的内部列表中
移动光标的操作*相对*到其当前位置
相关`stream`.
叫`rl.commit()`以查看此方法的效果，除非`autoCommit: true`
已传递给构造函数。

#### `rl.rollback()`

<!-- YAML
added: v17.0.0
-->

*   返回：此

这`rl.rollback`方法清除挂起操作的内部列表，而不
将其发送到关联的`stream`.

### `readlinePromises.createInterface(options)`

<!-- YAML
added: v17.0.0
-->

*   `options`{对象}
    *   `input`{流。可读}[读][Readable]要收听的流。此选项
        是*必填*.
    *   `output`{流。可写}[写][Writable]用于写入读线数据的流
        卢武铉
    *   `completer`{函数}用于 Tab 自动完成功能的可选功能。
    *   `terminal`{布尔值}`true`如果`input`和`output`流应该是
        像TTY一样对待，并有ANSI / VT100转义代码写入它。
        **违约：**检查`isTTY`在`output`实例化时流式传输。
    *   `history`{字符串\[]}历史线的初始列表。此选项有意义
        仅当`terminal`设置为`true`由用户或内部`output`
        检查，否则根本不初始化历史记录缓存机制。
        **违约：** `[]`.
    *   `historySize`{数字}保留的最大历史线数。禁用
        历史记录将此值设置为`0`.此选项仅在以下情况下才有意义
        `terminal`设置为`true`由用户或内部`output`检查
        否则，历史记录缓存机制根本不初始化。
        **违约：** `30`.
    *   `removeHistoryDuplicates`{布尔值}如果`true`，当添加了新的输入行时
        到历史记录列表复制较旧的列表，这将删除较旧的行
        从列表中。**违约：** `false`.
    *   `prompt`{字符串}要使用的提示字符串。**违约：** `'> '`.
    *   `crlfDelay`{数字}如果延迟介于`\r`和`\n`超过
        `crlfDelay`毫秒，两者`\r`和`\n`将被视为单独处理
        下线输入。`crlfDelay`将被强制为不小于的数字
        `100`.可以设置为`Infinity`，在这种情况下`\r`其次`\n`
        将始终被视为单个换行符（这可能是合理的
        [读取文件][reading files]跟`\r\n`行分隔符）。**违约：** `100`.
    *   `escapeCodeTimeout`{数字}持续时间`readlinePromises`将等待
        字符（当读取以毫秒为单位的模糊键序列时，
        既可以使用到目前为止读取的输入形成完整的键序列，并且可以
        需要额外的输入来完成更长的键序列）。
        **违约：** `500`.
    *   `tabSize`{整数}制表符等于的空格数（最小值为 1）。
        **违约：** `8`.
*   返回： {readlinePromises.Interface}

这`readlinePromises.createInterface()`方法创建一个新的`readlinePromises.Interface`
实例。

```js
const readlinePromises = require('node:readline/promises');
const rl = readlinePromises.createInterface({
  input: process.stdin,
  output: process.stdout
});
```

一旦`readlinePromises.Interface`实例已创建，最常见的情况
就是要听`'line'`事件：

```js
rl.on('line', (line) => {
  console.log(`Received: ${line}`);
});
```

如果`terminal`是`true`对于此实例，则`output`流将获得
最佳兼容性，如果它定义了`output.columns`属性和发出
一个`'resize'`事件`output`如果或当列发生更改时
([`process.stdout`][process.stdout]当它是TTY时自动执行此操作）。

#### 使用`completer`功能

这`completer`函数采用用户输入的当前行
作为参数，并返回`Array`包含 2 个条目：

*   一`Array`具有匹配的完成条目。
*   用于匹配的子字符串。

例如：`[[substr1, substr2, ...], originalsubstring]`.

```js
function completer(line) {
  const completions = '.help .error .exit .quit .q'.split(' ');
  const hits = completions.filter((c) => c.startsWith(line));
  // Show all completions if none found
  return [hits.length ? hits : completions, line];
}
```

这`completer`函数也可以返回 {Promise}，或者是异步的：

```js
async function completer(linePartial) {
  await someAsyncWork();
  return [['123'], linePartial];
}
```

## 回调接口

<!-- YAML
added: v0.1.104
-->

### 类：`readline.Interface`

<!-- YAML
added: v0.1.104
changes:
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/37947
    description: The class `readline.Interface` now inherits from `Interface`.
-->

*   扩展：{读取行。接口构造函数}

的实例`readline.Interface`类是使用
`readline.createInterface()`方法。每个实例都与
单`input` [读][Readable]流和单个`output` [写][Writable]流。
这`output`流用于打印到达的用户输入的提示，
并从中读取，`input`流。

#### `rl.question(query[, options], callback)`

<!-- YAML
added: v0.3.3
-->

*   `query`{字符串}要写入的语句或查询`output`，前面附加在
    提示。
*   `options`{对象}
    *   `signal`{中止信号}（可选）允许`question()`要取消
        使用`AbortController`.
*   `callback`{函数}使用用户的
    响应于`query`.

这`rl.question()`方法显示`query`通过将其写入`output`,
等待 提供用户输入`input`，然后调用`callback`
函数将提供的输入作为第一个参数传递。

当被调用时，`rl.question()`将恢复`input`流（如果已）
暂停。

如果`readline.Interface`创建于`output`设置为`null`或
`undefined`这`query`未写入。

这`callback`函数传递给`rl.question()`不遵循典型
接受`Error`对象或`null`作为第一个参数。
这`callback`以提供的答案作为唯一参数进行调用。

如果调用`rl.question()`后`rl.close()`.

用法示例：

```js
rl.question('What is your favorite food? ', (answer) => {
  console.log(`Oh, so your favorite food is ${answer}`);
});
```

使用`AbortController`以取消问题。

```js
const ac = new AbortController();
const signal = ac.signal;

rl.question('What is your favorite food? ', { signal }, (answer) => {
  console.log(`Oh, so your favorite food is ${answer}`);
});

signal.addEventListener('abort', () => {
  console.log('The food question timed out');
}, { once: true });

setTimeout(() => ac.abort(), 10000);
```

### `readline.clearLine(stream, dir[, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28674
    description: The stream's write() callback and return value are exposed.
-->

*   `stream`{流。可写}
*   `dir`{数字}
    *   `-1`：从光标向左
    *   `1`：从光标向右
    *   `0`：整条生产线
*   `callback`{函数}操作完成后调用。
*   返回：{布尔值}`false`如果`stream`希望呼叫代码等待
    这`'drain'`在继续写入其他数据之前要发出的事件;
    否则`true`.

这`readline.clearLine()`方法清除给定的当前行[断续器][TTY]流
在指定方向上由`dir`.

### `readline.clearScreenDown(stream[, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28641
    description: The stream's write() callback and return value are exposed.
-->

*   `stream`{流。可写}
*   `callback`{函数}操作完成后调用。
*   返回：{布尔值}`false`如果`stream`希望呼叫代码等待
    这`'drain'`在继续写入其他数据之前要发出的事件;
    否则`true`.

这`readline.clearScreenDown()`方法清除给定的[断续器][TTY]流自
光标的当前位置向下。

### `readline.createInterface(options)`

<!-- YAML
added: v0.1.98
changes:
  - version:
      - v15.14.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37932
    description: The `signal` option is supported now.
  - version:
      - v15.8.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/33662
    description: The `history` option is supported now.
  - version: v13.9.0
    pr-url: https://github.com/nodejs/node/pull/31318
    description: The `tabSize` option is supported now.
  - version:
    - v8.3.0
    - v6.11.4
    pr-url: https://github.com/nodejs/node/pull/13497
    description: Remove max limit of `crlfDelay` option.
  - version: v6.6.0
    pr-url: https://github.com/nodejs/node/pull/8109
    description: The `crlfDelay` option is supported now.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/7125
    description: The `prompt` option is supported now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6352
    description: The `historySize` option can be `0` now.
-->

*   `options`{对象}
    *   `input`{流。可读}[读][Readable]要收听的流。此选项
        是*必填*.
    *   `output`{流。可写}[写][Writable]用于写入读线数据的流
        自。
    *   `completer`{函数}用于 Tab 自动完成功能的可选功能。
    *   `terminal`{布尔值}`true`如果`input`和`output`流应该是
        像TTY一样对待，并有ANSI / VT100转义代码写入它。
        **违约：**检查`isTTY`在`output`实例化时流式传输。
    *   `history`{字符串\[]}历史线的初始列表。此选项有意义
        仅当`terminal`设置为`true`由用户或内部`output`
        检查，否则根本不初始化历史记录缓存机制。
        **违约：** `[]`.
    *   `historySize`{数字}保留的最大历史线数。禁用
        历史记录将此值设置为`0`.此选项仅在以下情况下才有意义
        `terminal`设置为`true`由用户或内部`output`检查
        否则，历史记录缓存机制根本不初始化。
        **违约：** `30`.
    *   `removeHistoryDuplicates`{布尔值}如果`true`，当添加了新的输入行时
        到历史记录列表复制较旧的列表，这将删除较旧的行
        从列表中。**违约：** `false`.
    *   `prompt`{字符串}要使用的提示字符串。**违约：** `'> '`.
    *   `crlfDelay`{数字}如果延迟介于`\r`和`\n`超过
        `crlfDelay`毫秒，两者`\r`和`\n`将被视为单独处理
        下线输入。`crlfDelay`将被强制为不小于的数字
        `100`.可以设置为`Infinity`，在这种情况下`\r`其次`\n`
        将始终被视为单个换行符（这可能是合理的
        [读取文件][reading files]跟`\r\n`行分隔符）。**违约：** `100`.
    *   `escapeCodeTimeout`{数字}持续时间`readline`将等待
        字符（当读取以毫秒为单位的模糊键序列时，
        既可以使用到目前为止读取的输入形成完整的键序列，并且可以
        需要额外的输入来完成更长的键序列）。
        **违约：** `500`.
    *   `tabSize`{整数}制表符等于的空格数（最小值为 1）。
        **违约：** `8`.
    *   `signal`{中止信号}允许使用中止信号关闭接口。
        中止信号将在内部调用`close`在界面上。
*   返回：{读行。接口}

这`readline.createInterface()`方法创建一个新的`readline.Interface`
实例。

```js
const readline = require('node:readline');
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});
```

一旦`readline.Interface`实例已创建，最常见的情况是
收听`'line'`事件：

```js
rl.on('line', (line) => {
  console.log(`Received: ${line}`);
});
```

如果`terminal`是`true`对于此实例，则`output`流将获得
最佳兼容性，如果它定义了`output.columns`属性和发出
一个`'resize'`事件`output`如果或当列发生更改时
([`process.stdout`][process.stdout]当它是TTY时自动执行此操作）。

创建`readline.Interface`用`stdin`作为输入，程序
在收到[EOF 字符][EOF character].退出时不带
等待用户输入，调用`process.stdin.unref()`.

#### 使用`completer`功能

这`completer`函数采用用户输入的当前行
作为参数，并返回`Array`包含 2 个条目：

*   一`Array`具有匹配的完成条目。
*   用于匹配的子字符串。

例如：`[[substr1, substr2, ...], originalsubstring]`.

```js
function completer(line) {
  const completions = '.help .error .exit .quit .q'.split(' ');
  const hits = completions.filter((c) => c.startsWith(line));
  // Show all completions if none found
  return [hits.length ? hits : completions, line];
}
```

这`completer`如果函数接受两个，则可以异步调用函数
参数：

```js
function completer(linePartial, callback) {
  callback(null, [['123'], linePartial]);
}
```

### `readline.cursorTo(stream, x[, y][, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28674
    description: The stream's write() callback and return value are exposed.
-->

*   `stream`{流。可写}
*   `x`{数字}
*   `y`{数字}
*   `callback`{函数}操作完成后调用。
*   返回：{布尔值}`false`如果`stream`希望呼叫代码等待
    这`'drain'`在继续写入其他数据之前要发出的事件;
    否则`true`.

这`readline.cursorTo()`方法将光标移动到
鉴于[断续器][TTY] `stream`.

### `readline.moveCursor(stream, dx, dy[, callback])`

<!-- YAML
added: v0.7.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28674
    description: The stream's write() callback and return value are exposed.
-->

*   `stream`{流。可写}
*   `dx`{数字}
*   `dy`{数字}
*   `callback`{函数}操作完成后调用。
*   返回：{布尔值}`false`如果`stream`希望呼叫代码等待
    这`'drain'`在继续写入其他数据之前要发出的事件;
    否则`true`.

这`readline.moveCursor()`方法移动光标*相对*到其当前
给定位置[断续器][TTY] `stream`.

## `readline.emitKeypressEvents(stream[, interface])`

<!-- YAML
added: v0.7.7
-->

*   `stream`{流。可读}
*   `interface`{读线。接口构造函数}

这`readline.emitKeypressEvents()`方法导致给定[读][Readable]
流开始发射`'keypress'`与接收的输入对应的事件。

选择`interface`指定`readline.Interface`其中的实例
检测到复制粘贴的输入时，将自动完成功能被禁用。

如果`stream`是一个[断续器][TTY]，则它必须处于原始模式。

这由其上的任何读线实例自动调用`input`如果
`input`是终端。关闭`readline`实例不停止
这`input`从发射`'keypress'`事件。

```js
readline.emitKeypressEvents(process.stdin);
if (process.stdin.isTTY)
  process.stdin.setRawMode(true);
```

## 示例：微型 CLI

以下示例说明了`readline.Interface`类到
实现一个小型命令行界面：

```js
const readline = require('node:readline');
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  prompt: 'OHAI> '
});

rl.prompt();

rl.on('line', (line) => {
  switch (line.trim()) {
    case 'hello':
      console.log('world!');
      break;
    default:
      console.log(`Say what? I might have heard '${line.trim()}'`);
      break;
  }
  rl.prompt();
}).on('close', () => {
  console.log('Have a great day!');
  process.exit(0);
});
```

## 示例：逐行读取文件流

一个常见的用例`readline`是在一行处使用输入文件
时间。要做到这一点，最简单的方法是利用[`fs.ReadStream`][fs.ReadStream]API as
以及作为`for await...of`圈：

```js
const fs = require('node:fs');
const readline = require('node:readline');

async function processLineByLine() {
  const fileStream = fs.createReadStream('input.txt');

  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity
  });
  // Note: we use the crlfDelay option to recognize all instances of CR LF
  // ('\r\n') in input.txt as a single line break.

  for await (const line of rl) {
    // Each line in input.txt will be successively available here as `line`.
    console.log(`Line from file: ${line}`);
  }
}

processLineByLine();
```

或者，可以使用[`'line'`]['line']事件：

```js
const fs = require('node:fs');
const readline = require('node:readline');

const rl = readline.createInterface({
  input: fs.createReadStream('sample.txt'),
  crlfDelay: Infinity
});

rl.on('line', (line) => {
  console.log(`Line from file: ${line}`);
});
```

现在`for await...of`循环可能会慢一点。如果`async`/`await`
流量和速度都是必不可少的，可以应用混合方法：

```js
const { once } = require('node:events');
const { createReadStream } = require('node:fs');
const { createInterface } = require('node:readline');

(async function processLineByLine() {
  try {
    const rl = createInterface({
      input: createReadStream('big-file.txt'),
      crlfDelay: Infinity
    });

    rl.on('line', (line) => {
      // Process the line.
    });

    await once(rl, 'close');

    console.log('File processed.');
  } catch (err) {
    console.error(err);
  }
})();
```

## TTY 键绑定

<table>
  <tr>
    <th>Keybindings</th>
    <th>Description</th>
    <th>Notes</th>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Backspace</kbd></td>
    <td>Delete line left</td>
    <td>Doesn't work on Linux, Mac and Windows</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Delete</kbd></td>
    <td>Delete line right</td>
    <td>Doesn't work on Mac</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>C</kbd></td>
    <td>Emit <code>SIGINT</code> or close the readline instance</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>H</kbd></td>
    <td>Delete left</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>D</kbd></td>
    <td>Delete right or close the readline instance in case the current line is empty / EOF</td>
    <td>Doesn't work on Windows</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>U</kbd></td>
    <td>Delete from the current position to the line start</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>K</kbd></td>
    <td>Delete from the current position to the end of line</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Y</kbd></td>
    <td>Yank (Recall) the previously deleted text</td>
    <td>Only works with text deleted by <kbd>Ctrl</kbd>+<kbd>U</kbd> or <kbd>Ctrl</kbd>+<kbd>K</kbd></td>
  </tr>
  <tr>
    <td><kbd>Meta</kbd>+<kbd>Y</kbd></td>
    <td>Cycle among previously deleted lines</td>
    <td>Only available when the last keystroke is <kbd>Ctrl</kbd>+<kbd>Y</kbd></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>A</kbd></td>
    <td>Go to start of line</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>E</kbd></td>
    <td>Go to end of line</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>B</kbd></td>
    <td>Back one character</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>F</kbd></td>
    <td>Forward one character</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>L</kbd></td>
    <td>Clear screen</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>N</kbd></td>
    <td>Next history item</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>P</kbd></td>
    <td>Previous history item</td>
    <td></td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>-</kbd></td>
    <td>Undo previous change</td>
    <td>Any keystroke that emits key code <code>0x1F</code> will do this action.
    In many terminals, for example <code>xterm</code>,
    this is bound to <kbd>Ctrl</kbd>+<kbd>-</kbd>.</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>6</kbd></td>
    <td>Redo previous change</td>
    <td>Many terminals don't have a default redo keystroke.
    We choose key code <code>0x1E</code> to perform redo.
    In <code>xterm</code>, it is bound to <kbd>Ctrl</kbd>+<kbd>6</kbd>
    by default.</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Z</kbd></td>
    <td>Moves running process into background. Type
    <code>fg</code> and press <kbd>Enter</kbd>
    to return.</td>
    <td>Doesn't work on Windows</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>W</kbd> or <kbd>Ctrl</kbd>
   +<kbd>Backspace</kbd></td>
    <td>Delete backward to a word boundary</td>
    <td><kbd>Ctrl</kbd>+<kbd>Backspace</kbd> Doesn't
    work on Linux, Mac and Windows</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Delete</kbd></td>
    <td>Delete forward to a word boundary</td>
    <td>Doesn't work on Mac</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Left arrow</kbd> or
    <kbd>Meta</kbd>+<kbd>B</kbd></td>
    <td>Word left</td>
    <td><kbd>Ctrl</kbd>+<kbd>Left arrow</kbd> Doesn't work
    on Mac</td>
  </tr>
  <tr>
    <td><kbd>Ctrl</kbd>+<kbd>Right arrow</kbd> or
    <kbd>Meta</kbd>+<kbd>F</kbd></td>
    <td>Word right</td>
    <td><kbd>Ctrl</kbd>+<kbd>Right arrow</kbd> Doesn't work
    on Mac</td>
  </tr>
  <tr>
    <td><kbd>Meta</kbd>+<kbd>D</kbd> or <kbd>Meta</kbd>
   +<kbd>Delete</kbd></td>
    <td>Delete word right</td>
    <td><kbd>Meta</kbd>+<kbd>Delete</kbd> Doesn't work
    on windows</td>
  </tr>
  <tr>
    <td><kbd>Meta</kbd>+<kbd>Backspace</kbd></td>
    <td>Delete word left</td>
    <td>Doesn't work on Mac</td>
  </tr>
</table>

[EOF character]: https://en.wikipedia.org/wiki/End-of-file#EOF_character

[Readable]: stream.md#readable-streams

[TTY]: tty.md

[TTY keybindings]: #tty-keybindings

[Writable]: stream.md#writable-streams

[`'SIGCONT'`]: #event-sigcont

[`'SIGTSTP'`]: #event-sigtstp

[`'line'`]: #event-line

[`fs.ReadStream`]: fs.md#class-fsreadstream

[`process.stdin`]: process.md#processstdin

[`process.stdout`]: process.md#processstdout

[`rl.close()`]: #rlclose

[reading files]: #example-read-file-stream-line-by-line
