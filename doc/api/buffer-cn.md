# 缓冲区

<!--introduced_in=v0.1.90-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/buffer.js -->

`Buffer`对象用于表示固定长度的字节序列。多
节点.js接口支持`Buffer`s.

这`Buffer`类是 JavaScript 的子类[`Uint8Array`][Uint8Array]类和
使用涵盖其他用例的方法对其进行扩展。节点.js接口接受
平原[`Uint8Array`][Uint8Array]s 无论位置`Buffer`也支持 。

而`Buffer`类在全局范围内可用，它仍然
建议通过 import 或 require 语句显式引用它。

```mjs
import { Buffer } from 'node:buffer';

// Creates a zero-filled Buffer of length 10.
const buf1 = Buffer.alloc(10);

// Creates a Buffer of length 10,
// filled with bytes which all have the value `1`.
const buf2 = Buffer.alloc(10, 1);

// Creates an uninitialized buffer of length 10.
// This is faster than calling Buffer.alloc() but the returned
// Buffer instance might contain old data that needs to be
// overwritten using fill(), write(), or other functions that fill the Buffer's
// contents.
const buf3 = Buffer.allocUnsafe(10);

// Creates a Buffer containing the bytes [1, 2, 3].
const buf4 = Buffer.from([1, 2, 3]);

// Creates a Buffer containing the bytes [1, 1, 1, 1] – the entries
// are all truncated using `(value & 255)` to fit into the range 0–255.
const buf5 = Buffer.from([257, 257.5, -255, '1']);

// Creates a Buffer containing the UTF-8-encoded bytes for the string 'tést':
// [0x74, 0xc3, 0xa9, 0x73, 0x74] (in hexadecimal notation)
// [116, 195, 169, 115, 116] (in decimal notation)
const buf6 = Buffer.from('tést');

// Creates a Buffer containing the Latin-1 bytes [0x74, 0xe9, 0x73, 0x74].
const buf7 = Buffer.from('tést', 'latin1');
```

```cjs
const { Buffer } = require('node:buffer');

// Creates a zero-filled Buffer of length 10.
const buf1 = Buffer.alloc(10);

// Creates a Buffer of length 10,
// filled with bytes which all have the value `1`.
const buf2 = Buffer.alloc(10, 1);

// Creates an uninitialized buffer of length 10.
// This is faster than calling Buffer.alloc() but the returned
// Buffer instance might contain old data that needs to be
// overwritten using fill(), write(), or other functions that fill the Buffer's
// contents.
const buf3 = Buffer.allocUnsafe(10);

// Creates a Buffer containing the bytes [1, 2, 3].
const buf4 = Buffer.from([1, 2, 3]);

// Creates a Buffer containing the bytes [1, 1, 1, 1] – the entries
// are all truncated using `(value & 255)` to fit into the range 0–255.
const buf5 = Buffer.from([257, 257.5, -255, '1']);

// Creates a Buffer containing the UTF-8-encoded bytes for the string 'tést':
// [0x74, 0xc3, 0xa9, 0x73, 0x74] (in hexadecimal notation)
// [116, 195, 169, 115, 116] (in decimal notation)
const buf6 = Buffer.from('tést');

// Creates a Buffer containing the Latin-1 bytes [0x74, 0xe9, 0x73, 0x74].
const buf7 = Buffer.from('tést', 'latin1');
```

## 缓冲区和字符编码

<!-- YAML
changes:
  - version:
      - v15.7.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/36952
    description: Introduced `base64url` encoding.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7111
    description: Introduced `latin1` as an alias for `binary`.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2859
    description: Removed the deprecated `raw` and `raws` encodings.
-->

在`Buffer`s和字符串，字符编码可以是
指定。如果未指定字符编码，则 UTF-8 将用作
违约。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('hello world', 'utf8');

console.log(buf.toString('hex'));
// Prints: 68656c6c6f20776f726c64
console.log(buf.toString('base64'));
// Prints: aGVsbG8gd29ybGQ=

console.log(Buffer.from('fhqwhgads', 'utf8'));
// Prints: <Buffer 66 68 71 77 68 67 61 64 73>
console.log(Buffer.from('fhqwhgads', 'utf16le'));
// Prints: <Buffer 66 00 68 00 71 00 77 00 68 00 67 00 61 00 64 00 73 00>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('hello world', 'utf8');

console.log(buf.toString('hex'));
// Prints: 68656c6c6f20776f726c64
console.log(buf.toString('base64'));
// Prints: aGVsbG8gd29ybGQ=

console.log(Buffer.from('fhqwhgads', 'utf8'));
// Prints: <Buffer 66 68 71 77 68 67 61 64 73>
console.log(Buffer.from('fhqwhgads', 'utf16le'));
// Prints: <Buffer 66 00 68 00 71 00 77 00 68 00 67 00 61 00 64 00 73 00>
```

Node.js 缓冲区接受编码字符串的所有大小写变体
收到。例如，可以将 UTF-8 指定为`'utf8'`,`'UTF8'`或`'uTf8'`.

Node.js 当前支持的字符编码如下：

*   `'utf8'`（别名：`'utf-8'`）：多字节编码的 Unicode 字符。许多网站
    页面和其他文档格式使用[UTF-8][].这是默认字符
    编码。解码时`Buffer`转换为不独占的字符串
    包含有效的 UTF-8 数据，Unicode 替换字符`U+FFFD`将是
    用于表示这些错误。

*   `'utf16le'`（别名：`'utf-16le'`）：多字节编码的 Unicode 字符。
    与`'utf8'`，字符串中的每个字符将使用 2 之一进行编码
    或 4 个字节。节点.js仅支持[小端序][endianness]的变体
    [UTF-16][].

*   `'latin1'`： 拉丁语-1 代表[国际标准-8859-1][ISO-8859-1].仅此字符编码
    支持来自`U+0000`自`U+00FF`.每个字符都是
    使用单个字节编码。不适合该范围的字符是
    截断，并将映射到该范围内的字符。

转换`Buffer`使用上述之一进入字符串称为
解码，并将字符串转换为`Buffer`称为编码。

Node.js还支持以下二进制到文本编码。为
二进制到文本编码，命名约定颠倒过来：转换
`Buffer`转换为字符串通常称为编码，并将
字符串转换为`Buffer`作为解码。

*   `'base64'`:[基地64][Base64]编码。创建`Buffer`从字符串，
    此编码还将正确接受“URL和文件名安全字母表”作为
    指定于[RFC 4648，第 5 节][RFC 4648, Section 5].空格字符，如空格、
    tab 和 base64 编码字符串中包含的新行将被忽略。

*   `'base64url'`:[base64url][]中指定的编码
    [RFC 4648，第 5 节][RFC 4648, Section 5].创建`Buffer`从字符串中，此
    编码还将正确接受常规 base64 编码的字符串。什么时候
    编码`Buffer`对于字符串，此编码将省略填充。

*   `'hex'`：将每个字节编码为两个十六进制字符。数据截断
    当解码不完全由偶数组成的字符串时可能发生
    十六进制字符数。有关示例，请参见下文。

还支持以下旧字符编码：

*   `'ascii'`：对于 7 位[ASCII][]仅数据。将字符串编码为
    `Buffer`，这相当于使用`'latin1'`.解码时`Buffer`
    转换为字符串，使用此编码将另外取消设置
    解码前的每个字节为`'latin1'`.
    通常，应该没有理由使用此编码，因为`'utf8'`
    （或者，如果已知数据始终是仅 ASCII，`'latin1'`） 将是一个
    编码或解码仅 ASCII 文本时的更好选择。仅提供
    实现传统兼容性。

*   `'binary'`：别名`'latin1'`.看[二进制字符串][binary strings]更多背景信息
    关于这个话题。此编码的名称可能非常具有误导性，因为所有
    此处列出的编码在字符串和二进制数据之间转换。用于转换
    在字符串和之间`Buffer`s，通常`'utf8'`是正确的选择。

*   `'ucs2'`,`'ucs-2'`：别名`'utf16le'`.UCS-2 用于指代变体
    不支持码位大于
    U+FFFF.在 Node.js 中，始终支持这些代码点。

```mjs
import { Buffer } from 'node:buffer';

Buffer.from('1ag123', 'hex');
// Prints <Buffer 1a>, data truncated when first non-hexadecimal value
// ('g') encountered.

Buffer.from('1a7', 'hex');
// Prints <Buffer 1a>, data truncated when data ends in single digit ('7').

Buffer.from('1634', 'hex');
// Prints <Buffer 16 34>, all data represented.
```

```cjs
const { Buffer } = require('node:buffer');

Buffer.from('1ag123', 'hex');
// Prints <Buffer 1a>, data truncated when first non-hexadecimal value
// ('g') encountered.

Buffer.from('1a7', 'hex');
// Prints <Buffer 1a>, data truncated when data ends in single digit ('7').

Buffer.from('1634', 'hex');
// Prints <Buffer 16 34>, all data represented.
```

现代 Web 浏览器遵循[WHATWG 编码标准][WHATWG Encoding Standard]哪些别名
双`'latin1'`和`'ISO-8859-1'`自`'win-1252'`.这意味着在做的同时
类似的东西`http.get()`，如果返回的字符集是
WHATWG规范 服务器可能实际返回
`'win-1252'`-编码数据，并使用`'latin1'`编码可能错误地解码
字符。

## 缓冲区和类型数组

<!-- YAML
changes:
  - version: v3.0.0
    pr-url: https://github.com/nodejs/node/pull/2002
    description: The `Buffer`s class now inherits from `Uint8Array`.
-->

`Buffer`实例也是 JavaScript[`Uint8Array`][Uint8Array]和[`TypedArray`][TypedArray]
实例。都[`TypedArray`][TypedArray]方法在`Buffer`s.有
然而，微妙的不兼容性`Buffer`接口和
[`TypedArray`][TypedArray]应用程序接口。

特别：

*   而[`TypedArray.prototype.slice()`][TypedArray.prototype.slice()]创建部分的副本`TypedArray`,
    [`Buffer.prototype.slice()`][`buf.slice()`]创建对现有视图的视图`Buffer`
    无需复制。这种行为可能令人惊讶，并且仅适用于遗留问题
    兼容性。[`TypedArray.prototype.subarray()`][TypedArray.prototype.subarray()]可用于实现
    的行为[`Buffer.prototype.slice()`][`buf.slice()`]在两者上`Buffer`s
    和其他`TypedArray`s，并且应该是首选。
*   [`buf.toString()`][buf.toString()]与`TypedArray`等效。
*   许多方法，例如[`buf.indexOf()`][buf.indexOf()]，则支持其他参数。

有两种方法可以创建新的[`TypedArray`][TypedArray]实例`Buffer`:

*   通过`Buffer`到[`TypedArray`][TypedArray]构造函数将复制`Buffer`s
    内容，解释为整数数组，而不是字节序列
    的目标类型。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, 2, 3, 4]);
const uint32array = new Uint32Array(buf);

console.log(uint32array);

// Prints: Uint32Array(4) [ 1, 2, 3, 4 ]
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, 2, 3, 4]);
const uint32array = new Uint32Array(buf);

console.log(uint32array);

// Prints: Uint32Array(4) [ 1, 2, 3, 4 ]
```

*   通过`Buffer`s 底层[`ArrayBuffer`][ArrayBuffer]将创建一个
    [`TypedArray`][TypedArray]与`Buffer`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('hello', 'utf16le');
const uint16array = new Uint16Array(
  buf.buffer,
  buf.byteOffset,
  buf.length / Uint16Array.BYTES_PER_ELEMENT);

console.log(uint16array);

// Prints: Uint16Array(5) [ 104, 101, 108, 108, 111 ]
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('hello', 'utf16le');
const uint16array = new Uint16Array(
  buf.buffer,
  buf.byteOffset,
  buf.length / Uint16Array.BYTES_PER_ELEMENT);

console.log(uint16array);

// Prints: Uint16Array(5) [ 104, 101, 108, 108, 111 ]
```

可以创建一个新的`Buffer`共享相同的分配
内存作为[`TypedArray`][TypedArray]实例`TypedArray`对象的
`.buffer`以同样的方式财产。[`Buffer.from()`][`Buffer.from(arrayBuf)`]
行为类似于`new Uint8Array()`在这种情况下。

```mjs
import { Buffer } from 'node:buffer';

const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Copies the contents of `arr`.
const buf1 = Buffer.from(arr);

// Shares memory with `arr`.
const buf2 = Buffer.from(arr.buffer);

console.log(buf1);
// Prints: <Buffer 88 a0>
console.log(buf2);
// Prints: <Buffer 88 13 a0 0f>

arr[1] = 6000;

console.log(buf1);
// Prints: <Buffer 88 a0>
console.log(buf2);
// Prints: <Buffer 88 13 70 17>
```

```cjs
const { Buffer } = require('node:buffer');

const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Copies the contents of `arr`.
const buf1 = Buffer.from(arr);

// Shares memory with `arr`.
const buf2 = Buffer.from(arr.buffer);

console.log(buf1);
// Prints: <Buffer 88 a0>
console.log(buf2);
// Prints: <Buffer 88 13 a0 0f>

arr[1] = 6000;

console.log(buf1);
// Prints: <Buffer 88 a0>
console.log(buf2);
// Prints: <Buffer 88 13 70 17>
```

创建`Buffer`使用[`TypedArray`][TypedArray]的`.buffer`是的
可能仅使用底层的一部分[`ArrayBuffer`][ArrayBuffer]通过传递
`byteOffset`和`length`参数。

```mjs
import { Buffer } from 'node:buffer';

const arr = new Uint16Array(20);
const buf = Buffer.from(arr.buffer, 0, 16);

console.log(buf.length);
// Prints: 16
```

```cjs
const { Buffer } = require('node:buffer');

const arr = new Uint16Array(20);
const buf = Buffer.from(arr.buffer, 0, 16);

console.log(buf.length);
// Prints: 16
```

这`Buffer.from()`和[`TypedArray.from()`][TypedArray.from()]具有不同的签名和
实现。具体来说，[`TypedArray`][TypedArray]变体接受第二个
参数，它是在 的每个元素上调用的映射函数
类型化数组：

*   `TypedArray.from(source[, mapFn[, thisArg]])`

这`Buffer.from()`但是，方法不支持使用映射
功能：

*   [`Buffer.from(array)`][Buffer.from(array)]
*   [`Buffer.from(buffer)`][Buffer.from(buffer)]
*   [`Buffer.from(arrayBuffer[, byteOffset[, length]])`][`Buffer.from(arrayBuf)`]
*   [`Buffer.from(string[, encoding])`][`Buffer.from(string)`]

## 缓冲区和迭代

`Buffer`实例可以迭代使用`for..of`语法：

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, 2, 3]);

for (const b of buf) {
  console.log(b);
}
// Prints:
//   1
//   2
//   3
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, 2, 3]);

for (const b of buf) {
  console.log(b);
}
// Prints:
//   1
//   2
//   3
```

