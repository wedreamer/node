# 断续器

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/https.js -->

HTTPS是TLS/SSL上的HTTP协议。在 Node 中.js这是作为
单独的模块。

## 确定加密支持是否不可用

.js 可以在不包含对
`node:crypto`模块。在这种情况下，尝试`import`从`https`或
叫`require('node:https')`将导致引发错误。

使用 CommonJS 时，可以使用 try/catch 捕获引发的错误：

<!-- eslint-skip -->

```cjs
let https;
try {
  https = require('node:https');
} catch (err) {
  console.log('https support is disabled!');
}
```

使用词法 ESM 时`import`关键字，错误只能是
如果处理程序`process.on('uncaughtException')`已注册
*以前*进行任何加载模块的尝试（例如，使用
预加载模块）。

使用 ESM 时，如果代码有可能在构建版本上运行
的节点.js未启用加密支持，请考虑使用
[`import()`][import()]函数而不是词法`import`关键词：

```mjs
let https;
try {
  https = await import('node:https');
} catch (err) {
  console.log('https support is disabled!');
}
```

## 类：`https.Agent`

<!-- YAML
added: v0.4.5
changes:
  - version: v5.3.0
    pr-url: https://github.com/nodejs/node/pull/4252
    description: support `0` `maxCachedSessions` to disable TLS session caching.
  - version: v2.5.0
    pr-url: https://github.com/nodejs/node/pull/2228
    description: parameter `maxCachedSessions` added to `options` for TLS
                 sessions reuse.
-->

一[`Agent`][Agent]对象用于 HTTPS，类似于[`http.Agent`][http.Agent].看
[`https.request()`][https.request()]了解更多信息。

### `new Agent([options])`

<!-- YAML
changes:
  - version: v12.5.0
    pr-url: https://github.com/nodejs/node/pull/28209
    description: do not automatically set servername if the target host was
                 specified using an IP address.
-->

*   `options`{对象}要在代理上设置的可配置选项集。
    可以具有与 相同的字段[`http.Agent(options)`][http.Agent(options)]和
    *   `maxCachedSessions`{数量} TLS 缓存会话的最大数量。
        用`0`以禁用 TLS 会话缓存。**违约：** `100`.
    *   `servername`{字符串} 的值
        [服务器名称指示扩展][sni wiki]以发送到服务器。用
        空字符串`''`以禁用发送扩展程序。
        **违约：**目标服务器的主机名，除非目标服务器
        使用 IP 地址指定，在这种情况下，默认值为`''`（否
        扩展名）。

        看[`Session Resumption`][Session Resumption]以获取有关 TLS 会话重用的信息。

#### 事件：`'keylog'`

<!-- YAML
added:
 - v13.2.0
 - v12.16.0
-->

*   `line`{缓冲区}ASCII 文本行，在 NSS 中`SSLKEYLOGFILE`格式。
*   `tlsSocket`{tls.TLSSocket} The`tls.TLSSocket`它所在的实例
    生成。

这`keylog`事件在由
由此代理管理的连接（通常在握手完成之前，但
不一定）。可以存储此键控材料以进行调试，因为它
允许对捕获的 TLS 流量进行解密。它可能被多次发射
对于每个套接字。

一个典型的用例是将接收的行追加到一个通用的文本文件中，即
后来被软件（如Wireshark）用于解密流量：

```js
// ...
https.globalAgent.on('keylog', (line, tlsSocket) => {
  fs.appendFileSync('/tmp/ssl-keys.log', line, { mode: 0o600 });
});
```

## 类：`https.Server`

<!-- YAML
added: v0.3.4
-->

*   扩展：{tls。服务器}

看[`http.Server`][http.Server]了解更多信息。

### `server.close([callback])`

<!-- YAML
added: v0.1.90
-->

*   `callback`{函数}
*   返回：{https。服务器}

看[`server.close()`][server.close()]在`node:http`模块。

### `server.closeAllConnections()`

<!-- YAML
added: v18.2.0
-->

看[`server.closeAllConnections()`][server.closeAllConnections()]在`node:http`模块。

### `server.closeIdleConnections()`

<!-- YAML
added: v18.2.0
-->

看[`server.closeIdleConnections()`][server.closeIdleConnections()]在`node:http`模块。

### `server.headersTimeout`

<!-- YAML
added: v11.3.0
-->

*   {数字}**违约：** `60000`

看[`server.headersTimeout`][server.headersTimeout]在`node:http`模块。

