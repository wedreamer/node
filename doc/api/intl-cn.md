# 国际化支持

<!--introduced_in=v8.2.0-->

<!-- type=misc -->

Node.js具有许多功能，可以更轻松地编写国际化
程序。其中一些是：

*   中的区分区域设置或 Unicode 感知的函数[ECMAScript Language
    规范][ECMA-262]:
    *   [`String.prototype.normalize()`][String.prototype.normalize()]
    *   [`String.prototype.toLowerCase()`][String.prototype.toLowerCase()]
    *   [`String.prototype.toUpperCase()`][String.prototype.toUpperCase()]
*   中描述的所有功能[ECMAScript Internationalization API
    规范][ECMA-402]（又名 ECMA-402）：
    *   [`Intl`][Intl]对象
    *   区分区域设置的方法，如[`String.prototype.localeCompare()`][String.prototype.localeCompare()]和
        [`Date.prototype.toLocaleString()`][Date.prototype.toLocaleString()]
*   这[WHATWG URL parser][]的[国际化域名][internationalized domain names]（国际化域名） 支持
*   [`require('node:buffer').transcode()`][require('node:buffer').transcode()]
*   更准确[REPL][]行编辑
*   [`require('node:util').TextDecoder`][require('node:util').TextDecoder]
*   [`RegExp`Unicode 属性转义][RegExp Unicode Property Escapes]

节点.js和底层 V8 引擎的使用
[Unicode （ICU） 的国际组件][ICU]实现这些功能
在本机 C/C++ 代码中。默认情况下，完整的 ICU 数据集由 Node.js提供。
但是，由于ICU数据文件的大小，有几个
在以下任一情况下，提供了用于自定义 ICU 数据集的选项
构建或运行节点.js。

## 用于构建节点的选项.js

要控制 ICU 在 Node 中的使用方式.js，四`configure`选项可用
在编译期间。记录了有关如何编译 Node.js的更多详细信息
在[BUILDING.md][].

*   `--with-intl=none`/`--without-intl`
*   `--with-intl=system-icu`
*   `--with-intl=small-icu`
*   `--with-intl=full-icu`（默认值）

每个可用的 Node.js 和 JavaScript 功能概述`configure`
选择：

|功能|`none`|`system-icu`|`small-icu`|`full-icu`|
|---------------------------------------- |--------------------------------- |---------------------------- |---------------------- |---------- |
|[`String.prototype.normalize()`][String.prototype.normalize()]|none（函数为 no-op）|全|全|全|
|`String.prototype.to*Case()`|全|全|全|全|
|[`Intl`][Intl]|none （对象不存在） |部分/全部（取决于操作系统）|部分（仅英语）|全|
|[`String.prototype.localeCompare()`][String.prototype.localeCompare()]|部分（非区域设置感知）|全|全|全|
|`String.prototype.toLocale*Case()`|部分（非区域设置感知）|全|全|全|
|[`Number.prototype.toLocaleString()`][Number.prototype.toLocaleString()]|部分（非区域设置感知）|部分/全部（取决于操作系统）|部分（仅英语）|全|
|`Date.prototype.toLocale*String()`|部分（非区域设置感知）|部分/全部（取决于操作系统）|部分（仅英语）|全|
|[旧版 URL 解析器][Legacy URL Parser]|部分（不支持 IDN）|全|全|全|
|[WHATWG URL Parser][]|部分（不支持 IDN）|全|全|全|
|[`require('node:buffer').transcode()`][require('node:buffer').transcode()]|none （函数不存在） |全|全|全|
|[REPL][]|部分（不准确的行编辑）|全|全|全|
|[`require('node:util').TextDecoder`][require('node:util').TextDecoder]|部分（基本编码支持）|部分/全部（取决于操作系统）|部分（仅 Unicode）|全|
|[`RegExp`Unicode 属性转义][RegExp Unicode Property Escapes]|无（无效）`RegExp`错误） |全|全|全|

“（非区域设置感知）”指定表示该函数执行其
操作就像非`Locale`函数的版本（如果有）
存在。例如，在`none`模式`Date.prototype.toLocaleString()`的
操作与`Date.prototype.toString()`.

### 禁用所有国际化功能 （`none`)

如果选择此选项，则 ICU 将被禁用，并且大多数国际化
上面提到的功能将是**不能利用的**在结果中`node`二元的。

### 使用预安装的 ICU （`system-icu`)

Node.js可以链接到系统上已安装的 ICU 内部版本。事实上
大多数Linux发行版已经安装了ICU，此选项将
使重用 由其他组件使用的同一组数据成为可能
平衡计分卡

仅需要 ICU 库本身的功能，例如
[`String.prototype.normalize()`][String.prototype.normalize()]和[WHATWG URL parser][]，完全
支持`system-icu`.需要 ICU 区域设置数据的功能
加法，例如[`Intl.DateTimeFormat`][Intl.DateTimeFormat] *五月*全部或部分
支持，具体取决于安装在
系统。

### 嵌入一组有限的 ICU 数据 （`small-icu`)

此选项使生成的二进制链接静态地连接到 ICU 库，
并包括 ICU 数据的子集（通常仅包含英语区域设置）
这`node`可执行。