此外，[`buf.values()`][buf.values()],[`buf.keys()`][buf.keys()]和
[`buf.entries()`][buf.entries()]方法可用于创建迭代器。

## 类：`Blob`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41270
    description: No longer experimental.
-->

一个[`Blob`][Blob]封装了可以安全共享的不可变原始数据
多个工作线程。

### `new buffer.Blob([sources[, options]])`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
changes:
  - version: v16.7.0
    pr-url: https://github.com/nodejs/node/pull/39708
    description: Added the standard `endings` option to replace line-endings,
                 and removed the non-standard `encoding` option.
-->

*   `sources`{字符串\[]|ArrayBuffer\[]|TypedArray\[]|数据视图\[]|Blob\[]} An
    数组的字符串， {ArrayBuffer}， {TypedArray}， {DataView}， or {Blob} 对象，
    或此类对象的任何组合，这些对象将存储在`Blob`.
*   `options`{对象}
    *   `endings`{字符串}其中之一`'transparent'`或`'native'`.设置时
        自`'native'`，字符串源部分中的行尾将转换为
        由 指定的平台本机行结束`require('node:os').EOL`.
    *   `type`{字符串}Blob 内容类型。目的是`type`输送
        数据的 MIME 媒体类型，但不验证类型格式
        执行。

创建新的`Blob`对象包含给定源的串联。

{ArrayBuffer}、{TypedArray}、{DataView}和 {Buffer} 源被复制到
“Blob”，因此可以在创建“Blob”后安全地进行修改。

字符串源被编码为 UTF-8 字节序列，并复制到 Blob 中。
每个字符串部分中不匹配的代理项对将被 Unicode 替换
U+FFFD 替换字符。

### `blob.arrayBuffer()`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
-->

*   返回： {承诺}

返回一个承诺，该承诺通过 {ArrayBuffer} 实现，该 {ArrayBuffer} 包含
这`Blob`数据。

### `blob.size`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
-->

的总大小`Blob`以字节为单位。

### `blob.slice([start[, end[, type]]])`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
-->

*   `start`{数字}起始索引。
*   `end`{数字}结束索引。
*   `type`{字符串}新内容类型`Blob`

创建并返回新的`Blob`包含此的子集`Blob`对象
数据。原创`Blob`不更改。

### `blob.stream()`

<!-- YAML
added: v16.7.0
-->

*   返回：{可读流}

返回新的`ReadableStream`允许的内容`Blob`以供阅读。

### `blob.text()`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
-->

*   返回： {承诺}

返回一个承诺，该承诺与`Blob`解码为
UTF-8 字符串。

### `blob.type`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
-->

*   类型： {字符串}

的内容类型`Blob`.

### `Blob`对象和`MessageChannel`

创建 {Blob} 对象后，可以通过以下方式发送`MessagePort`到多个
目标，而无需传输或立即复制数据。数据
包含在`Blob`仅当`arrayBuffer()`或`text()`
调用方法。

```mjs
import { Blob, Buffer } from 'node:buffer';
import { setTimeout as delay } from 'node:timers/promises';

const blob = new Blob(['hello there']);

const mc1 = new MessageChannel();
const mc2 = new MessageChannel();

mc1.port1.onmessage = async ({ data }) => {
  console.log(await data.arrayBuffer());
  mc1.port1.close();
};

mc2.port1.onmessage = async ({ data }) => {
  await delay(1000);
  console.log(await data.arrayBuffer());
  mc2.port1.close();
};

mc1.port2.postMessage(blob);
mc2.port2.postMessage(blob);

// The Blob is still usable after posting.
blob.text().then(console.log);
```

```cjs
const { Blob, Buffer } = require('node:buffer');
const { setTimeout: delay } = require('node:timers/promises');

const blob = new Blob(['hello there']);

const mc1 = new MessageChannel();
const mc2 = new MessageChannel();

mc1.port1.onmessage = async ({ data }) => {
  console.log(await data.arrayBuffer());
  mc1.port1.close();
};

mc2.port1.onmessage = async ({ data }) => {
  await delay(1000);
  console.log(await data.arrayBuffer());
  mc2.port1.close();
};

mc1.port2.postMessage(blob);
mc2.port2.postMessage(blob);

// The Blob is still usable after posting.
blob.text().then(console.log);
```

## 类：`Buffer`

这`Buffer`class 是用于直接处理二进制数据的全局类型。
它可以通过多种方式构建。

### 静态方法：`Buffer.alloc(size[, fill[, encoding]])`

<!-- YAML
added: v5.10.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/34682
    description: Throw ERR_INVALID_ARG_VALUE instead of ERR_INVALID_OPT_VALUE
                 for invalid input arguments.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18129
    description: Attempting to fill a non-zero length buffer with a zero length
                 buffer triggers a thrown exception.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17427
    description: Specifying an invalid string for `fill` triggers a thrown
                 exception.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17428
    description: Specifying an invalid string for `fill` now results in a
                 zero-filled buffer.
-->

*   `size`{整数}新品的所需长度`Buffer`.
*   `fill`{字符串|缓冲区|Uint8Array|integer} 用于预填充新`Buffer`
    跟。**违约：** `0`.
*   `encoding`{字符串}如果`fill`是一个字符串，这是它的编码。
    **违约：** `'utf8'`.

分配新的`Buffer`之`size`字节。如果`fill`是`undefined`这
`Buffer`将为零填充。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.alloc(5);

console.log(buf);
// Prints: <Buffer 00 00 00 00 00>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.alloc(5);

console.log(buf);
// Prints: <Buffer 00 00 00 00 00>
```

如果`size`大于
[`buffer.constants.MAX_LENGTH`][buffer.constants.MAX_LENGTH]或小于 0，[`ERR_INVALID_ARG_VALUE`][ERR_INVALID_ARG_VALUE]
被抛出。

如果`fill`指定，分配`Buffer`将通过调用来初始化
[`buf.fill(fill)`][`buf.fill()`].

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.alloc(5, 'a');

console.log(buf);
// Prints: <Buffer 61 61 61 61 61>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.alloc(5, 'a');

console.log(buf);
// Prints: <Buffer 61 61 61 61 61>
```

如果两者兼而有之`fill`和`encoding`指定，分配`Buffer`将是
通过调用进行初始化[`buf.fill(fill, encoding)`][`buf.fill()`].

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

console.log(buf);
// Prints: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

console.log(buf);
// Prints: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

叫[`Buffer.alloc()`][Buffer.alloc()]可能比替代方案慢得多
[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]但确保新创建的`Buffer`实例
内容永远不会包含以前分配的敏感数据，包括
可能尚未分配的数据`Buffer`s.

一个`TypeError`将在以下情况下被抛出`size`不是一个数字。

### 静态方法：`Buffer.allocUnsafe(size)`

<!-- YAML
added: v5.10.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/34682
    description: Throw ERR_INVALID_ARG_VALUE instead of ERR_INVALID_OPT_VALUE
                 for invalid input arguments.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7079
    description: Passing a negative `size` will now throw an error.
-->

*   `size`{整数}新品的所需长度`Buffer`.

分配新的`Buffer`之`size`字节。如果`size`大于
[`buffer.constants.MAX_LENGTH`][buffer.constants.MAX_LENGTH]或小于 0，[`ERR_INVALID_ARG_VALUE`][ERR_INVALID_ARG_VALUE]
被抛出。

底层内存`Buffer`以这种方式创建的实例是*不
初始 化*.新创建的内容`Buffer`未知，并且
*可能包含敏感数据*.用[`Buffer.alloc()`][Buffer.alloc()]而是初始化
`Buffer`带零的实例。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(10);

console.log(buf);
// Prints (contents may vary): <Buffer a0 8b 28 3f 01 00 00 00 50 32>

buf.fill(0);

console.log(buf);
// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(10);

console.log(buf);
// Prints (contents may vary): <Buffer a0 8b 28 3f 01 00 00 00 50 32>

buf.fill(0);

console.log(buf);
// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
```

一个`TypeError`将在以下情况下被抛出`size`不是一个数字。

这`Buffer`模块预分配内部`Buffer`的实例
大小[`Buffer.poolSize`][Buffer.poolSize]用作快速分配新品的池
`Buffer`使用 创建的实例[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()],
[`Buffer.from(array)`][Buffer.from(array)],[`Buffer.concat()`][Buffer.concat()]和已弃用的
`new Buffer(size)`构造函数仅在以下情况下`size`小于或等于
自`Buffer.poolSize >> 1`（地板[`Buffer.poolSize`][Buffer.poolSize]除以 2）。

使用这种预先分配的内部存储器池是两者之间的关键区别
叫`Buffer.alloc(size, fill)`与。`Buffer.allocUnsafe(size).fill(fill)`.
具体说来`Buffer.alloc(size, fill)`将*从不*使用内部`Buffer`
游泳池，同时`Buffer.allocUnsafe(size).fill(fill)` *将*使用内部
`Buffer`池如果`size`小于或等于一半[`Buffer.poolSize`][Buffer.poolSize].这
差异是微妙的，但当应用程序需要
额外的性能[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]提供。

### 静态方法：`Buffer.allocUnsafeSlow(size)`

<!-- YAML
added: v5.12.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/34682
    description: Throw ERR_INVALID_ARG_VALUE instead of ERR_INVALID_OPT_VALUE
                 for invalid input arguments.
-->

*   `size`{整数}新品的所需长度`Buffer`.

分配新的`Buffer`之`size`字节。如果`size`大于
[`buffer.constants.MAX_LENGTH`][buffer.constants.MAX_LENGTH]或小于 0，[`ERR_INVALID_ARG_VALUE`][ERR_INVALID_ARG_VALUE]
被抛出。零长度`Buffer`在以下情况下创建：`size`为 0。

底层内存`Buffer`以这种方式创建的实例是*不
初始 化*.新创建的内容`Buffer`未知，并且
*可能包含敏感数据*.用[`buf.fill(0)`][`buf.fill()`]初始化
这样`Buffer`带零的实例。

使用时[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]以分配新的`Buffer`实例
4 KiB 以下的分配从单个预分配的分配中切片`Buffer`.这
允许应用程序避免创建许多垃圾回收开销
单独分配`Buffer`实例。这种方法可以同时改善两者
性能和内存使用，无需跟踪和清理
许多人`ArrayBuffer`对象。

但是，在开发人员可能需要保留一小部分的情况下
内存从池中停留不确定的时间量，它可能是合适的
以创建非池化`Buffer`实例使用`Buffer.allocUnsafeSlow()`和
然后复制出相关位。

```mjs
import { Buffer } from 'node:buffer';

// Need to keep around a few small chunks of memory.
const store = [];

socket.on('readable', () => {
  let data;
  while (null !== (data = readable.read())) {
    // Allocate for retained data.
    const sb = Buffer.allocUnsafeSlow(10);

    // Copy the data into the new allocation.
    data.copy(sb, 0, 0, 10);

    store.push(sb);
  }
});
```

```cjs
const { Buffer } = require('node:buffer');

// Need to keep around a few small chunks of memory.
const store = [];

socket.on('readable', () => {
  let data;
  while (null !== (data = readable.read())) {
    // Allocate for retained data.
    const sb = Buffer.allocUnsafeSlow(10);

    // Copy the data into the new allocation.
    data.copy(sb, 0, 0, 10);

    store.push(sb);
  }
});
```

一个`TypeError`将在以下情况下被抛出`size`不是一个数字。

### 静态方法：`Buffer.byteLength(string[, encoding])`

<!-- YAML
added: v0.1.90
changes:
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8946
    description: Passing invalid input will now throw an error.
  - version: v5.10.0
    pr-url: https://github.com/nodejs/node/pull/5255
    description: The `string` parameter can now be any `TypedArray`, `DataView`
                 or `ArrayBuffer`.
-->

*   `string`{字符串|缓冲区|TypedArray|数据视图|ArrayBuffer|SharedArrayBuffer} A
    值来计算长度。
*   `encoding`{字符串}如果`string`是一个字符串，这是它的编码。
    **违约：** `'utf8'`.
*   返回：{整数} 中包含的字节数`string`.

使用`encoding`.
这与[`String.prototype.length`][String.prototype.length]，其中不考虑
用于将字符串转换为字节的编码。

为`'base64'`,`'base64url'`和`'hex'`，则此函数假定输入有效。
对于包含非 base64/十六进制编码数据（例如空格）的字符串，
返回值可能大于的长度`Buffer`创建自
字符串。

```mjs
import { Buffer } from 'node:buffer';

const str = '\u00bd + \u00bc = \u00be';

console.log(`${str}: ${str.length} characters, ` +
            `${Buffer.byteLength(str, 'utf8')} bytes`);
// Prints: ½ + ¼ = ¾: 9 characters, 12 bytes
```

```cjs
const { Buffer } = require('node:buffer');

const str = '\u00bd + \u00bc = \u00be';

console.log(`${str}: ${str.length} characters, ` +
            `${Buffer.byteLength(str, 'utf8')} bytes`);
// Prints: ½ + ¼ = ¾: 9 characters, 12 bytes
```

什么时候`string`是一个`Buffer`/[`DataView`][DataView]/[`TypedArray`][TypedArray]/[`ArrayBuffer`][ArrayBuffer]/
[`SharedArrayBuffer`][SharedArrayBuffer]，则报告的字节长度`.byteLength`
返回。

### 静态方法：`Buffer.compare(buf1, buf2)`

<!-- YAML
added: v0.11.13
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The arguments can now be `Uint8Array`s.
-->

*   `buf1`{缓冲区|Uint8Array}
*   `buf2`{缓冲区|Uint8Array}
*   返回：{整数} 任一`-1`,`0`或`1`，具体取决于
    比较。看[`buf.compare()`][buf.compare()]了解详情。

比较`buf1`自`buf2`，通常用于对数组进行排序
`Buffer`实例。这相当于调用
[`buf1.compare(buf2)`][`buf.compare()`].

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from('1234');
const buf2 = Buffer.from('0123');
const arr = [buf1, buf2];

console.log(arr.sort(Buffer.compare));
// Prints: [ <Buffer 30 31 32 33>, <Buffer 31 32 33 34> ]
// (This result is equal to: [buf2, buf1].)
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from('1234');
const buf2 = Buffer.from('0123');
const arr = [buf1, buf2];

console.log(arr.sort(Buffer.compare));
// Prints: [ <Buffer 30 31 32 33>, <Buffer 31 32 33 34> ]
// (This result is equal to: [buf2, buf1].)
```

### 静态方法：`Buffer.concat(list[, totalLength])`

<!-- YAML
added: v0.7.11
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The elements of `list` can now be `Uint8Array`s.
-->

*   `list`{缓冲器\[] |Uint8Array\[]} 列表`Buffer`或[`Uint8Array`][Uint8Array]
    要连接的实例。
*   `totalLength`{整数}总长度`Buffer`中的实例`list`
    当串联时。
*   返回：{缓冲区}

