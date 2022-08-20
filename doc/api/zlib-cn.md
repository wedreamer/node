# 兹利布

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/zlib.js -->

这`node:zlib`模块提供使用
Gzip，Deflate/Inflate和Brotli。

要访问它：

```js
const zlib = require('node:zlib');
```

压缩和解压缩是围绕节点构建的.js[Streams API][].

压缩或解压缩流（如文件）可以通过以下方式完成
通过管道将源流通过`zlib` `Transform`流式传输到目标
流：

```js
const { createGzip } = require('node:zlib');
const { pipeline } = require('node:stream');
const {
  createReadStream,
  createWriteStream
} = require('node:fs');

const gzip = createGzip();
const source = createReadStream('input.txt');
const destination = createWriteStream('input.txt.gz');

pipeline(source, gzip, destination, (err) => {
  if (err) {
    console.error('An error occurred:', err);
    process.exitCode = 1;
  }
});

// Or, Promisified

const { promisify } = require('node:util');
const pipe = promisify(pipeline);

async function do_gzip(input, output) {
  const gzip = createGzip();
  const source = createReadStream(input);
  const destination = createWriteStream(output);
  await pipe(source, gzip, destination);
}

do_gzip('input.txt', 'input.txt.gz')
  .catch((err) => {
    console.error('An error occurred:', err);
    process.exitCode = 1;
  });
```

也可以在单个步骤中压缩或解压缩数据：

```js
const { deflate, unzip } = require('node:zlib');

const input = '.................................';
deflate(input, (err, buffer) => {
  if (err) {
    console.error('An error occurred:', err);
    process.exitCode = 1;
  }
  console.log(buffer.toString('base64'));
});

const buffer = Buffer.from('eJzT0yMAAGTvBe8=', 'base64');
unzip(buffer, (err, buffer) => {
  if (err) {
    console.error('An error occurred:', err);
    process.exitCode = 1;
  }
  console.log(buffer.toString());
});

// Or, Promisified

const { promisify } = require('node:util');
const do_unzip = promisify(unzip);

do_unzip(buffer)
  .then((buf) => console.log(buf.toString()))
  .catch((err) => {
    console.error('An error occurred:', err);
    process.exitCode = 1;
  });
```

## 线程池使用和性能注意事项

都`zlib`除了那些显式同步的 API 之外，它们都使用 Node.js
内部线程池。这可以带来令人惊讶的效果和性能
某些应用程序中的限制。

同时创建和使用大量 zlib 对象可能会导致
严重的内存碎片。

```js
const zlib = require('node:zlib');

const payload = Buffer.from('This is some data');

// WARNING: DO NOT DO THIS!
for (let i = 0; i < 30000; ++i) {
  zlib.deflate(payload, (err, buffer) => {});
}
```

在前面的示例中，同时创建了 30000 个 deflate 实例。
由于某些操作系统如何处理内存分配和
解除分配，这可能会导致严重的内存碎片。

强烈建议压缩的结果
缓存操作以避免重复工作。

## 压缩 HTTP 请求和响应

