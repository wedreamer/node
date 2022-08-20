# Child process

<!--introduced_in=v0.10.0-->

> Stability: 2 - Stable

<!-- source_link=lib/child_process.js -->

`node:child_process` 模块提供了以与 popen(3) 类似但不相同的方式生成子进程的能力。此功能主要由 [`child_process.spawn()`][] 函数提供:

```js
const { spawn } = require('node:child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

默认情况下，`stdin`、`stdout` 和 `stderr` 的管道在父 Node.js 进程和衍生的子进程之间建立。这些管道的容量有限（且特定于平台）。如果子进程写入标准输出超出该限制而没有捕获输出，则子进程会阻塞等待管道缓冲区接受更多数据。这与 shell 中管道的行为相同。如果输出不会被消耗，请使用 `{ stdio: 'ignore' }` 选项.

如果 `env` 在 `options` 对象中，则使用 `options.env.PATH` 环境变量执行命令查找。否则，使用 `process.env.PATH`。如果 `options.env` 未设置 `PATH`，则在 Unix 上的默认搜索路径搜索 `/usr/bin:/bin` 执行查找（请参阅操作系统的 execvpe/execvp 手册），在 Windows 上当前使用进程环境变量“PATH”.

在 Windows 上，环境变量不区分大小写。 Node.js 按字典顺序对 `env` 键进行排序，并使用不区分大小写匹配的第一个键。只有第一个（按字典顺序）条目将传递给子进程。当将对象传递给具有相同键的多个变体的 `env` 选项时，这可能会在 Windows 上导致问题，例如 `PATH` 和 `Path`.

[`child_process.spawn()`][] 方法异步生成子进程，而不会阻塞 Node.js 事件循环。 [`child_process.spawnSync()`][] 函数以同步方式提供等效功能，该功能会阻塞事件循环，直到生成的进程退出或终止.

为方便起见，`node:child_process` 模块为 [`child_process.spawn()`][] 和 [`child_process.spawnSync()`][] 提供了一些同步和异步替代方案。这些替代方案中的每一个都在 [`child_process.spawn()`][] 或 [`child_process.spawnSync()`][] 之上实现.

* [`child_process.exec()`][]: 生成一个 shell 并在该 shell 中运行一个命令，完成时将 `stdout` 和 `stderr` 传递给回调函数.
* [`child_process.execFile()`][]: 与 [`child_process.exec()`][] 类似，不同之处在于它直接生成命令，而不是默认情况下首先生成 shell.
* [`child_process.fork()`][]: 生成一个新的 Node.js 进程并调用一个指定的模块，并建立一个 IPC 通信通道，允许在父子节点之间发送消息.
* [`child_process.execSync()`][]: 的同步版本
  [`child_process.exec()`][] 这将阻止 Node.js 事件循环.
* [`child_process.execFileSync()`][]: 的同步版本
  [`child_process.execFile()`][] 这将阻止 Node.js 事件循环.

对于某些用例，例如自动化 shell 脚本，[同步对应项][] 可能更方便。然而，在许多情况下，同步方法可能会对性能产生重大影响，因为在生成的进程完成时会停止事件循环.

## Asynchronous process creation

[`child_process.spawn()`][]、[`child_process.fork()`][]、[`child_process.exec()`][] 和 [`child_process.execFile()`][] 方法都遵循其他 Node.js API 典型的惯用异步编程模式.

每个方法都返回一个 [`ChildProcess`][] 实例。这些对象实现了 Node.js [`EventEmitter`][] API，允许父进程注册监听函数，当子进程生命周期中发生某些事件时调用这些监听函数.

[`child_process.exec()`][] 和 [`child_process.execFile()`][] 方法还允许指定在子进程终止时调用的可选 `callback` 函数.

### Spawning `.bat` and `.cmd` files on Windows

[`child_process.exec()`][] 和 [`child_process.execFile()`][] 之间区别的重要性可能因平台而异。在 Unix 类型的操作系统（Unix、Linux、macOS）上，[`child_process.execFile()`][] 可能更有效，因为它默认不生成 shell。然而，在 Windows 上，`.bat` 和 `.cmd` 文件在没有终端的情况下无法单独执行，因此无法使用 [`child_process.execFile()`][] 启动.
在 Windows 上运行时，可以使用带有 `shell` 选项集的 [`child_process.spawn()`][] 调用 `.bat` 和 `.cmd` 文件，并使用 [`child_process.exec()`][] ，或者通过生成 `cmd.exe` 并将 `.bat` 或 `.cmd` 文件作为参数传递（这是 `shell` 选项和 [`child_process.exec()`][] 所做的）。无论如何，如果脚本文件名包含空格，则需要引用.

```js
// On Windows Only...
const { spawn } = require('node:child_process');
const bat = spawn('cmd.exe', ['/c', 'my.bat']);

bat.stdout.on('data', (data) => {
  console.log(data.toString());
});

bat.stderr.on('data', (data) => {
  console.error(data.toString());
});

bat.on('exit', (code) => {
  console.log(`Child exited with code ${code}`);
});
```

```js
// OR...
const { exec, spawn } = require('node:child_process');
exec('my.bat', (err, stdout, stderr) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(stdout);
});

// Script with spaces in the filename:
const bat = spawn('"my script.cmd"', ['a', 'b'], { shell: true });
// or:
exec('"my script.cmd" a b', (err, stdout, stderr) => {
  // ...
});
```

### `child_process.exec(command[, options][, callback])`

<!-- YAML
added: v0.1.90
changes:
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v15.4.0
    pr-url: https://github.com/nodejs/node/pull/36308
    description: AbortSignal support was added.
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
-->

* `command` {string} 要运行的命令，带有空格分隔的参数.
* `options` {Object}
  * `cwd` {string|URL} 子进程的当前工作目录.
    **Default:** `process.cwd()`.
  * `env` {Object} 环境键值对。 **默认值：** `process.env`.
  * `encoding` {string} **Default:** `'utf8'`
  * `shell` {string} 用于执行命令的 Shell。请参阅 [Shell 要求][] 和 [默认 Windows shell][]。 **默认值：** Unix 上为 `'/bin/sh'`，Windows 上为 `process.env.ComSpec`.
  * `signal` {AbortSignal} 允许使用 AbortSignal 中止子进程.
  * `timeout` {number} **Default:** `0`
  * `maxBuffer` {number} stdout 或 stderr 上允许的最大数据量（以字节为单位）。如果超过，则终止子进程并截断任何输出。请参阅 [`maxBuffer` 和 Unicode][] 处的警告.
    **Default:** `1024 * 1024`.
  * `killSignal` {string|integer} **Default:** `'SIGTERM'`
  * `uid` {number} Sets the user identity of the process (see setuid(2)).
  * `gid` {number} Sets the group identity of the process (see setgid(2)).
  * `windowsHide` {boolean} Hide the subprocess console window that would
    normally be created on Windows systems. **Default:** `false`.
* `callback` {Function} called with the output when process terminates.
  * `error` {Error}
  * `stdout` {string|Buffer}
  * `stderr` {string|Buffer}
* Returns: {ChildProcess}

Spawns a shell then executes the `command` within that shell, buffering any
generated output. The `command` string passed to the exec function is processed
directly by the shell and special characters (vary based on
[shell](https://en.wikipedia.org/wiki/List_of_command-line_interpreters))
need to be dealt with accordingly:

```js
const { exec } = require('node:child_process');

exec('"/path/to/test file/test.sh" arg1 arg2');
// Double quotes are used so that the space in the path is not interpreted as
// a delimiter of multiple arguments.

