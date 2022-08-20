# 网址

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/url.js -->

这`node:url`模块提供了用于 URL 解析和解析的实用程序。可以使用以下命令访问它：

```mjs
import url from 'node:url';
```

```cjs
const url = require('node:url');
```

## 网址字符串和网址对象

URL 字符串是包含多个有意义组件的结构化字符串。
分析时，将返回一个 URL 对象，其中包含每个组件的属性。

这`node:url`模块提供了两个用于处理 URL 的 API：一个是特定于 Node .js的旧 API，另一个是实现相同 URL 的较新的 API。[WHATWG URL Standard][]由网络浏览器使用。

下面提供了 WHATWG 和 Legacy API 之间的比较。网址上方`'https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash'`，则由旧版返回的对象的属性`url.parse()`显示。下面是 WHATWG 的属性`URL`对象。

WHATWG 网址`origin`属性包括`protocol`和`host`，但不是`username`或`password`.

```text
┌────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                              href                                              │
├──────────┬──┬─────────────────────┬────────────────────────┬───────────────────────────┬───────┤
│ protocol │  │        auth         │          host          │           path            │ hash  │
│          │  │                     ├─────────────────┬──────┼──────────┬────────────────┤       │
│          │  │                     │    hostname     │ port │ pathname │     search     │       │
│          │  │                     │                 │      │          ├─┬──────────────┤       │
│          │  │                     │                 │      │          │ │    query     │       │
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
(All spaces in the "" line should be ignored. They are purely for formatting.)
```

使用 WHATWG API 解析 URL 字符串：

```js
const myURL =
  new URL('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');
```

使用旧版 API 解析 URL 字符串：

```mjs
import url from 'node:url';
const myURL =
  url.parse('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');
```

```cjs
const url = require('node:url');
const myURL =
  url.parse('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');
```

### 从组件部件构造 URL 并获取构造的字符串

可以使用属性 setter 或模板文本字符串从组件部件构造 WHATWG URL：

```js
const myURL = new URL('https://example.org');
myURL.pathname = '/a/b/c';
myURL.search = '?d=e';
myURL.hash = '#fgh';
```

```js
const pathname = '/a/b/c';
const search = '?d=e';
const hash = '#fgh';
const myURL = new URL(`https://example.org${pathname}${search}${hash}`);
```

若要获取构造的 URL 字符串，请使用`href`属性访问器：

```js
console.log(myURL.href);
```

## The WHATWG URL API

### 类：`URL`

<!-- YAML
added:
  - v7.0.0
  - v6.13.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18281
    description: The class is now available on the global object.
-->

浏览器兼容`URL`类，通过遵循 WHATWG URL 标准实现。[已解析的网址示例][Examples of parsed URLs]可以在标准本身中找到。
这`URL`类在全局对象上也可用。

根据浏览器约定，所有属性`URL`对象在类原型上作为 getter 和 setter 实现，而不是作为对象本身的数据属性实现。因此，与[遗产`urlObject`][legacy urlObject]s，使用`delete`关键字上的任何属性`URL`对象（例如`delete  myURL.protocol`,`delete myURL.pathname`等）没有效果，但仍会返回`true`.

#### `new URL(input[, base])`

*   `input`{字符串}要分析的绝对或相对输入 URL。如果`input`是相对的，则`base`是必需的。如果`input`是绝对的，`base`被忽略。如果`input`不是字符串，而是[转换为字符串][converted to a string]第一。
*   `base`{字符串}要解析的基 URL，如果`input`不是绝对的。如果`base`不是字符串，而是[转换为字符串][converted to a string]第一。

创建新的`URL`对象，方法是解析`input`相对于`base`.如果`base`作为字符串传递，它将被解析等效于`new URL(base)`.

```js
const myURL = new URL('/foo', 'https://example.org/');
// https://example.org/foo
```

URL 构造函数可作为全局对象的属性进行访问。
它也可以从内置的 url 模块导入：

```mjs
import { URL } from 'node:url';
console.log(URL === globalThis.URL); // Prints 'true'.
```

```cjs
console.log(URL === require('node:url').URL); // Prints 'true'.
```

一个`TypeError`如果`input`或`base`不是有效的网址。请注意，将努力将给定的值强制转换为字符串。例如：

```js
const myURL = new URL({ toString: () => 'https://example.org/' });
// https://example.org/
```

主机名中出现的 Unicode 字符`input`将使用[小密码][Punycode]算法。

```js
const myURL = new URL('https://測試');
// https://xn--g6w251d/
```

此功能仅在以下情况下可用：`node`可执行文件编译为[重症监护室][ICU]启用。否则，域名将原封不动地通过。

在事先不知道的情况下，如果`input`是一个绝对的 URL 和一个`base`，建议验证`origin`的`URL`对象是预期的。

```js
let myURL = new URL('http://Example.com/', 'https://example.org/');
// http://example.com/

myURL = new URL('https://Example.com/', 'https://example.org/');
// https://example.com/

myURL = new URL('foo://Example.com/', 'https://example.org/');
// foo://Example.com/

myURL = new URL('http:Example.com/', 'https://example.org/');
// http://example.com/

myURL = new URL('https:Example.com/', 'https://example.org/');
// https://example.org/Example.com/

myURL = new URL('foo:Example.com/', 'https://example.org/');
// foo:Example.com/
```

#### `url.hash`

*   {字符串}

获取并设置 URL 的片段部分。

```js
const myURL = new URL('https://example.org/foo#bar');
console.log(myURL.hash);
// Prints #bar

myURL.hash = 'baz';
console.log(myURL.href);
// Prints https://example.org/foo#baz
```

分配给 的值中包含的无效 URL 字符`hash`属性是[百分比编码][percent-encoded].选择要百分比编码的字符可能与[`url.parse()`][url.parse()]和[`url.format()`][url.format()]方法将产生。

#### `url.host`

*   {字符串}

获取并设置 URL 的主机部分。

```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.host);
// Prints example.org:81

myURL.host = 'example.com:82';
console.log(myURL.href);
// Prints https://example.com:82/foo
```

分配给 的主机值无效`host`属性将被忽略。

#### `url.hostname`

*   {字符串}

获取并设置 URL 的主机名部分。关键区别`url.host`和`url.hostname`那是`url.hostname`做*不*包括端口。

```js
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.hostname);
// Prints example.org

