# 文件系统

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!--name=fs-->

<!-- source_link=lib/fs.js -->

这`node:fs`模块允许在
以标准 POSIX 函数为模型的方式。

要使用基于承诺的 API，请执行以下操作：

```mjs
import * as fs from 'node:fs/promises';
```

```cjs
const fs = require('node:fs/promises');
```

要使用回调和同步 API，请执行以下操作：

```mjs
import * as fs from 'node:fs';
```

```cjs
const fs = require('node:fs');
```

所有文件系统操作都具有同步、回调和基于承诺的操作
表单，并且可以使用 CommonJS 语法和 ES6 模块 （ESM） 进行访问。

## 承诺示例

基于承诺的操作返回在
异步操作已完成。

```mjs
import { unlink } from 'node:fs/promises';

try {
  await unlink('/tmp/hello');
  console.log('successfully deleted /tmp/hello');
} catch (error) {
  console.error('there was an error:', error.message);
}
```

```cjs
const { unlink } = require('node:fs/promises');

(async function(path) {
  try {
    await unlink(path);
    console.log(`successfully deleted ${path}`);
  } catch (error) {
    console.error('there was an error:', error.message);
  }
})('/tmp/hello');
```

## 回调示例

回调表单将完成回调函数作为其最后一个函数
参数，并异步调用操作。传递给的参数
完成回调取决于方法，但第一个参数始终是
为异常保留。如果操作已成功完成，则
第一个参数是`null`或`undefined`.

```mjs
import { unlink } from 'node:fs';

unlink('/tmp/hello', (err) => {
  if (err) throw err;
  console.log('successfully deleted /tmp/hello');
});
```

```cjs
const { unlink } = require('node:fs');

unlink('/tmp/hello', (err) => {
  if (err) throw err;
  console.log('successfully deleted /tmp/hello');
});
```

基于回调的版本`node:fs`模块 API 优于
在最大性能时使用承诺 API（无论是在
执行时间和内存分配）是必需的。

## 同步示例

同步 API 会阻塞 Node.js事件循环和进一步的 JavaScript
执行，直到操作完成。立即引发异常
并可以使用`try…catch`，或者可以允许冒泡。

```mjs
import { unlinkSync } from 'node:fs';

try {
  unlinkSync('/tmp/hello');
  console.log('successfully deleted /tmp/hello');
} catch (err) {
  // handle the error
}
```

```cjs
const { unlinkSync } = require('node:fs');

try {
  unlinkSync('/tmp/hello');
  console.log('successfully deleted /tmp/hello');
} catch (err) {
  // handle the error
}
```

## Promises API

<!-- YAML
added: v10.0.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31553
    description: Exposed as `require('fs/promises')`.
  - version:
    - v11.14.0
    - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/26581
    description: This API is no longer experimental.
  - version: v10.1.0
    pr-url: https://github.com/nodejs/node/pull/20504
    description: The API is accessible via `require('fs').promises` only.
-->

这`fs/promises`API 提供异步文件系统方法，这些方法可返回
承诺。

承诺 API 使用底层 Node.js 线程池来执行文件
系统操作脱离事件循环线程。这些操作不是
已同步或线程安全。执行多个时必须小心
可能会对同一文件进行并发修改或发生数据损坏。

### 类：`FileHandle`

<!-- YAML
added: v10.0.0
-->

{FileHandle} 对象是数字文件描述符的对象包装器。

{FileHandle} 对象的实例由`fsPromises.open()`
方法。

所有 {FileHandle} 对象都是 {EventEmitter}s。

如果 {FileHandle} 未使用`filehandle.close()`方法，它将
尝试自动关闭文件描述符并发出进程警告，
有助于防止内存泄漏。请不要依赖此行为，因为
它可能不可靠，文件可能无法关闭。相反，始终显式
关闭 {FileHandle}s.Node.js可能会在将来更改此行为。

#### 事件：`'close'`

<!-- YAML
added: v15.4.0
-->

这`'close'`事件在 {FileHandle} 已关闭且不能
使用时间更长。

#### `filehandle.appendFile(data[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version:
      - v15.14.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37490
    description: The `data` argument supports `AsyncIterable`, `Iterable`, and `Stream`.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `data` parameter won't coerce unsupported input to
                 strings anymore.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图|AsyncIterable|可迭代|流}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
*   返回：{承诺} 履行`undefined`成功后。

的别名[`filehandle.writeFile()`][filehandle.writeFile()].

对文件句柄进行操作时，无法更改设置的模式
到[`fsPromises.open()`][fsPromises.open()].因此，这等效于
[`filehandle.writeFile()`][filehandle.writeFile()].

#### `filehandle.chmod(mode)`

<!-- YAML
added: v10.0.0
-->

*   `mode`{整数} 文件模式位掩码。
*   返回：{承诺} 履行`undefined`成功后。

修改文件的权限。参见 chmod（2）。

#### `filehandle.chown(uid, gid)`

<!-- YAML
added: v10.0.0
-->

*   `uid`{整数}文件的新所有者的用户 ID。
*   `gid`{整数}文件的新组的组 ID。
*   返回：{承诺} 履行`undefined`成功后。

更改文件的所有权。chown（2） 的包装器。

#### `filehandle.close()`

<!-- YAML
added: v10.0.0
-->

*   返回：{承诺} 履行`undefined`成功后。

在等待对句柄执行任何挂起操作后关闭文件句柄
完成。

```mjs
import { open } from 'node:fs/promises';

let filehandle;
try {
  filehandle = await open('thefile.txt', 'r');
} finally {
  await filehandle?.close();
}
```

#### `filehandle.createReadStream([options])`

<!-- YAML
added: v16.11.0
-->

*   `options`{对象}
    *   `encoding`{字符串}**违约：** `null`
    *   `autoClose`{布尔值}**违约：** `true`
    *   `emitClose`{布尔值}**违约：** `true`
    *   `start`{整数}
    *   `end`{整数}**违约：** `Infinity`
    *   `highWaterMark`{整数}**违约：** `64 * 1024`
*   返回值：{fs。读取流}

与 16 KiB 默认值不同`highWaterMark`对于 {流。可读}，流
此方法返回的默认值`highWaterMark`的 64 KiB。

`options`可以包括`start`和`end`要从中读取一定范围的字节的值
该文件而不是整个文件。双`start`和`end`具有包容性，并且
从 0 开始计数，允许的值位于
\[0,[`Number.MAX_SAFE_INTEGER`][Number.MAX_SAFE_INTEGER]]范围。如果`start`是
省略或`undefined`,`filehandle.createReadStream()`按顺序读取
当前文件位置。这`encoding`可以是接受的任何一个
{缓冲区}.

如果`FileHandle`指向仅支持阻止的字符设备
读取（如键盘或声卡），读取操作直到数据完成
可用。这可以防止进程退出，并且流从
自然关闭。

默认情况将发出`'close'`事件之后
摧毁。 设置`emitClose`选项`false`以更改此行为。

```mjs
import { open } from 'node:fs/promises';

const fd = await open('/dev/input/event0');
// Create a stream from some character device.
const stream = fd.createReadStream();
setTimeout(() => {
  stream.close(); // This may not close the stream.
  // Artificially marking end-of-stream, as if the underlying resource had
  // indicated end-of-file by itself, allows the stream to close.
  // This does not cancel pending read operations, and if there is such an
  // operation, the process may still not be able to exit successfully
  // until it finishes.
  stream.push(null);
  stream.read(0);
}, 100);
```

如果`autoClose`为 false，则文件描述符不会关闭，即使
出现错误。应用程序有责任将其关闭并
确保没有文件描述符泄漏。如果`autoClose`设置为 true（默认值
行为），开`'error'`或`'end'`文件描述符将关闭
自然而然。

读取长度为 100 个字节的文件的最后 10 个字节的示例：

```mjs
import { open } from 'node:fs/promises';

const fd = await open('sample.txt');
fd.createReadStream({ start: 90, end: 99 });
```

#### `filehandle.createWriteStream([options])`

<!-- YAML
added: v16.11.0
-->

*   `options`{对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
    *   `autoClose`{布尔值}**违约：** `true`
    *   `emitClose`{布尔值}**违约：** `true`
    *   `start`{整数}
*   返回值：{fs。写入流}

`options`还可以包括`start`选项以允许在某个位置写入数据
位置超过文件开头，允许的值位于
\[0,[`Number.MAX_SAFE_INTEGER`][Number.MAX_SAFE_INTEGER]]范围。修改文件而不是
替换它可能需要`flags` `open`要设置为的选项`r+`而不是
默认值`r`.这`encoding`可以是 {Buffer} 接受的任何一个。

如果`autoClose`设置为 true（默认行为）打开`'error'`或`'finish'`
文件描述符将自动关闭。如果`autoClose`是假的，
则文件描述符不会关闭，即使存在错误也是如此。
应用程序有责任关闭它并确保没有
文件描述符泄漏。

默认情况将发出`'close'`事件之后
摧毁。 设置`emitClose`选项`false`以更改此行为。

#### `filehandle.datasync()`

<!-- YAML
added: v10.0.0
-->

*   返回：{承诺} 履行`undefined`成功后。

将与文件关联的所有当前排队的 I/O 操作强制到
操作系统的同步 I/O 完成状态。请参阅 POSIX
fdatasync（2） 文档了解详细信息。

与`filehandle.sync`此方法不会刷新已修改的元数据。

#### `filehandle.fd`

<!-- YAML
added: v10.0.0
-->

*   {数字}由 {FileHandle} 对象管理的数字文件描述符。

#### `filehandle.read(buffer, offset, length, position)`

<!-- YAML
added: v10.0.0
-->

*   `buffer`{缓冲区|TypedArray|DataView} 将用
    文件数据读取。
*   `offset`{整数}缓冲区中要开始填充的位置。
*   `length`{整数}要读取的字节数。
*   `position`{integer|null}开始从
    文件。如果`null`，将从当前文件位置读取数据，以及
    该位置将更新。如果`position`是一个整数，当前
    文件位置将保持不变。
*   返回：{Promise} 成功后对具有两个属性的对象实现：
    *   `bytesRead`{整数}读取的字节数
    *   `buffer`{缓冲区|TypedArray|DataView} 对传入的引用`buffer`
        论点。

从文件中读取数据并将其存储在给定的缓冲区中。

如果未同时修改文件，则当
读取的字节数为零。

#### `filehandle.read([options])`

<!-- YAML
added:
 - v13.11.0
 - v12.17.0
-->

*   `options`{对象}
    *   `buffer`{缓冲区|TypedArray|DataView} 将用
        文件数据读取。**违约：** `Buffer.alloc(16384)`
    *   `offset`{整数}缓冲区中要开始填充的位置。
        **违约：** `0`
    *   `length`{整数}要读取的字节数。**违约：**
        `buffer.byteLength - offset`
    *   `position`{integer|null}开始从
        文件。如果`null`，将从当前文件位置读取数据，以及
        该位置将更新。如果`position`是一个整数，当前
        文件位置将保持不变。**违约：**:`null`
*   返回：{Promise} 成功后对具有两个属性的对象实现：
    *   `bytesRead`{整数}读取的字节数
    *   `buffer`{缓冲区|TypedArray|DataView} 对传入的引用`buffer`
        论点。

从文件中读取数据并将其存储在给定的缓冲区中。

如果未同时修改文件，则当
读取的字节数为零。

#### `filehandle.read(buffer[, options])`

<!-- YAML
added: v18.2.0
-->

*   `buffer`{缓冲区|TypedArray|DataView} 将用
    文件数据读取。
*   `options`{对象}
    *   `offset`{整数}缓冲区中要开始填充的位置。
        **违约：** `0`
    *   `length`{整数}要读取的字节数。**违约：**
        `buffer.byteLength - offset`
    *   `position`{整数}开始从
        文件。如果`null`，将从当前文件位置读取数据，以及
        该位置将更新。如果`position`是一个整数，当前
        文件位置将保持不变。**违约：**:`null`
*   返回：{Promise} 成功后对具有两个属性的对象实现：
    *   `bytesRead`{整数}读取的字节数
    *   `buffer`{缓冲区|TypedArray|DataView} 对传入的引用`buffer`
        论点。

从文件中读取数据并将其存储在给定的缓冲区中。

如果未同时修改文件，则当
读取的字节数为零。

#### `filehandle.readableWebStream()`

<!-- YAML
added: v17.0.0
-->

> 稳定性： 1 - 实验

*   返回：{可读流}

返回`ReadableStream`可用于读取文件数据。

如果多次调用此方法或调用此方法，则会引发错误
在`FileHandle`已关闭或已关闭。

```mjs
import {
  open,
} from 'node:fs/promises';

const file = await open('./some/file/to/read');

for await (const chunk of file.readableWebStream())
  console.log(chunk);

await file.close();
```

```cjs
const {
  open,
} = require('node:fs/promises');

(async () => {
  const file = await open('./some/file/to/read');

  for await (const chunk of file.readableWebStream())
    console.log(chunk);

  await file.close();
})();
```

而`ReadableStream`将读取文件完成，它不会
关闭`FileHandle`自然而然。用户代码仍必须调用
`fileHandle.close()`方法。

#### `filehandle.readFile(options)`

<!-- YAML
added: v10.0.0
-->

*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `null`
    *   `signal`{中止信号} 允许中止正在进行的读取文件
*   返回：{承诺} 成功读取后履行
    文件。如果未指定编码（使用`options.encoding`），则数据为
    作为 {Buffer} 对象返回。否则，数据将是一个字符串。

异步读取文件的全部内容。

如果`options`是一个字符串，然后它指定`encoding`.

{FileHandle} 必须支持读取。

如果一个或多个`filehandle.read()`调用是在文件句柄上进行的，然后对
`filehandle.readFile()`调用后，将从当前读取数据
位置直到文件末尾。它并不总是从头开始阅读
的文件。

#### `filehandle.readv(buffers[, position])`

<!-- YAML
added:
 - v13.13.0
 - v12.17.0
-->

*   `buffers`{Buffer\[]|TypedArray\[]|DataView\[]}
*   `position`{integer|null}从文件开头开始的偏移量，其中
    应从中读取数据。如果`position`不是`number`，数据将
    从当前位置读取。**违约：** `null`
*   返回：{Promise} 成功时完成包含两个属性的对象：
    *   `bytesRead`{整数} 读取的字节数
    *   `buffers`{Buffer\[]|TypedArray\[]|DataView\[]} 属性包含
        对`buffers`输入。

从文件中读取并写入 {ArrayBufferView} 的数组

#### `filehandle.stat([options])`

<!-- YAML
added: v10.0.0
changes:
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
-->

*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   返回：{Promise} 用 {fs 实现。文件的统计信息}。

#### `filehandle.sync()`

<!-- YAML
added: v10.0.0
-->

*   返回：{承诺} 履行`undefined`成功后。

请求将打开的文件描述符的所有数据刷新到存储中
装置。具体实现是特定于操作系统和设备的。
有关更多详细信息，请参阅 POSIX fsync（2） 文档。

#### `filehandle.truncate(len)`

<!-- YAML
added: v10.0.0
-->

*   `len`{整数}**违约：** `0`
*   返回：{承诺} 履行`undefined`成功后。

截断文件。

如果文件大于`len`字节，只有第一个`len`字节数将为
保留在文件中。

下面的示例仅保留文件的前四个字节：

```mjs
import { open } from 'node:fs/promises';

let filehandle = null;
try {
  filehandle = await open('temp.txt', 'r+');
  await filehandle.truncate(4);
} finally {
  await filehandle?.close();
}
```

如果文件以前短于`len`字节，它被扩展，并且
扩展部分填充空字节 （`'\0'`):

如果`len`则为负数`0`将使用。

#### `filehandle.utimes(atime, mtime)`

<!-- YAML
added: v10.0.0
-->

*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}
*   返回： {承诺}

更改 {FileHandle} 引用的对象的文件系统时间戳
然后解决承诺，在成功时没有争论。

#### `filehandle.write(buffer, offset[, length[, position]])`

<!-- YAML
added: v10.0.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `buffer` parameter won't coerce unsupported input to
                 buffers anymore.
-->

*   `buffer`{缓冲区|TypedArray|数据视图}
*   `offset`{整数}由内而外的起始位置`buffer`其中数据
    开始写。
*   `length`{整数}字节数`buffer`写。**违约：**
    `buffer.byteLength - offset`
*   `position`{integer|null}从文件开头开始的偏移量，其中
    数据来自`buffer`应该写。如果`position`不是`number`,
    数据将写入当前位置。参见 POSIX pwrite（2）
    文档以获取更多详细信息。**违约：** `null`
*   返回： {承诺}

写`buffer`添加到该文件。

使用包含两个属性的对象解析 promise：

*   `bytesWritten`{整数} 写入的字节数
*   `buffer`{缓冲区|TypedArray|DataView} 对
    `buffer`写。

使用不安全`filehandle.write()`在同一文件上多次
无需等待承诺得到解决（或拒绝）。为此
场景， 使用[`filehandle.createWriteStream()`][filehandle.createWriteStream()].

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

#### `filehandle.write(buffer[, options])`

<!-- YAML
added: v18.3.0
-->

*   `buffer`{缓冲区|TypedArray|数据视图}
*   `options`{对象}
    *   `offset`{整数}**违约：** `0`
    *   `length`{整数}**违约：** `buffer.byteLength - offset`
    *   `position`{整数}**违约：** `null`
*   返回： {承诺}

写`buffer`添加到该文件。

与上述类似`filehandle.write`函数，此版本采用
自选`options`对象。如果不是`options`对象被指定，它将
默认值为上述值。

#### `filehandle.write(string[, position[, encoding]])`

<!-- YAML
added: v10.0.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `string` parameter won't coerce unsupported input to
                 strings anymore.
-->

*   `string`{字符串}
*   `position`{integer|null}从文件开头开始的偏移量，其中
    数据来自`string`应该写。如果`position`不是`number`这
    数据将写入当前位置。参见 POSIX pwrite（2）
    文档以获取更多详细信息。**违约：** `null`
*   `encoding`{字符串}预期的字符串编码。**违约：** `'utf8'`
*   返回： {承诺}

写`string`添加到该文件。如果`string`不是字符串，承诺是
被拒绝并显示错误。

使用包含两个属性的对象解析 promise：

*   `bytesWritten`{整数} 写入的字节数
*   `buffer`{字符串} 对`string`写。

使用不安全`filehandle.write()`在同一文件上多次
无需等待承诺得到解决（或拒绝）。为此
场景， 使用[`filehandle.createWriteStream()`][filehandle.createWriteStream()].

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

#### `filehandle.writeFile(data, options)`

<!-- YAML
added: v10.0.0
changes:
  - version:
      - v15.14.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37490
    description: The `data` argument supports `AsyncIterable`, `Iterable`, and `Stream`.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `data` parameter won't coerce unsupported input to
                 strings anymore.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图|AsyncIterable|可迭代|流}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}当`data`是一个
        字符串。**违约：** `'utf8'`
*   返回： {承诺}

异步将数据写入文件，如果文件已存在，则替换该文件。
`data`可以是字符串、缓冲区、{AsyncIterable} 或 {Iterable} 对象。
承诺得到了解决，成功时没有争论。

如果`options`是一个字符串，然后它指定`encoding`.

{FileHandle} 必须支持写入。

使用不安全`filehandle.writeFile()`在同一文件上多次
无需等待承诺得到解决（或拒绝）。

如果一个或多个`filehandle.write()`调用是在文件句柄上进行的，然后对
`filehandle.writeFile()`调用后，数据将从
当前位置直到文件末尾。它并不总是从
文件的开头。

#### `filehandle.writev(buffers[, position])`

<!-- YAML
added: v12.9.0
-->

*   `buffers`{Buffer\[]|TypedArray\[]|DataView\[]}
*   `position`{integer|null}从文件开头开始的偏移量，其中
    数据来自`buffers`应该写。如果`position`不是`number`,
    数据将写入当前位置。**违约：** `null`
*   返回： {承诺}

将 {ArrayBufferView} s 的数组写入该文件。

使用包含两个属性的对象解析 promise：

*   `bytesWritten`{整数} 写入的字节数
*   `buffers`{Buffer\[]|TypedArray\[]|DataView\[]} 对`buffers`
    输入。

打电话是不安全的`writev()`在同一文件上多次，无需等待
使承诺得到解决（或被拒绝）。

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

### `fsPromises.access(path[, mode])`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{整数}**违约：** `fs.constants.F_OK`
*   返回：{承诺} 履行`undefined`成功后。

测试用户对`path`.
这`mode`参数是一个可选整数，用于指定可访问性
要执行的检查。`mode`应为值`fs.constants.F_OK`
或由以下任一项的按位 OR 组成的掩码`fs.constants.R_OK`,
`fs.constants.W_OK`和`fs.constants.X_OK`（例如
`fs.constants.W_OK | fs.constants.R_OK`).检查[文件访问常量][File access constants]为
可能的值`mode`.

如果辅助功能检查成功，则承诺在无
价值。如果任何辅助功能检查失败，则承诺将被拒绝
带有 {Error} 对象。下面的示例检查文件
`/etc/passwd`可以由当前进程读取和写入。

```mjs
import { access, constants } from 'node:fs/promises';

try {
  await access('/etc/passwd', constants.R_OK | constants.W_OK);
  console.log('can access');
} catch {
  console.error('cannot access');
}
```

用`fsPromises.access()`以检查文件的可访问性
叫`fsPromises.open()`不建议这样做。这样做会引入一场竞赛
条件，因为其他进程可能会在两者之间更改文件的状态
调用。相反，用户代码应直接打开/读/写文件并处理
如果文件不可访问，则引发错误。

### `fsPromises.appendFile(path, data[, options])`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址|FileHandle} 文件名或 {FileHandle}
*   `data`{字符串|缓冲区}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `mode`{整数}**违约：** `0o666`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'a'`.
*   返回：{承诺} 履行`undefined`成功后。

将数据异步追加到文件，如果尚未将数据追加到文件，则创建该文件
存在。`data`可以是字符串或 {缓冲区}。

如果`options`是一个字符串，然后它指定`encoding`.

这`mode`选项仅影响新创建的文件。看[`fs.open()`][fs.open()]
了解更多详情。

这`path`可以指定为已打开的 {FileHandle}
用于追加（使用`fsPromises.open()`).

### `fsPromises.chmod(path, mode)`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{string|integer}
*   返回：{承诺} 履行`undefined`成功后。

更改文件的权限。

### `fsPromises.chown(path, uid, gid)`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `uid`{整数}
*   `gid`{整数}
*   返回：{承诺} 履行`undefined`成功后。

更改文件的所有权。

### `fsPromises.copyFile(src, dest[, mode])`

<!-- YAML
added: v10.0.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/27044
    description: Changed 'flags' argument to 'mode' and imposed
                 stricter type validation.
-->

*   `src`{字符串|缓冲区|要复制的 URL} 源文件名
*   `dest`{字符串|缓冲区|URL} 复制操作的目标文件名
*   `mode`{整数}指定副本行为的可选修饰符
    操作。可以创建一个由按位 OR 组成的掩码
    两个或多个值（例如
    `fs.constants.COPYFILE_EXCL | fs.constants.COPYFILE_FICLONE`)
    **违约：** `0`.
    *   `fs.constants.COPYFILE_EXCL`：如果出现以下情况，复制操作将失败：`dest`
        已存在。
    *   `fs.constants.COPYFILE_FICLONE`：复制操作将尝试创建
        写入时复制引用链接。如果平台不支持写入时复制，
        然后使用回退复制机制。
    *   `fs.constants.COPYFILE_FICLONE_FORCE`：复制操作将尝试
        创建写入时复制引用链接。如果平台不支持
        写入时复制，则操作将失败。
*   返回：{承诺} 履行`undefined`成功后。

异步复制`src`自`dest`.默认情况下，`dest`如果它被覆盖
已存在。

不保证复制操作的原子性。如果
打开目标文件进行写入后发生错误，尝试
将被删除目的地。

```mjs
import { copyFile, constants } from 'node:fs/promises';