这`node:zlib`模块可用于实现对`gzip`,`deflate`
和`br`内容编码机制由 定义
[断续器](https://tools.ietf.org/html/rfc7230#section-4.2).

The HTTP[`Accept-Encoding`][Accept-Encoding]标头在 HTTP 请求中用于标识
客户端接受的压缩编码。这[`Content-Encoding`][Content-Encoding]
header 用于标识实际应用于
消息。

下面给出的示例经过了极大的简化，以显示基本概念。
用`zlib`编码可能很昂贵，并且应该缓存结果。
看[内存使用情况调整][Memory usage tuning]有关速度/内存/压缩的更多信息
涉及的权衡`zlib`用法。

```js
// Client request example
const zlib = require('node:zlib');
const http = require('node:http');
const fs = require('node:fs');
const { pipeline } = require('node:stream');

const request = http.get({ host: 'example.com',
                           path: '/',
                           port: 80,
                           headers: { 'Accept-Encoding': 'br,gzip,deflate' } });
request.on('response', (response) => {
  const output = fs.createWriteStream('example.com_index.html');

  const onError = (err) => {
    if (err) {
      console.error('An error occurred:', err);
      process.exitCode = 1;
    }
  };

  switch (response.headers['content-encoding']) {
    case 'br':
      pipeline(response, zlib.createBrotliDecompress(), output, onError);
      break;
    // Or, just use zlib.createUnzip() to handle both of the following cases:
    case 'gzip':
      pipeline(response, zlib.createGunzip(), output, onError);
      break;
    case 'deflate':
      pipeline(response, zlib.createInflate(), output, onError);
      break;
    default:
      pipeline(response, output, onError);
      break;
  }
});
```

```js
// server example
// Running a gzip operation on every request is quite expensive.
// It would be much more efficient to cache the compressed buffer.
const zlib = require('node:zlib');
const http = require('node:http');
const fs = require('node:fs');
const { pipeline } = require('node:stream');

http.createServer((request, response) => {
  const raw = fs.createReadStream('index.html');
  // Store both a compressed and an uncompressed version of the resource.
  response.setHeader('Vary', 'Accept-Encoding');
  let acceptEncoding = request.headers['accept-encoding'];
  if (!acceptEncoding) {
    acceptEncoding = '';
  }

  const onError = (err) => {
    if (err) {
      // If an error occurs, there's not much we can do because
      // the server has already sent the 200 response code and
      // some amount of data has already been sent to the client.
      // The best we can do is terminate the response immediately
      // and log the error.
      response.end();
      console.error('An error occurred:', err);
    }
  };

  // Note: This is not a conformant accept-encoding parser.
  // See https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3
  if (/\bdeflate\b/.test(acceptEncoding)) {
    response.writeHead(200, { 'Content-Encoding': 'deflate' });
    pipeline(raw, zlib.createDeflate(), response, onError);
  } else if (/\bgzip\b/.test(acceptEncoding)) {
    response.writeHead(200, { 'Content-Encoding': 'gzip' });
    pipeline(raw, zlib.createGzip(), response, onError);
  } else if (/\bbr\b/.test(acceptEncoding)) {
    response.writeHead(200, { 'Content-Encoding': 'br' });
    pipeline(raw, zlib.createBrotliCompress(), response, onError);
  } else {
    response.writeHead(200, {});
    pipeline(raw, response, onError);
  }
}).listen(1337);
```

默认情况下，`zlib`方法在解压缩时会抛出错误
截断的数据。但是，如果已知数据不完整，或者
愿望是只检查压缩文件的开头，它是
可以通过更改刷新来抑制默认的错误处理
用于解压缩最后一个输入数据块的方法：

```js
// This is a truncated version of the buffer from the above examples
const buffer = Buffer.from('eJzT0yMA', 'base64');

zlib.unzip(
  buffer,
  // For Brotli, the equivalent is zlib.constants.BROTLI_OPERATION_FLUSH.
  { finishFlush: zlib.constants.Z_SYNC_FLUSH },
  (err, buffer) => {
    if (err) {
      console.error('An error occurred:', err);
      process.exitCode = 1;
    }
    console.log(buffer.toString());
  });
```

这不会改变其他错误引发情况下的行为，例如
当输入数据的格式无效时。使用此方法，它不会是
可以确定输入是过早结束还是缺少
完整性检查，使得有必要手动检查
解压缩结果有效。

## 内存使用情况调整

<!--type=misc-->

### 对于基于 zlib 的流

从`zlib/zconf.h`，针对节点.js用法进行了修改：

压缩的内存要求为（以字节为单位）：

<!-- eslint-disable semi -->

```js
(1 << (windowBits + 2)) + (1 << (memLevel + 9))
```

即：128K 用于`windowBits`= 15 + 128K`memLevel`= 8
（默认值）加上小对象的几千字节。

例如，要将默认内存要求从 256K 降低到 128K，则
选项应设置为：

```js
const options = { windowBits: 14, memLevel: 7 };
```

但是，这通常会降低压缩性能。

膨胀的内存要求为（以字节为单位）`1 << windowBits`.
也就是说，32K 用于`windowBits`= 15（默认值）加上几千字节
对于小物体。

这是对单个内部输出板坯缓冲器的补充
`chunkSize`，默认值为 16K。

速度`zlib`压缩受
`level`设置。较高的级别将导致更好的压缩，但是
需要更长的时间才能完成。水平越低，越少
压缩，但会快得多。

通常，更大的内存使用选项将意味着Node.js必须使
更少的呼叫`zlib`因为它将能够处理更多的数据
每`write`操作。因此，这是影响
速度，以牺牲内存使用为代价。

### 对于基于 Brotli 的流

对于基于 Brotli 的流，有与 zlib 选项等效的等效项，尽管
这些选项的范围与 zlib 选项不同：

*   兹利布`level`选项匹配布罗特利`BROTLI_PARAM_QUALITY`选择。
*   兹利布`windowBits`选项匹配布罗特利`BROTLI_PARAM_LGWIN`选择。

看[下面][Brotli parameters]有关 Brotli 特定选项的更多详细信息。

## 冲洗

叫[`.flush()`][.flush()]在压缩流上会使`zlib`回报率
当前可能输出。这可能以压缩降级为代价。
质量，但在需要尽快提供数据时可能很有用。

在以下示例中，`flush()`用于写入压缩部分
对客户端的 HTTP 响应：

```js
const zlib = require('node:zlib');
const http = require('node:http');
const { pipeline } = require('node:stream');

http.createServer((request, response) => {
  // For the sake of simplicity, the Accept-Encoding checks are omitted.
  response.writeHead(200, { 'content-encoding': 'gzip' });
  const output = zlib.createGzip();
  let i;

  pipeline(output, response, (err) => {
    if (err) {
      // If an error occurs, there's not much we can do because
      // the server has already sent the 200 response code and
      // some amount of data has already been sent to the client.
      // The best we can do is terminate the response immediately
      // and log the error.
      clearInterval(i);
      response.end();
      console.error('An error occurred:', err);
    }
  });

  i = setInterval(() => {
    output.write(`The current time is ${Date()}\n`, () => {
      // The data has been passed to zlib, but the compression algorithm may
      // have decided to buffer the data for more efficient compression.
      // Calling .flush() will make the data available as soon as the client
      // is ready to receive it.
      output.flush();
    });
  }, 1000);
}).listen(1337);
```

## 常数

<!-- YAML
added: v0.5.8
-->

<!--type=misc-->

### zlib 常量

中定义的所有常量`zlib.h`也定义在
`require('node:zlib').constants`.在正常的操作过程中，它将
没有必要使用这些常量。它们被记录下来，以便他们的
存在并不奇怪。本节几乎直接取自
[zlib 文档][zlib documentation].

以前，常量可直接从`require('node:zlib')`,
例如`zlib.Z_NO_FLUSH`.直接从模块访问常量
目前仍然可行，但已弃用。

允许的刷新值。

*   `zlib.constants.Z_NO_FLUSH`
*   `zlib.constants.Z_PARTIAL_FLUSH`
*   `zlib.constants.Z_SYNC_FLUSH`
*   `zlib.constants.Z_FULL_FLUSH`
*   `zlib.constants.Z_FINISH`
*   `zlib.constants.Z_BLOCK`
*   `zlib.constants.Z_TREES`

压缩/解压缩函数的返回代码。阴性
值是错误，正值用于特殊但正常的
事件。

*   `zlib.constants.Z_OK`
*   `zlib.constants.Z_STREAM_END`
*   `zlib.constants.Z_NEED_DICT`
*   `zlib.constants.Z_ERRNO`
*   `zlib.constants.Z_STREAM_ERROR`
*   `zlib.constants.Z_DATA_ERROR`
*   `zlib.constants.Z_MEM_ERROR`
*   `zlib.constants.Z_BUF_ERROR`
*   `zlib.constants.Z_VERSION_ERROR`

压缩级别。

*   `zlib.constants.Z_NO_COMPRESSION`
*   `zlib.constants.Z_BEST_SPEED`
*   `zlib.constants.Z_BEST_COMPRESSION`
*   `zlib.constants.Z_DEFAULT_COMPRESSION`

压缩策略。

*   `zlib.constants.Z_FILTERED`
*   `zlib.constants.Z_HUFFMAN_ONLY`
*   `zlib.constants.Z_RLE`
*   `zlib.constants.Z_FIXED`
*   `zlib.constants.Z_DEFAULT_STRATEGY`

### 布罗特利常数

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

有几个选项和其他常量可用于基于 Brotli 的
流：

#### 冲洗操作

以下值是基于 Brotli 的流的有效刷新操作：

*   `zlib.constants.BROTLI_OPERATION_PROCESS`（所有操作的默认值）
*   `zlib.constants.BROTLI_OPERATION_FLUSH`（呼叫时默认`.flush()`)
*   `zlib.constants.BROTLI_OPERATION_FINISH`（最后一个区块的默认值）
*   `zlib.constants.BROTLI_OPERATION_EMIT_METADATA`
    *   此特定操作可能难以在 Node.js 上下文中使用，
        因为流层很难知道哪些数据最终会结束
        在这个框架中。此外，目前没有办法通过以下方式使用此数据
        节点.js API。