### `server.listen()`

启动侦听加密连接的 HTTPS 服务器。
此方法与[`server.listen()`][server.listen()]从[`net.Server`][net.Server].

### `server.maxHeadersCount`

*   {数字}**违约：** `2000`

看[`server.maxHeadersCount`][server.maxHeadersCount]在`node:http`模块。

### `server.requestTimeout`

<!-- YAML
added: v14.11.0
-->

*   {数字}**违约：** `0`

看[`server.requestTimeout`][server.requestTimeout]在`node:http`模块。

### `server.setTimeout([msecs][, callback])`

<!-- YAML
added: v0.11.2
-->

*   `msecs`{数字}**违约：** `120000`（2分钟）
*   `callback`{函数}
*   返回：{https。服务器}

看[`server.setTimeout()`][server.setTimeout()]在`node:http`模块。

### `server.timeout`

<!-- YAML
added: v0.11.2
changes:
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27558
    description: The default timeout changed from 120s to 0 (no timeout).
-->

*   {数字}**违约：**0（无超时）

看[`server.timeout`][server.timeout]在`node:http`模块。

### `server.keepAliveTimeout`

<!-- YAML
added: v8.0.0
-->

*   {数字}**违约：** `5000`（5 秒）

看[`server.keepAliveTimeout`][server.keepAliveTimeout]在`node:http`模块。

## `https.createServer([options][, requestListener])`

<!-- YAML
added: v0.3.4
-->

*   `options`{对象}接受`options`从[`tls.createServer()`][tls.createServer()],
    [`tls.createSecureContext()`][tls.createSecureContext()]和[`http.createServer()`][http.createServer()].
*   `requestListener`{函数}要添加到`'request'`事件。
*   返回：{https。服务器}

```js
// curl -k https://localhost:8000/
const https = require('node:https');
const fs = require('node:fs');

const options = {
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('hello world\n');
}).listen(8000);
```

或

```js
const https = require('node:https');
const fs = require('node:fs');

const options = {
  pfx: fs.readFileSync('test/fixtures/test_cert.pfx'),
  passphrase: 'sample'
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('hello world\n');
}).listen(8000);
```

## `https.get(options[, callback])`

## `https.get(url[, options][, callback])`

<!-- YAML
added: v0.3.6
changes:
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/21616
    description: The `url` parameter can now be passed along with a separate
                 `options` object.
  - version: v7.5.0
    pr-url: https://github.com/nodejs/node/pull/10638
    description: The `options` parameter can be a WHATWG `URL` object.
-->

*   `url`{字符串|网址}
*   `options`{对象|字符串|URL} 接受相同的`options`如
    [`https.request()`][https.request()]，其中`method`始终设置为`GET`.
*   `callback`{函数}

喜欢[`http.get()`][http.get()]但对于HTTPS。

`options`可以是对象、字符串或[`URL`][URL]对象。如果`options`是一个
字符串，它会自动解析为[`new URL()`][new URL()].如果是[`URL`][URL]
对象，它会自动转换为普通对象`options`对象。

```js
const https = require('node:https');

https.get('https://encrypted.google.com/', (res) => {
  console.log('statusCode:', res.statusCode);
  console.log('headers:', res.headers);

  res.on('data', (d) => {
    process.stdout.write(d);
  });

}).on('error', (e) => {
  console.error(e);
});
```

## `https.globalAgent`

<!-- YAML
added: v0.5.9
changes:
  - version:
      - REPLACEME
    pr-url: https://github.com/nodejs/node/pull/43522
    description: The agent now uses HTTP Keep-Alive by default.
-->

的全局实例[`https.Agent`][https.Agent]对于所有 HTTPS 客户端请求。

## `https.request(options[, callback])`

## `https.request(url[, options][, callback])`

<!-- YAML
added: v0.3.6
changes:
  - version:
      - v16.7.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/39310
    description: When using a `URL` object parsed username
                 and password will now be properly URI decoded.
  - version:
      - v14.1.0
      - v13.14.0
    pr-url: https://github.com/nodejs/node/pull/32786
    description: The `highWaterMark` option is accepted now.
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/21616
    description: The `url` parameter can now be passed along with a separate
                 `options` object.
  - version: v9.3.0
    pr-url: https://github.com/nodejs/node/pull/14903
    description: The `options` parameter can now include `clientCertEngine`.
  - version: v7.5.0
    pr-url: https://github.com/nodejs/node/pull/10638
    description: The `options` parameter can be a WHATWG `URL` object.