// Setting the hostname does not change the port
myURL.hostname = 'example.com:82';
console.log(myURL.href);
// Prints https://example.com:81/foo

// Use myURL.host to change the hostname and port
myURL.host = 'example.org:82';
console.log(myURL.href);
// Prints https://example.org:82/foo
```

分配给 的主机名值无效`hostname`属性将被忽略。

#### `url.href`

*   {字符串}

获取并设置序列化的 URL。

```js
const myURL = new URL('https://example.org/foo');
console.log(myURL.href);
// Prints https://example.org/foo

myURL.href = 'https://example.com/bar';
console.log(myURL.href);
// Prints https://example.com/bar
```

获取`href`属性等效于调用[`url.toString()`][url.toString()].

将此属性的值设置为新值等效于创建新的`URL`对象使用[`new URL(value)`][`new URL()`].每个`URL`对象的属性将被修改。

如果将值分配给`href`属性不是有效的 URL，一个`TypeError`将被抛出。

#### `url.origin`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33325
    description: The scheme "gopher" is no longer special and `url.origin` now
                 returns `'null'` for it.
-->

*   {字符串}

获取 URL 源的只读序列化。

```js
const myURL = new URL('https://example.org/foo/bar?baz');
console.log(myURL.origin);
// Prints https://example.org
```

```js
const idnURL = new URL('https://測試');
console.log(idnURL.origin);
// Prints https://xn--g6w251d

console.log(idnURL.hostname);
// Prints xn--g6w251d
```

#### `url.password`

*   {字符串}

获取并设置 URL 的密码部分。

```js
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.password);
// Prints xyz

myURL.password = '123';
console.log(myURL.href);
// Prints https://abc:123@example.com
```

分配给 的值中包含的无效 URL 字符`password`属性是[百分比编码][percent-encoded].选择要百分比编码的字符可能与[`url.parse()`][url.parse()]和[`url.format()`][url.format()]方法将产生。

#### `url.pathname`

*   {字符串}

获取并设置 URL 的路径部分。

```js
const myURL = new URL('https://example.org/abc/xyz?123');
console.log(myURL.pathname);
// Prints /abc/xyz

myURL.pathname = '/abcdef';
console.log(myURL.href);
// Prints https://example.org/abcdef?123
```

分配给 的值中包含的无效 URL 字符`pathname`属性是[百分比编码][percent-encoded].选择要百分比编码的字符可能与[`url.parse()`][url.parse()]和[`url.format()`][url.format()]方法将产生。

#### `url.port`

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33325
    description: The scheme "gopher" is no longer special.
-->

*   {字符串}

获取并设置 URL 的端口部分。

端口值可以是数字或包含区域中数字的字符串`0`自`65535`（包括）。将值设置为 的默认端口`URL`给定的对象`protocol`将导致`port`值成为空字符串 （`''`).

端口值可以是空字符串，在这种情况下，端口取决于协议/方案：

|协议|端口 |
|-------- |---- |
|“ftp” |21 |
|“文件”|     |
|“http” |80 |
|“https” |443 |
|“ws” |80 |
|“wss” |443 |

将值分配给端口后，该值将首先转换为字符串，使用`.toString()`.

如果该字符串无效，但它以数字开头，则将前导数字分配给`port`.
如果数字超出上述范围，则忽略该数字。

```js
const myURL = new URL('https://example.org:8888');
console.log(myURL.port);
// Prints 8888

// Default ports are automatically transformed to the empty string
// (HTTPS protocol's default port is 443)
myURL.port = '443';
console.log(myURL.port);
// Prints the empty string
console.log(myURL.href);
// Prints https://example.org/

myURL.port = 1234;
console.log(myURL.port);
// Prints 1234
console.log(myURL.href);
// Prints https://example.org:1234/

// Completely invalid port strings are ignored
myURL.port = 'abcd';
console.log(myURL.port);
// Prints 1234

// Leading numbers are treated as a port number
myURL.port = '5678abcd';
console.log(myURL.port);
// Prints 5678

// Non-integers are truncated
myURL.port = 1234.5678;
console.log(myURL.port);
// Prints 1234

// Out-of-range numbers which are not represented in scientific notation
// will be ignored.
myURL.port = 1e10; // 10000000000, will be range-checked as described below
console.log(myURL.port);
// Prints 1234
```

包含小数点的数字（如浮点数或科学记数法中的数字）并非此规则的例外。
小数点前导数字将设置为 URL 的端口，前提是它们是有效的：

```js
myURL.port = 4.567e21;
console.log(myURL.port);
// Prints 4 (because it is the leading number in the string '4.567e21')
```

#### `url.protocol`

*   {字符串}

获取并设置 URL 的协议部分。

```js
const myURL = new URL('https://example.org');
console.log(myURL.protocol);
// Prints https:

myURL.protocol = 'ftp';
console.log(myURL.href);
// Prints ftp://example.org/
```

分配给 的 URL 协议值无效`protocol`属性将被忽略。

##### 特殊方案

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/33325
    description: The scheme "gopher" is no longer special.
-->

这[WHATWG URL Standard][]认为少数 URL 协议方案是*特殊*就如何解析和序列化它们而言。当使用这些特殊协议之一解析 URL 时，`url.protocol`属性可以更改为另一个特殊协议，但不能更改为非特殊协议，反之亦然。

例如，从`http`自`https`工程：

```js
const u = new URL('http://example.org');
u.protocol = 'https';
console.log(u.href);
// https://example.org
```

但是，从`http`到一个假设`fish`协议没有，因为新协议不特殊。

```js
const u = new URL('http://example.org');
u.protocol = 'fish';
console.log(u.href);
// http://example.org
```

同样，也不允许从非特别议定书改为特别议定书：

```js
const u = new URL('fish://example.org');
u.protocol = 'http';
console.log(u.href);
// fish://example.org
```

根据WHATWG URL标准，特殊协议方案是`ftp`,`file`,`http`,`https`,`ws`和`wss`.

#### `url.search`

*   {字符串}

获取并设置 URL 的序列化查询部分。

```js
const myURL = new URL('https://example.org/abc?123');
console.log(myURL.search);
// Prints ?123