#### 压缩机选件

可以在 Brotli 编码器上设置多个选项，从而影响
压缩效率和速度。键和值都可以访问
作为 的属性`zlib.constants`对象。

最重要的选项是：

*   `BROTLI_PARAM_MODE`
    *   `BROTLI_MODE_GENERIC`（默认值）
    *   `BROTLI_MODE_TEXT`，针对 UTF-8 文本进行了调整
    *   `BROTLI_MODE_FONT`，针对 WOFF 2.0 字体进行了调整
*   `BROTLI_PARAM_QUALITY`
    *   范围从`BROTLI_MIN_QUALITY`自`BROTLI_MAX_QUALITY`,
        默认值为`BROTLI_DEFAULT_QUALITY`.
*   `BROTLI_PARAM_SIZE_HINT`
    *   表示预期输入大小的整数值;
        默认为`0`对于未知的输入大小。

可以设置以下标志以对压缩进行高级控制
算法和内存使用调整：

*   `BROTLI_PARAM_LGWIN`
    *   范围从`BROTLI_MIN_WINDOW_BITS`自`BROTLI_MAX_WINDOW_BITS`,
        默认值为`BROTLI_DEFAULT_WINDOW`，或最多
        `BROTLI_LARGE_MAX_WINDOW_BITS`如果`BROTLI_PARAM_LARGE_WINDOW`旗
        已设置。
