# 域名解析

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/dns.js -->

这`node:dns`模块启用名称解析。例如，使用它来查找 IP
主机名的地址。

虽然以[域名系统][Domain Name System (DNS)]，它并不总是使用
用于查找的 DNS 协议。[`dns.lookup()`][dns.lookup()]使用操作系统
执行名称解析的工具。它可能不需要执行任何网络
通信。以同一应用程序上其他应用程序的方式执行名称解析
系统做，使用[`dns.lookup()`][dns.lookup()].

```js
const dns = require('node:dns');

dns.lookup('example.org', (err, address, family) => {
  console.log('address: %j family: IPv%s', address, family);
});
// address: "93.184.216.34" family: IPv4
```

中的所有其他功能`node:dns`模块连接到实际的 DNS 服务器
执行名称解析。他们将始终使用网络来执行 DNS
查询。这些函数不使用
[`dns.lookup()`][dns.lookup()]（例如`/etc/hosts`).使用这些功能始终执行
DNS 查询，绕过其他名称解析功能。

```js
const dns = require('node:dns');

dns.resolve4('archive.org', (err, addresses) => {
  if (err) throw err;

  console.log(`addresses: ${JSON.stringify(addresses)}`);

  addresses.forEach((a) => {
    dns.reverse(a, (err, hostnames) => {
      if (err) {
        throw err;
      }
      console.log(`reverse for ${a}: ${JSON.stringify(hostnames)}`);
    });
  });
});
```

查看[实现注意事项部分][Implementation considerations section]了解更多信息。

## 类：`dns.Resolver`

<!-- YAML
added: v8.3.0
-->

用于 DNS 请求的独立解析程序。

创建新的冲突解决程序将使用默认服务器设置。设置
用于冲突解决程序的服务器使用
[`resolver.setServers()`][`dns.setServers()`]不影响
其他解析器：

```js
const { Resolver } = require('node:dns');
const resolver = new Resolver();
resolver.setServers(['4.4.4.4']);

// This request will use the server at 4.4.4.4, independent of global settings.
resolver.resolve4('example.org', (err, addresses) => {
  // ...
});
```

以下方法从`node:dns`模块可用：

*   [`resolver.getServers()`][`dns.getServers()`]
*   [`resolver.resolve()`][`dns.resolve()`]
*   [`resolver.resolve4()`][`dns.resolve4()`]
*   [`resolver.resolve6()`][`dns.resolve6()`]
*   [`resolver.resolveAny()`][`dns.resolveAny()`]
*   [`resolver.resolveCaa()`][`dns.resolveCaa()`]
*   [`resolver.resolveCname()`][`dns.resolveCname()`]
*   [`resolver.resolveMx()`][`dns.resolveMx()`]
*   [`resolver.resolveNaptr()`][`dns.resolveNaptr()`]
*   [`resolver.resolveNs()`][`dns.resolveNs()`]
*   [`resolver.resolvePtr()`][`dns.resolvePtr()`]
*   [`resolver.resolveSoa()`][`dns.resolveSoa()`]
*   [`resolver.resolveSrv()`][`dns.resolveSrv()`]
*   [`resolver.resolveTxt()`][`dns.resolveTxt()`]
*   [`resolver.reverse()`][`dns.reverse()`]
*   [`resolver.setServers()`][`dns.setServers()`]

### `Resolver([options])`

<!-- YAML
added: v8.3.0
changes:
  - version:
      - v16.7.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/39610
    description: The `options` object now accepts a `tries` option.
  - version: v12.18.3
    pr-url: https://github.com/nodejs/node/pull/33472
    description: The constructor now accepts an `options` object.
                 The single supported option is `timeout`.
-->

创建新的冲突解决程序。

*   `options`{对象}
    *   `timeout`{整数}查询超时（以毫秒为单位），或`-1`以使用
        默认超时。
    *   `tries`{整数}解析程序将尝试联系的尝试次数
        每个名称服务器在放弃之前。**违约：** `4`

### `resolver.cancel()`

<!-- YAML
added: v8.3.0
-->

取消此解析程序进行的所有未完成的 DNS 查询。相应的
调用回调时会出现带有代码的错误`ECANCELLED`.

### `resolver.setLocalAddress([ipv4][, ipv6])`

<!-- YAML
added:
  - v15.1.0
  - v14.17.0
-->

*   `ipv4`{字符串}IPv4 地址的字符串表示形式。
    **违约：** `'0.0.0.0'`
*   `ipv6`{字符串}IPv6 地址的字符串表示形式。
    **违约：** `'::0'`

解析程序实例将从指定的 IP 地址发送其请求。
这允许程序在多宿主上使用时指定出站接口
系统。

如果未指定 v4 或 v6 地址，则将其设置为默认值，并且
操作系统将自动选择本地地址。

解析程序在向 IPv4 DNS 发出请求时将使用 v4 本地地址
服务器，以及向 IPv6 DNS 服务器发出请求时的 v6 本地地址。
这`rrtype`的解析请求对所使用的本地地址没有影响。

## `dns.getServers()`

<!-- YAML
added: v0.11.3
-->

*   返回：{字符串\[]}

返回 IP 地址字符串的数组，其格式根据[RFC 5952][],
当前已针对 DNS 解析进行了配置。字符串将包含端口
部分（如果使用自定义端口）。

<!-- eslint-disable semi-->

```js
[
  '4.4.4.4',
  '2001:4860:4860::8888',
  '4.4.4.4:1053',
  '[2001:4860:4860::8888]:1053',
]
```

## `dns.lookup(hostname[, options], callback)`

<!-- YAML
added: v0.1.90
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43054
    description: For compatibility with `node:net`, when passing an option
                 object the `family` option can be the string `'IPv4'` or the
                 string `'IPv6'`.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v17.0.0
    pr-url: https://github.com/nodejs/node/pull/39987
    description: The `verbatim` options defaults to `true` now.
  - version: v8.5.0
    pr-url: https://github.com/nodejs/node/pull/14731
    description: The `verbatim` option is supported now.
  - version: v1.2.0
    pr-url: https://github.com/nodejs/node/pull/744
    description: The `all` option is supported now.
-->