返回新的`Buffer`这是连接所有`Buffer`
中的实例`list`一起。

如果列表中没有项目，或者如果`totalLength`为 0，则为新的零长度
`Buffer`返回。

如果`totalLength`未提供，它是根据`Buffer`实例
在`list`通过添加它们的长度。

如果`totalLength`，则强制为无符号整数。如果
组合长度`Buffer`s 在`list`超过`totalLength`，结果是
截断为`totalLength`.

```mjs
import { Buffer } from 'node:buffer';

// Create a single `Buffer` from a list of three `Buffer` instances.

const buf1 = Buffer.alloc(10);
const buf2 = Buffer.alloc(14);
const buf3 = Buffer.alloc(18);
const totalLength = buf1.length + buf2.length + buf3.length;

console.log(totalLength);
// Prints: 42

const bufA = Buffer.concat([buf1, buf2, buf3], totalLength);

console.log(bufA);
// Prints: <Buffer 00 00 00 00 ...>
console.log(bufA.length);
// Prints: 42
```

```cjs
const { Buffer } = require('node:buffer');

// Create a single `Buffer` from a list of three `Buffer` instances.

const buf1 = Buffer.alloc(10);
const buf2 = Buffer.alloc(14);
const buf3 = Buffer.alloc(18);
const totalLength = buf1.length + buf2.length + buf3.length;

console.log(totalLength);
// Prints: 42

const bufA = Buffer.concat([buf1, buf2, buf3], totalLength);

console.log(bufA);
// Prints: <Buffer 00 00 00 00 ...>
console.log(bufA.length);
// Prints: 42
```

`Buffer.concat()`也可使用内部`Buffer`泳池喜欢
[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]确实如此。

### 静态方法：`Buffer.from(array)`

<!-- YAML
added: v5.10.0
-->

*   `array`{整数\[]}

分配新的`Buffer`使用`array`范围内的字节数`0`–`255`.
超出该范围的数组条目将被截断以适合它。

```mjs
import { Buffer } from 'node:buffer';

// Creates a new Buffer containing the UTF-8 bytes of the string 'buffer'.
const buf = Buffer.from([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

```cjs
const { Buffer } = require('node:buffer');

