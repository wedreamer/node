# 错误

<!--introduced_in=v4.0.0-->

<!--type=misc-->

在 Node.js 中运行的应用程序通常会遇到四类
错误：

*   Standard JavaScript errors， such {EvalError}， {SyntaxError}， {RangeError}，
    {ReferenceError}， {TypeError}， and {URIError}.
*   由底层操作系统约束触发的系统错误，例如
    尝试打开不存在的文件或尝试发送数据
    在封闭的套接字上。
*   由应用程序代码触发的用户指定错误。
*   `AssertionError`s 是一类特殊的错误，可在以下情况下触发
    Node.js 检测不应发生的特殊逻辑冲突。这些
    通常由`node:assert`模块。

由 Node 引发的所有 JavaScript 和系统错误.js继承自或
的实例，标准 JavaScript {Error} 类，并得到保证
提供*至少*该类上可用的属性。

## 错误传播和拦截

<!--type=misc-->

Node.js支持多种传播和处理错误的机制
在应用程序运行时发生。如何报告这些错误以及
处理完全取决于的类型`Error`以及 API 的样式
叫。

所有 JavaScript 错误都作为异常处理*马上*生成
并使用标准 JavaScript 引发错误`throw`机制。这些
使用[`try…catch`构建][try-catch]由
JavaScript 语言。

```js
// Throws with a ReferenceError because z is not defined.
try {
  const m = 1;
  const n = m + z;
} catch (err) {
  // Handle the error here.
}
```

任何使用 JavaScript`throw`机制将引发一个异常
*必须*使用`try…catch`或者节点.js进程将退出
马上。

除少数例外情况外，*同步*API（任何不这样做的阻止方法）
接受`callback`函数，例如[`fs.readFileSync`][fs.readFileSync]），将使用`throw`
以报告错误。

在*异步接口*可能以多种方式报告：

*   大多数接受`callback`函数将接受
    `Error`对象作为第一个参数传递给该函数。如果那先
    参数不是`null`并且是 的一个实例`Error`，然后发生错误
    这应该得到处理。

    <!-- eslint-disable no-useless-return -->

    ```js
    const fs = require('node:fs');
    fs.readFile('a file that does not exist', (err, data) => {
      if (err) {
        console.error('There was an error reading the file!', err);
        return;
      }
      // Otherwise handle the data
    });
    ```

*   当在对象上调用异步方法时，该对象是
    [`EventEmitter`][EventEmitter]，错误可以路由到该对象的`'error'`事件。

    ```js
    const net = require('node:net');
    const connection = net.connect('localhost');

    // Adding an 'error' event handler to a stream:
    connection.on('error', (err) => {
      // If the connection is reset by the server, or if it can't
      // connect at all, or on any sort of error encountered by
      // the connection, the error will be sent here.
      console.error(err);
    });

    connection.pipe(process.stdout);
    ```

*   Node.js API 中的少数典型异步方法可能仍存在
    使用`throw`引发必须使用
    `try…catch`.没有关于这些方法的全面清单;请
    请参阅每种方法的文档以确定适当的
    需要错误处理机制。

用途`'error'`事件机制最常见于[基于流][stream-based]
和[基于事件发射器][event emitter-based]API，它们本身代表一系列
随时间变化的异步操作（与可能
通过或失败）。

为*都* [`EventEmitter`][EventEmitter]对象，如果`'error'`事件处理程序不是
则将引发错误，导致 Node.js 进程报告
未捕获的异常和崩溃，除非：[`domain`][domains]模块是
适当地使用或已为
[`'uncaughtException'`]['uncaughtException']事件。

```js
const EventEmitter = require('node:events');
const ee = new EventEmitter();

setImmediate(() => {
  // This will crash the process because no 'error' event
  // handler has been added.
  ee.emit('error', new Error('This will crash'));
});
```

以这种方式生成的错误*不能*使用`try…catch`如
他们被扔掉*后*调用代码已退出。

开发人员必须参考每种方法的文档来确定
这些方法引发的错误是如何传播的。

### 错误优先回调

<!--type=misc-->

Node.js核心 API 公开的大多数异步方法都遵循一个惯用语
模式称为*错误优先回调*.使用此模式，回调
函数作为参数传递给方法。当操作
完成或引发错误，调用回调函数时
`Error`对象（如果有）作为第一个参数传递。如果未引发任何错误，
第一个参数将传递为`null`.

```js
const fs = require('node:fs');

function errorFirstCallback(err, data) {
  if (err) {
    console.error('There was an error', err);
    return;
  }
  console.log(data);
}

fs.readFile('/some/file/that/does-not-exist', errorFirstCallback);
fs.readFile('/some/file/that/does-exist', errorFirstCallback);
```

The JavaScript`try…catch`机制**不能**用于拦截错误
由异步 API 生成。初学者的一个常见错误是尝试
用`throw`在错误优先回调中：

```js
// THIS WILL NOT WORK:
const fs = require('node:fs');

try {
  fs.readFile('/some/file/that/does-not-exist', (err, data) => {
    // Mistaken assumption: throwing here...
    if (err) {
      throw err;
    }
  });
} catch (err) {
  // This will not catch the throw!
  console.error(err);
}
```

这将不起作用，因为回调函数传递给`fs.readFile()`是
异步调用。在调用回调时，
周围的代码，包括`try…catch`块，将已经退出。
在回调中引发错误**可能会使节点崩溃.js进程**在大多数
例。如果[域][domains]已启用，或者已将处理程序注册为
`process.on('uncaughtException')`，则可以截获此类错误。

## 类：`Error`

<!--type=class-->

一个通用的 JavaScript {Error} 对象，不表示任何特定的
错误发生原因的情况。`Error`对象捕获“堆栈跟踪”
详细说明代码中`Error`已实例化，并且可能
提供错误的文本描述。

Node.js生成的所有错误，包括所有系统和 JavaScript 错误，
将是 的实例或继承自`Error`类。

### `new Error(message[, options])`

*   `message`{字符串}
*   `options`{对象}
    *   `cause`{任何}导致新创建错误的错误。

创建新的`Error`对象并设置`error.message`属性到
提供短信。如果对象作为传递`message`，短信
通过调用生成`String(message)`.如果`cause`提供选项，
它被分配给`error.cause`财产。这`error.stack`属性将
表示代码中`new Error()`被调用。堆栈跟踪
依赖于[V8 的堆栈跟踪 API][V8's stack trace API].堆栈跟踪仅扩展到
（一）开头*同步代码执行*，或 （b） 帧数
由物业提供`Error.stackTraceLimit`，以较小者为准。

### `Error.captureStackTrace(targetObject[, constructorOpt])`

*   `targetObject`{对象}
*   `constructorOpt`{函数}

创建`.stack`属性`targetObject`，当被访问时返回
一个字符串，表示代码中的位置
`Error.captureStackTrace()`被调用。

```js
const myObject = {};
Error.captureStackTrace(myObject);
myObject.stack;  // Similar to `new Error().stack`
```

跟踪的第一行将以
`${myObject.name}: ${myObject.message}`.

可选`constructorOpt`参数接受函数。如果给定，则所有帧
以上`constructorOpt`包括`constructorOpt`，将从 中省略
生成的堆栈跟踪。

这`constructorOpt`参数对于隐藏实现很有用
用户生成的错误的详细信息。例如：

```js
function MyError() {
  Error.captureStackTrace(this, MyError);
}

// Without passing MyError to captureStackTrace, the MyError
// frame would show up in the .stack property. By passing
// the constructor, we omit that frame, and retain all frames below it.
new MyError().stack;
```

### `Error.stackTraceLimit`

*   {数字}

这`Error.stackTraceLimit`属性指定堆栈帧数
由堆栈跟踪收集（是否由`new Error().stack`或
`Error.captureStackTrace(obj)`).

默认值为`10`但可以设置为任何有效的 JavaScript 编号。变化
将影响捕获的任何堆栈跟踪*后*该值已更改。

如果设置为非数字值或设置为负数，则堆栈跟踪将
不捕获任何帧。

### `error.cause`

<!-- YAML
added: v16.9.0
-->

*   {任何}

如果存在，则`error.cause`属性是导致`Error`.
当捕获错误并抛出具有不同错误的新错误时，使用它
消息或代码，以便仍可以访问原始错误。

这`error.cause`属性通常通过调用来设置
`new Error(message, { cause })`.它不由构造函数设置，如果
`cause`未提供选项。

此属性允许链接错误。序列化时`Error`对象
[`util.inspect()`][util.inspect()]递归序列化`error.cause`如果已设置。

```js
const cause = new Error('The remote HTTP server responded with a 500 status');
const symptom = new Error('The message failed to send', { cause });

console.log(symptom);
// Prints:
//   Error: The message failed to send
//       at REPL2:1:17
//       at Script.runInThisContext (node:vm:130:12)
//       ... 7 lines matching cause stack trace ...
//       at [_line] [as _line] (node:internal/readline/interface:886:18) {
//     [cause]: Error: The remote HTTP server responded with a 500 status
//         at REPL1:1:15
//         at Script.runInThisContext (node:vm:130:12)
//         at REPLServer.defaultEval (node:repl:574:29)
//         at bound (node:domain:426:15)
//         at REPLServer.runBound [as eval] (node:domain:437:12)
//         at REPLServer.onLine (node:repl:902:10)
//         at REPLServer.emit (node:events:549:35)
//         at REPLServer.emit (node:domain:482:12)
//         at [_onLine] [as _onLine] (node:internal/readline/interface:425:12)
//         at [_line] [as _line] (node:internal/readline/interface:886:18)
```

### `error.code`

*   {字符串}

这`error.code`属性是标识错误类型的字符串标签。
`error.code`是识别错误的最稳定方法。它只会改变
在 Node 的主要版本之间.js。相比之下，`error.message`字符串可能
在任何版本的 Node.js之间更改。看[节点.js错误代码][Node.js error codes]了解详情
关于特定代码。

### `error.message`

*   {字符串}

这`error.message`属性是错误设置的字符串描述
叫`new Error(message)`.这`message`传递给构造函数也会
出现在堆栈跟踪的第一行`Error`，但正在更改
此属性之后`Error`对象已创建*可能不是*更改第一个
堆栈跟踪的行（例如，当`error.stack`在此之前阅读
属性已更改）。

```js
const err = new Error('The message');
console.error(err.message);
// Prints: The message
```

### `error.stack`

*   {字符串}

这`error.stack`属性是一个字符串，用于描述代码中
这`Error`已实例化。

```console
Error: Things keep happening!
   at /home/gbusey/file.js:525:2
   at Frobnicator.refrobulate (/home/gbusey/business-logic.js:424:21)
   at Actor.<anonymous> (/home/gbusey/actors.js:400:8)
   at increaseSynergy (/home/gbusey/actors.js:701:6)
```

第一行的格式设置为`<error class name>: <error message>`和
后跟一系列堆栈帧（每行以“at”开头）。
每个帧描述代码中导致错误的调用站点
生成。V8 尝试为每个函数显示一个名称（按变量名称、
函数名称或对象方法名称），但有时它将无法
找到一个合适的名字。如果 V8 无法确定函数的名称，则仅
将显示该帧的位置信息。否则，
确定的函数名称将显示，并附加位置信息
在括号中。

帧仅为 JavaScript 函数生成。例如，如果执行
同步传递一个C++插件函数，称为`cheetahify`哪
本身调用一个 JavaScript 函数，该帧表示`cheetahify`叫
将不存在于堆栈跟踪中：

```js
const cheetahify = require('./native-binding.node');

function makeFaster() {
  // `cheetahify()` *synchronously* calls speedy.
  cheetahify(function speedy() {
    throw new Error('oh no!');
  });
}

makeFaster();
// will throw:
//   /home/gbusey/file.js:6
//       throw new Error('oh no!');
//           ^
//   Error: oh no!
//       at speedy (/home/gbusey/file.js:6:11)
//       at makeFaster (/home/gbusey/file.js:5:3)
//       at Object.<anonymous> (/home/gbusey/file.js:10:1)
//       at Module._compile (module.js:456:26)
//       at Object.Module._extensions..js (module.js:474:10)
//       at Module.load (module.js:356:32)
//       at Function.Module._load (module.js:312:12)
//       at Function.Module.runMain (module.js:497:10)
//       at startup (node.js:119:16)
//       at node.js:906:3
```

位置信息将是以下信息之一：

*   `native`，如果帧表示 V8 内部的调用（如`[].forEach`).
*   `plain-filename.js:line:column`，如果框架表示内部调用
    到节点.js。
*   `/absolute/path/to/file.js:line:column`，如果框架表示
    用户程序（使用 CommonJS 模块系统）或其依赖项。
*   `<transport-protocol>:///url/to/module/file.mjs:line:column`，如果框架
    表示用户程序中的调用（使用 ES 模块系统），或者
    它的依赖关系。

表示堆栈跟踪的字符串在
`error.stack`属性是**访问**.

堆栈跟踪捕获的帧数以
`Error.stackTraceLimit`或当前事件的可用帧数
循环滴答声。

## 类：`AssertionError`

*   扩展：{错误。错误}

指示断言失败。有关详细信息，请参阅
[`Class: assert.AssertionError`][Class: assert.AssertionError].

## 类：`RangeError`

*   扩展：{错误。错误}

指示提供的参数不在 的集合或范围内
函数的可接受值;无论是数字范围，还是
在给定函数参数的选项集之外。

```js
require('node:net').connect(-1);
// Throws "RangeError: "port" option should be >= 0 and < 65536: -1"
```

节点.js将生成并抛出`RangeError`实例*马上*作为形式
参数验证。

## 类：`ReferenceError`

*   扩展：{错误。错误}

指示正在尝试访问未访问的变量
定义。此类错误通常表示代码中存在拼写错误，或者其他错误已损坏
程序。

虽然客户端代码可能会生成并传播这些错误，但实际上，只有 V8
会这样做。

```js
doesNotExist;
// Throws ReferenceError, doesNotExist is not a variable in this program.
```

除非应用程序正在动态生成和运行代码，
`ReferenceError`实例指示代码或其依赖项中的 bug。

## 类：`SyntaxError`