*   `hostname`{字符串}
*   `options`{整数|对象}
    *   `family`{整数|字符串}记录系列。必须是`4`,`6`或`0`.为
        向后兼容原因，`'IPv4'`和`'IPv6'`被解释为`4`
        和`6`分别。价值`0`表示 IPv4 和 IPv6 地址
        都返回。**违约：** `0`.
    *   `hints`{数字}一个或多个[支持`getaddrinfo`标志][supported getaddrinfo flags].倍数
        标志可以通过按位传递`OR`他们的价值观。
    *   `all`{布尔值}什么时候`true`，则回调返回 中所有已解析的地址
        一个数组。否则，返回单个地址。**违约：** `false`.
    *   `verbatim`{布尔值}什么时候`true`，回调接收 IPv4 和 IPv6
        地址按 DNS 解析程序返回的顺序排列。什么时候`false`,
        IPv4 地址位于 IPv6 地址之前。
        **违约：** `true`（地址不会重新排序）。默认值为
        可配置使用[`dns.setDefaultResultOrder()`][dns.setDefaultResultOrder()]或
        [`--dns-result-order`][--dns-result-order].
*   `callback`{函数}
    *   `err`{错误}
    *   `address`{字符串}IPv4 或 IPv6 地址的字符串表示形式。
    *   `family`{整数}`4`或`6`，表示`address`或`0`如果
        该地址不是 IPv4 或 IPv6 地址。`0`是
        操作系统使用的名称解析服务中的 bug。

解析主机名（例如`'nodejs.org'`） 添加到第一个找到的 A （IPv4） 或
AAAA （IPv6） 记录。都`option`属性是可选的。如果`options`是一个
整数，则它必须是`4`或`6`– 如果`options`是`0`或未提供，则
如果找到 IPv4 和 IPv6 地址，则同时返回。

随着`all`选项设置为`true`，参数`callback`更改为
`(err, addresses)`跟`addresses`是具有
性能`address`和`family`.

出错时，`err`是一个[`Error`][Error]对象，其中`err.code`是错误代码。
请记住，`err.code`将设置为`'ENOTFOUND'`不仅在以下情况下
主机名不存在，但当查找以其他方式失败时也存在
例如，没有可用的文件描述符。

`dns.lookup()`不一定与 DNS 协议有任何关系。
该实现使用可以关联名称的操作系统工具
与地址，反之亦然。此实现可以具有微妙但
对任何Node.js程序的行为产生重要影响。请拿一些
时间咨询[实现注意事项部分][Implementation considerations section]使用前
`dns.lookup()`.

用法示例：

```js
const dns = require('node:dns');
const options = {
  family: 6,
  hints: dns.ADDRCONFIG | dns.V4MAPPED,
};
dns.lookup('example.com', options, (err, address, family) =>
  console.log('address: %j family: IPv%s', address, family));
// address: "2606:2800:220:1:248:1893:25c8:1946" family: IPv6

// When options.all is true, the result will be an Array.
options.all = true;
dns.lookup('example.com', options, (err, addresses) =>
  console.log('addresses: %j', addresses));
// addresses: [{"address":"2606:2800:220:1:248:1893:25c8:1946","family":6}]
```

如果调用此方法作为其[`util.promisify()`][util.promisify()]ed 版本，以及`all`
未设置为`true`，则返回一个`Promise`对于`Object`跟`address`和
`family`性能。

### 支持的 getaddrinfo 标志

<!-- YAML
changes:
  - version:
     - v13.13.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32183
    description: Added support for the `dns.ALL` flag.
-->

以下标志可以作为提示传递给[`dns.lookup()`][dns.lookup()].

*   `dns.ADDRCONFIG`：将返回的地址类型限制为非环回类型
    系统上配置的地址。例如，IPv4 地址仅
    如果当前系统至少配置了一个 IPv4 地址，则返回。
*   `dns.V4MAPPED`：如果指定了 IPv6 系列，但没有 IPv6 地址
    找到，然后返回 IPv4 映射的 IPv6 地址。它不受支持
    在某些操作系统上 （例如 FreeBSD 10.1）。
*   `dns.ALL`： 如果`dns.V4MAPPED`，返回解析的 IPv6 地址为
    以及 IPv4 映射的 IPv6 地址。

## `dns.lookupService(address, port, callback)`

<!-- YAML
added: v0.11.14
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `address`{字符串}
*   `port`{数字}
*   `callback`{函数}
    *   `err`{错误}
    *   `hostname`{字符串} 例如`example.com`
    *   `service`{字符串} 例如`http`

解析给定的`address`和`port`使用到主机名和服务中
操作系统的底层`getnameinfo`实现。

如果`address`不是有效的 IP 地址，一个`TypeError`将被抛出。
这`port`将被强制为一个数字。如果它不是合法港口，则`TypeError`
将被抛出。

如果出现错误，`err`是一个[`Error`][Error]对象，其中`err.code`是错误代码。

```js
const dns = require('node:dns');
dns.lookupService('127.0.0.1', 22, (err, hostname, service) => {
  console.log(hostname, service);
  // Prints: localhost ssh
});
```

如果调用此方法作为其[`util.promisify()`][util.promisify()]ed 版本，它返回一个
`Promise`对于`Object`跟`hostname`和`service`性能。

## `dns.resolve(hostname[, rrtype], callback)`

<!-- YAML
added: v0.1.27
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}要解析的主机名。
*   `rrtype`{字符串}资源记录类型。**违约：** `'A'`.
*   `callback`{函数}
    *   `err`{错误}
    *   `records`{字符串\[] |对象 \[] |对象}

使用 DNS 协议解析主机名（例如`'nodejs.org'`） 放入数组中
的资源记录。这`callback`函数有参数
`(err, records)`.成功后，`records`将是一个资源数组
记录。单个结果的类型和结构因`rrtype`:

|`rrtype`|`records`包含 |结果类型 |速记方法|
|--------- |------------------------------ |----------- |------------------------ |
|`'A'`|IPv4 地址（默认）|{字符串} |[`dns.resolve4()`][dns.resolve4()]|
|`'AAAA'`||的 IPv6 地址{字符串} |[`dns.resolve6()`][dns.resolve6()]|
|`'ANY'`|任何记录|{对象} |[`dns.resolveAny()`][dns.resolveAny()]|
|`'CAA'`|CA 授权记录|{对象} |[`dns.resolveCaa()`][dns.resolveCaa()]|
|`'CNAME'`|规范名称记录|{字符串} |[`dns.resolveCname()`][dns.resolveCname()]|
|`'MX'`|邮件交换记录|{对象} |[`dns.resolveMx()`][dns.resolveMx()]|
|`'NAPTR'`|名称颁发机构指针记录|{对象} |[`dns.resolveNaptr()`][dns.resolveNaptr()]|
|`'NS'`|名称服务器记录|{字符串} |[`dns.resolveNs()`][dns.resolveNs()]|
|`'PTR'`|指针记录|{字符串} |[`dns.resolvePtr()`][dns.resolvePtr()]|
|`'SOA'`|启动机构记录|{对象} |[`dns.resolveSoa()`][dns.resolveSoa()]|
|`'SRV'`|服务记录|{对象} |[`dns.resolveSrv()`][dns.resolveSrv()]|
|`'TXT'`|文本记录|{字符串\[]} |[`dns.resolveTxt()`][dns.resolveTxt()]|

出错时，`err`是一个[`Error`][Error]对象，其中`err.code`是其中之一
[域名解析错误代码][DNS error codes].

## `dns.resolve4(hostname[, options], callback)`

<!-- YAML
added: v0.1.16
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9296
    description: This method now supports passing `options`,
                 specifically `options.ttl`.
-->

*   `hostname`{字符串}要解析的主机名。
*   `options`{对象}
    *   `ttl`{布尔值}检索每条记录的生存时间值 （TTL）。
        什么时候`true`，回调接收到一个数组
        `{ address: '1.2.3.4', ttl: 60 }`对象而不是字符串数组，
        TTL以秒为单位表示。
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{字符串\[] |对象\[]}

使用 DNS 协议解析 IPv4 地址 （`A`记录）
`hostname`.这`addresses`参数传递给`callback`功能
将包含一个 IPv4 地址数组（例如
`['74.125.79.104', '74.125.79.105', '74.125.79.106']`).

## `dns.resolve6(hostname[, options], callback)`

<!-- YAML
added: v0.1.16
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9296
    description: This method now supports passing `options`,
                 specifically `options.ttl`.
-->

*   `hostname`{字符串}要解析的主机名。
*   `options`{对象}
    *   `ttl`{布尔值}检索每条记录的生存时间值 （TTL）。
        什么时候`true`，回调接收到一个数组
        `{ address: '0:1:2:3:4:5:6:7', ttl: 60 }`对象而不是数组
        字符串，TTL 以秒为单位表示。
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{字符串\[] |对象\[]}

使用 DNS 协议解析 IPv6 地址 （`AAAA`记录）
`hostname`.这`addresses`参数传递给`callback`功能
将包含一个 IPv6 地址数组。

## `dns.resolveAny(hostname, callback)`

<!-- YAML
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `ret`{对象\[]}

使用 DNS 协议解析所有记录（也称为`ANY`或`*`查询）。
这`ret`参数传递给`callback`函数将是一个数组，其中包含
各种类型的记录。每个对象都有一个属性`type`表示
当前记录的类型。并且取决于`type`、其他属性
将出现在对象上：

|类型 |属性 |
|--------- |------------------------------------------------------------------------------------------------------------------------------------------------ |
|`'A'`|`address`/`ttl`|
|`'AAAA'`|`address`/`ttl`|
|`'CNAME'`|`value`|
|`'MX'`|指[`dns.resolveMx()`][dns.resolveMx()]|
|`'NAPTR'`|指[`dns.resolveNaptr()`][dns.resolveNaptr()]|
|`'NS'`|`value`|
|`'PTR'`|`value`|
|`'SOA'`|指[`dns.resolveSoa()`][dns.resolveSoa()]|
|`'SRV'`|指[`dns.resolveSrv()`][dns.resolveSrv()]|
|`'TXT'`|这种类型的记录包含一个名为`entries`这是指[`dns.resolveTxt()`][dns.resolveTxt()]，例如`{ entries: ['...'], type: 'TXT' }`|

下面是一个示例`ret`传递给回调的对象：

<!-- eslint-disable semi -->

```js
[ { type: 'A', address: '127.0.0.1', ttl: 299 },
  { type: 'CNAME', value: 'example.com' },
  { type: 'MX', exchange: 'alt4.aspmx.l.example.com', priority: 50 },
  { type: 'NS', value: 'ns1.example.com' },
  { type: 'TXT', entries: [ 'v=spf1 include:_spf.example.com ~all' ] },
  { type: 'SOA',
    nsname: 'ns1.example.com',
    hostmaster: 'admin.example.com',
    serial: 156696742,
    refresh: 900,
    retry: 900,
    expire: 1800,
    minttl: 60 } ]
```

DNS服务器运营商可以选择不响应`ANY`
查询。最好调用单个方法，例如[`dns.resolve4()`][dns.resolve4()],
[`dns.resolveMx()`][dns.resolveMx()]，依此类推。有关更多详细信息，请参阅[RFC 8482][].

## `dns.resolveCname(hostname, callback)`

<!-- YAML
added: v0.3.2
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{字符串\[]}

使用 DNS 协议进行解析`CNAME`的记录`hostname`.这
`addresses`参数传递给`callback`功能
将包含一个可用于`hostname`
（例如`['bar.example.com']`).

## `dns.resolveCaa(hostname, callback)`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `records`{对象\[]}

使用 DNS 协议进行解析`CAA`的记录`hostname`.这
`addresses`参数传递给`callback`功能
将包含一组证书颁发机构授权记录
可用于`hostname`（例如`[{critical: 0, iodef:
'mailto:pki@example.com'}, {critical: 128, issue: 'pki.example.com'}]`).

## `dns.resolveMx(hostname, callback)`

<!-- YAML
added: v0.1.27
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{对象\[]}

使用 DNS 协议解析邮件交换记录 （`MX`记录）
`hostname`.这`addresses`参数传递给`callback`函数将
包含一个对象数组，其中包含`priority`和`exchange`
属性（例如`[{priority: 10, exchange: 'mx.example.com'}, ...]`).

## `dns.resolveNaptr(hostname, callback)`

<!-- YAML
added: v0.9.12
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{对象\[]}

使用 DNS 协议解析基于正则表达式的记录 （`NAPTR`
记录）`hostname`.这`addresses`参数传递给`callback`
函数将包含具有以下属性的对象数组：