try {
  await copyFile('source.txt', 'destination.txt');
  console.log('source.txt was copied to destination.txt');
} catch {
  console.log('The file could not be copied');
}

// By using COPYFILE_EXCL, the operation will fail if destination.txt exists.
try {
  await copyFile('source.txt', 'destination.txt', constants.COPYFILE_EXCL);
  console.log('source.txt was copied to destination.txt');
} catch {
  console.log('The file could not be copied');
}
```

### `fsPromises.cp(src, dest[, options])`

<!-- YAML
added: v16.7.0
changes:
  - version:
    - v17.6.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41819
    description: Accepts an additional `verbatimSymlinks` option to specify
                 whether to perform path resolution for symlinks.
-->

> 稳定性： 1 - 实验

*   `src`{字符串|URL} 要复制的源路径。
*   `dest`{字符串|要复制到的 URL} 目标路径。
*   `options`{对象}
    *   `dereference`{boolean} dereference symlinks.**违约：** `false`.
    *   `errorOnExist`{布尔值} 当`force`是`false`和目标
        存在，引发错误。**违约：** `false`.
    *   `filter`{函数}过滤复制的文件/目录的函数。返回
        `true`以复制项目，`false`以忽略它。还可以返回一个`Promise`
        解析为`true`或`false` **违约：** `undefined`.
    *   `force`{布尔值} 覆盖现有文件或目录。副本
        如果将此设置为 false 并且目标，则操作将忽略错误
        存在。使用`errorOnExist`选项以更改此行为。
        **违约：** `true`.
    *   `preserveTimestamps`{布尔值}什么时候`true`时间戳来自`src`将
        被保存下来。**违约：** `false`.
    *   `recursive`{布尔} 递归复制目录**违约：** `false`
    *   `verbatimSymlinks`{布尔值}什么时候`true`，符号链接的路径分辨率将
        被跳过。**违约：** `false`
*   返回：{承诺} 履行`undefined`成功后。

异步复制整个目录结构`src`自`dest`,
包括子目录和文件。

将一个目录复制到另一个目录时，不支持 globs 和
行为类似于`cp dir1/ dir2/`.

### `fsPromises.lchmod(path, mode)`

<!-- YAML
deprecated: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{整数}
*   返回：{承诺} 履行`undefined`成功后。

更改符号链接的权限。

此方法仅在 macOS 上实现。

### `fsPromises.lchown(path, uid, gid)`

<!-- YAML
added: v10.0.0
changes:
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/21498
    description: This API is no longer deprecated.
-->

*   `path`{字符串|缓冲区|网址}
*   `uid`{整数}
*   `gid`{整数}
*   返回：{承诺} 履行`undefined`成功后。

更改符号链接的所有权。

### `fsPromises.lutimes(path, atime, mtime)`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

*   `path`{字符串|缓冲区|网址}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}
*   返回：{承诺} 履行`undefined`成功后。

更改文件的访问和修改时间的方式与
[`fsPromises.utimes()`][fsPromises.utimes()]，区别在于如果路径引用
符号链接，则不会取消引用该链接：相反，时间戳为
符号链接本身已更改。

### `fsPromises.link(existingPath, newPath)`

<!-- YAML
added: v10.0.0
-->

*   `existingPath`{字符串|缓冲区|网址}
*   `newPath`{字符串|缓冲区|网址}
*   返回：{承诺} 履行`undefined`成功后。

从 创建新链接`existingPath`到`newPath`.查看 POSIX
link（2） 文档提供更多详细信息。

### `fsPromises.lstat(path[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   返回：{Promise} 使用 {fs 实现。给定的 Stats} 对象
    符号链接`path`.

相当于[`fsPromises.stat()`][fsPromises.stat()]除非`path`指符号链接，
在这种情况下，链接本身是统计的，而不是它所引用的文件。
请参阅 POSIX lstat（2） 文档了解更多详情。

### `fsPromises.mkdir(path[, options])`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{Object|integer}
    *   `recursive`{布尔值}**违约：** `false`
    *   `mode`{string|integer}在 Windows 上不受支持。**违约：** `0o777`.
*   返回：{承诺} 成功后，实现`undefined`如果`recursive`
    是`false`，或在以下情况下创建的第一个目录路径：`recursive`是`true`.

异步创建目录。

可选`options`参数可以是指定`mode`（权限
和粘滞位），或具有`mode`属性和`recursive`
属性，指示是否应创建父目录。叫
`fsPromises.mkdir()`什么时候`path`是一个目录，该目录会导致
仅在以下情况下拒绝`recursive`是假的。

```mjs
import { mkdir } from 'node:fs/promises';

try {
  const projectFolder = new URL('./test/project/', import.meta.url);
  const createDir = await mkdir(path, { recursive: true });

  console.log(`created ${createDir}`);
} catch (err) {
  console.error(err.message);
}
```

```cjs
const { mkdir } = require('node:fs/promises');
const { resolve, join } = require('node:path');

async function makeDirectory() {
  const projectFolder = join(__dirname, 'test', 'project');
  const dirCreation = await mkdir(projectFolder, { recursive: true });

  console.log(dirCreation);
  return dirCreation;
}

makeDirectory().catch(console.error);
```

### `fsPromises.mkdtemp(prefix[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version:
      - v16.5.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/39028
    description: The `prefix` parameter now accepts an empty string.
-->

*   `prefix`{字符串}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{Promise} 使用包含文件系统路径的字符串实现
    新创建的临时目录。

创建唯一的临时目录。唯一的目录名称由以下公式生成：
将六个随机字符附加到提供的末尾`prefix`.由于
平台不一致，避免尾随`X`字符`prefix`.一些
平台，特别是BSD，可以返回六个以上的随机字符，并且
替换尾随`X`字符`prefix`随机字符。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码。

```mjs
import { mkdtemp } from 'node:fs/promises';

try {
  await mkdtemp(path.join(os.tmpdir(), 'foo-'));
} catch (err) {
  console.error(err);
}
```

这`fsPromises.mkdtemp()`方法将附加随机选择的六个
字符直接到`prefix`字符串。例如，给定一个目录
`/tmp`，如果目的是创建临时目录*在* `/tmp`这
`prefix`必须以尾随平台特定的路径分隔符结尾
(`require('node:path').sep`).

### `fsPromises.open(path, flags[, mode])`

<!-- YAML
added: v10.0.0
changes:
  - version: v11.1.0
    pr-url: https://github.com/nodejs/node/pull/23767
    description: The `flags` argument is now optional and defaults to `'r'`.
-->

*   `path`{字符串|缓冲区|网址}
*   `flags`{字符串|数字}看[文件系统支持`flags`][support of file system flags].
    **违约：** `'r'`.
*   `mode`{string|integer}设置文件模式（权限和粘滞位）
    如果文件已创建。**违约：** `0o666`（可读写）
*   返回：{Promise} 用 {FileHandle} 对象实现。

打开 {文件处理}。

有关更多详细信息，请参阅 POSIX open（2） 文档。

一些字符 （`< > : " / \ | ? *`） 保留在 Windows 下，如文档所示
由[命名文件、路径和命名空间][Naming Files, Paths, and Namespaces].在 NTFS 下，如果文件名包含
冒号 Node.js 将打开文件系统流，如
[此 MSDN 页面][MSDN-Using-Streams].

### `fsPromises.opendir(path[, options])`

<!-- YAML
added: v12.12.0
changes:
  - version:
     - v13.1.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30114
    description: The `bufferSize` option was introduced.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `bufferSize`{数字}缓冲的目录条目数
        从目录读取时在内部。值越高，越好
        性能，但内存使用率更高。**违约：** `32`
*   返回：{Promise} 用 {fs 实现。目录}.

异步打开目录以进行迭代扫描。查看 POSIX
opendir（3） 文档提供更多详细信息。

创建一个 {fs。Dir}，其中包含用于读取
并清理目录。

这`encoding`选项设置`path`打开
目录和后续读取操作。

使用异步迭代的示例：

```mjs
import { opendir } from 'node:fs/promises';

try {
  const dir = await opendir('./');
  for await (const dirent of dir)
    console.log(dirent.name);
} catch (err) {
  console.error(err);
}
```

使用异步迭代器时，{fs.Dir} 对象将自动
迭代器退出后关闭。

### `fsPromises.readdir(path[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version: v10.11.0
    pr-url: https://github.com/nodejs/node/pull/22020
    description: New option `withFileTypes` was added.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
    *   `withFileTypes`{布尔值}**违约：** `false`
*   返回：{Promise} 使用 中文件名称的数组实现
    目录不包括`'.'`和`'..'`.

读取目录的内容。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
文件名。如果`encoding`设置为`'buffer'`，则返回文件名
将作为 {Buffer} 对象传递。

如果`options.withFileTypes`设置为`true`，解析的数组将包含
{fs.Dirent} 对象。

```mjs
import { readdir } from 'node:fs/promises';

try {
  const files = await readdir(path);
  for (const file of files)
    console.log(file);
} catch (err) {
  console.error(err);
}
```

### `fsPromises.readFile(path[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version:
    - v15.2.0
    - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/35911
    description: The options argument may include an AbortSignal to abort an
                 ongoing readFile request.
-->

*   `path`{字符串|缓冲区|网址|FileHandle} 文件名或`FileHandle`
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `null`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'r'`.
    *   `signal`{中止信号} 允许中止正在进行的读取文件
*   返回：{Promise} 使用文件的内容实现。

异步读取文件的全部内容。

如果未指定编码（使用`options.encoding`），则返回数据
作为 {Buffer} 对象。否则，数据将是一个字符串。

如果`options`是一个字符串，然后它指定编码。

当`path`是一个目录，行为`fsPromises.readFile()`是
特定于平台。在macOS，Linux和Windows上，承诺将被拒绝。
出现错误。在 FreeBSD 上， 目录内容的表示形式将是
返回。

可以中止正在进行的`readFile`使用 {AbortSignal}。如果
请求中止 返回的承诺被拒绝，并带有`AbortError`:

```mjs
import { readFile } from 'node:fs/promises';

try {
  const controller = new AbortController();
  const { signal } = controller;
  const promise = readFile(fileName, { signal });

  // Abort the request before the promise settles.
  controller.abort();

  await promise;
} catch (err) {
  // When a request is aborted - err is an AbortError
  console.error(err);
}
```

中止正在进行的请求不会中止单个操作
系统请求，而是内部缓冲`fs.readFile`执行。

任何指定的 {FileHandle} 都必须支持读取。

### `fsPromises.readlink(path[, options])`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{承诺} 与`linkString`成功后。

读取所引用的符号链接的内容`path`.查看 POSIX
阅读链接（2） 文档以获取更多详细信息。承诺通过
`linkString`成功后。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
返回的链接路径。如果`encoding`设置为`'buffer'`，链接路径
返回的将作为 {Buffer} 对象传递。

### `fsPromises.realpath(path[, options])`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{承诺} 成功后使用解析的路径实现。

确定 的实际位置`path`使用与
`fs.realpath.native()`功能。

仅支持可转换为 UTF8 字符串的路径。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
路径。如果`encoding`设置为`'buffer'`，则返回的路径将为
作为 {Buffer} 对象传递。

在 Linux 上，当 Node.js 与 musl libc 链接时，procfs 文件系统必须
安装在`/proc`为了使此功能正常工作。格列布克没有
此限制。

### `fsPromises.rename(oldPath, newPath)`

<!-- YAML
added: v10.0.0
-->

*   `oldPath`{字符串|缓冲区|网址}
*   `newPath`{字符串|缓冲区|网址}
*   返回：{承诺} 履行`undefined`成功后。

重 命名`oldPath`自`newPath`.

### `fsPromises.rmdir(path[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37216
    description: "Using `fsPromises.rmdir(path, { recursive: true })` on a `path`
                 that is a file is no longer permitted and results in an
                 `ENOENT` error on Windows and an `ENOTDIR` error on POSIX."
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37216
    description: "Using `fsPromises.rmdir(path, { recursive: true })` on a `path`
                 that does not exist is no longer permitted and results in a
                 `ENOENT` error."
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37302
    description: The `recursive` option is deprecated, using it triggers a
                 deprecation warning.
  - version: v14.14.0
    pr-url: https://github.com/nodejs/node/pull/35579
    description: The `recursive` option is deprecated, use `fsPromises.rm` instead.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30644
    description: The `maxBusyTries` option is renamed to `maxRetries`, and its
                 default is 0. The `emfileWait` option has been removed, and
                 `EMFILE` errors use the same retry logic as other errors. The
                 `retryDelay` option is now supported. `ENFILE` errors are now
                 retried.
  - version: v12.10.0
    pr-url: https://github.com/nodejs/node/pull/29168
    description: The `recursive`, `maxBusyTries`, and `emfileWait` options are
                  now supported.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `maxRetries`{整数}如果`EBUSY`,`EMFILE`,`ENFILE`,`ENOTEMPTY`或
        `EPERM`遇到错误，节点.js以线性方式重试操作
        退避等待`retryDelay`每次尝试时都会延长几毫秒。此选项
        表示重试次数。如果`recursive`
        选项不是`true`.**违约：** `0`.
    *   `recursive`{布尔值}如果`true`，执行递归目录删除。在
        递归模式，操作在失败时重试。**违约：** `false`.
        **荒废的。**
    *   `retryDelay`{整数}等待的时间（以毫秒为单位）介于
        重试。如果`recursive`选项不是`true`.
        **违约：** `100`.
*   返回：{承诺} 履行`undefined`成功后。

删除 标识的目录`path`.

用`fsPromises.rmdir()`在文件（而不是目录）上，结果
承诺被拒绝`ENOENT`Windows 上的错误和`ENOTDIR`
POSIX 上的错误。

要获取类似于`rm -rf`Unix 命令，使用
[`fsPromises.rm()`][fsPromises.rm()]与选项`{ recursive: true, force: true }`.

### `fsPromises.rm(path[, options])`

<!-- YAML
added: v14.14.0
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `force`{布尔值}什么时候`true`，则在以下情况下将忽略异常`path`做
        不存在。**违约：** `false`.
    *   `maxRetries`{整数}如果`EBUSY`,`EMFILE`,`ENFILE`,`ENOTEMPTY`或
        `EPERM`遇到错误，节点.js将重试线性操作
        退避等待`retryDelay`每次尝试时都会延长几毫秒。此选项
        表示重试次数。如果`recursive`
        选项不是`true`.**违约：** `0`.
    *   `recursive`{布尔值}如果`true`，执行递归目录删除。在
        递归模式操作在失败时重试。**违约：** `false`.
    *   `retryDelay`{整数}等待的时间（以毫秒为单位）介于
        重试。如果`recursive`选项不是`true`.
        **违约：** `100`.
*   返回：{承诺} 履行`undefined`成功后。

删除文件和目录（以标准 POSIX 为模型）`rm`实用程序）。

### `fsPromises.stat(path[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   返回：{Promise} 使用 {fs 实现。Stats} 对象
    鉴于`path`.

### `fsPromises.symlink(target, path[, type])`

<!-- YAML
added: v10.0.0
-->

*   `target`{字符串|缓冲区|网址}
*   `path`{字符串|缓冲区|网址}
*   `type`{字符串}**违约：** `'file'`
*   返回：{承诺} 履行`undefined`成功后。

创建符号链接。

这`type`参数仅在 Windows 平台上使用，并且可以是`'dir'`,
`'file'`或`'junction'`.Windows 交接点需要目标路径
是绝对的。使用时`'junction'`这`target`参数将
自动归一化为绝对路径。

### `fsPromises.truncate(path[, len])`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `len`{整数}**违约：** `0`
*   返回：{承诺} 履行`undefined`成功后。

截断（缩短或延长长度）的内容`path`自`len`
字节。

### `fsPromises.unlink(path)`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   返回：{承诺} 履行`undefined`成功后。

如果`path`引用符号链接，则删除该链接而不影响
该链接所引用的文件或目录。如果`path`引用文件
不是符号链接的路径，文件将被删除。请参阅 POSIX 取消链接（2）
文档以获取更多详细信息。

### `fsPromises.utimes(path, atime, mtime)`

<!-- YAML
added: v10.0.0
-->

*   `path`{字符串|缓冲区|网址}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}
*   返回：{承诺} 履行`undefined`成功后。

更改 引用的对象的文件系统时间戳`path`.

这`atime`和`mtime`参数遵循以下规则：

*   值可以是表示 Unix 纪元时间的数字，`Date`s，或
    数字字符串，如`'123456789.0'`.
*   如果该值不能转换为数字，或者`NaN`,`Infinity`或
    `-Infinity`一`Error`将被抛出。

### `fsPromises.watch(filename[, options])`

<!-- YAML
added:
  - v15.9.0
  - v14.18.0
-->

*   `filename`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `persistent`{布尔值}指示进程是否应继续运行
        只要文件被监视。**违约：** `true`.
    *   `recursive`{布尔值}指示是否所有子目录都应
        已监视，或仅监视当前目录。当目录
        指定且仅在支持的平台上（请参阅[警告][caveats]).**违约：**
        `false`.
    *   `encoding`{字符串}指定要用于
        文件名传递给侦听器。**违约：** `'utf8'`.
    *   `signal`{中止信号}{中止信号} 用于在观察程序发出信号时发出信号
        应该停止。
*   返回：{AsyncIterator} 的对象，其属性为：
    *   `eventType`{字符串}变更类型
    *   `filename`{字符串|缓冲区} 文件的名称已更改。

返回一个异步迭代器，该迭代器监视 上的更改`filename`哪里`filename`
是文件或目录。

```js
const { watch } = require('node:fs/promises');

const ac = new AbortController();
const { signal } = ac;
setTimeout(() => ac.abort(), 10000);

(async () => {
  try {
    const watcher = watch(__filename, { signal });
    for await (const event of watcher)
      console.log(event);
  } catch (err) {
    if (err.name === 'AbortError')
      return;
    throw err;
  }
})();
```

在大多数平台上，`'rename'`每当出现文件名或
在目录中消失。

所有[警告][caveats]为`fs.watch()`也适用于`fsPromises.watch()`.

### `fsPromises.writeFile(file, data[, options])`

<!-- YAML
added: v10.0.0
changes:
  - version:
      - v15.14.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37490
    description: The `data` argument supports `AsyncIterable`, `Iterable`, and `Stream`.
  - version:
      - v15.2.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/35993
    description: The options argument may include an AbortSignal to abort an
                 ongoing writeFile request.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `data` parameter won't coerce unsupported input to
                 strings anymore.
-->

*   `file`{字符串|缓冲区|网址|FileHandle} 文件名或`FileHandle`
*   `data`{字符串|缓冲区|TypedArray|数据视图|AsyncIterable|可迭代|流}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `mode`{整数}**违约：** `0o666`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'w'`.
    *   `signal`{中止信号} 允许中止正在进行的写入文件
*   返回：{承诺} 履行`undefined`成功后。

异步将数据写入文件，如果文件已存在，则替换该文件。
`data`可以是字符串、缓冲区、{AsyncIterable} 或 {Iterable} 对象。

这`encoding`选项在以下情况下被忽略：`data`是缓冲区。

如果`options`是一个字符串，然后它指定编码。

这`mode`选项仅影响新创建的文件。看[`fs.open()`][fs.open()]
了解更多详情。

任何指定的 {FileHandle} 都必须支持写入。

使用不安全`fsPromises.writeFile()`在同一文件上多次
无需等待承诺得到解决。

类似于`fsPromises.readFile`-`fsPromises.writeFile`是一种方便
执行多个方法的方法`write`在内部调用以写入缓冲区
传递给它。对于性能敏感的代码，请考虑使用
[`fs.createWriteStream()`][fs.createWriteStream()]或[`filehandle.createWriteStream()`][filehandle.createWriteStream()].

可以使用 {AbortSignal} 来取消`fsPromises.writeFile()`.
取消是“尽力而为”，并且可能仍然会有一定数量的数据
待写。

```mjs
import { writeFile } from 'node:fs/promises';
import { Buffer } from 'node:buffer';

try {
  const controller = new AbortController();
  const { signal } = controller;
  const data = new Uint8Array(Buffer.from('Hello Node.js'));
  const promise = writeFile('message.txt', data, { signal });

  // Abort the request before the promise settles.
  controller.abort();

  await promise;
} catch (err) {
  // When a request is aborted - err is an AbortError
  console.error(err);
}
```

中止正在进行的请求不会中止单个操作
系统请求，而是内部缓冲`fs.writeFile`执行。

### `fsPromises.constants`

*   {对象}

返回一个对象，其中包含文件系统的常用常量
操作。对象与`fs.constants`.看[FS 常量][FS constants]
了解更多详情。

## 回调接口

回调 API 异步执行所有操作，而不会阻塞
事件循环，然后在完成或出错时调用回调函数。

回调 API 使用底层 Node.js 线程池来执行文件
系统操作脱离事件循环线程。这些操作不是
已同步或线程安全。执行多个时必须小心
可能会对同一文件进行并发修改或发生数据损坏。

### `fs.access(path[, mode], callback)`

<!-- YAML
added: v0.11.15
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6534
    description: The constants like `fs.R_OK`, etc which were present directly
                 on `fs` were moved into `fs.constants` as a soft deprecation.
                 Thus for Node.js `< v6.3.0` use `fs`
                 to access those constants, or
                 do something like `(fs.constants || fs).R_OK` to work with all
                 versions.
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{整数}**违约：** `fs.constants.F_OK`
*   `callback`{函数}
    *   `err`{错误}

测试用户对`path`.
这`mode`参数是一个可选整数，用于指定可访问性
要执行的检查。`mode`应为值`fs.constants.F_OK`
或由以下任一项的按位 OR 组成的掩码`fs.constants.R_OK`,
`fs.constants.W_OK`和`fs.constants.X_OK`（例如
`fs.constants.W_OK | fs.constants.R_OK`).检查[文件访问常量][File access constants]为
可能的值`mode`.

最后的论点，`callback`是调用的回调函数
一个可能的错误参数。如果任何辅助功能检查失败，则错误
参数将是一个`Error`对象。以下示例检查是否
`package.json`存在，并且如果它是可读的或可写的。

```mjs
import { access, constants } from 'node:fs';

const file = 'package.json';

// Check if the file exists in the current directory.
access(file, constants.F_OK, (err) => {
  console.log(`${file} ${err ? 'does not exist' : 'exists'}`);
});

// Check if the file is readable.
access(file, constants.R_OK, (err) => {
  console.log(`${file} ${err ? 'is not readable' : 'is readable'}`);
});

// Check if the file is writable.
access(file, constants.W_OK, (err) => {
  console.log(`${file} ${err ? 'is not writable' : 'is writable'}`);
});

// Check if the file is readable and writable.
access(file, constants.R_OK | constants.W_OK, (err) => {
  console.log(`${file} ${err ? 'is not' : 'is'} readable and writable`);
});
```

请勿使用`fs.access()`在调用之前检查文件的可访问性
`fs.open()`,`fs.readFile()`或`fs.writeFile()`.行为
因此引入了争用条件，因为其他进程可能会更改文件的
两个调用之间的状态。相反，用户代码应打开/读取/写入
，并在文件不可访问时处理引发的错误。

**写入（不推荐）**

```mjs
import { access, open, close } from 'node:fs';

access('myfile', (err) => {
  if (!err) {
    console.error('myfile already exists');
    return;
  }

  open('myfile', 'wx', (err, fd) => {
    if (err) throw err;

    try {
      writeMyData(fd);
    } finally {
      close(fd, (err) => {
        if (err) throw err;
      });
    }
  });
});
```

**写入（推荐）**

```mjs
import { open, close } from 'node:fs';

open('myfile', 'wx', (err, fd) => {
  if (err) {
    if (err.code === 'EEXIST') {
      console.error('myfile already exists');
      return;
    }

    throw err;
  }

  try {
    writeMyData(fd);
  } finally {
    close(fd, (err) => {
      if (err) throw err;
    });
  }
});
```

**阅读（不推荐）**

```mjs
import { access, open, close } from 'node:fs';
access('myfile', (err) => {
  if (err) {
    if (err.code === 'ENOENT') {
      console.error('myfile does not exist');
      return;
    }

    throw err;
  }

  open('myfile', 'r', (err, fd) => {
    if (err) throw err;

    try {
      readMyData(fd);
    } finally {
      close(fd, (err) => {
        if (err) throw err;
      });
    }
  });
});
```

**阅读（推荐）**

```mjs
import { open, close } from 'node:fs';

open('myfile', 'r', (err, fd) => {
  if (err) {
    if (err.code === 'ENOENT') {
      console.error('myfile does not exist');
      return;
    }

    throw err;
  }

  try {
    readMyData(fd);
  } finally {
    close(fd, (err) => {
      if (err) throw err;
    });
  }
});
```

上面的“不推荐”示例检查可访问性，然后使用
文件;“推荐”示例更好，因为它们直接使用文件
并处理错误（如果有）。

通常，仅当文件不是
直接使用，例如，当其可访问性是来自另一个的信号时
过程。

在 Windows 上，目录上的访问控制策略 （ACL） 可能会将访问限制为
文件或目录。这`fs.access()`但是，函数不检查
ACL，因此可以报告路径可访问，即使 ACL 限制
用户从读取或写入它。

### `fs.appendFile(path, data[, options], callback)`

<!-- YAML
added: v0.6.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7831
    description: The passed `options` object will never be modified.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3163
    description: The `file` parameter can be a file descriptor now.
-->

*   `path`{字符串|缓冲区|URL|编号} 文件名或文件描述符
*   `data`{字符串|缓冲区}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `mode`{整数}**违约：** `0o666`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'a'`.
*   `callback`{函数}
    *   `err`{错误}

将数据异步追加到文件，如果尚未将数据追加到文件，则创建该文件
存在。`data`可以是字符串或 {缓冲区}。

这`mode`选项仅影响新创建的文件。看[`fs.open()`][fs.open()]
了解更多详情。

```mjs
import { appendFile } from 'node:fs';

appendFile('message.txt', 'data to append', (err) => {
  if (err) throw err;
  console.log('The "data to append" was appended to file!');
});
```

如果`options`是一个字符串，然后它指定编码：

```mjs
import { appendFile } from 'node:fs';

appendFile('message.txt', 'data to append', 'utf8', callback);
```

这`path`可以指定为已打开的数字文件描述符
用于追加（使用`fs.open()`或`fs.openSync()`).文件描述符将
不会自动关闭。

```mjs
import { open, close, appendFile } from 'node:fs';

function closeFd(fd) {
  close(fd, (err) => {
    if (err) throw err;
  });
}

open('message.txt', 'a', (err, fd) => {
  if (err) throw err;

  try {
    appendFile(fd, 'data to append', 'utf8', (err) => {
      closeFd(fd);
      if (err) throw err;
    });
  } catch (err) {
    closeFd(fd);
    throw err;
  }
});
```

### `fs.chmod(path, mode, callback)`

<!-- YAML
added: v0.1.30
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{string|integer}
*   `callback`{函数}
    *   `err`{错误}

异步更改文件的权限。除
完成回调可能出现异常。

参见 POSIX chmod（2） 文档了解更多详情。

```mjs
import { chmod } from 'node:fs';

chmod('my_file.txt', 0o775, (err) => {
  if (err) throw err;
  console.log('The permissions for file "my_file.txt" have been changed!');
});
```

#### 文件模式

这`mode`参数用于`fs.chmod()`和`fs.chmodSync()`
方法是使用以下逻辑 OR 创建的数字位掩码
常数：

|恒定|八进制|描述 |
|---------------------- |------- |------------------------ |
|`fs.constants.S_IRUSR`|`0o400`|由所有者|阅读
|`fs.constants.S_IWUSR`|`0o200`|由所有者|撰写
|`fs.constants.S_IXUSR`|`0o100`|按所有者|执行/搜索
|`fs.constants.S_IRGRP`|`0o40`|按组|读取
|`fs.constants.S_IWGRP`|`0o20`|按组|编写
|`fs.constants.S_IXGRP`|`0o10`|按组|执行/搜索
|`fs.constants.S_IROTH`|`0o4`|由其他人|阅读
|`fs.constants.S_IWOTH`|`0o2`|由其他人|撰写
|`fs.constants.S_IXOTH`|`0o1`|执行/搜索其他人|

一种更简单的构造方法`mode`是使用三个序列
八进制数字（例如`765`).最左边的数字 （`7`在示例中），指定
文件所有者的权限。中间数字 （`6`在示例中），
指定组的权限。最右边的数字 （`5`在示例中），
指定其他人的权限。