*   `BROTLI_PARAM_LGBLOCK`
    *   范围从`BROTLI_MIN_INPUT_BLOCK_BITS`自`BROTLI_MAX_INPUT_BLOCK_BITS`.
*   `BROTLI_PARAM_DISABLE_LITERAL_CONTEXT_MODELING`
    *   降低压缩比以支持
        减压速度。
*   `BROTLI_PARAM_LARGE_WINDOW`
    *   启用“大窗口 Brotli”模式的布尔标志（与
        Brotli 格式标准化于[RFC 7932][]).
*   `BROTLI_PARAM_NPOSTFIX`
    *   范围从`0`自`BROTLI_MAX_NPOSTFIX`.
*   `BROTLI_PARAM_NDIRECT`
    *   范围从`0`自`15 << NPOSTFIX`在步骤中`1 << NPOSTFIX`.

#### 解压缩程序选项

这些高级选项可用于控制解压缩：

*   `BROTLI_DECODER_PARAM_DISABLE_RING_BUFFER_REALLOCATION`
    *   影响内部内存分配模式的布尔标志。
*   `BROTLI_DECODER_PARAM_LARGE_WINDOW`
    *   启用“大窗口 Brotli”模式的布尔标志（与
        Brotli 格式标准化于[RFC 7932][]).

## 类：`Options`

<!-- YAML
added: v0.11.1
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33516
    description: The `maxOutputLength` option is supported now.
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `dictionary` option can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `dictionary` option can be an `Uint8Array` now.
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/6069
    description: The `finishFlush` option is supported now.