*   `flags`
*   `service`
*   `regexp`
*   `replacement`
*   `order`
*   `preference`

<!-- eslint-skip -->

```js
{
  flags: 's',
  service: 'SIP+D2U',
  regexp: '',
  replacement: '_sip._udp.example.com',
  order: 30,
  preference: 100
}
```

## `dns.resolveNs(hostname, callback)`

<!-- YAML
added: v0.1.90
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{字符串\[]}

使用 DNS 协议解析名称服务器记录 （`NS`记录）
`hostname`.这`addresses`参数传递给`callback`函数将
包含可用于`hostname`
（例如`['ns1.example.com', 'ns2.example.com']`).

## `dns.resolvePtr(hostname, callback)`

<!-- YAML
added: v6.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{字符串\[]}

使用 DNS 协议解析指针记录 （`PTR`记录）
`hostname`.这`addresses`参数传递给`callback`函数将
是包含回复记录的字符串数组。

## `dns.resolveSoa(hostname, callback)`

<!-- YAML
added: v0.11.10
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `address`{对象}

使用 DNS 协议解析颁发机构记录的开始 （`SOA`记录）
这`hostname`.这`address`参数传递给`callback`函数将
是具有以下属性的对象：

*   `nsname`
*   `hostmaster`
*   `serial`
*   `refresh`
*   `retry`
*   `expire`
*   `minttl`

<!-- eslint-skip -->

```js
{
  nsname: 'ns.example.com',
  hostmaster: 'root.example.com',
  serial: 2013101809,
  refresh: 10000,
  retry: 2400,
  expire: 604800,
  minttl: 3600
}
```

## `dns.resolveSrv(hostname, callback)`

<!-- YAML
added: v0.1.27
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `addresses`{对象\[]}

使用 DNS 协议解析服务记录 （`SRV`记录）
`hostname`.这`addresses`参数传递给`callback`函数将
是具有以下属性的对象数组：

*   `priority`
*   `weight`
*   `port`
*   `name`

<!-- eslint-skip -->

```js
{
  priority: 10,
  weight: 5,
  port: 21223,
  name: 'service.example.com'
}
```

## `dns.resolveTxt(hostname, callback)`

<!-- YAML
added: v0.1.27
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

<!--lint disable no-undefined-references list-item-bullet-indent-->

*   `hostname`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `records` <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type" class="type"><串\[]\[]></a>

<!--lint enable no-undefined-references list-item-bullet-indent-->

使用 DNS 协议解析文本查询 （`TXT`记录）
`hostname`.这`records`参数传递给`callback`函数是一个
文本记录的二维数组可用于`hostname`（例如
`[ ['v=spf1 ip4:0.0.0.0 ', '~all' ] ]`).每个子数组都包含 TXT 块
一条记录。根据用例，它们可以连接在一起，也可以连接在一起
单独处理。

## `dns.reverse(ip, callback)`

<!-- YAML
added: v0.1.16
-->

*   `ip`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `hostnames`{字符串\[]}

执行反向 DNS 查询，将 IPv4 或 IPv6 地址解析为
主机名的数组。

出错时，`err`是一个[`Error`][Error]对象，其中`err.code`是
其中之一[域名解析错误代码][DNS error codes].

## `dns.setDefaultResultOrder(order)`

<!-- YAML
added:
  - v16.4.0
  - v14.18.0
-->

*   `order`{字符串} 必须是`'ipv4first'`或`'verbatim'`.

将默认值设置为`verbatim`在[`dns.lookup()`][dns.lookup()]和
[`dnsPromises.lookup()`][dnsPromises.lookup()].该值可以是：

*   `ipv4first`：设置默认值`verbatim` `false`.
*   `verbatim`：设置默认值`verbatim` `true`.

默认值为`ipv4first`和[`dns.setDefaultResultOrder()`][dns.setDefaultResultOrder()]有更高的
优先级比[`--dns-result-order`][--dns-result-order].使用时[工作线程][worker threads],
[`dns.setDefaultResultOrder()`][dns.setDefaultResultOrder()]从主线程不会影响默认值
工作人员中的 dns 命令。

## `dns.setServers(servers)`

<!-- YAML
added: v0.11.3
-->

*   `servers`{字符串\[]} 数组[RFC 5952][]格式化地址

设置执行 DNS 时要使用的服务器的 IP 地址和端口
分辨率。这`servers`参数是[RFC 5952][]格式 化
地址。如果端口是 IANA 默认 DNS 端口 （53），则可以省略该端口。

```js
dns.setServers([
  '4.4.4.4',
  '[2001:4860:4860::8888]',
  '4.4.4.4:1053',
  '[2001:4860:4860::8888]:1053',
]);
```

如果提供的地址无效，将引发错误。

这`dns.setServers()`当 DNS 查询位于
进展。

这[`dns.setServers()`][dns.setServers()]方法仅影响[`dns.resolve()`][dns.resolve()],
`dns.resolve*()`和[`dns.reverse()`][dns.reverse()]（特别是*不*
[`dns.lookup()`][dns.lookup()]).