myURL.search = 'abc=xyz';
console.log(myURL.href);
// Prints https://example.org/abc?abc=xyz
```

任何无效的 URL 字符出现在分配的值中`search`
属性将是[百分比编码][percent-encoded].选择哪一种
字符到百分比编码可能与[`url.parse()`][url.parse()]
和[`url.format()`][url.format()]方法将产生。

#### `url.searchParams`

*   {URLSearchParams}

获取[`URLSearchParams`][URLSearchParams]对象表示 的查询参数
网址。此属性是只读的，但`URLSearchParams`它提供的对象
可用于改变 URL 实例;替换整个查询
网址的参数，使用[`url.search`][url.search]二传手。看
[`URLSearchParams`][URLSearchParams]文档以了解详细信息。

使用时要小心`.searchParams`以修改`URL`因为
根据 WHATWG 规范，`URLSearchParams`对象用途
用于确定要对哪些字符进行百分比编码的不同规则。为
实例，`URL`对象不会对 ASCII 波形符 （`~`)
字符，而`URLSearchParams`将始终对其进行编码：

```js
const myUrl = new URL('https://example.org/abc?foo=~bar');

console.log(myUrl.search);  // prints ?foo=~bar

// Modify the URL via searchParams...
myUrl.searchParams.sort();

console.log(myUrl.search);  // prints ?foo=%7Ebar
```

#### `url.username`

*   {字符串}

获取并设置 URL 的用户名部分。

```js
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.username);
// Prints abc

myURL.username = '123';
console.log(myURL.href);
// Prints https://123:xyz@example.com/
```

任何无效的 URL 字符出现在分配的值中`username`
属性将是[百分比编码][percent-encoded].选择哪一种
字符到百分比编码可能与[`url.parse()`][url.parse()]
和[`url.format()`][url.format()]方法将产生。

#### `url.toString()`

*   返回：{字符串}

这`toString()`方法`URL`对象返回序列化的 URL。这
返回的值等效于[`url.href`][url.href]和[`url.toJSON()`][url.toJSON()].

#### `url.toJSON()`

*   返回：{字符串}

这`toJSON()`方法`URL`对象返回序列化的 URL。这
返回的值等效于[`url.href`][url.href]和
[`url.toString()`][url.toString()].

此方法在`URL`对象被序列化
跟[`JSON.stringify()`][JSON.stringify()].

```js
const myURLs = [
  new URL('https://www.example.com'),
  new URL('https://test.example.org'),
];
console.log(JSON.stringify(myURLs));
// Prints ["https://www.example.com/","https://test.example.org/"]
```

#### `URL.createObjectURL(blob)`

<!-- YAML
added: v16.7.0
-->

> 稳定性： 1 - 实验

*   `blob`{Blob}
*   返回：{字符串}

创建`'blob:nodedata:...'`表示给定 {Blob} 的 URL 字符串
对象 和可用于检索`Blob`后。

```js
const {
  Blob,
  resolveObjectURL,
} = require('node:buffer');

const blob = new Blob(['hello']);
const id = URL.createObjectURL(blob);

// later...

const otherBlob = resolveObjectURL(id);
console.log(otherBlob.size);
```

已注册的 {Blob} 存储的数据将保留在内存中，直到
`URL.revokeObjectURL()`调用以将其删除。

`Blob`对象在当前线程中注册。如果使用工作人员
线程`Blob`在一个工作线程中注册的对象将不可用
到其他工作线程或主线程。

#### `URL.revokeObjectURL(id)`

<!-- YAML
added: v16.7.0
-->

> 稳定性： 1 - 实验

*   `id`{字符串}一个`'blob:nodedata:...`由先前调用返回的 URL 字符串
    `URL.createObjectURL()`.

删除由给定 ID 标识的存储的 {Blob}。尝试吊销
未注册的 ID 将以静默方式失败。

### 类：`URLSearchParams`

<!-- YAML
added:
  - v7.5.0
  - v6.13.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18281
    description: The class is now available on the global object.
-->

这`URLSearchParams`API 提供对查询的读写访问权限
`URL`.这`URLSearchParams`类也可以与其中一个单独使用
以下四个构造函数。
这`URLSearchParams`类在全局对象上也可用。

The WHATWG`URLSearchParams`接口和[`querystring`][querystring]模块有
目的相似，但目的[`querystring`][querystring]模块更多
一般，因为它允许自定义分隔符字符 （`&`和`=`).
另一方面，此 API 纯粹是为 URL 查询字符串设计的。

```js
const myURL = new URL('https://example.org/?abc=123');
console.log(myURL.searchParams.get('abc'));
// Prints 123

myURL.searchParams.append('abc', 'xyz');
console.log(myURL.href);
// Prints https://example.org/?abc=123&abc=xyz

myURL.searchParams.delete('abc');
myURL.searchParams.set('a', 'b');
console.log(myURL.href);
// Prints https://example.org/?a=b

const newSearchParams = new URLSearchParams(myURL.searchParams);
// The above is equivalent to
// const newSearchParams = new URLSearchParams(myURL.search);

newSearchParams.append('a', 'c');
console.log(myURL.href);
// Prints https://example.org/?a=b
console.log(newSearchParams.toString());
// Prints a=b&a=c

// newSearchParams.toString() is implicitly called
myURL.search = newSearchParams;
console.log(myURL.href);
// Prints https://example.org/?a=b&a=c
newSearchParams.delete('a');
console.log(myURL.href);
// Prints https://example.org/?a=b&a=c
```

#### `new URLSearchParams()`

实例化新的空`URLSearchParams`对象。

#### `new URLSearchParams(string)`

*   `string`{字符串}查询字符串

解析`string`作为查询字符串，并使用它来实例化新的
`URLSearchParams`对象。领先`'?'`（如果存在）将被忽略。

```js
let params;

params = new URLSearchParams('user=abc&query=xyz');
console.log(params.get('user'));
// Prints 'abc'
console.log(params.toString());
// Prints 'user=abc&query=xyz'

params = new URLSearchParams('?user=abc&query=xyz');
console.log(params.toString());
// Prints 'user=abc&query=xyz'
```

#### `new URLSearchParams(obj)`

<!-- YAML
added:
  - v7.10.0
  - v6.13.0
-->

*   `obj`{对象}表示键值对集合的对象

实例化新的`URLSearchParams`具有查询哈希映射的对象。密钥和
每个属性的值`obj`总是被强制为字符串。