|数字 |描述 |
|------ |------------------------ |
|`7`|读取、写入和执行|
|`6`|读写|
|`5`|读取和执行|
|`4`|只读|
|`3`|编写和执行|
|`2`|只写|
|`1`|仅执行|
|`0`|无权限|

例如，八进制值`0o765`方法：

*   所有者可以读取、写入和执行文件。
*   该组可以读取和写入该文件。
*   其他人可以读取并执行该文件。

在预期文件模式中使用原始数字时，任何大于
`0o777`可能会导致不支持工作的特定于平台的行为
一贯。因此，常量如`S_ISVTX`,`S_ISGID`或`S_ISUID`是
未暴露在`fs.constants`.

注意事项：在Windows上，只能更改写入权限，并且
组、所有者或其他人的权限之间的区别不
实现。

### `fs.chown(path, uid, gid, callback)`

<!-- YAML
added: v0.1.97
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `uid`{整数}
*   `gid`{整数}
*   `callback`{函数}
    *   `err`{错误}

异步更改文件的所有者和组。除
完成回调可能出现异常。

参见 POSIX chown（2） 文档了解更多详情。

### `fs.close(fd[, callback])`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version:
      - v15.9.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37174
    description: A default callback is now used if one is not provided.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `callback`{函数}
    *   `err`{错误}

关闭文件描述符。除了可能的异常之外，没有其他参数
交给完成回调。

叫`fs.close()`在任何文件描述符 （`fd`） 当前正在使用
通过任何其他`fs`操作可能会导致未定义的行为。

请参阅 POSIX close（2） 文档了解更多详情。

### `fs.copyFile(src, dest[, mode], callback)`

<!-- YAML
added: v8.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/27044
    description: Changed 'flags' argument to 'mode' and imposed
                 stricter type validation.
-->

*   `src`{字符串|缓冲区|要复制的 URL} 源文件名
*   `dest`{字符串|缓冲区|URL} 复制操作的目标文件名
*   `mode`{整数} 用于复制操作的修饰符。**违约：** `0`.
*   `callback`{函数}

异步复制`src`自`dest`.默认情况下，`dest`如果它被覆盖
已存在。除了可能的异常之外，没有其他参数提供给
回调函数。Node.js不保证副本的原子性
操作。如果在 为 的目标文件打开后发生错误
写入，节点.js将尝试删除目标。

`mode`是指定行为的可选整数
的复制操作。可以创建由按位
两个或多个值的 OR（例如
`fs.constants.COPYFILE_EXCL | fs.constants.COPYFILE_FICLONE`).

*   `fs.constants.COPYFILE_EXCL`：如果出现以下情况，复制操作将失败：`dest`已经
    存在。
*   `fs.constants.COPYFILE_FICLONE`：复制操作将尝试创建一个
    写入时复制引用链接。如果平台不支持写入时复制，则
    使用回退复制机制。
*   `fs.constants.COPYFILE_FICLONE_FORCE`：复制操作将尝试
    创建写入时复制引用链接。如果平台不支持
    写入时复制，则操作将失败。

```mjs
import { copyFile, constants } from 'node:fs';

function callback(err) {
  if (err) throw err;
  console.log('source.txt was copied to destination.txt');
}

// destination.txt will be created or overwritten by default.
copyFile('source.txt', 'destination.txt', callback);

// By using COPYFILE_EXCL, the operation will fail if destination.txt exists.
copyFile('source.txt', 'destination.txt', constants.COPYFILE_EXCL, callback);
```

### `fs.cp(src, dest[, options], callback)`

<!-- YAML
added: v16.7.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version:
    - v17.6.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41819
    description: Accepts an additional `verbatimSymlinks` option to specify
                 whether to perform path resolution for symlinks.
-->

> 稳定性： 1 - 实验

*   `src`{字符串|URL} 要复制的源路径。
*   `dest`{字符串|要复制到的 URL} 目标路径。
*   `options`{对象}
    *   `dereference`{boolean} dereference symlinks.**违约：** `false`.
    *   `errorOnExist`{布尔值} 当`force`是`false`和目标
        存在，引发错误。**违约：** `false`.
    *   `filter`{函数}过滤复制的文件/目录的函数。返回
        `true`以复制项目，`false`以忽略它。还可以返回一个`Promise`
        解析为`true`或`false` **违约：** `undefined`.
    *   `force`{布尔值} 覆盖现有文件或目录。副本
        如果将此设置为 false 并且目标，则操作将忽略错误
        存在。使用`errorOnExist`选项以更改此行为。
        **违约：** `true`.
    *   `preserveTimestamps`{布尔值}什么时候`true`时间戳来自`src`将
        被保存下来。**违约：** `false`.
    *   `recursive`{布尔} 递归复制目录**违约：** `false`
    *   `verbatimSymlinks`{布尔值}什么时候`true`，符号链接的路径分辨率将
        被跳过。**违约：** `false`
*   `callback`{函数}

异步复制整个目录结构`src`自`dest`,
包括子目录和文件。

将一个目录复制到另一个目录时，不支持 globs 和
行为类似于`cp dir1/ dir2/`.

### `fs.createReadStream(path[, options])`

<!-- YAML
added: v0.1.31
changes:
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/40013
    description: The `fs` option does not need `open` method if an `fd` was provided.
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/40013
    description: The `fs` option does not need `close` method if `autoClose` is `false`.
  - version:
     - v15.4.0
    pr-url: https://github.com/nodejs/node/pull/35922
    description: The `fd` option accepts FileHandle arguments.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31408
    description: Change `emitClose` default to `true`.
  - version:
     - v13.6.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/29083
    description: The `fs` options allow overriding the used `fs`
                 implementation.
  - version: v12.10.0
    pr-url: https://github.com/nodejs/node/pull/29212
    description: Enable `emitClose` option.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/19898
    description: Impose new restrictions on `start` and `end`, throwing
                 more appropriate errors in cases when we cannot reasonably
                 handle the input values.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7831
    description: The passed `options` object will never be modified.
  - version: v2.3.0
    pr-url: https://github.com/nodejs/node/pull/1845
    description: The passed `options` object can be a string now.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `flags`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：**
        `'r'`.
    *   `encoding`{字符串}**违约：** `null`
    *   `fd`{整数|文件处理}**违约：** `null`
    *   `mode`{整数}**违约：** `0o666`
    *   `autoClose`{布尔值}**违约：** `true`
    *   `emitClose`{布尔值}**违约：** `true`
    *   `start`{整数}
    *   `end`{整数}**违约：** `Infinity`
    *   `highWaterMark`{整数}**违约：** `64 * 1024`
    *   `fs`{对象|空洞}**违约：** `null`
*   返回值：{fs。读取流}

与 16 KiB 默认值不同`highWaterMark`对于 {流。可读}，流
此方法返回的默认值`highWaterMark`的 64 KiB。

`options`可以包括`start`和`end`要从中读取一定范围的字节的值
该文件而不是整个文件。双`start`和`end`具有包容性，并且
从 0 开始计数，允许的值位于
\[0,[`Number.MAX_SAFE_INTEGER`][Number.MAX_SAFE_INTEGER]]范围。如果`fd`已指定并`start`是
省略或`undefined`,`fs.createReadStream()`按顺序从
当前文件位置。这`encoding`可以是接受的任何一个
{缓冲区}.

如果`fd`指定，`ReadStream`将忽略`path`参数和将使用
指定的文件描述符。这意味着没有`'open'`事件将是
排放。`fd`应该是阻塞;非阻塞`fd`s 应传递给
{网.套接字}。

如果`fd`指向仅支持阻止读取的字符设备
（如键盘或声卡），读取操作直到数据
可用。这可以防止进程退出，并且流从
自然关闭。

默认情况将发出`'close'`事件之后
摧毁。 设置`emitClose`选项`false`以更改此行为。

通过提供`fs`选项，可以覆盖相应的`fs`
的实现`open`,`read`和`close`.提供`fs`选择
的覆盖`read`是必需的。如果不是`fd`提供，覆盖
`open`也是必需的。如果`autoClose`是`true`，覆盖`close`是
也是必需的。

```mjs
import { createReadStream } from 'node:fs';

// Create a stream from some character device.
const stream = createReadStream('/dev/input/event0');
setTimeout(() => {
  stream.close(); // This may not close the stream.
  // Artificially marking end-of-stream, as if the underlying resource had
  // indicated end-of-file by itself, allows the stream to close.
  // This does not cancel pending read operations, and if there is such an
  // operation, the process may still not be able to exit successfully
  // until it finishes.
  stream.push(null);
  stream.read(0);
}, 100);
```

如果`autoClose`为 false，则文件描述符不会关闭，即使
出现错误。应用程序有责任将其关闭并
确保没有文件描述符泄漏。如果`autoClose`设置为 true（默认值
行为），开`'error'`或`'end'`文件描述符将关闭
自然而然。

`mode`设置文件模式（权限和粘滞位），但前提是
文件已创建。

读取长度为 100 个字节的文件的最后 10 个字节的示例：

```mjs
import { createReadStream } from 'node:fs';

createReadStream('sample.txt', { start: 90, end: 99 });
```

如果`options`是一个字符串，然后它指定编码。

### `fs.createWriteStream(path[, options])`

<!-- YAML
added: v0.1.31
changes:
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/40013
    description: The `fs` option does not need `open` method if an `fd` was provided.
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/40013
    description: The `fs` option does not need `close` method if `autoClose` is `false`.
  - version:
     - v15.4.0
    pr-url: https://github.com/nodejs/node/pull/35922
    description: The `fd` option accepts FileHandle arguments.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31408
    description: Change `emitClose` default to `true`.
  - version:
     - v13.6.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/29083
    description: The `fs` options allow overriding the used `fs`
                 implementation.
  - version: v12.10.0
    pr-url: https://github.com/nodejs/node/pull/29212
    description: Enable `emitClose` option.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7831
    description: The passed `options` object will never be modified.
  - version: v5.5.0
    pr-url: https://github.com/nodejs/node/pull/3679
    description: The `autoClose` option is supported now.
  - version: v2.3.0
    pr-url: https://github.com/nodejs/node/pull/1845
    description: The passed `options` object can be a string now.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `flags`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：**
        `'w'`.
    *   `encoding`{字符串}**违约：** `'utf8'`
    *   `fd`{整数|文件处理}**违约：** `null`
    *   `mode`{整数}**违约：** `0o666`
    *   `autoClose`{布尔值}**违约：** `true`
    *   `emitClose`{布尔值}**违约：** `true`
    *   `start`{整数}
    *   `fs`{对象|空洞}**违约：** `null`
*   返回值：{fs。写入流}

`options`还可以包括`start`选项以允许在某个位置写入数据
位置超过文件开头，允许的值位于
\[0,[`Number.MAX_SAFE_INTEGER`][Number.MAX_SAFE_INTEGER]]范围。修改文件而不是
替换它可能需要`flags`要设置为的选项`r+`而不是
违约`w`.这`encoding`可以是 {Buffer} 接受的任何一个。

如果`autoClose`设置为 true（默认行为）打开`'error'`或`'finish'`
文件描述符将自动关闭。如果`autoClose`是假的，
则文件描述符不会关闭，即使存在错误也是如此。
应用程序有责任关闭它并确保没有
文件描述符泄漏。

默认情况将发出`'close'`事件之后
摧毁。 设置`emitClose`选项`false`以更改此行为。

通过提供`fs`选项可以覆盖相应的`fs`
的实现`open`,`write`,`writev`和`close`.重写`write()`
没有`writev()`可能会降低性能，因为某些优化（`_writev()`)
将被禁用。提供`fs`选项，覆盖至少一个
`write`和`writev`是必需的。如果不是`fd`提供选项，覆盖
为`open`也是必需的。如果`autoClose`是`true`，覆盖`close`
也是必需的。

比如 {fs.ReadStream}，如果`fd`指定为 {fs。WriteStream} 将忽略
`path`参数，并将使用指定的文件描述符。这意味着没有
`'open'`将发出事件。`fd`应该是阻塞;非阻塞`fd`s
应传递给 {net。套接字}。

如果`options`是一个字符串，然后它指定编码。

### `fs.exists(path, callback)`

<!-- YAML
added: v0.0.2
deprecated: v1.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
-->

> 稳定性：0 - 已弃用：使用[`fs.stat()`][fs.stat()]或[`fs.access()`][fs.access()]相反。

*   `path`{字符串|缓冲区|网址}
*   `callback`{函数}
    *   `exists`{布尔值}

通过检查文件系统来测试给定路径是否存在。
然后调用`callback`参数为真或假：

```mjs
import { exists } from 'node:fs';

exists('/etc/passwd', (e) => {
  console.log(e ? 'it exists' : 'no passwd!');
});
```

**此回调的参数与其他 Node 不一致.js
回调。**通常，Node.js回调的第一个参数是`err`
参数，后跟其他参数（可选）。这`fs.exists()`回调
只有一个布尔参数。这是一个原因`fs.access()`建议使用
而不是`fs.exists()`.

用`fs.exists()`在调用之前检查文件是否存在
`fs.open()`,`fs.readFile()`或`fs.writeFile()`不建议这样做。行为
因此引入了争用条件，因为其他进程可能会更改文件的
两个调用之间的状态。相反，用户代码应打开/读取/写入
，并在文件不存在时处理引发的错误。

**写入（不推荐）**

```mjs
import { exists, open, close } from 'node:fs';

exists('myfile', (e) => {
  if (e) {
    console.error('myfile already exists');
  } else {
    open('myfile', 'wx', (err, fd) => {
      if (err) throw err;

      try {
        writeMyData(fd);
      } finally {
        close(fd, (err) => {
          if (err) throw err;
        });
      }
    });
  }
});
```

**写入（推荐）**

```mjs
import { open, close } from 'node:fs';
open('myfile', 'wx', (err, fd) => {
  if (err) {
    if (err.code === 'EEXIST') {
      console.error('myfile already exists');
      return;
    }

    throw err;
  }

  try {
    writeMyData(fd);
  } finally {
    close(fd, (err) => {
      if (err) throw err;
    });
  }
});
```

**阅读（不推荐）**

```mjs
import { open, close, exists } from 'node:fs';

exists('myfile', (e) => {
  if (e) {
    open('myfile', 'r', (err, fd) => {
      if (err) throw err;

      try {
        readMyData(fd);
      } finally {
        close(fd, (err) => {
          if (err) throw err;
        });
      }
    });
  } else {
    console.error('myfile does not exist');
  }
});
```

**阅读（推荐）**

```mjs
import { open, close } from 'node:fs';

open('myfile', 'r', (err, fd) => {
  if (err) {
    if (err.code === 'ENOENT') {
      console.error('myfile does not exist');
      return;
    }

    throw err;
  }

  try {
    readMyData(fd);
  } finally {
    close(fd, (err) => {
      if (err) throw err;
    });
  }
});
```

上面的“不推荐”示例检查是否存在，然后使用
文件;“推荐”示例更好，因为它们直接使用文件
并处理错误（如果有）。

通常，仅当文件不是
直接使用，例如，当它的存在是来自另一个人的信号时
过程。

### `fs.fchmod(fd, mode, callback)`

<!-- YAML
added: v0.4.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `mode`{string|integer}
*   `callback`{函数}
    *   `err`{错误}

设置对文件的权限。除了可能的异常之外，没有其他参数
被赋予完成回调。

参见 POSIX fchmod（2） 文档了解更多详情。

### `fs.fchown(fd, uid, gid, callback)`

<!-- YAML
added: v0.4.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `uid`{整数}
*   `gid`{整数}
*   `callback`{函数}
    *   `err`{错误}

设置文件的所有者。除了可能的异常之外，没有其他参数
交给完成回调。

请参阅 POSIX fchown（2） 文档了解更多详情。

### `fs.fdatasync(fd, callback)`

<!-- YAML
added: v0.1.96
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `callback`{函数}
    *   `err`{错误}

将与文件关联的所有当前排队的 I/O 操作强制到
操作系统的同步 I/O 完成状态。请参阅 POSIX
fdatasync（2） 文档了解详细信息。除了可能的参数外，没有其他参数
异常被赋予完成回调。

### `fs.fstat(fd[, options], callback)`

<!-- YAML
added: v0.1.95
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   `callback`{函数}
    *   `err`{错误}
    *   `stats`{fs.统计}

使用 {fs 调用回调。Stats} 用于文件描述符。

请参阅 POSIX fstat（2） 文档了解更多详情。

### `fs.fsync(fd, callback)`

<!-- YAML
added: v0.1.96
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `callback`{函数}
    *   `err`{错误}

请求将打开的文件描述符的所有数据刷新到存储中
装置。具体实现是特定于操作系统和设备的。
有关更多详细信息，请参阅 POSIX fsync（2） 文档。无参数 其他
比一个可能的异常被赋予完成回调。

### `fs.ftruncate(fd[, len], callback)`

<!-- YAML
added: v0.8.6
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `len`{整数}**违约：** `0`
*   `callback`{函数}
    *   `err`{错误}

截断文件描述符。除了可能的异常之外，没有其他参数
交给完成回调。

请参阅 POSIX ftruncate（2） 文档了解更多详情。

如果文件描述符引用的文件大于`len`字节，仅
第一个`len`字节将保留在文件中。

例如，以下程序仅保留
文件：

```mjs
import { open, close, ftruncate } from 'node:fs';

function closeFd(fd) {
  close(fd, (err) => {
    if (err) throw err;
  });
}

open('temp.txt', 'r+', (err, fd) => {
  if (err) throw err;

  try {
    ftruncate(fd, 4, (err) => {
      closeFd(fd);
      if (err) throw err;
    });
  } catch (err) {
    closeFd(fd);
    if (err) throw err;
  }
});
```

如果文件以前短于`len`字节，它被扩展，并且
扩展部分填充空字节 （`'\0'`):

如果`len`则为负数`0`将使用。

### `fs.futimes(fd, atime, mtime, callback)`

<!-- YAML
added: v0.4.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v4.1.0
    pr-url: https://github.com/nodejs/node/pull/2387
    description: Numeric strings, `NaN`, and `Infinity` are now allowed
                 time specifiers.
-->

*   `fd`{整数}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}
*   `callback`{函数}
    *   `err`{错误}

更改所提供文件所引用的对象的文件系统时间戳
描述符。看[`fs.utimes()`][fs.utimes()].

### `fs.lchmod(path, mode, callback)`

<!-- YAML
deprecated: v0.4.7
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37460
    description: The error returned may be an `AggregateError` if more than one
                 error is returned.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{整数}
*   `callback`{函数}
    *   `err`{错误|AggregateError}

更改符号链接的权限。除了可能的参数外，没有其他参数
异常被赋予完成回调。

此方法仅在 macOS 上实现。

参见 POSIX lchmod（2） 文档了解更多详情。

### `fs.lchown(path, uid, gid, callback)`

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/21498
    description: This API is no longer deprecated.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v0.4.7
    description: Documentation-only deprecation.