-->

<!--type=misc-->

每个基于 zlib 的类都采用`options`对象。不需要任何选项。

某些选项仅在压缩时才相关，并且
被解压缩类忽略。

*   `flush`{整数}**违约：** `zlib.constants.Z_NO_FLUSH`
*   `finishFlush`{整数}**违约：** `zlib.constants.Z_FINISH`
*   `chunkSize`{整数}**违约：** `16 * 1024`
*   `windowBits`{整数}
*   `level`{整数}（仅压缩）
*   `memLevel`{整数}（仅压缩）
*   `strategy`{整数}（仅压缩）
*   `dictionary`{缓冲区|TypedArray|数据视图|ArrayBuffer} （仅放气/充气，
    默认为空字典）
*   `info`{布尔值}（如果`true`，返回一个对象`buffer`和`engine`.)
*   `maxOutputLength`{整数}使用时限制输出大小
    [方便的方法][convenience methods].**违约：** [`buffer.kMaxLength`][buffer.kMaxLength]

查看[`deflateInit2`和`inflateInit2`][deflateInit2 and inflateInit2]文档以获取更多信息
信息。

## 类：`BrotliOptions`

<!-- YAML
added: v11.7.0
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33516
    description: The `maxOutputLength` option is supported now.
-->

<!--type=misc-->

每个基于Brotli的课程都采用`options`对象。所有选项都是可选的。

*   `flush`{整数}**违约：** `zlib.constants.BROTLI_OPERATION_PROCESS`
*   `finishFlush`{整数}**违约：** `zlib.constants.BROTLI_OPERATION_FINISH`
*   `chunkSize`{整数}**违约：** `16 * 1024`
*   `params`{对象}包含索引的键值对象[布罗特利参数][Brotli parameters].
*   `maxOutputLength`{整数}使用时限制输出大小
    [方便的方法][convenience methods].**违约：** [`buffer.kMaxLength`][buffer.kMaxLength]

例如：

```js
const stream = zlib.createBrotliCompress({
  chunkSize: 32 * 1024,
  params: {
    [zlib.constants.BROTLI_PARAM_MODE]: zlib.constants.BROTLI_MODE_TEXT,
    [zlib.constants.BROTLI_PARAM_QUALITY]: 4,
    [zlib.constants.BROTLI_PARAM_SIZE_HINT]: fs.statSync(inputFile).size
  }
});
```

## 类：`zlib.BrotliCompress`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

使用 Brotli 算法压缩数据。

## 类：`zlib.BrotliDecompress`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

使用 Brotli 算法解压缩数据。

## 类：`zlib.Deflate`

<!-- YAML
added: v0.5.8
-->

使用放气压缩数据。

## 类：`zlib.DeflateRaw`

<!-- YAML
added: v0.5.8
-->

使用放气压缩压缩数据，并且不追加`zlib`页眉。

## 类：`zlib.Gunzip`

<!-- YAML
added: v0.5.8
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5883
    description: Trailing garbage at the end of the input stream will now
                 result in an `'error'` event.
  - version: v5.9.0
    pr-url: https://github.com/nodejs/node/pull/5120
    description: Multiple concatenated gzip file members are supported now.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2595
    description: A truncated input stream will now result in an `'error'` event.
-->

解压缩 gzip 流。

## 类：`zlib.Gzip`

<!-- YAML
added: v0.5.8
-->

使用 gzip 压缩数据。

## 类：`zlib.Inflate`

<!-- YAML
added: v0.5.8
changes:
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2595
    description: A truncated input stream will now result in an `'error'` event.
-->

解压缩放气流。

## 类：`zlib.InflateRaw`

<!-- YAML
added: v0.5.8
changes:
  - version: v6.8.0
    pr-url: https://github.com/nodejs/node/pull/8512
    description: Custom dictionaries are now supported by `InflateRaw`.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2595
    description: A truncated input stream will now result in an `'error'` event.
-->

解压缩原始放气流。

## 类：`zlib.Unzip`

