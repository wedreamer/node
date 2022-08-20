# 查询字符串

<!--introduced_in=v0.1.25-->

> 稳定性： 3 - 旧版

<!--name=querystring-->

<!-- source_link=lib/querystring.js -->

这`node:querystring`模块提供了用于解析和格式化 URL 的实用程序
查询字符串。可以使用以下命令访问它：

```js
const querystring = require('node:querystring');
```

这`querystring`API 被视为旧版。虽然它仍然保持，
新代码应该使用 {URLSearchParams} API 来代替。

## `querystring.decode()`

<!-- YAML
added: v0.1.99
-->

这`querystring.decode()`函数是 的别名`querystring.parse()`.

## `querystring.encode()`

<!-- YAML
added: v0.1.99
-->

这`querystring.encode()`函数是 的别名`querystring.stringify()`.

## `querystring.escape(str)`

<!-- YAML
added: v0.1.25
-->

*   `str`{字符串}

这`querystring.escape()`方法在给定的
`str`以针对 URL 的特定要求进行优化的方式
查询字符串。

这`querystring.escape()`方法由`querystring.stringify()`并且是
一般不期望直接使用。它主要导出以允许
用于提供替换百分比编码实现的应用程序代码，如果
通过分配`querystring.escape`替换为替代函数。

## `querystring.parse(str[, sep[, eq[, options]]])`

<!-- YAML
added: v0.1.25
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10967
    description: Multiple empty entries are now parsed correctly (e.g. `&=&=`).
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6055
    description: The returned object no longer inherits from `Object.prototype`.
  - version:
    - v6.0.0
    - v4.2.4
    pr-url: https://github.com/nodejs/node/pull/3807
    description: The `eq` parameter may now have a length of more than `1`.
-->

*   `str`{字符串}要分析的 URL 查询字符串
*   `sep`{字符串}用于分隔
    查询字符串。**违约：** `'&'`.
*   `eq`{字符串}。用于分隔
    查询字符串。**违约：** `'='`.
*   `options`{对象}
    *   `decodeURIComponent`{函数}解码时使用的功能
        查询字符串中的百分比编码字符。**违约：**
        `querystring.unescape()`.
    *   `maxKeys`{数字}指定要分析的最大键数。
        指定`0`以删除键计数限制。**违约：** `1000`.

这`querystring.parse()`方法解析 URL 查询字符串 （`str`） 转换为
键和值对的集合。

例如，查询字符串`'foo=bar&abc=xyz&abc=123'`被解析为：

<!-- eslint-skip -->

```js
{
  foo: 'bar',
  abc: ['xyz', '123']
}
```

返回的对象`querystring.parse()`方法*不*
原型继承自 JavaScript`Object`.这意味着
`Object`方法，例如`obj.toString()`,`obj.hasOwnProperty()`和其他
未定义且*将不起作用*.

默认情况下，将假定查询字符串中的百分比编码字符
以使用 UTF-8 编码。如果使用替代字符编码，则
另类`decodeURIComponent`选项需要指定：

```js
// Assuming gbkDecodeURIComponent function already exists...

querystring.parse('w=%D6%D0%CE%C4&foo=bar', null, null,
                  { decodeURIComponent: gbkDecodeURIComponent });
```

## `querystring.stringify(obj[, sep[, eq[, options]]])`

<!-- YAML
added: v0.1.25
-->

*   `obj`{对象}要序列化为 URL 查询字符串的对象
*   `sep`{字符串}用于分隔
    查询字符串。**违约：** `'&'`.
*   `eq`{字符串}。用于分隔
    查询字符串。**违约：** `'='`.
*   `options`
    *   `encodeURIComponent`{函数}转换时要使用的函数
        查询字符串中 URL 不安全字符到百分比编码。**违约：**
        `querystring.escape()`.

这`querystring.stringify()`方法从
鉴于`obj`通过循环访问对象的“自己的属性”。

它序列化传入的以下类型的值`obj`:
{string|number|bigint|boolean|string\[]|number\[]|bigint\[]|boolean\[]}
数值必须是有限的。任何其他输入值将被强制为
空字符串。

```js
querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' });
// Returns 'foo=bar&baz=qux&baz=quux&corge='

querystring.stringify({ foo: 'bar', baz: 'qux' }, ';', ':');
// Returns 'foo:bar;baz:qux'
```

默认情况下，查询字符串中需要百分比编码的字符将
编码为 UTF-8。如果需要替代编码，则使用替代编码
`encodeURIComponent`选项需要指定：

```js
// Assuming gbkEncodeURIComponent function already exists,

querystring.stringify({ w: '中文', foo: 'bar' }, null, null,
                      { encodeURIComponent: gbkEncodeURIComponent });
```

## `querystring.unescape(str)`

<!-- YAML
added: v0.1.25
-->

*   `str`{字符串}

这`querystring.unescape()`方法执行 URL 百分比编码的解码
给定字符`str`.

这`querystring.unescape()`方法由`querystring.parse()`并且是
一般不期望直接使用。它主要导出以允许
应用程序代码以提供替代解码实现，如果
通过分配`querystring.unescape`替换为替代函数。

默认情况下，`querystring.unescape()`方法将尝试使用
内置 JavaScript`decodeURIComponent()`解码方法。如果失败了，
将使用一个更安全的等效项，不会抛出格式错误的URL。
