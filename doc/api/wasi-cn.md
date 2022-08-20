# WebAssembly System Interface (WASI)

<!--introduced_in=v12.16.0-->

> Stability: 1 - Experimental

<!-- source_link=lib/wasi.js -->

WASI API 提供了 [WebAssembly System Interface][] 规范的实现。 WASI 让沙盒 WebAssembly 应用程序通过一系列类似 POSIX 的函数访问底层操作系统.

```mjs
import { readFile } from 'node:fs/promises';
import { WASI } from 'wasi';
import { argv, env } from 'node:process';

const wasi = new WASI({
  args: argv,
  env,
  preopens: {
    '/sandbox': '/some/real/path/that/wasm/can/access'
  }
});

// Some WASI binaries require:
//   const importObject = { wasi_unstable: wasi.wasiImport };
const importObject = { wasi_snapshot_preview1: wasi.wasiImport };

const wasm = await WebAssembly.compile(
  await readFile(new URL('./demo.wasm', import.meta.url))
);
const instance = await WebAssembly.instantiate(wasm, importObject);

wasi.start(instance);
```

```cjs
'use strict';
const { readFile } = require('node:fs/promises');
const { WASI } = require('wasi');
const { argv, env } = require('node:process');
const { join } = require('node:path');

const wasi = new WASI({
  args: argv,
  env,
  preopens: {
    '/sandbox': '/some/real/path/that/wasm/can/access'
  }
});

// Some WASI binaries require:
//   const importObject = { wasi_unstable: wasi.wasiImport };
const importObject = { wasi_snapshot_preview1: wasi.wasiImport };

(async () => {
  const wasm = await WebAssembly.compile(
    await readFile(join(__dirname, 'demo.wasm'))
  );
  const instance = await WebAssembly.instantiate(wasm, importObject);

  wasi.start(instance);
})();
```

要运行上面的示例，请创建一个名为 `demo.wat` 的新 WebAssembly 文本格式文件:

```text
(module
    ;; Import the required fd_write WASI function which will write the given io vectors to stdout
    ;; The function signature for fd_write is:
    ;; (File Descriptor, *iovs, iovs_len, nwritten) -> Returns number of bytes written
    (import "wasi_snapshot_preview1" "fd_write" (func $fd_write (param i32 i32 i32 i32) (result i32)))

    (memory 1)
    (export "memory" (memory 0))

    ;; Write 'hello world\n' to memory at an offset of 8 bytes
    ;; Note the trailing newline which is required for the text to appear
    (data (i32.const 8) "hello world\n")

    (func $main (export "_start")
        ;; Creating a new io vector within linear memory
        (i32.store (i32.const 0) (i32.const 8))  ;; iov.iov_base - This is a pointer to the start of the 'hello world\n' string
        (i32.store (i32.const 4) (i32.const 12))  ;; iov.iov_len - The length of the 'hello world\n' string

        (call $fd_write
            (i32.const 1) ;; file_descriptor - 1 for stdout
            (i32.const 0) ;; *iovs - The pointer to the iov array, which is stored at memory location 0
            (i32.const 1) ;; iovs_len - We're printing 1 string stored in an iov - so one.
            (i32.const 20) ;; nwritten - A place in memory to store the number of bytes written
        )
        drop ;; Discard the number of bytes written from the top of the stack
    )
)
```

使用 [wabt](https://github.com/WebAssembly/wabt) 将 `.wat` 编译为 `.wasm`

```console
$ wat2wasm demo.wat
```

运行此示例需要 `--experimental-wasi-unstable-preview1` CLI 参数.

## Class: `WASI`

<!-- YAML
added:
 - v13.3.0
 - v12.16.0
-->

`WASI` 类提供了 WASI 系统调用 API 和用于处理基于 WASI 的应用程序的其他便利方法。每个“WASI”实例代表一个不同的沙盒环境。出于安全考虑，每个“WASI”实例都必须明确配置其命令行参数、环境变量和沙箱目录结构.

### `new WASI([options])`

<!-- YAML
added:
 - v13.3.0
 - v12.16.0
-->

* `options` {Object}
  * `args` {Array} An array of strings that the WebAssembly application will
    see as command-line arguments. The first argument is the virtual path to the
    WASI command itself. **Default:** `[]`.
  * `env` {Object} An object similar to `process.env` that the WebAssembly
    application will see as its environment. **Default:** `{}`.
  * `preopens` {Object} This object represents the WebAssembly application's
    sandbox directory structure. The string keys of `preopens` are treated as
    directories within the sandbox. The corresponding values in `preopens` are
    the real paths to those directories on the host machine.
  * `returnOnExit` {boolean} By default, WASI applications terminate the Node.js
    process via the `__wasi_proc_exit()` function. Setting this option to `true`
    causes `wasi.start()` to return the exit code rather than terminate the
    process. **Default:** `false`.
  * `stdin` {integer} The file descriptor used as standard input in the
    WebAssembly application. **Default:** `0`.
  * `stdout` {integer} The file descriptor used as standard output in the
    WebAssembly application. **Default:** `1`.
  * `stderr` {integer} The file descriptor used as standard error in the
    WebAssembly application. **Default:** `2`.

### `wasi.start(instance)`

<!-- YAML
added:
 - v13.3.0
 - v12.16.0
-->

* `instance` {WebAssembly.Instance}

尝试通过调用 `_start()` 导出来将 `instance` 作为 WASI 命令开始执行。如果 `instance` 不包含 `_start()` 导出，或者 `instance` 包含 `_initialize()` 导出，则抛出异常.

`start()` 要求 `instance` 导出一个名为 `memory` 的 [`WebAssembly.Memory`][]。如果 `instance` 没有 `memory` 导出，则会引发异常.

如果 `start()` 被多次调用，则抛出异常.

### `wasi.initialize(instance)`

<!-- YAML
added:
 - v14.6.0
 - v12.19.0
-->

* `instance` {WebAssembly.Instance}

尝试通过调用其 _initialize() 导出（如果存在）将 `instance` 初始化为 WASI 反应器。如果 `instance` 包含 `_start()` 导出，则抛出异常.

`initialize()` 要求 `instance` 导出一个名为 `memory` 的 [`WebAssembly.Memory`][]。如果 `instance` 没有 `memory` 导出，则会引发异常.

如果 `initialize()` 被多次调用，则抛出异常.

### `wasi.wasiImport`

<!-- YAML
added:
 - v13.3.0
 - v12.16.0
-->

* {Object}

`wasiImport` is an object that implements the WASI system call API. This object
should be passed as the `wasi_snapshot_preview1` import during the instantiation
of a [`WebAssembly.Instance`][].

[WebAssembly System Interface]: https://wasi.dev/
[`WebAssembly.Instance`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Instance
[`WebAssembly.Memory`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Memory