与[`querystring`][querystring]模块，数组值形式的重复键是
不允许。数组使用字符串化[`array.toString()`][array.toString()]，只需
用逗号连接所有数组元素。

```js
const params = new URLSearchParams({
  user: 'abc',
  query: ['first', 'second']
});
console.log(params.getAll('query'));
// Prints [ 'first,second' ]
console.log(params.toString());
// Prints 'user=abc&query=first%2Csecond'
```

#### `new URLSearchParams(iterable)`

<!-- YAML
added:
  - v7.10.0
  - v6.13.0
-->

*   `iterable`{可迭代}元素为键值对的可迭代对象

实例化新的`URLSearchParams`具有可迭代映射的对象，其方式为
类似于[`Map`][Map]的构造函数。`iterable`可以是`Array`或任何
可迭代对象。这意味着`iterable`可以是另一个`URLSearchParams`在
在这种情况下，构造函数将简单地创建提供的克隆
`URLSearchParams`.的要素`iterable`是键值对，并且可以
它们本身是任何可迭代的对象。

允许重复的密钥。

```js
let params;

// Using an array
params = new URLSearchParams([
  ['user', 'abc'],
  ['query', 'first'],
  ['query', 'second'],
]);
console.log(params.toString());
// Prints 'user=abc&query=first&query=second'

// Using a Map object
const map = new Map();
map.set('user', 'abc');
map.set('query', 'xyz');
params = new URLSearchParams(map);
console.log(params.toString());
// Prints 'user=abc&query=xyz'

// Using a generator function
function* getQueryPairs() {
  yield ['user', 'abc'];
  yield ['query', 'first'];
  yield ['query', 'second'];
}
params = new URLSearchParams(getQueryPairs());
console.log(params.toString());
// Prints 'user=abc&query=first&query=second'

// Each key-value pair must have exactly two elements
new URLSearchParams([
  ['user', 'abc', 'error'],
]);
// Throws TypeError [ERR_INVALID_TUPLE]:
//        Each query pair must be an iterable [name, value] tuple
```

#### `urlSearchParams.append(name, value)`

*   `name`{字符串}
*   `value`{字符串}

将新的名称/值对追加到查询字符串。

#### `urlSearchParams.delete(name)`

*   `name`{字符串}

删除名称为`name`.

#### `urlSearchParams.entries()`

*   返回： {迭代器}

返回 ES6`Iterator`在查询中的每个名称/值对上。
迭代器的每个项都是一个 JavaScript`Array`.的第一项`Array`
是`name`，则为 的第二项`Array`是`value`.

的别名[`urlSearchParams[@@iterator]()`][`urlSearchParams@@iterator()`].

#### `urlSearchParams.forEach(fn[, thisArg])`

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `fn` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `fn`{函数}为查询中的每个名称/值对调用
*   `thisArg`{对象}用作`this`当的值`fn`称为

循环访问查询中的每个名称/值对并调用给定的函数。

```js
const myURL = new URL('https://example.org/?a=b&c=d');
myURL.searchParams.forEach((value, name, searchParams) => {
  console.log(name, value, myURL.searchParams === searchParams);
});
// Prints:
//   a b true
//   c d true
```

#### `urlSearchParams.get(name)`

*   `name`{字符串}
*   返回：{字符串} 或`null`如果给定的没有名称-值对
    `name`.

返回名称为`name`.如果有
没有这样的对，`null`返回。

#### `urlSearchParams.getAll(name)`

*   `name`{字符串}
*   返回：{字符串\[]}

返回名称为`name`.如果有
没有这样的对，则返回一个空数组。

#### `urlSearchParams.has(name)`

*   `name`{字符串}
*   返回：{布尔值}

返回`true`如果至少有一个名称-值对的名称为`name`.

#### `urlSearchParams.keys()`

*   返回： {迭代器}

返回 ES6`Iterator`在每个名称-值对的名称之上。

```js
const params = new URLSearchParams('foo=bar&foo=baz');
for (const name of params.keys()) {
  console.log(name);
}
// Prints:
//   foo
//   foo
```

#### `urlSearchParams.set(name, value)`

*   `name`{字符串}
*   `value`{字符串}

设置`URLSearchParams`对象关联`name`自
`value`.如果存在任何预先存在的名称-值对，其名称为`name`,
将第一个此类货币对的值设置为`value`并删除所有其他内容。如果没有，
将名称/值对追加到查询字符串。

```js
const params = new URLSearchParams();
params.append('foo', 'bar');
params.append('foo', 'baz');
params.append('abc', 'def');
console.log(params.toString());
// Prints foo=bar&foo=baz&abc=def

params.set('foo', 'def');
params.set('xyz', 'opq');
console.log(params.toString());
// Prints foo=def&abc=def&xyz=opq
```

#### `urlSearchParams.sort()`

<!-- YAML
added:
  - v7.7.0
  - v6.13.0
-->

按名称对所有现有名称/值对进行就地排序。排序完成
与[稳定的排序算法][stable sorting algorithm]，因此名称-值对之间的相对顺序
保留具有相同名称的名称。

此方法尤其可用于增加缓存命中率。

```js
const params = new URLSearchParams('query[]=abc&type=search&query[]=123');
params.sort();
console.log(params.toString());
// Prints query%5B%5D=abc&query%5B%5D=123&type=search
```

#### `urlSearchParams.toString()`

*   返回：{字符串}

返回序列化为字符串的搜索参数，其中包含字符
必要时进行百分比编码。

#### `urlSearchParams.values()`

*   返回： {迭代器}

返回 ES6`Iterator`在每个名称-值对的值之上。

#### `urlSearchParams[Symbol.iterator]()`

*   返回： {迭代器}

返回 ES6`Iterator`在查询字符串中的每个名称/值对上。
迭代器的每个项都是一个 JavaScript`Array`.的第一项`Array`
是`name`，则为 的第二项`Array`是`value`.

的别名[`urlSearchParams.entries()`][urlSearchParams.entries()].

```js
const params = new URLSearchParams('foo=bar&xyz=baz');
for (const [name, value] of params) {
  console.log(name, value);
}
// Prints:
//   foo bar
//   xyz baz
```

### `url.domainToASCII(domain)`

<!-- YAML
added:
  - v7.4.0
  - v6.13.0
-->

*   `domain`{字符串}
*   返回：{字符串}