exec('echo "The \\$HOME variable is $HOME"');
// The $HOME variable is escaped in the first instance, but not in the second.
```

**Never pass unsanitized user input to this function. Any input containing shell
metacharacters may be used to trigger arbitrary command execution.**

If a `callback` function is provided, it is called with the arguments
`(error, stdout, stderr)`. On success, `error` will be `null`. On error,
`error` will be an instance of [`Error`][]. The `error.code` property will be
the exit code of the process. By convention, any exit code other than `0`
indicates an error. `error.signal` will be the signal that terminated the
process.

The `stdout` and `stderr` arguments passed to the callback will contain the
stdout and stderr output of the child process. By default, Node.js will decode
the output as UTF-8 and pass strings to the callback. The `encoding` option
can be used to specify the character encoding used to decode the stdout and
stderr output. If `encoding` is `'buffer'`, or an unrecognized character
encoding, `Buffer` objects will be passed to the callback instead.

```js
const { exec } = require('node:child_process');
exec('cat *.js missing_file | wc -l', (error, stdout, stderr) => {
  if (error) {
    console.error(`exec error: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
  console.error(`stderr: ${stderr}`);
});
```

If `timeout` is greater than `0`, the parent will send the signal
identified by the `killSignal` property (the default is `'SIGTERM'`) if the
child runs longer than `timeout` milliseconds.

Unlike the exec(3) POSIX system call, `child_process.exec()` does not replace
the existing process and uses a shell to execute the command.

If this method is invoked as its [`util.promisify()`][]ed version, it returns
a `Promise` for an `Object` with `stdout` and `stderr` properties. The returned
`ChildProcess` instance is attached to the `Promise` as a `child` property. In
case of an error (including any error resulting in an exit code other than 0), a
rejected promise is returned, with the same `error` object given in the
callback, but with two additional properties `stdout` and `stderr`.

```js
const util = require('node:util');
const exec = util.promisify(require('node:child_process').exec);

async function lsExample() {
  const { stdout, stderr } = await exec('ls');
  console.log('stdout:', stdout);
  console.error('stderr:', stderr);
}
lsExample();
```

If the `signal` option is enabled, calling `.abort()` on the corresponding
`AbortController` is similar to calling `.kill()` on the child process except
the error passed to the callback will be an `AbortError`:

```js
const { exec } = require('node:child_process');
const controller = new AbortController();
const { signal } = controller;
const child = exec('grep ssh', { signal }, (error) => {
  console.log(error); // an AbortError
});
controller.abort();
```

### `child_process.execFile(file[, args][, options][, callback])`

<!-- YAML
added: v0.1.91
changes:
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version:
      - v15.4.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/36308
    description: AbortSignal support was added.
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
-->

* `file` {string} The name or path of the executable file to run.
* `args` {string\[]} List of string arguments.
* `options` {Object}
  * `cwd` {string|URL} Current working directory of the child process.
  * `env` {Object} Environment key-value pairs. **Default:** `process.env`.
  * `encoding` {string} **Default:** `'utf8'`
  * `timeout` {number} **Default:** `0`
  * `maxBuffer` {number} Largest amount of data in bytes allowed on stdout or
    stderr. If exceeded, the child process is terminated and any output is
    truncated. See caveat at [`maxBuffer` and Unicode][].
    **Default:** `1024 * 1024`.
  * `killSignal` {string|integer} **Default:** `'SIGTERM'`
  * `uid` {number} 设置进程的用户身份（参见 setuid(2)）.
  * `gid` {number} 设置进程的组标识（参见 setgid(2)）.
  * `windowsHide` {boolean} 隐藏通常在 Windows 系统上创建的子进程控制台窗口。 **默认值：** `false`.
  * `windowsVerbatimArguments` {boolean} 在 Windows 上不会引用或转义参数。在 Unix 上被忽略。 **默认值：** `false`.
  * `shell` {boolean|string} 如果为 `true`，则在 shell 中运行`command`。在 Unix 上使用 `'/bin/sh'`，在 Windows 上使用 `process.env.ComSpec`。可以将不同的 shell 指定为字符串。请参阅 [Shell 要求][] 和 [默认 Windows shell][]。 **默认值：** `false`（无外壳）.
  * `signal` {AbortSignal} 允许使用 AbortSignal 中止子进程.
* `callback` {Function} 进程终止时使用输出调用.
  * `error` {Error}
  * `stdout` {string|Buffer}
  * `stderr` {string|Buffer}
* Returns: {ChildProcess}

`child_process.execFile()` 函数与 [`child_process.exec()`][] 类似，只是它默认不生成 shell。相反，指定的可执行文件作为新进程直接生成，比 [`child_process.exec()`][].

支持与 [`child_process.exec()`][] 相同的选项。由于未生成 shell，因此不支持 I/O 重定向和文件通配等行为.

```js
const { execFile } = require('node:child_process');
const child = execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    throw error;
  }
  console.log(stdout);
});
```

传递给回调的 `stdout` 和 `stderr` 参数将包含子进程的 stdout 和 stderr 输出。默认情况下，Node.js 会将输出解码为 UTF-8 并将字符串传递给回调。 `encoding` 选项可用于指定用于解码 stdout 和 stderr 输出的字符编码。如果 `encoding` 是 `'buffer'`，或无法识别的字符编码，则 `Buffer` 对象将被传递给回调.

如果此方法作为其 [`util.promisify()`][]ed 版本调用，它会为具有 `stdout` 和 `stderr` 属性的`Object` 返回一个`Promise`。返回的 `ChildProcess` 实例作为 `child` 属性附加到 `Promise`。如果出现错误（包括导致退出代码不是 0 的任何错误），将返回一个被拒绝的 promise，并在回调中给出相同的 `error` 对象，但具有两个附加属性 `stdout` 和 `stderr`.

```js
const util = require('node:util');
const execFile = util.promisify(require('node:child_process').execFile);
async function getVersion() {
  const { stdout } = await execFile('node', ['--version']);
  console.log(stdout);
}
getVersion();
```

**如果启用了 `shell` 选项，请不要将未经处理的用户输入传递给此函数。任何包含 shell 元字符的输入都可用于触发
任意命令执行。**

如果启用了 `signal` 选项，则在相应的 `AbortController` 上调用 `.abort()` 类似于在子进程上调用 `.kill()`，除了传递给回调的错误将是 `AbortError`:

```js
const { execFile } = require('node:child_process');
const controller = new AbortController();
const { signal } = controller;
const child = execFile('node', ['--version'], { signal }, (error) => {
  console.log(error); // an AbortError
});
controller.abort();
```

### `child_process.fork(modulePath[, args][, options])`

<!-- YAML
added: v0.5.0
changes:
  - version:
      - v17.4.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41225
    description: The `modulePath` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version:
      - v15.13.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37256
    description: timeout was added.
  - version:
      - v15.11.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37325
    description: killSignal for AbortSignal was added.
  - version:
      - v15.6.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/36603
    description: AbortSignal support was added.
  - version:
      - v13.2.0
      - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30162
    description: The `serialization` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10866
    description: The `stdio` option can now be a string.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7811
    description: The `stdio` option is supported now.
-->

* `modulePath` {string|URL} The module to run in the child.
* `args` {string\[]} List of string arguments.
* `options` {Object}
  * `cwd` {string|URL} Current working directory of the child process.
  * `detached` {boolean} Prepare child to run independently of its parent
    process. Specific behavior depends on the platform, see
    [`options.detached`][]).
  * `env` {Object} Environment key-value pairs. **Default:** `process.env`.
  * `execPath` {string} Executable used to create the child process.
  * `execArgv` {string\[]} List of string arguments passed to the executable.
    **Default:** `process.execArgv`.
  * `gid` {number} Sets the group identity of the process (see setgid(2)).
  * `serialization` {string} Specify the kind of serialization used for sending
    messages between processes. Possible values are `'json'` and `'advanced'`.
    See [Advanced serialization][] for more details. **Default:** `'json'`.
  * `signal` {AbortSignal} Allows closing the child process using an
    AbortSignal.
  * `killSignal` {string|integer} The signal value to be used when the spawned
    process will be killed by timeout or abort signal. **Default:** `'SIGTERM'`.
  * `silent` {boolean} If `true`, stdin, stdout, and stderr of the child will be
    piped to the parent, otherwise they will be inherited from the parent, see
    the `'pipe'` and `'inherit'` options for [`child_process.spawn()`][]'s
    [`stdio`][] for more details. **Default:** `false`.
  * `stdio` {Array|string} See [`child_process.spawn()`][]'s [`stdio`][].
    When this option is provided, it overrides `silent`. If the array variant
    is used, it must contain exactly one item with value `'ipc'` or an error
    will be thrown. For instance `[0, 1, 2, 'ipc']`.
  * `uid` {number} Sets the user identity of the process (see setuid(2)).
  * `windowsVerbatimArguments` {boolean} No quoting or escaping of arguments is
    done on Windows. Ignored on Unix. **Default:** `false`.
  * `timeout` {number} In milliseconds the maximum amount of time the process
    is allowed to run. **Default:** `undefined`.
* Returns: {ChildProcess}

`child_process.fork()` 方法是 [`child_process.spawn()`][] 的一个特例，专门用于生成新的 Node.js 进程.
与 [`child_process.spawn()`][] 一样，返回一个 [`ChildProcess`][] 对象。返回的 [`ChildProcess`][] 将有一个额外的内置通信通道，允许消息在父子进程之间来回传递。有关详细信息，请参阅 [`subprocess.send()`][].

请记住，生成的 Node.js 子进程独立于父进程，但两者之间建立的 IPC 通信通道除外。每个进程都有自己的内存，有自己的 V8 实例。由于需要额外的资源分配，不建议生成大量子 Node.js 进程.

默认情况下，`child_process.fork()` 将使用父进程的 [`process.execPath`][] 生成新的 Node.js 实例。 `options` 对象中的 `execPath` 属性允许使用替代执行路径.

使用自定义 `execPath` 启动的 Node.js 进程将使用子进程上的环境变量 `NODE_CHANNEL_FD` 标识的文件描述符 (fd) 与父进程通信.

与 fork(2) POSIX 系统调用不同，`child_process.fork()` 不会克隆当前进程.

`child_process.fork()` 不支持 [`child_process.spawn()`][] 中可用的 `shell` 选项，如果设置将被忽略.

如果启用了 `signal` 选项，则在相应的 `AbortController` 上调用 `.abort()` 类似于在子进程上调用 `.kill()`，除了传递给回调的错误将是 `AbortError`:

```js
if (process.argv[2] === 'child') {
  setTimeout(() => {
    console.log(`Hello from ${process.argv[2]}!`);
  }, 1_000);
} else {
  const { fork } = require('node:child_process');
  const controller = new AbortController();
  const { signal } = controller;
  const child = fork(__filename, ['child'], { signal });
  child.on('error', (err) => {
    // This will be called with err being an AbortError if the controller aborts
  });
  controller.abort(); // Stops the child process
}
```

### `child_process.spawn(command[, args][, options])`

<!-- YAML
added: v0.1.90
changes:
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version:
      - v15.13.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37256
    description: timeout was added.
  - version:
      - v15.11.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37325
    description: killSignal for AbortSignal was added.
  - version:
      - v15.5.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/36432
    description: AbortSignal support was added.
  - version:
      - v13.2.0
      - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30162
    description: The `serialization` option is supported now.
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7696
    description: The `argv0` option is supported now.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4598
    description: The `shell` option is supported now.
-->

* `command` {string} The command to run.
* `args` {string\[]} List of string arguments.
* `options` {Object}
  * `cwd` {string|URL} Current working directory of the child process.
  * `env` {Object} Environment key-value pairs. **Default:** `process.env`.
  * `argv0` {string} Explicitly set the value of `argv[0]` sent to the child
    process. This will be set to `command` if not specified.
  * `stdio` {Array|string} Child's stdio configuration (see
    [`options.stdio`][`stdio`]).
  * `detached` {boolean} Prepare child to run independently of its parent
    process. Specific behavior depends on the platform, see
    [`options.detached`][]).
  * `uid` {number} Sets the user identity of the process (see setuid(2)).
  * `gid` {number} Sets the group identity of the process (see setgid(2)).
  * `serialization` {string} Specify the kind of serialization used for sending
    messages between processes. Possible values are `'json'` and `'advanced'`.
    See [Advanced serialization][] for more details. **Default:** `'json'`.
  * `shell` {boolean|string} If `true`, runs `command` inside of a shell. Uses
    `'/bin/sh'` on Unix, and `process.env.ComSpec` on Windows. A different
    shell can be specified as a string. See [Shell requirements][] and
    [Default Windows shell][]. **Default:** `false` (no shell).
  * `windowsVerbatimArguments` {boolean} No quoting or escaping of arguments is
    done on Windows. Ignored on Unix. This is set to `true` automatically
    when `shell` is specified and is CMD. **Default:** `false`.
  * `windowsHide` {boolean} Hide the subprocess console window that would
    normally be created on Windows systems. **Default:** `false`.
  * `signal` {AbortSignal} allows aborting the child process using an
    AbortSignal.
  * `timeout` {number} In milliseconds the maximum amount of time the process
    is allowed to run. **Default:** `undefined`.
  * `killSignal` {string|integer} The signal value to be used when the spawned
    process will be killed by timeout or abort signal. **Default:** `'SIGTERM'`.
* Returns: {ChildProcess}

`child_process.spawn()` 方法使用给定的 `command` 生成一个新进程，命令行参数在 `args` 中。如果省略，`args` 默认为空数组.

**如果启用了 `shell` 选项，请不要将未经处理的用户输入传递给此函数。任何包含 shell 元字符的输入都可用于触发任意命令执行.**

第三个参数可用于指定其他选项，这些默认值:

```js
const defaults = {
  cwd: undefined,
  env: process.env
};
```

使用 `cwd` 指定生成进程的工作目录.
如果没有给出，默认是继承当前工作目录。如果给定，但路径不存在，子进程会发出“ENOENT”错误并立即退出。当命令不存在时也会发出 `ENOENT`.

使用 `env` 指定对新进程可见的环境变量，默认为 [`process.env`][].

`env` 中的 `undefined` 值将被忽略.

运行 `ls -lh /usr`、捕获 `stdout`、`stderr` 和退出代码的示例

```js
const { spawn } = require('node:child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

示例：运行 `ps ax | 的一种非常精细的方式grep ssh`

```js
const { spawn } = require('node:child_process');
const ps = spawn('ps', ['ax']);
const grep = spawn('grep', ['ssh']);

ps.stdout.on('data', (data) => {
  grep.stdin.write(data);
});

ps.stderr.on('data', (data) => {
  console.error(`ps stderr: ${data}`);
});

ps.on('close', (code) => {
  if (code !== 0) {
    console.log(`ps process exited with code ${code}`);
  }
  grep.stdin.end();
});

grep.stdout.on('data', (data) => {
  console.log(data.toString());
});

grep.stderr.on('data', (data) => {
  console.error(`grep stderr: ${data}`);
});

grep.on('close', (code) => {
  if (code !== 0) {
    console.log(`grep process exited with code ${code}`);
  }
});
```

检查失败的`spawn`的示例:

```js
const { spawn } = require('node:child_process');
const subprocess = spawn('bad_command');

subprocess.on('error', (err) => {
  console.error('Failed to start subprocess.');
});
```

某些平台（macOS、Linux）将使用 `argv[0]` 的值作为进程标题，而其他平台（Windows、SunOS）将使用 `command`.

Node.js 目前在启动时用 `process.execPath` 覆盖 `argv[0]`，因此 Node.js 子进程中的 `process.argv[0]` 将不匹配从父级，改为使用 `process.argv0` 属性检索它.

如果启用了 `signal` 选项，则在相应的 `AbortController` 上调用 `.abort()` 类似于在子进程上调用 `.kill()`，除了传递给回调的错误将是 `AbortError`:

```js
const { spawn } = require('node:child_process');
const controller = new AbortController();
const { signal } = controller;
const grep = spawn('grep', ['ssh'], { signal });
grep.on('error', (err) => {
  // This will be called with err being an AbortError if the controller aborts
});
controller.abort(); // Stops the child process
```

#### `options.detached`

<!-- YAML
added: v0.7.10
-->

在 Windows 上，将 `options.detached` 设置为 `true` 可以让子进程在父进程退出后继续运行。孩子将拥有自己的控制台窗口。一旦为子进程启用，它就不能被禁用.

在非 Windows 平台上，如果 `options.detached` 设置为 `true`，子进程将成为新进程组和会话的领导者。子进程可以在父进程退出后继续运行，无论它们是否分离。有关详细信息，请参阅 setid(2).

默认情况下，父级将等待分离的子级退出。为了防止父进程等待给定的 `subprocess` 退出，请使用 `subprocess.unref()` 方法。这样做会导致父级的事件循环不将子级包含在其引用计数中，从而允许父级独立于子级退出，除非子级和父级之间已建立 IPC 通道.

当使用 `detached` 选项启动一个长时间运行的进程时，进程将不会在父进程退出后继续在后台运行，除非它提供了一个未连接到父进程的 `stdio` 配置.
如果父级的`stdio`被继承，子级将保持连接到控制终端.

长时间运行进程的示例，通过分离并忽略其父“stdio”文件描述符，以忽略父进程的终止:

```js
const { spawn } = require('node:child_process');

const subprocess = spawn(process.argv[0], ['child_program.js'], {
  detached: true,
  stdio: 'ignore'
});

subprocess.unref();
```

或者，可以将子进程的输出重定向到文件中:

```js
const fs = require('node:fs');
const { spawn } = require('node:child_process');
const out = fs.openSync('./out.log', 'a');
const err = fs.openSync('./out.log', 'a');

const subprocess = spawn('prg', [], {
  detached: true,
  stdio: [ 'ignore', out, err ]
});

subprocess.unref();
```

#### `options.stdio`

<!-- YAML
added: v0.7.10
changes:
  - version:
      - v15.6.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/29412
    description: Added the `overlapped` stdio flag.
  - version: v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2727
    description: The value `0` is now accepted as a file descriptor.
-->

`options.stdio` 选项用于配置在父进程和子进程之间建立的管道。默认情况下，子进程的 stdin、stdout 和 stderr 被重定向到相应的 [`subprocess.stdin`][]、[`subprocess.stdout`][] 和 [`subprocess.stderr`][] 流ChildProcess`][] 对象。这相当于将 `options.stdio` 设置为 `['pipe', 'pipe', 'pipe']`.