<!-- YAML
added: v0.5.8
-->

通过自动检测来解压缩 Gzip 或 Deflate 压缩流
标头。

## 类：`zlib.ZlibBase`

<!-- YAML
added: v0.5.8
changes:
  - version:
     - v11.7.0
     - v10.16.0
    pr-url: https://github.com/nodejs/node/pull/24939
    description: This class was renamed from `Zlib` to `ZlibBase`.
-->

未由`node:zlib`模块。它记录在这里，因为它是
压缩器/解压缩器类的基类。

此类继承自[`stream.Transform`][stream.Transform]允许`node:zlib`对象
用于管道和类似的流操作。

### `zlib.bytesRead`

<!-- YAML
added: v8.1.0
deprecated: v10.0.0
-->

> 稳定性：0 - 已弃用：使用[`zlib.bytesWritten`][zlib.bytesWritten]相反。

*   {数字}

已弃用的别名[`zlib.bytesWritten`][zlib.bytesWritten].选择此原始名称
因为将值解释为字节数也是有意义的
由引擎读取，但与 Node 中的其他流不一致.js
公开这些名称下的值。

### `zlib.bytesWritten`

<!-- YAML
added: v10.0.0
-->

*   {数字}

这`zlib.bytesWritten`属性指定写入的字节数
引擎，在处理字节之前（压缩或解压缩，
适用于派生类）。

### `zlib.close([callback])`

<!-- YAML
added: v0.9.4
-->

*   `callback`{函数}

关闭基础句柄。

### `zlib.flush([kind, ]callback)`

<!-- YAML
added: v0.5.8
-->

*   `kind` **违约：** `zlib.constants.Z_FULL_FLUSH`对于基于 zlib 的流，
    `zlib.constants.BROTLI_OPERATION_FLUSH`对于基于 Brotli 的流。
*   `callback`{函数}

刷新挂起的数据。不要轻率地称之为，过早的冲洗是负面的
影响压缩算法的有效性。

调用 this 只会刷新内部数据`zlib`状态，并且不
在流级别执行任何类型的刷新。相反，它的行为就像一个
正常调用`.write()`，即它将排在其他待处理之后
写入，并且仅在从流中读取数据时生成输出。

### `zlib.params(level, strategy, callback)`

<!-- YAML
added: v0.11.4
-->

*   `level`{整数}
*   `strategy`{整数}
*   `callback`{函数}

此功能仅适用于基于 zlib 的流，即不适用于 Brotli。

动态更新压缩级别和压缩策略。
仅适用于放气算法。

### `zlib.reset()`

<!-- YAML
added: v0.7.0
-->

将压缩器/解压缩器重置为出厂默认值。仅适用于
膨胀和放气算法。

## `zlib.constants`

<!-- YAML
added: v7.0.0
-->

提供一个枚举 Zlib 相关常量的对象。

## `zlib.createBrotliCompress([options])`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

*   `options`{brotli options}

创建并返回新的[`BrotliCompress`][BrotliCompress]对象。

## `zlib.createBrotliDecompress([options])`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

*   `options`{brotli options}

创建并返回新的[`BrotliDecompress`][BrotliDecompress]对象。

## `zlib.createDeflate([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`Deflate`][Deflate]对象。

## `zlib.createDeflateRaw([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`DeflateRaw`][DeflateRaw]对象。

zlib 从 1.2.8 升级到 1.2.11 时行为已更改`windowBits`
对于原始放气流，设置为 8。zlib 会自动设置`windowBits`
如果最初设置为 8，则为 9。较新版本的 zlib 将引发异常，
所以Node.js恢复了将值8升级到9的原始行为，
自通过`windowBits = 9`到 zlib 实际上会导致压缩流
实际上仅使用 8 位窗口。

## `zlib.createGunzip([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`Gunzip`][Gunzip]对象。

## `zlib.createGzip([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`Gzip`][Gzip]对象。
看[例][zlib.createGzip example].

## `zlib.createInflate([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`Inflate`][Inflate]对象。

## `zlib.createInflateRaw([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`InflateRaw`][InflateRaw]对象。