返回[小密码][Punycode]的 ASCII 序列化`domain`.如果`domain`是一个
无效域，则返回空字符串。

它执行反向运算[`url.domainToUnicode()`][url.domainToUnicode()].

此功能仅在以下情况下可用：`node`可执行文件编译为
[重症监护室][ICU]启用。否则，域名将原封不动地通过。

```mjs
import url from 'node:url';

console.log(url.domainToASCII('español.com'));
// Prints xn--espaol-zwa.com
console.log(url.domainToASCII('中文.com'));
// Prints xn--fiq228c.com
console.log(url.domainToASCII('xn--iñvalid.com'));
// Prints an empty string
```

```cjs
const url = require('node:url');

console.log(url.domainToASCII('español.com'));
// Prints xn--espaol-zwa.com
console.log(url.domainToASCII('中文.com'));
// Prints xn--fiq228c.com
console.log(url.domainToASCII('xn--iñvalid.com'));
// Prints an empty string
```

### `url.domainToUnicode(domain)`

<!-- YAML
added:
  - v7.4.0
  - v6.13.0
-->

*   `domain`{字符串}
*   返回：{字符串}

返回`domain`.如果`domain`是无效的
域中，返回空字符串。

它执行反向运算[`url.domainToASCII()`][url.domainToASCII()].

此功能仅在以下情况下可用：`node`可执行文件编译为
[重症监护室][ICU]启用。否则，域名将原封不动地通过。

```mjs
import url from 'node:url';

console.log(url.domainToUnicode('xn--espaol-zwa.com'));
// Prints español.com
console.log(url.domainToUnicode('xn--fiq228c.com'));
// Prints 中文.com
console.log(url.domainToUnicode('xn--iñvalid.com'));
// Prints an empty string
```

```cjs
const url = require('node:url');

console.log(url.domainToUnicode('xn--espaol-zwa.com'));
// Prints español.com
console.log(url.domainToUnicode('xn--fiq228c.com'));
// Prints 中文.com
console.log(url.domainToUnicode('xn--iñvalid.com'));
// Prints an empty string
```

### `url.fileURLToPath(url)`

<!-- YAML
added: v10.12.0
-->

*   `url`{URL | string}要转换为路径的文件 URL 字符串或 URL 对象。
*   返回：{string} 完全解析的特定于平台的 Node.js文件路径。

此函数可确保正确解码百分比编码字符
以及确保跨平台有效的绝对路径字符串。

```mjs
import { fileURLToPath } from 'node:url';

const __filename = fileURLToPath(import.meta.url);

new URL('file:///C:/path/').pathname;      // Incorrect: /C:/path/
fileURLToPath('file:///C:/path/');         // Correct:   C:\path\ (Windows)

new URL('file://nas/foo.txt').pathname;    // Incorrect: /foo.txt
fileURLToPath('file://nas/foo.txt');       // Correct:   \\nas\foo.txt (Windows)

new URL('file:///你好.txt').pathname;      // Incorrect: /%E4%BD%A0%E5%A5%BD.txt
fileURLToPath('file:///你好.txt');         // Correct:   /你好.txt (POSIX)

new URL('file:///hello world').pathname;   // Incorrect: /hello%20world
fileURLToPath('file:///hello world');      // Correct:   /hello world (POSIX)
```

```cjs
const { fileURLToPath } = require('node:url');
new URL('file:///C:/path/').pathname;      // Incorrect: /C:/path/
fileURLToPath('file:///C:/path/');         // Correct:   C:\path\ (Windows)

new URL('file://nas/foo.txt').pathname;    // Incorrect: /foo.txt
fileURLToPath('file://nas/foo.txt');       // Correct:   \\nas\foo.txt (Windows)

new URL('file:///你好.txt').pathname;      // Incorrect: /%E4%BD%A0%E5%A5%BD.txt
fileURLToPath('file:///你好.txt');         // Correct:   /你好.txt (POSIX)

new URL('file:///hello world').pathname;   // Incorrect: /hello%20world
fileURLToPath('file:///hello world');      // Correct:   /hello world (POSIX)
```

### `url.format(URL[, options])`

<!-- YAML
added: v7.6.0
-->

*   `URL`{网址}一个[WHATWG URL][]对象
*   `options`{对象}
    *   `auth`{布尔值}`true`如果序列化的 URL 字符串应包含
        用户名和密码，`false`否则。**违约：** `true`.
    *   `fragment`{布尔值}`true`如果序列化的 URL 字符串应包含
        片段`false`否则。**违约：** `true`.
    *   `search`{布尔值}`true`如果序列化的 URL 字符串应包含
        搜索查询，`false`否则。**违约：** `true`.
    *   `unicode`{布尔值}`true`如果主机中出现 Unicode 字符
        URL字符串的组件应直接编码，而不是
        Punycode 编码。**违约：** `false`.
*   返回：{字符串}

返回 URL 的可自定义序列化`String`表示
[WHATWG URL][]对象。

URL 对象具有`toString()`方法和`href`返回的属性
URL 的字符串序列化。但是，这些不能在
无论如何。这`url.format(URL[, options])`方法允许基本自定义
的输出。

```mjs
import url from 'node:url';
const myURL = new URL('https://a:b@測試?abc#foo');

console.log(myURL.href);
// Prints https://a:b@xn--g6w251d/?abc#foo

console.log(myURL.toString());
// Prints https://a:b@xn--g6w251d/?abc#foo

console.log(url.format(myURL, { fragment: false, unicode: true, auth: false }));
// Prints 'https://測試/?abc'
```

```cjs
const url = require('node:url');
const myURL = new URL('https://a:b@測試?abc#foo');

console.log(myURL.href);
// Prints https://a:b@xn--g6w251d/?abc#foo

console.log(myURL.toString());
// Prints https://a:b@xn--g6w251d/?abc#foo

console.log(url.format(myURL, { fragment: false, unicode: true, auth: false }));
// Prints 'https://測試/?abc'
```

### `url.pathToFileURL(path)`

<!-- YAML
added: v10.12.0
-->

*   `path`{字符串}要转换为文件 URL 的路径。
*   返回：{URL} 文件 URL 对象。

此功能可确保`path`是绝对解析的，并且 URL
控制字符在转换为文件 URL 时正确编码。