// Creates a new Buffer containing the UTF-8 bytes of the string 'buffer'.
const buf = Buffer.from([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

一个`TypeError`将在以下情况下被抛出`array`不是`Array`或其他类型
适用于`Buffer.from()`变种。

`Buffer.from(array)`和[`Buffer.from(string)`][Buffer.from(string)]也可使用内部
`Buffer`泳池喜欢[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]确实如此。

### 静态方法：`Buffer.from(arrayBuffer[, byteOffset[, length]])`

<!-- YAML
added: v5.10.0
-->

*   `arrayBuffer`{ArrayBuffer|SharedArrayBuffer} An[`ArrayBuffer`][ArrayBuffer],
    [`SharedArrayBuffer`][SharedArrayBuffer]，例如`.buffer`的属性
    [`TypedArray`][TypedArray].
*   `byteOffset`{整数}要公开的第一个字节的索引。**违约：** `0`.
*   `length`{整数}要公开的字节数。
    **违约：** `arrayBuffer.byteLength - byteOffset`.

这将创建[`ArrayBuffer`][ArrayBuffer]无需复制底层
记忆。例如，当传递对`.buffer`的属性
[`TypedArray`][TypedArray]实例，新创建的`Buffer`将共享相同
分配的内存作为[`TypedArray`][TypedArray]的底层`ArrayBuffer`.

```mjs
import { Buffer } from 'node:buffer';

const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`.
const buf = Buffer.from(arr.buffer);

console.log(buf);
// Prints: <Buffer 88 13 a0 0f>

// Changing the original Uint16Array changes the Buffer also.
arr[1] = 6000;

console.log(buf);
// Prints: <Buffer 88 13 70 17>
```

```cjs
const { Buffer } = require('node:buffer');

const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`.
const buf = Buffer.from(arr.buffer);

console.log(buf);
// Prints: <Buffer 88 13 a0 0f>

// Changing the original Uint16Array changes the Buffer also.
arr[1] = 6000;

console.log(buf);
// Prints: <Buffer 88 13 70 17>
```

可选`byteOffset`和`length`参数指定内存范围
这`arrayBuffer`将由`Buffer`.

```mjs
import { Buffer } from 'node:buffer';

const ab = new ArrayBuffer(10);
const buf = Buffer.from(ab, 0, 2);

console.log(buf.length);
// Prints: 2
```

```cjs
const { Buffer } = require('node:buffer');

const ab = new ArrayBuffer(10);
const buf = Buffer.from(ab, 0, 2);

console.log(buf.length);
// Prints: 2
```

一个`TypeError`将在以下情况下被抛出`arrayBuffer`不是[`ArrayBuffer`][ArrayBuffer]或
[`SharedArrayBuffer`][SharedArrayBuffer]或适用于`Buffer.from()`
变种。

重要的是要记住，支持`ArrayBuffer`可以覆盖一个范围
超出 a 边界的内存`TypedArray`视图。一个新的
`Buffer`使用`buffer`的属性`TypedArray`可延长
超出范围`TypedArray`:

```mjs
import { Buffer } from 'node:buffer';

const arrA = Uint8Array.from([0x63, 0x64, 0x65, 0x66]); // 4 elements
const arrB = new Uint8Array(arrA.buffer, 1, 2); // 2 elements
console.log(arrA.buffer === arrB.buffer); // true

const buf = Buffer.from(arrB.buffer);
console.log(buf);
// Prints: <Buffer 63 64 65 66>
```

```cjs
const { Buffer } = require('node:buffer');

const arrA = Uint8Array.from([0x63, 0x64, 0x65, 0x66]); // 4 elements
const arrB = new Uint8Array(arrA.buffer, 1, 2); // 2 elements
console.log(arrA.buffer === arrB.buffer); // true

const buf = Buffer.from(arrB.buffer);
console.log(buf);
// Prints: <Buffer 63 64 65 66>
```

### 静态方法：`Buffer.from(buffer)`

<!-- YAML
added: v5.10.0
-->

*   `buffer`{缓冲区|Uint8Array} An existing`Buffer`或[`Uint8Array`][Uint8Array]从
    要复制数据。

复制通过的`buffer`数据到新的`Buffer`实例。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from('buffer');
const buf2 = Buffer.from(buf1);

buf1[0] = 0x61;

console.log(buf1.toString());
// Prints: auffer
console.log(buf2.toString());
// Prints: buffer
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from('buffer');
const buf2 = Buffer.from(buf1);

buf1[0] = 0x61;

console.log(buf1.toString());
// Prints: auffer
console.log(buf2.toString());
// Prints: buffer
```

一个`TypeError`将在以下情况下被抛出`buffer`不是`Buffer`或其他类型
适用于`Buffer.from()`变种。

### 静态方法：`Buffer.from(object[, offsetOrEncoding[, length]])`

<!-- YAML
added: v8.2.0
-->

*   `object`{对象}支持的对象`Symbol.toPrimitive`或`valueOf()`.
*   `offsetOrEncoding`{整数|字符串}字节偏移量或编码。
*   `length`{整数}长度。

对于`valueOf()`函数返回一个不严格等于
`object`返回`Buffer.from(object.valueOf(), offsetOrEncoding, length)`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from(new String('this is a test'));
// Prints: <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from(new String('this is a test'));
// Prints: <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

对于支持的对象`Symbol.toPrimitive`返回
`Buffer.from(object[Symbol.toPrimitive]('string'), offsetOrEncoding)`.

```mjs
import { Buffer } from 'node:buffer';

class Foo {
  [Symbol.toPrimitive]() {
    return 'this is a test';
  }
}

const buf = Buffer.from(new Foo(), 'utf8');
// Prints: <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

```cjs
const { Buffer } = require('node:buffer');

class Foo {
  [Symbol.toPrimitive]() {
    return 'this is a test';
  }
}

const buf = Buffer.from(new Foo(), 'utf8');
// Prints: <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

一个`TypeError`将在以下情况下被抛出`object`没有上述方法或
不属于适合`Buffer.from()`变种。

### 静态方法：`Buffer.from(string[, encoding])`

<!-- YAML
added: v5.10.0
-->

*   `string`{字符串}要编码的字符串。
*   `encoding`{字符串}的编码`string`.**违约：** `'utf8'`.

创建新的`Buffer`含`string`.这`encoding`参数标识
转换时要使用的字符编码`string`转换为字节。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from('this is a tést');
const buf2 = Buffer.from('7468697320697320612074c3a97374', 'hex');

console.log(buf1.toString());
// Prints: this is a tést
console.log(buf2.toString());
// Prints: this is a tést
console.log(buf1.toString('latin1'));
// Prints: this is a tÃ©st
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from('this is a tést');
const buf2 = Buffer.from('7468697320697320612074c3a97374', 'hex');

console.log(buf1.toString());
// Prints: this is a tést
console.log(buf2.toString());
// Prints: this is a tést
console.log(buf1.toString('latin1'));
// Prints: this is a tÃ©st
```

一个`TypeError`将在以下情况下被抛出`string`不是字符串或其他类型
适用于`Buffer.from()`变种。

### 静态方法：`Buffer.isBuffer(obj)`

<!-- YAML
added: v0.1.101
-->

*   `obj`{对象}
*   返回：{布尔值}

返回`true`如果`obj`是一个`Buffer`,`false`否则。

```mjs
import { Buffer } from 'node:buffer';

Buffer.isBuffer(Buffer.alloc(10)); // true
Buffer.isBuffer(Buffer.from('foo')); // true
Buffer.isBuffer('a string'); // false
Buffer.isBuffer([]); // false
Buffer.isBuffer(new Uint8Array(1024)); // false
```

```cjs
const { Buffer } = require('node:buffer');

Buffer.isBuffer(Buffer.alloc(10)); // true
Buffer.isBuffer(Buffer.from('foo')); // true
Buffer.isBuffer('a string'); // false
Buffer.isBuffer([]); // false
Buffer.isBuffer(new Uint8Array(1024)); // false
```

### 静态方法：`Buffer.isEncoding(encoding)`

<!-- YAML
added: v0.9.1
-->

*   `encoding`{字符串}要检查的字符编码名称。
*   返回：{布尔值}

返回`true`如果`encoding`是受支持的字符编码的名称，
或`false`否则。

```mjs
import { Buffer } from 'node:buffer';

console.log(Buffer.isEncoding('utf8'));
// Prints: true

console.log(Buffer.isEncoding('hex'));
// Prints: true

console.log(Buffer.isEncoding('utf/8'));
// Prints: false

console.log(Buffer.isEncoding(''));
// Prints: false
```

```cjs
const { Buffer } = require('node:buffer');

console.log(Buffer.isEncoding('utf8'));
// Prints: true

console.log(Buffer.isEncoding('hex'));
// Prints: true

console.log(Buffer.isEncoding('utf/8'));
// Prints: false

console.log(Buffer.isEncoding(''));
// Prints: false
```

### 类属性：`Buffer.poolSize`

<!-- YAML
added: v0.11.3
-->

*   {整数}**违约：** `8192`

这是预分配的内部大小（以字节为单位）`Buffer`使用的实例
用于泳池。可以修改此值。

### `buf[index]`

*   `index`{整数}

索引运算符`[index]`可用于获取和设置八位字节的位置
`index`在`buf`.这些值是指单个字节，因此法定值
范围介于`0x00`和`0xFF`（十六进制）或`0`和`255`（十进制）。

此运算符继承自`Uint8Array`，因此它在越界时的行为
访问与相同`Uint8Array`.换句话说，`buf[index]`返回
`undefined`什么时候`index`为负数或大于或等于`buf.length`和
`buf[index] = value`在以下情况下不修改缓冲区：`index`为负数或
`>= buf.length`.

```mjs
import { Buffer } from 'node:buffer';

// Copy an ASCII string into a `Buffer` one byte at a time.
// (This only works for ASCII-only strings. In general, one should use
// `Buffer.from()` to perform this conversion.)

const str = 'Node.js';
const buf = Buffer.allocUnsafe(str.length);

for (let i = 0; i < str.length; i++) {
  buf[i] = str.charCodeAt(i);
}

console.log(buf.toString('utf8'));
// Prints: Node.js
```

```cjs
const { Buffer } = require('node:buffer');

// Copy an ASCII string into a `Buffer` one byte at a time.
// (This only works for ASCII-only strings. In general, one should use
// `Buffer.from()` to perform this conversion.)

const str = 'Node.js';
const buf = Buffer.allocUnsafe(str.length);

for (let i = 0; i < str.length; i++) {
  buf[i] = str.charCodeAt(i);
}

console.log(buf.toString('utf8'));
// Prints: Node.js
```

### `buf.buffer`

*   {ArrayBuffer}基础`ArrayBuffer`对象基于此`Buffer`
    对象已创建。

这`ArrayBuffer`不保证与原件完全对应
`Buffer`.请参阅上的注释`buf.byteOffset`了解详情。

```mjs
import { Buffer } from 'node:buffer';

const arrayBuffer = new ArrayBuffer(16);
const buffer = Buffer.from(arrayBuffer);

console.log(buffer.buffer === arrayBuffer);
// Prints: true
```

```cjs
const { Buffer } = require('node:buffer');

const arrayBuffer = new ArrayBuffer(16);
const buffer = Buffer.from(arrayBuffer);

console.log(buffer.buffer === arrayBuffer);
// Prints: true
```

### `buf.byteOffset`

*   {整数}这`byteOffset`的`Buffer`s 底层`ArrayBuffer`对象。

设置时`byteOffset`在`Buffer.from(ArrayBuffer, byteOffset, length)`,
或者有时在分配`Buffer`小于`Buffer.poolSize`这
缓冲区不从底层的零偏移量开始`ArrayBuffer`.

这可能会在访问底层计算机时出现问题`ArrayBuffer`径直
用`buf.buffer`，作为其他部分的`ArrayBuffer`可能无关
到`Buffer`对象本身。

创建`TypedArray`与 共享其内存的对象
一个`Buffer`在这种情况下，需要指定`byteOffset`正确：

```mjs
import { Buffer } from 'node:buffer';

// Create a buffer smaller than `Buffer.poolSize`.
const nodeBuffer = Buffer.from([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

// When casting the Node.js Buffer to an Int8Array, use the byteOffset
// to refer only to the part of `nodeBuffer.buffer` that contains the memory
// for `nodeBuffer`.
new Int8Array(nodeBuffer.buffer, nodeBuffer.byteOffset, nodeBuffer.length);
```

```cjs
const { Buffer } = require('node:buffer');

// Create a buffer smaller than `Buffer.poolSize`.
const nodeBuffer = Buffer.from([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

// When casting the Node.js Buffer to an Int8Array, use the byteOffset
// to refer only to the part of `nodeBuffer.buffer` that contains the memory
// for `nodeBuffer`.
new Int8Array(nodeBuffer.buffer, nodeBuffer.byteOffset, nodeBuffer.length);
```

### `buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]])`

<!-- YAML
added: v0.11.13
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `target` parameter can now be a `Uint8Array`.
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/5880
    description: Additional parameters for specifying offsets are supported now.
-->

*   `target`{缓冲区|Uint8Array} A`Buffer`或[`Uint8Array`][Uint8Array]用它
    比较`buf`.
*   `targetStart`{整数}内的偏移量`target`从哪里开始
    比较。**违约：** `0`.
*   `targetEnd`{整数}内的偏移量`target`从哪里结束比较
    （不包括在内）。**违约：** `target.length`.
*   `sourceStart`{整数}内的偏移量`buf`从哪里开始比较。
    **违约：** `0`.
*   `sourceEnd`{整数}内的偏移量`buf`从哪里结束比较
    （不包括在内）。**违约：** [`buf.length`][buf.length].
*   返回：{整数}

比较`buf`跟`target`并返回一个数字，指示是否`buf`
之前、之后或与`target`按排序顺序。
比较基于每个字节的实际序列`Buffer`.

*   `0`在以下情况下返回`target`与 相同`buf`
*   `1`在以下情况下返回`target`应该来*以前* `buf`排序时。
*   `-1`在以下情况下返回`target`应该来*后* `buf`排序时。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('BCD');
const buf3 = Buffer.from('ABCD');

console.log(buf1.compare(buf1));
// Prints: 0
console.log(buf1.compare(buf2));
// Prints: -1
console.log(buf1.compare(buf3));
// Prints: -1
console.log(buf2.compare(buf1));
// Prints: 1
console.log(buf2.compare(buf3));
// Prints: 1
console.log([buf1, buf2, buf3].sort(Buffer.compare));
// Prints: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
// (This result is equal to: [buf1, buf3, buf2].)
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('BCD');
const buf3 = Buffer.from('ABCD');

console.log(buf1.compare(buf1));
// Prints: 0
console.log(buf1.compare(buf2));
// Prints: -1
console.log(buf1.compare(buf3));
// Prints: -1
console.log(buf2.compare(buf1));
// Prints: 1
console.log(buf2.compare(buf3));
// Prints: 1
console.log([buf1, buf2, buf3].sort(Buffer.compare));
// Prints: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
// (This result is equal to: [buf1, buf3, buf2].)
```

可选`targetStart`,`targetEnd`,`sourceStart`和`sourceEnd`
参数可用于将比较限制为内的特定范围`target`
和`buf`分别。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const buf2 = Buffer.from([5, 6, 7, 8, 9, 1, 2, 3, 4]);

console.log(buf1.compare(buf2, 5, 9, 0, 4));
// Prints: 0
console.log(buf1.compare(buf2, 0, 6, 4));
// Prints: -1
console.log(buf1.compare(buf2, 5, 6, 5));
// Prints: 1
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const buf2 = Buffer.from([5, 6, 7, 8, 9, 1, 2, 3, 4]);

console.log(buf1.compare(buf2, 5, 9, 0, 4));
// Prints: 0
console.log(buf1.compare(buf2, 0, 6, 4));
// Prints: -1
console.log(buf1.compare(buf2, 5, 6, 5));
// Prints: 1
```

[`ERR_OUT_OF_RANGE`][ERR_OUT_OF_RANGE]在以下情况下被抛出`targetStart < 0`,`sourceStart < 0`,
`targetEnd > target.byteLength`或`sourceEnd > source.byteLength`.

### `buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])`

<!-- YAML
added: v0.1.90
-->

*   `target`{缓冲区|Uint8Array} A`Buffer`或[`Uint8Array`][Uint8Array]以复制到。
*   `targetStart`{整数}内的偏移量`target`从哪里开始
    写作。**违约：** `0`.
*   `sourceStart`{整数}内的偏移量`buf`从中开始复制。
    **违约：** `0`.
*   `sourceEnd`{整数}内的偏移量`buf`在哪个位置停止复制（不是
    包含）。**违约：** [`buf.length`][buf.length].
*   返回：{整数} 复制的字节数。

从`buf`到`target`，即使`target`
内存区域与`buf`.

[`TypedArray.prototype.set()`][TypedArray.prototype.set()]执行相同的操作，并且可用
对于所有 TypedArrays，包括 Node.js`Buffer`s，尽管它需要
不同的函数参数。

```mjs
import { Buffer } from 'node:buffer';

// Create two `Buffer` instances.
const buf1 = Buffer.allocUnsafe(26);
const buf2 = Buffer.allocUnsafe(26).fill('!');

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf1[i] = i + 97;
}

// Copy `buf1` bytes 16 through 19 into `buf2` starting at byte 8 of `buf2`.
buf1.copy(buf2, 8, 16, 20);
// This is equivalent to:
// buf2.set(buf1.subarray(16, 20), 8);

console.log(buf2.toString('ascii', 0, 25));
// Prints: !!!!!!!!qrst!!!!!!!!!!!!!
```

```cjs
const { Buffer } = require('node:buffer');

// Create two `Buffer` instances.
const buf1 = Buffer.allocUnsafe(26);
const buf2 = Buffer.allocUnsafe(26).fill('!');

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf1[i] = i + 97;
}

// Copy `buf1` bytes 16 through 19 into `buf2` starting at byte 8 of `buf2`.
buf1.copy(buf2, 8, 16, 20);
// This is equivalent to:
// buf2.set(buf1.subarray(16, 20), 8);

console.log(buf2.toString('ascii', 0, 25));
// Prints: !!!!!!!!qrst!!!!!!!!!!!!!
```

```mjs
import { Buffer } from 'node:buffer';

// Create a `Buffer` and copy data from one region to an overlapping region
// within the same `Buffer`.

const buf = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf[i] = i + 97;
}

buf.copy(buf, 0, 4, 10);

console.log(buf.toString());
// Prints: efghijghijklmnopqrstuvwxyz
```

```cjs
const { Buffer } = require('node:buffer');

// Create a `Buffer` and copy data from one region to an overlapping region
// within the same `Buffer`.

const buf = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf[i] = i + 97;
}

buf.copy(buf, 0, 4, 10);

console.log(buf.toString());
// Prints: efghijghijklmnopqrstuvwxyz
```

### `buf.entries()`

<!-- YAML
added: v1.1.0
-->

*   返回： {迭代器}

创建并返回[迭 代][iterator]之`[index, byte]`从内容中配对
之`buf`.

```mjs
import { Buffer } from 'node:buffer';

// Log the entire contents of a `Buffer`.

const buf = Buffer.from('buffer');

for (const pair of buf.entries()) {
  console.log(pair);
}
// Prints:
//   [0, 98]
//   [1, 117]
//   [2, 102]
//   [3, 102]
//   [4, 101]
//   [5, 114]
```

```cjs
const { Buffer } = require('node:buffer');

// Log the entire contents of a `Buffer`.

const buf = Buffer.from('buffer');

for (const pair of buf.entries()) {
  console.log(pair);
}
// Prints:
//   [0, 98]
//   [1, 117]
//   [2, 102]
//   [3, 102]
//   [4, 101]
//   [5, 114]
```

### `buf.equals(otherBuffer)`

<!-- YAML
added: v0.11.13
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The arguments can now be `Uint8Array`s.
-->

*   `otherBuffer`{缓冲区|Uint8Array} A`Buffer`或[`Uint8Array`][Uint8Array]用它
    比较`buf`.
*   返回：{布尔值}

返回`true`如果两者兼而有之`buf`和`otherBuffer`具有完全相同的字节，
`false`否则。相当于
[`buf.compare(otherBuffer) === 0`][`buf.compare()`].

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('414243', 'hex');
const buf3 = Buffer.from('ABCD');

console.log(buf1.equals(buf2));
// Prints: true
console.log(buf1.equals(buf3));
// Prints: false
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('414243', 'hex');
const buf3 = Buffer.from('ABCD');

console.log(buf1.equals(buf2));
// Prints: true
console.log(buf1.equals(buf3));
// Prints: false
```

### `buf.fill(value[, offset[, end]][, encoding])`

<!-- YAML
added: v0.5.0
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22969
    description: Throws `ERR_OUT_OF_RANGE` instead of `ERR_INDEX_OUT_OF_RANGE`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18790
    description: Negative `end` values throw an `ERR_INDEX_OUT_OF_RANGE` error.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18129
    description: Attempting to fill a non-zero length buffer with a zero length
                 buffer triggers a thrown exception.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17427
    description: Specifying an invalid string for `value` triggers a thrown
                 exception.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4935
    description: The `encoding` parameter is supported now.
-->

*   `value`{字符串|缓冲区|Uint8数组|整数} 要填充的值`buf`.
*   `offset`{整数}开始填充之前要跳过的字节数`buf`.
    **违约：** `0`.
*   `end`{整数}在哪里停止灌装`buf`（不包括在内）。**违约：**
    [`buf.length`][buf.length].
*   `encoding`{字符串}的编码`value`如果`value`是一个字符串。
    **违约：** `'utf8'`.
*   返回：{缓冲区} 对`buf`.

充满`buf`具有指定的`value`.如果`offset`和`end`未给出，
整个`buf`将填写：

```mjs
import { Buffer } from 'node:buffer';

// Fill a `Buffer` with the ASCII character 'h'.

const b = Buffer.allocUnsafe(50).fill('h');

console.log(b.toString());
// Prints: hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
```

```cjs
const { Buffer } = require('node:buffer');

// Fill a `Buffer` with the ASCII character 'h'.

const b = Buffer.allocUnsafe(50).fill('h');

console.log(b.toString());
// Prints: hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
```

`value`被胁迫到`uint32`值（如果它不是字符串），`Buffer`或
整数。如果生成的整数大于`255`（十进制），`buf`将是
填充`value & 255`.

如果最终写入`fill()`操作落在多字节字符上，
则只有该字符的字节适合`buf`都写：

```mjs
import { Buffer } from 'node:buffer';

// Fill a `Buffer` with character that takes up two bytes in UTF-8.

console.log(Buffer.allocUnsafe(5).fill('\u0222'));
// Prints: <Buffer c8 a2 c8 a2 c8>
```

```cjs
const { Buffer } = require('node:buffer');

// Fill a `Buffer` with character that takes up two bytes in UTF-8.

console.log(Buffer.allocUnsafe(5).fill('\u0222'));
// Prints: <Buffer c8 a2 c8 a2 c8>
```

如果`value`包含无效字符，它被截断;如果没有有效
填充数据仍然存在，将引发异常：

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(5);

console.log(buf.fill('a'));
// Prints: <Buffer 61 61 61 61 61>
console.log(buf.fill('aazz', 'hex'));
// Prints: <Buffer aa aa aa aa aa>
console.log(buf.fill('zz', 'hex'));
// Throws an exception.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(5);

console.log(buf.fill('a'));
// Prints: <Buffer 61 61 61 61 61>
console.log(buf.fill('aazz', 'hex'));
// Prints: <Buffer aa aa aa aa aa>
console.log(buf.fill('zz', 'hex'));
// Throws an exception.
```

### `buf.includes(value[, byteOffset][, encoding])`

<!-- YAML
added: v5.3.0
-->

*   `value`{字符串|缓冲区|Uint8Array|integer} 要搜索的内容。
*   `byteOffset`{整数}从哪里开始搜索`buf`.如果为负，则
    偏移量从`buf`.**违约：** `0`.
*   `encoding`{字符串}如果`value`是一个字符串，这是它的编码。
    **违约：** `'utf8'`.
*   返回：{布尔值}`true`如果`value`被发现于`buf`,`false`否则。

相当于[`buf.indexOf() !== -1`][`buf.indexOf()`].

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('this is a buffer');

console.log(buf.includes('this'));
// Prints: true
console.log(buf.includes('is'));
// Prints: true
console.log(buf.includes(Buffer.from('a buffer')));
// Prints: true
console.log(buf.includes(97));
// Prints: true (97 is the decimal ASCII value for 'a')
console.log(buf.includes(Buffer.from('a buffer example')));
// Prints: false
console.log(buf.includes(Buffer.from('a buffer example').slice(0, 8)));
// Prints: true
console.log(buf.includes('this', 4));
// Prints: false
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('this is a buffer');

console.log(buf.includes('this'));
// Prints: true
console.log(buf.includes('is'));
// Prints: true
console.log(buf.includes(Buffer.from('a buffer')));
// Prints: true
console.log(buf.includes(97));
// Prints: true (97 is the decimal ASCII value for 'a')
console.log(buf.includes(Buffer.from('a buffer example')));
// Prints: false
console.log(buf.includes(Buffer.from('a buffer example').slice(0, 8)));
// Prints: true
console.log(buf.includes('this', 4));
// Prints: false
```

### `buf.indexOf(value[, byteOffset][, encoding])`

<!-- YAML
added: v1.5.0
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `value` can now be a `Uint8Array`.
  - version:
    - v5.7.0
    - v4.4.0
    pr-url: https://github.com/nodejs/node/pull/4803
    description: When `encoding` is being passed, the `byteOffset` parameter
                 is no longer required.
-->

*   `value`{字符串|缓冲区|Uint8Array|integer} 要搜索的内容。
*   `byteOffset`{整数}从哪里开始搜索`buf`.如果为负，则
    偏移量从`buf`.**违约：** `0`.
*   `encoding`{字符串}如果`value`是一个字符串，这是用于
    确定将在 中搜索的字符串的二进制表示形式
    `buf`.**违约：** `'utf8'`.
*   返回值：{整数} 第一次出现的索引`value`在`buf`或
    `-1`如果`buf`不包含`value`.

如果`value`是：

*   一个字符串，`value`根据 中的字符编码进行解释
    `encoding`.
*   一个`Buffer`或[`Uint8Array`][Uint8Array],`value`将全部使用。
    比较部分`Buffer`用[`buf.subarray`][buf.subarray].
*   一个数字，`value`将被解释为无符号 8 位整数
    值介于`0`和`255`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('this is a buffer');

console.log(buf.indexOf('this'));
// Prints: 0
console.log(buf.indexOf('is'));
// Prints: 2
console.log(buf.indexOf(Buffer.from('a buffer')));
// Prints: 8
console.log(buf.indexOf(97));
// Prints: 8 (97 is the decimal ASCII value for 'a')
console.log(buf.indexOf(Buffer.from('a buffer example')));
// Prints: -1
console.log(buf.indexOf(Buffer.from('a buffer example').slice(0, 8)));
// Prints: 8

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.indexOf('\u03a3', 0, 'utf16le'));
// Prints: 4
console.log(utf16Buffer.indexOf('\u03a3', -4, 'utf16le'));
// Prints: 6
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('this is a buffer');

console.log(buf.indexOf('this'));
// Prints: 0
console.log(buf.indexOf('is'));
// Prints: 2
console.log(buf.indexOf(Buffer.from('a buffer')));
// Prints: 8
console.log(buf.indexOf(97));
// Prints: 8 (97 is the decimal ASCII value for 'a')
console.log(buf.indexOf(Buffer.from('a buffer example')));
// Prints: -1
console.log(buf.indexOf(Buffer.from('a buffer example').slice(0, 8)));
// Prints: 8

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.indexOf('\u03a3', 0, 'utf16le'));
// Prints: 4
console.log(utf16Buffer.indexOf('\u03a3', -4, 'utf16le'));
// Prints: 6
```

如果`value`不是字符串、数字或`Buffer`，此方法将抛出一个
`TypeError`.如果`value`是一个数字，它将被强制为有效的字节值，
介于 0 和 255 之间的整数。

如果`byteOffset`不是一个数字，它将被强制为一个数字。如果结果
的胁迫是`NaN`或`0`，则将搜索整个缓冲区。这
行为匹配[`String.prototype.indexOf()`][String.prototype.indexOf()].

```mjs
import { Buffer } from 'node:buffer';

const b = Buffer.from('abcdef');

// Passing a value that's a number, but not a valid byte.
// Prints: 2, equivalent to searching for 99 or 'c'.
console.log(b.indexOf(99.9));
console.log(b.indexOf(256 + 99));

// Passing a byteOffset that coerces to NaN or 0.
// Prints: 1, searching the whole buffer.
console.log(b.indexOf('b', undefined));
console.log(b.indexOf('b', {}));
console.log(b.indexOf('b', null));
console.log(b.indexOf('b', []));
```

```cjs
const { Buffer } = require('node:buffer');

const b = Buffer.from('abcdef');

// Passing a value that's a number, but not a valid byte.
// Prints: 2, equivalent to searching for 99 or 'c'.
console.log(b.indexOf(99.9));
console.log(b.indexOf(256 + 99));

// Passing a byteOffset that coerces to NaN or 0.
// Prints: 1, searching the whole buffer.
console.log(b.indexOf('b', undefined));
console.log(b.indexOf('b', {}));
console.log(b.indexOf('b', null));
console.log(b.indexOf('b', []));
```

如果`value`为空字符串或空`Buffer`和`byteOffset`更少
比`buf.length`,`byteOffset`将被退回。如果`value`为空，并且
`byteOffset`至少`buf.length`,`buf.length`将被退回。

### `buf.keys()`

<!-- YAML
added: v1.1.0
-->

*   返回： {迭代器}

创建并返回[迭 代][iterator]之`buf`键（索引）。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('buffer');

for (const key of buf.keys()) {
  console.log(key);
}
// Prints:
//   0
//   1
//   2
//   3
//   4
//   5
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('buffer');

for (const key of buf.keys()) {
  console.log(key);
}
// Prints:
//   0
//   1
//   2
//   3
//   4
//   5
```

### `buf.lastIndexOf(value[, byteOffset][, encoding])`

<!-- YAML
added: v6.0.0
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `value` can now be a `Uint8Array`.
-->

*   `value`{字符串|缓冲区|Uint8Array|integer} 要搜索的内容。
*   `byteOffset`{整数}从哪里开始搜索`buf`.如果为负，则
    偏移量从`buf`.**违约：**
    `buf.length - 1`.
*   `encoding`{字符串}如果`value`是一个字符串，这是用于
    确定将在 中搜索的字符串的二进制表示形式
    `buf`.**违约：** `'utf8'`.
*   返回值：{整数} 最后一次出现的索引`value`在`buf`或
    `-1`如果`buf`不包含`value`.

与[`buf.indexOf()`][buf.indexOf()]，但最后一次出现的除外`value`找到
而不是第一次出现。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('this buffer is a buffer');

console.log(buf.lastIndexOf('this'));
// Prints: 0
console.log(buf.lastIndexOf('buffer'));
// Prints: 17
console.log(buf.lastIndexOf(Buffer.from('buffer')));
// Prints: 17
console.log(buf.lastIndexOf(97));
// Prints: 15 (97 is the decimal ASCII value for 'a')
console.log(buf.lastIndexOf(Buffer.from('yolo')));
// Prints: -1
console.log(buf.lastIndexOf('buffer', 5));
// Prints: 5
console.log(buf.lastIndexOf('buffer', 4));
// Prints: -1

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.lastIndexOf('\u03a3', undefined, 'utf16le'));
// Prints: 6
console.log(utf16Buffer.lastIndexOf('\u03a3', -5, 'utf16le'));
// Prints: 4
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('this buffer is a buffer');

console.log(buf.lastIndexOf('this'));
// Prints: 0
console.log(buf.lastIndexOf('buffer'));
// Prints: 17
console.log(buf.lastIndexOf(Buffer.from('buffer')));
// Prints: 17
console.log(buf.lastIndexOf(97));
// Prints: 15 (97 is the decimal ASCII value for 'a')
console.log(buf.lastIndexOf(Buffer.from('yolo')));
// Prints: -1
console.log(buf.lastIndexOf('buffer', 5));
// Prints: 5
console.log(buf.lastIndexOf('buffer', 4));
// Prints: -1

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.lastIndexOf('\u03a3', undefined, 'utf16le'));
// Prints: 6
console.log(utf16Buffer.lastIndexOf('\u03a3', -5, 'utf16le'));
// Prints: 4
```

如果`value`不是字符串、数字或`Buffer`，此方法将抛出一个
`TypeError`.如果`value`是一个数字，它将被强制为有效的字节值，
介于 0 和 255 之间的整数。

如果`byteOffset`不是一个数字，它将被强制为一个数字。任何参数
强制`NaN`喜欢`{}`或`undefined`，将搜索整个缓冲区。
此行为匹配[`String.prototype.lastIndexOf()`][String.prototype.lastIndexOf()].

```mjs
import { Buffer } from 'node:buffer';

const b = Buffer.from('abcdef');

// Passing a value that's a number, but not a valid byte.
// Prints: 2, equivalent to searching for 99 or 'c'.
console.log(b.lastIndexOf(99.9));
console.log(b.lastIndexOf(256 + 99));

// Passing a byteOffset that coerces to NaN.
// Prints: 1, searching the whole buffer.
console.log(b.lastIndexOf('b', undefined));
console.log(b.lastIndexOf('b', {}));

// Passing a byteOffset that coerces to 0.
// Prints: -1, equivalent to passing 0.
console.log(b.lastIndexOf('b', null));
console.log(b.lastIndexOf('b', []));
```

```cjs
const { Buffer } = require('node:buffer');

const b = Buffer.from('abcdef');

// Passing a value that's a number, but not a valid byte.
// Prints: 2, equivalent to searching for 99 or 'c'.
console.log(b.lastIndexOf(99.9));
console.log(b.lastIndexOf(256 + 99));

// Passing a byteOffset that coerces to NaN.
// Prints: 1, searching the whole buffer.
console.log(b.lastIndexOf('b', undefined));
console.log(b.lastIndexOf('b', {}));

// Passing a byteOffset that coerces to 0.
// Prints: -1, equivalent to passing 0.
console.log(b.lastIndexOf('b', null));
console.log(b.lastIndexOf('b', []));
```

如果`value`为空字符串或空`Buffer`,`byteOffset`将被退回。

### `buf.length`

<!-- YAML
added: v0.1.90
-->

*   {整数}

返回 字节数`buf`.

```mjs
import { Buffer } from 'node:buffer';

// Create a `Buffer` and write a shorter string to it using UTF-8.

const buf = Buffer.alloc(1234);

console.log(buf.length);
// Prints: 1234

buf.write('some string', 0, 'utf8');

console.log(buf.length);
// Prints: 1234
```

```cjs
const { Buffer } = require('node:buffer');

// Create a `Buffer` and write a shorter string to it using UTF-8.

const buf = Buffer.alloc(1234);

console.log(buf.length);
// Prints: 1234

buf.write('some string', 0, 'utf8');

console.log(buf.length);
// Prints: 1234
```

### `buf.parent`

<!-- YAML
deprecated: v8.0.0
-->

> 稳定性：0 - 已弃用：使用[`buf.buffer`][buf.buffer]相反。

这`buf.parent`属性是 的已弃用别名`buf.buffer`.

### `buf.readBigInt64BE([offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{bigint}

读取有符号的大端 64 位整数`buf`在指定的`offset`.

从`Buffer`被解释为二的补码符号
值。

### `buf.readBigInt64LE([offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{bigint}

读取有符号的小端 64 位整数`buf`在指定的
`offset`.

从`Buffer`被解释为二的补码符号
值。

### `buf.readBigUInt64BE([offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
changes:
  - version:
    - v14.10.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34960
    description: This function is also available as `buf.readBigUint64BE()`.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{bigint}

读取无符号的大端 64 位整数`buf`在指定的
`offset`.

此功能也可在`readBigUint64BE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff]);

console.log(buf.readBigUInt64BE(0));
// Prints: 4294967295n
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff]);

console.log(buf.readBigUInt64BE(0));
// Prints: 4294967295n
```

### `buf.readBigUInt64LE([offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
changes:
  - version:
    - v14.10.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34960
    description: This function is also available as `buf.readBigUint64LE()`.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{bigint}

读取无符号的小端 64 位整数`buf`在指定的
`offset`.

此功能也可在`readBigUint64LE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff]);

console.log(buf.readBigUInt64LE(0));
// Prints: 18446744069414584320n
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff]);

console.log(buf.readBigUInt64LE(0));
// Prints: 18446744069414584320n
```

### `buf.readDoubleBE([offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回值：{数字}

读取 64 位大端双精度值`buf`在指定的`offset`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

console.log(buf.readDoubleBE(0));
// Prints: 8.20788039913184e-304
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

console.log(buf.readDoubleBE(0));
// Prints: 8.20788039913184e-304
```

### `buf.readDoubleLE([offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回值：{数字}

读取 64 位小端双精度值`buf`在指定的`offset`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

console.log(buf.readDoubleLE(0));
// Prints: 5.447603722011605e-270
console.log(buf.readDoubleLE(1));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

console.log(buf.readDoubleLE(0));
// Prints: 5.447603722011605e-270
console.log(buf.readDoubleLE(1));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readFloatBE([offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回值：{数字}

读取 32 位大端浮点数`buf`在指定的`offset`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, 2, 3, 4]);

console.log(buf.readFloatBE(0));
// Prints: 2.387939260590663e-38
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, 2, 3, 4]);

console.log(buf.readFloatBE(0));
// Prints: 2.387939260590663e-38
```

### `buf.readFloatLE([offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回值：{数字}

读取 32 位小端浮点数`buf`在指定的`offset`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, 2, 3, 4]);

console.log(buf.readFloatLE(0));
// Prints: 1.539989614439558e-36
console.log(buf.readFloatLE(1));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, 2, 3, 4]);