*   扩展：{错误。错误}

指示程序不是有效的 JavaScript。这些错误可能只是
作为代码评估的结果生成和传播。代码评估可能
由于以下原因而发生`eval`,`Function`,`require`或[虚拟机][vm].这些错误
几乎总是表明程序已损坏。

```js
try {
  require('node:vm').runInThisContext('binary ! isNotOk');
} catch (err) {
  // 'err' will be a SyntaxError.
}
```

`SyntaxError`实例在创建它们的上下文中是不可恢复的 –
它们可能只会被其他上下文捕获。

## 类：`SystemError`

*   扩展：{错误。错误}

Node.js在其运行时内发生异常时生成系统错误
环境。这些通常发生在应用程序违反操作时
系统约束。例如，如果应用程序
尝试读取不存在的文件。

*   `address`{字符串}如果存在，则为网络连接的地址
    失败
*   `code`{字符串}字符串错误代码
*   `dest`{字符串}如果存在，则为报告文件时的文件路径目标
    系统错误
*   `errno`{数字}系统提供的错误号
*   `info`{对象}如果存在，则提供有关错误条件的额外详细信息
*   `message`{字符串}系统提供人类可读的错误描述
*   `path`{字符串}如果存在，则为报告文件系统错误时的文件路径
*   `port`{数字}如果存在，则为不可用的网络连接端口
*   `syscall`{字符串}触发错误的系统调用的名称

### `error.address`

*   {字符串}

如果存在，`error.address`是一个字符串，描述
网络连接失败。

### `error.code`

*   {字符串}

这`error.code`属性是表示错误代码的字符串。

### `error.dest`

*   {字符串}

如果存在，`error.dest`是报告文件时的文件路径目标
系统错误。

### `error.errno`

*   {数字}

这`error.errno`属性是一个负数，它对应于
到 中定义的错误代码[`libuv Error handling`][libuv Error handling].

在 Windows 上，系统提供的错误号将由 libuv 规范化。

若要获取错误代码的字符串表示形式，请使用
[`util.getSystemErrorName(error.errno)`][util.getSystemErrorName(error.errno)].

### `error.info`

*   {对象}

如果存在，`error.info`是包含有关错误条件的详细信息的对象。

### `error.message`

*   {字符串}

`error.message`是系统提供的人类可读的错误描述。

### `error.path`

*   {字符串}

如果存在，`error.path`是包含相关无效路径名的字符串。

### `error.port`

*   {数字}

如果存在，`error.port`是不可用的网络连接端口。

### `error.syscall`

*   {字符串}

这`error.syscall`属性是描述[系统调用][syscall]失败了。

### 常见系统错误

这是编写 Node 时经常遇到的系统错误的列表.js
程序。有关完整列表，请参阅[`errno`（3） 手册页][errno(3) man page].

*   `EACCES`（权限被拒绝）：尝试以某种方式访问文件
    被其文件访问权限禁止。

*   `EADDRINUSE`（地址已在使用）：尝试绑定服务器
    ([`net`][net],[`http`][http]或[`https`][https]） 到本地地址失败，原因是
    本地系统上已占用该地址的另一台服务器。

*   `ECONNREFUSED`（连接被拒绝）：无法建立连接，因为
    目标计算机主动拒绝了它。这通常是由于尝试
    连接到外部主机上处于非活动状态的服务。

*   `ECONNRESET`（连接被对等方重置）：连接被强行关闭
    对等体。这通常是由于遥控器上的连接丢失造成的
    套接字由于超时或重新启动。通常通过[`http`][http]
    和[`net`][net]模块。

*   `EEXIST`（文件存在）：现有文件是以下操作的目标：
    要求目标不存在。

*   `EISDIR`（是目录）：操作需要一个文件，但给定
    路径名是一个目录。

*   `EMFILE`（系统中打开的文件过多）：最大
    [文件描述符][file descriptors]已达到系统上的允许，并且
    对另一个描述符的请求在至少一个描述符之前无法满足
    已关闭。在 中一次打开多个文件时会遇到这种情况
    并行，特别是在系统（特别是macOS）上，其中存在低
    进程的文件描述符限制。要修正下限，请运行
    `ulimit -n 2048`在将运行 Node.js 进程的同一 shell 中。

*   `ENOENT`（无此类文件或目录）：通常由[`fs`][fs]操作
    以指示指定路径名的组件不存在。不
    实体（文件或目录）可以通过给定的路径找到。

*   `ENOTDIR`（不是目录）：给定路径名的组件存在，但
    不是预期的目录。通常由[`fs.readdir`][fs.readdir].

*   `ENOTEMPTY`（目录不为空）：目标为包含条目的目录
    需要空目录的操作，通常[`fs.unlink`][fs.unlink].

*   `ENOTFOUND`（DNS 查找失败）：指示 DNS 故障
    `EAI_NODATA`或`EAI_NONAME`.这不是一个标准的 POSIX 错误。

*   `EPERM`（不允许操作）：尝试执行
    需要提升权限的操作。

*   `EPIPE`（管道损坏）：在管道、套接字或 FIFO 上写入，而存在
    没有读取数据的进程。常见于[`net`][net]和
    [`http`][http]层，指示流的远程一侧
    写入 已关闭。

*   `ETIMEDOUT`（操作超时）：连接或发送请求失败，因为
    关联方在一段时间后没有正确响应。通常
    遇到[`http`][http]或[`net`][net].通常是一个迹象，表明`socket.end()`
    未正确调用。

## 类：`TypeError`

*   扩展 {错误。错误}

指示提供的参数不是允许的类型。例如
将函数传递给一个参数，该参数期望字符串将是`TypeError`.

```js
require('node:url').parse(() => { });
// Throws TypeError, since it expected a string.
```

节点.js将生成并抛出`TypeError`实例*马上*作为形式
参数验证。

## 异常与错误

<!--type=misc-->

JavaScript 异常是由于无效而引发的值
操作或作为目标`throw`陈述。虽然不是必需的
这些值是`Error`或继承自
`Error`，Node.js或 JavaScript 运行时引发的所有异常*将*是
的实例`Error`.

一些例外情况是*不可恢复*在 JavaScript 层。此类例外情况
将*总是*导致节点.js进程崩溃。示例包括`assert()`
检查或`abort()`C++层中的调用。

## 打开 SSL 错误

源自`crypto`或`tls`属于类`Error`，并且除了
标准`.code`和`.message`属性，可能有一些额外的
特定于 OpenSSL 的属性。

### `error.opensslErrorStack`

一组错误，可以为 OpenSSL 库中的位置提供上下文
错误源自。

### `error.function`

错误源自的 OpenSSL 函数。

### `error.library`

错误源自的 OpenSSL 库。

### `error.reason`

描述错误原因的用户可读字符串。

<a id="nodejs-error-codes"></a>

## 节点.js错误代码

<a id="ABORT_ERR"></a>

### `ABORT_ERR`

<!-- YAML
added: v15.0.0
-->

在操作中止时使用（通常使用`AbortController`).

蜜蜂属*不*用`AbortSignal`s 通常不会对此代码引发错误。

此代码不使用常规`ERR_*`约定 节点.js 错误使用
为了与网络平台的`AbortError`.

<a id="ERR_AMBIGUOUS_ARGUMENT"></a>

### `ERR_AMBIGUOUS_ARGUMENT`

函数参数的使用方式表明该函数
签名可能被误解。这是由`node:assert`模块时
这`message`参数`assert.throws(block, message)`匹配错误
消息由抛出`block`因为这种用法表明用户相信
`message`是预期的消息，而不是消息`AssertionError`
将显示在以下情况下`block`不投掷。

<a id="ERR_ARG_NOT_ITERABLE"></a>

### `ERR_ARG_NOT_ITERABLE`

可迭代参数（即`for...of`循环） 是
必需的，但未提供给节点.js API。

<a id="ERR_ASSERTION"></a>

### `ERR_ASSERTION`

一种特殊类型的错误，只要 Node.js 检测到
不应发生的特殊逻辑冲突。这些通常被提出
由`node:assert`模块。

<a id="ERR_ASYNC_CALLBACK"></a>

### `ERR_ASYNC_CALLBACK`

尝试将不属于函数的内容注册为
`AsyncHooks`回调。

<a id="ERR_ASYNC_TYPE"></a>

### `ERR_ASYNC_TYPE`

异步资源的类型无效。用户还可以
以定义自己的类型（如果使用公共嵌入器 API）。

<a id="ERR_BROTLI_COMPRESSION_FAILED"></a>

### `ERR_BROTLI_COMPRESSION_FAILED`

传递到 Brotli 流的数据未成功压缩。

<a id="ERR_BROTLI_INVALID_PARAM"></a>

### `ERR_BROTLI_INVALID_PARAM`

在构造 Brotli 流期间传递了无效的参数键。

<a id="ERR_BUFFER_CONTEXT_NOT_AVAILABLE"></a>

### `ERR_BUFFER_CONTEXT_NOT_AVAILABLE`

已尝试创建节点.js`Buffer`来自插件或嵌入器的实例
代码，而在未与节点关联的 JS 引擎上下文中.js
实例。传递给`Buffer`方法将被释放
到方法返回时。

遇到此错误时，创建`Buffer`
实例是创建一个正常`Uint8Array`，其唯一区别在于
生成的对象的原型。`Uint8Array`s 在全部中普遍接受
节点.js核心 API，其中`Buffer`s 是;它们在所有上下文中都可用。

<a id="ERR_BUFFER_OUT_OF_BOUNDS"></a>

### `ERR_BUFFER_OUT_OF_BOUNDS`

超出`Buffer`已尝试。

<a id="ERR_BUFFER_TOO_LARGE"></a>

### `ERR_BUFFER_TOO_LARGE`

已尝试创建`Buffer`大于允许的最大值
大小。

<a id="ERR_CANNOT_WATCH_SIGINT"></a>

### `ERR_CANNOT_WATCH_SIGINT`

节点.js 无法监视`SIGINT`信号。

<a id="ERR_CHILD_CLOSED_BEFORE_REPLY"></a>

### `ERR_CHILD_CLOSED_BEFORE_REPLY`

子进程在父进程收到答复之前已关闭。

<a id="ERR_CHILD_PROCESS_IPC_REQUIRED"></a>

### `ERR_CHILD_PROCESS_IPC_REQUIRED`

在未指定 IPC 通道的情况下分叉子进程时使用。

<a id="ERR_CHILD_PROCESS_STDIO_MAXBUFFER"></a>

### `ERR_CHILD_PROCESS_STDIO_MAXBUFFER`

当主进程尝试从子进程的
STDERR/STDOUT，并且数据的长度比`maxBuffer`选择。

<a id="ERR_CLOSED_MESSAGE_PORT"></a>

### `ERR_CLOSED_MESSAGE_PORT`

<!--
added:
  - v16.2.0
  - v14.17.1
changes:
  - version: 11.12.0
    pr-url: https://github.com/nodejs/node/pull/26487
    description: The error message was removed.
  - version:
      - v16.2.0
      - v14.17.1
    pr-url: https://github.com/nodejs/node/pull/38510
    description: The error message was reintroduced.
-->

有人试图使用`MessagePort`已关闭的实例
状态，通常在`.close()`已被调用。

<a id="ERR_CONSOLE_WRITABLE_STREAM"></a>

### `ERR_CONSOLE_WRITABLE_STREAM`

`Console`在没有实例化的情况下`stdout`流，或`Console`具有
不可写`stdout`或`stderr`流。

<a id="ERR_CONSTRUCT_CALL_INVALID"></a>

### `ERR_CONSTRUCT_CALL_INVALID`

<!--
added: v12.5.0
-->

调用了不可调用的类构造函数。

<a id="ERR_CONSTRUCT_CALL_REQUIRED"></a>

### `ERR_CONSTRUCT_CALL_REQUIRED`

类的构造函数在未`new`.

<a id="ERR_CONTEXT_NOT_INITIALIZED"></a>

### `ERR_CONTEXT_NOT_INITIALIZED`

传递到 API 中的 vm 上下文尚未初始化。这可能会发生
当在创建期间发生（并捕获）错误时
上下文，例如，当分配失败或最大调用堆栈时
创建上下文时达到大小。

<a id="ERR_CRYPTO_CUSTOM_ENGINE_NOT_SUPPORTED"></a>

### `ERR_CRYPTO_CUSTOM_ENGINE_NOT_SUPPORTED`

请求的客户端证书引擎不受版本支持
的 OpenSSL 正在使用。

<a id="ERR_CRYPTO_ECDH_INVALID_FORMAT"></a>

### `ERR_CRYPTO_ECDH_INVALID_FORMAT`

的无效值`format`参数已传递给`crypto.ECDH()`
类`getPublicKey()`方法。

<a id="ERR_CRYPTO_ECDH_INVALID_PUBLIC_KEY"></a>

### `ERR_CRYPTO_ECDH_INVALID_PUBLIC_KEY`

的无效值`key`参数已传递给
`crypto.ECDH()`类`computeSecret()`方法。这意味着公众
键位于椭圆曲线之外。

<a id="ERR_CRYPTO_ENGINE_UNKNOWN"></a>

### `ERR_CRYPTO_ENGINE_UNKNOWN`

无效的加密引擎标识符已传递到
[`require('node:crypto').setEngine()`][require('node:crypto').setEngine()].

<a id="ERR_CRYPTO_FIPS_FORCED"></a>

### `ERR_CRYPTO_FIPS_FORCED`

这[`--force-fips`][--force-fips]使用了命令行参数，但进行了尝试
以在`node:crypto`模块。

<a id="ERR_CRYPTO_FIPS_UNAVAILABLE"></a>

### `ERR_CRYPTO_FIPS_UNAVAILABLE`

尝试启用或禁用 FIPS 模式，但未启用 FIPS 模式
可用。

<a id="ERR_CRYPTO_HASH_FINALIZED"></a>

### `ERR_CRYPTO_HASH_FINALIZED`

[`hash.digest()`][hash.digest()]被多次调用。这`hash.digest()`方法必须
每个实例的调用次数不超过一次`Hash`对象。

<a id="ERR_CRYPTO_HASH_UPDATE_FAILED"></a>