为方便起见，`options.stdio` 可能是以下字符串之一:

* `'pipe'`: equivalent to `['pipe', 'pipe', 'pipe']` (the default)
* `'overlapped'`: equivalent to `['overlapped', 'overlapped', 'overlapped']`
* `'ignore'`: equivalent to `['ignore', 'ignore', 'ignore']`
* `'inherit'`: equivalent to `['inherit', 'inherit', 'inherit']` or `[0, 1, 2]`

否则，`options.stdio` 的值是一个数组，其中每个索引对应于子项中的一个 fd。 fds 0、1 和 2 分别对应于 stdin、stdout 和 stderr。可以指定额外的 fd 来在父子节点之间创建额外的管道。该值为以下之一:

1. `'pipe'`: Create a pipe between the child process and the parent process.
   The parent end of the pipe is exposed to the parent as a property on the
   `child_process` object as [`subprocess.stdio[fd]`][`subprocess.stdio`]. Pipes
   created for fds 0, 1, and 2 are also available as [`subprocess.stdin`][],
   [`subprocess.stdout`][] and [`subprocess.stderr`][], respectively.
   Currently, these are not actual Unix pipes and therefore the child process
   can not use them by their descriptor files,
   e.g. `/dev/fd/2` or `/dev/stdout`.
2. `'overlapped'`: Same as `'pipe'` except that the `FILE_FLAG_OVERLAPPED` flag
   is set on the handle. This is necessary for overlapped I/O on the child
   process's stdio handles. See the
   [docs](https://docs.microsoft.com/en-us/windows/win32/fileio/synchronous-and-asynchronous-i-o)
   for more details. This is exactly the same as `'pipe'` on non-Windows
   systems.
3. `'ipc'`: Create an IPC channel for passing messages/file descriptors
   between parent and child. A [`ChildProcess`][] may have at most one IPC
   stdio file descriptor. Setting this option enables the
   [`subprocess.send()`][] method. If the child is a Node.js process, the
   presence of an IPC channel will enable [`process.send()`][] and
   [`process.disconnect()`][] methods, as well as [`'disconnect'`][] and
   [`'message'`][] events within the child.

   Accessing the IPC channel fd in any way other than [`process.send()`][]
   or using the IPC channel with a child process that is not a Node.js instance
   is not supported.
4. `'ignore'`: Instructs Node.js to ignore the fd in the child. While Node.js
   will always open fds 0, 1, and 2 for the processes it spawns, setting the fd
   to `'ignore'` will cause Node.js to open `/dev/null` and attach it to the
   child's fd.
5. `'inherit'`: Pass through the corresponding stdio stream to/from the
   parent process. In the first three positions, this is equivalent to
   `process.stdin`, `process.stdout`, and `process.stderr`, respectively. In
   any other position, equivalent to `'ignore'`.
6. {Stream} object: Share a readable or writable stream that refers to a tty,
   file, socket, or a pipe with the child process. The stream's underlying
   file descriptor is duplicated in the child process to the fd that
   corresponds to the index in the `stdio` array. The stream must have an
   underlying descriptor (file streams do not until the `'open'` event has
   occurred).
7. Positive integer: The integer value is interpreted as a file descriptor
   that is currently open in the parent process. It is shared with the child
   process, similar to how {Stream} objects can be shared. Passing sockets
   is not supported on Windows.
8. `null`, `undefined`: Use default value. For stdio fds 0, 1, and 2 (in other
   words, stdin, stdout, and stderr) a pipe is created. For fd 3 and up, the
   default is `'ignore'`.

```js
const { spawn } = require('node:child_process');

// Child will use parent's stdios.
spawn('prg', [], { stdio: 'inherit' });

// Spawn child sharing only stderr.
spawn('prg', [], { stdio: ['pipe', 'pipe', process.stderr] });

// Open an extra fd=4, to interact with programs presenting a
// startd-style interface.
spawn('prg', [], { stdio: ['pipe', null, null, null, 'pipe'] });
```

_值得注意的是，当父子进程之间建立了IPC通道，并且子进程是Node.js进程时，子进程以未引用的IPC通道启动（使用`unref()`），直到子进程注册一个[`'disconnect'`][] 事件或 [`'message'`][] 事件的事件处理程序。这允许子进程正常退出，而不会被打开的 IPC 通道保持打开状态._

在类 Unix 操作系统上，[`child_process.spawn()`][] 方法在将事件循环与子进程分离之前同步执行内存操作。具有大内存占用的应用程序可能会发现频繁的 [`child_process.spawn()`][] 调用是一个瓶颈。有关详细信息，请参阅 [V8 问题 7381](https://bugs.chromium.org/p/v8/issues/detail?id=7381).

另请参阅：[`child_process.exec()`][] 和 [`child_process.fork()`][].

## Synchronous process creation

[`child_process.spawnSync()`][]、[`child_process.execSync()`][] 和 [`child_process.execFileSync()`][] 方法是同步的，会阻塞 Node.js 事件循环，暂停执行任何附加代码，直到衍生的进程退出.

像这样的阻塞调用对于简化通用脚本任务和简化启动时应用程序配置的加载/处理非常有用.

### `child_process.execFileSync(file[, args][, options])`

<!-- YAML
added: v0.11.12
changes:
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22409
    description: The `input` option can now be any `TypedArray` or a
                 `DataView`.
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10653
    description: The `input` option can now be a `Uint8Array`.
  - version:
    - v6.2.1
    - v4.5.0
    pr-url: https://github.com/nodejs/node/pull/6939
    description: The `encoding` option can now explicitly be set to `buffer`.
-->

* `file` {string} The name or path of the executable file to run.
* `args` {string\[]} List of string arguments.
* `options` {Object}
  * `cwd` {string|URL} Current working directory of the child process.
  * `input` {string|Buffer|TypedArray|DataView} The value which will be passed
    as stdin to the spawned process. Supplying this value will override
    `stdio[0]`.
  * `stdio` {string|Array} Child's stdio configuration. `stderr` by default will
    be output to the parent process' stderr unless `stdio` is specified.
    **Default:** `'pipe'`.
  * `env` {Object} Environment key-value pairs. **Default:** `process.env`.
  * `uid` {number} Sets the user identity of the process (see setuid(2)).
  * `gid` {number} Sets the group identity of the process (see setgid(2)).
  * `timeout` {number} In milliseconds the maximum amount of time the process
    is allowed to run. **Default:** `undefined`.
  * `killSignal` {string|integer} The signal value to be used when the spawned
    process will be killed. **Default:** `'SIGTERM'`.
  * `maxBuffer` {number} Largest amount of data in bytes allowed on stdout or
    stderr. If exceeded, the child process is terminated. See caveat at
    [`maxBuffer` and Unicode][]. **Default:** `1024 * 1024`.
  * `encoding` {string} The encoding used for all stdio inputs and outputs.
    **Default:** `'buffer'`.
  * `windowsHide` {boolean} Hide the subprocess console window that would
    normally be created on Windows systems. **Default:** `false`.
  * `shell` {boolean|string} If `true`, runs `command` inside of a shell. Uses
    `'/bin/sh'` on Unix, and `process.env.ComSpec` on Windows. A different
    shell can be specified as a string. See [Shell requirements][] and
    [Default Windows shell][]. **Default:** `false` (no shell).
* Returns: {Buffer|string} The stdout from the command.

`child_process.execFileSync()` 方法通常与 [`child_process.execFile()`][] 相同，只是该方法在子进程完全关闭之前不会返回。当遇到超时并发送 `killSignal` 时，该方法将在进程完全退出之前不会返回.

如果子进程拦截并处理了`SIGTERM`信号并没有退出，父进程仍然会一直等到子进程退出.

如果进程超时或有非零退出代码，此方法将抛出 [`Error`][]，其中将包含底层 [`child_process.spawnSync()`][] 的完整结果.

**如果启用了 `shell` 选项，请不要将未经处理的用户输入传递给此函数。任何包含 shell 元字符的输入都可用于触发任意命令执行.**

### `child_process.execSync(command[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22409
    description: The `input` option can now be any `TypedArray` or a
                 `DataView`.
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10653
    description: The `input` option can now be a `Uint8Array`.
-->

* `command` {string} The command to run.
* `options` {Object}
  * `cwd` {string|URL} Current working directory of the child process.
  * `input` {string|Buffer|TypedArray|DataView} The value which will be passed
    as stdin to the spawned process. Supplying this value will override
    `stdio[0]`.
  * `stdio` {string|Array} Child's stdio configuration. `stderr` by default will
    be output to the parent process' stderr unless `stdio` is specified.
    **Default:** `'pipe'`.
  * `env` {Object} Environment key-value pairs. **Default:** `process.env`.
  * `shell` {string} Shell to execute the command with. See
    [Shell requirements][] and [Default Windows shell][]. **Default:**
    `'/bin/sh'` on Unix, `process.env.ComSpec` on Windows.
  * `uid` {number} Sets the user identity of the process. (See setuid(2)).
  * `gid` {number} Sets the group identity of the process. (See setgid(2)).
  * `timeout` {number} In milliseconds the maximum amount of time the process
    is allowed to run. **Default:** `undefined`.
  * `killSignal` {string|integer} The signal value to be used when the spawned
    process will be killed. **Default:** `'SIGTERM'`.
  * `maxBuffer` {number} Largest amount of data in bytes allowed on stdout or
    stderr. If exceeded, the child process is terminated and any output is
    truncated. See caveat at [`maxBuffer` and Unicode][].
    **Default:** `1024 * 1024`.
  * `encoding` {string} The encoding used for all stdio inputs and outputs.
    **Default:** `'buffer'`.
  * `windowsHide` {boolean} Hide the subprocess console window that would
    normally be created on Windows systems. **Default:** `false`.
* Returns: {Buffer|string} The stdout from the command.

`child_process.execSync()` 方法通常与 [`child_process.exec()`][] 相同，只是该方法在子进程完全关闭之前不会返回。当遇到超时并发送 `killSignal` 时，该方法在进程完全退出之前不会返回。如果子进程拦截并处理了 `SIGTERM` 信号并且没有退出，则父进程会一直等到子进程退出.

如果进程超时或有非零退出代码，此方法将抛出.
[`Error`][] 对象将包含来自 [`child_process.spawnSync()`][] 的整个结果.

**切勿将未经处理的用户输入传递给此函数。任何包含 shell 元字符的输入都可用于触发任意命令执行.**

### `child_process.spawnSync(command[, args][, options])`

<!-- YAML
added: v0.11.12
changes:
  - version:
      - v16.4.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38862
    description: The `cwd` option can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22409
    description: The `input` option can now be any `TypedArray` or a
                 `DataView`.
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10653
    description: The `input` option can now be a `Uint8Array`.
  - version:
    - v6.2.1
    - v4.5.0
    pr-url: https://github.com/nodejs/node/pull/6939
    description: The `encoding` option can now explicitly be set to `buffer`.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4598
    description: The `shell` option is supported now.
-->

* `command` {string} The command to run.
* `args` {string\[]} List of string arguments.
* `options` {Object}
  * `cwd` {string|URL} Current working directory of the child process.
  * `input` {string|Buffer|TypedArray|DataView} The value which will be passed
    as stdin to the spawned process. Supplying this value will override
    `stdio[0]`.
  * `argv0` {string} Explicitly set the value of `argv[0]` sent to the child
    process. This will be set to `command` if not specified.
  * `stdio` {string|Array} Child's stdio configuration.
  * `env` {Object} Environment key-value pairs. **Default:** `process.env`.
  * `uid` {number} Sets the user identity of the process (see setuid(2)).
  * `gid` {number} Sets the group identity of the process (see setgid(2)).
  * `timeout` {number} In milliseconds the maximum amount of time the process
    is allowed to run. **Default:** `undefined`.
  * `killSignal` {string|integer} The signal value to be used when the spawned
    process will be killed. **Default:** `'SIGTERM'`.
  * `maxBuffer` {number} Largest amount of data in bytes allowed on stdout or
    stderr. If exceeded, the child process is terminated and any output is
    truncated. See caveat at [`maxBuffer` and Unicode][].
    **Default:** `1024 * 1024`.
  * `encoding` {string} The encoding used for all stdio inputs and outputs.
    **Default:** `'buffer'`.
  * `shell` {boolean|string} If `true`, runs `command` inside of a shell. Uses
    `'/bin/sh'` on Unix, and `process.env.ComSpec` on Windows. A different
    shell can be specified as a string. See [Shell requirements][] and
    [Default Windows shell][]. **Default:** `false` (no shell).
  * `windowsVerbatimArguments` {boolean} No quoting or escaping of arguments is
    done on Windows. Ignored on Unix. This is set to `true` automatically
    when `shell` is specified and is CMD. **Default:** `false`.
  * `windowsHide` {boolean} Hide the subprocess console window that would
    normally be created on Windows systems. **Default:** `false`.
* Returns: {Object}
  * `pid` {number} Pid of the child process.
  * `output` {Array} Array of results from stdio output.
  * `stdout` {Buffer|string} The contents of `output[1]`.
  * `stderr` {Buffer|string} The contents of `output[2]`.
  * `status` {number|null} The exit code of the subprocess, or `null` if the
    subprocess terminated due to a signal.
  * `signal` {string|null} The signal used to kill the subprocess, or `null` if
    the subprocess did not terminate due to a signal.
  * `error` {Error} The error object if the child process failed or timed out.

`child_process.spawnSync()` 方法通常与 [`child_process.spawn()`][] 相同，只是该函数在子进程完全关闭之前不会返回。当遇到超时并发送 `killSignal` 时，该方法在进程完全退出之前不会返回。如果进程拦截并处理了 SIGTERM 信号并且没有退出，则父进程将一直等待，直到子进程
退出.

**如果启用了 `shell` 选项，请不要将未经处理的用户输入传递给此函数。任何包含 shell 元字符的输入都可用于触发任意命令执行.**

## Class: `ChildProcess`

<!-- YAML
added: v2.2.0
-->

* Extends: {EventEmitter}

`ChildProcess` 的实例代表生成的子进程.

`ChildProcess` 的实例不打算直接创建。而是使用 [`child_process.spawn()`][]、[`child_process.exec()`][]、[`child_process.execFile()`][] 或 [`child_process.fork()`] [] 创建 `ChildProcess` 实例的方法.

### Event: `'close'`

<!-- YAML
added: v0.7.7
-->

* `code` {number} The exit code if the child exited on its own.
* `signal` {string} The signal by which the child process was terminated.

`'close'` 事件在进程结束后触发 - 并且_ 子进程的 stdio 流已关闭。这与 [`'exit'`][] 事件不同，因为多个进程可能共享相同的 stdio 流。 `'close'` 事件将始终在 [`'exit'`][] 已经发出后发出，或者如果孩子未能生成则 [`'error'`][].

```js
const { spawn } = require('node:child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process close all stdio with code ${code}`);
});