console.log(buf.readFloatLE(0));
// Prints: 1.539989614439558e-36
console.log(buf.readFloatLE(1));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readInt8([offset])`

<!-- YAML
added: v0.5.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 1`.**违约：** `0`.
*   返回：{整数}

读取有符号的 8 位整数`buf`在指定的`offset`.

从`Buffer`被解释为二的补码有符号值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([-1, 5]);

console.log(buf.readInt8(0));
// Prints: -1
console.log(buf.readInt8(1));
// Prints: 5
console.log(buf.readInt8(2));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([-1, 5]);

console.log(buf.readInt8(0));
// Prints: -1
console.log(buf.readInt8(1));
// Prints: 5
console.log(buf.readInt8(2));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readInt16BE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}

读取有符号的大端 16 位整数`buf`在指定的`offset`.

从`Buffer`被解释为二的补码有符号值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0, 5]);

console.log(buf.readInt16BE(0));
// Prints: 5
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0, 5]);

console.log(buf.readInt16BE(0));
// Prints: 5
```

### `buf.readInt16LE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}

读取有符号的小端 16 位整数`buf`在指定的
`offset`.

从`Buffer`被解释为二的补码有符号值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0, 5]);

console.log(buf.readInt16LE(0));
// Prints: 1280
console.log(buf.readInt16LE(1));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0, 5]);

console.log(buf.readInt16LE(0));
// Prints: 1280
console.log(buf.readInt16LE(1));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readInt32BE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}

读取有符号的大端 32 位整数`buf`在指定的`offset`.

从`Buffer`被解释为二的补码有符号值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0, 0, 0, 5]);

console.log(buf.readInt32BE(0));
// Prints: 5
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0, 0, 0, 5]);

console.log(buf.readInt32BE(0));
// Prints: 5
```

### `buf.readInt32LE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}

读取有符号的小端 32 位整数`buf`在指定的
`offset`.

从`Buffer`被解释为二的补码有符号值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0, 0, 0, 5]);

console.log(buf.readInt32LE(0));
// Prints: 83886080
console.log(buf.readInt32LE(1));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0, 0, 0, 5]);

console.log(buf.readInt32LE(0));
// Prints: 83886080
console.log(buf.readInt32LE(1));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readIntBE(offset, byteLength)`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要读取的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}

读`byteLength`字节数`buf`在指定的`offset`
并将结果解释为大端序，即二的补码有符号值
支持高达 48 位的精度。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readIntBE(0, 6).toString(16));
// Prints: 1234567890ab
console.log(buf.readIntBE(1, 6).toString(16));
// Throws ERR_OUT_OF_RANGE.
console.log(buf.readIntBE(1, 0).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readIntBE(0, 6).toString(16));
// Prints: 1234567890ab
console.log(buf.readIntBE(1, 6).toString(16));
// Throws ERR_OUT_OF_RANGE.
console.log(buf.readIntBE(1, 0).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readIntLE(offset, byteLength)`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要读取的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}

读`byteLength`字节数`buf`在指定的`offset`
并将结果解释为小端序，即二的补码有符号值
支持高达 48 位的精度。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readIntLE(0, 6).toString(16));
// Prints: -546f87a9cbee
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readIntLE(0, 6).toString(16));
// Prints: -546f87a9cbee
```

### `buf.readUInt8([offset])`

<!-- YAML
added: v0.5.0
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUint8()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 1`.**违约：** `0`.
*   返回：{整数}

从中读取无符号的 8 位整数`buf`在指定的`offset`.

此功能也可在`readUint8`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([1, -2]);

console.log(buf.readUInt8(0));
// Prints: 1
console.log(buf.readUInt8(1));
// Prints: 254
console.log(buf.readUInt8(2));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([1, -2]);

console.log(buf.readUInt8(0));
// Prints: 1
console.log(buf.readUInt8(1));
// Prints: 254
console.log(buf.readUInt8(2));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readUInt16BE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUint16BE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}

读取无符号的大端 16 位整数`buf`在指定的
`offset`.

此功能也可在`readUint16BE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56]);

console.log(buf.readUInt16BE(0).toString(16));
// Prints: 1234
console.log(buf.readUInt16BE(1).toString(16));
// Prints: 3456
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56]);

console.log(buf.readUInt16BE(0).toString(16));
// Prints: 1234
console.log(buf.readUInt16BE(1).toString(16));
// Prints: 3456
```