### `ERR_CRYPTO_HASH_UPDATE_FAILED`

[`hash.update()`][hash.update()]由于任何原因而失败。这应该很少发生，如果有的话。

<a id="ERR_CRYPTO_INCOMPATIBLE_KEY"></a>

### `ERR_CRYPTO_INCOMPATIBLE_KEY`

给定的加密密钥与尝试的操作不兼容。

<a id="ERR_CRYPTO_INCOMPATIBLE_KEY_OPTIONS"></a>

### `ERR_CRYPTO_INCOMPATIBLE_KEY_OPTIONS`

选定的公钥或私钥编码与其他选项不兼容。

<a id="ERR_CRYPTO_INITIALIZATION_FAILED"></a>

### `ERR_CRYPTO_INITIALIZATION_FAILED`

<!-- YAML
added: v15.0.0
-->

加密子系统的初始化失败。

<a id="ERR_CRYPTO_INVALID_AUTH_TAG"></a>

### `ERR_CRYPTO_INVALID_AUTH_TAG`

<!-- YAML
added: v15.0.0
-->

提供的身份验证标记无效。

<a id="ERR_CRYPTO_INVALID_COUNTER"></a>

### `ERR_CRYPTO_INVALID_COUNTER`

<!-- YAML
added: v15.0.0
-->

为计数器模式密码提供了无效的计数器。

<a id="ERR_CRYPTO_INVALID_CURVE"></a>

### `ERR_CRYPTO_INVALID_CURVE`

<!-- YAML
added: v15.0.0
-->

提供了无效的椭圆曲线。

<a id="ERR_CRYPTO_INVALID_DIGEST"></a>

### `ERR_CRYPTO_INVALID_DIGEST`

无效[加密摘要算法][crypto digest algorithm]已指定。

<a id="ERR_CRYPTO_INVALID_IV"></a>

### `ERR_CRYPTO_INVALID_IV`

<!-- YAML
added: v15.0.0
-->

提供了无效的初始化向量。

<a id="ERR_CRYPTO_INVALID_JWK"></a>

### `ERR_CRYPTO_INVALID_JWK`

<!-- YAML
added: v15.0.0
-->

提供了无效的 JSON Web 密钥。

<a id="ERR_CRYPTO_INVALID_KEY_OBJECT_TYPE"></a>

### `ERR_CRYPTO_INVALID_KEY_OBJECT_TYPE`

给定的加密密钥对象的类型对于尝试的操作无效。

<a id="ERR_CRYPTO_INVALID_KEYLEN"></a>

### `ERR_CRYPTO_INVALID_KEYLEN`

<!-- YAML
added: v15.0.0
-->

提供的密钥长度无效。

<a id="ERR_CRYPTO_INVALID_KEYPAIR"></a>

### `ERR_CRYPTO_INVALID_KEYPAIR`

<!-- YAML
added: v15.0.0
-->

提供的密钥对无效。

<a id="ERR_CRYPTO_INVALID_KEYTYPE"></a>

### `ERR_CRYPTO_INVALID_KEYTYPE`

<!-- YAML
added: v15.0.0
-->

提供的密钥类型无效。

<a id="ERR_CRYPTO_INVALID_MESSAGELEN"></a>

### `ERR_CRYPTO_INVALID_MESSAGELEN`

<!-- YAML
added: v15.0.0
-->

提供的消息长度无效。

<a id="ERR_CRYPTO_INVALID_SCRYPT_PARAMS"></a>

### `ERR_CRYPTO_INVALID_SCRYPT_PARAMS`

<!-- YAML
added: v15.0.0
-->

提供的 scrypt 算法参数无效。

<a id="ERR_CRYPTO_INVALID_STATE"></a>

### `ERR_CRYPTO_INVALID_STATE`

对处于无效状态的对象使用了加密方法。为
实例， 调用[`cipher.getAuthTag()`][cipher.getAuthTag()]致电前`cipher.final()`.

<a id="ERR_CRYPTO_INVALID_TAG_LENGTH"></a>

### `ERR_CRYPTO_INVALID_TAG_LENGTH`

<!-- YAML
added: v15.0.0
-->

提供的身份验证标记长度无效。

<a id="ERR_CRYPTO_JOB_INIT_FAILED"></a>

### `ERR_CRYPTO_JOB_INIT_FAILED`

<!-- YAML
added: v15.0.0
-->

异步加密操作的初始化失败。

<a id="ERR_CRYPTO_JWK_UNSUPPORTED_CURVE"></a>

### `ERR_CRYPTO_JWK_UNSUPPORTED_CURVE`

Key 的椭圆曲线未注册用于
[JSON Web Key 椭圆曲线注册表][JSON Web Key Elliptic Curve Registry].

<a id="ERR_CRYPTO_JWK_UNSUPPORTED_KEY_TYPE"></a>

### `ERR_CRYPTO_JWK_UNSUPPORTED_KEY_TYPE`

密钥的非对称密钥类型未注册用于
[JSON Web 项类型注册表][JSON Web Key Types Registry].

<a id="ERR_CRYPTO_OPERATION_FAILED"></a>

### `ERR_CRYPTO_OPERATION_FAILED`

<!-- YAML
added: v15.0.0
-->

加密操作由于其他未指定的原因而失败。

<a id="ERR_CRYPTO_PBKDF2_ERROR"></a>

### `ERR_CRYPTO_PBKDF2_ERROR`

PBKDF2 算法由于未指定的原因而失败。OpenSSL 不提供
更多细节，因此Node.js也没有。

<a id="ERR_CRYPTO_SCRYPT_INVALID_PARAMETER"></a>

### `ERR_CRYPTO_SCRYPT_INVALID_PARAMETER`

一个或多个[`crypto.scrypt()`][crypto.scrypt()]或[`crypto.scryptSync()`][crypto.scryptSync()]参数为
超出其法律范围。

<a id="ERR_CRYPTO_SCRYPT_NOT_SUPPORTED"></a>

### `ERR_CRYPTO_SCRYPT_NOT_SUPPORTED`

节点.js编译时没有`scrypt`支持。与官方不可能
发布二进制文件，但可以通过自定义版本（包括发行版内部版本）进行。

<a id="ERR_CRYPTO_SIGN_KEY_REQUIRED"></a>

### `ERR_CRYPTO_SIGN_KEY_REQUIRED`

签名`key`未提供给[`sign.sign()`][sign.sign()]方法。

<a id="ERR_CRYPTO_TIMING_SAFE_EQUAL_LENGTH"></a>

### `ERR_CRYPTO_TIMING_SAFE_EQUAL_LENGTH`

[`crypto.timingSafeEqual()`][crypto.timingSafeEqual()]被调用`Buffer`,`TypedArray`或
`DataView`不同长度的参数。

<a id="ERR_CRYPTO_UNKNOWN_CIPHER"></a>

### `ERR_CRYPTO_UNKNOWN_CIPHER`

指定了未知密码。

<a id="ERR_CRYPTO_UNKNOWN_DH_GROUP"></a>

### `ERR_CRYPTO_UNKNOWN_DH_GROUP`

一个未知的Diffie-Hellman小组名称被赋予了。看
[`crypto.getDiffieHellman()`][crypto.getDiffieHellman()]以获取有效组名的列表。

<a id="ERR_CRYPTO_UNSUPPORTED_OPERATION"></a>

### `ERR_CRYPTO_UNSUPPORTED_OPERATION`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

尝试调用不受支持的加密操作。

<a id="ERR_DEBUGGER_ERROR"></a>

### `ERR_DEBUGGER_ERROR`

<!-- YAML
added:
  - v16.4.0
  - v14.17.4
-->

的 发生错误[调试器][debugger].

<a id="ERR_DEBUGGER_STARTUP_ERROR"></a>

### `ERR_DEBUGGER_STARTUP_ERROR`

<!-- YAML
added:
  - v16.4.0
  - v14.17.4
-->

这[调试器][debugger]等待所需的主机/端口空闲时超时。

<a id="ERR_DLOPEN_DISABLED"></a>

### `ERR_DLOPEN_DISABLED`

<!-- YAML
added:
  - v16.10.0
  - v14.19.0
-->

已禁用加载本机插件[`--no-addons`][--no-addons].

<a id="ERR_DLOPEN_FAILED"></a>

### `ERR_DLOPEN_FAILED`

<!-- YAML
added: v15.0.0
-->

调用`process.dlopen()`失败。

<a id="ERR_DIR_CLOSED"></a>

### `ERR_DIR_CLOSED`

这[`fs.Dir`][fs.Dir]以前已关闭。

<a id="ERR_DIR_CONCURRENT_OPERATION"></a>

### `ERR_DIR_CONCURRENT_OPERATION`

<!-- YAML
added: v14.3.0
-->

尝试对[`fs.Dir`][fs.Dir]它具有
正在进行的异步操作。

<a id="ERR_DNS_SET_SERVERS_FAILED"></a>

### `ERR_DNS_SET_SERVERS_FAILED`

`c-ares`无法设置 DNS 服务器。

<a id="ERR_DOMAIN_CALLBACK_NOT_AVAILABLE"></a>

### `ERR_DOMAIN_CALLBACK_NOT_AVAILABLE`

这`node:domain`模块不可用，因为它无法建立
需要错误处理钩子，因为
[`process.setUncaughtExceptionCaptureCallback()`][process.setUncaughtExceptionCaptureCallback()]曾被叫到
较早的时间点。

<a id="ERR_DOMAIN_CANNOT_SET_UNCAUGHT_EXCEPTION_CAPTURE"></a>

### `ERR_DOMAIN_CANNOT_SET_UNCAUGHT_EXCEPTION_CAPTURE`

[`process.setUncaughtExceptionCaptureCallback()`][process.setUncaughtExceptionCaptureCallback()]无法调用
因为`node:domain`模块已在较早的时间点加载。

堆栈跟踪已扩展为包括
`node:domain`模块已加载。

<a id="ERR_DUPLICATE_STARTUP_SNAPSHOT_MAIN_FUNCTION"></a>

### `ERR_DUPLICATE_STARTUP_SNAPSHOT_MAIN_FUNCTION`

[`v8.startupSnapshot.setDeserializeMainFunction()`][v8.startupSnapshot.setDeserializeMainFunction()]无法调用
因为它以前已经被调用过。

<a id="ERR_ENCODING_INVALID_ENCODED_DATA"></a>

### `ERR_ENCODING_INVALID_ENCODED_DATA`

提供给的数据`TextDecoder()`根据编码，API 无效
提供。

<a id="ERR_ENCODING_NOT_SUPPORTED"></a>

### `ERR_ENCODING_NOT_SUPPORTED`

编码提供给`TextDecoder()`API 不是其中之一
[WHATWG 支持的编码][WHATWG Supported Encodings].

<a id="ERR_EVAL_ESM_CANNOT_PRINT"></a>

### `ERR_EVAL_ESM_CANNOT_PRINT`

`--print`不能与 ESM 输入一起使用。

<a id="ERR_EVENT_RECURSION"></a>

### `ERR_EVENT_RECURSION`

在尝试以递归方式调度事件时抛出`EventTarget`.

<a id="ERR_EXECUTION_ENVIRONMENT_NOT_AVAILABLE"></a>

### `ERR_EXECUTION_ENVIRONMENT_NOT_AVAILABLE`

JS 执行上下文不与节点.js环境相关联。
当 Node.js 用作嵌入式库和某些钩子时，可能会发生这种情况
对于JS引擎没有正确设置。

<a id="ERR_FALSY_VALUE_REJECTION"></a>

### `ERR_FALSY_VALUE_REJECTION`

一个`Promise`已通过以下方式回调`util.callbackify()`被拒绝与
虚假价值。

<a id="ERR_FEATURE_UNAVAILABLE_ON_PLATFORM"></a>

### `ERR_FEATURE_UNAVAILABLE_ON_PLATFORM`

<!-- YAML
added: v14.0.0
-->

在功能不可用时使用
到当前运行 Node.js 平台。

<a id="ERR_FS_CP_DIR_TO_NON_DIR"></a>

### `ERR_FS_CP_DIR_TO_NON_DIR`

<!--
added: v16.7.0
-->

尝试将目录复制到非目录（文件、符号链接、
等）用[`fs.cp()`][fs.cp()].

<a id="ERR_FS_CP_EEXIST"></a>

### `ERR_FS_CP_EEXIST`

<!--
added: v16.7.0
-->

尝试复制已存在的文件
[`fs.cp()`][fs.cp()]，其中`force`和`errorOnExist`设置为`true`.

<a id="ERR_FS_CP_EINVAL"></a>

### `ERR_FS_CP_EINVAL`

<!--
added: v16.7.0
-->

使用时[`fs.cp()`][fs.cp()],`src`或`dest`指向无效路径。

<a id="ERR_FS_CP_FIFO_PIPE"></a>

### `ERR_FS_CP_FIFO_PIPE`

<!--
added: v16.7.0
-->

尝试复制命名管道[`fs.cp()`][fs.cp()].

<a id="ERR_FS_CP_NON_DIR_TO_DIR"></a>

### `ERR_FS_CP_NON_DIR_TO_DIR`

<!--
added: v16.7.0
-->

尝试将非目录（文件、符号链接等）复制到目录
用[`fs.cp()`][fs.cp()].

<a id="ERR_FS_CP_SOCKET"></a>

### `ERR_FS_CP_SOCKET`

<!--
added: v16.7.0
-->

尝试将[`fs.cp()`][fs.cp()].

<a id="ERR_FS_CP_SYMLINK_TO_SUBDIRECTORY"></a>

### `ERR_FS_CP_SYMLINK_TO_SUBDIRECTORY`

<!--
added: v16.7.0
-->

使用时[`fs.cp()`][fs.cp()]，符号链接`dest`指向子目录
之`src`.

<a id="ERR_FS_CP_UNKNOWN"></a>

### `ERR_FS_CP_UNKNOWN`

<!--
added: v16.7.0
-->

尝试将复制到未知文件类型[`fs.cp()`][fs.cp()].

<a id="ERR_FS_EISDIR"></a>

### `ERR_FS_EISDIR`

路径是一个目录。