```mjs
import { pathToFileURL } from 'node:url';

new URL('/foo#1', 'file:');           // Incorrect: file:///foo#1
pathToFileURL('/foo#1');              // Correct:   file:///foo%231 (POSIX)

new URL('/some/path%.c', 'file:');    // Incorrect: file:///some/path%.c
pathToFileURL('/some/path%.c');       // Correct:   file:///some/path%25.c (POSIX)
```

```cjs
const { pathToFileURL } = require('node:url');
new URL(__filename);                  // Incorrect: throws (POSIX)
new URL(__filename);                  // Incorrect: C:\... (Windows)
pathToFileURL(__filename);            // Correct:   file:///... (POSIX)
pathToFileURL(__filename);            // Correct:   file:///C:/... (Windows)

new URL('/foo#1', 'file:');           // Incorrect: file:///foo#1
pathToFileURL('/foo#1');              // Correct:   file:///foo%231 (POSIX)

new URL('/some/path%.c', 'file:');    // Incorrect: file:///some/path%.c
pathToFileURL('/some/path%.c');       // Correct:   file:///some/path%25.c (POSIX)
```

### `url.urlToHttpOptions(url)`

<!-- YAML
added:
  - v15.7.0
  - v14.18.0
-->

*   `url`{网址}这[WHATWG URL][]对象以转换为选项对象。
*   返回：{对象} 选项对象
    *   `protocol`{字符串}要使用的协议。
    *   `hostname`{字符串}要颁发的服务器的域名或 IP 地址
        请求。
    *   `hash`{字符串}URL 的片段部分。
    *   `search`{字符串}URL 的序列化查询部分。
    *   `pathname`{字符串}URL 的路径部分。
    *   `path`{字符串}请求路径。应包括查询字符串（如果有）。
        例如`'/index.html?page=12'`.当请求路径时引发异常
        包含非法字符。目前，只有空格被拒绝，但
        将来可能会改变。
    *   `href`{字符串}序列化的 URL。
    *   `port`{数字}远程服务器的端口。
    *   `auth`{字符串}基本身份验证，即`'user:password'`以计算
        授权标头。

此实用程序函数将 URL 对象转换为普通选项对象
预期由[`http.request()`][http.request()]和[`https.request()`][https.request()]蜜蜂属。

```mjs
import { urlToHttpOptions } from 'node:url';
const myURL = new URL('https://a:b@測試?abc#foo');

console.log(urlToHttpOptions(myURL));
/*
{
  protocol: 'https:',
  hostname: 'xn--g6w251d',
  hash: '#foo',
  search: '?abc',
  pathname: '/',
  path: '/?abc',
  href: 'https://a:b@xn--g6w251d/?abc#foo',
  auth: 'a:b'
}
*/
```

```cjs
const { urlToHttpOptions } = require('node:url');
const myURL = new URL('https://a:b@測試?abc#foo');

console.log(urlToHttpOptions(myUrl));
/*
{
  protocol: 'https:',
  hostname: 'xn--g6w251d',
  hash: '#foo',
  search: '?abc',
  pathname: '/',
  path: '/?abc',
  href: 'https://a:b@xn--g6w251d/?abc#foo',
  auth: 'a:b'
}
*/
```

## 旧版网址 API

<!-- YAML
changes:
  - version:
      - v15.13.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37784
    description: Deprecation revoked. Status changed to "Legacy".
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22715
    description: This API is deprecated.
-->

> 稳定性：3 - 旧版：改用 WHATWG URL API。

### 遗产`urlObject`

<!-- YAML
changes:
  - version:
      - v15.13.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37784
    description: Deprecation revoked. Status changed to "Legacy".
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22715
    description: The Legacy URL API is deprecated. Use the WHATWG URL API.
-->

> 稳定性：3 - 旧版：改用 WHATWG URL API。