### `buf.readUInt16LE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUint16LE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}

读取无符号的小端 16 位整数`buf`在指定的
`offset`.

此功能也可在`readUint16LE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56]);

console.log(buf.readUInt16LE(0).toString(16));
// Prints: 3412
console.log(buf.readUInt16LE(1).toString(16));
// Prints: 5634
console.log(buf.readUInt16LE(2).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56]);

console.log(buf.readUInt16LE(0).toString(16));
// Prints: 3412
console.log(buf.readUInt16LE(1).toString(16));
// Prints: 5634
console.log(buf.readUInt16LE(2).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readUInt32BE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUint32BE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}

读取无符号的大端 32 位整数`buf`在指定的
`offset`.

此功能也可在`readUint32BE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

console.log(buf.readUInt32BE(0).toString(16));
// Prints: 12345678
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

console.log(buf.readUInt32BE(0).toString(16));
// Prints: 12345678
```

### `buf.readUInt32LE([offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUint32LE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}

读取无符号的小端 32 位整数`buf`在指定的
`offset`.

此功能也可在`readUint32LE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

console.log(buf.readUInt32LE(0).toString(16));
// Prints: 78563412
console.log(buf.readUInt32LE(1).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

console.log(buf.readUInt32LE(0).toString(16));
// Prints: 78563412
console.log(buf.readUInt32LE(1).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readUIntBE(offset, byteLength)`

<!-- YAML
added: v0.11.15
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUintBE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要读取的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}

读`byteLength`字节数`buf`在指定的`offset`
并将结果解释为支持
精度高达 48 位。

此功能也可在`readUintBE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readUIntBE(0, 6).toString(16));
// Prints: 1234567890ab
console.log(buf.readUIntBE(1, 6).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readUIntBE(0, 6).toString(16));
// Prints: 1234567890ab
console.log(buf.readUIntBE(1, 6).toString(16));
// Throws ERR_OUT_OF_RANGE.
```

### `buf.readUIntLE(offset, byteLength)`

<!-- YAML
added: v0.11.15
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.readUintLE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `offset`{整数}开始读取之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要读取的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}

读`byteLength`字节数`buf`在指定的`offset`
并将结果解释为无符号的小端整数支持
精度高达 48 位。

此功能也可在`readUintLE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readUIntLE(0, 6).toString(16));
// Prints: ab9078563412
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readUIntLE(0, 6).toString(16));
// Prints: ab9078563412
```

### `buf.subarray([start[, end]])`

<!-- YAML
added: v3.0.0
-->

*   `start`{整数}其中新的`Buffer`将启动。**违约：** `0`.
*   `end`{整数}其中新的`Buffer`将结束（不包括）。
    **违约：** [`buf.length`][buf.length].
*   返回：{缓冲区}

返回新的`Buffer`引用与原始内存相同的内存，但
偏移和裁剪`start`和`end`指标。

指定`end`大于[`buf.length`][buf.length]将返回与
即`end`等于[`buf.length`][buf.length].

此方法继承自[`TypedArray.prototype.subarray()`][TypedArray.prototype.subarray()].

修改新的`Buffer`切片将修改原始内存`Buffer`
因为两个对象的分配内存重叠。

```mjs
import { Buffer } from 'node:buffer';

// Create a `Buffer` with the ASCII alphabet, take a slice, and modify one byte
// from the original `Buffer`.

const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf1[i] = i + 97;
}

const buf2 = buf1.subarray(0, 3);

console.log(buf2.toString('ascii', 0, buf2.length));
// Prints: abc

buf1[0] = 33;

console.log(buf2.toString('ascii', 0, buf2.length));
// Prints: !bc
```

```cjs
const { Buffer } = require('node:buffer');

// Create a `Buffer` with the ASCII alphabet, take a slice, and modify one byte
// from the original `Buffer`.

const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf1[i] = i + 97;
}

const buf2 = buf1.subarray(0, 3);

console.log(buf2.toString('ascii', 0, buf2.length));
// Prints: abc

buf1[0] = 33;

console.log(buf2.toString('ascii', 0, buf2.length));
// Prints: !bc
```

指定负索引会导致相对于
结束`buf`而不是开始。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('buffer');

console.log(buf.subarray(-6, -1).toString());
// Prints: buffe
// (Equivalent to buf.subarray(0, 5).)

console.log(buf.subarray(-6, -2).toString());
// Prints: buff
// (Equivalent to buf.subarray(0, 4).)

console.log(buf.subarray(-5, -2).toString());
// Prints: uff
// (Equivalent to buf.subarray(1, 4).)
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('buffer');

console.log(buf.subarray(-6, -1).toString());
// Prints: buffe
// (Equivalent to buf.subarray(0, 5).)

console.log(buf.subarray(-6, -2).toString());
// Prints: buff
// (Equivalent to buf.subarray(0, 4).)

console.log(buf.subarray(-5, -2).toString());
// Prints: uff
// (Equivalent to buf.subarray(1, 4).)
```

### `buf.slice([start[, end]])`

<!-- YAML
added: v0.3.0
changes:
  - version:
    - v17.5.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41596
    description: The buf.slice() method has been deprecated.
  - version:
    - v7.1.0
    - v6.9.2
    pr-url: https://github.com/nodejs/node/pull/9341
    description: Coercing the offsets to integers now handles values outside
                 the 32-bit integer range properly.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/9101
    description: All offsets are now coerced to integers before doing any
                 calculations with them.
-->

*   `start`{整数}其中新的`Buffer`将启动。**违约：** `0`.
*   `end`{整数}其中新的`Buffer`将结束（不包括）。
    **违约：** [`buf.length`][buf.length].
*   返回：{缓冲区}

> 稳定性：0 - 已弃用：使用[`buf.subarray`][buf.subarray]相反。

返回新的`Buffer`引用与原始内存相同的内存，但
偏移和裁剪`start`和`end`指标。

此方法与`Uint8Array.prototype.slice()`,
这是一个超类`Buffer`.要复制切片，请使用
`Uint8Array.prototype.slice()`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('buffer');

const copiedBuf = Uint8Array.prototype.slice.call(buf);
copiedBuf[0]++;
console.log(copiedBuf.toString());
// Prints: cuffer

console.log(buf.toString());
// Prints: buffer

// With buf.slice(), the original buffer is modified.
const notReallyCopiedBuf = buf.slice();
notReallyCopiedBuf[0]++;
console.log(notReallyCopiedBuf.toString());
// Prints: cuffer
console.log(buf.toString());
// Also prints: cuffer (!)
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('buffer');

const copiedBuf = Uint8Array.prototype.slice.call(buf);
copiedBuf[0]++;
console.log(copiedBuf.toString());
// Prints: cuffer

console.log(buf.toString());
// Prints: buffer

// With buf.slice(), the original buffer is modified.
const notReallyCopiedBuf = buf.slice();
notReallyCopiedBuf[0]++;
console.log(notReallyCopiedBuf.toString());
// Prints: cuffer
console.log(buf.toString());
// Also prints: cuffer (!)
```

### `buf.swap16()`

<!-- YAML
added: v5.10.0
-->

*   返回：{缓冲区} 对`buf`.

解释`buf`作为无符号 16 位整数数组，并交换
字节顺序*就地*.抛出[`ERR_INVALID_BUFFER_SIZE`][ERR_INVALID_BUFFER_SIZE]如果[`buf.length`][buf.length]
不是 2 的倍数。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// Prints: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap16();

console.log(buf1);
// Prints: <Buffer 02 01 04 03 06 05 08 07>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap16();
// Throws ERR_INVALID_BUFFER_SIZE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// Prints: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap16();

console.log(buf1);
// Prints: <Buffer 02 01 04 03 06 05 08 07>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap16();
// Throws ERR_INVALID_BUFFER_SIZE.
```

一次便捷使用`buf.swap16()`是执行快速就地转换
在 UTF-16 小端序和 UTF-16 大端序之间：

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('This is little-endian UTF-16', 'utf16le');
buf.swap16(); // Convert to big-endian UTF-16 text.
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('This is little-endian UTF-16', 'utf16le');
buf.swap16(); // Convert to big-endian UTF-16 text.
```

### `buf.swap32()`

<!-- YAML
added: v5.10.0
-->

*   返回：{缓冲区} 对`buf`.

解释`buf`作为无符号 32 位整数数组，并交换
字节顺序*就地*.抛出[`ERR_INVALID_BUFFER_SIZE`][ERR_INVALID_BUFFER_SIZE]如果[`buf.length`][buf.length]
不是 4 的倍数。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// Prints: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap32();

console.log(buf1);
// Prints: <Buffer 04 03 02 01 08 07 06 05>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap32();
// Throws ERR_INVALID_BUFFER_SIZE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// Prints: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap32();

console.log(buf1);
// Prints: <Buffer 04 03 02 01 08 07 06 05>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap32();
// Throws ERR_INVALID_BUFFER_SIZE.
```

### `buf.swap64()`

<!-- YAML
added: v6.3.0
-->

*   返回：{缓冲区} 对`buf`.

解释`buf`作为 64 位数字和交换字节顺序的数组*就地*.
抛出[`ERR_INVALID_BUFFER_SIZE`][ERR_INVALID_BUFFER_SIZE]如果[`buf.length`][buf.length]不是 8 的倍数。

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// Prints: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap64();

console.log(buf1);
// Prints: <Buffer 08 07 06 05 04 03 02 01>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap64();
// Throws ERR_INVALID_BUFFER_SIZE.
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// Prints: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap64();

console.log(buf1);
// Prints: <Buffer 08 07 06 05 04 03 02 01>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap64();
// Throws ERR_INVALID_BUFFER_SIZE.
```

### `buf.toJSON()`

<!-- YAML
added: v0.9.2
-->

*   返回： {对象}

返回 JSON 表示形式`buf`.[`JSON.stringify()`][JSON.stringify()]隐式调用
此函数在字符串化`Buffer`实例。

`Buffer.from()`接受从此方法返回的格式的对象。
特别`Buffer.from(buf.toJSON())`工作方式`Buffer.from(buf)`.

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

console.log(json);
// Prints: {"type":"Buffer","data":[1,2,3,4,5]}

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value) :
    value;
});

console.log(copy);
// Prints: <Buffer 01 02 03 04 05>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

console.log(json);
// Prints: {"type":"Buffer","data":[1,2,3,4,5]}

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value) :
    value;
});

console.log(copy);
// Prints: <Buffer 01 02 03 04 05>
```

### `buf.toString([encoding[, start[, end]]])`

<!-- YAML
added: v0.1.90
-->

*   `encoding`{字符串}要使用的字符编码。**违约：** `'utf8'`.
*   `start`{整数}开始解码的字节偏移量。**违约：** `0`.
*   `end`{整数}要在以下位置停止解码的字节偏移量（不包括）。
    **违约：** [`buf.length`][buf.length].
*   返回：{字符串}

解码`buf`根据 中指定的字符编码转换为字符串
`encoding`.`start`和`end`可以传递以仅解码的子集`buf`.

如果`encoding`是`'utf8'`并且输入中的字节序列无效 UTF-8，
则每个无效字节都替换为替换字符`U+FFFD`.

字符串实例的最大长度（以 UTF-16 代码单位为单位）可用
如[`buffer.constants.MAX_STRING_LENGTH`][buffer.constants.MAX_STRING_LENGTH].

```mjs
import { Buffer } from 'node:buffer';

const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf1[i] = i + 97;
}

console.log(buf1.toString('utf8'));
// Prints: abcdefghijklmnopqrstuvwxyz
console.log(buf1.toString('utf8', 0, 5));
// Prints: abcde

const buf2 = Buffer.from('tést');

console.log(buf2.toString('hex'));
// Prints: 74c3a97374
console.log(buf2.toString('utf8', 0, 3));
// Prints: té
console.log(buf2.toString(undefined, 0, 3));
// Prints: té
```

```cjs
const { Buffer } = require('node:buffer');

const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'.
  buf1[i] = i + 97;
}

console.log(buf1.toString('utf8'));
// Prints: abcdefghijklmnopqrstuvwxyz
console.log(buf1.toString('utf8', 0, 5));
// Prints: abcde

const buf2 = Buffer.from('tést');

console.log(buf2.toString('hex'));
// Prints: 74c3a97374
console.log(buf2.toString('utf8', 0, 3));
// Prints: té
console.log(buf2.toString(undefined, 0, 3));
// Prints: té
```

### `buf.values()`

<!-- YAML
added: v1.1.0
-->

*   返回： {迭代器}

创建并返回[迭 代][iterator]为`buf`值（字节）。此函数是
当`Buffer`用于`for..of`陈述。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.from('buffer');

for (const value of buf.values()) {
  console.log(value);
}
// Prints:
//   98
//   117
//   102
//   102
//   101
//   114

for (const value of buf) {
  console.log(value);
}
// Prints:
//   98
//   117
//   102
//   102
//   101
//   114
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.from('buffer');

for (const value of buf.values()) {
  console.log(value);
}
// Prints:
//   98
//   117
//   102
//   102
//   101
//   114

for (const value of buf) {
  console.log(value);
}
// Prints:
//   98
//   117
//   102
//   102
//   101
//   114
```

### `buf.write(string[, offset[, length]][, encoding])`

<!-- YAML
added: v0.1.90
-->

*   `string`{字符串}要写入的字符串`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数`string`.
    **违约：** `0`.
*   `length`{整数}要写入的最大字节数（写入的字节不会
    超过`buf.length - offset`).**违约：** `buf.length - offset`.
*   `encoding`{字符串}的字符编码`string`.**违约：** `'utf8'`.
*   返回：{整数} 写入的字节数。

写`string`自`buf`在`offset`根据字符编码
`encoding`.这`length`参数是要写入的字节数。如果`buf`做了
不包含足够的空间来容纳整个字符串，只有`string`将是
写。但是，不会写入部分编码的字符。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.alloc(256);

const len = buf.write('\u00bd + \u00bc = \u00be', 0);

console.log(`${len} bytes: ${buf.toString('utf8', 0, len)}`);
// Prints: 12 bytes: ½ + ¼ = ¾

const buffer = Buffer.alloc(10);

const length = buffer.write('abcd', 8);

console.log(`${length} bytes: ${buffer.toString('utf8', 8, 10)}`);
// Prints: 2 bytes : ab
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.alloc(256);

const len = buf.write('\u00bd + \u00bc = \u00be', 0);

console.log(`${len} bytes: ${buf.toString('utf8', 0, len)}`);
// Prints: 12 bytes: ½ + ¼ = ¾

const buffer = Buffer.alloc(10);

const length = buffer.write('abcd', 8);

console.log(`${length} bytes: ${buffer.toString('utf8', 8, 10)}`);
// Prints: 2 bytes : ab
```

### `buf.writeBigInt64BE(value[, offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

*   `value`{bigint}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。