-->

*   `path`{字符串|缓冲区|网址}
*   `uid`{整数}
*   `gid`{整数}
*   `callback`{函数}
    *   `err`{错误}

设置符号链接的所有者。除了可能的参数外，没有其他参数
异常被赋予完成回调。

参见 POSIX lchown（2） 文档了解更多详情。

### `fs.lutimes(path, atime, mtime, callback)`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `path`{字符串|缓冲区|网址}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}
*   `callback`{函数}
    *   `err`{错误}

更改文件的访问和修改时间的方式与
[`fs.utimes()`][fs.utimes()]，区别在于如果路径引用符号
链接，则不会取消引用链接：而是
符号链接本身已更改。

除了可能的异常之外，没有为完成提供任何其他参数
回调。

### `fs.link(existingPath, newPath, callback)`

<!-- YAML
added: v0.1.31
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `existingPath` and `newPath` parameters can be WHATWG
                 `URL` objects using `file:` protocol. Support is currently
                 still *experimental*.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `existingPath`{字符串|缓冲区|网址}
*   `newPath`{字符串|缓冲区|网址}
*   `callback`{函数}
    *   `err`{错误}

从 创建新链接`existingPath`到`newPath`.查看 POSIX
link（2） 文档提供更多详细信息。除了可能的参数外，没有其他参数
异常被赋予完成回调。

### `fs.lstat(path[, options], callback)`

<!-- YAML
added: v0.1.30
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   `callback`{函数}
    *   `err`{错误}
    *   `stats`{fs.统计}

检索 {fs.Stats} 表示路径所引用的符号链接。
回调获取两个参数`(err, stats)`哪里`stats`是一个 {fs。统计}
对象。`lstat()`与`stat()`，但以下情况除外：`path`是一个象征性的
链接，则链接本身是统计的，而不是它引用的文件。

参见 POSIX lstat（2） 文档了解更多详情。

### `fs.mkdir(path[, options], callback)`

<!-- YAML
added: v0.1.8
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version:
     - v13.11.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31530
    description: In `recursive` mode, the callback now receives the first
                 created path as an argument.
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/21875
    description: The second argument can now be an `options` object with
                 `recursive` and `mode` properties.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{Object|integer}
    *   `recursive`{布尔值}**违约：** `false`
    *   `mode`{string|integer}在 Windows 上不受支持。**违约：** `0o777`.
*   `callback`{函数}
    *   `err`{错误}
    *   `path`{字符串|未定义}仅当目录创建时显示
        `recursive`设置为`true`.

异步创建目录。

回调被赋予一个可能的异常，并且`recursive`是`true`这
创建的第一个目录路径，`(err[, path])`.
`path`仍然可以`undefined`什么时候`recursive`是`true`，如果没有目录
创建。

可选`options`参数可以是指定`mode`（权限
和粘滞位），或具有`mode`属性和`recursive`
属性，指示是否应创建父目录。叫
`fs.mkdir()`什么时候`path`是存在的目录，仅导致错误
什么时候`recursive`是假的。

```mjs
import { mkdir } from 'node:fs';

// Creates /tmp/a/apple, regardless of whether `/tmp` and /tmp/a exist.
mkdir('/tmp/a/apple', { recursive: true }, (err) => {
  if (err) throw err;
});
```

在 Windows 上，使用`fs.mkdir()`在根目录上，即使使用递归也会
导致错误：

```mjs
import { mkdir } from 'node:fs';

mkdir('/', { recursive: true }, (err) => {
  // => [Error: EPERM: operation not permitted, mkdir 'C:\']
});
```

参见 POSIX mkdir（2） 文档了解更多详情。

### `fs.mkdtemp(prefix[, options], callback)`

<!-- YAML
added: v5.10.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version:
      - v16.5.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/39028
    description: The `prefix` parameter now accepts an empty string.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v6.2.1
    pr-url: https://github.com/nodejs/node/pull/6828
    description: The `callback` parameter is optional now.
-->

*   `prefix`{字符串}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   `callback`{函数}
    *   `err`{错误}
    *   `directory`{字符串}

创建唯一的临时目录。

生成六个要追加在所需字符后面的随机字符
`prefix`以创建唯一的临时目录。由于平台
不一致，避免尾随`X`字符`prefix`.一些平台，
特别是BSD，可以返回六个以上的随机字符，并替换
尾随`X`字符`prefix`随机字符。

创建的目录路径作为字符串传递给回调的第二个
参数。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码。

```mjs
import { mkdtemp } from 'node:fs';

mkdtemp(path.join(os.tmpdir(), 'foo-'), (err, directory) => {
  if (err) throw err;
  console.log(directory);
  // Prints: /tmp/foo-itXde2 or C:\Users\...\AppData\Local\Temp\foo-itXde2
});
```

这`fs.mkdtemp()`方法将附加六个随机选择的字符
直接到`prefix`字符串。例如，给定一个目录`/tmp`，如果
目的是创建一个临时目录*在* `/tmp`这`prefix`
必须以尾随平台特定的路径分隔符结尾
(`require('node:path').sep`).

```mjs
import { tmpdir } from 'node:os';
import { mkdtemp } from 'node:fs';

// The parent directory for the new temporary directory
const tmpDir = tmpdir();

// This method is *INCORRECT*:
mkdtemp(tmpDir, (err, directory) => {
  if (err) throw err;
  console.log(directory);
  // Will print something similar to `/tmpabc123`.
  // A new temporary directory is created at the file system root
  // rather than *within* the /tmp directory.
});

// This method is *CORRECT*:
import { sep } from 'node:path';
mkdtemp(`${tmpDir}${sep}`, (err, directory) => {
  if (err) throw err;
  console.log(directory);
  // Will print something similar to `/tmp/abc123`.
  // A new temporary directory is created within
  // the /tmp directory.
});
```

### `fs.open(path[, flags[, mode]], callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v11.1.0
    pr-url: https://github.com/nodejs/node/pull/23767
    description: The `flags` argument is now optional and defaults to `'r'`.
  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/18801
    description: The `as` and `as+` flags are supported now.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `flags`{字符串|数字}看[文件系统支持`flags`][support of file system flags].
    **违约：** `'r'`.
*   `mode`{string|integer}**违约：** `0o666`（可读写）
*   `callback`{函数}
    *   `err`{错误}
    *   `fd`{整数}

异步文件打开。请参阅 POSIX open（2） 文档了解更多详情。

`mode`设置文件模式（权限和粘滞位），但前提是文件
创建。在Windows上，只能操作写入权限;看
[`fs.chmod()`][fs.chmod()].

回调获取两个参数`(err, fd)`.

一些字符 （`< > : " / \ | ? *`） 保留在 Windows 下，如文档所示
由[命名文件、路径和命名空间][Naming Files, Paths, and Namespaces].在 NTFS 下，如果文件名包含
冒号 Node.js 将打开文件系统流，如
[此 MSDN 页面][MSDN-Using-Streams].

基于以下各项的功能`fs.open()`也表现出以下行为：
`fs.writeFile()`,`fs.readFile()`等。

### `fs.opendir(path[, options], callback)`

<!-- YAML
added: v12.12.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version:
     - v13.1.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30114
    description: The `bufferSize` option was introduced.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `bufferSize`{数字}缓冲的目录条目数
        从目录读取时在内部。值越高，越好
        性能，但内存使用率更高。**违约：** `32`
*   `callback`{函数}
    *   `err`{错误}
    *   `dir`{fs.目录}

异步打开目录。请参阅 POSIX opendir（3） 文档
更多详情。

创建一个 {fs。Dir}，其中包含用于读取
并清理目录。

这`encoding`选项设置`path`打开
目录和后续读取操作。

### `fs.read(fd, buffer, offset, length, position, callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22150
    description: The `buffer` parameter can now be any `TypedArray`, or a
                 `DataView`.
  - version: v7.4.0
    pr-url: https://github.com/nodejs/node/pull/10382
    description: The `buffer` parameter can now be a `Uint8Array`.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4518
    description: The `length` parameter can now be `0`.
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|DataView} 数据将具有的缓冲区
    写到。
*   `offset`{整数}中的位置`buffer`将数据写入。
*   `length`{整数}要读取的字节数。
*   `position`{integer|bigint|null}指定从
    文件。如果`position`是`null`或` -1  `，数据将从当前读取
    文件位置，并且文件位置将更新。如果`position`是一个
    整数，文件位置将保持不变。
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesRead`{整数}
    *   `buffer`{缓冲区}

从 指定的文件中读取数据`fd`.

回调给出了三个参数，`(err, bytesRead, buffer)`.

如果未同时修改文件，则当
读取的字节数为零。

如果调用此方法作为其[`util.promisify()`][util.promisify()]ed 版本，它返回
对`Object`跟`bytesRead`和`buffer`性能。

### `fs.read(fd[, options], callback)`

<!-- YAML
added:
 - v13.11.0
 - v12.17.0
changes:
  - version:
     - v13.11.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31402
    description: Options object can be passed in
                 to make buffer, offset, length, and position optional.
-->

*   `fd`{整数}
*   `options`{对象}
    *   `buffer`{缓冲区|TypedArray|数据视图}**违约：** `Buffer.alloc(16384)`
    *   `offset`{整数}**违约：** `0`
    *   `length`{整数}**违约：** `buffer.byteLength - offset`
    *   `position`{integer|bigint|null}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesRead`{整数}
    *   `buffer`{缓冲区}

类似于[`fs.read()`][fs.read()]函数，此版本采用可选
`options`对象。如果不是`options`对象已指定，它将默认使用
高于值。

### `fs.read(fd, buffer[, options], callback)`

<!-- YAML
added: v18.2.0
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|DataView} 数据将具有的缓冲区
    写到。
*   `options`{对象}
    *   `offset`{整数}**违约：** `0`
    *   `length`{整数}**违约：** `buffer.byteLength - offset`
    *   `position`{integer|bigint}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesRead`{整数}
    *   `buffer`{缓冲区}

类似于[`fs.read()`][fs.read()]函数，此版本采用可选
`options`对象。如果不是`options`对象已指定，它将默认使用
高于值。

### `fs.readdir(path[, options], callback)`

<!-- YAML
added: v0.1.8
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22020
    description: New option `withFileTypes` was added.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5616
    description: The `options` parameter was added.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
    *   `withFileTypes`{布尔值}**违约：** `false`
*   `callback`{函数}
    *   `err`{错误}
    *   `files`{字符串\[]|缓冲区\[]|fs.Dirent\[]}

读取目录的内容。回调获取两个参数`(err, files)`
哪里`files`是目录中文件名称的数组，不包括
`'.'`和`'..'`.

请参阅 POSIX 读像器（3） 文档了解更多详情。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
传递给回调的文件名。如果`encoding`设置为`'buffer'`,
返回的文件名将作为 {Buffer} 对象传递。

如果`options.withFileTypes`设置为`true`这`files`数组将包含
{fs.Dirent} 对象。

### `fs.readFile(path[, options], callback)`

<!-- YAML
added: v0.1.29
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37460
    description: The error returned may be an `AggregateError` if more than one
                 error is returned.
  - version:
      - v15.2.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/35911
    description: The options argument may include an AbortSignal to abort an
                 ongoing readFile request.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v5.1.0
    pr-url: https://github.com/nodejs/node/pull/3740
    description: The `callback` will always be called with `null` as the `error`
                 parameter in case of success.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3163
    description: The `path` parameter can be a file descriptor now.
-->

*   `path`{字符串|缓冲区|URL|integer} 文件名或文件描述符
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `null`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'r'`.
    *   `signal`{中止信号} 允许中止正在进行的读取文件
*   `callback`{函数}
    *   `err`{错误|AggregateError}
    *   `data`{字符串|缓冲区}

异步读取文件的全部内容。

```mjs
import { readFile } from 'node:fs';

readFile('/etc/passwd', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

回调传递两个参数`(err, data)`哪里`data`是
文件的内容。

如果未指定编码，则返回原始缓冲区。

如果`options`是一个字符串，然后它指定编码：

```mjs
import { readFile } from 'node:fs';

readFile('/etc/passwd', 'utf8', callback);
```

当路径为目录时，行为`fs.readFile()`和
[`fs.readFileSync()`][fs.readFileSync()]是特定于平台的。在 macOS、Linux 和 Windows 上，一个
将返回错误。在 FreeBSD 上，目录内容的表示形式
将被退回。

```mjs
import { readFile } from 'node:fs';

// macOS, Linux, and Windows
readFile('<directory>', (err, data) => {
  // => [Error: EISDIR: illegal operation on a directory, read <directory>]
});

//  FreeBSD
readFile('<directory>', (err, data) => {
  // => null, <data>
});
```

可以使用`AbortSignal`.如果
请求中止，回调被调用`AbortError`:

```mjs
import { readFile } from 'node:fs';

const controller = new AbortController();
const signal = controller.signal;
readFile(fileInfo[0].name, { signal }, (err, buf) => {
  // ...
});
// When you want to abort the request
controller.abort();
```

这`fs.readFile()`函数缓冲整个文件。为了最大限度地降低内存成本，
如果可能的话，更喜欢通过`fs.createReadStream()`.

中止正在进行的请求不会中止单个操作
系统请求，而是内部缓冲`fs.readFile`执行。

#### 文件描述符

1.  任何指定的文件描述符都必须支持读取。
2.  如果将文件描述符指定为`path`，它不会被关闭
    自然而然。
3.  读数将从当前位置开始。例如，如果文件
    已经有`'Hello World`' 和六个字节使用文件描述符读取，
    调用`fs.readFile()`使用相同的文件描述符，会给
    `'World'`，而不是`'Hello World'`.

#### 性能注意事项

这`fs.readFile()`方法异步读取文件的内容
一次内存一个块，允许事件循环在每个块之间切换。
这允许读取操作对可能可能的其他活动产生较小的影响
正在使用底层的 libuv 线程池，但这意味着这将花费更长的时间
将完整文件读入内存。

额外的读取开销可能因不同的系统而异，并且取决于
正在读取的文件类型。如果文件类型不是常规文件（管道
例如）和 Node.js 无法确定实际文件大小，每次读取
操作将加载 64 KiB 的数据。对于常规文件，每次读取都将处理
512 KiB 的数据。

对于需要尽可能快地读取文件内容的应用程序，它
使用效果更好`fs.read()`直接并供应用程序代码管理
读取文件本身的完整内容。

节点.js GitHub 问题[#25741][]提供更多信息和详细
性能分析`fs.readFile()`对于
不同的节点.js版本。

### `fs.readlink(path[, options], callback)`

<!-- YAML
added: v0.1.31
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   `callback`{函数}
    *   `err`{错误}
    *   `linkString`{字符串|缓冲区}

读取所引用的符号链接的内容`path`.回调得到
两个参数`(err, linkString)`.

有关更多详细信息，请参阅 POSIX 阅读链接（2） 文档。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
传递给回调的链接路径。如果`encoding`设置为`'buffer'`,
返回的链接路径将作为 {Buffer} 对象传递。

### `fs.readv(fd, buffers[, position], callback)`

<!-- YAML
added:
  - v13.13.0
  - v12.17.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `fd`{整数}
*   `buffers`{ArrayBufferView\[]}
*   `position`{integer|null}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesRead`{整数}
    *   `buffers`{ArrayBufferView\[]}

从 指定的文件中读取`fd`并写入数组`ArrayBufferView`s
用`readv()`.

`position`是从文件开头开始的偏移量，数据从何处
应阅读。如果`typeof position !== 'number'`，数据将被读取
从当前位置。

回调将给出三个参数：`err`,`bytesRead`和
`buffers`.`bytesRead`是从文件中读取的字节数。

如果调用此方法作为其[`util.promisify()`][util.promisify()]ed 版本，它返回
对`Object`跟`bytesRead`和`buffers`性能。

### `fs.realpath(path[, options], callback)`

<!-- YAML
added: v0.1.31
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/13028
    description: Pipe/Socket resolve support was added.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7899
    description: Calling `realpath` now works again for various edge cases
                 on Windows.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/3594
    description: The `cache` parameter was removed.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   `callback`{函数}
    *   `err`{错误}
    *   `resolvedPath`{字符串|缓冲区}

通过解析`.`,`..`和
符号链接。

规范路径名不一定是唯一的。硬链接和绑定挂载可以
通过多个路径名公开文件系统实体。

这个函数的行为类似于 realpath（3）， 但有一些例外：

1.  不对不区分大小写的文件系统执行大小写转换。

2.  符号链接的最大数量是独立于平台的，并且通常
    （远）高于本机 realpath（3） 实现所支持的。

这`callback`获取两个参数`(err, resolvedPath)`.可以使用`process.cwd`
以解析相对路径。

仅支持可转换为 UTF8 字符串的路径。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
传递给回调的路径。如果`encoding`设置为`'buffer'`,
返回的路径将作为 {Buffer} 对象传递。

如果`path`解析为套接字或管道，函数将返回系统
该对象的依赖名称。

### `fs.realpath.native(path[, options], callback)`

<!-- YAML
added: v9.2.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   `callback`{函数}
    *   `err`{错误}
    *   `resolvedPath`{字符串|缓冲区}

异步实路径（3）.

这`callback`获取两个参数`(err, resolvedPath)`.

仅支持可转换为 UTF8 字符串的路径。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
传递给回调的路径。如果`encoding`设置为`'buffer'`,
返回的路径将作为 {Buffer} 对象传递。

在 Linux 上，当 Node.js 与 musl libc 链接时，procfs 文件系统必须
安装在`/proc`为了使此功能正常工作。格列布克没有
此限制。

### `fs.rename(oldPath, newPath, callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `oldPath` and `newPath` parameters can be WHATWG `URL`
                 objects using `file:` protocol. Support is currently still
                 *experimental*.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `oldPath`{字符串|缓冲区|网址}
*   `newPath`{字符串|缓冲区|网址}
*   `callback`{函数}
    *   `err`{错误}

异步重命名文件`oldPath`到提供的路径名
如`newPath`.在以下情况下`newPath`已经存在，它将
被覆盖。如果目录位于`newPath`，错误将
而是被提出来。除了可能的异常之外，没有其他参数
交给完成回调。

另请参见：重命名（2）。

```mjs
import { rename } from 'node:fs';

rename('oldFile.txt', 'newFile.txt', (err) => {
  if (err) throw err;
  console.log('Rename complete!');
});
```

### `fs.rmdir(path[, options], callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37216
    description: "Using `fs.rmdir(path, { recursive: true })` on a `path` that is
                 a file is no longer permitted and results in an `ENOENT` error
                 on Windows and an `ENOTDIR` error on POSIX."
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37216
    description: "Using `fs.rmdir(path, { recursive: true })` on a `path` that
                 does not exist is no longer permitted and results in a `ENOENT`
                 error."
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37302
    description: The `recursive` option is deprecated, using it triggers a
                 deprecation warning.
  - version: v14.14.0
    pr-url: https://github.com/nodejs/node/pull/35579
    description: The `recursive` option is deprecated, use `fs.rm` instead.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30644
    description: The `maxBusyTries` option is renamed to `maxRetries`, and its
                 default is 0. The `emfileWait` option has been removed, and
                 `EMFILE` errors use the same retry logic as other errors. The
                 `retryDelay` option is now supported. `ENFILE` errors are now
                 retried.
  - version: v12.10.0
    pr-url: https://github.com/nodejs/node/pull/29168
    description: The `recursive`, `maxBusyTries`, and `emfileWait` options are
                 now supported.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameters can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `maxRetries`{整数}如果`EBUSY`,`EMFILE`,`ENFILE`,`ENOTEMPTY`或
        `EPERM`遇到错误，节点.js以线性方式重试操作
        退避等待`retryDelay`每次尝试时都会延长几毫秒。此选项
        表示重试次数。如果`recursive`
        选项不是`true`.**违约：** `0`.
    *   `recursive`{布尔值}如果`true`，执行递归目录删除。在
        递归模式，操作在失败时重试。**违约：** `false`.
        **荒废的。**
    *   `retryDelay`{整数}等待的时间（以毫秒为单位）介于
        重试。如果`recursive`选项不是`true`.
        **违约：** `100`.
*   `callback`{函数}
    *   `err`{错误}

异步 rmdir（2）.除了可能的异常之外，没有给出其他参数
到完成回调。

用`fs.rmdir()`在文件（而不是目录）上，导致`ENOENT`错误
窗口和`ENOTDIR`POSIX 上的错误。

要获取类似于`rm -rf`Unix 命令，使用[`fs.rm()`][fs.rm()]
与选项`{ recursive: true, force: true }`.

### `fs.rm(path[, options], callback)`

<!-- YAML
added: v14.14.0
changes:
  - version:
      - v17.3.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41132
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `force`{布尔值}什么时候`true`，则在以下情况下将忽略异常`path`做
        不存在。**违约：** `false`.
    *   `maxRetries`{整数}如果`EBUSY`,`EMFILE`,`ENFILE`,`ENOTEMPTY`或
        `EPERM`遇到错误，节点.js将重试线性操作
        退避等待`retryDelay`每次尝试时都会延长几毫秒。此选项
        表示重试次数。如果`recursive`
        选项不是`true`.**违约：** `0`.
    *   `recursive`{布尔值}如果`true`中，执行递归删除。在
        递归模式操作在失败时重试。**违约：** `false`.
    *   `retryDelay`{整数}等待的时间（以毫秒为单位）介于
        重试。如果`recursive`选项不是`true`.
        **违约：** `100`.
*   `callback`{函数}
    *   `err`{错误}

异步删除文件和目录（以标准 POSIX 为模型）`rm`
实用程序）。除了可能的异常之外，没有其他参数提供给
完成回调。

### `fs.stat(path[, options], callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   `callback`{函数}
    *   `err`{错误}
    *   `stats`{fs.统计}

异步统计（2）.回调获取两个参数`(err, stats)`哪里
`stats`是一个 {fs。Stats} 对象。

如果出现错误，`err.code`将是其中之一[常见系统错误][Common System Errors].

用`fs.stat()`在调用之前检查文件是否存在
`fs.open()`,`fs.readFile()`或`fs.writeFile()`不建议这样做。
相反，用户代码应直接打开/读/写文件并处理
如果文件不可用，则会引发错误。

要检查文件是否存在而不在事后对其进行操作，[`fs.access()`][fs.access()]
建议使用。

例如，给定以下目录结构：

```text
- txtDir
-- file.txt
- app.js
```

下一个程序将检查给定路径的统计信息：

```mjs
import { stat } from 'node:fs';

const pathsToCheck = ['./txtDir', './txtDir/file.txt'];

for (let i = 0; i < pathsToCheck.length; i++) {
  stat(pathsToCheck[i], (err, stats) => {
    console.log(stats.isDirectory());
    console.log(stats);
  });
}
```

生成的输出将类似于：

```console
true
Stats {
  dev: 16777220,
  mode: 16877,
  nlink: 3,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4096,
  ino: 14214262,
  size: 96,
  blocks: 0,
  atimeMs: 1561174653071.963,
  mtimeMs: 1561174614583.3518,
  ctimeMs: 1561174626623.5366,
  birthtimeMs: 1561174126937.2893,
  atime: 2019-06-22T03:37:33.072Z,
  mtime: 2019-06-22T03:36:54.583Z,
  ctime: 2019-06-22T03:37:06.624Z,
  birthtime: 2019-06-22T03:28:46.937Z
}
false
Stats {
  dev: 16777220,
  mode: 33188,
  nlink: 1,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4096,
  ino: 14214074,
  size: 8,
  blocks: 8,
  atimeMs: 1561174616618.8555,
  mtimeMs: 1561174614584,
  ctimeMs: 1561174614583.8145,
  birthtimeMs: 1561174007710.7478,
  atime: 2019-06-22T03:36:56.619Z,
  mtime: 2019-06-22T03:36:54.584Z,
  ctime: 2019-06-22T03:36:54.584Z,
  birthtime: 2019-06-22T03:26:47.711Z
}
```

### `fs.symlink(target, path[, type], callback)`

<!-- YAML
added: v0.1.31
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23724
    description: If the `type` argument is left undefined, Node will autodetect
                 `target` type and automatically select `dir` or `file`.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `target` and `path` parameters can be WHATWG `URL` objects
                 using `file:` protocol. Support is currently still
                 *experimental*.
-->

*   `target`{字符串|缓冲区|网址}
*   `path`{字符串|缓冲区|网址}
*   `type`{字符串|null}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}

创建名为`path`指向`target`.除
完成回调可能出现异常。

参见 POSIX 符号链接（2） 文档了解更多详情。

这`type`参数仅在 Windows 上可用，在其他平台上被忽略。
可以设置为`'dir'`,`'file'`或`'junction'`.如果`type`参数为
不是字符串，节点.js将自动检测`target`类型和用途`'file'`或`'dir'`.
如果`target`不存在，`'file'`将使用。窗户交汇点
要求目标路径为绝对路径。使用时`'junction'`这
`target`参数将自动规范化为绝对路径。

相对目标相对于链接的父目录。

```mjs
import { symlink } from 'node:fs';

symlink('./mew', './mewtwo', callback);
```

上面的示例创建一个符号链接`mewtwo`它指向`mew`在
同一目录：

```bash
$ tree .
.
├── mew
└── mewtwo -> ./mew
```

### `fs.truncate(path[, len], callback)`

<!-- YAML
added: v0.8.6
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37460
    description: The error returned may be an `AggregateError` if more than one
                 error is returned.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `len`{整数}**违约：** `0`
*   `callback`{函数}
    *   `err`{错误|AggregateError}

截断文件。除了可能的异常之外，没有其他参数
交给完成回调。文件描述符也可以作为
第一个参数。在这种情况下，`fs.ftruncate()`被调用。

```mjs
import { truncate } from 'node:fs';
// Assuming that 'path/file.txt' is a regular file.
truncate('path/file.txt', (err) => {
  if (err) throw err;
  console.log('path/file.txt was truncated');
});
```

```cjs
const { truncate } = require('node:fs');
// Assuming that 'path/file.txt' is a regular file.
truncate('path/file.txt', (err) => {
  if (err) throw err;
  console.log('path/file.txt was truncated');
});
```

传递文件描述符已弃用，并可能导致引发错误
在未来。

有关更多详细信息，请参阅 POSIX 截断（2） 文档。

### `fs.unlink(path, callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `path`{字符串|缓冲区|网址}
*   `callback`{函数}
    *   `err`{错误}

异步删除文件或符号链接。除
完成回调可能出现异常。

```mjs
import { unlink } from 'node:fs';
// Assuming that 'path/file.txt' is a regular file.
unlink('path/file.txt', (err) => {
  if (err) throw err;
  console.log('path/file.txt was deleted');
});
```

`fs.unlink()`将不适用于空目录或其他目录。删除
目录， 使用[`fs.rmdir()`][fs.rmdir()].

有关更多详细信息，请参阅 POSIX 取消链接（2） 文档。

### `fs.unwatchFile(filename[, listener])`

<!-- YAML
added: v0.1.31
-->

*   `filename`{字符串|缓冲区|网址}
*   `listener`{函数}可选，先前附加的侦听器使用
    `fs.watchFile()`

停止监视 上的更改`filename`.如果`listener`指定，仅指定
删除了特定的侦听器。否则*都*侦听器被删除，
有效停止观看`filename`.

叫`fs.unwatchFile()`文件名未被监视是
没有操作，不是错误。

用[`fs.watch()`][fs.watch()]比`fs.watchFile()`和
`fs.unwatchFile()`.`fs.watch()`应该使用而不是`fs.watchFile()`
和`fs.unwatchFile()`如果可能。

### `fs.utimes(path, atime, mtime, callback)`

<!-- YAML
added: v0.4.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11919
    description: "`NaN`, `Infinity`, and `-Infinity` are no longer valid time
                 specifiers."
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v4.1.0
    pr-url: https://github.com/nodejs/node/pull/2387
    description: Numeric strings, `NaN`, and `Infinity` are now allowed
                 time specifiers.
-->

*   `path`{字符串|缓冲区|网址}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}
*   `callback`{函数}
    *   `err`{错误}