ls.on('exit', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

### Event: `'disconnect'`

<!-- YAML
added: v0.7.2
-->

`'disconnect'` 事件在父进程调用 [`subprocess.disconnect()`][] 方法或子进程调用 [`process.disconnect()`][] 后触发。断开连接后不再可能发送或接收消息，并且 [`subprocess.connected`][] 属性为 `false`.

### Event: `'error'`

* `err` {Error} The error.

`'error'` 事件在任何时候触发:

1. 无法生成该进程，或者
2. 该进程无法被杀死，或者
3. 向子进程发送消息失败.

发生错误后，“退出”事件可能会触发，也可能不会触发。当同时监听 `'exit'` 和 `'error'` 事件时，注意防止意外调用处理函数多次.

另见 [`subprocess.kill()`][] 和 [`subprocess.send()`][].

### Event: `'exit'`

<!-- YAML
added: v0.1.90
-->

* `code` {number} The exit code if the child exited on its own.
* `signal` {string} The signal by which the child process was terminated.

子进程结束后会发出“exit”事件。如果进程退出，`code` 是进程的最终退出代码，否则为 `null`。如果进程因收到信号而终止，“signal”为信号的字符串名称，否则为“null”。两者之一将始终为非`null`.

当触发 `'exit'` 事件时，子进程 stdio 流可能仍处于打开状态.

Node.js 为 `SIGINT` 和 `SIGTERM` 建立信号处理程序，并且 Node.js 进程不会因为收到这些信号而立即终止。
相反，Node.js 将执行一系列清理操作，然后重新引发已处理的信号.

见 waitpid(2).

### Event: `'message'`

<!-- YAML
added: v0.5.9
-->

* `message` {Object} A parsed JSON object or primitive value.
* `sendHandle` {Handle} A [`net.Socket`][] or [`net.Server`][] object, or
  undefined.

`'message'` 事件在子进程使用 [`process.send()`][] 发送消息时触发.

消息经过序列化和解析。生成的消息可能与最初发送的消息不同.

如果在生成子进程时将“序列化”选项设置为“高级”，则“消息”参数可以包含 JSON 无法包含的数据
代表.
更多细节见[高级序列化][].

### Event: `'spawn'`

<!-- YAML
added:
  - v15.1.0
  - v14.17.0
-->

一旦子进程成功生成，就会发出“spawn”事件.
如果子进程没有成功生成，则不会发出“spawn”事件，而是发出“错误”事件.

如果发出，“spawn”事件会在所有其他事件之前发生，并且在通过“stdout”或“stderr”接收任何数据之前.

`'spawn'` 事件将触发，无论是否发生错误**在**产生的进程中。例如，如果 `bash some-command` 成功生成,
`'spawn'` 事件将触发，但 `bash` 可能无法生成 `some-command`.
此警告也适用于使用 `{ shell: true }`.

### `subprocess.channel`

<!-- YAML
added: v7.1.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/30165
    description: The object no longer accidentally exposes native C++ bindings.
-->

* {Object} A pipe representing the IPC channel to the child process.

`subprocess.channel` 属性是对子 IPC 通道的引用。如果当前不存在 IPC 通道，则此属性为 `undefined`.

#### `subprocess.channel.ref()`

<!-- YAML
added: v7.1.0
-->

如果之前调用过`.unref()`，此方法使IPC通道保持父进程的事件循环运行.

#### `subprocess.channel.unref()`

<!-- YAML
added: v7.1.0
-->

此方法使 IPC 通道不保持父进程的事件循环运行，即使在通道打开时也让它完成.

### `subprocess.connected`

<!-- YAML
added: v0.7.2
-->

* {boolean} Set to `false` after `subprocess.disconnect()` is called.

`subprocess.connected` 属性指示是否仍然可以从子进程发送和接收消息。当 `subprocess.connected` 为 `false` 时，不再可能发送或接收消息.

### `subprocess.disconnect()`

<!-- YAML
added: v0.7.2
-->

关闭父子之间的 IPC 通道，允许子在没有其他连接保持其活动时优雅地退出。调用此方法后，父进程和子进程（分别）中的 `subprocess.connected` 和 `process.connected` 属性将设置为 `false`，并且不再可能在进程之间传递消息.

当没有接收到消息时，将发出 `'disconnect'` 事件。这通常会在调用 `subprocess.disconnect()` 后立即触发.

当子进程是 Node.js 实例时（例如，使用 [`child_process.fork()`][] 生成），也可以在子进程中调用 `process.disconnect()` 方法来关闭 IPC 通道.

### `subprocess.exitCode`

* {integer}

`subprocess.exitCode` 属性表示子进程的退出代码.
如果子进程仍在运行，则该字段将为“null”.

### `subprocess.kill([signal])`

<!-- YAML
added: v0.1.90
-->

* `signal` {number|string}
* Returns: {boolean}

`subprocess.kill()` 方法向子进程发送信号。如果没有给出参数，进程将收到“SIGTERM”信号。有关可用信号的列表，请参见 signal(7)。如果 kill(2) 成功，此函数返回 `true`，否则返回 `false`.

```js
const { spawn } = require('node:child_process');
const grep = spawn('grep', ['ssh']);

grep.on('close', (code, signal) => {
  console.log(
    `child process terminated due to receipt of signal ${signal}`);
});

// Send SIGHUP to process.
grep.kill('SIGHUP');
```

如果无法传递信号，[`ChildProcess`][] 对象可能会发出 [`'error'`][] 事件。向已经退出的子进程发送信号不是错误，但可能会产生无法预料的后果。具体来说，如果进程标识符 (PID) 已重新分配给另一个进程，则信号将被传递给该进程，而这可能会产生意想不到的结果.

虽然该函数被称为 `kill`，但传递给子进程的信号实际上可能不会终止该进程.

参考 kill(2).

在不存在 POSIX 信号的 Windows 上，`signal` 参数将被忽略，进程将被强行终止（类似于
`'SIGKILL'`）.
有关详细信息，请参阅 [信号事件][].

在 Linux 上，子进程的子进程在试图杀死其父进程时不会被终止。在 shell 中运行新进程或使用 `ChildProcess` 的 `shell` 选项时，可能会发生这种情况:

```js
'use strict';
const { spawn } = require('node:child_process');

const subprocess = spawn(
  'sh',
  [
    '-c',
    `node -e "setInterval(() => {
      console.log(process.pid, 'is alive')
    }, 500);"`,
  ], {
    stdio: ['inherit', 'inherit', 'inherit']
  }
);