此方法的工作方式很像
[resolve.conf](https://man7.org/linux/man-pages/man5/resolv.conf.5.html).
也就是说，如果尝试使用提供的第一台服务器进行解析，则会导致
`NOTFOUND`错误，`resolve()`方法将*不*尝试解析
提供后续服务器。仅当
较早的超时或导致其他错误。

## DNS promises API

<!-- YAML
added: v10.6.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/32953
    description: Exposed as `require('dns/promises')`.
  - version:
    - v11.14.0
    - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/26592
    description: This API is no longer experimental.
-->

这`dns.promises`API 提供了一组替代的异步 DNS 方法
返回`Promise`对象，而不是使用回调。该接口是可访问的
通过`require('node:dns').promises`或`require('node:dns/promises')`.

### 类：`dnsPromises.Resolver`

<!-- YAML
added: v10.6.0
-->

用于 DNS 请求的独立解析程序。

创建新的冲突解决程序将使用默认服务器设置。设置
用于冲突解决程序的服务器使用
[`resolver.setServers()`][`dnsPromises.setServers()`]不影响
其他解析器：

```js
const { Resolver } = require('node:dns').promises;
const resolver = new Resolver();
resolver.setServers(['4.4.4.4']);

// This request will use the server at 4.4.4.4, independent of global settings.
resolver.resolve4('example.org').then((addresses) => {
  // ...
});

// Alternatively, the same code can be written using async-await style.
(async function() {
  const addresses = await resolver.resolve4('example.org');
})();
```

以下方法从`dnsPromises`API 可用：

*   [`resolver.getServers()`][`dnsPromises.getServers()`]
*   [`resolver.resolve()`][`dnsPromises.resolve()`]
*   [`resolver.resolve4()`][`dnsPromises.resolve4()`]
*   [`resolver.resolve6()`][`dnsPromises.resolve6()`]
*   [`resolver.resolveAny()`][`dnsPromises.resolveAny()`]
*   [`resolver.resolveCaa()`][`dnsPromises.resolveCaa()`]
*   [`resolver.resolveCname()`][`dnsPromises.resolveCname()`]
*   [`resolver.resolveMx()`][`dnsPromises.resolveMx()`]
*   [`resolver.resolveNaptr()`][`dnsPromises.resolveNaptr()`]
*   [`resolver.resolveNs()`][`dnsPromises.resolveNs()`]
*   [`resolver.resolvePtr()`][`dnsPromises.resolvePtr()`]
*   [`resolver.resolveSoa()`][`dnsPromises.resolveSoa()`]
*   [`resolver.resolveSrv()`][`dnsPromises.resolveSrv()`]
*   [`resolver.resolveTxt()`][`dnsPromises.resolveTxt()`]
*   [`resolver.reverse()`][`dnsPromises.reverse()`]
*   [`resolver.setServers()`][`dnsPromises.setServers()`]

### `resolver.cancel()`

<!-- YAML
added:
  - v15.3.0
  - v14.17.0
-->

取消此解析程序进行的所有未完成的 DNS 查询。相应的
承诺将被拒绝，代码错误`ECANCELLED`.

### `dnsPromises.getServers()`

<!-- YAML
added: v10.6.0
-->

*   返回：{字符串\[]}

返回 IP 地址字符串的数组，其格式根据[RFC 5952][],
当前已针对 DNS 解析进行了配置。字符串将包含端口
部分（如果使用自定义端口）。

<!-- eslint-disable semi-->

```js
[
  '4.4.4.4',
  '2001:4860:4860::8888',
  '4.4.4.4:1053',
  '[2001:4860:4860::8888]:1053',
]
```

### `dnsPromises.lookup(hostname[, options])`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}
*   `options`{整数|对象}
    *   `family`{整数}记录系列。必须是`4`,`6`或`0`.价值
        `0`表示同时返回 IPv4 和 IPv6 地址。**违约：**
        `0`.
    *   `hints`{数字}一个或多个[支持`getaddrinfo`标志][supported getaddrinfo flags].倍数
        标志可以通过按位传递`OR`他们的价值观。
    *   `all`{布尔值}什么时候`true`这`Promise`使用中的所有地址解析
        一个数组。否则，返回单个地址。**违约：** `false`.
    *   `verbatim`{布尔值}什么时候`true`这`Promise`已使用 IPv4 解决，并且
        IPv6 地址按 DNS 解析程序返回的顺序排列。什么时候`false`,
        IPv4 地址位于 IPv6 地址之前。
        **违约：**现在`false`（地址被重新排序），但这是
        预计在不久的将来会发生变化。默认值为
        可配置使用[`dns.setDefaultResultOrder()`][dns.setDefaultResultOrder()]或
        [`--dns-result-order`][--dns-result-order].新代码应使用`{ verbatim: true }`.

解析主机名（例如`'nodejs.org'`） 添加到第一个找到的 A （IPv4） 或
AAAA （IPv6） 记录。都`option`属性是可选的。如果`options`是一个
整数，则它必须是`4`或`6`– 如果`options`未提供，则为 IPv4
如果找到，则同时返回 IPv6 地址。

随着`all`选项设置为`true`这`Promise`通过`addresses`
是具有属性的对象数组`address`和`family`.

出错时，`Promise`被拒绝与[`Error`][Error]对象，其中`err.code`
是错误代码。
请记住，`err.code`将设置为`'ENOTFOUND'`不仅在以下情况下
主机名不存在，但当查找以其他方式失败时也存在
例如，没有可用的文件描述符。

[`dnsPromises.lookup()`][dnsPromises.lookup()]不一定与 DNS 有任何关系
协议。该实现使用操作系统工具，该工具可以
将名称与地址相关联，反之亦然。此实现可以具有
对任何Node.js程序的行为产生微妙但重要的影响。请
花一些时间咨询[实现注意事项部分][Implementation considerations section]以前
用`dnsPromises.lookup()`.

用法示例：

```js
const dns = require('node:dns');
const dnsPromises = dns.promises;
const options = {
  family: 6,
  hints: dns.ADDRCONFIG | dns.V4MAPPED,
};

dnsPromises.lookup('example.com', options).then((result) => {
  console.log('address: %j family: IPv%s', result.address, result.family);
  // address: "2606:2800:220:1:248:1893:25c8:1946" family: IPv6
});

// When options.all is true, the result will be an Array.
options.all = true;
dnsPromises.lookup('example.com', options).then((result) => {
  console.log('addresses: %j', result);
  // addresses: [{"address":"2606:2800:220:1:248:1893:25c8:1946","family":6}]
});
```

### `dnsPromises.lookupService(address, port)`

<!-- YAML
added: v10.6.0
-->

*   `address`{字符串}
*   `port`{数字}

解析给定的`address`和`port`使用到主机名和服务中
操作系统的底层`getnameinfo`实现。

如果`address`不是有效的 IP 地址，一个`TypeError`将被抛出。
这`port`将被强制为一个数字。如果它不是合法港口，则`TypeError`
将被抛出。

出错时，`Promise`被拒绝与[`Error`][Error]对象，其中`err.code`
是错误代码。

```js
const dnsPromises = require('node:dns').promises;
dnsPromises.lookupService('127.0.0.1', 22).then((result) => {
  console.log(result.hostname, result.service);
  // Prints: localhost ssh
});
```

### `dnsPromises.resolve(hostname[, rrtype])`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}要解析的主机名。
*   `rrtype`{字符串}资源记录类型。**违约：** `'A'`.

使用 DNS 协议解析主机名（例如`'nodejs.org'`） 放入数组中
的资源记录。成功后，`Promise`通过
资源记录的数组。单个结果的类型和结构各不相同
基于`rrtype`:

|`rrtype`|`records`包含 |结果类型 |速记方法|
|--------- |------------------------------ |----------- |-------------------------------- |
|`'A'`|IPv4 地址（默认）|{字符串} |[`dnsPromises.resolve4()`][dnsPromises.resolve4()]|
|`'AAAA'`||的 IPv6 地址{字符串} |[`dnsPromises.resolve6()`][dnsPromises.resolve6()]|
|`'ANY'`|任何记录|{对象} |[`dnsPromises.resolveAny()`][dnsPromises.resolveAny()]|
|`'CAA'`|CA 授权记录|{对象} |[`dnsPromises.resolveCaa()`][dnsPromises.resolveCaa()]|
|`'CNAME'`|规范名称记录|{字符串} |[`dnsPromises.resolveCname()`][dnsPromises.resolveCname()]|
|`'MX'`|邮件交换记录|{对象} |[`dnsPromises.resolveMx()`][dnsPromises.resolveMx()]|
|`'NAPTR'`|名称颁发机构指针记录|{对象} |[`dnsPromises.resolveNaptr()`][dnsPromises.resolveNaptr()]|
|`'NS'`|名称服务器记录|{字符串} |[`dnsPromises.resolveNs()`][dnsPromises.resolveNs()]|
|`'PTR'`|指针记录|{字符串} |[`dnsPromises.resolvePtr()`][dnsPromises.resolvePtr()]|
|`'SOA'`|启动机构记录|{对象} |[`dnsPromises.resolveSoa()`][dnsPromises.resolveSoa()]|
|`'SRV'`|服务记录|{对象} |[`dnsPromises.resolveSrv()`][dnsPromises.resolveSrv()]|
|`'TXT'`|文本记录|{字符串\[]} |[`dnsPromises.resolveTxt()`][dnsPromises.resolveTxt()]|

出错时，`Promise`被拒绝与[`Error`][Error]对象，其中`err.code`
是其中之一[域名解析错误代码][DNS error codes].

### `dnsPromises.resolve4(hostname[, options])`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}要解析的主机名。
*   `options`{对象}
    *   `ttl`{布尔值}检索每条记录的生存时间值 （TTL）。
        什么时候`true`这`Promise`使用数组解析
        `{ address: '1.2.3.4', ttl: 60 }`对象而不是字符串数组，
        TTL以秒为单位表示。

使用 DNS 协议解析 IPv4 地址 （`A`记录）
`hostname`.关于成功，`Promise`使用 IPv4 数组解析
地址（例如`['74.125.79.104', '74.125.79.105', '74.125.79.106']`).