<a id="ERR_FS_FILE_TOO_LARGE"></a>

### `ERR_FS_FILE_TOO_LARGE`

已尝试读取大小大于最大值的文件
允许的大小`Buffer`.

<a id="ERR_FS_INVALID_SYMLINK_TYPE"></a>

### `ERR_FS_INVALID_SYMLINK_TYPE`

无效的符号链接类型已传递给[`fs.symlink()`][fs.symlink()]或
[`fs.symlinkSync()`][fs.symlinkSync()]方法。

<a id="ERR_HTTP_HEADERS_SENT"></a>

### `ERR_HTTP_HEADERS_SENT`

尝试在发送标头后添加更多标头。

<a id="ERR_HTTP_INVALID_HEADER_VALUE"></a>

### `ERR_HTTP_INVALID_HEADER_VALUE`

指定的 HTTP 标头值无效。

<a id="ERR_HTTP_INVALID_STATUS_CODE"></a>

### `ERR_HTTP_INVALID_STATUS_CODE`

状态代码超出常规状态代码范围 （100-999）。

<a id="ERR_HTTP_REQUEST_TIMEOUT"></a>

### `ERR_HTTP_REQUEST_TIMEOUT`

客户端未在允许的时间内发送整个请求。

<a id="ERR_HTTP_SOCKET_ENCODING"></a>

### `ERR_HTTP_SOCKET_ENCODING`

不允许更改套接字编码[RFC 7230 第 3 节][RFC 7230 Section 3].

<a id="ERR_HTTP_TRAILER_INVALID"></a>

### `ERR_HTTP_TRAILER_INVALID`

这`Trailer`标头已设置，即使传输编码不支持
那。

<a id="ERR_HTTP2_ALTSVC_INVALID_ORIGIN"></a>

### `ERR_HTTP2_ALTSVC_INVALID_ORIGIN`

HTTP/2 ALTSVC 帧需要有效的源。

<a id="ERR_HTTP2_ALTSVC_LENGTH"></a>

### `ERR_HTTP2_ALTSVC_LENGTH`

HTTP/2 ALTSVC 帧限制为最多 16，382 个有效负载字节。

<a id="ERR_HTTP2_CONNECT_AUTHORITY"></a>

### `ERR_HTTP2_CONNECT_AUTHORITY`

对于使用`CONNECT`方法，`:authority`伪标头
是必需的。

<a id="ERR_HTTP2_CONNECT_PATH"></a>

### `ERR_HTTP2_CONNECT_PATH`

对于使用`CONNECT`方法，`:path`伪标头是
禁止。

<a id="ERR_HTTP2_CONNECT_SCHEME"></a>

### `ERR_HTTP2_CONNECT_SCHEME`

对于使用`CONNECT`方法，`:scheme`伪标头是
禁止。

<a id="ERR_HTTP2_ERROR"></a>

### `ERR_HTTP2_ERROR`

发生了非特定的 HTTP/2 错误。

<a id="ERR_HTTP2_GOAWAY_SESSION"></a>

### `ERR_HTTP2_GOAWAY_SESSION`

新的 HTTP/2 流在`Http2Session`已收到
`GOAWAY`来自连接的对等体的帧。

<a id="ERR_HTTP2_HEADER_SINGLE_VALUE"></a>

### `ERR_HTTP2_HEADER_SINGLE_VALUE`

为 HTTP/2 标头字段提供了多个值，该值需要
只有一个值。

<a id="ERR_HTTP2_HEADERS_AFTER_RESPOND"></a>

### `ERR_HTTP2_HEADERS_AFTER_RESPOND`

在启动 HTTP/2 响应后指定了其他标头。

<a id="ERR_HTTP2_HEADERS_SENT"></a>

### `ERR_HTTP2_HEADERS_SENT`

尝试发送多个响应标头。

<a id="ERR_HTTP2_INFO_STATUS_NOT_ALLOWED"></a>

### `ERR_HTTP2_INFO_STATUS_NOT_ALLOWED`

信息性 HTTP 状态代码 （`1xx`） 可能未设置为响应状态
HTTP/2 响应上的代码。

<a id="ERR_HTTP2_INVALID_CONNECTION_HEADERS"></a>

### `ERR_HTTP2_INVALID_CONNECTION_HEADERS`

禁止在 HTTP/2 中使用特定于 HTTP/1 的标头
请求和响应。

<a id="ERR_HTTP2_INVALID_HEADER_VALUE"></a>

### `ERR_HTTP2_INVALID_HEADER_VALUE`

指定的 HTTP/2 标头值无效。

<a id="ERR_HTTP2_INVALID_INFO_STATUS"></a>

### `ERR_HTTP2_INVALID_INFO_STATUS`

指定了无效的 HTTP 信息状态代码。信息
状态代码必须是介于`100`和`199`（包括）。

<a id="ERR_HTTP2_INVALID_ORIGIN"></a>

### `ERR_HTTP2_INVALID_ORIGIN`

HTTP/2`ORIGIN`帧需要有效的源。

<a id="ERR_HTTP2_INVALID_PACKED_SETTINGS_LENGTH"></a>

### `ERR_HTTP2_INVALID_PACKED_SETTINGS_LENGTH`

输入`Buffer`和`Uint8Array`传递给
`http2.getUnpackedSettings()`API 的长度必须是
六。

<a id="ERR_HTTP2_INVALID_PSEUDOHEADER"></a>

### `ERR_HTTP2_INVALID_PSEUDOHEADER`

只有有效的 HTTP/2 伪标题 （`:status`,`:path`,`:authority`,`:scheme`,
和`:method`） 可能被使用。

<a id="ERR_HTTP2_INVALID_SESSION"></a>

### `ERR_HTTP2_INVALID_SESSION`

对`Http2Session`已存在的对象
摧毁。

<a id="ERR_HTTP2_INVALID_SETTING_VALUE"></a>

### `ERR_HTTP2_INVALID_SETTING_VALUE`

为 HTTP/2 设置指定了无效值。

<a id="ERR_HTTP2_INVALID_STREAM"></a>

### `ERR_HTTP2_INVALID_STREAM`

对已销毁的流执行了操作。

<a id="ERR_HTTP2_MAX_PENDING_SETTINGS_ACK"></a>

### `ERR_HTTP2_MAX_PENDING_SETTINGS_ACK`

每当一个HTTP /2`SETTINGS`帧被发送到连接的对等体，该对等体是
需要发送确认已收到并应用新的
`SETTINGS`.默认情况下，未确认的最大数量`SETTINGS`帧可能
在任何给定时间发送。当该限制已达到此限制时，将使用此错误代码
达到。

<a id="ERR_HTTP2_NESTED_PUSH"></a>

### `ERR_HTTP2_NESTED_PUSH`

尝试从推送流中启动新的推送流。
不允许嵌套推送流。

<a id="ERR_HTTP2_NO_MEM"></a>

### `ERR_HTTP2_NO_MEM`

使用 时内存不足`http2session.setLocalWindowSize(windowSize)`应用程序接口。

<a id="ERR_HTTP2_NO_SOCKET_MANIPULATION"></a>

### `ERR_HTTP2_NO_SOCKET_MANIPULATION`

尝试直接操作（读取、写入、暂停、恢复等）a
套接字连接到`Http2Session`.

<a id="ERR_HTTP2_ORIGIN_LENGTH"></a>

### `ERR_HTTP2_ORIGIN_LENGTH`

HTTP/2`ORIGIN`帧的长度限制为 16382 字节。

<a id="ERR_HTTP2_OUT_OF_STREAMS"></a>

### `ERR_HTTP2_OUT_OF_STREAMS`

在单个 HTTP/2 会话上创建的流数达到最大值
限制。

<a id="ERR_HTTP2_PAYLOAD_FORBIDDEN"></a>

### `ERR_HTTP2_PAYLOAD_FORBIDDEN`

为 HTTP 响应代码指定了消息负载，其有效负载为
禁止。

<a id="ERR_HTTP2_PING_CANCEL"></a>

### `ERR_HTTP2_PING_CANCEL`

HTTP/2 ping 已取消。

<a id="ERR_HTTP2_PING_LENGTH"></a>

### `ERR_HTTP2_PING_LENGTH`

HTTP/2 ping 有效负载的长度必须正好为 8 个字节。

<a id="ERR_HTTP2_PSEUDOHEADER_NOT_ALLOWED"></a>

### `ERR_HTTP2_PSEUDOHEADER_NOT_ALLOWED`

HTTP/2 伪标头使用不当。伪标头是标头
以`:`前缀。

<a id="ERR_HTTP2_PUSH_DISABLED"></a>

### `ERR_HTTP2_PUSH_DISABLED`

已尝试创建推送流，该推送流已被
客户。

<a id="ERR_HTTP2_SEND_FILE"></a>

### `ERR_HTTP2_SEND_FILE`

已尝试使用`Http2Stream.prototype.responseWithFile()`接口到
发送目录。

<a id="ERR_HTTP2_SEND_FILE_NOSEEK"></a>

### `ERR_HTTP2_SEND_FILE_NOSEEK`

已尝试使用`Http2Stream.prototype.responseWithFile()`接口到
发送常规文件以外的内容，但`offset`或`length`选项是
提供。

<a id="ERR_HTTP2_SESSION_ERROR"></a>

### `ERR_HTTP2_SESSION_ERROR`

这`Http2Session`以非零错误代码关闭。

<a id="ERR_HTTP2_SETTINGS_CANCEL"></a>

### `ERR_HTTP2_SETTINGS_CANCEL`

这`Http2Session`设置已取消。

<a id="ERR_HTTP2_SOCKET_BOUND"></a>

### `ERR_HTTP2_SOCKET_BOUND`

已尝试连接`Http2Session`对象到`net.Socket`或
`tls.TLSSocket`已经绑定到另一个`Http2Session`对象。

<a id="ERR_HTTP2_SOCKET_UNBOUND"></a>

### `ERR_HTTP2_SOCKET_UNBOUND`

已尝试使用`socket`的属性`Http2Session`那
已关闭。

<a id="ERR_HTTP2_STATUS_101"></a>

### `ERR_HTTP2_STATUS_101`

使用`101`在 HTTP/2 中禁止信息状态代码。

<a id="ERR_HTTP2_STATUS_INVALID"></a>

### `ERR_HTTP2_STATUS_INVALID`

指定的 HTTP 状态代码无效。状态代码必须是整数
之间`100`和`599`（包括）。

<a id="ERR_HTTP2_STREAM_CANCEL"></a>

### `ERR_HTTP2_STREAM_CANCEL`

一`Http2Stream`在任何数据传输到连接的之前被销毁
同辈。

<a id="ERR_HTTP2_STREAM_ERROR"></a>

### `ERR_HTTP2_STREAM_ERROR`

在`RST_STREAM`框架。

<a id="ERR_HTTP2_STREAM_SELF_DEPENDENCY"></a>

### `ERR_HTTP2_STREAM_SELF_DEPENDENCY`

为 HTTP/2 流设置优先级时，该流可能被标记为
父流的依赖项。当尝试
用于标记流并依赖于自身。

<a id="ERR_HTTP2_TOO_MANY_INVALID_FRAMES"></a>

### `ERR_HTTP2_TOO_MANY_INVALID_FRAMES`

<!--
added: v15.14.0
-->

对等体发送的可接受的无效 HTTP/2 协议帧的限制，
通过`maxSessionInvalidFrames`选项，已超出。

<a id="ERR_HTTP2_TRAILERS_ALREADY_SENT"></a>

### `ERR_HTTP2_TRAILERS_ALREADY_SENT`

尾随标头已在 上发送`Http2Stream`.

<a id="ERR_HTTP2_TRAILERS_NOT_READY"></a>

### `ERR_HTTP2_TRAILERS_NOT_READY`

这`http2stream.sendTrailers()`方法不能调用，直到
`'wantTrailers'`事件在`Http2Stream`对象。这
`'wantTrailers'`仅当`waitForTrailers`选择
设置为`Http2Stream`.

<a id="ERR_HTTP2_UNSUPPORTED_PROTOCOL"></a>

### `ERR_HTTP2_UNSUPPORTED_PROTOCOL`

`http2.connect()`传递了一个 URL，该 URL 使用除以下协议以外的任何协议`http:`或
`https:`.

<a id="ERR_ILLEGAL_CONSTRUCTOR"></a>

### `ERR_ILLEGAL_CONSTRUCTOR`

尝试使用非公共构造函数构造对象。

<a id="ERR_IMPORT_ASSERTION_TYPE_FAILED"></a>

### `ERR_IMPORT_ASSERTION_TYPE_FAILED`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

导入断言失败，导致无法导入指定的模块。

<a id="ERR_IMPORT_ASSERTION_TYPE_MISSING"></a>

### `ERR_IMPORT_ASSERTION_TYPE_MISSING`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

缺少导入断言，从而阻止导入指定的模块。

<a id="ERR_IMPORT_ASSERTION_TYPE_UNSUPPORTED"></a>

### `ERR_IMPORT_ASSERTION_TYPE_UNSUPPORTED`

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

此版本的 Node.js 不支持导入断言。

<a id="ERR_INCOMPATIBLE_OPTION_PAIR"></a>

### `ERR_INCOMPATIBLE_OPTION_PAIR`

选项对彼此不兼容，不能同时使用
时间。

<a id="ERR_INPUT_TYPE_NOT_ALLOWED"></a>

### `ERR_INPUT_TYPE_NOT_ALLOWED`

> 稳定性： 1 - 实验

这`--input-type`标志用于尝试执行文件。此标志可以
仅用于输入通过`--eval`,`--print`或`STDIN`.

<a id="ERR_INSPECTOR_ALREADY_ACTIVATED"></a>

### `ERR_INSPECTOR_ALREADY_ACTIVATED`

使用`node:inspector`模块中，已尝试激活
检查器，当它已经开始侦听端口时。用`inspector.close()`
在不同的地址上激活它之前。

<a id="ERR_INSPECTOR_ALREADY_CONNECTED"></a>

### `ERR_INSPECTOR_ALREADY_CONNECTED`

使用`node:inspector`模块，尝试在
检查员已连接。

<a id="ERR_INSPECTOR_CLOSED"></a>