更改 引用的对象的文件系统时间戳`path`.

这`atime`和`mtime`参数遵循以下规则：

*   值可以是表示 Unix 纪元时间（以秒为单位）的数字，
    `Date`s，或数字字符串，如`'123456789.0'`.
*   如果该值不能转换为数字，或者`NaN`,`Infinity`或
    `-Infinity`一`Error`将被抛出。

### `fs.watch(filename[, options][, listener])`

<!-- YAML
added: v0.5.10
changes:
  - version:
      - v15.9.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37190
    description: Added support for closing the watcher with an AbortSignal.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `filename` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7831
    description: The passed `options` object will never be modified.
-->

*   `filename`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `persistent`{布尔值}指示进程是否应继续运行
        只要文件被监视。**违约：** `true`.
    *   `recursive`{布尔值}指示是否所有子目录都应
        已监视，或仅监视当前目录。当目录
        指定且仅在支持的平台上（请参阅[警告][caveats]).**违约：**
        `false`.
    *   `encoding`{字符串}指定要用于
        文件名传递给侦听器。**违约：** `'utf8'`.
    *   `signal`{AbortSignal} 允许使用 AbortSignal 关闭观察程序。
*   `listener`{函数|未定义}**违约：** `undefined`
    *   `eventType`{字符串}
    *   `filename`{字符串|缓冲区}
*   返回值：{fs。FSWatcher}

监视 上的更改`filename`哪里`filename`是文件或
目录。

第二个参数是可选的。如果`options`以字符串形式提供，它
指定`encoding`.否则`options`应作为对象传递。

侦听器回调获取两个参数`(eventType, filename)`.`eventType`
是`'rename'`或`'change'`和`filename`是文件的名称
触发了事件。

在大多数平台上，`'rename'`每当出现文件名或
在目录中消失。

侦听器回调附加到`'change'`触发者的事件
{fs.FSWatcher}，但它与`'change'`的值
`eventType`.

如果`signal`已通过，中止相应的中止控制器将关闭
返回的 {fs.FSWatcher}.

#### 警告

<!--type=misc-->

这`fs.watch`API 在各个平台之间不是 100% 一致的，并且
在某些情况下不可用。

递归选项仅在 macOS 和 Windows 上受支持。
一`ERR_FEATURE_UNAVAILABLE_ON_PLATFORM`将引发异常
当该选项在不支持它的平台上使用时。

在 Windows 上，如果移动或移动监视目录，则不会发出任何事件
重 命名。一`EPERM`删除监视目录时报告错误。

##### 可用性

<!--type=misc-->

此功能取决于底层操作系统提供的方法
以接收文件系统更改的通知。

*   在 Linux 系统上，这使用[`inotify(7)`][inotify(7)].
*   在BSD系统上，这使用[`kqueue(2)`][kqueue(2)].
*   在 macOS 上，此选项使用[`kqueue(2)`][kqueue(2)]对于文件和[`FSEvents`][FSEvents]为
    目录。
*   在 SunOS 系统（包括 Solaris 和 SmartOS）上，这使用[`event ports`][event ports].
*   在 Windows 系统上，此功能取决于[`ReadDirectoryChangesW`][ReadDirectoryChangesW].
*   在 AIX 系统上，此功能取决于[`AHAFS`][AHAFS]，必须启用。
*   在 IBM i 系统上，不支持此功能。

如果基础功能由于某种原因不可用，则
`fs.watch()`将无法运行，并可能引发异常。
例如，监视文件或目录可能不可靠，并且在某些
在网络文件系统（NFS、SMB 等）或主机文件系统上不可能的情况
当使用虚拟化软件（如 Vagrant 或 Docker）时。

仍然可以使用`fs.watchFile()`，它使用统计轮询，但是
这种方法速度较慢且可靠性较差。

##### 伊诺德斯

<!--type=misc-->

在 Linux 和 macOS 系统上，`fs.watch()`解析路径[inode][]和
观看inode。如果删除并重新创建监视路径，则会为其分配
一个新的inode。监视将发出删除事件，但将继续
观看*源语言*inode.不会发出新 inode 的事件。
这是预期的行为。

AIX 文件在文件的生存期内保留相同的 inode。保存和关闭
AIX 上的监视文件将导致两个通知（一个用于添加新的
内容，一个用于截断）。

##### 文件名参数

<!--type=misc-->

提供`filename`回调中的参数仅在 Linux 上受支持，
macOS、Windows 和 AIX。即使在支持的平台上，`filename`并不总是
保证提供。因此，不要假设`filename`参数为
总是在回调中提供，并且有一些回退逻辑（如果是`null`.

```mjs
import { watch } from 'node:fs';
watch('somedir', (eventType, filename) => {
  console.log(`event type is: ${eventType}`);
  if (filename) {
    console.log(`filename provided: ${filename}`);
  } else {
    console.log('filename not provided');
  }
});
```

### `fs.watchFile(filename[, options], listener)`

<!-- YAML
added: v0.1.31
changes:
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: The `bigint` option is now supported.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `filename` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
-->

*   `filename`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}**违约：** `false`
    *   `persistent`{布尔值}**违约：** `true`
    *   `interval`{整数}**违约：** `5007`
*   `listener`{函数}
    *   `current`{fs.统计}
    *   `previous`{fs.统计}
*   返回值：{fs。StatWatcher}

监视 上的更改`filename`.回调`listener`将调用每个
访问文件的时间。

这`options`参数可以省略。如果提供，它应该是一个对象。这
`options`对象可以包含一个名为`persistent`表示
只要文件被监视，进程是否应该继续运行。
这`options`对象可以指定`interval`属性指示的频率
目标应以毫秒为单位进行轮询。

这`listener`获取当前统计对象和上一个参数的两个参数
统计对象：

```mjs
import { watchFile } from 'node:fs';

watchFile('message.text', (curr, prev) => {
  console.log(`the current mtime is: ${curr.mtime}`);
  console.log(`the previous mtime was: ${prev.mtime}`);
});
```

这些统计对象是`fs.Stat`.如果`bigint`选项是`true`,
这些对象中的数值指定为`BigInt`s.

要在文件被修改时收到通知，而不仅仅是被访问，这是必要的
进行比较`curr.mtimeMs`和`prev.mtimeMs`.

当`fs.watchFile`操作导致`ENOENT`错误，它
将调用侦听器一次，所有字段都归零（或者，对于日期，
Unix Epoch）。如果稍后创建该文件，则将调用侦听器
再次，使用最新的统计对象。这是功能上的更改，因为
版本0.10.

用[`fs.watch()`][fs.watch()]比`fs.watchFile`和
`fs.unwatchFile`.`fs.watch`应该使用而不是`fs.watchFile`和
`fs.unwatchFile`如果可能。

当文件被监视时`fs.watchFile()`消失并重新出现，
然后内容`previous`在第二个回调事件中（文件的
重新出现）将与内容相同`previous`在第一个
回调事件（其消失）。

在以下情况下会发生这种情况：

*   文件被删除，然后进行还原
*   文件被重命名，然后再次重命名回其原始名称

### `fs.write(fd, buffer, offset[, length[, position]], callback)`

<!-- YAML
added: v0.0.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `buffer` parameter won't coerce unsupported input to
                 strings anymore.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22150
    description: The `buffer` parameter can now be any `TypedArray` or a
                 `DataView`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.4.0
    pr-url: https://github.com/nodejs/node/pull/10382
    description: The `buffer` parameter can now be a `Uint8Array`.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/7856
    description: The `offset` and `length` parameters are optional now.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|数据视图}
*   `offset`{整数}**违约：** `0`
*   `length`{整数}**违约：** `buffer.byteLength - offset`
*   `position`{integer|null}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesWritten`{整数}
    *   `buffer`{缓冲区|TypedArray|数据视图}

写`buffer`到 指定的文件`fd`.

`offset`确定要写入的缓冲区部分，以及`length`是
一个整数，指定要写入的字节数。

`position`是指此数据从文件开头开始的偏移量
应该写。如果`typeof position !== 'number'`，数据将被写入
在当前位置。参见 pwrite（2）。

回调将给出三个参数`(err, bytesWritten, buffer)`哪里
`bytesWritten`指定多少个*字节*写自`buffer`.

如果调用此方法作为其[`util.promisify()`][util.promisify()]ed 版本，它返回
对`Object`跟`bytesWritten`和`buffer`性能。

使用不安全`fs.write()`在同一文件上多次，无需等待
用于回调。对于此方案，[`fs.createWriteStream()`][fs.createWriteStream()]是
推荐。

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

### `fs.write(fd, buffer[, options], callback)`

<!-- YAML
added: v18.3.0
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|数据视图}
*   `options`{对象}
    *   `offset`{整数}**违约：** `0`
    *   `length`{整数}**违约：** `buffer.byteLength - offset`
    *   `position`{整数}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesWritten`{整数}
    *   `buffer`{缓冲区|TypedArray|数据视图}

写`buffer`到 指定的文件`fd`.

与上述类似`fs.write`函数，此版本采用
自选`options`对象。如果不是`options`对象被指定，它将
默认值为上述值。

### `fs.write(fd, string[, position[, encoding]], callback)`

<!-- YAML
added: v0.11.5
changes:
  - version: v17.8.0
    pr-url: https://github.com/nodejs/node/pull/42149
    description: Passing to the `string` parameter an object with an own
                 `toString` function is deprecated.
  - version: v14.12.0
    pr-url: https://github.com/nodejs/node/pull/34993
    description: The `string` parameter will stringify an object with an
                 explicit `toString` function.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `string` parameter won't coerce unsupported input to
                 strings anymore.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/7856
    description: The `position` parameter is optional now.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
-->

*   `fd`{整数}
*   `string`{字符串|对象}
*   `position`{integer|null}**违约：** `null`
*   `encoding`{字符串}**违约：** `'utf8'`
*   `callback`{函数}
    *   `err`{错误}
    *   `written`{整数}
    *   `string`{字符串}

写`string`到 指定的文件`fd`.如果`string`不是字符串，或者
具有自己的对象`toString`函数属性，则引发异常。

`position`是指此数据从文件开头开始的偏移量
应该写。如果`typeof position !== 'number'`数据将写入
当前位置。参见 pwrite（2）。

`encoding`是预期的字符串编码。

回调将接收参数`(err, written, string)`哪里`written`
指定多少个*字节*需要写入的传递字符串。字节
写入不一定与写入的字符串相同。看
[`Buffer.byteLength`][Buffer.byteLength].

使用不安全`fs.write()`在同一文件上多次，无需等待
用于回调。对于此方案，[`fs.createWriteStream()`][fs.createWriteStream()]是
推荐。

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

在 Windows 上，如果文件描述符连接到控制台（例如`fd == 1`
或`stdout`） 包含非 ASCII 字符的字符串将不会呈现
默认情况下正确，无论使用何种编码。
可以通过更改
活动代码页，其中包含`chcp 65001`命令。查看[断续器][chcp]文档以获取更多信息
详。

### `fs.writeFile(file, data[, options], callback)`

<!-- YAML
added: v0.1.29
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v17.8.0
    pr-url: https://github.com/nodejs/node/pull/42149
    description: Passing to the `string` parameter an object with an own
                 `toString` function is deprecated.
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37460
    description: The error returned may be an `AggregateError` if more than one
                 error is returned.
  - version:
      - v15.2.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/35993
    description: The options argument may include an AbortSignal to abort an
                 ongoing writeFile request.
  - version: v14.12.0
    pr-url: https://github.com/nodejs/node/pull/34993
    description: The `data` parameter will stringify an object with an
                 explicit `toString` function.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `data` parameter won't coerce unsupported input to
                 strings anymore.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22150
    description: The `data` parameter can now be any `TypedArray` or a
                 `DataView`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/12562
    description: The `callback` parameter is no longer optional. Not passing
                 it will throw a `TypeError` at runtime.
  - version: v7.4.0
    pr-url: https://github.com/nodejs/node/pull/10382
    description: The `data` parameter can now be a `Uint8Array`.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7897
    description: The `callback` parameter is no longer optional. Not passing
                 it will emit a deprecation warning with id DEP0013.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3163
    description: The `file` parameter can be a file descriptor now.
-->

*   `file`{字符串|缓冲区|URL|integer} 文件名或文件描述符
*   `data`{字符串|缓冲区|TypedArray|数据视图|对象}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `mode`{整数}**违约：** `0o666`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'w'`.
    *   `signal`{中止信号} 允许中止正在进行的写入文件
*   `callback`{函数}
    *   `err`{错误|AggregateError}

什么时候`file`是文件名，将数据异步写入文件，将
文件（如果已存在）。`data`可以是字符串或缓冲区。

什么时候`file`是文件描述符，行为类似于调用
`fs.write()`直接（建议使用）。请参阅以下有关使用的说明
文件描述符。

这`encoding`选项在以下情况下被忽略：`data`是缓冲区。

这`mode`选项仅影响新创建的文件。看[`fs.open()`][fs.open()]
了解更多详情。

```mjs
import { writeFile } from 'node:fs';
import { Buffer } from 'node:buffer';

const data = new Uint8Array(Buffer.from('Hello Node.js'));
writeFile('message.txt', data, (err) => {
  if (err) throw err;
  console.log('The file has been saved!');
});
```

如果`options`是一个字符串，然后它指定编码：

```mjs
import { writeFile } from 'node:fs';

writeFile('message.txt', 'Hello Node.js', 'utf8', callback);
```

使用不安全`fs.writeFile()`在同一文件上多次，没有
等待回调。对于此方案，[`fs.createWriteStream()`][fs.createWriteStream()]是
推荐。

类似于`fs.readFile`-`fs.writeFile`是一种方便的方法
执行多个`write`在内部调用以写入传递给它的缓冲区。
对于性能敏感的代码，请考虑使用[`fs.createWriteStream()`][fs.createWriteStream()].

可以使用 {AbortSignal} 来取消`fs.writeFile()`.
取消是“尽力而为”，并且可能仍然会有一定数量的数据
待写。

```mjs
import { writeFile } from 'node:fs';
import { Buffer } from 'node:buffer';

const controller = new AbortController();
const { signal } = controller;
const data = new Uint8Array(Buffer.from('Hello Node.js'));
writeFile('message.txt', data, { signal }, (err) => {
  // When a request is aborted - the callback is called with an AbortError
});
// When the request should be aborted
controller.abort();
```

中止正在进行的请求不会中止单个操作
系统请求，而是内部缓冲`fs.writeFile`执行。

#### 用`fs.writeFile()`使用文件描述符

什么时候`file`是一个文件描述符，行为几乎与直接相同
叫`fs.write()`喜欢：

```mjs
import { write } from 'node:fs';
import { Buffer } from 'node:buffer';

write(fd, Buffer.from(data, options.encoding), callback);
```

与直接呼叫的区别`fs.write()`就是在一些不寻常的
条件`fs.write()`可能只写入缓冲区的一部分，并且需要
重试写入剩余数据，而`fs.writeFile()`重试直到
数据完全写入（或发生错误）。

其含义是混淆的常见来源。在
文件描述符大小写，文件不被替换！数据不一定
写入文件开头，文件的原始数据可能保留
在新写入的数据之前和/或之后。

例如，如果`fs.writeFile()`连续调用两次，第一次写入
字符串`'Hello'`，然后写入字符串`', World'`，该文件将包含
`'Hello, World'`，并且可能包含文件的一些原始数据（取决于
，以及文件描述符的位置）。如果
使用了文件名而不是描述符，该文件将得到保证
仅包含`', World'`.

### `fs.writev(fd, buffers[, position], callback)`

<!-- YAML
added: v12.9.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `fd`{整数}
*   `buffers`{ArrayBufferView\[]}
*   `position`{integer|null}**违约：** `null`
*   `callback`{函数}
    *   `err`{错误}
    *   `bytesWritten`{整数}
    *   `buffers`{ArrayBufferView\[]}

编写一个数组`ArrayBufferView`s 到`fd`用
`writev()`.

`position`是此数据从文件开头开始的偏移量
应该写。如果`typeof position !== 'number'`，数据将被写入
在当前位置。

回调将给出三个参数：`err`,`bytesWritten`和
`buffers`.`bytesWritten`是从中写入的字节数`buffers`.

如果此方法是[`util.promisify()`][util.promisify()]ed，它返回一个承诺
`Object`跟`bytesWritten`和`buffers`性能。

使用不安全`fs.writev()`在同一文件上多次，没有
等待回调。对于此方案，请使用[`fs.createWriteStream()`][fs.createWriteStream()].

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

## 同步接口

同步 API 同步执行所有操作，阻止
事件循环，直到操作完成或失败。

### `fs.accessSync(path[, mode])`

<!-- YAML
added: v0.11.15
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{整数}**违约：** `fs.constants.F_OK`

同步测试用户对指定文件或目录的权限
由`path`.这`mode`参数是一个可选整数，它指定
要执行的辅助功能检查。`mode`应为值
`fs.constants.F_OK`或由以下任一项的按位 OR 组成的掩码
`fs.constants.R_OK`,`fs.constants.W_OK`和`fs.constants.X_OK`（例如
`fs.constants.W_OK | fs.constants.R_OK`).检查[文件访问常量][File access constants]为
可能的值`mode`.

如果任何辅助功能检查失败，则`Error`将被抛出。否则
该方法将返回`undefined`.

```mjs
import { accessSync, constants } from 'node:fs';

try {
  accessSync('etc/passwd', constants.R_OK | constants.W_OK);
  console.log('can read/write');
} catch (err) {
  console.error('no access!');
}
```

### `fs.appendFileSync(path, data[, options])`

<!-- YAML
added: v0.6.7
changes:
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7831
    description: The passed `options` object will never be modified.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3163
    description: The `file` parameter can be a file descriptor now.
-->

*   `path`{字符串|缓冲区|URL|编号} 文件名或文件描述符
*   `data`{字符串|缓冲区}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `mode`{整数}**违约：** `0o666`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'a'`.

同步将数据追加到文件，如果尚未创建文件，则创建该文件
存在。`data`可以是字符串或 {缓冲区}。

这`mode`选项仅影响新创建的文件。看[`fs.open()`][fs.open()]
了解更多详情。

```mjs
import { appendFileSync } from 'node:fs';

try {
  appendFileSync('message.txt', 'data to append');
  console.log('The "data to append" was appended to file!');
} catch (err) {
  /* Handle the error */
}
```

如果`options`是一个字符串，然后它指定编码：

```mjs
import { appendFileSync } from 'node:fs';

appendFileSync('message.txt', 'data to append', 'utf8');
```

这`path`可以指定为已打开的数字文件描述符
用于追加（使用`fs.open()`或`fs.openSync()`).文件描述符将
不会自动关闭。

```mjs
import { openSync, closeSync, appendFileSync } from 'node:fs';

let fd;

try {
  fd = openSync('message.txt', 'a');
  appendFileSync(fd, 'data to append', 'utf8');
} catch (err) {
  /* Handle the error */
} finally {
  if (fd !== undefined)
    closeSync(fd);
}
```

### `fs.chmodSync(path, mode)`

