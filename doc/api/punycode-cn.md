# 小密码

<!-- YAML
deprecated: v7.0.0
-->

<!--introduced_in=v0.10.0-->

> 稳定性：0 - 已弃用

<!-- source_link=lib/punycode.js -->

**Node.js中捆绑的 punycode 模块版本将被弃用。**
在 Node 的未来主要版本中.js此模块将被删除。用户
当前取决于`punycode`模块应切换到使用
提供用户空间[小密码.js][Punycode.js]模块。对于基于小代码的网址
编码，请参见[`url.domainToASCII`][url.domainToASCII]或者，更一般地说，
[WHATWG URL API][].

这`punycode`模块是 的捆绑版本[小密码.js][Punycode.js]模块。它
可以使用以下命令访问：

```js
const punycode = require('punycode');
```

[小密码][Punycode]是由 RFC 3492 定义的字符编码方案，即
主要用于国际化域名。因为主机
URL 中的名称仅限于 ASCII 字符，域名包含
非 ASCII 字符必须使用 Punycode 方案转换为 ASCII。
例如，翻译成英语单词的日语字符，
`'example'`是`'例'`.国际化域名，`'例.com'`（等效
自`'example.com'`） 由 Punycode 表示为 ASCII 字符串
`'xn--fsq.com'`.

这`punycode`模块提供了 Punycode 标准的简单实现。

这`punycode`模块是 Node 使用的第三方依赖项.js和
为方便起见，可供开发人员使用。修复或其他修改
该模块必须定向到[小密码.js][Punycode.js]项目。

## `punycode.decode(string)`

<!-- YAML
added: v0.5.1
-->

*   `string`{字符串}

这`punycode.decode()`方法将[小密码][Punycode]仅 ASCII 的字符串
字符到等效的 Unicode 码位字符串。

```js
punycode.decode('maana-pta'); // 'mañana'
punycode.decode('--dqo34k'); // '☃-⌘'
```

## `punycode.encode(string)`

<!-- YAML
added: v0.5.1
-->

*   `string`{字符串}

这`punycode.encode()`方法将 Unicode 代码点的字符串转换为
[小密码][Punycode]仅 ASCII 字符的字符串。

```js
punycode.encode('mañana'); // 'maana-pta'
punycode.encode('☃-⌘'); // '--dqo34k'
```

## `punycode.toASCII(domain)`

<!-- YAML
added: v0.6.1
-->

*   `domain`{字符串}

这`punycode.toASCII()`方法转换一个 Unicode 字符串，该字符串表示
国际化域名[小密码][Punycode].仅
域名将被转换。叫`punycode.toASCII()`在字符串上
已经只包含ASCII字符将不起作用。

```js
// encode domain names
punycode.toASCII('mañana.com');  // 'xn--maana-pta.com'
punycode.toASCII('☃-⌘.com');   // 'xn----dqo34k.com'
punycode.toASCII('example.com'); // 'example.com'
```

## `punycode.toUnicode(domain)`

<!-- YAML
added: v0.6.1
-->

*   `domain`{字符串}

这`punycode.toUnicode()`方法转换表示域名的字符串
含[小密码][Punycode]将字符编码为 Unicode。只有[小密码][Punycode]
对域名的编码部分进行转换。

```js
// decode domain names
punycode.toUnicode('xn--maana-pta.com'); // 'mañana.com'
punycode.toUnicode('xn----dqo34k.com');  // '☃-⌘.com'
punycode.toUnicode('example.com');       // 'example.com'
```

## `punycode.ucs2`

<!-- YAML
added: v0.7.0
-->

### `punycode.ucs2.decode(string)`

<!-- YAML
added: v0.7.0
-->

*   `string`{字符串}

这`punycode.ucs2.decode()`方法返回一个包含数字的数组
字符串中每个 Unicode 符号的代码点值。

```js
punycode.ucs2.decode('abc'); // [0x61, 0x62, 0x63]
// surrogate pair for U+1D306 tetragram for centre:
punycode.ucs2.decode('\uD834\uDF06'); // [0x1D306]
```

### `punycode.ucs2.encode(codePoints)`

<!-- YAML
added: v0.7.0
-->

*   `codePoints`{整数\[]}

这`punycode.ucs2.encode()`方法返回基于数组的字符串
数字码位值。

```js
punycode.ucs2.encode([0x61, 0x62, 0x63]); // 'abc'
punycode.ucs2.encode([0x1D306]); // '\uD834\uDF06'
```

## `punycode.version`

<!-- YAML
added: v0.6.1
-->

*   {字符串}

返回标识当前[小密码.js][Punycode.js]版本号。

[Punycode]: https://tools.ietf.org/html/rfc3492

[Punycode.js]: https://github.com/bestiejs/punycode.js

[WHATWG URL API]: url.md#the-whatwg-url-api

[`url.domainToASCII`]: url.md#urldomaintoasciidomain
