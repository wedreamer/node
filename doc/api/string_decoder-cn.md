# String decoder

<!--introduced_in=v0.10.0-->

> Stability: 2 - Stable

<!-- source_link=lib/string_decoder.js -->

`node:string_decoder` 模块提供了一个 API，用于以保留编码的多字节 UTF-8 和 UTF-16 字符的方式将 `Buffer` 对象解码为字符串。它可以使用:

```js
const { StringDecoder } = require('node:string_decoder');
```

下面的例子展示了 `StringDecoder` 类的基本使用.

```js
const { StringDecoder } = require('node:string_decoder');
const decoder = new StringDecoder('utf8');

const cent = Buffer.from([0xC2, 0xA2]);
console.log(decoder.write(cent));

const euro = Buffer.from([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro));
```

当 `Buffer` 实例写入 `StringDecoder` 实例时，内部缓冲区用于确保解码后的字符串不包含任何不完整的多字节字符。这些被保存在缓冲区中，直到下一次调用 `stringDecoder.write()` 或直到 `stringDecoder.end()` 被调用.

在以下示例中，欧洲欧元符号 (`€`) 的三个 UTF-8 编码字节被写入三个单独的操作:

```js
const { StringDecoder } = require('node:string_decoder');
const decoder = new StringDecoder('utf8');

decoder.write(Buffer.from([0xE2]));
decoder.write(Buffer.from([0x82]));
console.log(decoder.end(Buffer.from([0xAC])));
```

## Class: `StringDecoder`

### `new StringDecoder([encoding])`

<!-- YAML
added: v0.1.99
-->

* `encoding` {string} The character [encoding][] the `StringDecoder` will use.
  **Default:** `'utf8'`.

创建一个新的 `StringDecoder` 实例.

### `stringDecoder.end([buffer])`

<!-- YAML
added: v0.9.3
-->

* `buffer` {Buffer|TypedArray|DataView} A `Buffer`, or `TypedArray`, or
  `DataView` containing the bytes to decode.
* Returns: {string}

将存储在内部缓冲区中的任何剩余输入作为字符串返回。表示不完整 UTF-8 和 UTF-16 字符的字节将被替换为适合字符编码的替换字符.

如果提供了 `buffer` 参数，则在返回剩余输入之前执行最后一次调用 `stringDecoder.write()`.
调用 `end()` 后，`stringDecoder` 对象可以被重用于新的输入.

### `stringDecoder.write(buffer)`

<!-- YAML
added: v0.1.99
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9618
    description: Each invalid character is now replaced by a single replacement
                 character instead of one for each individual byte.
-->

* `buffer` {Buffer|TypedArray|DataView} A `Buffer`, or `TypedArray`, or
  `DataView` containing the bytes to decode.
* Returns: {string}

返回解码后的字符串，确保 `Buffer` 或 `TypedArray` 或 `DataView` 末尾的任何不完整的多字节字符从返回的字符串中省略，并存储在内部缓冲区中以供下次调用 `stringDecoder.write ()` 或 `stringDecoder.end()`.

[encoding]: buffer.md#buffers-and-character-encodings