<!-- YAML
added: v0.6.7
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{string|integer}

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.chmod()`][fs.chmod()].

参见 POSIX chmod（2） 文档了解更多详情。

### `fs.chownSync(path, uid, gid)`

<!-- YAML
added: v0.1.97
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `uid`{整数}
*   `gid`{整数}

同步更改文件的所有者和组。返回`undefined`.
这是的同步版本[`fs.chown()`][fs.chown()].

参见 POSIX chown（2） 文档了解更多详情。

### `fs.closeSync(fd)`

<!-- YAML
added: v0.1.21
-->

*   `fd`{整数}

关闭文件描述符。返回`undefined`.

叫`fs.closeSync()`在任何文件描述符 （`fd`） 当前正在使用
通过任何其他`fs`操作可能会导致未定义的行为。

请参阅 POSIX close（2） 文档了解更多详情。

### `fs.copyFileSync(src, dest[, mode])`

<!-- YAML
added: v8.5.0
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/27044
    description: Changed 'flags' argument to 'mode' and imposed
                 stricter type validation.
-->

*   `src`{字符串|缓冲区|要复制的 URL} 源文件名
*   `dest`{字符串|缓冲区|URL} 复制操作的目标文件名
*   `mode`{整数} 用于复制操作的修饰符。**违约：** `0`.

同步拷贝`src`自`dest`.默认情况下，`dest`如果它被覆盖
已存在。返回`undefined`.Node.js 对
复制操作的原子性。如果在目标文件之后发生错误
已打开写入，节点.js将尝试删除目标。

`mode`是指定行为的可选整数
的复制操作。可以创建由按位
两个或多个值的 OR（例如
`fs.constants.COPYFILE_EXCL | fs.constants.COPYFILE_FICLONE`).

*   `fs.constants.COPYFILE_EXCL`：如果出现以下情况，复制操作将失败：`dest`已经
    存在。
*   `fs.constants.COPYFILE_FICLONE`：复制操作将尝试创建一个
    写入时复制引用链接。如果平台不支持写入时复制，则
    使用回退复制机制。
*   `fs.constants.COPYFILE_FICLONE_FORCE`：复制操作将尝试
    创建写入时复制引用链接。如果平台不支持
    写入时复制，则操作将失败。

```mjs
import { copyFileSync, constants } from 'node:fs';

// destination.txt will be created or overwritten by default.
copyFileSync('source.txt', 'destination.txt');
console.log('source.txt was copied to destination.txt');

// By using COPYFILE_EXCL, the operation will fail if destination.txt exists.
copyFileSync('source.txt', 'destination.txt', constants.COPYFILE_EXCL);
```

### `fs.cpSync(src, dest[, options])`

<!-- YAML
added: v16.7.0
changes:
  - version:
    - v17.6.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41819
    description: Accepts an additional `verbatimSymlinks` option to specify
                 whether to perform path resolution for symlinks.
-->

> 稳定性： 1 - 实验

*   `src`{字符串|URL} 要复制的源路径。
*   `dest`{字符串|要复制到的 URL} 目标路径。
*   `options`{对象}
    *   `dereference`{boolean} dereference symlinks.**违约：** `false`.
    *   `errorOnExist`{布尔值} 当`force`是`false`和目标
        存在，引发错误。**违约：** `false`.
    *   `filter`{函数}过滤复制的文件/目录的函数。返回
        `true`以复制项目，`false`以忽略它。**违约：** `undefined`
    *   `force`{布尔值} 覆盖现有文件或目录。副本
        如果将此设置为 false 并且目标，则操作将忽略错误
        存在。使用`errorOnExist`选项以更改此行为。
        **违约：** `true`.
    *   `preserveTimestamps`{布尔值}什么时候`true`时间戳来自`src`将
        被保存下来。**违约：** `false`.
    *   `recursive`{布尔} 递归复制目录**违约：** `false`
    *   `verbatimSymlinks`{布尔值}什么时候`true`，符号链接的路径分辨率将
        被跳过。**违约：** `false`

同步复制整个目录结构`src`自`dest`,
包括子目录和文件。

将一个目录复制到另一个目录时，不支持 globs 和
行为类似于`cp dir1/ dir2/`.

### `fs.existsSync(path)`

<!-- YAML
added: v0.1.21
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   返回：{布尔值}

返回`true`如果路径存在，`false`否则。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.exists()`][fs.exists()].

`fs.exists()`已弃用，但`fs.existsSync()`莫。这`callback`
参数`fs.exists()`接受与其他参数不一致的参数
节点.js回调。`fs.existsSync()`不使用回调。

```mjs
import { existsSync } from 'node:fs';

if (existsSync('/etc/passwd'))
  console.log('The path exists.');
```

### `fs.fchmodSync(fd, mode)`

<!-- YAML
added: v0.4.7
-->

*   `fd`{整数}
*   `mode`{string|integer}

设置对文件的权限。返回`undefined`.

参见 POSIX fchmod（2） 文档了解更多详情。

### `fs.fchownSync(fd, uid, gid)`

<!-- YAML
added: v0.4.7
-->

*   `fd`{整数}
*   `uid`{整数}文件的新所有者的用户 ID。
*   `gid`{整数}文件的新组的组 ID。

设置文件的所有者。返回`undefined`.

请参阅 POSIX fchown（2） 文档了解更多详情。

### `fs.fdatasyncSync(fd)`

<!-- YAML
added: v0.1.96
-->

*   `fd`{整数}

将与文件关联的所有当前排队的 I/O 操作强制到
操作系统的同步 I/O 完成状态。请参阅 POSIX
fdatasync（2） 文档了解详细信息。返回`undefined`.

### `fs.fstatSync(fd[, options])`

<!-- YAML
added: v0.1.95
changes:
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
-->

*   `fd`{整数}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
*   返回值：{fs。统计}

检索 {fs.Stats} 用于文件描述符。

请参阅 POSIX fstat（2） 文档了解更多详情。

### `fs.fsyncSync(fd)`

<!-- YAML
added: v0.1.96
-->

*   `fd`{整数}

请求将打开的文件描述符的所有数据刷新到存储中
装置。具体实现是特定于操作系统和设备的。
有关更多详细信息，请参阅 POSIX fsync（2） 文档。返回`undefined`.

### `fs.ftruncateSync(fd[, len])`

<!-- YAML
added: v0.8.6
-->

*   `fd`{整数}
*   `len`{整数}**违约：** `0`

截断文件描述符。返回`undefined`.

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.ftruncate()`][fs.ftruncate()].

### `fs.futimesSync(fd, atime, mtime)`

<!-- YAML
added: v0.4.2
changes:
  - version: v4.1.0
    pr-url: https://github.com/nodejs/node/pull/2387
    description: Numeric strings, `NaN`, and `Infinity` are now allowed
                 time specifiers.
-->

*   `fd`{整数}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}

同步版本[`fs.futimes()`][fs.futimes()].返回`undefined`.

### `fs.lchmodSync(path, mode)`

<!-- YAML
deprecated: v0.4.7
-->

*   `path`{字符串|缓冲区|网址}
*   `mode`{整数}

更改符号链接的权限。返回`undefined`.

此方法仅在 macOS 上实现。

参见 POSIX lchmod（2） 文档了解更多详情。

### `fs.lchownSync(path, uid, gid)`

<!-- YAML
changes:
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/21498
    description: This API is no longer deprecated.
  - version: v0.4.7
    description: Documentation-only deprecation.
-->

*   `path`{字符串|缓冲区|网址}
*   `uid`{整数}文件的新所有者的用户 ID。
*   `gid`{整数}文件的新组的组 ID。

设置路径的所有者。返回`undefined`.

参见 POSIX lchown（2） 文档了解更多详情。

### `fs.lutimesSync(path, atime, mtime)`

<!-- YAML
added:
  - v14.5.0
  - v12.19.0
-->

*   `path`{字符串|缓冲区|网址}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}

更改 引用的符号链接的文件系统时间戳`path`.
返回`undefined`，或在参数不正确时引发异常，或者
操作失败。这是的同步版本[`fs.lutimes()`][fs.lutimes()].

### `fs.linkSync(existingPath, newPath)`

<!-- YAML
added: v0.1.31
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `existingPath` and `newPath` parameters can be WHATWG
                 `URL` objects using `file:` protocol. Support is currently
                 still *experimental*.
-->

*   `existingPath`{字符串|缓冲区|网址}
*   `newPath`{字符串|缓冲区|网址}

从 创建新链接`existingPath`到`newPath`.查看 POSIX
link（2） 文档提供更多详细信息。返回`undefined`.

### `fs.lstatSync(path[, options])`

<!-- YAML
added: v0.1.30
changes:
  - version:
    - v15.3.0
    - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/33716
    description: Accepts a `throwIfNoEntry` option to specify whether
                 an exception should be thrown if the entry does not exist.
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
    *   `throwIfNoEntry`{布尔值}是否将引发异常
        如果不存在文件系统条目，而不是返回`undefined`.
        **违约：** `true`.
*   返回值：{fs。统计}

检索 {fs.Stats} 表示由 引用的符号链接`path`.

参见 POSIX lstat（2） 文档了解更多详情。

### `fs.mkdirSync(path[, options])`

<!-- YAML
added: v0.1.21
changes:
  - version:
     - v13.11.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31530
    description: In `recursive` mode, the first created path is returned now.
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/21875
    description: The second argument can now be an `options` object with
                 `recursive` and `mode` properties.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{Object|integer}
    *   `recursive`{布尔值}**违约：** `false`
    *   `mode`{string|integer}在 Windows 上不受支持。**违约：** `0o777`.
*   返回：{字符串|未定义}

同步创建目录。返回`undefined`，或者如果`recursive`是
`true`，则创建的第一个目录路径。
这是的同步版本[`fs.mkdir()`][fs.mkdir()].

参见 POSIX mkdir（2） 文档了解更多详情。

### `fs.mkdtempSync(prefix[, options])`

<!-- YAML
added: v5.10.0
changes:
  - version:
      - v16.5.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/39028
    description: The `prefix` parameter now accepts an empty string.
-->

*   `prefix`{字符串}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{字符串}

返回创建的目录路径。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.mkdtemp()`][fs.mkdtemp()].

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码。

### `fs.opendirSync(path[, options])`

<!-- YAML
added: v12.12.0
changes:
  - version:
     - v13.1.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30114
    description: The `bufferSize` option was introduced.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `bufferSize`{数字}缓冲的目录条目数
        从目录读取时在内部。值越高，越好
        性能，但内存使用率更高。**违约：** `32`
*   返回值：{fs。目录}

同步打开目录。参见 opendir（3）。

创建一个 {fs。Dir}，其中包含用于读取
并清理目录。

这`encoding`选项设置`path`打开
目录和后续读取操作。

### `fs.openSync(path[, flags[, mode]])`

<!-- YAML
added: v0.1.21
changes:
  - version: v11.1.0
    pr-url: https://github.com/nodejs/node/pull/23767
    description: The `flags` argument is now optional and defaults to `'r'`.
  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/18801
    description: The `as` and `as+` flags are supported now.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `flags`{字符串|数字}**违约：** `'r'`.
    看[文件系统支持`flags`][support of file system flags].
*   `mode`{string|integer}**违约：** `0o666`
*   返回值：{数字}

返回一个表示文件描述符的整数。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.open()`][fs.open()].

### `fs.readdirSync(path[, options])`

<!-- YAML
added: v0.1.21
changes:
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22020
    description: New option `withFileTypes` was added.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
    *   `withFileTypes`{布尔值}**违约：** `false`
*   返回：{字符串\[]|缓冲区\[]|fs.Dirent\[]}

读取目录的内容。

请参阅 POSIX 读像器（3） 文档了解更多详情。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
返回的文件名。如果`encoding`设置为`'buffer'`,
返回的文件名将作为 {Buffer} 对象传递。

如果`options.withFileTypes`设置为`true`，结果将包含
{fs.Dirent} 对象。

### `fs.readFileSync(path[, options])`

<!-- YAML
added: v0.1.8
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3163
    description: The `path` parameter can be a file descriptor now.
-->

*   `path`{字符串|缓冲区|URL|integer} 文件名或文件描述符
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `null`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'r'`.
*   返回：{字符串|缓冲区}

返回`path`.

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.readFile()`][fs.readFile()].

如果`encoding`指定选项，则此函数返回
字符串。否则，它将返回一个缓冲区。

似[`fs.readFile()`][fs.readFile()]，当路径是目录时，行为
`fs.readFileSync()`是特定于平台的。

```mjs
import { readFileSync } from 'node:fs';

// macOS, Linux, and Windows
readFileSync('<directory>');
// => [Error: EISDIR: illegal operation on a directory, read <directory>]

//  FreeBSD
readFileSync('<directory>'); // => <data>
```

### `fs.readlinkSync(path[, options])`

<!-- YAML
added: v0.1.31
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{字符串|缓冲区}

返回符号链接的字符串值。

有关更多详细信息，请参阅 POSIX 阅读链接（2） 文档。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
返回的链接路径。如果`encoding`设置为`'buffer'`,
返回的链接路径将作为 {Buffer} 对象传递。

### `fs.readSync(fd, buffer, offset, length[, position])`

<!-- YAML
added: v0.1.21
changes:
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22150
    description: The `buffer` parameter can now be any `TypedArray` or a
                 `DataView`.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4518
    description: The `length` parameter can now be `0`.
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|数据视图}
*   `offset`{整数}
*   `length`{整数}
*   `position`{integer|bigint|null}**违约：** `null`
*   返回值：{数字}

返回`bytesRead`.

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.read()`][fs.read()].

### `fs.readSync(fd, buffer[, options])`

<!-- YAML
added:
 - v13.13.0
 - v12.17.0
changes:
  - version:
     - v13.13.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32460
    description: Options object can be passed in
                 to make offset, length, and position optional.
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|数据视图}
*   `options`{对象}
    *   `offset`{整数}**违约：** `0`
    *   `length`{整数}**违约：** `buffer.byteLength - offset`
    *   `position`{integer|bigint|null}**违约：** `null`
*   返回值：{数字}

返回`bytesRead`.

与上述类似`fs.readSync`函数，此版本采用可选`options`对象。
如果不是`options`指定对象，它将默认使用上述值。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.read()`][fs.read()].

### `fs.readvSync(fd, buffers[, position])`

<!-- YAML
added:
 - v13.13.0
 - v12.17.0
-->

*   `fd`{整数}
*   `buffers`{ArrayBufferView\[]}
*   `position`{integer|null}**违约：** `null`
*   返回：{数字} 读取的字节数。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.readv()`][fs.readv()].

### `fs.realpathSync(path[, options])`

<!-- YAML
added: v0.1.31
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/13028
    description: Pipe/Socket resolve support was added.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using
                 `file:` protocol.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7899
    description: Calling `realpathSync` now works again for various edge cases
                 on Windows.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/3594
    description: The `cache` parameter was removed.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{字符串|缓冲区}

返回解析的路径名。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.realpath()`][fs.realpath()].

### `fs.realpathSync.native(path[, options])`

<!-- YAML
added: v9.2.0
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{字符串|对象}
    *   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{字符串|缓冲区}

同步实路径（3）.

仅支持可转换为 UTF8 字符串的路径。

可选`options`参数可以是指定编码的字符串，也可以是
具有`encoding`属性，指定要使用的字符编码
返回的路径。如果`encoding`设置为`'buffer'`,
返回的路径将作为 {Buffer} 对象传递。

在 Linux 上，当 Node.js 与 musl libc 链接时，procfs 文件系统必须
安装在`/proc`为了使此功能正常工作。格列布克没有
此限制。

### `fs.renameSync(oldPath, newPath)`

<!-- YAML
added: v0.1.21
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `oldPath` and `newPath` parameters can be WHATWG `URL`
                 objects using `file:` protocol. Support is currently still
                 *experimental*.
-->

*   `oldPath`{字符串|缓冲区|网址}
*   `newPath`{字符串|缓冲区|网址}

重命名文件`oldPath`自`newPath`.返回`undefined`.

参见 POSIX rename（2） 文档了解更多详情。

### `fs.rmdirSync(path[, options])`

<!-- YAML
added: v0.1.21
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37216
    description: "Using `fs.rmdirSync(path, { recursive: true })` on a `path`
                 that is a file is no longer permitted and results in an
                 `ENOENT` error on Windows and an `ENOTDIR` error on POSIX."
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37216
    description: "Using `fs.rmdirSync(path, { recursive: true })` on a `path`
                 that does not exist is no longer permitted and results in a
                 `ENOENT` error."
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37302
    description: The `recursive` option is deprecated, using it triggers a
                 deprecation warning.
  - version: v14.14.0
    pr-url: https://github.com/nodejs/node/pull/35579
    description: The `recursive` option is deprecated, use `fs.rmSync` instead.
  - version:
     - v13.3.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30644
    description: The `maxBusyTries` option is renamed to `maxRetries`, and its
                 default is 0. The `emfileWait` option has been removed, and
                 `EMFILE` errors use the same retry logic as other errors. The
                 `retryDelay` option is now supported. `ENFILE` errors are now
                 retried.
  - version: v12.10.0
    pr-url: https://github.com/nodejs/node/pull/29168
    description: The `recursive`, `maxBusyTries`, and `emfileWait` options are
                 now supported.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameters can be a WHATWG `URL` object using
                 `file:` protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `maxRetries`{整数}如果`EBUSY`,`EMFILE`,`ENFILE`,`ENOTEMPTY`或
        `EPERM`遇到错误，节点.js以线性方式重试操作
        退避等待`retryDelay`每次尝试时都会延长几毫秒。此选项
        表示重试次数。如果`recursive`
        选项不是`true`.**违约：** `0`.
    *   `recursive`{布尔值}如果`true`，执行递归目录删除。在
        递归模式，操作在失败时重试。**违约：** `false`.
        **荒废的。**
    *   `retryDelay`{整数}等待的时间（以毫秒为单位）介于
        重试。如果`recursive`选项不是`true`.
        **违约：** `100`.

同步 rmdir（2）.返回`undefined`.

用`fs.rmdirSync()`在文件（而不是目录）上，导致`ENOENT`错误
在 Windows 上，并且`ENOTDIR`POSIX 上的错误。

要获取类似于`rm -rf`Unix 命令，使用[`fs.rmSync()`][fs.rmSync()]
与选项`{ recursive: true, force: true }`.

### `fs.rmSync(path[, options])`

<!-- YAML
added: v14.14.0
changes:
  - version:
      - v17.3.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41132
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `force`{布尔值}什么时候`true`，则在以下情况下将忽略异常`path`做
        不存在。**违约：** `false`.
    *   `maxRetries`{整数}如果`EBUSY`,`EMFILE`,`ENFILE`,`ENOTEMPTY`或
        `EPERM`遇到错误，节点.js将重试线性操作
        退避等待`retryDelay`每次尝试时都会延长几毫秒。此选项
        表示重试次数。如果`recursive`
        选项不是`true`.**违约：** `0`.
    *   `recursive`{布尔值}如果`true`，执行递归目录删除。在
        递归模式操作在失败时重试。**违约：** `false`.
    *   `retryDelay`{整数}等待的时间（以毫秒为单位）介于
        重试。如果`recursive`选项不是`true`.
        **违约：** `100`.

同步删除文件和目录（以标准 POSIX 为模型）`rm`
实用程序）。返回`undefined`.

### `fs.statSync(path[, options])`

<!-- YAML
added: v0.1.21
changes:
  - version:
    - v15.3.0
    - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/33716
    description: Accepts a `throwIfNoEntry` option to specify whether
                 an exception should be thrown if the entry does not exist.
  - version: v10.5.0
    pr-url: https://github.com/nodejs/node/pull/20220
    description: Accepts an additional `options` object to specify whether
                 the numeric values returned should be bigint.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}
*   `options`{对象}
    *   `bigint`{布尔值}返回的数值是否
        {fs.Stats} 对象应为`bigint`.**违约：** `false`.
    *   `throwIfNoEntry`{布尔值}是否将引发异常
        如果不存在文件系统条目，而不是返回`undefined`.
        **违约：** `true`.
*   返回值：{fs。统计}

检索 {fs.路径的统计信息} 。

### `fs.symlinkSync(target, path[, type])`

<!-- YAML
added: v0.1.31
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23724
    description: If the `type` argument is left undefined, Node will autodetect
                 `target` type and automatically select `dir` or `file`.
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `target` and `path` parameters can be WHATWG `URL` objects
                 using `file:` protocol. Support is currently still
                 *experimental*.
-->

*   `target`{字符串|缓冲区|网址}
*   `path`{字符串|缓冲区|网址}
*   `type`{字符串|null}**违约：** `null`

返回`undefined`.

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.symlink()`][fs.symlink()].

### `fs.truncateSync(path[, len])`

<!-- YAML
added: v0.8.6
-->

*   `path`{字符串|缓冲区|网址}
*   `len`{整数}**违约：** `0`

截断文件。返回`undefined`.文件描述符也可以是
作为第一个参数传递。在这种情况下，`fs.ftruncateSync()`被调用。

传递文件描述符已弃用，并可能导致引发错误
在未来。

### `fs.unlinkSync(path)`

<!-- YAML
added: v0.1.21
changes:
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
-->

*   `path`{字符串|缓冲区|网址}

同步取消链接（2）。返回`undefined`.

### `fs.utimesSync(path, atime, mtime)`

<!-- YAML
added: v0.4.2
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11919
    description: "`NaN`, `Infinity`, and `-Infinity` are no longer valid time
                 specifiers."
  - version: v7.6.0
    pr-url: https://github.com/nodejs/node/pull/10739
    description: The `path` parameter can be a WHATWG `URL` object using `file:`
                 protocol.
  - version: v4.1.0
    pr-url: https://github.com/nodejs/node/pull/2387
    description: Numeric strings, `NaN`, and `Infinity` are now allowed
                 time specifiers.
-->

*   `path`{字符串|缓冲区|网址}
*   `atime`{数字|字符串|日期}
*   `mtime`{数字|字符串|日期}

返回`undefined`.

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.utimes()`][fs.utimes()].

### `fs.writeFileSync(file, data[, options])`

<!-- YAML
added: v0.1.29
changes:
  - version: v17.8.0
    pr-url: https://github.com/nodejs/node/pull/42149
    description: Passing to the `data` parameter an object with an own
                 `toString` function is deprecated.
  - version: v14.12.0
    pr-url: https://github.com/nodejs/node/pull/34993
    description: The `data` parameter will stringify an object with an
                 explicit `toString` function.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `data` parameter won't coerce unsupported input to
                 strings anymore.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22150
    description: The `data` parameter can now be any `TypedArray` or a
                 `DataView`.
  - version: v7.4.0
    pr-url: https://github.com/nodejs/node/pull/10382
    description: The `data` parameter can now be a `Uint8Array`.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3163
    description: The `file` parameter can be a file descriptor now.
-->

*   `file`{字符串|缓冲区|URL|integer} 文件名或文件描述符
*   `data`{字符串|缓冲区|TypedArray|数据视图|对象}
*   `options`{对象|字符串}
    *   `encoding`{字符串|null}**违约：** `'utf8'`
    *   `mode`{整数}**违约：** `0o666`
    *   `flag`{字符串}看[文件系统支持`flags`][support of file system flags].**违约：** `'w'`.

返回`undefined`.

这`mode`选项仅影响新创建的文件。看[`fs.open()`][fs.open()]
了解更多详情。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.writeFile()`][fs.writeFile()].

### `fs.writeSync(fd, buffer, offset[, length[, position]])`

<!-- YAML
added: v0.1.21
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `buffer` parameter won't coerce unsupported input to
                 strings anymore.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/22150
    description: The `buffer` parameter can now be any `TypedArray` or a
                 `DataView`.
  - version: v7.4.0
    pr-url: https://github.com/nodejs/node/pull/10382
    description: The `buffer` parameter can now be a `Uint8Array`.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/7856
    description: The `offset` and `length` parameters are optional now.
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|数据视图}
*   `offset`{整数}**违约：** `0`
*   `length`{整数}**违约：** `buffer.byteLength - offset`
*   `position`{integer|null}**违约：** `null`
*   返回：{数字} 写入的字节数。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.write(fd, buffer...)`][fs.write(fd, buffer...)].

### `fs.writeSync(fd, buffer[, options])`

<!-- YAML
added: v18.3.0
-->

*   `fd`{整数}
*   `buffer`{缓冲区|TypedArray|数据视图}
*   `options`{对象}
    *   `offset`{整数}**违约：** `0`
    *   `length`{整数}**违约：** `buffer.byteLength - offset`
    *   `position`{整数}**违约：** `null`
*   返回：{数字} 写入的字节数。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.write(fd, buffer...)`][fs.write(fd, buffer...)].