仅需要 ICU 库本身的功能，例如
[`String.prototype.normalize()`][String.prototype.normalize()]和[WHATWG URL parser][]，完全
支持`small-icu`.此外，还需要 ICU 区域设置数据的功能，
如[`Intl.DateTimeFormat`][Intl.DateTimeFormat]，通常只适用于英语区域设置：

```js
const january = new Date(9e8);
const english = new Intl.DateTimeFormat('en', { month: 'long' });
const spanish = new Intl.DateTimeFormat('es', { month: 'long' });

console.log(english.format(january));
// Prints "January"
console.log(spanish.format(january));
// Prints either "M01" or "January" on small-icu, depending on the user’s default locale
// Should print "enero"
```

此模式在功能和二进制大小之间提供平衡。

#### 在运行时提供 ICU 数据

如果`small-icu`选项，仍然可以提供其他区域设置数据
在运行时，以便 JS 方法适用于所有 ICU 语言环境。假设
数据文件存储在`/some/directory`，它可以提供给ICU
通过以下任一方式：

*   这[`NODE_ICU_DATA`][NODE_ICU_DATA]环境变量：

    ```bash
    env NODE_ICU_DATA=/some/directory node
    ```

*   这[`--icu-data-dir`][--icu-data-dir]命令行参数：

    ```bash
    node --icu-data-dir=/some/directory
    ```

（如果同时指定了两者，则`--icu-data-dir`CLI 参数优先。

ICU能够自动查找和加载各种数据格式，但是
数据必须适合 ICU 版本，并且文件必须正确命名。
数据文件的最常见名称是`icudt6X[bl].dat`哪里`6X`表示
预期的 ICU 版本，以及`b`或`l`指示系统的字节序。
检查[“重症监护室数据”]["ICU Data"]ICU 用户指南中有关其他受支持格式的文章
以及有关 ICU 数据的更多详细信息。

这[全伊库][full-icu]npm 模块可以通过以下方式大大简化 ICU 数据安装：
检测正在运行的 ICU 版本`node`可执行文件并下载
相应的数据文件。安装模块后通过`npm i full-icu`,
数据文件将在`./node_modules/full-icu`.此路径可以是
然后传递给`NODE_ICU_DATA`或`--icu-data-dir`如上所示
启用完全`Intl`支持。

### 嵌入整个 ICU （`full-icu`)

此选项使生成的针对 ICU 的二进制链接静态化，并包括
一整套 ICU 数据。以这种方式创建的二进制文件没有进一步的外部
依赖并支持所有区域设置，但可能相当大。这是
默认行为（如果不是）`--with-intl`标志已通过。官方二进制文件
也内置在此模式下。

## 检测国际化支持

要验证 ICU 是否已启用 （`system-icu`,`small-icu`或
`full-icu`），只需检查是否存在`Intl`应该足够：

```js
const hasICU = typeof Intl === 'object';
```

或者，检查`process.versions.icu`，仅定义的属性
启用 ICU 后，也可以：

```js
const hasICU = typeof process.versions.icu === 'string';
```

检查对非英语区域设置（即`full-icu`或
`system-icu`),[`Intl.DateTimeFormat`][Intl.DateTimeFormat]可以是一个很好的区分因素：

```js
const hasFullICU = (() => {
  try {
    const january = new Date(9e8);
    const spanish = new Intl.DateTimeFormat('es', { month: 'long' });
    return spanish.format(january) === 'enero';
  } catch (err) {
    return false;
  }
})();
```

有关更详细的测试`Intl`支持，可以找到以下资源
提供帮助：

*   [btest402][]：通常用于检查节点.js`Intl`支持是
    正确构建。
*   [测试262][Test262]：ECMAScript 的官方一致性测试套件包含一个部分
    专用于 ECMA-402。

["ICU Data"]: http://userguide.icu-project.org/icudata

[BUILDING.md]: https://github.com/nodejs/node/blob/HEAD/BUILDING.md

[ECMA-262]: https://tc39.github.io/ecma262/

[ECMA-402]: https://tc39.github.io/ecma402/

[ICU]: http://site.icu-project.org/

[Legacy URL parser]: url.md#legacy-url-api

[REPL]: repl.md#repl

[Test262]: https://github.com/tc39/test262/tree/HEAD/test/intl402

[WHATWG URL parser]: url.md#the-whatwg-url-api

[`--icu-data-dir`]: cli.md#--icu-data-dirfile

[`Date.prototype.toLocaleString()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toLocaleString

[`Intl.DateTimeFormat`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat

[`Intl`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl

[`NODE_ICU_DATA`]: cli.md#node_icu_datafile

[`Number.prototype.toLocaleString()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString

[`RegExp` Unicode Property Escapes]: https://github.com/tc39/proposal-regexp-unicode-property-escapes

[`String.prototype.localeCompare()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare

[`String.prototype.normalize()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/normalize

[`String.prototype.toLowerCase()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/toLowerCase

[`String.prototype.toUpperCase()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/toUpperCase

[`require('node:buffer').transcode()`]: buffer.md#buffertranscodesource-fromenc-toenc

[`require('node:util').TextDecoder`]: util.md#class-utiltextdecoder

[btest402]: https://github.com/srl295/btest402

[full-icu]: https://www.npmjs.com/package/full-icu

[internationalized domain names]: https://en.wikipedia.org/wiki/Internationalized_domain_name