### `ERR_INSPECTOR_CLOSED`

使用`node:inspector`模块中，尝试使用
会话结束后的检查器已关闭。

<a id="ERR_INSPECTOR_COMMAND"></a>

### `ERR_INSPECTOR_COMMAND`

通过 发出命令时出错`node:inspector`模块。

<a id="ERR_INSPECTOR_NOT_ACTIVE"></a>

### `ERR_INSPECTOR_NOT_ACTIVE`

这`inspector`在以下情况下不处于活动状态`inspector.waitForDebugger()`被调用。

<a id="ERR_INSPECTOR_NOT_AVAILABLE"></a>

### `ERR_INSPECTOR_NOT_AVAILABLE`

这`node:inspector`模块不可用。

<a id="ERR_INSPECTOR_NOT_CONNECTED"></a>

### `ERR_INSPECTOR_NOT_CONNECTED`

使用`node:inspector`模块中，尝试使用
连接之前的检查器。

<a id="ERR_INSPECTOR_NOT_WORKER"></a>

### `ERR_INSPECTOR_NOT_WORKER`

在主线程上调用了一个 API，该 API 只能从
工作线程。

<a id="ERR_INTERNAL_ASSERTION"></a>

### `ERR_INTERNAL_ASSERTION`

Node 中存在错误.js或者 Node.js内部的不正确用法。
要修复此错误，请在<https://github.com/nodejs/node/issues>.

<a id="ERR_INVALID_ADDRESS_FAMILY"></a>

### `ERR_INVALID_ADDRESS_FAMILY`

Node.js API 无法理解提供的地址系列。

<a id="ERR_INVALID_ARG_TYPE"></a>

### `ERR_INVALID_ARG_TYPE`

错误类型的参数被传递给 Node.js API。

<a id="ERR_INVALID_ARG_VALUE"></a>

### `ERR_INVALID_ARG_VALUE`

为给定参数传递了无效或不受支持的值。

<a id="ERR_INVALID_ASYNC_ID"></a>

### `ERR_INVALID_ASYNC_ID`

无效`asyncId`或`triggerAsyncId`已通过使用`AsyncHooks`.一个标识
小于 -1 应该永远不会发生。

<a id="ERR_INVALID_BUFFER_SIZE"></a>

### `ERR_INVALID_BUFFER_SIZE`

在 上执行交换`Buffer`但它的大小与
操作。

<a id="ERR_INVALID_CHAR"></a>

### `ERR_INVALID_CHAR`

在标头中检测到无效字符。

<a id="ERR_INVALID_CURSOR_POS"></a>

### `ERR_INVALID_CURSOR_POS`

给定流上的游标不能移动到指定的行，而没有
指定的列。

<a id="ERR_INVALID_FD"></a>

### `ERR_INVALID_FD`

文件描述符 （'fd'） 无效（例如，它是负值）。

<a id="ERR_INVALID_FD_TYPE"></a>

### `ERR_INVALID_FD_TYPE`

文件描述符 （'fd'） 类型无效。

<a id="ERR_INVALID_FILE_URL_HOST"></a>

### `ERR_INVALID_FILE_URL_HOST`

使用.js节点的 API`file:`网址（例如
[`fs`][fs]模块） 遇到主机不兼容的文件 URL。这
这种情况只能发生在类Unix系统上，其中`localhost`或空
支持主机。

<a id="ERR_INVALID_FILE_URL_PATH"></a>

### `ERR_INVALID_FILE_URL_PATH`

使用.js节点的 API`file:`网址（例如
[`fs`][fs]模块） 遇到路径不兼容的文件 URL。确切
用于确定是否可以使用路径的语义与平台相关。

<a id="ERR_INVALID_HANDLE_TYPE"></a>

### `ERR_INVALID_HANDLE_TYPE`

尝试通过IPC通信发送不受支持的“句柄”
通道到子进程。看[`subprocess.send()`][subprocess.send()]和[`process.send()`][process.send()]
了解更多信息。

<a id="ERR_INVALID_HTTP_TOKEN"></a>

### `ERR_INVALID_HTTP_TOKEN`

提供的 HTTP 令牌无效。

<a id="ERR_INVALID_IP_ADDRESS"></a>

### `ERR_INVALID_IP_ADDRESS`

IP 地址无效。

<a id="ERR_INVALID_MODULE"></a>

### `ERR_INVALID_MODULE`

<!-- YAML
added:
  - v15.0.0
  - v14.18.0
-->

尝试加载不存在或不存在的模块
有效。

<a id="ERR_INVALID_MODULE_SPECIFIER"></a>

### `ERR_INVALID_MODULE_SPECIFIER`

导入的模块字符串是无效的 URL、包名称或包子路径
规范。

<a id="ERR_INVALID_OBJECT_DEFINE_PROPERTY"></a>

### `ERR_INVALID_OBJECT_DEFINE_PROPERTY`

在 的属性上设置无效属性时出错
一个对象。

<a id="ERR_INVALID_PACKAGE_CONFIG"></a>

### `ERR_INVALID_PACKAGE_CONFIG`

无效[`package.json`][package.json]文件解析失败。

<a id="ERR_INVALID_PACKAGE_TARGET"></a>

### `ERR_INVALID_PACKAGE_TARGET`

这`package.json` [`"exports"`]["exports"]字段包含无效的目标映射
尝试的模块分辨率的值。

<a id="ERR_INVALID_PERFORMANCE_MARK"></a>

### `ERR_INVALID_PERFORMANCE_MARK`

使用性能计时 API （`perf_hooks`），性能标记为
无效。

<a id="ERR_INVALID_PROTOCOL"></a>

### `ERR_INVALID_PROTOCOL`

无效`options.protocol`已传递到`http.request()`.

<a id="ERR_INVALID_REPL_EVAL_CONFIG"></a>

### `ERR_INVALID_REPL_EVAL_CONFIG`

双`breakEvalOnSigint`和`eval`选项已在[`REPL`][REPL]配置，
不支持。

<a id="ERR_INVALID_REPL_INPUT"></a>

### `ERR_INVALID_REPL_INPUT`

输入不能用于[`REPL`][REPL].在此条件下
使用错误在[`REPL`][REPL]文档。

<a id="ERR_INVALID_RETURN_PROPERTY"></a>

### `ERR_INVALID_RETURN_PROPERTY`

如果函数选项未为其之一提供有效值，则抛出
执行时返回的对象属性。

<a id="ERR_INVALID_RETURN_PROPERTY_VALUE"></a>

### `ERR_INVALID_RETURN_PROPERTY_VALUE`

在函数选项未提供预期值的情况下抛出
执行时其返回的对象属性之一的类型。

<a id="ERR_INVALID_RETURN_VALUE"></a>

### `ERR_INVALID_RETURN_VALUE`

在函数选项未返回预期值的情况下抛出
执行时键入，例如当函数应返回 promise 时。

<a id="ERR_INVALID_STATE"></a>

### `ERR_INVALID_STATE`

<!-- YAML
added: v15.0.0
-->

指示由于无效状态而无法完成操作。
例如，对象可能已被销毁，或者可能已销毁
执行另一个操作。

<a id="ERR_INVALID_SYNC_FORK_INPUT"></a>

### `ERR_INVALID_SYNC_FORK_INPUT`

一个`Buffer`,`TypedArray`,`DataView`或`string`作为 stdio 输入提供给
异步分叉。请参阅文档[`child_process`][child_process]模块
了解更多信息。

<a id="ERR_INVALID_THIS"></a>

### `ERR_INVALID_THIS`

节点.js API 函数被调用，但使用了不兼容的函数`this`价值。

```js
const urlSearchParams = new URLSearchParams('foo=bar&baz=new');

const buf = Buffer.alloc(1);
urlSearchParams.has.call(buf, 'foo');
// Throws a TypeError with code 'ERR_INVALID_THIS'
```

<a id="ERR_INVALID_TRANSFER_OBJECT"></a>

### `ERR_INVALID_TRANSFER_OBJECT`

无效的传输对象已传递到`postMessage()`.

<a id="ERR_INVALID_TUPLE"></a>

### `ERR_INVALID_TUPLE`

中的元素`iterable`提供给[什么WG][WHATWG URL API]
[`URLSearchParams`构造 函数][`new URLSearchParams(iterable)`]冇
表示`[name, value]`元组 – 即，如果元素不可迭代，或者
不完全由两个元素组成。

<a id="ERR_INVALID_URI"></a>

### `ERR_INVALID_URI`

传递了无效的 URI。

<a id="ERR_INVALID_URL"></a>

### `ERR_INVALID_URL`

无效的 URL 已传递给[什么WG][WHATWG URL API] [`URL`
构造 函数][`new URL(input)`]或遗产[`url.parse()`][url.parse()]要解析。
引发的错误对象通常具有附加属性`'input'`那
包含解析失败的 URL。

<a id="ERR_INVALID_URL_SCHEME"></a>

### `ERR_INVALID_URL_SCHEME`

尝试将不兼容的方案（协议）的 URL 用于
特定目的。它仅用于[WHATWG URL API][]支持
[`fs`][fs]模块（仅接受具有以下特征的 URL`'file'`方案），但可以使用
在其他 Node.js API 以及将来也是如此。

<a id="ERR_IPC_CHANNEL_CLOSED"></a>

### `ERR_IPC_CHANNEL_CLOSED`

尝试使用已关闭的IPC通信信道。

<a id="ERR_IPC_DISCONNECTED"></a>

### `ERR_IPC_DISCONNECTED`

尝试断开已
断开。请参阅文档[`child_process`][child_process]模块
了解更多信息。

<a id="ERR_IPC_ONE_PIPE"></a>

### `ERR_IPC_ONE_PIPE`

尝试使用多个 IPC 创建子节点.js进程
沟通渠道。请参阅文档[`child_process`][child_process]模块
了解更多信息。

<a id="ERR_IPC_SYNC_FORK"></a>

### `ERR_IPC_SYNC_FORK`

尝试以同步方式打开 IPC 通信通道
分叉节点.js进程。请参阅文档[`child_process`][child_process]模块
了解更多信息。

<a id="ERR_LOADER_CHAIN_INCOMPLETE"></a>

### `ERR_LOADER_CHAIN_INCOMPLETE`

<!-- YAML
added: v18.6.0
-->

返回 ESM 加载程序挂接而不调用`next()`并且没有明确
发出短路信号。

<a id="ERR_MANIFEST_ASSERT_INTEGRITY"></a>

### `ERR_MANIFEST_ASSERT_INTEGRITY`

尝试加载资源，但该资源与
由策略清单定义的完整性。请参阅文档[政策][policy]
清单以获取更多信息。

<a id="ERR_MANIFEST_DEPENDENCY_MISSING"></a>

### `ERR_MANIFEST_DEPENDENCY_MISSING`

尝试加载资源，但该资源未列为
来自尝试加载它的位置的依赖项。查看文档
为[政策][policy]清单以获取更多信息。

<a id="ERR_MANIFEST_INTEGRITY_MISMATCH"></a>

### `ERR_MANIFEST_INTEGRITY_MISMATCH`

尝试加载策略清单，但清单具有多个
资源之间不匹配的条目。更新清单
要匹配的条目以解决此错误。请参阅文档
[政策][policy]清单以获取更多信息。

<a id="ERR_MANIFEST_INVALID_RESOURCE_FIELD"></a>

### `ERR_MANIFEST_INVALID_RESOURCE_FIELD`

策略清单资源中的某个字段的值无效。更新
要匹配的清单条目，以解决此错误。查看
文档[政策][policy]清单以获取更多信息。

<a id="ERR_MANIFEST_INVALID_SPECIFIER"></a>

### `ERR_MANIFEST_INVALID_SPECIFIER`

策略清单资源对其依赖项之一具有无效值
映射。更新清单条目以匹配以解决此错误。查看
文档[政策][policy]清单以获取更多信息。

<a id="ERR_MANIFEST_PARSE_POLICY"></a>

### `ERR_MANIFEST_PARSE_POLICY`

尝试加载策略清单，但清单无法
被解析。请参阅文档[政策][policy]清单以获取更多信息。

<a id="ERR_MANIFEST_TDZ"></a>

### `ERR_MANIFEST_TDZ`

尝试从策略清单中读取，但清单
初始化尚未进行。这可能是 Node.js 中的一个错误。

<a id="ERR_MANIFEST_UNKNOWN_ONERROR"></a>

### `ERR_MANIFEST_UNKNOWN_ONERROR`

已加载策略清单，但其“onerror”的值未知
行为。请参阅文档[政策][policy]清单以获取更多信息。

<a id="ERR_MEMORY_ALLOCATION_FAILED"></a>

### `ERR_MEMORY_ALLOCATION_FAILED`

尝试分配内存（通常在C++层中），但它
失败。

<a id="ERR_MESSAGE_TARGET_CONTEXT_UNAVAILABLE"></a>

### `ERR_MESSAGE_TARGET_CONTEXT_UNAVAILABLE`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

发布到[`MessagePort`][MessagePort]无法在目标中反序列化
[虚拟机][vm] `Context`.并非所有 Node.js 对象都可以在 中成功实例化
此时的任何上下文，并尝试使用`postMessage()`
在这种情况下，接收端可能会失败。

<a id="ERR_METHOD_NOT_IMPLEMENTED"></a>

### `ERR_METHOD_NOT_IMPLEMENTED`

方法为必填项，但未实现。

<a id="ERR_MISSING_ARGS"></a>

### `ERR_MISSING_ARGS`

未传递节点.js API 的必需参数。这仅用于
严格遵守 API 规范（在某些情况下可能会接受
`func(undefined)`但不是`func()`).在大多数本机节点.js API 中，
`func(undefined)`和`func()`以相同的方式对待，并且
[`ERR_INVALID_ARG_TYPE`][ERR_INVALID_ARG_TYPE]可以改用错误代码。

<a id="ERR_MISSING_OPTION"></a>

### `ERR_MISSING_OPTION`

对于接受选项对象的 API，某些选项可能是必需的。此代码
如果缺少必需的选项，则抛出。

<a id="ERR_MISSING_PASSPHRASE"></a>