## `zlib.createUnzip([options])`

<!-- YAML
added: v0.5.8
-->

*   `options`{zlib options}

创建并返回新的[`Unzip`][Unzip]对象。

## 方便的方法

<!--type=misc-->

所有这些都需要一个[`Buffer`][Buffer],[`TypedArray`][TypedArray],[`DataView`][DataView],
[`ArrayBuffer`][ArrayBuffer]或字符串作为第一个参数，可选的第二个参数
为`zlib`类，并将调用提供的回调
跟`callback(error, result)`.

每个方法都有一个`*Sync`对应方，接受相同的论点，但
没有回调。

### `zlib.brotliCompress(buffer[, options], callback)`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{brotli options}
*   `callback`{函数}

### `zlib.brotliCompressSync(buffer[, options])`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{brotli options}

压缩一大块数据[`BrotliCompress`][BrotliCompress].

### `zlib.brotliDecompress(buffer[, options], callback)`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{brotli options}
*   `callback`{函数}

### `zlib.brotliDecompressSync(buffer[, options])`

<!-- YAML
added:
 - v11.7.0
 - v10.16.0
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{brotli options}

使用 解压缩数据块[`BrotliDecompress`][BrotliDecompress].

### `zlib.deflate(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.deflateSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

压缩一大块数据[`Deflate`][Deflate].

### `zlib.deflateRaw(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.deflateRawSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

压缩一大块数据[`DeflateRaw`][DeflateRaw].

### `zlib.gunzip(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.gunzipSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

使用 解压缩数据块[`Gunzip`][Gunzip].

### `zlib.gzip(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.gzipSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

压缩一大块数据[`Gzip`][Gzip].

### `zlib.inflate(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.inflateSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

使用 解压缩数据块[`Inflate`][Inflate].

### `zlib.inflateRaw(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.inflateRawSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

使用 解压缩数据块[`InflateRaw`][InflateRaw].

### `zlib.unzip(buffer[, options], callback)`

<!-- YAML
added: v0.6.0
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}
*   `callback`{函数}

### `zlib.unzipSync(buffer[, options])`

<!-- YAML
added: v0.11.12
changes:
  - version: v9.4.0
    pr-url: https://github.com/nodejs/node/pull/16042
    description: The `buffer` parameter can be an `ArrayBuffer`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `buffer` parameter can be any `TypedArray` or `DataView`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12001
    description: The `buffer` parameter can be an `Uint8Array` now.
-->

*   `buffer`{缓冲区|TypedArray|数据视图|ArrayBuffer|string}
*   `options`{zlib options}

使用 解压缩数据块[`Unzip`][Unzip].

[Brotli parameters]: #brotli-constants

[Memory usage tuning]: #memory-usage-tuning

[RFC 7932]: https://www.rfc-editor.org/rfc/rfc7932.txt

[Streams API]: stream.md

[`.flush()`]: #zlibflushkind-callback

[`Accept-Encoding`]: https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3

[`ArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

[`BrotliCompress`]: #class-zlibbrotlicompress

[`BrotliDecompress`]: #class-zlibbrotlidecompress

[`Buffer`]: buffer.md#class-buffer

[`Content-Encoding`]: https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11

[`DataView`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView

[`DeflateRaw`]: #class-zlibdeflateraw

[`Deflate`]: #class-zlibdeflate

[`Gunzip`]: #class-zlibgunzip

[`Gzip`]: #class-zlibgzip

[`InflateRaw`]: #class-zlibinflateraw

[`Inflate`]: #class-zlibinflate

[`TypedArray`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray

[`Unzip`]: #class-zlibunzip

[`buffer.kMaxLength`]: buffer.md#bufferkmaxlength

[`deflateInit2` and `inflateInit2`]: https://zlib.net/manual.html#Advanced

[`stream.Transform`]: stream.md#class-streamtransform

[`zlib.bytesWritten`]: #zlibbyteswritten

[convenience methods]: #convenience-methods

[zlib documentation]: https://zlib.net/manual.html#Constants

[zlib.createGzip example]: #zlib