### `fs.writeSync(fd, string[, position[, encoding]])`

<!-- YAML
added: v0.11.5
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/31030
    description: The `string` parameter won't coerce unsupported input to
                 strings anymore.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/7856
    description: The `position` parameter is optional now.
-->

*   `fd`{整数}
*   `string`{字符串}
*   `position`{integer|null}**违约：** `null`
*   `encoding`{字符串}**违约：** `'utf8'`
*   返回：{数字} 写入的字节数。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.write(fd, string...)`][fs.write(fd, string...)].

### `fs.writevSync(fd, buffers[, position])`

<!-- YAML
added: v12.9.0
-->

*   `fd`{整数}
*   `buffers`{ArrayBufferView\[]}
*   `position`{integer|null}**违约：** `null`
*   返回：{数字} 写入的字节数。

有关详细信息，请参阅 异步版本的文档
此 API：[`fs.writev()`][fs.writev()].

## 常见对象

公共对象由所有文件系统 API 变体共享
（承诺、回调和同步）。

### 类：`fs.Dir`

<!-- YAML
added: v12.12.0
-->

表示目录流的类。

创建者[`fs.opendir()`][fs.opendir()],[`fs.opendirSync()`][fs.opendirSync()]或
[`fsPromises.opendir()`][fsPromises.opendir()].

```mjs
import { opendir } from 'node:fs/promises';

try {
  const dir = await opendir('./');
  for await (const dirent of dir)
    console.log(dirent.name);
} catch (err) {
  console.error(err);
}
```

使用异步迭代器时，{fs.Dir} 对象将自动
迭代器退出后关闭。

#### `dir.close()`

<!-- YAML
added: v12.12.0
-->

*   返回： {承诺}

异步关闭目录的基础资源句柄。
后续读取将导致错误。

返回一个承诺，该承诺将在资源完成后解决
闭。

#### `dir.close(callback)`

<!-- YAML
added: v12.12.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `callback`{函数}
    *   `err`{错误}

异步关闭目录的基础资源句柄。
后续读取将导致错误。

这`callback`将在资源句柄关闭后调用。

#### `dir.closeSync()`

<!-- YAML
added: v12.12.0
-->

同步关闭目录的基础资源句柄。
后续读取将导致错误。

#### `dir.path`

<!-- YAML
added: v12.12.0
-->

*   {字符串}

此目录的只读路径，已提供给[`fs.opendir()`][fs.opendir()],
[`fs.opendirSync()`][fs.opendirSync()]或[`fsPromises.opendir()`][fsPromises.opendir()].

#### `dir.read()`

<!-- YAML
added: v12.12.0
-->

*   返回：{Promise} 包含 {fs。Dirent|null}

通过 readdir（3） 异步读取下一个目录条目作为
{fs.Dirent}.

返回一个 promise，该 promise 将使用 {fs 进行解析。Dirent}，或`null`
如果没有更多要读取的目录条目。

此函数返回的目录条目没有特定的顺序，因为
由操作系统的底层目录机制提供。
在迭代目录时添加或删除的条目可能不是
包含在迭代结果中。

#### `dir.read(callback)`

<!-- YAML
added: v12.12.0
-->

*   `callback`{函数}
    *   `err`{错误}
    *   `dirent`{fs.Dirent|null}

通过 readdir（3） 异步读取下一个目录条目作为
{fs.Dirent}.

读取完成后，`callback`将使用
{fs.Dirent}，或`null`如果没有更多要读取的目录条目。

此函数返回的目录条目没有特定的顺序，因为
由操作系统的底层目录机制提供。
在迭代目录时添加或删除的条目可能不是
包含在迭代结果中。

#### `dir.readSync()`

<!-- YAML
added: v12.12.0
-->

*   返回值：{fs。Dirent|null}

以 {fs 形式同步读取下一个目录条目。Dirent}.查看
POSIX 读写器（3） 文档了解更多详情。

如果没有更多要读取的目录条目，`null`将被退回。

此函数返回的目录条目没有特定的顺序，因为
由操作系统的底层目录机制提供。
在迭代目录时添加或删除的条目可能不是
包含在迭代结果中。

#### `dir[Symbol.asyncIterator]()`

<!-- YAML
added: v12.12.0
-->

*   返回： {AsyncIterator} of {fs.Dirent}

异步循环访问目录，直到所有条目都具有
已阅读。有关更多详细信息，请参阅 POSIX 读写器（3） 文档。

异步迭代器返回的条目始终是 {fs。Dirent}.
这`null`案例来自`dir.read()`在内部处理。

请参阅 {fs。Dir} 作为示例。

此迭代器返回的目录条目没有特定的顺序，如
由操作系统的底层目录机制提供。
在迭代目录时添加或删除的条目可能不是
包含在迭代结果中。

### 类：`fs.Dirent`

<!-- YAML
added: v10.10.0
-->

目录条目的表示形式，可以是文件或子目录
在目录中，通过读取 {fs 返回。目录}.这
目录条目是文件名和文件类型对的组合。

此外，当[`fs.readdir()`][fs.readdir()]或[`fs.readdirSync()`][fs.readdirSync()]调用方式为
这`withFileTypes`选项设置为`true`，则生成的数组填充
{fs.Dirent} 对象，而不是字符串或 {Buffer}s。

#### `dirent.isBlockDevice()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述了一个块设备。

#### `dirent.isCharacterDevice()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述一个字符设备。

#### `dirent.isDirectory()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述文件系统
目录。

#### `dirent.isFIFO()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述了先进先出
（先进先出）管道。

#### `dirent.isFile()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述一个常规文件。

#### `dirent.isSocket()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述一个套接字。

#### `dirent.isSymbolicLink()`

<!-- YAML
added: v10.10.0
-->

*   返回：{布尔值}

返回`true`如果 {fs.Dirent} 对象描述了一个符号链接。

#### `dirent.name`

<!-- YAML
added: v10.10.0
-->

*   {字符串|缓冲区}

此 {fs. 的文件名。Dirent} 对象引用。此类型
值由`options.encoding`传递到[`fs.readdir()`][fs.readdir()]或
[`fs.readdirSync()`][fs.readdirSync()].

### 类：`fs.FSWatcher`

<!-- YAML
added: v0.5.8
-->

*   Extends {EventEmitter}

成功调用[`fs.watch()`][fs.watch()]方法将返回一个新的 {fs。FSWatcher}
对象。

所有 {fs.FSWatcher} 对象发出`'change'`事件，每当一个特定的观看
文件被修改。

#### 事件：`'change'`

<!-- YAML
added: v0.5.8
-->

*   `eventType`{字符串}已发生的变更事件的类型
*   `filename`{字符串|Buffer} 已更改的文件名（如果相关/可用）

当监视目录或文件中的某些内容发生更改时发出。
查看更多详细信息[`fs.watch()`][fs.watch()].

这`filename`参数可能未提供，具体取决于操作系统
支持。如果`filename`，如果出现以下情况，它将作为 {Buffer} 提供
`fs.watch()`被调用`encoding`选项设置为`'buffer'`否则
`filename`将是 UTF-8 字符串。

```mjs
import { watch } from 'node:fs';
// Example when handled through fs.watch() listener
watch('./tmp', { encoding: 'buffer' }, (eventType, filename) => {
  if (filename) {
    console.log(filename);
    // Prints: <Buffer ...>
  }
});
```

#### 事件：`'close'`

<!-- YAML
added: v10.0.0
-->

当观察程序停止监视更改时发出。已关闭
{fs.FSWatcher} 对象在事件处理程序中不再可用。

#### 事件：`'error'`

<!-- YAML
added: v0.5.8
-->

*   `error`{错误}

在监视文件时发生错误时发出。错误的
{fs.FSWatcher} 对象在事件处理程序中不再可用。

#### `watcher.close()`

<!-- YAML
added: v0.5.8
-->

停止监视给定 {fs 上的更改。FSWatcher}.一旦停止，
{fs.FSWatcher} 对象不再可用。

#### `watcher.ref()`

<!-- YAML
added:
  - v14.3.0
  - v12.20.0
-->

*   返回值：{fs。FSWatcher}

调用时，请求 Node.js事件循环*不*退出只要
{fs.FSWatcher} 处于活动状态。叫`watcher.ref()`多次将有
没有效果。

默认情况下，所有 {fs.FSWatcher} 对象被“引用”，使其正常
无需调用`watcher.ref()`除非`watcher.unref()`一直
之前调用。

#### `watcher.unref()`

<!-- YAML
added:
  - v14.3.0
  - v12.20.0
-->

*   返回值：{fs。FSWatcher}

调用时，活动 {fs.FSWatcher} 对象将不需要 Node.js
事件循环以保持活动状态。如果没有其他活动，请保留
事件循环正在运行，进程可能会在 {fs 之前退出。FSWatcher} 对象的
调用回调。叫`watcher.unref()`多次将有
没有效果。

### 类：`fs.StatWatcher`

<!-- YAML
added:
  - v14.3.0
  - v12.20.0
-->

*   Extends {EventEmitter}

成功调用`fs.watchFile()`方法将返回一个新的 {fs。StatWatcher}
对象。

#### `watcher.ref()`

<!-- YAML
added:
  - v14.3.0
  - v12.20.0
-->

*   返回值：{fs。StatWatcher}

调用时，请求 Node.js事件循环*不*退出只要
{fs.StatWatcher} 处于活动状态。叫`watcher.ref()`多次将有
没有效果。

默认情况下，所有 {fs.StatWatcher} 对象是“ref'ed”的，使其正常
无需调用`watcher.ref()`除非`watcher.unref()`一直
之前调用。

#### `watcher.unref()`

<!-- YAML
added:
  - v14.3.0
  - v12.20.0
-->

*   返回值：{fs。StatWatcher}

调用时，活动 {fs.StatWatcher} 对象不需要 Node.js
事件循环以保持活动状态。如果没有其他活动，请保留
事件循环正在运行，进程可能会在 {fs 之前退出。StatWatcher} object's
调用回调。叫`watcher.unref()`多次将有
没有效果。

### 类：`fs.ReadStream`

<!-- YAML
added: v0.1.93
-->

*   扩展：{流。可读}

{fs. 的实例。ReadStream} 是使用
[`fs.createReadStream()`][fs.createReadStream()]功能。

#### 事件：`'close'`

<!-- YAML
added: v0.1.93
-->

当 {fs.ReadStream} 的基础文件描述符已关闭。

#### 事件：`'open'`

<!-- YAML
added: v0.1.93
-->

*   `fd`{整数}{fs 使用的整数文件描述符。ReadStream}.

当 {fs.ReadStream} 的文件描述符已打开。

#### 事件：`'ready'`

<!-- YAML
added: v9.11.0
-->

当 {fs.ReadStream} 已准备就绪，可供使用。

火灾后立即发生`'open'`.

#### `readStream.bytesRead`

<!-- YAML
added: v6.4.0
-->

*   {数字}

到目前为止已读取的字节数。

#### `readStream.path`

<!-- YAML
added: v0.1.93
-->

*   {字符串|缓冲区}

流从中读取的文件的路径，如第一个
参数`fs.createReadStream()`.如果`path`作为字符串传递，则
`readStream.path`将是一个字符串。如果`path`作为 {缓冲区} 传递，则
`readStream.path`将是 {缓冲区}。如果`fd`，然后
`readStream.path`将是`undefined`.

#### `readStream.pending`

<!-- YAML
added:
 - v11.2.0
 - v10.16.0
-->

*   {布尔值}

此属性是`true`如果基础文件尚未打开，
即在`'ready'`发出事件。

### 类：`fs.Stats`

<!-- YAML
added: v0.1.21
changes:
  - version: v8.1.0
    pr-url: https://github.com/nodejs/node/pull/13173
    description: Added times as numbers.
-->

A {fs.Stats} 对象提供有关文件的信息。

从 返回的对象[`fs.stat()`][fs.stat()],[`fs.lstat()`][fs.lstat()],[`fs.fstat()`][fs.fstat()]和
它们的同步对应物属于此类型。
如果`bigint`在`options`传递给这些方法是 true，数值
将是`bigint`而不是`number`，并且该对象将包含其他
纳秒精度属性后缀为`Ns`.

```console
Stats {
  dev: 2114,
  ino: 48064969,
  mode: 33188,
  nlink: 1,
  uid: 85,
  gid: 100,
  rdev: 0,
  size: 527,
  blksize: 4096,
  blocks: 8,
  atimeMs: 1318289051000.1,
  mtimeMs: 1318289051000.1,
  ctimeMs: 1318289051000.1,
  birthtimeMs: 1318289051000.1,
  atime: Mon, 10 Oct 2011 23:24:11 GMT,
  mtime: Mon, 10 Oct 2011 23:24:11 GMT,
  ctime: Mon, 10 Oct 2011 23:24:11 GMT,
  birthtime: Mon, 10 Oct 2011 23:24:11 GMT }
```

`bigint`版本：

```console
BigIntStats {
  dev: 2114n,
  ino: 48064969n,
  mode: 33188n,
  nlink: 1n,
  uid: 85n,
  gid: 100n,
  rdev: 0n,
  size: 527n,
  blksize: 4096n,
  blocks: 8n,
  atimeMs: 1318289051000n,
  mtimeMs: 1318289051000n,
  ctimeMs: 1318289051000n,
  birthtimeMs: 1318289051000n,
  atimeNs: 1318289051000000000n,
  mtimeNs: 1318289051000000000n,
  ctimeNs: 1318289051000000000n,
  birthtimeNs: 1318289051000000000n,
  atime: Mon, 10 Oct 2011 23:24:11 GMT,
  mtime: Mon, 10 Oct 2011 23:24:11 GMT,
  ctime: Mon, 10 Oct 2011 23:24:11 GMT,
  birthtime: Mon, 10 Oct 2011 23:24:11 GMT }
```

#### `stats.isBlockDevice()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述块设备。

#### `stats.isCharacterDevice()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述一个字符设备。

#### `stats.isDirectory()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述文件系统目录。

如果 {fs.Stats} 对象是从以下位置获取的[`fs.lstat()`][fs.lstat()]，此方法将
总是回来`false`.这是因为[`fs.lstat()`][fs.lstat()]返回信息
关于符号链接本身，而不是它解析到的路径。

#### `stats.isFIFO()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述先进先出 （FIFO）
管。

#### `stats.isFile()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述一个常规文件。

#### `stats.isSocket()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述一个套接字。

#### `stats.isSymbolicLink()`

<!-- YAML
added: v0.1.10
-->

*   返回：{布尔值}

返回`true`如果 {fs.Stats} 对象描述一个符号链接。

此方法仅在使用时有效[`fs.lstat()`][fs.lstat()].

#### `stats.dev`

*   {数字|比金}

包含该文件的设备的数字标识符。

#### `stats.ino`

*   {数字|比金}

文件的文件系统特定的“Inode”编号。

#### `stats.mode`

*   {数字|比金}

描述文件类型和模式的位字段。

#### `stats.nlink`

*   {数字|比金}

文件存在的硬链接数。

#### `stats.uid`

*   {数字|比金}

拥有文件的用户的数字用户标识符 （POSIX）。

#### `stats.gid`

*   {数字|比金}

拥有文件的组 （POSIX） 的数字组标识符。

#### `stats.rdev`

*   {数字|比金}

数字设备标识符（如果文件表示设备）。

#### `stats.size`

*   {数字|比金}

文件的大小（以字节为单位）。

如果底层文件系统不支持获取文件大小，
这将是`0`.

#### `stats.blksize`

*   {数字|比金}

i/o 操作的文件系统块大小。

#### `stats.blocks`

*   {数字|比金}

为此文件分配的块数。

#### `stats.atimeMs`

<!-- YAML
added: v8.1.0
-->

*   {数字|比金}

指示上次访问此文件的时间戳，表示为
自 POSIX 纪元以来的毫秒。

#### `stats.mtimeMs`

<!-- YAML
added: v8.1.0
-->

*   {数字|比金}

指示上次修改此文件的时间戳，表示为
自 POSIX 纪元以来的毫秒。

#### `stats.ctimeMs`

<!-- YAML
added: v8.1.0
-->

*   {数字|比金}

表示上次更改文件状态的时间戳
自POSIX时代以来的毫秒。

#### `stats.birthtimeMs`

<!-- YAML
added: v8.1.0
-->

*   {数字|比金}

指示此文件的创建时间的时间戳，表示为
自 POSIX 纪元以来的毫秒。

#### `stats.atimeNs`

<!-- YAML
added: v12.10.0
-->

*   {bigint}

仅在以下情况下存在`bigint: true`传递到生成
对象。
指示上次访问此文件的时间戳，表示为
自POSIX时代以来的纳秒。

#### `stats.mtimeNs`

<!-- YAML
added: v12.10.0
-->

*   {bigint}

仅在以下情况下存在`bigint: true`传递到生成
对象。
指示上次修改此文件的时间戳，表示为
自POSIX时代以来的纳秒。

#### `stats.ctimeNs`

<!-- YAML
added: v12.10.0
-->

*   {bigint}

仅在以下情况下存在`bigint: true`传递到生成
对象。
表示上次更改文件状态的时间戳
自POSIX时代以来的纳秒。

#### `stats.birthtimeNs`

<!-- YAML
added: v12.10.0
-->

*   {bigint}

仅在以下情况下存在`bigint: true`传递到生成
对象。
指示此文件的创建时间的时间戳，表示为
自POSIX时代以来的纳秒。

#### `stats.atime`

<!-- YAML
added: v0.11.13
-->

*   {日期}

指示上次访问此文件的时间戳。

#### `stats.mtime`

<!-- YAML
added: v0.11.13
-->

*   {日期}

指示上次修改此文件的时间戳。

#### `stats.ctime`

<!-- YAML
added: v0.11.13
-->

*   {日期}

指示上次更改文件状态的时间戳。

#### `stats.birthtime`

<!-- YAML
added: v0.11.13
-->

*   {日期}

指示此文件的创建时间的时间戳。

#### 统计时间值

这`atimeMs`,`mtimeMs`,`ctimeMs`,`birthtimeMs`属性是
保存相应时间（以毫秒为单位）的数值。他们
精度是特定于平台的。什么时候`bigint: true`被传递到
生成对象的方法，属性将为[比金茨][bigints],
否则他们将是[数字][MDN-Number].

这`atimeNs`,`mtimeNs`,`ctimeNs`,`birthtimeNs`属性是
[比金茨][bigints]以纳秒为单位保存相应的时间。他们是
仅在以下情况下存在`bigint: true`传递到生成
对象。它们的精度是特定于平台的。

`atime`,`mtime`,`ctime`和`birthtime`是
[`Date`][MDN-Date]对象不同时间的替代表示形式。这
`Date`和数字值未连接。分配新的数字值，或
改变`Date`值，不会反映在相应的替代项中
表示法。

统计对象中的时间具有以下语义：

*   `atime`“访问时间”：上次访问文件数据的时间。改变
    通过 mknod（2）、utimes（2） 和 read（2） 系统调用。
*   `mtime`“修改时间”：上次修改文件数据的时间。
    由 mknod（2）、utimes（2） 和 write（2） 系统调用更改。
*   `ctime`“更改时间”：上次更改文件状态的时间
    （索引数据修改）。由 chmod（2）， chown（2） 更改，
    link（2）， mknod（2）， rename（2）， unlink（2）， utimes（2），
    读 （2） 和写 （2） 系统调用。
*   `birthtime`“出生时间”：文件创建时间。设置一次时
    文件已创建。在出生时间不可用的文件系统上，
    此字段可以改为保留`ctime`或
    `1970-01-01T00:00Z`（即，Unix epoch 时间戳`0`).此值可能更大
    比`atime`或`mtime`在这种情况下。在 Darwin 和其他 FreeBSD 变体上，
    还设置如果`atime`显式设置为早于当前值的值
    `birthtime`使用 utimes（2） 系统调用。

在 Node.js 0.12 之前，`ctime`举行了`birthtime`在 Windows 系统上。如
的 0.12，`ctime`不是“创建时间”，在Unix系统上，它从来都不是。

### 类：`fs.WriteStream`

<!-- YAML
added: v0.1.93
-->

*   扩展 {流。可写}

{fs. 的实例。WriteStream} 是使用
[`fs.createWriteStream()`][fs.createWriteStream()]功能。

#### 事件：`'close'`

<!-- YAML
added: v0.1.93
-->

当 {fs.WriteStream} 的基础文件描述符已关闭。

#### 事件：`'open'`

<!-- YAML
added: v0.1.93
-->

*   `fd`{整数}{fs 使用的整数文件描述符。WriteStream}.

当 {fs.WriteStream} 的文件已打开。

#### 事件：`'ready'`

<!-- YAML
added: v9.11.0
-->

当 {fs.WriteStream} 已准备好使用。

火灾后立即发生`'open'`.

#### `writeStream.bytesWritten`

<!-- YAML
added: v0.4.7
-->

到目前为止写入的字节数。不包括仍在排队的数据
用于写作。

#### `writeStream.close([callback])`

<!-- YAML
added: v0.9.4
-->

*   `callback`{函数}
    *   `err`{错误}

关闭`writeStream`.（可选）接受
将在`writeStream`
已关闭。

#### `writeStream.path`

<!-- YAML
added: v0.1.93
-->

流要写入的文件的路径，如第一个
参数[`fs.createWriteStream()`][fs.createWriteStream()].如果`path`作为字符串传递，则
`writeStream.path`将是一个字符串。如果`path`作为 {缓冲区} 传递，则
`writeStream.path`将是 {缓冲区}。

#### `writeStream.pending`

<!-- YAML
added: v11.2.0
-->

*   {布尔值}

此属性是`true`如果基础文件尚未打开，
即在`'ready'`发出事件。

### `fs.constants`

*   {对象}

返回一个对象，其中包含文件系统的常用常量
操作。

#### FS 常量

以下常量由 导出`fs.constants`和`fsPromises.constants`.

并非每个常量在每个操作系统上都可用;
这对于Windows尤其重要，其中许多POSIX特定于Windows。
定义不可用。
对于便携式应用，建议检查它们是否存在
使用前。

要使用多个常量，请使用按位 OR`|`算子。

例：

```mjs
import { open, constants } from 'node:fs';

const {
  O_RDWR,
  O_CREAT,
  O_EXCL
} = constants;

open('/path/to/my/file', O_RDWR | O_CREAT | O_EXCL, (err, fd) => {
  // ...
});
```

##### 文件访问常量

以下常量旨在用作`mode`参数传递给
[`fsPromises.access()`][fsPromises.access()],[`fs.access()`][fs.access()]和[`fs.accessSync()`][fs.accessSync()].

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>F_OK</code></td>
    <td>Flag indicating that the file is visible to the calling process.
     This is useful for determining if a file exists, but says nothing
     about <code>rwx</code> permissions. Default if no mode is specified.</td>
  </tr>
  <tr>
    <td><code>R_OK</code></td>
    <td>Flag indicating that the file can be read by the calling process.</td>
  </tr>
  <tr>
    <td><code>W_OK</code></td>
    <td>Flag indicating that the file can be written by the calling
    process.</td>
  </tr>
  <tr>
    <td><code>X_OK</code></td>
    <td>Flag indicating that the file can be executed by the calling
    process. This has no effect on Windows
    (will behave like <code>fs.constants.F_OK</code>).</td>
  </tr>
</table>

这些定义在 Windows 上也可用。

##### 文件复制常量