### `dnsPromises.resolve6(hostname[, options])`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}要解析的主机名。
*   `options`{对象}
    *   `ttl`{布尔值}检索每条记录的生存时间值 （TTL）。
        什么时候`true`这`Promise`使用数组解析
        `{ address: '0:1:2:3:4:5:6:7', ttl: 60 }`对象而不是数组
        字符串，TTL 以秒为单位表示。

使用 DNS 协议解析 IPv6 地址 （`AAAA`记录）
`hostname`.关于成功，`Promise`使用 IPv6 数组解析
地址。

### `dnsPromises.resolveAny(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析所有记录（也称为`ANY`或`*`查询）。
关于成功，`Promise`使用包含各种类型的
记录。每个对象都有一个属性`type`指示的类型
当前记录。并且取决于`type`，则其他属性将为
出现在对象上：

|类型 |属性 |
|--------- |-------------------------------------------------------------------------------------------------------------------------------------------------------- |
|`'A'`|`address`/`ttl`|
|`'AAAA'`|`address`/`ttl`|
|`'CNAME'`|`value`|
|`'MX'`|指[`dnsPromises.resolveMx()`][dnsPromises.resolveMx()]|
|`'NAPTR'`|指[`dnsPromises.resolveNaptr()`][dnsPromises.resolveNaptr()]|
|`'NS'`|`value`|
|`'PTR'`|`value`|
|`'SOA'`|指[`dnsPromises.resolveSoa()`][dnsPromises.resolveSoa()]|
|`'SRV'`|指[`dnsPromises.resolveSrv()`][dnsPromises.resolveSrv()]|
|`'TXT'`|这种类型的记录包含一个名为`entries`这是指[`dnsPromises.resolveTxt()`][dnsPromises.resolveTxt()]，例如`{ entries: ['...'], type: 'TXT' }`|

下面是结果对象的示例：

<!-- eslint-disable semi -->

```js
[ { type: 'A', address: '127.0.0.1', ttl: 299 },
  { type: 'CNAME', value: 'example.com' },
  { type: 'MX', exchange: 'alt4.aspmx.l.example.com', priority: 50 },
  { type: 'NS', value: 'ns1.example.com' },
  { type: 'TXT', entries: [ 'v=spf1 include:_spf.example.com ~all' ] },
  { type: 'SOA',
    nsname: 'ns1.example.com',
    hostmaster: 'admin.example.com',
    serial: 156696742,
    refresh: 900,
    retry: 900,
    expire: 1800,
    minttl: 60 } ]
```

### `dnsPromises.resolveCaa(hostname)`

<!-- YAML
added:
  - v15.0.0
  - v14.17.0
-->

*   `hostname`{字符串}

使用 DNS 协议进行解析`CAA`的记录`hostname`.关于成功，
这`Promise`使用包含可用对象的数组进行解析
证书颁发机构授权记录可用于`hostname`
（例如`[{critical: 0, iodef: 'mailto:pki@example.com'},{critical: 128, issue:
'pki.example.com'}]`).

### `dnsPromises.resolveCname(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议进行解析`CNAME`的记录`hostname`.关于成功，
这`Promise`使用可用于
这`hostname`（例如`['bar.example.com']`).

### `dnsPromises.resolveMx(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析邮件交换记录 （`MX`记录）
`hostname`.关于成功，`Promise`使用对象数组解析
同时包含`priority`和`exchange`属性（例如
`[{priority: 10, exchange: 'mx.example.com'}, ...]`).