setTimeout(() => {
  subprocess.kill(); // Does not terminate the Node.js process in the shell.
}, 2000);
```

### `subprocess.killed`

<!-- YAML
added: v0.5.10
-->

* {boolean} Set to `true` after `subprocess.kill()` is used to successfully
  send a signal to the child process.

`subprocess.killed` 属性指示子进程是否成功接收到来自 `subprocess.kill()` 的信号。 `killed` 属性并不表示子进程已被终止.

### `subprocess.pid`

<!-- YAML
added: v0.1.90
-->

* {integer|undefined}

返回子进程的进程标识符 (PID)。如果子进程由于错误而无法生成，则值为 `undefined` 并发出 `error`.

```js
const { spawn } = require('node:child_process');
const grep = spawn('grep', ['ssh']);

console.log(`Spawned child pid: ${grep.pid}`);
grep.stdin.end();
```

### `subprocess.ref()`

<!-- YAML
added: v0.7.10
-->

在调用 `subprocess.unref()` 后调用 `subprocess.ref()` 将恢复子进程删除的引用计数，迫使父进程在退出之前等待子进程退出.

```js
const { spawn } = require('node:child_process');

const subprocess = spawn(process.argv[0], ['child_program.js'], {
  detached: true,
  stdio: 'ignore'
});