以下常量旨在与[`fs.copyFile()`][fs.copyFile()].

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>COPYFILE_EXCL</code></td>
    <td>If present, the copy operation will fail with an error if the
    destination path already exists.</td>
  </tr>
  <tr>
    <td><code>COPYFILE_FICLONE</code></td>
    <td>If present, the copy operation will attempt to create a
    copy-on-write reflink. If the underlying platform does not support
    copy-on-write, then a fallback copy mechanism is used.</td>
  </tr>
  <tr>
    <td><code>COPYFILE_FICLONE_FORCE</code></td>
    <td>If present, the copy operation will attempt to create a
    copy-on-write reflink. If the underlying platform does not support
    copy-on-write, then the operation will fail with an error.</td>
  </tr>
</table>

这些定义在 Windows 上也可用。

##### 文件打开常量

以下常量旨在与`fs.open()`.

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>O_RDONLY</code></td>
    <td>Flag indicating to open a file for read-only access.</td>
  </tr>
  <tr>
    <td><code>O_WRONLY</code></td>
    <td>Flag indicating to open a file for write-only access.</td>
  </tr>
  <tr>
    <td><code>O_RDWR</code></td>
    <td>Flag indicating to open a file for read-write access.</td>
  </tr>
  <tr>
    <td><code>O_CREAT</code></td>
    <td>Flag indicating to create the file if it does not already exist.</td>
  </tr>
  <tr>
    <td><code>O_EXCL</code></td>
    <td>Flag indicating that opening a file should fail if the
    <code>O_CREAT</code> flag is set and the file already exists.</td>
  </tr>
  <tr>
    <td><code>O_NOCTTY</code></td>
    <td>Flag indicating that if path identifies a terminal device, opening the
    path shall not cause that terminal to become the controlling terminal for
    the process (if the process does not already have one).</td>
  </tr>
  <tr>
    <td><code>O_TRUNC</code></td>
    <td>Flag indicating that if the file exists and is a regular file, and the
    file is opened successfully for write access, its length shall be truncated
    to zero.</td>
  </tr>
  <tr>
    <td><code>O_APPEND</code></td>
    <td>Flag indicating that data will be appended to the end of the file.</td>
  </tr>
  <tr>
    <td><code>O_DIRECTORY</code></td>
    <td>Flag indicating that the open should fail if the path is not a
    directory.</td>
  </tr>
  <tr>
  <td><code>O_NOATIME</code></td>
    <td>Flag indicating reading accesses to the file system will no longer
    result in an update to the <code>atime</code> information associated with
    the file. This flag is available on Linux operating systems only.</td>
  </tr>
  <tr>
    <td><code>O_NOFOLLOW</code></td>
    <td>Flag indicating that the open should fail if the path is a symbolic
    link.</td>
  </tr>
  <tr>
    <td><code>O_SYNC</code></td>
    <td>Flag indicating that the file is opened for synchronized I/O with write
    operations waiting for file integrity.</td>
  </tr>
  <tr>
    <td><code>O_DSYNC</code></td>
    <td>Flag indicating that the file is opened for synchronized I/O with write
    operations waiting for data integrity.</td>
  </tr>
  <tr>
    <td><code>O_SYMLINK</code></td>
    <td>Flag indicating to open the symbolic link itself rather than the
    resource it is pointing to.</td>
  </tr>
  <tr>
    <td><code>O_DIRECT</code></td>
    <td>When set, an attempt will be made to minimize caching effects of file
    I/O.</td>
  </tr>
  <tr>
    <td><code>O_NONBLOCK</code></td>
    <td>Flag indicating to open the file in nonblocking mode when possible.</td>
  </tr>
  <tr>
    <td><code>UV_FS_O_FILEMAP</code></td>
    <td>When set, a memory file mapping is used to access the file. This flag
    is available on Windows operating systems only. On other operating systems,
    this flag is ignored.</td>
  </tr>
</table>

在 Windows 上，仅`O_APPEND`,`O_CREAT`,`O_EXCL`,`O_RDONLY`,`O_RDWR`,
`O_TRUNC`,`O_WRONLY`和`UV_FS_O_FILEMAP`可用。

##### 文件类型常量

以下常量旨在与 {fs 一起使用。Stats} 对象的
`mode`属性，用于确定文件类型。

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>S_IFMT</code></td>
    <td>Bit mask used to extract the file type code.</td>
  </tr>
  <tr>
    <td><code>S_IFREG</code></td>
    <td>File type constant for a regular file.</td>
  </tr>
  <tr>
    <td><code>S_IFDIR</code></td>
    <td>File type constant for a directory.</td>
  </tr>
  <tr>
    <td><code>S_IFCHR</code></td>
    <td>File type constant for a character-oriented device file.</td>
  </tr>
  <tr>
    <td><code>S_IFBLK</code></td>
    <td>File type constant for a block-oriented device file.</td>
  </tr>
  <tr>
    <td><code>S_IFIFO</code></td>
    <td>File type constant for a FIFO/pipe.</td>
  </tr>
  <tr>
    <td><code>S_IFLNK</code></td>
    <td>File type constant for a symbolic link.</td>
  </tr>
  <tr>
    <td><code>S_IFSOCK</code></td>
    <td>File type constant for a socket.</td>
  </tr>
</table>

在 Windows 上，仅`S_IFCHR`,`S_IFDIR`,`S_IFLNK`,`S_IFMT`和`S_IFREG`,
可用。

##### 文件模式常量

以下常量旨在与 {fs 一起使用。Stats} 对象的
`mode`属性，用于确定文件的访问权限。

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>S_IRWXU</code></td>
    <td>File mode indicating readable, writable, and executable by owner.</td>
  </tr>
  <tr>
    <td><code>S_IRUSR</code></td>
    <td>File mode indicating readable by owner.</td>
  </tr>
  <tr>
    <td><code>S_IWUSR</code></td>
    <td>File mode indicating writable by owner.</td>
  </tr>
  <tr>
    <td><code>S_IXUSR</code></td>
    <td>File mode indicating executable by owner.</td>
  </tr>
  <tr>
    <td><code>S_IRWXG</code></td>
    <td>File mode indicating readable, writable, and executable by group.</td>
  </tr>
  <tr>
    <td><code>S_IRGRP</code></td>
    <td>File mode indicating readable by group.</td>
  </tr>
  <tr>
    <td><code>S_IWGRP</code></td>
    <td>File mode indicating writable by group.</td>
  </tr>
  <tr>
    <td><code>S_IXGRP</code></td>
    <td>File mode indicating executable by group.</td>
  </tr>
  <tr>
    <td><code>S_IRWXO</code></td>
    <td>File mode indicating readable, writable, and executable by others.</td>
  </tr>
  <tr>
    <td><code>S_IROTH</code></td>
    <td>File mode indicating readable by others.</td>
  </tr>
  <tr>
    <td><code>S_IWOTH</code></td>
    <td>File mode indicating writable by others.</td>
  </tr>
  <tr>
    <td><code>S_IXOTH</code></td>
    <td>File mode indicating executable by others.</td>
  </tr>
</table>

在 Windows 上，仅`S_IRUSR`和`S_IWUSR`可用。

## 笔记

### 回调和基于承诺的操作排序

因为它们是由底层线程池异步执行的，
使用回调或
基于承诺的方法。

例如，以下内容容易出错，因为`fs.stat()`
操作可能在`fs.rename()`操作：

```js
fs.rename('/tmp/hello', '/tmp/world', (err) => {
  if (err) throw err;
  console.log('renamed complete');
});
fs.stat('/tmp/world', (err, stats) => {
  if (err) throw err;
  console.log(`stats: ${JSON.stringify(stats)}`);
});
```

通过等待结果来正确排序操作非常重要
在调用另一个之前调用另一个：

```mjs
import { rename, stat } from 'node:fs/promises';

const from = '/tmp/hello';
const to = '/tmp/world';

try {
  await rename(from, to);
  const stats = await stat(to);
  console.log(`stats: ${JSON.stringify(stats)}`);
} catch (error) {
  console.error('there was an error:', error.message);
}
```

```cjs
const { rename, stat } = require('node:fs/promises');

(async function(from, to) {
  try {
    await rename(from, to);
    const stats = await stat(to);
    console.log(`stats: ${JSON.stringify(stats)}`);
  } catch (error) {
    console.error('there was an error:', error.message);
  }
})('/tmp/hello', '/tmp/world');
```

或者，在使用回调 API 时，将`fs.stat()`调用回调
的`fs.rename()`操作：

```mjs
import { rename, stat } from 'node:fs';

rename('/tmp/hello', '/tmp/world', (err) => {
  if (err) throw err;
  stat('/tmp/world', (err, stats) => {
    if (err) throw err;
    console.log(`stats: ${JSON.stringify(stats)}`);
  });
});
```

```cjs
const { rename, stat } = require('node:fs/promises');

rename('/tmp/hello', '/tmp/world', (err) => {
  if (err) throw err;
  stat('/tmp/world', (err, stats) => {
    if (err) throw err;
    console.log(`stats: ${JSON.stringify(stats)}`);
  });
});
```

### 文件路径

最`fs`操作接受可能以以下格式指定的文件路径
字符串、{缓冲区} 或 {URL} 对象，使用`file:`协议。

#### 字符串路径

字符串路径被解释为 UTF-8 字符序列，用于标识
绝对或相对文件名。相对路径将解析为相对路径
到当前工作目录，通过调用`process.cwd()`.

在 POSIX 上使用绝对路径的示例：

```mjs
import { open } from 'node:fs/promises';

let fd;
try {
  fd = await open('/open/some/file.txt', 'r');
  // Do something with the file
} finally {
  await fd.close();
}
```

在 POSIX 上使用相对路径的示例（相对于`process.cwd()`):

```mjs
import { open } from 'node:fs/promises';

let fd;
try {
  fd = await open('file.txt', 'r');
  // Do something with the file
} finally {
  await fd.close();
}
```

#### 文件网址路径

<!-- YAML
added: v7.6.0
-->

对于大多数人来说`node:fs`模块函数，`path`或`filename`参数可能是
作为 {URL} 对象传递，使用`file:`协议。

```mjs
import { readFileSync } from 'node:fs';

readFileSync(new URL('file:///tmp/hello'));
```

`file:`URL 始终是绝对路径。

##### 特定于平台的注意事项

在 Windows 上，`file:`{具有主机名的 URL}s 转换为 UNC 路径，而`file:`
{带有驱动器号的 URL}s 转换为本地绝对路径。`file:`{URL}s
没有主机名和没有驱动器号将导致错误：

```mjs
import { readFileSync } from 'node:fs';
// On Windows :

// - WHATWG file URLs with hostname convert to UNC path
// file://hostname/p/a/t/h/file => \\hostname\p\a\t\h\file
readFileSync(new URL('file://hostname/p/a/t/h/file'));

// - WHATWG file URLs with drive letters convert to absolute path
// file:///C:/tmp/hello => C:\tmp\hello
readFileSync(new URL('file:///C:/tmp/hello'));

// - WHATWG file URLs without hostname must have a drive letters
readFileSync(new URL('file:///notdriveletter/p/a/t/h/file'));
readFileSync(new URL('file:///c/p/a/t/h/file'));
// TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must be absolute
```

`file:`{带有驱动器号的 URL} 必须使用`:`作为分隔符紧随其后
驱动器号。使用其他分隔符将导致错误。

在所有其他平台上，`file:`{具有主机名的 URL} 不受支持，并且
将导致错误：

```mjs
import { readFileSync } from 'node:fs';
// On other platforms:

// - WHATWG file URLs with hostname are unsupported
// file://hostname/p/a/t/h/file => throw!
readFileSync(new URL('file://hostname/p/a/t/h/file'));
// TypeError [ERR_INVALID_FILE_URL_PATH]: must be absolute

// - WHATWG file URLs convert to absolute path
// file:///tmp/hello => /tmp/hello
readFileSync(new URL('file:///tmp/hello'));
```

一个`file:`{URL} 具有编码的斜杠字符将导致所有字符出错
平台：

```mjs
import { readFileSync } from 'node:fs';

// On Windows
readFileSync(new URL('file:///C:/p/a/t/h/%2F'));
readFileSync(new URL('file:///C:/p/a/t/h/%2f'));
/* TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must not include encoded
\ or / characters */

// On POSIX
readFileSync(new URL('file:///p/a/t/h/%2F'));
readFileSync(new URL('file:///p/a/t/h/%2f'));
/* TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must not include encoded
/ characters */
```

在 Windows 上，`file:`{URL}s 具有编码的反斜杠将导致错误：

```mjs
import { readFileSync } from 'node:fs';

// On Windows
readFileSync(new URL('file:///C:/path/%5C'));
readFileSync(new URL('file:///C:/path/%5c'));
/* TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must not include encoded
\ or / characters */
```

#### 缓冲区路径

使用 {Buffer} 指定的路径主要在某些 POSIX 上有用
将文件路径视为不透明字节序列的操作系统。关于这样
系统，单个文件路径可能包含
使用多字符编码。与字符串路径一样，{缓冲区} 路径可以
是相对的或绝对的：

在 POSIX 上使用绝对路径的示例：

```mjs
import { open } from 'node:fs/promises';
import { Buffer } from 'node:buffer';

let fd;
try {
  fd = await open(Buffer.from('/open/some/file.txt'), 'r');
  // Do something with the file
} finally {
  await fd.close();
}
```

#### Windows 上的每驱动器工作目录

在 Windows 上，Node.js遵循每个驱动器工作目录的概念。这
使用不带反斜杠的驱动器路径时，可以观察到行为。为
例`fs.readdirSync('C:\\')`可能返回与以下结果不同的结果
`fs.readdirSync('C:')`.有关详细信息，请参阅
[此 MSDN 页面][MSDN-Rel-Path].

### 文件描述符

在 POSIX 系统上，对于每个进程，内核都维护一个当前
打开文件和资源。每个打开的文件都分配有一个简单的数字
标识符称为*文件描述符*.在系统级别，所有文件系统
操作使用这些文件描述符来标识和跟踪每个特定
文件。Windows 系统使用不同但概念上相似的机制
跟踪资源。为了简化用户，Node.js抽象出
操作系统之间的差异，并为所有打开的文件分配一个数字文件
描述符。

基于回调`fs.open()`和同步`fs.openSync()`方法打开一个
文件并分配一个新的文件描述符。分配后，文件描述符可以
用于从文件读取数据、向其中写入数据或请求有关文件的信息。

操作系统限制可能打开的文件描述符的数量
在任何给定时间，因此在操作时关闭描述符至关重要
已完成。如果不这样做，将导致内存泄漏，从而
最终导致应用程序崩溃。

```mjs
import { open, close, fstat } from 'node:fs';

function closeFd(fd) {
  close(fd, (err) => {
    if (err) throw err;
  });
}

open('/open/some/file.txt', 'r', (err, fd) => {
  if (err) throw err;
  try {
    fstat(fd, (err, stat) => {
      if (err) {
        closeFd(fd);
        throw err;
      }

      // use stat

      closeFd(fd);
    });
  } catch (err) {
    closeFd(fd);
    throw err;
  }
});
```

基于 promise 的 API 使用 {FileHandle} 对象代替数字
文件描述符。这些对象由系统更好地管理，以确保
资源不会泄露。但是，仍然要求它们是
操作完成后关闭：

```mjs
import { open } from 'node:fs/promises';

let file;
try {
  file = await open('/open/some/file.txt', 'r');
  const stat = await file.stat();
  // use stat
} finally {
  await file.close();
}
```

### 线程池使用情况

所有回调和基于承诺的文件系统 API（以下情况除外）
`fs.FSWatcher()`） 使用 libuv 的 threadpool。这可能有令人惊讶和消极的
对某些应用程序的性能影响。查看
[`UV_THREADPOOL_SIZE`][UV_THREADPOOL_SIZE]文档以获取更多信息。

### 文件系统标志

以下标志在`flag`选项需要一个
字符串。

*   `'a'`：打开要追加的文件。
    如果文件不存在，则创建该文件。

*   `'ax'`： 赞`'a'`但如果路径存在，则失败。

*   `'a+'`：打开文件以进行读取和追加。
    如果文件不存在，则创建该文件。

*   `'ax+'`： 赞`'a+'`但如果路径存在，则失败。

*   `'as'`：打开文件以在同步模式下追加。
    如果文件不存在，则创建该文件。

*   `'as+'`：打开文件以在同步模式下读取和追加。
    如果文件不存在，则创建该文件。

*   `'r'`：打开文件进行读取。
    如果该文件不存在，则会发生异常。

*   `'r+'`：打开文件进行读取和写入。
    如果该文件不存在，则会发生异常。

*   `'rs+'`：打开文件以在同步模式下进行读写。指示
    绕过本地文件系统缓存的操作系统。

    这主要用于打开NFS挂载上的文件，因为它允许
    跳过可能过时的本地缓存。它对
    I/O 性能，因此除非需要，否则不建议使用此标志。

    这不会转弯`fs.open()`或`fsPromises.open()`进入同步
    阻止呼叫。如果需要同步操作，则类似于
    `fs.openSync()`应该使用。

*   `'w'`：打开要写入的文件。
    将创建（如果文件不存在）或截断（如果文件存在）。

*   `'wx'`： 赞`'w'`但如果路径存在，则失败。

*   `'w+'`：打开文件进行读取和写入。
    将创建（如果文件不存在）或截断（如果文件存在）。

*   `'wx+'`： 赞`'w+'`但如果路径存在，则失败。

`flag`也可以是 open（2） 记录的数字;常用常量
可从`fs.constants`.在 Windows 上，标志被转换为
在适用的情况下，它们的等效项，例如`O_WRONLY`自`FILE_GENERIC_WRITE`,
或`O_EXCL|O_CREAT`自`CREATE_NEW`，接受者`CreateFileW`.

独家标志`'x'`(`O_EXCL`标志在打开（2）） 导致操作
如果路径已存在，则返回错误。在 POSIX 上，如果路径是符号
链接， 使用`O_EXCL`即使链接指向的路径也返回错误
不存在。独占标志可能不适用于网络文件系统。

在 Linux 上，当文件以追加模式打开时，位置写入不起作用。
内核忽略 position 参数，并始终将数据追加到
文件的末尾。

修改文件而不是替换文件可能需要`flag`选项
设置为`'r+'`而不是默认值`'w'`.

某些标志的行为是特定于平台的。因此，打开目录
在 macOS 和 Linux 上`'a+'`标志，如下面的示例所示，将返回一个
错误。相反，在 Windows 和 FreeBSD 上，一个文件描述符或一个`FileHandle`
将被退回。

```js
// macOS and Linux
fs.open('<directory>', 'a+', (err, fd) => {
  // => [Error: EISDIR: illegal operation on a directory, open <directory>]
});

// Windows and FreeBSD
fs.open('<directory>', 'a+', (err, fd) => {
  // => null, <fd>
});
```

在 Windows 上，使用`'w'`标志（任一
通过`fs.open()`,`fs.writeFile()`或`fsPromises.open()`） 将失败
`EPERM`.可以使用`'r+'`旗。

调用`fs.ftruncate()`或`filehandle.truncate()`可用于复位
文件内容。

[#25741]: https://github.com/nodejs/node/issues/25741

[Common System Errors]: errors.md#common-system-errors

[FS constants]: #fs-constants

[File access constants]: #file-access-constants

[MDN-Date]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date

[MDN-Number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type

[MSDN-Rel-Path]: https://docs.microsoft.com/en-us/windows/desktop/FileIO/naming-a-file#fully-qualified-vs-relative-paths

[MSDN-Using-Streams]: https://docs.microsoft.com/en-us/windows/desktop/FileIO/using-streams

[Naming Files, Paths, and Namespaces]: https://docs.microsoft.com/en-us/windows/desktop/FileIO/naming-a-file

[`AHAFS`]: https://developer.ibm.com/articles/au-aix_event_infrastructure/

[`Buffer.byteLength`]: buffer.md#static-method-bufferbytelengthstring-encoding

[`FSEvents`]: https://developer.apple.com/documentation/coreservices/file_system_events

[`Number.MAX_SAFE_INTEGER`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER

[`ReadDirectoryChangesW`]: https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-readdirectorychangesw

[`UV_THREADPOOL_SIZE`]: cli.md#uv_threadpool_sizesize

[`event ports`]: https://illumos.org/man/port_create

[`filehandle.createWriteStream()`]: #filehandlecreatewritestreamoptions

[`filehandle.writeFile()`]: #filehandlewritefiledata-options

[`fs.access()`]: #fsaccesspath-mode-callback

[`fs.accessSync()`]: #fsaccesssyncpath-mode

[`fs.chmod()`]: #fschmodpath-mode-callback

[`fs.chown()`]: #fschownpath-uid-gid-callback

[`fs.copyFile()`]: #fscopyfilesrc-dest-mode-callback

[`fs.createReadStream()`]: #fscreatereadstreampath-options

[`fs.createWriteStream()`]: #fscreatewritestreampath-options

[`fs.exists()`]: #fsexistspath-callback

[`fs.fstat()`]: #fsfstatfd-options-callback

[`fs.ftruncate()`]: #fsftruncatefd-len-callback

[`fs.futimes()`]: #fsfutimesfd-atime-mtime-callback

[`fs.lstat()`]: #fslstatpath-options-callback

[`fs.lutimes()`]: #fslutimespath-atime-mtime-callback

[`fs.mkdir()`]: #fsmkdirpath-options-callback

[`fs.mkdtemp()`]: #fsmkdtempprefix-options-callback

[`fs.open()`]: #fsopenpath-flags-mode-callback

[`fs.opendir()`]: #fsopendirpath-options-callback

[`fs.opendirSync()`]: #fsopendirsyncpath-options

[`fs.read()`]: #fsreadfd-buffer-offset-length-position-callback

[`fs.readFile()`]: #fsreadfilepath-options-callback

[`fs.readFileSync()`]: #fsreadfilesyncpath-options

[`fs.readdir()`]: #fsreaddirpath-options-callback

[`fs.readdirSync()`]: #fsreaddirsyncpath-options

[`fs.readv()`]: #fsreadvfd-buffers-position-callback

[`fs.realpath()`]: #fsrealpathpath-options-callback

[`fs.rm()`]: #fsrmpath-options-callback

[`fs.rmSync()`]: #fsrmsyncpath-options

[`fs.rmdir()`]: #fsrmdirpath-options-callback

[`fs.stat()`]: #fsstatpath-options-callback

[`fs.symlink()`]: #fssymlinktarget-path-type-callback

[`fs.utimes()`]: #fsutimespath-atime-mtime-callback

[`fs.watch()`]: #fswatchfilename-options-listener

[`fs.write(fd, buffer...)`]: #fswritefd-buffer-offset-length-position-callback

[`fs.write(fd, string...)`]: #fswritefd-string-position-encoding-callback

[`fs.writeFile()`]: #fswritefilefile-data-options-callback

[`fs.writev()`]: #fswritevfd-buffers-position-callback

[`fsPromises.access()`]: #fspromisesaccesspath-mode

[`fsPromises.open()`]: #fspromisesopenpath-flags-mode

[`fsPromises.opendir()`]: #fspromisesopendirpath-options

[`fsPromises.rm()`]: #fspromisesrmpath-options

[`fsPromises.stat()`]: #fspromisesstatpath-options

[`fsPromises.utimes()`]: #fspromisesutimespath-atime-mtime

[`inotify(7)`]: https://man7.org/linux/man-pages/man7/inotify.7.html

[`kqueue(2)`]: https://www.freebsd.org/cgi/man.cgi?query=kqueue&sektion=2

[`util.promisify()`]: util.md#utilpromisifyoriginal

[bigints]: https://tc39.github.io/proposal-bigint

[caveats]: #caveats

[chcp]: https://ss64.com/nt/chcp.html

[inode]: https://en.wikipedia.org/wiki/Inode

[support of file system `flags`]: #file-system-flags