### `ERR_MISSING_PASSPHRASE`

尝试在不指定密码的情况下读取加密密钥。

<a id="ERR_MISSING_PLATFORM_FOR_WORKER"></a>

### `ERR_MISSING_PLATFORM_FOR_WORKER`

此 Node.js 实例使用的 V8 平台不支持创建
工人。这是由于缺乏对 Worker 的嵌入器支持造成的。特别
此错误不会发生在 Node.js 的标准版本中。

<a id="ERR_MISSING_TRANSFERABLE_IN_TRANSFER_LIST"></a>

### `ERR_MISSING_TRANSFERABLE_IN_TRANSFER_LIST`

<!-- YAML
added: v15.0.0
-->

需要在`transferList`论点
在传递给 的对象中[`postMessage()`][postMessage()]呼叫，但未提供
在`transferList`用于该调用。通常，这是一个`MessagePort`.

在 node.js v15.0.0 之前的版本中，此处使用的错误代码为
[`ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST`][ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST].但是，该集
可转移对象类型已扩展为涵盖比
`MessagePort`.

<a id="ERR_MODULE_NOT_FOUND"></a>

### `ERR_MODULE_NOT_FOUND`

模块文件无法由 ECMAScript 模块加载程序解析，而
尝试`import`操作或加载程序入口点时。

<a id="ERR_MULTIPLE_CALLBACK"></a>

### `ERR_MULTIPLE_CALLBACK`

调用回调不止一次。

回调几乎总是意味着只调用一次作为查询
可以满足或拒绝，但不能同时满足或拒绝两者。后者
可以通过多次调用回调来实现。

<a id="ERR_NAPI_CONS_FUNCTION"></a>

### `ERR_NAPI_CONS_FUNCTION`

使用时`Node-API`，则传递的构造函数不是函数。

<a id="ERR_NAPI_INVALID_DATAVIEW_ARGS"></a>

### `ERR_NAPI_INVALID_DATAVIEW_ARGS`

拨打电话时`napi_create_dataview()`，给定`offset`超出界限
的数据视图或`offset + length`大于给定的长度`buffer`.

<a id="ERR_NAPI_INVALID_TYPEDARRAY_ALIGNMENT"></a>

### `ERR_NAPI_INVALID_TYPEDARRAY_ALIGNMENT`

拨打电话时`napi_create_typedarray()`，提供的`offset`不是
元素大小的倍数。

<a id="ERR_NAPI_INVALID_TYPEDARRAY_LENGTH"></a>

### `ERR_NAPI_INVALID_TYPEDARRAY_LENGTH`

拨打电话时`napi_create_typedarray()`,`(length * size_of_element) +
byte_offset`大于给定的长度`buffer`.

<a id="ERR_NAPI_TSFN_CALL_JS"></a>

### `ERR_NAPI_TSFN_CALL_JS`

调用线程安全的 JavaScript 部分时出错
功能。

<a id="ERR_NAPI_TSFN_GET_UNDEFINED"></a>

### `ERR_NAPI_TSFN_GET_UNDEFINED`

尝试检索 JavaScript 时出错`undefined`
价值。

<a id="ERR_NAPI_TSFN_START_IDLE_LOOP"></a>

### `ERR_NAPI_TSFN_START_IDLE_LOOP`

在主线程上，将从与
空闲循环中的线程安全函数。此错误表示错误
尝试启动循环时发生。

<a id="ERR_NAPI_TSFN_STOP_IDLE_LOOP"></a>

### `ERR_NAPI_TSFN_STOP_IDLE_LOOP`

一旦队列中没有更多项目，就必须挂起空闲循环。这
错误表示空闲循环已无法停止。

<a id="ERR_NOT_BUILDING_SNAPSHOT"></a>

### `ERR_NOT_BUILDING_SNAPSHOT`

尝试使用只能在构建时使用的操作
V8 启动快照，即使 Node.js 没有构建一个。

<a id="ERR_NO_CRYPTO"></a>

### `ERR_NO_CRYPTO`

尝试使用加密功能，而 Node.js 未编译
OpenSSL 加密支持。

<a id="ERR_NO_ICU"></a>

### `ERR_NO_ICU`

尝试使用需要的功能[重症监护室][ICU]，但 Node.js 不是
在 ICU 支持下编译。

<a id="ERR_NON_CONTEXT_AWARE_DISABLED"></a>

### `ERR_NON_CONTEXT_AWARE_DISABLED`

非上下文感知的本机插件加载到不允许它们的进程中。

<a id="ERR_OUT_OF_RANGE"></a>

### `ERR_OUT_OF_RANGE`

给定值超出可接受的范围。

<a id="ERR_PACKAGE_IMPORT_NOT_DEFINED"></a>

### `ERR_PACKAGE_IMPORT_NOT_DEFINED`

这`package.json` [`"imports"`]["imports"]字段未定义给定的内部
包说明符映射。

<a id="ERR_PACKAGE_PATH_NOT_EXPORTED"></a>

### `ERR_PACKAGE_PATH_NOT_EXPORTED`

这`package.json` [`"exports"`]["exports"]字段不会导出请求的子路径。
由于导出是封装的，因此未导出的私有内部模块
无法通过包解析导入，除非使用绝对 URL。

<a id="ERR_PARSE_ARGS_INVALID_OPTION_VALUE"></a>

### `ERR_PARSE_ARGS_INVALID_OPTION_VALUE`

<!-- YAML
added: v18.3.0
-->

什么时候`strict`设置为`true`，投掷者[`util.parseArgs()`][util.parseArgs()]如果 {布尔值}
为 {字符串} 类型的选项提供值，或者如果是 {字符串}
值为 {布尔值} 类型的选项提供。

<a id="ERR_PARSE_ARGS_UNEXPECTED_POSITIONAL"></a>

### `ERR_PARSE_ARGS_UNEXPECTED_POSITIONAL`

<!-- YAML
added: v18.3.0
-->

抛出者[`util.parseArgs()`][util.parseArgs()]，当提供位置参数和
`allowPositionals`设置为`false`.

<a id="ERR_PARSE_ARGS_UNKNOWN_OPTION"></a>

### `ERR_PARSE_ARGS_UNKNOWN_OPTION`

<!-- YAML
added: v18.3.0
-->

什么时候`strict`设置为`true`，投掷者[`util.parseArgs()`][util.parseArgs()]如果参数
未在 中配置`options`.

<a id="ERR_PERFORMANCE_INVALID_TIMESTAMP"></a>

### `ERR_PERFORMANCE_INVALID_TIMESTAMP`

为性能标记或度量提供了无效的时间戳值。

<a id="ERR_PERFORMANCE_MEASURE_INVALID_OPTIONS"></a>

### `ERR_PERFORMANCE_MEASURE_INVALID_OPTIONS`

为性能度量提供了无效的选项。

<a id="ERR_PROTO_ACCESS"></a>

### `ERR_PROTO_ACCESS`

访问`Object.prototype.__proto__`已禁止使用
[`--disable-proto=throw`][--disable-proto=throw].[`Object.getPrototypeOf`][Object.getPrototypeOf]和
[`Object.setPrototypeOf`][Object.setPrototypeOf]应该用于获取和设置原型
对象。

<a id="ERR_REQUIRE_ESM"></a>

### `ERR_REQUIRE_ESM`

> 稳定性： 1 - 实验

已尝试`require()`一[电子模块][ES Module].

<a id="ERR_SCRIPT_EXECUTION_INTERRUPTED"></a>

### `ERR_SCRIPT_EXECUTION_INTERRUPTED`

脚本执行被中断`SIGINT`（对于
例<kbd>Ctrl</kbd>+<kbd>C</kbd>被按下。

<a id="ERR_SCRIPT_EXECUTION_TIMEOUT"></a>

### `ERR_SCRIPT_EXECUTION_TIMEOUT`

脚本执行超时，可能是由于正在执行的脚本中存在错误。

<a id="ERR_SERVER_ALREADY_LISTEN"></a>

### `ERR_SERVER_ALREADY_LISTEN`

这[`server.listen()`][server.listen()]方法被调用，而`net.Server`已经
注意的。这适用于`net.Server`，包括 HTTP、HTTPS、
和 HTTP/2`Server`实例。

<a id="ERR_SERVER_NOT_RUNNING"></a>

### `ERR_SERVER_NOT_RUNNING`

这[`server.close()`][server.close()]当`net.Server`不是
运行。这适用于`net.Server`，包括 HTTP、HTTPS、
和 HTTP/2`Server`实例。

<a id="ERR_SOCKET_ALREADY_BOUND"></a>

### `ERR_SOCKET_ALREADY_BOUND`

尝试绑定已绑定的套接字。

<a id="ERR_SOCKET_BAD_BUFFER_SIZE"></a>

### `ERR_SOCKET_BAD_BUFFER_SIZE`

为`recvBufferSize`或
`sendBufferSize`中的选项[`dgram.createSocket()`][dgram.createSocket()].

<a id="ERR_SOCKET_BAD_PORT"></a>

### `ERR_SOCKET_BAD_PORT`

期望端口 >= 0 且< 65536 的 API 函数收到无效值。

<a id="ERR_SOCKET_BAD_TYPE"></a>

### `ERR_SOCKET_BAD_TYPE`

需要套接字类型 （`udp4`或`udp6`） 收到无效的
价值。

<a id="ERR_SOCKET_BUFFER_SIZE"></a>

### `ERR_SOCKET_BUFFER_SIZE`

使用时[`dgram.createSocket()`][dgram.createSocket()]，接收或发送的大小`Buffer`
无法确定。

<a id="ERR_SOCKET_CLOSED"></a>

### `ERR_SOCKET_CLOSED`

尝试对已关闭的套接字进行操作。

<a id="ERR_SOCKET_DGRAM_IS_CONNECTED"></a>

### `ERR_SOCKET_DGRAM_IS_CONNECTED`

一个[`dgram.connect()`][dgram.connect()]在已连接的套接字上进行了调用。

<a id="ERR_SOCKET_DGRAM_NOT_CONNECTED"></a>

### `ERR_SOCKET_DGRAM_NOT_CONNECTED`

一个[`dgram.disconnect()`][dgram.disconnect()]或[`dgram.remoteAddress()`][dgram.remoteAddress()]已在 上发出呼叫
已断开连接的插座。

<a id="ERR_SOCKET_DGRAM_NOT_RUNNING"></a>

### `ERR_SOCKET_DGRAM_NOT_RUNNING`

已进行调用，但 UDP 子系统未运行。

<a id="ERR_SRI_PARSE"></a>

### `ERR_SRI_PARSE`

为子资源完整性检查提供了一个字符串，但无法
解析。通过查看
[子资源完整性规范][Subresource Integrity specification].

<a id="ERR_STREAM_ALREADY_FINISHED"></a>

### `ERR_STREAM_ALREADY_FINISHED`

调用了无法完成的流方法，因为流是
完成。

<a id="ERR_STREAM_CANNOT_PIPE"></a>

### `ERR_STREAM_CANNOT_PIPE`

已尝试调用[`stream.pipe()`][stream.pipe()]安娜[`Writable`][Writable]流。

<a id="ERR_STREAM_DESTROYED"></a>

### `ERR_STREAM_DESTROYED`

调用了无法完成的流方法，因为流是
销毁使用`stream.destroy()`.

<a id="ERR_STREAM_NULL_VALUES"></a>

### `ERR_STREAM_NULL_VALUES`

已尝试调用[`stream.write()`][stream.write()]与`null`块。

<a id="ERR_STREAM_PREMATURE_CLOSE"></a>

### `ERR_STREAM_PREMATURE_CLOSE`

返回的错误`stream.finished()`和`stream.pipeline()`，当流
或者管道以非正常方式结束，没有明确的错误。

<a id="ERR_STREAM_PUSH_AFTER_EOF"></a>

### `ERR_STREAM_PUSH_AFTER_EOF`

已尝试调用[`stream.push()`][stream.push()]在`null`（EOF） 曾
推送到流。

<a id="ERR_STREAM_UNSHIFT_AFTER_END_EVENT"></a>

### `ERR_STREAM_UNSHIFT_AFTER_END_EVENT`

已尝试调用[`stream.unshift()`][stream.unshift()]在`'end'`事件是
排放。

<a id="ERR_STREAM_WRAP"></a>

### `ERR_STREAM_WRAP`

防止在套接字上设置字符串解码器或解码器时中止
位于`objectMode`.

```js
const Socket = require('node:net').Socket;
const instance = new Socket();

instance.setEncoding('utf8');
```

<a id="ERR_STREAM_WRITE_AFTER_END"></a>

### `ERR_STREAM_WRITE_AFTER_END`

已尝试调用[`stream.write()`][stream.write()]后`stream.end()`已经
叫。

<a id="ERR_STRING_TOO_LONG"></a>

### `ERR_STRING_TOO_LONG`

已尝试创建长度超过允许的最大值的字符串
长度。

<a id="ERR_SYNTHETIC"></a>

### `ERR_SYNTHETIC`

用于捕获呼叫堆栈以进行诊断的人为错误对象
报告。

<a id="ERR_SYSTEM_ERROR"></a>

### `ERR_SYSTEM_ERROR`

节点中发生了未指定或非特定的系统错误.js
过程。错误对象将具有一个`err.info`对象属性
其他详细信息。

<a id="ERR_TEST_FAILURE"></a>

### `ERR_TEST_FAILURE`

此错误表示测试失败。有关故障的其他信息
可通过`cause`财产。这`failureType`属性指定
发生故障时测试正在执行的操作。

<a id="ERR_TLS_CERT_ALTNAME_FORMAT"></a>

### `ERR_TLS_CERT_ALTNAME_FORMAT`

此错误由`checkServerIdentity`如果用户提供
`subjectaltname`属性违反编码规则。生成的证书对象
由Node.js本身始终遵守编码规则，永远不会导致
此错误。

<a id="ERR_TLS_CERT_ALTNAME_INVALID"></a>

### `ERR_TLS_CERT_ALTNAME_INVALID`

使用 TLS 时，对等体的主机名/IP 与任何
`subjectAltNames`在其证书中。

<a id="ERR_TLS_DH_PARAM_SIZE"></a>