subprocess.unref();
subprocess.ref();
```

### `subprocess.send(message[, sendHandle[, options]][, callback])`

<!-- YAML
added: v0.5.9
changes:
  - version: v5.8.0
    pr-url: https://github.com/nodejs/node/pull/5283
    description: The `options` parameter, and the `keepOpen` option
                 in particular, is supported now.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3516
    description: This method returns a boolean for flow control now.
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

当父子进程之间建立了 IPC 通道时（即使用 [`child_process.fork()`][]），可以使用 `subprocess.send()` 方法向子进程发送消息。当子进程是 Node.js 实例时，可以通过 [`'message'`][] 事件接收这些消息.

消息经过序列化和解析。生成的消息可能与最初发送的消息不同.

例如，在父脚本中:

```js
const cp = require('node:child_process');
const n = cp.fork(`${__dirname}/sub.js`);

n.on('message', (m) => {
  console.log('PARENT got message:', m);
});

// Causes the child to print: CHILD got message: { hello: 'world' }
n.send({ hello: 'world' });
```

And then the child script, `'sub.js'` might look like this:

```js
process.on('message', (m) => {
  console.log('CHILD got message:', m);
});

// Causes the parent to print: PARENT got message: { foo: 'bar', baz: null }
process.send({ foo: 'bar', baz: NaN });
```

子 Node.js 进程将有自己的 [`process.send()`][] 方法，允许子进程将消息发送回父进程.

发送 `{cmd: 'NODE_foo'}` 消息时有一种特殊情况。在 `cmd` 属性中包含 `NODE_` 前缀的消息保留在 Node.js 核心中使用，并且不会在子级的 [`'message'`][] 事件中发出。相反，此类消息是使用“internalMessage”事件发出的，并由 Node.js 在内部使用.
应用程序应避免使用此类消息或侦听“internalMessage”事件，因为它可能会更改，恕不另行通知.

可以传递给 `subprocess.send()` 的可选 `sendHandle` 参数用于将 TCP 服务器或套接字对象传递给子进程。子进程将接收对象作为传递给在 [`'message'`][] 事件上注册的回调函数的第二个参数。套接字中接收和缓冲的任何数据都不会发送给子进程.

可选的 `callback` 是一个函数，它在消息发送之后但在孩子可能收到消息之前被调用。使用单个参数调用该函数：成功时为 `null`，失败时为 [`Error`][] 对象.

如果没有提供 `callback` 函数并且无法发送消息，则 [`ChildProcess`][] 对象将发出 `'error'` 事件。例如，当子进程已经退出时，可能会发生这种情况.

如果通道已关闭或未发送消息的积压超过阈值，导致发送更多消息是不明智的，则“subprocess.send()”将返回“false”。否则，该方法返回 `true`。 `callback` 函数可用于实现流量控制.

#### Example: sending a server object

例如，可以使用“sendHandle”参数将 TCP 服务器对象的句柄传递给子进程，如下例所示:

```js
const subprocess = require('node:child_process').fork('subprocess.js');