-->

*   `url`{字符串|网址}
*   `options`{对象|字符串|URL} 全部接受`options`从
    [`http.request()`][http.request()]，在默认值上有一些差异：
    *   `protocol` **违约：** `'https:'`
    *   `port` **违约：** `443`
    *   `agent` **违约：** `https.globalAgent`
*   `callback`{函数}
*   返回值：{http。客户端请求}

向安全的 Web 服务器发出请求。

以下附加功能`options`从[`tls.connect()`][tls.connect()]也接受：
`ca`,`cert`,`ciphers`,`clientCertEngine`,`crl`,`dhparam`,`ecdhCurve`,
`honorCipherOrder`,`key`,`passphrase`,`pfx`,`rejectUnauthorized`,
`secureOptions`,`secureProtocol`,`servername`,`sessionIdContext`,
`highWaterMark`.

`options`可以是对象、字符串或[`URL`][URL]对象。如果`options`是一个
字符串，它会自动解析为[`new URL()`][new URL()].如果是[`URL`][URL]
对象，它会自动转换为普通对象`options`对象。

`https.request()`返回[`http.ClientRequest`][http.ClientRequest]
类。这`ClientRequest`实例是可写流。如果需要
上传带有 POST 请求的文件，然后写入`ClientRequest`对象。

```js
const https = require('node:https');

const options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET'
};

const req = https.request(options, (res) => {
  console.log('statusCode:', res.statusCode);
  console.log('headers:', res.headers);

  res.on('data', (d) => {
    process.stdout.write(d);
  });
});

req.on('error', (e) => {
  console.error(e);
});
req.end();
```

使用选项的示例[`tls.connect()`][tls.connect()]:

```js
const options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET',
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};
options.agent = new https.Agent(options);

const req = https.request(options, (res) => {
  // ...
});
```

或者，通过不使用[`Agent`][Agent].

```js
const options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET',
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem'),
  agent: false
};

const req = https.request(options, (res) => {
  // ...
});
```

使用的示例[`URL`][URL]如`options`:

```js
const options = new URL('https://abc:xyz@example.com');

const req = https.request(options, (res) => {
  // ...
});
```

固定在证书指纹或公钥上的示例（类似于
`pin-sha256`):

```js
const tls = require('node:tls');
const https = require('node:https');
const crypto = require('node:crypto');

function sha256(s) {
  return crypto.createHash('sha256').update(s).digest('base64');
}
const options = {
  hostname: 'github.com',
  port: 443,
  path: '/',
  method: 'GET',
  checkServerIdentity: function(host, cert) {
    // Make sure the certificate is issued to the host we are connected to
    const err = tls.checkServerIdentity(host, cert);
    if (err) {
      return err;
    }

    // Pin the public key, similar to HPKP pin-sha256 pinning
    const pubkey256 = 'pL1+qb9HTMRZJmuC/bB/ZI9d302BYrrqiVuRyW+DGrU=';
    if (sha256(cert.pubkey) !== pubkey256) {
      const msg = 'Certificate verification error: ' +
        `The public key of '${cert.subject.CN}' ` +
        'does not match our pinned fingerprint';
      return new Error(msg);
    }

    // Pin the exact certificate, rather than the pub key
    const cert256 = '25:FE:39:32:D9:63:8C:8A:FC:A1:9A:29:87:' +
      'D8:3E:4C:1D:98:DB:71:E4:1A:48:03:98:EA:22:6A:BD:8B:93:16';
    if (cert.fingerprint256 !== cert256) {
      const msg = 'Certificate verification error: ' +
        `The certificate of '${cert.subject.CN}' ` +
        'does not match our pinned fingerprint';
      return new Error(msg);
    }

    // This loop is informational only.
    // Print the certificate and public key fingerprints of all certs in the
    // chain. Its common to pin the public key of the issuer on the public
    // internet, while pinning the public key of the service in sensitive
    // environments.
    do {
      console.log('Subject Common Name:', cert.subject.CN);
      console.log('  Certificate SHA256 fingerprint:', cert.fingerprint256);

      hash = crypto.createHash('sha256');
      console.log('  Public key ping-sha256:', sha256(cert.pubkey));

      lastprint256 = cert.fingerprint256;
      cert = cert.issuerCertificate;
    } while (cert.fingerprint256 !== lastprint256);

  },
};

options.agent = new https.Agent(options);
const req = https.request(options, (res) => {
  console.log('All OK. Server matched our pinned cert or public key');
  console.log('statusCode:', res.statusCode);
  // Print the HPKP values
  console.log('headers:', res.headers['public-key-pins']);

  res.on('data', (d) => {});
});

req.on('error', (e) => {
  console.error(e.message);
});
req.end();
```