`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(8);

buf.writeBigInt64BE(0x0102030405060708n, 0);

console.log(buf);
// Prints: <Buffer 01 02 03 04 05 06 07 08>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(8);

buf.writeBigInt64BE(0x0102030405060708n, 0);

console.log(buf);
// Prints: <Buffer 01 02 03 04 05 06 07 08>
```

### `buf.writeBigInt64LE(value[, offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
-->

*   `value`{bigint}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。

`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(8);

buf.writeBigInt64LE(0x0102030405060708n, 0);

console.log(buf);
// Prints: <Buffer 08 07 06 05 04 03 02 01>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(8);

buf.writeBigInt64LE(0x0102030405060708n, 0);

console.log(buf);
// Prints: <Buffer 08 07 06 05 04 03 02 01>
```

### `buf.writeBigUInt64BE(value[, offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
changes:
  - version:
    - v14.10.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34960
    description: This function is also available as `buf.writeBigUint64BE()`.
-->

*   `value`{bigint}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。

此功能也可在`writeBigUint64BE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(8);

buf.writeBigUInt64BE(0xdecafafecacefaden, 0);

console.log(buf);
// Prints: <Buffer de ca fa fe ca ce fa de>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(8);

buf.writeBigUInt64BE(0xdecafafecacefaden, 0);

console.log(buf);
// Prints: <Buffer de ca fa fe ca ce fa de>
```

### `buf.writeBigUInt64LE(value[, offset])`

<!-- YAML
added:
 - v12.0.0
 - v10.20.0
changes:
  - version:
    - v14.10.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34960
    description: This function is also available as `buf.writeBigUint64LE()`.
-->

*   `value`{bigint}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足：`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(8);

buf.writeBigUInt64LE(0xdecafafecacefaden, 0);

console.log(buf);
// Prints: <Buffer de fa ce ca fe fa ca de>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(8);

buf.writeBigUInt64LE(0xdecafafecacefaden, 0);

console.log(buf);
// Prints: <Buffer de fa ce ca fe fa ca de>
```

此功能也可在`writeBigUint64LE`别名。

### `buf.writeDoubleBE(value[, offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{数字}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。这`value`
必须是 JavaScript 编号。在以下情况下行为未定义`value`是任何东西
除了 JavaScript 编号。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(8);

buf.writeDoubleBE(123.456, 0);

console.log(buf);
// Prints: <Buffer 40 5e dd 2f 1a 9f be 77>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(8);

buf.writeDoubleBE(123.456, 0);