### `ERR_TLS_DH_PARAM_SIZE`

使用 TLS 时，为 Diffie-Hellman 提供的参数 （`DH`)
密钥协议太小。默认情况下，密钥长度必须更大
大于或等于 1024 位以避免漏洞，即使它是强
建议使用 2048 位或更大，以获得更强的安全性。

<a id="ERR_TLS_HANDSHAKE_TIMEOUT"></a>

### `ERR_TLS_HANDSHAKE_TIMEOUT`

TLS/SSL 握手超时。在这种情况下，服务器还必须中止
连接。

<a id="ERR_TLS_INVALID_CONTEXT"></a>

### `ERR_TLS_INVALID_CONTEXT`

<!-- YAML
added: v13.3.0
-->

上下文必须是`SecureContext`.

<a id="ERR_TLS_INVALID_PROTOCOL_METHOD"></a>

### `ERR_TLS_INVALID_PROTOCOL_METHOD`

指定的`secureProtocol`方法无效。它要么是未知的，要么是
禁用，因为它不安全。

<a id="ERR_TLS_INVALID_PROTOCOL_VERSION"></a>

### `ERR_TLS_INVALID_PROTOCOL_VERSION`

有效的 TLS 协议版本是`'TLSv1'`,`'TLSv1.1'`或`'TLSv1.2'`.

<a id="ERR_TLS_INVALID_STATE"></a>

### `ERR_TLS_INVALID_STATE`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

TLS 套接字必须连接并安全建立。确保“安全”
事件在继续之前发出。

<a id="ERR_TLS_PROTOCOL_VERSION_CONFLICT"></a>

### `ERR_TLS_PROTOCOL_VERSION_CONFLICT`

尝试设置 TLS 协议`minVersion`或`maxVersion`与
尝试将`secureProtocol`明确地。使用一种机制或另一种机制。

<a id="ERR_TLS_PSK_SET_IDENTIY_HINT_FAILED"></a>

### `ERR_TLS_PSK_SET_IDENTIY_HINT_FAILED`

无法设置 PSK 标识提示。提示可能太长。

<a id="ERR_TLS_RENEGOTIATION_DISABLED"></a>

### `ERR_TLS_RENEGOTIATION_DISABLED`

尝试在禁用 TLS 的套接字实例上重新协商 TLS。

<a id="ERR_TLS_REQUIRED_SERVER_NAME"></a>

### `ERR_TLS_REQUIRED_SERVER_NAME`

使用 TLS 时，`server.addContext()`调用方法时未提供
第一个参数中的主机名。

<a id="ERR_TLS_SESSION_ATTACK"></a>

### `ERR_TLS_SESSION_ATTACK`

检测到过多的 TLS 重新协商，这是一个潜在的
拒绝服务攻击的媒介。

<a id="ERR_TLS_SNI_FROM_SERVER"></a>

### `ERR_TLS_SNI_FROM_SERVER`

已尝试从 TLS 服务器端发出服务器名称指示
套接字，它仅对客户端有效。

<a id="ERR_TRACE_EVENTS_CATEGORY_REQUIRED"></a>

### `ERR_TRACE_EVENTS_CATEGORY_REQUIRED`

这`trace_events.createTracing()`方法至少需要一个跟踪事件
类别。

<a id="ERR_TRACE_EVENTS_UNAVAILABLE"></a>

### `ERR_TRACE_EVENTS_UNAVAILABLE`

这`node:trace_events`无法加载模块，因为已编译 Node.js
与`--without-v8-platform`旗。

<a id="ERR_TRANSFORM_ALREADY_TRANSFORMING"></a>

### `ERR_TRANSFORM_ALREADY_TRANSFORMING`

一个`Transform`流在仍在转换时完成。

<a id="ERR_TRANSFORM_WITH_LENGTH_0"></a>

### `ERR_TRANSFORM_WITH_LENGTH_0`

一个`Transform`流已完成，数据仍在写入缓冲区中。

<a id="ERR_TTY_INIT_FAILED"></a>

### `ERR_TTY_INIT_FAILED`

由于系统错误，TTY 的初始化失败。

<a id="ERR_UNAVAILABLE_DURING_EXIT"></a>

### `ERR_UNAVAILABLE_DURING_EXIT`

函数在[`process.on('exit')`][process.on('exit')]不应该是
调用范围[`process.on('exit')`][process.on('exit')]处理器。

<a id="ERR_UNCAUGHT_EXCEPTION_CAPTURE_ALREADY_SET"></a>

### `ERR_UNCAUGHT_EXCEPTION_CAPTURE_ALREADY_SET`

[`process.setUncaughtExceptionCaptureCallback()`][process.setUncaughtExceptionCaptureCallback()]被叫了两次，
无需先将回调重置为`null`.

此错误旨在防止意外覆盖已注册的回调
从另一个模块。

<a id="ERR_UNESCAPED_CHARACTERS"></a>

### `ERR_UNESCAPED_CHARACTERS`

收到包含未转义字符的字符串。

<a id="ERR_UNHANDLED_ERROR"></a>

### `ERR_UNHANDLED_ERROR`

发生未处理的错误（例如，当`'error'`发出事件
由[`EventEmitter`][EventEmitter]但是`'error'`处理程序未注册）。

<a id="ERR_UNKNOWN_BUILTIN_MODULE"></a>

### `ERR_UNKNOWN_BUILTIN_MODULE`

用于标识特定类型的内部 Node.js不应出错
通常由用户代码触发。此错误的实例指向
节点内部错误.js二进制文件本身。

<a id="ERR_UNKNOWN_CREDENTIAL"></a>

### `ERR_UNKNOWN_CREDENTIAL`

传递了不存在的 Unix 组或用户标识符。

<a id="ERR_UNKNOWN_ENCODING"></a>

### `ERR_UNKNOWN_ENCODING`

无效或未知的编码选项已传递到 API。

<a id="ERR_UNKNOWN_FILE_EXTENSION"></a>

### `ERR_UNKNOWN_FILE_EXTENSION`

> 稳定性： 1 - 实验

尝试加载包含未知或不受支持的文件的模块
外延。

<a id="ERR_UNKNOWN_MODULE_FORMAT"></a>

### `ERR_UNKNOWN_MODULE_FORMAT`

> 稳定性： 1 - 实验

尝试加载格式未知或不受支持的模块。

<a id="ERR_UNKNOWN_SIGNAL"></a>

### `ERR_UNKNOWN_SIGNAL`

无效或未知的进程信号被传递到 API，期望有效的
信号（如[`subprocess.kill()`][subprocess.kill()]).

<a id="ERR_UNSUPPORTED_DIR_IMPORT"></a>

### `ERR_UNSUPPORTED_DIR_IMPORT`

`import`不支持目录 URL。相反
[使用包的名称自引用包][self-reference a package using its name]和[定义自定义子路径][define a custom subpath]在
这[`"exports"`]["exports"]的领域[`package.json`][package.json]文件。

<!-- eslint-skip -->

```js
import './'; // unsupported
import './index.js'; // supported
import 'package-name'; // supported
```

<a id="ERR_UNSUPPORTED_ESM_URL_SCHEME"></a>

### `ERR_UNSUPPORTED_ESM_URL_SCHEME`

`import`使用 URL 方案，而不是`file`和`data`不受支持。

<a id="ERR_USE_AFTER_CLOSE"></a>

### `ERR_USE_AFTER_CLOSE`

> 稳定性： 1 - 实验

尝试使用已关闭的内容。

<a id="ERR_VALID_PERFORMANCE_ENTRY_TYPE"></a>

### `ERR_VALID_PERFORMANCE_ENTRY_TYPE`

使用性能计时 API （`perf_hooks`），没有有效的性能
找到条目类型。

<a id="ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING"></a>

### `ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`

未指定动态导入回调。

<a id="ERR_VM_MODULE_ALREADY_LINKED"></a>

### `ERR_VM_MODULE_ALREADY_LINKED`

尝试链接的模块不符合链接条件，因为
原因如下：

*   它已被链接 （`linkingStatus`是`'linked'`)
*   它正在被链接（`linkingStatus`是`'linking'`)
*   此模块的链接失败 （`linkingStatus`是`'errored'`)

<a id="ERR_VM_MODULE_CACHED_DATA_REJECTED"></a>

### `ERR_VM_MODULE_CACHED_DATA_REJECTED`

这`cachedData`传递给模块构造函数的选项无效。

<a id="ERR_VM_MODULE_CANNOT_CREATE_CACHED_DATA"></a>

### `ERR_VM_MODULE_CANNOT_CREATE_CACHED_DATA`

无法为已评估的模块创建缓存数据。

<a id="ERR_VM_MODULE_DIFFERENT_CONTEXT"></a>

### `ERR_VM_MODULE_DIFFERENT_CONTEXT`

从链接器函数返回的模块来自不同的上下文
而不是父模块。链接模块必须共享相同的上下文。

<a id="ERR_VM_MODULE_LINK_FAILURE"></a>

### `ERR_VM_MODULE_LINK_FAILURE`

由于故障，模块无法链接。

<a id="ERR_VM_MODULE_NOT_MODULE"></a>

### `ERR_VM_MODULE_NOT_MODULE`

链接承诺的已实现价值不是`vm.Module`对象。

<a id="ERR_VM_MODULE_STATUS"></a>

### `ERR_VM_MODULE_STATUS`

当前模块的状态不允许此操作。具体
错误的含义取决于特定功能。

<a id="ERR_WASI_ALREADY_STARTED"></a>

### `ERR_WASI_ALREADY_STARTED`

WASI 实例已启动。

<a id="ERR_WASI_NOT_STARTED"></a>

### `ERR_WASI_NOT_STARTED`

WASI 实例尚未启动。

<a id="ERR_WEBASSEMBLY_RESPONSE"></a>

### `ERR_WEBASSEMBLY_RESPONSE`

<!-- YAML
added: v18.1.0
-->

这`Response`已传递给`WebAssembly.compileStreaming`或
`WebAssembly.instantiateStreaming`不是有效的 WebAssembly 响应。

<a id="ERR_WORKER_INIT_FAILED"></a>

### `ERR_WORKER_INIT_FAILED`

这`Worker`初始化失败。

<a id="ERR_WORKER_INVALID_EXEC_ARGV"></a>

### `ERR_WORKER_INVALID_EXEC_ARGV`

这`execArgv`选项传递给`Worker`构造函数包含
无效标志。

<a id="ERR_WORKER_NOT_RUNNING"></a>

### `ERR_WORKER_NOT_RUNNING`

操作失败，因为`Worker`实例当前未运行。

<a id="ERR_WORKER_OUT_OF_MEMORY"></a>

### `ERR_WORKER_OUT_OF_MEMORY`

这`Worker`实例已终止，因为它已达到其内存限制。

<a id="ERR_WORKER_PATH"></a>

### `ERR_WORKER_PATH`

工作线程的主脚本的路径既不是绝对路径
也不是以 开头的相对路径`./`或`../`.

<a id="ERR_WORKER_UNSERIALIZABLE_ERROR"></a>

### `ERR_WORKER_UNSERIALIZABLE_ERROR`

序列化来自工作线程的未捕获异常的所有尝试都失败。

<a id="ERR_WORKER_UNSUPPORTED_OPERATION"></a>

### `ERR_WORKER_UNSUPPORTED_OPERATION`

工作线程中不支持请求的功能。

<a id="ERR_ZLIB_INITIALIZATION_FAILED"></a>

### `ERR_ZLIB_INITIALIZATION_FAILED`

创建[`zlib`][zlib]对象由于配置不正确而失败。

<a id="HPE_HEADER_OVERFLOW"></a>

### `HPE_HEADER_OVERFLOW`

<!-- YAML
changes:
  - version:
     - v11.4.0
     - v10.15.0
    commit: 186035243fad247e3955f
    pr-url: https://github.com/nodejs-private/node-private/pull/143
    description: Max header size in `http_parser` was set to 8 KiB.
-->

接收到的 HTTP 标头数据过多。为了防止恶意或
配置错误的客户端，如果接收到的 HTTP 标头数据超过 8 KiB，则
HTTP 解析将在不创建请求或响应对象的情况下中止，并且
一`Error`将发出此代码。

<a id="HPE_UNEXPECTED_CONTENT_LENGTH"></a>

### `HPE_UNEXPECTED_CONTENT_LENGTH`

服务器正在发送两个`Content-Length`标头和`Transfer-Encoding: chunked`.

`Transfer-Encoding: chunked`允许服务器维护 HTTP 持久性
动态生成内容的连接。
在这种情况下，`Content-Length`不能使用 HTTP 标头。

用`Content-Length`或`Transfer-Encoding: chunked`.

<a id="MODULE_NOT_FOUND"></a>

### `MODULE_NOT_FOUND`

<!-- YAML
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/25690
    description: Added `requireStack` property.
-->

模块文件无法由 CommonJS 模块加载程序解析，而
尝试[`require()`][require()]操作或加载程序入口点时。

## 旧节点.js错误代码

> 稳定性：0 - 已弃用。这些错误代码要么不一致，要么具有
> 已删除。

<a id="ERR_CANNOT_TRANSFER_OBJECT"></a>

### `ERR_CANNOT_TRANSFER_OBJECT`

<!--
added: v10.5.0
removed: v12.5.0
-->

传递给的值`postMessage()`包含不支持的对象
用于转移。

<a id="ERR_CRYPTO_HASH_DIGEST_NO_UTF16"></a>

### `ERR_CRYPTO_HASH_DIGEST_NO_UTF16`

<!-- YAML
added: v9.0.0
removed: v12.12.0
-->

UTF-16 编码与[`hash.digest()`][hash.digest()].而
`hash.digest()`方法允许`encoding`要传入的参数，
导致方法返回字符串而不是`Buffer`，UTF-16
编码（例如`ucs`或`utf16le`） 不受支持。

<a id="ERR_HTTP2_FRAME_ERROR"></a>

### `ERR_HTTP2_FRAME_ERROR`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在发生故障时使用，在 HTTP/2 上发送单个帧
会期。

<a id="ERR_HTTP2_HEADERS_OBJECT"></a>