// Open up the server object and send the handle.
const server = require('node:net').createServer();
server.on('connection', (socket) => {
  socket.end('handled by parent');
});
server.listen(1337, () => {
  subprocess.send('server', server);
});
```

The child would then receive the server object as:

```js
process.on('message', (m, server) => {
  if (m === 'server') {
    server.on('connection', (socket) => {
      socket.end('handled by child');
    });
  }
});
```

一旦服务器现在在父母和孩子之间共享，一些连接可以由父母处理，一些由孩子处理.

虽然上面的示例使用使用 `node:net` 模块创建的服务器，但 `node:dgram` 模块服务器使用完全相同的工作流程，除了监听 `'message'` 事件而不是 `'connection'` 和使用 `server.bind()` 而不是 `server.listen()`。但是，目前仅在 Unix 平台上支持.

#### Example: sending a socket object

类似地，`sendHandler` 参数可用于将套接字的句柄传递给子进程。下面的示例生成了两个孩子，每个孩子都处理具有“正常”或“特殊”优先级的连接:

```js
const { fork } = require('node:child_process');
const normal = fork('subprocess.js', ['normal']);
const special = fork('subprocess.js', ['special']);

// Open up the server and send sockets to child. Use pauseOnConnect to prevent
// the sockets from being read before they are sent to the child process.
const server = require('node:net').createServer({ pauseOnConnect: true });
server.on('connection', (socket) => {

  // If this is special priority...
  if (socket.remoteAddress === '74.125.127.100') {
    special.send('socket', socket);
    return;
  }
  // This is normal priority.
  normal.send('socket', socket);
});
server.listen(1337);
```

`subprocess.js` 将接收套接字句柄作为传递给事件回调函数的第二个参数:

```js
process.on('message', (m, socket) => {
  if (m === 'socket') {
    if (socket) {
      // Check that the client socket exists.
      // It is possible for the socket to be closed between the time it is
      // sent and the time it is received in the child process.
      socket.end(`Request handled with ${process.argv[2]} priority`);
    }
  }
});
```

不要在已传递给子进程的套接字上使用 `.maxConnections`.
父级无法跟踪套接字何时被销毁.

子进程中的任何“消息”处理程序都应验证“套接字”是否存在，因为在将连接发送给子进程期间连接可能已关闭.

### `subprocess.signalCode`

* {string|null}

`subprocess.signalCode` 属性表示子进程接收到的信号（如果有），否则为 `null`.

### `subprocess.spawnargs`

* {Array}

`subprocess.spawnargs` 属性表示子进程启动时使用的命令行参数的完整列表.

### `subprocess.spawnfile`

* {string}

`subprocess.spawnfile` 属性表示启动的子进程的可执行文件名.

对于 [`child_process.fork()`][]，其值将等于 [`process.execPath`][].
对于 [`child_process.spawn()`][]，其值为可执行文件的名称.
对于 [`child_process.exec()`][]，它的值将是启动子进程的 shell 的名称.

### `subprocess.stderr`

<!-- YAML
added: v0.1.90
-->

* {stream.Readable|null|undefined}

表示子进程的“stderr”的“可读流”.

If the child was spawned with `stdio[2]` set to anything other than `'pipe'`, then this will be `null`.

`subprocess.stderr` 是 `subprocess.stdio[2]` 的别名。两个属性将引用相同的值.

如果子进程无法成功生成，则 `subprocess.stderr` 属性可以是 `null` 或 `undefined`.

### `subprocess.stdin`

<!-- YAML
added: v0.1.90
-->

* {stream.Writable|null|undefined}

A `Writable Stream` that represents the child process's `stdin`.

如果子进程等待读取其所有输入，则子进程将不会继续，直到通过 `end()` 关闭此流.

如果孩子是在 `stdio[0]` 设置为 `'pipe'` 以外的任何内容的情况下生成的，那么这将是 `null`.

`subprocess.stdin` 是 `subprocess.stdio[0]` 的别名。两个属性将引用相同的值.

如果子进程无法成功生成，则 `subprocess.stdin` 属性可以是 `null` 或 `undefined`.

### `subprocess.stdio`

<!-- YAML
added: v0.7.10
-->

* {Array}

子进程的稀疏管道数组，对应于 [`stdio`][] 选项中传递给 [`child_process.spawn()`][] 的位置，这些位置已设置为值 `'pipe'`。 `subprocess.stdio[0]`、`subprocess.stdio[1]` 和 `subprocess.stdio[2]` 也可用作 `subprocess.stdin`、`subprocess.stdout` 和 `subprocess.stderr`，分别.

在下面的例子中，只有孩子的 fd `1`（stdout）被配置为管道，所以只有父母的 `subprocess.stdio[1]` 是一个流，数组中的所有其他值都是 `null`.

```js
const assert = require('node:assert');
const fs = require('node:fs');
const child_process = require('node:child_process');