遗产`urlObject`(`require('node:url').Url`或
`import { Url } from 'node:url'`） 是
由 创建并返回`url.parse()`功能。

#### `urlObject.auth`

这`auth`属性是 URL 的用户名和密码部分，也是
称为*用户信息*.此字符串子集遵循`protocol`和
双斜杠（如果存在）和`host`组件，由`@`.
该字符串要么是用户名，要么是分隔的用户名和密码
由`:`.

例如：`'user:pass'`.

#### `urlObject.hash`

这`hash`属性是 URL 的片段标识符部分，包括
主导`#`字符。

例如：`'#hash'`.

#### `urlObject.host`

这`host`属性是 URL 的完全小写主机部分，包括
这`port`如果指定。

例如：`'sub.example.com:8080'`.

#### `urlObject.hostname`

这`hostname`属性是`host`
元件*没有*这`port`包括。

例如：`'sub.example.com'`.

#### `urlObject.href`

这`href`属性是使用
`protocol`和`host`组件转换为小写。

例如：`'http://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash'`.

#### `urlObject.path`

这`path`属性是`pathname`和`search`
组件。

例如：`'/p/a/t/h?query=string'`.

无需解码`path`执行。

#### `urlObject.pathname`

这`pathname`属性由 URL 的整个路径部分组成。这
是遵循的一切`host`（包括`port`） 和开始之前
的`query`或`hash`组件，由 ASCII 问题分隔
标记（`?`） 或哈希 （`#`） 字符。

例如：`'/p/a/t/h'`.

不执行路径字符串的解码。

#### `urlObject.port`

这`port`属性是`host`元件。

例如：`'8080'`.

#### `urlObject.protocol`

这`protocol`属性标识 URL 的小写协议方案。

例如：`'http:'`.

#### `urlObject.query`

这`query`属性是没有前导 ASCII 的查询字符串
问号 （`?`），或返回的对象[`querystring`][querystring]模块的
`parse()`方法。是否`query`属性是字符串或对象是
由`parseQueryString`参数传递给`url.parse()`.

例如：`'query=string'`或`{'query': 'string'}`.

如果作为字符串返回，则不执行查询字符串的解码。如果
作为对象返回，键和值都被解码。

#### `urlObject.search`

这`search`属性由整个“查询字符串”部分组成
网址，包括前导 ASCII 问号 （`?`） 字符。

例如：`'?query=string'`.

不执行查询字符串的解码。

#### `urlObject.slashes`

这`slashes`属性是一个`boolean`值为`true`如果两个 ASCII
正斜杠字符 （`/`） 是必需的，在
`protocol`.

### `url.format(urlObject)`

<!-- YAML
added: v0.1.25
changes:
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/38631
    description: Now throws an `ERR_INVALID_URL` exception when Punycode
                 conversion of a hostname introduces changes that could cause
                 the URL to be re-parsed differently.
  - version:
      - v15.13.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37784
    description: Deprecation revoked. Status changed to "Legacy".
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22715
    description: The Legacy URL API is deprecated. Use the WHATWG URL API.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7234
    description: URLs with a `file:` scheme will now always use the correct
                 number of slashes regardless of `slashes` option. A falsy
                 `slashes` option with no protocol is now also respected at all
                 times.
-->

> 稳定性：3 - 旧版：改用 WHATWG URL API。

*   `urlObject`{对象|字符串}一个 URL 对象（由`url.parse()`或
    以其他方式构造）。如果是字符串，则通过传递将其转换为对象
    它到`url.parse()`.

这`url.format()`方法返回从以下公式派生的格式化 URL 字符串
`urlObject`.

```js
const url = require('node:url');
url.format({
  protocol: 'https',
  hostname: 'example.com',
  pathname: '/some/path',
  query: {
    page: 1,
    format: 'json'
  }
});

// => 'https://example.com/some/path?page=1&format=json'
```

如果`urlObject`不是对象或字符串，`url.format()`将抛出一个
[`TypeError`][TypeError].

格式化过程按如下方式运行：

*   新的空字符串`result`已创建。
*   如果`urlObject.protocol`是一个字符串，它按原样追加到`result`.
*   否则，如果`urlObject.protocol`莫`undefined`并且不是字符串，而是
    [`Error`][Error]被抛出。
*   对于`urlObject.protocol`那*不结束*使用 ASCII
    结肠 （`:`） 字符，文本字符串`:`将附加到`result`.
*   如果满足以下任一条件，则文本字符串`//`
    将附加到`result`:
    *   `urlObject.slashes`属性是真实的;
    *   `urlObject.protocol`开头为`http`,`https`,`ftp`,`gopher`或
        `file`;
*   如果`urlObject.auth`财产是真实的，并且
    `urlObject.host`或`urlObject.hostname`不是`undefined`，的值
    `urlObject.auth`将被强制转换为字符串并附加到`result`
    后跟文字字符串`@`.
*   如果`urlObject.host`属性是`undefined`然后：
    *   如果`urlObject.hostname`是一个字符串，它被追加到`result`.
    *   否则，如果`urlObject.hostname`莫`undefined`并且不是字符串，
        一[`Error`][Error]被抛出。
    *   如果`urlObject.port`财产价值是真实的，并且`urlObject.hostname`
        莫`undefined`:
        *   文本字符串`:`附加到`result`和
        *   的价值`urlObject.port`被强制为字符串并追加到
            `result`.
*   否则，如果`urlObject.host`物业价值是真实的，价值
    `urlObject.host`被强制为字符串并追加到`result`.
*   如果`urlObject.pathname`属性是一个不是空字符串的字符串：
    *   如果`urlObject.pathname` *不启动*带有 ASCII 正斜杠
        (`/`），然后是文本字符串`'/'`附加到`result`.
    *   的价值`urlObject.pathname`附加到`result`.
*   否则，如果`urlObject.pathname`莫`undefined`并且不是字符串，而是
    [`Error`][Error]被抛出。
*   如果`urlObject.search`属性是`undefined`并且如果`urlObject.query`
    属性是一个`Object`，文本字符串`?`附加到`result`
    后跟调用 的输出[`querystring`][querystring]模块的`stringify()`
    方法传递值`urlObject.query`.
*   否则，如果`urlObject.search`是一个字符串：
    *   如果`urlObject.search` *不启动*与 ASCII 问题
        标记（`?`） 字符，文本字符串`?`附加到`result`.
    *   的价值`urlObject.search`附加到`result`.
*   否则，如果`urlObject.search`莫`undefined`并且不是字符串，而是
    [`Error`][Error]被抛出。
*   如果`urlObject.hash`属性是一个字符串：
    *   如果`urlObject.hash` *不启动*使用 ASCII 哈希 （`#`)
        字符，文本字符串`#`附加到`result`.
    *   的价值`urlObject.hash`附加到`result`.
*   否则，如果`urlObject.hash`属性不是`undefined`并且不是
    字符串，一个[`Error`][Error]被抛出。
*   `result`返回。

### `url.parse(urlString[, parseQueryString[, slashesDenoteHost]])`

<!-- YAML
added: v0.1.25
changes:
  - version:
      - v15.13.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37784
    description: Deprecation revoked. Status changed to "Legacy".
  - version: v11.14.0
    pr-url: https://github.com/nodejs/node/pull/26941
    description: The `pathname` property on the returned URL object is now `/`
                 when there is no path and the protocol scheme is `ws:` or
                 `wss:`.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22715
    description: The Legacy URL API is deprecated. Use the WHATWG URL API.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/13606
    description: The `search` property on the returned URL object is now `null`
                 when no query string is present.
-->

> 稳定性：3 - 旧版：改用 WHATWG URL API。

*   `urlString`{字符串}要分析的 URL 字符串。
*   `parseQueryString`{布尔值}如果`true`这`query`属性将始终
    设置为 由 返回的对象[`querystring`][querystring]模块的`parse()`
    方法。如果`false`这`query`返回的 URL 对象上的属性将是一个
    未分析、未编码的字符串。**违约：** `false`.
*   `slashesDenoteHost`{布尔值}如果`true`，文字后的第一个标记
    字符串`//`和前面的下一个`/`将被解释为`host`.
    例如，给定`//foo/bar`，结果将是
    `{host: 'foo', pathname: '/bar'}`而不是`{pathname: '//foo/bar'}`.
    **违约：** `false`.

这`url.parse()`方法获取 URL 字符串，对其进行分析，然后返回 URL
对象。

一个`TypeError`在以下情况下被抛出`urlString`不是字符串。

一个`URIError`如果`auth`属性存在，但无法解码。

`url.parse()`使用宽松的非标准算法来解析 URL
字符串。它容易出现安全问题，例如[主机名欺骗][host name spoofing]
以及用户名和密码的不正确处理。

`url.parse()`是大多数旧版 API 的例外。尽管它很安全
关注，它是遗留的，而不是弃用的，因为它是：

*   比替代的WHATWG更快`URL`解析 器。
*   在相对URL方面比替代WHATWG更易于使用`URL`应用程序接口。
*   在 npm 生态系统中广泛依赖。

请谨慎使用。

### `url.resolve(from, to)`

<!-- YAML
added: v0.1.25
changes:
  - version:
      - v15.13.0
      - v14.17.0
    pr-url: https://github.com/nodejs/node/pull/37784
    description: Deprecation revoked. Status changed to "Legacy".
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22715
    description: The Legacy URL API is deprecated. Use the WHATWG URL API.
  - version: v6.6.0
    pr-url: https://github.com/nodejs/node/pull/8215
    description: The `auth` fields are now kept intact when `from` and `to`
                 refer to the same host.
  - version:
    - v6.5.0
    - v4.6.2
    pr-url: https://github.com/nodejs/node/pull/8214
    description: The `port` field is copied correctly now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/1480
    description: The `auth` fields is cleared now the `to` parameter
                 contains a hostname.
-->

> 稳定性：3 - 旧版：改用 WHATWG URL API。

*   `from`{字符串}在以下情况下要使用的基本 URL`to`是相对 URL。
*   `to`{字符串}要解析的目标 URL。

这`url.resolve()`方法解析相对于中基本 URL 的目标 URL 的方法
方式类似于 Web 浏览器解析锚点标记的方式。

```js
const url = require('node:url');
url.resolve('/one/two/three', 'four');         // '/one/two/four'
url.resolve('http://example.com/', '/one');    // 'http://example.com/one'
url.resolve('http://example.com/one', '/two'); // 'http://example.com/two'
```

要使用 WHATWG URL API 获得相同的结果，请执行以下操作：

```js
function resolve(from, to) {
  const resolvedUrl = new URL(to, new URL(from, 'resolve://'));
  if (resolvedUrl.protocol === 'resolve:') {
    // `from` is a relative URL.
    const { pathname, search, hash } = resolvedUrl;
    return pathname + search + hash;
  }
  return resolvedUrl.toString();
}

resolve('/one/two/three', 'four');         // '/one/two/four'
resolve('http://example.com/', '/one');    // 'http://example.com/one'
resolve('http://example.com/one', '/two'); // 'http://example.com/two'
```

<a id="whatwg-percent-encoding"></a>

## 网址中的百分比编码

URL 只允许包含一定范围的字符。任何字符
必须对超出该范围的内容进行编码。如何对这些字符进行编码，
以及要编码的字符完全取决于字符的位置
位于 URL 的结构中。

### 旧版 API

在旧版 API 中，空格 （`' '`）， 并且以下字符将是
在 URL 对象的属性中自动转义：

```text
< > " ` \r \n \t { } | \ ^ '
```

例如，ASCII 空格字符 （`' '`） 编码为`%20`.The ASCII
正斜杠 （`/`） 字符编码为`%3C`.

### WHATWG API

这[WHATWG URL Standard][]使用更具选择性和细粒度的方法
选择比旧版 API 使用的编码字符更多的编码字符。

WHATWG 算法定义了四个描述范围的“百分比编码集”
必须进行百分比编码的字符数：

*   这*C0 控制百分比编码集*包括代码点，范围 U+0000 到
    U+001F（含）和所有大于 U+007E 的码位。

*   这*片段百分比编码集*包括*C0 控制百分比编码集*
    和代码点 U+0020、U+0022、U+003C、U+003E 和 U+0060。

*   这*路径百分比编码集*包括*C0 控制百分比编码集*
    和码位 U+0020、U+0022、U+0023、U+003C、U+003E、U+003F、U+0060、
    U+007B 和 U+007D。

*   这*用户信息编码集*包括*路径百分比编码集*和代码
    点 U+002F， U+003A， U+003B， U+003D， U+0040， U+005B， U+005C， U+005D，
    U+005E 和 U+007C。

这*用户信息百分比编码集*仅用于用户名和
在 URL 中编码的密码。这*路径百分比编码集*用于
大多数网址的路径。这*片段百分比编码集*用于 URL 片段。
这*C0 控制百分比编码集*用于主机和路径下的某些
除所有其他情况外，其他所有情况均为特定情况。

当主机名中出现非 ASCII 字符时，将对主机名进行编码
使用[小密码][Punycode]算法。但请注意，主机名*五月*包含
*双*Punycode 编码字符和百分比编码字符：

```js
const myURL = new URL('https://%CF%80.example.com/foo');
console.log(myURL.href);
// Prints https://xn--1xa.example.com/foo
console.log(myURL.origin);
// Prints https://xn--1xa.example.com
```

[ICU]: intl.md#options-for-building-nodejs

[Punycode]: https://tools.ietf.org/html/rfc5891#section-4.4

[WHATWG URL]: #the-whatwg-url-api

[WHATWG URL Standard]: https://url.spec.whatwg.org/

[`Error`]: errors.md#class-error

[`JSON.stringify()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

[`Map`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map

[`TypeError`]: errors.md#class-typeerror

[`URLSearchParams`]: #class-urlsearchparams

[`array.toString()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toString

[`http.request()`]: http.md#httprequestoptions-callback

[`https.request()`]: https.md#httpsrequestoptions-callback

[`new URL()`]: #new-urlinput-base

[`querystring`]: querystring.md

[`url.domainToASCII()`]: #urldomaintoasciidomain

[`url.domainToUnicode()`]: #urldomaintounicodedomain

[`url.format()`]: #urlformaturlobject

[`url.href`]: #urlhref

[`url.parse()`]: #urlparseurlstring-parsequerystring-slashesdenotehost

[`url.search`]: #urlsearch

[`url.toJSON()`]: #urltojson

[`url.toString()`]: #urltostring

[`urlSearchParams.entries()`]: #urlsearchparamsentries

[`urlSearchParams@@iterator()`]: #urlsearchparamssymboliterator

[converted to a string]: https://tc39.es/ecma262/#sec-tostring

[examples of parsed URLs]: https://url.spec.whatwg.org/#example-url-parsing

[host name spoofing]: https://hackerone.com/reports/678487

[legacy `urlObject`]: #legacy-urlobject

[percent-encoded]: #percent-encoding-in-urls

[stable sorting algorithm]: https://en.wikipedia.org/wiki/Sorting_algorithm#Stability