输出例如：

```text
Subject Common Name: github.com
  Certificate SHA256 fingerprint: 25:FE:39:32:D9:63:8C:8A:FC:A1:9A:29:87:D8:3E:4C:1D:98:DB:71:E4:1A:48:03:98:EA:22:6A:BD:8B:93:16
  Public key ping-sha256: pL1+qb9HTMRZJmuC/bB/ZI9d302BYrrqiVuRyW+DGrU=
Subject Common Name: DigiCert SHA2 Extended Validation Server CA
  Certificate SHA256 fingerprint: 40:3E:06:2A:26:53:05:91:13:28:5B:AF:80:A0:D4:AE:42:2C:84:8C:9F:78:FA:D0:1F:C9:4B:C5:B8:7F:EF:1A
  Public key ping-sha256: RRM1dGqnDFsCJXBTHky16vi1obOlCgFFn/yOhI/y+ho=
Subject Common Name: DigiCert High Assurance EV Root CA
  Certificate SHA256 fingerprint: 74:31:E5:F4:C3:C1:CE:46:90:77:4F:0B:61:E0:54:40:88:3B:A9:A0:1E:D0:0B:A6:AB:D7:80:6E:D3:B1:18:CF
  Public key ping-sha256: WoiWRyIOVNa9ihaBciRSC7XHjliYS9VwUGOIud4PB18=
All OK. Server matched our pinned cert or public key
statusCode: 200
headers: max-age=0; pin-sha256="WoiWRyIOVNa9ihaBciRSC7XHjliYS9VwUGOIud4PB18="; pin-sha256="RRM1dGqnDFsCJXBTHky16vi1obOlCgFFn/yOhI/y+ho="; pin-sha256="k2v657xBsOVe1PQRwOsHsw3bsGT2VzIqz5K+59sNQws="; pin-sha256="K87oWBWM9UZfyddvDfoxL+8lpNyoUB2ptGtn0fv6G2Q="; pin-sha256="IQBnNBEiFuhj+8x6X8XLgh01V9Ic5/V3IRQLNFFc7v4="; pin-sha256="iie1VXtL7HzAMF+/PVPR9xzT80kQxdZeJ+zduCB3uj0="; pin-sha256="LvRiGEjRqfzurezaWuj8Wie2gyHMrW5Q06LspMnox7A="; includeSubDomains
```

[`Agent`]: #class-httpsagent

[`Session Resumption`]: tls.md#session-resumption

[`URL`]: url.md#the-whatwg-url-api

[`http.Agent(options)`]: http.md#new-agentoptions

[`http.Agent`]: http.md#class-httpagent

[`http.ClientRequest`]: http.md#class-httpclientrequest

[`http.Server`]: http.md#class-httpserver

[`http.createServer()`]: http.md#httpcreateserveroptions-requestlistener

[`http.get()`]: http.md#httpgetoptions-callback

[`http.request()`]: http.md#httprequestoptions-callback

[`https.Agent`]: #class-httpsagent

[`https.request()`]: #httpsrequestoptions-callback

[`import()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import

[`net.Server`]: net.md#class-netserver

[`new URL()`]: url.md#new-urlinput-base

[`server.close()`]: http.md#serverclosecallback

[`server.closeAllConnections()`]: http.md#servercloseallconnections

[`server.closeIdleConnections()`]: http.md#servercloseidleconnections

[`server.headersTimeout`]: http.md#serverheaderstimeout

[`server.keepAliveTimeout`]: http.md#serverkeepalivetimeout

[`server.listen()`]: net.md#serverlisten

[`server.maxHeadersCount`]: http.md#servermaxheaderscount

[`server.requestTimeout`]: http.md#serverrequesttimeout

[`server.setTimeout()`]: http.md#serversettimeoutmsecs-callback

[`server.timeout`]: http.md#servertimeout

[`tls.connect()`]: tls.md#tlsconnectoptions-callback

[`tls.createSecureContext()`]: tls.md#tlscreatesecurecontextoptions

[`tls.createServer()`]: tls.md#tlscreateserveroptions-secureconnectionlistener

[sni wiki]: https://en.wikipedia.org/wiki/Server_Name_Indication