### `dnsPromises.resolveNaptr(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析基于正则表达式的记录 （`NAPTR`
记录）`hostname`.关于成功，`Promise`使用数组解析
具有以下属性的对象：

*   `flags`
*   `service`
*   `regexp`
*   `replacement`
*   `order`
*   `preference`

<!-- eslint-skip -->

```js
{
  flags: 's',
  service: 'SIP+D2U',
  regexp: '',
  replacement: '_sip._udp.example.com',
  order: 30,
  preference: 100
}
```

### `dnsPromises.resolveNs(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析名称服务器记录 （`NS`记录）
`hostname`.关于成功，`Promise`使用名称服务器数组进行解析
记录可用于`hostname`（例如
`['ns1.example.com', 'ns2.example.com']`).

### `dnsPromises.resolvePtr(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析指针记录 （`PTR`记录）
`hostname`.关于成功，`Promise`使用字符串数组解析
包含回复记录。

### `dnsPromises.resolveSoa(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析颁发机构记录的开始 （`SOA`记录）
这`hostname`.关于成功，`Promise`使用具有以下
以下属性：

*   `nsname`
*   `hostmaster`
*   `serial`
*   `refresh`
*   `retry`
*   `expire`
*   `minttl`

<!-- eslint-skip -->

```js
{
  nsname: 'ns.example.com',
  hostmaster: 'root.example.com',
  serial: 2013101809,
  refresh: 10000,
  retry: 2400,
  expire: 604800,
  minttl: 3600
}
```

### `dnsPromises.resolveSrv(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析服务记录 （`SRV`记录）
`hostname`.关于成功，`Promise`使用对象数组解析
以下属性：

*   `priority`
*   `weight`
*   `port`
*   `name`

<!-- eslint-skip -->

```js
{
  priority: 10,
  weight: 5,
  port: 21223,
  name: 'service.example.com'
}
```

### `dnsPromises.resolveTxt(hostname)`

<!-- YAML
added: v10.6.0
-->

*   `hostname`{字符串}

使用 DNS 协议解析文本查询 （`TXT`记录）
`hostname`.关于成功，`Promise`使用二维数组解析
的文本记录可用于`hostname`（例如
`[ ['v=spf1 ip4:0.0.0.0 ', '~all' ] ]`).每个子数组都包含 TXT 块
一条记录。根据用例，它们可以连接在一起，也可以连接在一起
单独处理。

### `dnsPromises.reverse(ip)`

<!-- YAML
added: v10.6.0
-->

*   `ip`{字符串}

执行反向 DNS 查询，将 IPv4 或 IPv6 地址解析为
主机名的数组。

出错时，`Promise`被拒绝与[`Error`][Error]对象，其中`err.code`
是其中之一[域名解析错误代码][DNS error codes].

### `dnsPromises.setDefaultResultOrder(order)`

<!-- YAML
added:
  - v16.4.0
  - v14.18.0
-->

*   `order`{字符串} 必须是`'ipv4first'`或`'verbatim'`.

将默认值设置为`verbatim`在[`dns.lookup()`][dns.lookup()]和
[`dnsPromises.lookup()`][dnsPromises.lookup()].该值可以是：

*   `ipv4first`：设置默认值`verbatim` `false`.
*   `verbatim`：设置默认值`verbatim` `true`.

默认值为`ipv4first`和[`dnsPromises.setDefaultResultOrder()`][dnsPromises.setDefaultResultOrder()]有
优先级高于[`--dns-result-order`][--dns-result-order].使用时[工作线程][worker threads],
[`dnsPromises.setDefaultResultOrder()`][dnsPromises.setDefaultResultOrder()]从主线程不会影响
工作线程中的默认 dns 顺序。

### `dnsPromises.setServers(servers)`

<!-- YAML
added: v10.6.0
-->

*   `servers`{字符串\[]} 数组[RFC 5952][]格式化地址

设置执行 DNS 时要使用的服务器的 IP 地址和端口
分辨率。这`servers`参数是[RFC 5952][]格式 化
地址。如果端口是 IANA 默认 DNS 端口 （53），则可以省略该端口。

```js
dnsPromises.setServers([
  '4.4.4.4',
  '[2001:4860:4860::8888]',
  '4.4.4.4:1053',
  '[2001:4860:4860::8888]:1053',
]);
```

如果提供的地址无效，将引发错误。

这`dnsPromises.setServers()`当 DNS 查询位于
进展。