console.log(buf);
// Prints: <Buffer 40 5e dd 2f 1a 9f be 77>
```

### `buf.writeDoubleLE(value[, offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{数字}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 8`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。这`value`
必须是 JavaScript 编号。在以下情况下行为未定义`value`是任何东西
除了 JavaScript 编号。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(8);

buf.writeDoubleLE(123.456, 0);

console.log(buf);
// Prints: <Buffer 77 be 9f 1a 2f dd 5e 40>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(8);

buf.writeDoubleLE(123.456, 0);

console.log(buf);
// Prints: <Buffer 77 be 9f 1a 2f dd 5e 40>
```

### `buf.writeFloatBE(value[, offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{数字}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。行为是
未定义时`value`是 JavaScript 编号以外的任何数字。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeFloatBE(0xcafebabe, 0);

console.log(buf);
// Prints: <Buffer 4f 4a fe bb>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeFloatBE(0xcafebabe, 0);

console.log(buf);
// Prints: <Buffer 4f 4a fe bb>
```

### `buf.writeFloatLE(value[, offset])`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{数字}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。行为是
未定义时`value`是 JavaScript 编号以外的任何数字。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeFloatLE(0xcafebabe, 0);

console.log(buf);
// Prints: <Buffer bb fe 4a 4f>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeFloatLE(0xcafebabe, 0);

console.log(buf);
// Prints: <Buffer bb fe 4a 4f>
```

### `buf.writeInt8(value[, offset])`

<!-- YAML
added: v0.5.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 1`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`.`value`必须是有效的
有符号 8 位整数。在以下情况下行为未定义`value`是除
有符号 8 位整数。

`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(2);

buf.writeInt8(2, 0);
buf.writeInt8(-2, 1);

console.log(buf);
// Prints: <Buffer 02 fe>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(2);

buf.writeInt8(2, 0);
buf.writeInt8(-2, 1);

console.log(buf);
// Prints: <Buffer 02 fe>
```

### `buf.writeInt16BE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。 这`value`
必须是有效的有符号 16 位整数。在以下情况下行为未定义`value`是
除有符号 16 位整数以外的任何内容。

这`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(2);

buf.writeInt16BE(0x0102, 0);

console.log(buf);
// Prints: <Buffer 01 02>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(2);

buf.writeInt16BE(0x0102, 0);

console.log(buf);
// Prints: <Buffer 01 02>
```

### `buf.writeInt16LE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。 这`value`
必须是有效的有符号 16 位整数。在以下情况下行为未定义`value`是
除有符号 16 位整数以外的任何内容。

这`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(2);

buf.writeInt16LE(0x0304, 0);

console.log(buf);
// Prints: <Buffer 04 03>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(2);

buf.writeInt16LE(0x0304, 0);

console.log(buf);
// Prints: <Buffer 04 03>
```

### `buf.writeInt32BE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。这`value`
必须是有效的有符号 32 位整数。在以下情况下行为未定义`value`是
除有符号 32 位整数以外的任何内容。

这`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeInt32BE(0x01020304, 0);

console.log(buf);
// Prints: <Buffer 01 02 03 04>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeInt32BE(0x01020304, 0);

console.log(buf);
// Prints: <Buffer 01 02 03 04>
```

### `buf.writeInt32LE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。这`value`
必须是有效的有符号 32 位整数。在以下情况下行为未定义`value`是
除有符号 32 位整数以外的任何内容。

这`value`被解释并写成二的补码有符号整数。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeInt32LE(0x05060708, 0);

console.log(buf);
// Prints: <Buffer 08 07 06 05>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeInt32LE(0x05060708, 0);

console.log(buf);
// Prints: <Buffer 08 07 06 05>
```

### `buf.writeIntBE(value, offset, byteLength)`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要写入的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}`offset`加上写入的字节数。

写`byteLength`字节数`value`自`buf`在指定的`offset`
作为大端序。支持高达 48 位的精度。在以下情况下行为未定义
`value`是有符号整数以外的任何值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(6);

buf.writeIntBE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer 12 34 56 78 90 ab>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(6);

buf.writeIntBE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer 12 34 56 78 90 ab>
```

### `buf.writeIntLE(value, offset, byteLength)`

<!-- YAML
added: v0.11.15
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要写入的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}`offset`加上写入的字节数。

写`byteLength`字节数`value`自`buf`在指定的`offset`
作为小端序。支持高达 48 位的精度。行为未定义
什么时候`value`是有符号整数以外的任何值。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(6);

buf.writeIntLE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer ab 90 78 56 34 12>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(6);

buf.writeIntLE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer ab 90 78 56 34 12>
```

### `buf.writeUInt8(value[, offset])`

<!-- YAML
added: v0.5.0
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUint8()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 1`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`.`value`必须是
有效的无符号 8 位整数。在以下情况下行为未定义`value`是任何东西
除了无符号的 8 位整数。

此功能也可在`writeUint8`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

console.log(buf);
// Prints: <Buffer 03 04 23 42>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

console.log(buf);
// Prints: <Buffer 03 04 23 42>
```

### `buf.writeUInt16BE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUint16BE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。这`value`
必须是有效的无符号 16 位整数。在以下情况下行为未定义`value`
是无符号 16 位整数以外的任何值。

此功能也可在`writeUint16BE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

console.log(buf);
// Prints: <Buffer de ad be ef>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

console.log(buf);
// Prints: <Buffer de ad be ef>
```

### `buf.writeUInt16LE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUint16LE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 2`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。这`value`
必须是有效的无符号 16 位整数。在以下情况下行为未定义`value`是
除无符号 16 位整数以外的任何内容。

此功能也可在`writeUint16LE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

console.log(buf);
// Prints: <Buffer ad de ef be>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

console.log(buf);
// Prints: <Buffer ad de ef be>
```

### `buf.writeUInt32BE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUint32BE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为大端序。这`value`
必须是有效的无符号 32 位整数。在以下情况下行为未定义`value`
是无符号 32 位整数以外的任何内容。

此功能也可在`writeUint32BE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeUInt32BE(0xfeedface, 0);

console.log(buf);
// Prints: <Buffer fe ed fa ce>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeUInt32BE(0xfeedface, 0);

console.log(buf);
// Prints: <Buffer fe ed fa ce>
```

### `buf.writeUInt32LE(value[, offset])`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUint32LE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - 4`.**违约：** `0`.
*   返回：{整数}`offset`加上写入的字节数。

写`value`自`buf`在指定的`offset`作为小端序。这`value`
必须是有效的无符号 32 位整数。在以下情况下行为未定义`value`是
除无符号 32 位整数以外的任何内容。

此功能也可在`writeUint32LE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(4);

buf.writeUInt32LE(0xfeedface, 0);

console.log(buf);
// Prints: <Buffer ce fa ed fe>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(4);

buf.writeUInt32LE(0xfeedface, 0);

console.log(buf);
// Prints: <Buffer ce fa ed fe>
```

### `buf.writeUIntBE(value, offset, byteLength)`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUintBE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要写入的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}`offset`加上写入的字节数。

写`byteLength`字节数`value`自`buf`在指定的`offset`
作为大端序。支持高达 48 位的精度。行为未定义
什么时候`value`是无符号整数以外的任何值。

此功能也可在`writeUintBE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(6);

buf.writeUIntBE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer 12 34 56 78 90 ab>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(6);

buf.writeUIntBE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer 12 34 56 78 90 ab>
```

### `buf.writeUIntLE(value, offset, byteLength)`

<!-- YAML
added: v0.5.5
changes:
  - version:
    - v14.9.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34729
    description: This function is also available as `buf.writeUintLE()`.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18395
    description: Removed `noAssert` and no implicit coercion of the offset
                 and `byteLength` to `uint32` anymore.
-->

*   `value`{整数}要写入的数字`buf`.
*   `offset`{整数}开始写入之前要跳过的字节数。必须
    满足`0 <= offset <= buf.length - byteLength`.
*   `byteLength`{整数}要写入的字节数。必须满足
    `0 < byteLength <= 6`.
*   返回：{整数}`offset`加上写入的字节数。

写`byteLength`字节数`value`自`buf`在指定的`offset`
作为小端序。支持高达 48 位的精度。行为未定义
什么时候`value`是无符号整数以外的任何值。

此功能也可在`writeUintLE`别名。

```mjs
import { Buffer } from 'node:buffer';

const buf = Buffer.allocUnsafe(6);

buf.writeUIntLE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer ab 90 78 56 34 12>
```

```cjs
const { Buffer } = require('node:buffer');

const buf = Buffer.allocUnsafe(6);

buf.writeUIntLE(0x1234567890ab, 0, 6);

console.log(buf);
// Prints: <Buffer ab 90 78 56 34 12>
```

### `new Buffer(array)`

<!-- YAML
deprecated: v6.0.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19524
    description: Calling this constructor emits a deprecation warning when
                 run from code outside the `node_modules` directory.
  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> 稳定性：0 - 已弃用：使用[`Buffer.from(array)`][Buffer.from(array)]相反。

*   `array`{整数\[]}要从中复制的字节数组。

看[`Buffer.from(array)`][Buffer.from(array)].

### `new Buffer(arrayBuffer[, byteOffset[, length]])`

<!-- YAML
added: v3.0.0
deprecated: v6.0.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19524
    description: Calling this constructor emits a deprecation warning when
                 run from code outside the `node_modules` directory.
  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4682
    description: The `byteOffset` and `length` parameters are supported now.
-->

> 稳定性：0 - 已弃用：使用
> [`Buffer.from(arrayBuffer[, byteOffset[, length]])`][`Buffer.from(arrayBuf)`]
> 相反。

*   `arrayBuffer`{ArrayBuffer|SharedArrayBuffer} An[`ArrayBuffer`][ArrayBuffer],
    [`SharedArrayBuffer`][SharedArrayBuffer]或`.buffer`的属性[`TypedArray`][TypedArray].
*   `byteOffset`{整数}要公开的第一个字节的索引。**违约：** `0`.
*   `length`{整数}要公开的字节数。
    **违约：** `arrayBuffer.byteLength - byteOffset`.

看
[`Buffer.from(arrayBuffer[, byteOffset[, length]])`][`Buffer.from(arrayBuf)`].

### `new Buffer(buffer)`

<!-- YAML
deprecated: v6.0.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19524
    description: Calling this constructor emits a deprecation warning when
                 run from code outside the `node_modules` directory.
  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> 稳定性：0 - 已弃用：使用[`Buffer.from(buffer)`][Buffer.from(buffer)]相反。

*   `buffer`{缓冲区|Uint8Array} An existing`Buffer`或[`Uint8Array`][Uint8Array]从
    要复制数据。

看[`Buffer.from(buffer)`][Buffer.from(buffer)].

### `new Buffer(size)`

<!-- YAML
deprecated: v6.0.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19524
    description: Calling this constructor emits a deprecation warning when
                 run from code outside the `node_modules` directory.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12141
    description: The `new Buffer(size)` will return zero-filled memory by
                 default.
  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> 稳定性：0 - 已弃用：使用[`Buffer.alloc()`][Buffer.alloc()]相反（另请参见
> [`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]).

*   `size`{整数}新品的所需长度`Buffer`.

看[`Buffer.alloc()`][Buffer.alloc()]和[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()].此变体的
构造函数等效于[`Buffer.alloc()`][Buffer.alloc()].

### `new Buffer(string[, encoding])`

<!-- YAML
deprecated: v6.0.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19524
    description: Calling this constructor emits a deprecation warning when
                 run from code outside the `node_modules` directory.
  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> 稳定性：0 - 已弃用：
> 用[`Buffer.from(string[, encoding])`][`Buffer.from(string)`]相反。

*   `string`{字符串}要编码的字符串。
*   `encoding`{字符串}的编码`string`.**违约：** `'utf8'`.

看[`Buffer.from(string[, encoding])`][`Buffer.from(string)`].

## `node:buffer`模块接口

而，`Buffer`对象可作为全局对象使用，还有附加的
`Buffer`- 仅通过`node:buffer`模块
访问使用`require('node:buffer')`.

### `buffer.atob(data)`

<!-- YAML
added:
  - v15.13.0
  - v14.17.0
-->

> 稳定性： 3 - 旧版。用`Buffer.from(data, 'base64')`相反。

*   `data`{任何}Base64 编码的输入字符串。

将 Base64 编码数据的字符串解码为字节，并对这些字节进行编码
使用拉丁语-1 （ISO-8859-1） 转换为字符串。

这`data`可以是任何可以强制转换为字符串的 JavaScript 值。

**提供此功能只是为了与旧版 Web 平台 API 兼容
并且永远不应该在新代码中使用，因为它们使用字符串来表示
二进制数据，并且早于JavaScript中类型化数组的引入。
对于使用 Node.js API 运行的代码，在 base64 编码的字符串之间进行转换
和二进制数据应使用`Buffer.from(str, 'base64')`和
`buf.toString('base64')`.**

### `buffer.btoa(data)`

<!-- YAML
added:
  - v15.13.0
  - v14.17.0
-->

> 稳定性： 3 - 旧版。用`buf.toString('base64')`相反。

*   `data`{任何}一个 ASCII（拉丁语 1）字符串。

使用 Latin-1 （ISO-8859） 将字符串解码为字节，并对这些字节进行编码
使用 Base64 转换为字符串。

这`data`可以是任何可以强制转换为字符串的 JavaScript 值。

**提供此功能只是为了与旧版 Web 平台 API 兼容
并且永远不应该在新代码中使用，因为它们使用字符串来表示
二进制数据，并且早于JavaScript中类型化数组的引入。
对于使用 Node.js API 运行的代码，在 base64 编码的字符串之间进行转换
和二进制数据应使用`Buffer.from(str, 'base64')`和
`buf.toString('base64')`.**

### `buffer.INSPECT_MAX_BYTES`

<!-- YAML
added: v0.5.4
-->

*   {整数}**违约：** `50`

返回在以下情况下将返回的最大字节数
`buf.inspect()`被调用。这可以被用户模块覆盖。看
[`util.inspect()`][util.inspect()]有关更多详细信息`buf.inspect()`行为。

### `buffer.kMaxLength`

<!-- YAML
added: v3.0.0
-->

*   {整数}单个允许的最大尺寸`Buffer`实例。

的别名[`buffer.constants.MAX_LENGTH`][buffer.constants.MAX_LENGTH].

### `buffer.kStringMaxLength`

<!-- YAML
added: v3.0.0
-->

*   {整数}单个允许的最大长度`string`实例。

的别名[`buffer.constants.MAX_STRING_LENGTH`][buffer.constants.MAX_STRING_LENGTH].

### `buffer.resolveObjectURL(id)`

<!-- YAML
added: v16.7.0
-->

> 稳定性： 1 - 实验

*   `id`{字符串}一个`'blob:nodedata:...`由先前调用返回的 URL 字符串
    `URL.createObjectURL()`.
*   返回： {Blob}

解析`'blob:nodedata:...'`使用 注册的关联 {Blob} 对象
事先致电`URL.createObjectURL()`.

### `buffer.transcode(source, fromEnc, toEnc)`

<!-- YAML
added: v7.1.0
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `source` parameter can now be a `Uint8Array`.
-->

*   `source`{缓冲区|Uint8Array} A`Buffer`或`Uint8Array`实例。
*   `fromEnc`{字符串}当前编码。
*   `toEnc`{字符串}目标编码。
*   返回：{缓冲区}

重新编码给定的`Buffer`或`Uint8Array`一个字符的实例
编码到另一个。返回新的`Buffer`实例。

如果`fromEnc`或`toEnc`指定无效字符编码，或者如果
转换自`fromEnc`自`toEnc`是不允许的。

支持的编码`buffer.transcode()`是：`'ascii'`,`'utf8'`,
`'utf16le'`,`'ucs2'`,`'latin1'`和`'binary'`.

如果给定字节，转码过程将使用替换字符
序列不能在目标编码中充分表示。例如：

```mjs
import { Buffer, transcode } from 'node:buffer';

const newBuf = transcode(Buffer.from('€'), 'utf8', 'ascii');
console.log(newBuf.toString('ascii'));
// Prints: '?'
```

```cjs
const { Buffer, transcode } = require('node:buffer');

const newBuf = transcode(Buffer.from('€'), 'utf8', 'ascii');
console.log(newBuf.toString('ascii'));
// Prints: '?'
```

因为欧元（`€`） 符号在 US-ASCII 中不可表示，它被替换
跟`?`在转码后`Buffer`.

### 类：`SlowBuffer`

<!-- YAML
deprecated: v6.0.0
-->

> 稳定性：0 - 已弃用：使用[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()]相反。

看[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()].这从来都不是一门课，从某种意义上说
构造函数始终返回一个`Buffer`实例，而不是`SlowBuffer`
实例。

#### `new SlowBuffer(size)`

<!-- YAML
deprecated: v6.0.0
-->

> 稳定性：0 - 已弃用：使用[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()]相反。

*   `size`{整数}新品的所需长度`SlowBuffer`.

看[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()].

### 缓冲区常量

<!-- YAML
added: v8.2.0
-->

#### `buffer.constants.MAX_LENGTH`

<!-- YAML
added: v8.2.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35415
    description: Value is changed to 2<sup>32</sup> on 64-bit
      architectures.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/32116
    description: Value is changed from 2<sup>31</sup> - 1 to
      2<sup>32</sup> - 1 on 64-bit architectures.
-->

*   {整数}单个允许的最大尺寸`Buffer`实例。

在 32 位体系结构上，此值当前为 2<sup>30</sup>- 1（约1
GiB）。

在 64 位体系结构上，此值当前为 2<sup>32</sup>（约 4 GiB）。

它反映了[`v8::TypedArray::kMaxLength`][v8::TypedArray::kMaxLength]在引擎盖下。

此值也可用作[`buffer.kMaxLength`][buffer.kMaxLength].

#### `buffer.constants.MAX_STRING_LENGTH`

<!-- YAML
added: v8.2.0
-->

*   {整数}单个允许的最大长度`string`实例。

代表最大的`length`那一个`string`原语可以有，数
以 UTF-16 代码单位表示。

此值可能取决于正在使用的 JS 引擎。

## `Buffer.from()`,`Buffer.alloc()`和`Buffer.allocUnsafe()`

在 Node.js 6.0.0 之前的版本中，`Buffer`实例是使用
`Buffer`构造函数，用于分配返回的`Buffer`
根据提供的参数的不同：

*   将数字作为第一个参数传递给`Buffer()`（例如`new Buffer(10)`)
    分配新的`Buffer`指定大小的对象。在节点之前.js 8.0.0，
    为此类分配的内存`Buffer`实例是*不*初始化和
    *可以包含敏感数据*.这样`Buffer`实例*必须*随后
    通过使用[`buf.fill(0)`][`buf.fill()`]或写信至
    整个`Buffer`在从`Buffer`.
    虽然此行为是*故意*提高性能，
    发展经验表明，更明确的区别是
    在创建快速但未初始化之间需要`Buffer`与创建
    速度较慢但更安全`Buffer`.自节点.js 8.0.0，`Buffer(num)`和`new
    Buffer(num)`返回`Buffer`具有初始化的内存。
*   传递字符串、数组或`Buffer`因为第一个参数复制
    将对象的数据传递到`Buffer`.
*   通过[`ArrayBuffer`][ArrayBuffer]或[`SharedArrayBuffer`][SharedArrayBuffer]返回`Buffer`
    与给定数组缓冲区共享分配的内存。

因为的行为`new Buffer()`根据的类型而有所不同
第一个论点，安全性和可靠性问题可能会无意中引入
在参数验证或`Buffer`初始化不是
执行。

例如，如果攻击者可以使应用程序接收到一个数字，其中
一个字符串是预期的，应用程序可以调用`new Buffer(100)`
而不是`new Buffer("100")`，导致它分配一个 100 字节的缓冲区
分配包含内容的 3 字节缓冲区`"100"`.这通常是可能的
使用 JSON API 调用。由于 JSON 区分数字类型和字符串类型，
它允许注入数字，其中天真地编写的应用程序没有
充分验证其输入，可能期望始终收到字符串。
在 Node.js 8.0.0 之前，100 字节缓冲区可能包含
任意预先存在的内存中数据，因此可用于公开内存中
远程攻击者的机密。由于 Node.js 8.0.0，因此无法公开内存
发生是因为数据已填充零。但是，其他攻击仍然是
可能，例如导致服务器分配非常大的缓冲区，
导致性能下降或在内存耗尽时崩溃。

要使创建`Buffer`实例更可靠，更不容易出错，
各种形式的`new Buffer()`构造函数已**荒废的**
并替换为单独的`Buffer.from()`,[`Buffer.alloc()`][Buffer.alloc()]和
[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]方法。

*开发人员应迁移 所有现有用途`new Buffer()`构造 函数
添加到这些新 API 之一。*

*   [`Buffer.from(array)`][Buffer.from(array)]返回新的`Buffer`那*包含副本*的
    提供八位字节。
*   [`Buffer.from(arrayBuffer[, byteOffset[, length]])`][`Buffer.from(arrayBuf)`]
    返回新的`Buffer`那*共享相同的已分配内存*作为给定的
    [`ArrayBuffer`][ArrayBuffer].
*   [`Buffer.from(buffer)`][Buffer.from(buffer)]返回新的`Buffer`那*包含副本*的
    给定内容`Buffer`.
*   [`Buffer.from(string[, encoding])`][`Buffer.from(string)`]返回新的
    `Buffer`那*包含副本*提供的字符串。
*   [`Buffer.alloc(size[, fill[, encoding]])`][`Buffer.alloc()`]返回新的
    初始 化`Buffer`指定大小。此方法比
    [`Buffer.allocUnsafe(size)`][`Buffer.allocUnsafe()`]但保证新
    创建`Buffer`实例从不包含可能的旧数据
    灵敏。一个`TypeError`将在以下情况下被抛出`size`不是一个数字。
*   [`Buffer.allocUnsafe(size)`][`Buffer.allocUnsafe()`]和
    [`Buffer.allocUnsafeSlow(size)`][`Buffer.allocUnsafeSlow()`]每次返回一个
    新的未初始化`Buffer`的指定`size`.因为`Buffer`是
    未初始化，分配的内存段可能包含旧数据
    潜在敏感。

`Buffer`返回的实例[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]和
[`Buffer.from(array)`][Buffer.from(array)] *五月*从共享内部存储器池中分配
如果`size`小于或等于一半[`Buffer.poolSize`][Buffer.poolSize].实例
返回者[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()] *从不*使用共享的内部
内存池。

### 这`--zero-fill-buffers`命令行选项

<!-- YAML
added: v5.10.0
-->

节点.js可以使用`--zero-fill-buffers`命令行选项
导致所有新分配`Buffer`创建时要由 零填充的实例
违约。如果没有该选项，则使用[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()],
[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()]和`new SlowBuffer(size)`不是零填充的。
使用此标志可能会对性能产生可衡量的负面影响。使用
`--zero-fill-buffers`仅当有必要强制执行新分配时才选择
`Buffer`实例不能包含可能敏感的旧数据。

```console
$ node --zero-fill-buffers
> Buffer.allocUnsafe(5);
<Buffer 00 00 00 00 00>
```

### 什么使`Buffer.allocUnsafe()`和`Buffer.allocUnsafeSlow()`“不安全”？

呼叫时[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]和[`Buffer.allocUnsafeSlow()`][Buffer.allocUnsafeSlow()]这
分配的内存段为*初始 化*（它不是归零的）。而
这种设计使得内存的分配相当快，分配的段
内存可能包含可能敏感的旧数据。使用`Buffer`
创建者[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]没有*完全*覆盖
内存可以允许此旧数据泄漏时`Buffer`内存被读取。

虽然使用有明显的性能优势
[`Buffer.allocUnsafe()`][Buffer.allocUnsafe()]，格外小心*必须*采取以避免
将安全漏洞引入应用程序。

[ASCII]: https://en.wikipedia.org/wiki/ASCII

[Base64]: https://en.wikipedia.org/wiki/Base64

[ISO-8859-1]: https://en.wikipedia.org/wiki/ISO-8859-1

[RFC 4648, Section 5]: https://tools.ietf.org/html/rfc4648#section-5

[UTF-16]: https://en.wikipedia.org/wiki/UTF-16

[UTF-8]: https://en.wikipedia.org/wiki/UTF-8

[WHATWG Encoding Standard]: https://encoding.spec.whatwg.org/

[`ArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

[`Blob`]: https://developer.mozilla.org/en-US/docs/Web/API/Blob

[`Buffer.alloc()`]: #static-method-bufferallocsize-fill-encoding

[`Buffer.allocUnsafe()`]: #static-method-bufferallocunsafesize

[`Buffer.allocUnsafeSlow()`]: #static-method-bufferallocunsafeslowsize

[`Buffer.concat()`]: #static-method-bufferconcatlist-totallength

[`Buffer.from(array)`]: #static-method-bufferfromarray

[`Buffer.from(arrayBuf)`]: #static-method-bufferfromarraybuffer-byteoffset-length

[`Buffer.from(buffer)`]: #static-method-bufferfrombuffer

[`Buffer.from(string)`]: #static-method-bufferfromstring-encoding

[`Buffer.poolSize`]: #class-property-bufferpoolsize

[`DataView`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView

[`ERR_INVALID_ARG_VALUE`]: errors.md#err_invalid_arg_value

[`ERR_INVALID_BUFFER_SIZE`]: errors.md#err_invalid_buffer_size

[`ERR_OUT_OF_RANGE`]: errors.md#err_out_of_range

[`JSON.stringify()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

[`SharedArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer

[`String.prototype.indexOf()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf

[`String.prototype.lastIndexOf()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/lastIndexOf

[`String.prototype.length`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/length

[`TypedArray.from()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/from

[`TypedArray.prototype.set()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/set

[`TypedArray.prototype.slice()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/slice

[`TypedArray.prototype.subarray()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/subarray

[`TypedArray`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray

[`Uint8Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array

[`buf.buffer`]: #bufbuffer

[`buf.compare()`]: #bufcomparetarget-targetstart-targetend-sourcestart-sourceend

[`buf.entries()`]: #bufentries

[`buf.fill()`]: #buffillvalue-offset-end-encoding

[`buf.indexOf()`]: #bufindexofvalue-byteoffset-encoding

[`buf.keys()`]: #bufkeys

[`buf.length`]: #buflength

[`buf.slice()`]: #bufslicestart-end

[`buf.subarray`]: #bufsubarraystart-end

[`buf.toString()`]: #buftostringencoding-start-end

[`buf.values()`]: #bufvalues

[`buffer.constants.MAX_LENGTH`]: #bufferconstantsmax_length

[`buffer.constants.MAX_STRING_LENGTH`]: #bufferconstantsmax_string_length

[`buffer.kMaxLength`]: #bufferkmaxlength

[`util.inspect()`]: util.md#utilinspectobject-options

[`v8::TypedArray::kMaxLength`]: https://v8.github.io/api/head/classv8_1_1TypedArray.html#a54a48f4373da0850663c4393d843b9b0

[base64url]: https://tools.ietf.org/html/rfc4648#section-5

[binary strings]: https://developer.mozilla.org/en-US/docs/Web/API/DOMString/Binary

[endianness]: https://en.wikipedia.org/wiki/Endianness

[iterator]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