### `ERR_HTTP2_HEADERS_OBJECT`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在需要 HTTP/2 标头对象时使用。

<a id="ERR_HTTP2_HEADER_REQUIRED"></a>

### `ERR_HTTP2_HEADER_REQUIRED`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在 HTTP/2 消息中缺少必需的标头时使用。

<a id="ERR_HTTP2_INFO_HEADERS_AFTER_RESPOND"></a>

### `ERR_HTTP2_INFO_HEADERS_AFTER_RESPOND`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

HTTP/2 信息标头只能发送*事先*呼叫
`Http2Stream.prototype.respond()`方法。

<a id="ERR_HTTP2_STREAM_CLOSED"></a>

### `ERR_HTTP2_STREAM_CLOSED`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在已对已执行的 HTTP/2 流执行操作时使用
已关闭。

<a id="ERR_HTTP_INVALID_CHAR"></a>

### `ERR_HTTP_INVALID_CHAR`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在 HTTP 响应状态消息中发现无效字符时使用
（原因短语）。

<a id="ERR_INDEX_OUT_OF_RANGE"></a>

### `ERR_INDEX_OUT_OF_RANGE`

<!-- YAML
  added: v10.0.0
  removed: v11.0.0
-->

给定的索引超出可接受的范围（例如负偏移）。

<a id="ERR_INVALID_OPT_VALUE"></a>

### `ERR_INVALID_OPT_VALUE`

<!-- YAML
added: v8.0.0
removed: v15.0.0
-->

在选项对象中传递了无效或意外的值。

<a id="ERR_INVALID_OPT_VALUE_ENCODING"></a>

### `ERR_INVALID_OPT_VALUE_ENCODING`

<!-- YAML
added: v9.0.0
removed: v15.0.0
-->

传递了无效或未知的文件编码。

<a id="ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST"></a>

### `ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST`

<!-- YAML
removed: v15.0.0
-->

此错误代码已替换为[`ERR_MISSING_TRANSFERABLE_IN_TRANSFER_LIST`][ERR_MISSING_TRANSFERABLE_IN_TRANSFER_LIST]
在 Node.js v15.0.0 中，因为它不再像其他类型的
可转移对象现在也存在。

<a id="ERR_NAPI_CONS_PROTOTYPE_OBJECT"></a>

### `ERR_NAPI_CONS_PROTOTYPE_OBJECT`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

由`Node-API`什么时候`Constructor.prototype`不是对象。

<a id="ERR_NETWORK_IMPORT_BAD_RESPONSE"></a>

### `ERR_NETWORK_IMPORT_BAD_RESPONSE`

> 稳定性： 1 - 实验

已收到响应，但在通过网络导入模块时无效。

<a id="ERR_NETWORK_IMPORT_DISALLOWED"></a>

### `ERR_NETWORK_IMPORT_DISALLOWED`

> 稳定性： 1 - 实验

网络模块试图加载另一个不允许它
负荷。此限制可能是出于安全原因。

<a id="ERR_NO_LONGER_SUPPORTED"></a>

### `ERR_NO_LONGER_SUPPORTED`

节点.js API 以不受支持的方式调用，例如
`Buffer.write(string, encoding, offset[, length])`.

<a id="ERR_OPERATION_FAILED"></a>

### `ERR_OPERATION_FAILED`

<!-- YAML
added: v15.0.0
-->

操作失败。这通常用于发出一般故障的信号
的异步操作。

<a id="ERR_OUTOFMEMORY"></a>

### `ERR_OUTOFMEMORY`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

通常用于识别操作导致内存不足
条件。

<a id="ERR_PARSE_HISTORY_DATA"></a>

### `ERR_PARSE_HISTORY_DATA`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

这`node:repl`模块无法解析 REPL 历史记录文件中的数据。

<a id="ERR_SOCKET_CANNOT_SEND"></a>

### `ERR_SOCKET_CANNOT_SEND`

<!-- YAML
added: v9.0.0
removed: v14.0.0
-->

无法在套接字上发送数据。

<a id="ERR_STDERR_CLOSE"></a>

### `ERR_STDERR_CLOSE`

<!-- YAML
removed: v10.12.0
changes:
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/23053
    description: Rather than emitting an error, `process.stderr.end()` now
                 only closes the stream side but not the underlying resource,
                 making this error obsolete.
-->

已尝试关闭`process.stderr`流。根据设计，Node.js
不允许`stdout`或`stderr`要由用户代码关闭的流。

<a id="ERR_STDOUT_CLOSE"></a>

### `ERR_STDOUT_CLOSE`

<!-- YAML
removed: v10.12.0
changes:
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/23053
    description: Rather than emitting an error, `process.stderr.end()` now
                 only closes the stream side but not the underlying resource,
                 making this error obsolete.
-->

已尝试关闭`process.stdout`流。根据设计，Node.js
不允许`stdout`或`stderr`要由用户代码关闭的流。

<a id="ERR_STREAM_READ_NOT_IMPLEMENTED"></a>

### `ERR_STREAM_READ_NOT_IMPLEMENTED`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在尝试使用尚未实现的可读流时使用
[`readable._read()`][readable._read()].

<a id="ERR_TLS_RENEGOTIATION_FAILED"></a>

### `ERR_TLS_RENEGOTIATION_FAILED`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在 TLS 重新协商请求以非特定方式失败时使用。

<a id="ERR_TRANSFERRING_EXTERNALIZED_SHAREDARRAYBUFFER"></a>

### `ERR_TRANSFERRING_EXTERNALIZED_SHAREDARRAYBUFFER`

<!-- YAML
added: v10.5.0
removed: v14.0.0
-->

一个`SharedArrayBuffer`其内存不受 JavaScript 引擎管理
或由 Node.js 在序列化过程中遇到。这样的`SharedArrayBuffer`
无法序列化。

这只有在本机插件创建时才会发生`SharedArrayBuffer`s 在
“外化”模式，或将现有`SharedArrayBuffer`进入外化模式。

<a id="ERR_UNKNOWN_STDIN_TYPE"></a>

### `ERR_UNKNOWN_STDIN_TYPE`

<!-- YAML
added: v8.0.0
removed: v11.7.0
-->

尝试启动具有未知.js节点进程`stdin`文件
类型。此错误通常表示 Node 本身.js中存在错误，
尽管用户代码可以触发它。

<a id="ERR_UNKNOWN_STREAM_TYPE"></a>

### `ERR_UNKNOWN_STREAM_TYPE`

<!-- YAML
added: v8.0.0
removed: v11.7.0
-->

尝试启动具有未知.js节点进程`stdout`或
`stderr`文件类型。此错误通常表示 Node 中存在 bug.js
本身，尽管用户代码可以触发它。

<a id="ERR_V8BREAKITERATOR"></a>

### `ERR_V8BREAKITERATOR`

The V8`BreakIterator`使用了 API，但未安装完整的 ICU 数据集。

<a id="ERR_VALUE_OUT_OF_RANGE"></a>

### `ERR_VALUE_OUT_OF_RANGE`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在给定值超出可接受范围时使用。

<a id="ERR_VM_MODULE_NOT_LINKED"></a>

### `ERR_VM_MODULE_NOT_LINKED`

在实例化之前，必须成功链接模块。

<a id="ERR_VM_MODULE_LINKING_ERRORED"></a>

### `ERR_VM_MODULE_LINKING_ERRORED`

<!-- YAML
added: v10.0.0
removed: v18.1.0
-->

链接器函数返回了链接失败的模块。

<a id="ERR_WORKER_UNSUPPORTED_EXTENSION"></a>

### `ERR_WORKER_UNSUPPORTED_EXTENSION`

<!-- YAML
added: v11.0.0
removed: v16.9.0
-->

用于工作线程主脚本的路径名具有
未知文件扩展名。

<a id="ERR_ZLIB_BINDING_CLOSED"></a>

### `ERR_ZLIB_BINDING_CLOSED`

<!-- YAML
added: v9.0.0
removed: v10.0.0
-->

在尝试使用`zlib`对象之后，它已被
闭。

<a id="ERR_CPU_USAGE"></a>

### `ERR_CPU_USAGE`

<!-- YAML
removed: v15.0.0
-->

来自 的本机调用`process.cpuUsage`无法处理。

[ES Module]: esm.md

[ICU]: intl.md#internationalization-support

[JSON Web Key Elliptic Curve Registry]: https://www.iana.org/assignments/jose/jose.xhtml#web-key-elliptic-curve

[JSON Web Key Types Registry]: https://www.iana.org/assignments/jose/jose.xhtml#web-key-types

[Node.js error codes]: #nodejs-error-codes

[RFC 7230 Section 3]: https://tools.ietf.org/html/rfc7230#section-3

[Subresource Integrity specification]: https://www.w3.org/TR/SRI/#the-integrity-attribute

[V8's stack trace API]: https://v8.dev/docs/stack-trace-api

[WHATWG Supported Encodings]: util.md#whatwg-supported-encodings

[WHATWG URL API]: url.md#the-whatwg-url-api

[`"exports"`]: packages.md#exports

[`"imports"`]: packages.md#imports

[`'uncaughtException'`]: process.md#event-uncaughtexception

[`--disable-proto=throw`]: cli.md#--disable-protomode

[`--force-fips`]: cli.md#--force-fips

[`--no-addons`]: cli.md#--no-addons

[`Class: assert.AssertionError`]: assert.md#class-assertassertionerror

[`ERR_INVALID_ARG_TYPE`]: #err_invalid_arg_type

[`ERR_MISSING_MESSAGE_PORT_IN_TRANSFER_LIST`]: #err_missing_message_port_in_transfer_list

[`ERR_MISSING_TRANSFERABLE_IN_TRANSFER_LIST`]: #err_missing_transferable_in_transfer_list

[`EventEmitter`]: events.md#class-eventemitter

[`MessagePort`]: worker_threads.md#class-messageport

[`Object.getPrototypeOf`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf

[`Object.setPrototypeOf`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf

[`REPL`]: repl.md

[`Writable`]: stream.md#class-streamwritable

[`child_process`]: child_process.md

[`cipher.getAuthTag()`]: crypto.md#ciphergetauthtag

[`crypto.getDiffieHellman()`]: crypto.md#cryptogetdiffiehellmangroupname

[`crypto.scrypt()`]: crypto.md#cryptoscryptpassword-salt-keylen-options-callback

[`crypto.scryptSync()`]: crypto.md#cryptoscryptsyncpassword-salt-keylen-options

[`crypto.timingSafeEqual()`]: crypto.md#cryptotimingsafeequala-b

[`dgram.connect()`]: dgram.md#socketconnectport-address-callback

[`dgram.createSocket()`]: dgram.md#dgramcreatesocketoptions-callback

[`dgram.disconnect()`]: dgram.md#socketdisconnect

[`dgram.remoteAddress()`]: dgram.md#socketremoteaddress

[`errno`(3) man page]: https://man7.org/linux/man-pages/man3/errno.3.html

[`fs.Dir`]: fs.md#class-fsdir

[`fs.cp()`]: fs.md#fscpsrc-dest-options-callback

[`fs.readFileSync`]: fs.md#fsreadfilesyncpath-options

[`fs.readdir`]: fs.md#fsreaddirpath-options-callback

[`fs.symlink()`]: fs.md#fssymlinktarget-path-type-callback

[`fs.symlinkSync()`]: fs.md#fssymlinksynctarget-path-type

[`fs.unlink`]: fs.md#fsunlinkpath-callback

[`fs`]: fs.md

[`hash.digest()`]: crypto.md#hashdigestencoding

[`hash.update()`]: crypto.md#hashupdatedata-inputencoding

[`http`]: http.md

[`https`]: https.md

[`libuv Error handling`]: https://docs.libuv.org/en/v1.x/errors.html

[`net`]: net.md

[`new URL(input)`]: url.md#new-urlinput-base

[`new URLSearchParams(iterable)`]: url.md#new-urlsearchparamsiterable

[`package.json`]: packages.md#nodejs-packagejson-field-definitions

[`postMessage()`]: worker_threads.md#portpostmessagevalue-transferlist

[`process.on('exit')`]: process.md#event-exit

[`process.send()`]: process.md#processsendmessage-sendhandle-options-callback

[`process.setUncaughtExceptionCaptureCallback()`]: process.md#processsetuncaughtexceptioncapturecallbackfn

[`readable._read()`]: stream.md#readable_readsize

[`require('node:crypto').setEngine()`]: crypto.md#cryptosetengineengine-flags

[`require()`]: modules.md#requireid

[`server.close()`]: net.md#serverclosecallback

[`server.listen()`]: net.md#serverlisten

[`sign.sign()`]: crypto.md#signsignprivatekey-outputencoding

[`stream.pipe()`]: stream.md#readablepipedestination-options

[`stream.push()`]: stream.md#readablepushchunk-encoding

[`stream.unshift()`]: stream.md#readableunshiftchunk-encoding

[`stream.write()`]: stream.md#writablewritechunk-encoding-callback

[`subprocess.kill()`]: child_process.md#subprocesskillsignal

[`subprocess.send()`]: child_process.md#subprocesssendmessage-sendhandle-options-callback

[`url.parse()`]: url.md#urlparseurlstring-parsequerystring-slashesdenotehost

[`util.getSystemErrorName(error.errno)`]: util.md#utilgetsystemerrornameerr

[`util.inspect()`]: util.md#utilinspectobject-options

[`util.parseArgs()`]: util.md#utilparseargsconfig

[`v8.startupSnapshot.setDeserializeMainFunction()`]: v8.md#v8startupsnapshotsetdeserializemainfunctioncallback-data

[`zlib`]: zlib.md

[crypto digest algorithm]: crypto.md#cryptogethashes

[debugger]: debugger.md

[define a custom subpath]: packages.md#subpath-exports

[domains]: domain.md

[event emitter-based]: events.md#class-eventemitter

[file descriptors]: https://en.wikipedia.org/wiki/File_descriptor

[policy]: policy.md

[self-reference a package using its name]: packages.md#self-referencing-a-package-using-its-name

[stream-based]: stream.md

[syscall]: https://man7.org/linux/man-pages/man2/syscalls.2.html

[try-catch]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch

[vm]: vm.md