此方法的工作方式很像
[resolve.conf](https://man7.org/linux/man-pages/man5/resolv.conf.5.html).
也就是说，如果尝试使用提供的第一台服务器进行解析，则会导致
`NOTFOUND`错误，`resolve()`方法将*不*尝试解析
提供后续服务器。仅当
较早的超时或导致其他错误。

## 错误代码

每个 DNS 查询都可能返回以下错误代码之一：

*   `dns.NODATA`：DNS 服务器返回了没有数据的应答。
*   `dns.FORMERR`：DNS 服务器声明查询格式错误。
*   `dns.SERVFAIL`：DNS 服务器返回常规故障。
*   `dns.NOTFOUND`：找不到域名。
*   `dns.NOTIMP`：DNS 服务器不实现请求的操作。
*   `dns.REFUSED`：DNS 服务器拒绝查询。
*   `dns.BADQUERY`：DNS 查询格式错误。
*   `dns.BADNAME`：主机名格式错误。
*   `dns.BADFAMILY`：不支持的地址系列。
*   `dns.BADRESP`：DNS 回复格式错误。
*   `dns.CONNREFUSED`：无法联系 DNS 服务器。
*   `dns.TIMEOUT`：联系 DNS 服务器时超时。
*   `dns.EOF`：文件结尾。
*   `dns.FILE`：读取文件时出错。
*   `dns.NOMEM`：内存不足。
*   `dns.DESTRUCTION`：通道正在被破坏。
*   `dns.BADSTR`：字符串格式错误。
*   `dns.BADFLAGS`：指定非法标志。
*   `dns.NONAME`：给定的主机名不是数字。
*   `dns.BADHINTS`：指定了非法提示标志。
*   `dns.NOTINITIALIZED`：尚未执行 c-ares 库初始化。
*   `dns.LOADIPHLPAPI`：加载时出错`iphlpapi.dll`.
*   `dns.ADDRGETNETWORKPARAMS`： 找不到`GetNetworkParams`功能。
*   `dns.CANCELLED`：已取消 DNS 查询。

这`dnsPromises`API 还会导出上述错误代码，例如：`dnsPromises.NODATA`.

## 实施注意事项

虽然[`dns.lookup()`][dns.lookup()]和各种`dns.resolve*()/dns.reverse()`
函数具有相同的目标，即将网络名称与网络相关联
地址（反之亦然），他们的行为是完全不同的。这些差异
可能会对 Node 的行为产生微妙但重大的影响.js
程序。

### `dns.lookup()`

在引擎盖下，[`dns.lookup()`][dns.lookup()]使用相同的操作系统工具
与大多数其他程序一样。例如[`dns.lookup()`][dns.lookup()]几乎总是
以与`ping`命令。在大多数类似POSIX的
操作系统， 的行为[`dns.lookup()`][dns.lookup()]函数可以是
通过更改 nsswitch.conf（5） 和/或 resolv.conf（5） 中的设置进行修改，
但更改这些文件将更改所有其他文件的行为
在同一操作系统上运行的程序。

虽然呼吁`dns.lookup()`将从 JavaScript 的
透视，它被实现为对 getaddrinfo（3） 的同步调用，运行
在 libuv 的 threadpool 上。这可能会产生令人惊讶的负面表现
对某些应用程序的影响，请参阅[`UV_THREADPOOL_SIZE`][UV_THREADPOOL_SIZE]
文档以获取更多信息。

各种网络 API 将调用`dns.lookup()`在内部解决
主机名。如果这是一个问题，请考虑将主机名解析为某个地址
用`dns.resolve()`并使用地址而不是主机名。另外，一些
网络 API（例如[`socket.connect()`][socket.connect()]和[`dgram.createSocket()`][dgram.createSocket()])
允许默认解析程序，`dns.lookup()`，以进行替换。

### `dns.resolve()`,`dns.resolve*()`和`dns.reverse()`

这些函数的实现方式与[`dns.lookup()`][dns.lookup()].他们
不使用 getaddrinfo（3） 和他们*总是*在
网络。此网络通信始终以异步方式完成，并且不会
使用 libuv 的 threadpool。

因此，这些功能不能对其他功能产生相同的负面影响。
在 libuv 的 threadpool 上发生的处理[`dns.lookup()`][dns.lookup()]可以有。

它们不使用同一组配置文件，而不是[`dns.lookup()`][dns.lookup()]
使用。例如*他们不使用来自`/etc/hosts`*.

[DNS error codes]: #error-codes

[Domain Name System (DNS)]: https://en.wikipedia.org/wiki/Domain_Name_System

[Implementation considerations section]: #implementation-considerations

[RFC 5952]: https://tools.ietf.org/html/rfc5952#section-6

[RFC 8482]: https://tools.ietf.org/html/rfc8482

[`--dns-result-order`]: cli.md#--dns-result-orderorder

[`Error`]: errors.md#class-error

[`UV_THREADPOOL_SIZE`]: cli.md#uv_threadpool_sizesize

[`dgram.createSocket()`]: dgram.md#dgramcreatesocketoptions-callback

[`dns.getServers()`]: #dnsgetservers

[`dns.lookup()`]: #dnslookuphostname-options-callback

[`dns.resolve()`]: #dnsresolvehostname-rrtype-callback

[`dns.resolve4()`]: #dnsresolve4hostname-options-callback

[`dns.resolve6()`]: #dnsresolve6hostname-options-callback

[`dns.resolveAny()`]: #dnsresolveanyhostname-callback

[`dns.resolveCaa()`]: #dnsresolvecaahostname-callback

[`dns.resolveCname()`]: #dnsresolvecnamehostname-callback

[`dns.resolveMx()`]: #dnsresolvemxhostname-callback

[`dns.resolveNaptr()`]: #dnsresolvenaptrhostname-callback

[`dns.resolveNs()`]: #dnsresolvenshostname-callback

[`dns.resolvePtr()`]: #dnsresolveptrhostname-callback

[`dns.resolveSoa()`]: #dnsresolvesoahostname-callback

[`dns.resolveSrv()`]: #dnsresolvesrvhostname-callback

[`dns.resolveTxt()`]: #dnsresolvetxthostname-callback

[`dns.reverse()`]: #dnsreverseip-callback

[`dns.setDefaultResultOrder()`]: #dnssetdefaultresultorderorder

[`dns.setServers()`]: #dnssetserversservers

[`dnsPromises.getServers()`]: #dnspromisesgetservers

[`dnsPromises.lookup()`]: #dnspromiseslookuphostname-options

[`dnsPromises.resolve()`]: #dnspromisesresolvehostname-rrtype

[`dnsPromises.resolve4()`]: #dnspromisesresolve4hostname-options

[`dnsPromises.resolve6()`]: #dnspromisesresolve6hostname-options

[`dnsPromises.resolveAny()`]: #dnspromisesresolveanyhostname

[`dnsPromises.resolveCaa()`]: #dnspromisesresolvecaahostname

[`dnsPromises.resolveCname()`]: #dnspromisesresolvecnamehostname

[`dnsPromises.resolveMx()`]: #dnspromisesresolvemxhostname

[`dnsPromises.resolveNaptr()`]: #dnspromisesresolvenaptrhostname

[`dnsPromises.resolveNs()`]: #dnspromisesresolvenshostname

[`dnsPromises.resolvePtr()`]: #dnspromisesresolveptrhostname

[`dnsPromises.resolveSoa()`]: #dnspromisesresolvesoahostname

[`dnsPromises.resolveSrv()`]: #dnspromisesresolvesrvhostname

[`dnsPromises.resolveTxt()`]: #dnspromisesresolvetxthostname

[`dnsPromises.reverse()`]: #dnspromisesreverseip

[`dnsPromises.setDefaultResultOrder()`]: #dnspromisessetdefaultresultorderorder

[`dnsPromises.setServers()`]: #dnspromisessetserversservers

[`socket.connect()`]: net.md#socketconnectoptions-connectlistener

[`util.promisify()`]: util.md#utilpromisifyoriginal

[supported `getaddrinfo` flags]: #supported-getaddrinfo-flags

[worker threads]: worker_threads.md