const subprocess = child_process.spawn('ls', {
  stdio: [
    0, // Use parent's stdin for child.
    'pipe', // Pipe child's stdout to parent.
    fs.openSync('err.out', 'w'), // Direct child's stderr to a file.
  ]
});

assert.strictEqual(subprocess.stdio[0], null);
assert.strictEqual(subprocess.stdio[0], subprocess.stdin);

assert(subprocess.stdout);
assert.strictEqual(subprocess.stdio[1], subprocess.stdout);

assert.strictEqual(subprocess.stdio[2], null);
assert.strictEqual(subprocess.stdio[2], subprocess.stderr);
```

如果子进程无法成功生成，则 `subprocess.stdio` 属性可以是 `undefined`.

### `subprocess.stdout`

<!-- YAML
added: v0.1.90
-->

* {stream.Readable|null|undefined}

表示子进程的“stdout”的“可读流”.

如果孩子是在 `stdio[1]` 设置为 `'pipe'` 以外的任何值的情况下生成的，那么这将是 `null`.

`subprocess.stdout` 是 `subprocess.stdio[1]` 的别名。两个属性将引用相同的值.

```js
const { spawn } = require('node:child_process');

const subprocess = spawn('ls');

subprocess.stdout.on('data', (data) => {
  console.log(`Received chunk ${data}`);
});
```

The `subprocess.stdout` property can be `null` or `undefined`
if the child process could not be successfully spawned.

### `subprocess.unref()`

<!-- YAML
added: v0.7.10
-->

默认情况下，父级将等待分离的子级退出。为了防止父进程等待给定的 `subprocess` 退出，请使用 `subprocess.unref()` 方法。这样做会导致父级的事件循环不将子级包含在其引用计数中，从而允许父级独立于子级退出，除非子级和父级之间已建立 IPC 通道.

```js
const { spawn } = require('node:child_process');

const subprocess = spawn(process.argv[0], ['child_program.js'], {
  detached: true,
  stdio: 'ignore'
});

subprocess.unref();
```

## `maxBuffer` and Unicode

`maxBuffer` 选项指定 `stdout` 或 `stderr` 上允许的最大字节数。如果超过此值，则终止子进程.
这会影响包含多字节字符编码（例如 UTF-8 或 UTF-16）的输出。例如，`console.log('中文测试')` 将发送 13 个 UTF-8 编码字节到 `stdout`，尽管只有 4 个字符.

## Shell requirements

shell 应该理解 `-c` 开关。如果shell是`'cmd.exe'`，应该理解`/d /s /c`开关和命令行解析应该兼容.

## Default Windows shell

尽管 Microsoft 指定 `%COMSPEC%` 必须包含根环境中的 `'cmd.exe'` 的路径，但子进程并不总是受到相同要求的约束。因此，在可以生成 shell 的 `child_process` 函数中，如果 `process.env.ComSpec` 不可用，`'cmd.exe'` 用作备用.

## Advanced serialization

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

子进程支持 IPC 的序列化机制，该机制基于 [`node:v8` 模块的序列化 API][v8.serdes]，基于 [HTML 结构化克隆算法][]。这通常更强大，并且支持更多内置的 JavaScript 对象类型，例如 `BigInt`、`Map` 和 `Set`、`ArrayBuffer` 和 `TypedArray`、`Buffer`、`Error`、`RegExp` 等.

但是，这种格式不是 JSON 的完整超集，例如在此类内置类型的对象上设置的属性将不会通过序列化步骤传递。此外，性能可能不等同于 JSON，具体取决于传递数据的结构.
因此，此功能需要通过在调用 [`child_process.spawn()`][] 或 [`child_process.fork()`][] 时将 `serialization` 选项设置为 `'advanced'` 来选择加入.

[Advanced serialization]: #advanced-serialization
[Default Windows shell]: #default-windows-shell
[HTML structured clone algorithm]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm
[Shell requirements]: #shell-requirements
[Signal Events]: process.md#signal-events
[`'disconnect'`]: process.md#event-disconnect
[`'error'`]: #event-error
[`'exit'`]: #event-exit
[`'message'`]: process.md#event-message
[`ChildProcess`]: #class-childprocess
[`Error`]: errors.md#class-error
[`EventEmitter`]: events.md#class-eventemitter
[`child_process.exec()`]: #child_processexeccommand-options-callback
[`child_process.execFile()`]: #child_processexecfilefile-args-options-callback
[`child_process.execFileSync()`]: #child_processexecfilesyncfile-args-options
[`child_process.execSync()`]: #child_processexecsynccommand-options
[`child_process.fork()`]: #child_processforkmodulepath-args-options
[`child_process.spawn()`]: #child_processspawncommand-args-options
[`child_process.spawnSync()`]: #child_processspawnsynccommand-args-options
[`maxBuffer` and Unicode]: #maxbuffer-and-unicode
[`net.Server`]: net.md#class-netserver
[`net.Socket`]: net.md#class-netsocket
[`options.detached`]: #optionsdetached
[`process.disconnect()`]: process.md#processdisconnect
[`process.env`]: process.md#processenv
[`process.execPath`]: process.md#processexecpath
[`process.send()`]: process.md#processsendmessage-sendhandle-options-callback
[`stdio`]: #optionsstdio
[`subprocess.connected`]: #subprocessconnected
[`subprocess.disconnect()`]: #subprocessdisconnect
[`subprocess.kill()`]: #subprocesskillsignal
[`subprocess.send()`]: #subprocesssendmessage-sendhandle-options-callback
[`subprocess.stderr`]: #subprocessstderr
[`subprocess.stdin`]: #subprocessstdin
[`subprocess.stdio`]: #subprocessstdio
[`subprocess.stdout`]: #subprocessstdout
[`util.promisify()`]: util.md#utilpromisifyoriginal
[synchronous counterparts]: #synchronous-process-creation
[v8.serdes]: v8.md#serialization-api
