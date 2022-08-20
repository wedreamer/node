# TLS （SSL）

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/tls.js -->

这`node:tls`模块提供了传输层安全性的实现
（TLS）和建立在OpenSSL之上的安全套接字层（SSL）协议。
可以使用以下命令访问该模块：

```js
const tls = require('node:tls');
```

## 确定加密支持是否不可用

.js 可以在不包含对
`node:crypto`模块。在这种情况下，尝试`import`从`tls`或
叫`require('node:tls')`将导致引发错误。

使用 CommonJS 时，可以使用 try/catch 捕获引发的错误：

<!-- eslint-skip -->

```cjs
let tls;
try {
  tls = require('node:tls');
} catch (err) {
  console.log('tls support is disabled!');
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
let tls;
try {
  tls = await import('node:tls');
} catch (err) {
  console.log('tls support is disabled!');
}
```

## TLS/SSL 概念

TLS/SSL 是一组依赖于公钥基础结构 （PKI） 的协议
启用客户端和服务器之间的安全通信。对于最常见的
在案例中，每个服务器必须有一个私钥。

可以通过多种方式生成私钥。下面的示例说明了
使用 OpenSSL 命令行界面生成 2048 位 RSA 专用
钥匙：

```bash
openssl genrsa -out ryans-key.pem 2048
```

使用 TLS/SSL，所有服务器（和某些客户端）都必须具有*证书*.
证书是*公钥*对应于私钥，并且
由证书颁发机构或
私钥（此类证书称为“自签名”）。第一个
获取证书的步骤是创建一个*证书签名请求*
（CSR） 文件。

OpenSSL 命令行界面可用于为私有生成 CSR
钥匙：

```bash
openssl req -new -sha256 -key ryans-key.pem -out ryans-csr.pem
```

生成 CSR 文件后，可以将其发送到证书
用于签名或用于生成自签名证书的颁发机构。

使用 OpenSSL 命令行界面创建自签名证书
如下例所示：

```bash
openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem
```

生成证书后，它可用于生成`.pfx`或
`.p12`文件：

```bash
openssl pkcs12 -export -in ryans-cert.pem -inkey ryans-key.pem \
      -certfile ca-cert.pem -out ryans.pfx
```

哪里：

*   `in`：是签名证书
*   `inkey`：是关联的私钥
*   `certfile`：是所有证书颁发机构 （CA） 证书的串联
    单个文件，例如`cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`

### 完美的前向保密

<!-- type=misc -->

术语*[前向保密][forward secrecy]*或*完美的前向保密*描述一个功能
密钥协议（即密钥交换）方法。即服务器和客户端
键用于协商专门用于的新临时键和
仅适用于当前通信会话。实际上，这意味着即使
如果服务器的私钥被泄露，通信只能解密
通过窃听者，如果攻击者设法专门获取密钥对
为会话生成。

通过随机生成密钥对来实现完美的前向保密性
每次 TLS/SSL 握手时的密钥协议（与使用相同的密钥
所有会话）。实现此技术的方法称为“临时”。

目前常用两种方法来实现完美的前向保密（注
传统缩写后附加的字符“E”）：

*   [断续器][DHE]：Diffie-Hellman密钥协议的临时版本。
*   [断续器][ECDHE]：椭圆曲线的短暂版本 Diffie-Hellman
    密钥协议协议。

使用完美的前向保密使用`DHE`与`node:tls`模块，它是
需要生成 Diffie-Hellman 参数，并使用
`dhparam`选项[`tls.createSecureContext()`][tls.createSecureContext()].下图说明了
使用OpenSSL命令行界面生成这样的参数：

```bash
openssl dhparam -outform PEM -out dhparam.pem 2048
```

如果使用完美的前向保密使用`ECDHE`，Diffie-Hellman参数为
不需要，将使用默认的 ECDHE 曲线。这`ecdhCurve`财产
可以在创建 TLS 服务器时用于指定受支持名称的列表
要使用的曲线，请参阅[`tls.createServer()`][tls.createServer()]了解更多信息。

在 TLSv1.2 之前，完美的前向保密是可选的，但对于 TLSv1.2 来说，它不是可选的
TLSv1.3，因为所有 TLSv1.3 密码套件都使用 ECDHE。

### ALPN 和 SNI

<!-- type=misc -->

ALPN（应用层协议协商扩展）和
SNI（服务器名称指示）是 TLS 握手扩展：

*   ALPN：允许将一个 TLS 服务器用于多个协议（HTTP、HTTP/2）
*   SNI：允许对具有不同主机名的多个主机名使用一个 TLS 服务器
    证书。

### 预共享密钥

<!-- type=misc -->

TLS-PSK 支持可作为普通基于证书的替代方案提供
认证。它使用预共享密钥而不是证书来
对 TLS 连接进行身份验证，提供相互身份验证。
TLS-PSK 和公钥基础结构并不相互排斥。客户和
服务器可以同时容纳两者，在正常密码期间选择其中任何一个
谈判步骤。

TLS-PSK是一个不错的选择，因为存在安全共享的方法
密钥与每台连接机器，因此它不会替换公钥
用于大多数 TLS 用途的基础结构 （PKI）。
OpenSSL 中的 TLS-PSK 实现在
近年来，主要是因为它仅被少数应用程序使用。
在切换到 PSK 密码之前，请考虑所有替代解决方案。
生成 PSK 时，使用足够的熵作为
讨论于[RFC 4086][].从密码或其他
低熵源不安全。

默认情况下，PSK 密码处于禁用状态，因此使用 TLS-PSK 需要显式
指定密码套件`ciphers`选择。可用列表
密码可以通过以下方式检索`openssl ciphers -v 'PSK'`.所有 TLS 1.3
密码符合 PSK 的条件，但目前只有使用 SHA256 摘要的密码才符合条件
支持它们可以通过以下方式检索`openssl ciphers -v -s -tls1_3 -psk`.

根据[RFC 4279][]、长度不超过 128 字节的 PSK 标识和
必须支持长度不超过 64 个字节的 PSK。自 OpenSSL 1.1.0 起
最大标识大小为 128 字节，最大 PSK 长度为 256 字节。

当前实现不支持异步 PSK 回调，因为
底层 OpenSSL API 的局限性。

### 客户端启动的重新协商攻击缓解

<!-- type=misc -->

TLS 协议允许客户端重新协商 TLS 的某些方面
会期。不幸的是，会话重新谈判需要不成比例的金额
服务器端资源，使其成为拒绝服务的潜在媒介
攻击。

为了降低风险，重新协商限制为每十分钟三次。
一`'error'`事件在 上发出[`tls.TLSSocket`][tls.TLSSocket]实例时此
超出阈值。这些限制是可配置的：

*   `tls.CLIENT_RENEG_LIMIT`{数字}指定重新协商的次数
    请求。**违约：** `3`.
*   `tls.CLIENT_RENEG_WINDOW`{数字}指定时间重新协商窗口
    在几秒钟内。**违约：** `600`（10 分钟）。

如果没有完整的重新协商限制，则不应修改默认的重新协商限制
了解影响和风险。

TLSv1.3 不支持重新协商。

### 恢复会议

建立 TLS 会话可能相对较慢。该过程可以加快
通过保存并在以后重用会话状态来启动。有几种机制
为此，请在此处从最旧到最新（和首选）进行讨论。

#### 会话标识符

服务器为新连接生成唯一 ID，并且
将其发送给客户端。客户端和服务器保存会话状态。什么时候
重新连接，客户端发送其保存的会话状态的 ID 以及服务器
也具有该ID的状态，它可以同意使用它。否则，服务器
将创建一个新会话。看[RFC 2246][]有关更多信息，请参见第 23 页和
30\.

大多数 Web 浏览器在以下情况下支持使用会话标识符恢复
发出 HTTPS 请求。

对于 Node.js，客户端等待[`'session'`]['session']事件以获取会话数据，
并将数据提供给`session`后续选项的选项[`tls.connect()`][tls.connect()]
以重用会话。服务器必须
实现 的处理程序[`'newSession'`]['newSession']和[`'resumeSession'`]['resumeSession']事件
使用会话 ID 作为查找键来保存和恢复会话数据
重用会话。要跨负载均衡器或群集辅助角色重用会话，
服务器必须在其会话中使用共享会话缓存（如 Redis）
处理器。

#### 会议门票

服务器加密整个会话状态并将其发送
作为“票证”提供给客户。重新连接时，状态将发送到服务器
在初始连接中。此机制避免了对服务器端的需求
会话缓存。如果服务器由于任何原因不使用票证（失败）
解密它，它太旧了，等等），它将创建一个新会话并发送一个新的会话
票。看[RFC 5077][]了解更多信息。

使用会话票证恢复正成为许多 Web 的普遍支持
发出 HTTPS 请求时的浏览器。

对于 Node.js，客户端使用相同的 API 通过会话标识符进行恢复
至于恢复会议门票。对于调试，如果
[`tls.TLSSocket.getTLSTicket()`][tls.TLSSocket.getTLSTicket()]返回一个值，会话数据包含
ticket，否则它包含客户端会话状态。

使用 TLSv1.3 时，请注意服务器可能会发送多个票证，
导致多个`'session'`事件，请参阅[`'session'`]['session']了解更多
信息。

单进程服务器无需特定实现即可使用会话票证。
若要跨服务器重新启动或负载平衡器使用会话票证，服务器必须
都有相同的票证密钥。内部有三个 16 字节密钥，但
为方便起见，tls API 将它们公开为单个 48 字节缓冲区。

可以通过致电获取票证密钥[`server.getTicketKeys()`][server.getTicketKeys()]上
一个服务器实例，然后分发它们，但更合理的是
安全地生成 48 字节的安全随机数据，并将它们设置为
`ticketKeys`选项[`tls.createServer()`][tls.createServer()].密钥应定期
重新生成，服务器的密钥可以使用
[`server.setTicketKeys()`][server.setTicketKeys()].

会话票证密钥是加密密钥，它们***必须存储
安全***.使用TLS 1.2及更低版本，如果它们受到损害，则所有会话
用它们加密的二手票可以解密。不应存储它们
，并且应定期重新生成它们。

如果客户端公布对票证的支持，服务器将发送票证。这
服务器可以通过提供
`require('node:constants').SSL_OP_NO_TICKET`在`secureOptions`.

会话标识符和会话票证都超时，导致服务器
创建新会话。超时可以使用`sessionTimeout`
选项[`tls.createServer()`][tls.createServer()].

对于所有机制，当恢复失败时，服务器将创建新会话。
由于无法恢复会话不会导致 TLS/HTTPS 连接
失败，很容易不注意到不必要的不良TLS性能。这
OpenSSL CLI 可用于验证服务器是否正在恢复会话。使用
`-reconnect`选项`openssl s_client`例如：

```console
$ openssl s_client -connect localhost:443 -reconnect
```

通读调试输出。第一个连接应为“新建”，对于
例：

```text
New, TLSv1.2, Cipher is ECDHE-RSA-AES128-GCM-SHA256
```

后续连接应显示“重用”，例如：

```text
Reused, TLSv1.2, Cipher is ECDHE-RSA-AES128-GCM-SHA256
```

## 修改默认 TLS 密码套件

Node.js是使用一套默认的已启用和已禁用的 TLS 密码构建的。这
构建 Node 时，可以配置默认密码列表.js以允许
分发以提供自己的默认列表。

以下命令可用于显示默认密码套件：

```console
node -p crypto.constants.defaultCoreCipherList | tr ':' '\n'
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_GCM_SHA256
ECDHE-RSA-AES128-GCM-SHA256
ECDHE-ECDSA-AES128-GCM-SHA256
ECDHE-RSA-AES256-GCM-SHA384
ECDHE-ECDSA-AES256-GCM-SHA384
DHE-RSA-AES128-GCM-SHA256
ECDHE-RSA-AES128-SHA256
DHE-RSA-AES128-SHA256
ECDHE-RSA-AES256-SHA384
DHE-RSA-AES256-SHA384
ECDHE-RSA-AES256-SHA256
DHE-RSA-AES256-SHA256
HIGH
!aNULL
!eNULL
!EXPORT
!DES
!RC4
!MD5
!PSK
!SRP
!CAMELLIA
```

此默认值可以使用[`--tls-cipher-list`][--tls-cipher-list]
命令行开关（直接或通过[`NODE_OPTIONS`][NODE_OPTIONS]环境
变量）。例如，以下使`ECDHE-RSA-AES128-GCM-SHA256:!RC4`
默认的 TLS 密码套件：

```bash
node --tls-cipher-list='ECDHE-RSA-AES128-GCM-SHA256:!RC4' server.js

export NODE_OPTIONS=--tls-cipher-list='ECDHE-RSA-AES128-GCM-SHA256:!RC4'
node server.js
```

也可以使用每个客户端或服务器替换默认值
`ciphers`选项从[`tls.createSecureContext()`][tls.createSecureContext()]，这也可用
在[`tls.createServer()`][tls.createServer()],[`tls.connect()`][tls.connect()]，以及在创建新的
[`tls.TLSSocket`][tls.TLSSocket]s.

密码列表可以包含 TLSv1.3 密码套件名称的混合，即
从`'TLS_'`和 TLSv1.2 及以下密码的规范
套房。TLSv1.2 密码支持旧规范格式，请参阅
OpenSSL[密码列表格式][cipher list format]文档以获取详细信息，但那些
规格做*不*适用于 TLSv1.3 密码。TLSv1.3 套房只能
通过在密码列表中包含其全名来启用。他们不能，因为
例如，使用旧版 TLSv1.2 启用或禁用`'EECDH'`或
`'!EECDH'`规范。

尽管 TLSv1.3 和 TLSv1.2 密码套件的相对顺序，但 TLSv1.3
协议比 TLSv1.2 安全得多，并且将始终选择
如果握手指示它受支持，并且如果有任何 TLSv1.3，则通过 TLSv1.2
密码套件已启用。

Node.js中包含的默认密码套件已经过仔细处理
选择以反映当前的安全最佳做法和风险缓解。
更改默认密码套件可能会对安全性产生重大影响
的应用程序。这`--tls-cipher-list`开关和`ciphers`选项应由
仅在绝对必要时使用。

默认密码套件首选 GCM 密码[铬的“现代”
密码学设置][Chrome's 'modern
cryptography' setting]并且还更喜欢ECDHE和DHE密码，以实现完美的
前向保密，同时提供*一些*向后兼容性。

依赖于不安全且已弃用的 RC4 或基于 DES 的密码的旧客户端
（像Internet Explorer 6一样）无法完成握手过程
默认配置。如果这些客户端*必须*得到支持，
[红绿灯系统建议][TLS recommendations]可能提供兼容的密码套件。更多详情
在格式上，请参阅 OpenSSL[密码列表格式][cipher list format]文档。

只有五个 TLSv1.3 密码套件：

*   `'TLS_AES_256_GCM_SHA384'`
*   `'TLS_CHACHA20_POLY1305_SHA256'`
*   `'TLS_AES_128_GCM_SHA256'`
*   `'TLS_AES_128_CCM_SHA256'`
*   `'TLS_AES_128_CCM_8_SHA256'`

默认情况下，前三个处于启用状态。二`CCM`支持基于套件的套件
由TLSv1.3提供，因为它们在受约束的系统上可能性能更高，但它们
默认情况下不启用，因为它们提供的安全性较低。

## X509 证书错误代码

多个函数可能会由于证书错误而失败，这些错误由
OpenSSL.在这种情况下，该函数通过其回调提供 {Error}
有属性`code`它可以采用以下值之一：

<!--
values are taken from src/crypto/crypto_common.cc
description are taken from deps/openssl/openssl/crypto/x509/x509_txt.c
-->

*   `'UNABLE_TO_GET_ISSUER_CERT'`：无法获取颁发者证书。
*   `'UNABLE_TO_GET_CRL'`：无法获取证书 CRL。
*   `'UNABLE_TO_DECRYPT_CERT_SIGNATURE'`：无法解密证书的
    签名。
*   `'UNABLE_TO_DECRYPT_CRL_SIGNATURE'`：无法解密 CRL 的签名。
*   `'UNABLE_TO_DECODE_ISSUER_PUBLIC_KEY'`：无法解码颁发者公钥。
*   `'CERT_SIGNATURE_FAILURE'`：证书签名失败。
*   `'CRL_SIGNATURE_FAILURE'`：CRL 签名失败。
*   `'CERT_NOT_YET_VALID'`：证书尚无效。
*   `'CERT_HAS_EXPIRED'`：证书已过期。
*   `'CRL_NOT_YET_VALID'`：CRL 尚无效。
*   `'CRL_HAS_EXPIRED'`：CRL 已过期。
*   `'ERROR_IN_CERT_NOT_BEFORE_FIELD'`：证书中的格式错误不是在之前
    田。
*   `'ERROR_IN_CERT_NOT_AFTER_FIELD'`：证书中的格式错误 notAfter
    田。
*   `'ERROR_IN_CRL_LAST_UPDATE_FIELD'`：CRL 的 lastUpdate 字段中的格式错误。
*   `'ERROR_IN_CRL_NEXT_UPDATE_FIELD'`：CRL 的 nextUpdate 字段中的格式错误。
*   `'OUT_OF_MEM'`：内存不足。
*   `'DEPTH_ZERO_SELF_SIGNED_CERT'`：自签名证书。
*   `'SELF_SIGNED_CERT_IN_CHAIN'`：证书链中的自签名证书。
*   `'UNABLE_TO_GET_ISSUER_CERT_LOCALLY'`：无法获取本地颁发者证书。
*   `'UNABLE_TO_VERIFY_LEAF_SIGNATURE'`：无法验证第一个证书。
*   `'CERT_CHAIN_TOO_LONG'`：证书链太长。
*   `'CERT_REVOKED'`：证书已吊销。
*   `'INVALID_CA'`：CA 证书无效。
*   `'PATH_LENGTH_EXCEEDED'`：超出路径长度约束。
*   `'INVALID_PURPOSE'`：不支持的证书用途。
*   `'CERT_UNTRUSTED'`：证书不受信任。
*   `'CERT_REJECTED'`：证书被拒绝。
*   `'HOSTNAME_MISMATCH'`：主机名不匹配。

## 类：`tls.CryptoStream`

<!-- YAML
added: v0.3.4
deprecated: v0.11.3
-->

> 稳定性：0 - 已弃用：使用[`tls.TLSSocket`][tls.TLSSocket]相反。

这`tls.CryptoStream`类表示加密数据流。此类
已弃用，不应再使用。

### `cryptoStream.bytesWritten`

<!-- YAML
added: v0.3.4
deprecated: v0.11.3
-->

这`cryptoStream.bytesWritten`属性返回总字节数
写入底层套接字*包括*所需的字节数
TLS 协议的实现。

## 类：`tls.SecurePair`

<!-- YAML
added: v0.3.2
deprecated: v0.11.3
-->

> 稳定性：0 - 已弃用：使用[`tls.TLSSocket`][tls.TLSSocket]相反。

返回者[`tls.createSecurePair()`][tls.createSecurePair()].

### 事件：`'secure'`

<!-- YAML
added: v0.3.2
deprecated: v0.11.3
-->

这`'secure'`事件由`SecurePair`对象曾经是安全的
已建立连接。

与检查服务器一样
[`'secureConnection'`]['secureConnection']
事件`pair.cleartext.authorized`应进行检查以确认
使用的证书已正确授权。

## 类：`tls.Server`

<!-- YAML
added: v0.3.2
-->

*   扩展：{net.服务器}

接受使用 TLS 或 SSL 的加密连接。

### 事件：`'connection'`

<!-- YAML
added: v0.3.2
-->

*   `socket`{流。双工}

当在 TLS 之前建立新的 TCP 流时，将发出此事件
开始握手。`socket`通常是类型的对象[`net.Socket`][net.Socket]但
不会接收与从[`net.Server`][net.Server]
`'connection'`事件。通常，用户不希望访问此事件。

用户还可以显式发出此事件以注入连接
进入 TLS 服务器。在这种情况下，任何[`Duplex`][Duplex]可以传递流。

### 事件：`'keylog'`

<!-- YAML
added:
 - v12.3.0
 - v10.20.0
-->

*   `line`{缓冲区}ASCII 文本行，在 NSS 中`SSLKEYLOGFILE`格式。
*   `tlsSocket`{tls.TLSSocket} The`tls.TLSSocket`它所在的实例
    生成。

这`keylog`事件在生成或接收到以下人员生成或接收时发出
与此服务器的连接（通常在握手完成之前，但不是
必然）。可以存储此键控材料以进行调试，因为它允许
捕获的要解密的 TLS 流量。它可以被多次发射
每个套接字。

一个典型的用例是将接收的行附加到一个通用的文本文件中，该文件
后来被软件（如Wireshark）用于解密流量：

```js
const logFile = fs.createWriteStream('/tmp/ssl-keys.log', { flags: 'a' });
// ...
server.on('keylog', (line, tlsSocket) => {
  if (tlsSocket.remoteAddress !== '...')
    return; // Only log keys for a particular IP
  logFile.write(line);
});
```

### 事件：`'newSession'`

<!-- YAML
added: v0.9.2
changes:
  - version: v0.11.12
    pr-url: https://github.com/nodejs/node-v0.x-archive/pull/7118
    description: The `callback` argument is now supported.
-->

这`'newSession'`事件在创建新的 TLS 会话时发出。这可能
用于将会话存储在外部存储中。应将数据提供给
这[`'resumeSession'`]['resumeSession']回调。

侦听器回调在被调用时传递三个参数：

*   `sessionId`{缓冲区}TLS 会话标识符
*   `sessionData`{缓冲区}TLS 会话数据
*   `callback`{函数}一个回调函数，不采用必须
    调用以通过安全连接发送或接收数据。

侦听此事件只会对已建立的连接产生影响
在添加事件侦听器之后。

### 事件：`'OCSPRequest'`

<!-- YAML
added: v0.11.13
-->

这`'OCSPRequest'`事件在客户端发送证书状态时发出
请求。侦听器回调在被调用时传递三个参数：

*   `certificate`{缓冲区}服务器证书
*   `issuer`{缓冲区}发行人的证书
*   `callback`{函数}必须调用以提供的回调函数
    OCSP 请求的结果。

可以解析服务器的当前证书以获取 OCSP URL
和证书 ID;获得 OCSP 响应后，`callback(null, resp)`是
然后调用，其中`resp`是一个`Buffer`包含 OCSP 响应的实例。
双`certificate`和`issuer`是`Buffer`DER 表示的
主证书和颁发者的证书。这些可用于获取 OCSP
证书 ID 和 OCSP 终结点 URL。

或者`callback(null, null)`可能被调用，表示有
没有 OCSP 响应。

叫`callback(err)`将导致`socket.destroy(err)`叫。

OCSP 请求的典型流程如下：

1.  客户端连接到服务器并发送`'OCSPRequest'`（通过状态
    ClientHello 中的信息扩展）。
2.  服务器接收请求并发出`'OCSPRequest'`事件，调用
    侦听器（如果已注册）。
3.  服务器从`certificate`或`issuer`和
    执行[OCSP 请求][OCSP request]到 CA。
4.  服务器接收`'OCSPResponse'`从 CA 发送回客户端
    通过`callback`论点
5.  客户端验证响应并销毁套接字或执行
    握手。

这`issuer`可以是`null`如果证书是自签名的，或者
颁发者不在根证书列表中。（可能会提供发行人
通过`ca`建立 TLS 连接时的选项。

侦听此事件只会对已建立的连接产生影响
在添加事件侦听器之后。

一个 npm 模块，如[asn1.js][]可用于解析证书。

### 事件：`'resumeSession'`

<!-- YAML
added: v0.9.2
-->

这`'resumeSession'`事件在客户端请求恢复
以前的 TLS 会话。侦听器回调在以下情况下传递两个参数：
叫：

*   `sessionId`{缓冲区}TLS 会话标识符
*   `callback`{函数}在上一个会话时要调用的回调函数
    已恢复：`callback([err[, sessionData]])`
    *   `err`{错误}
    *   `sessionData`{缓冲区}

事件侦听器应在外部存储中执行查找
`sessionData`由 保存[`'newSession'`]['newSession']使用给定事件处理程序
`sessionId`.如果找到，请致电`callback(null, sessionData)`以恢复会话。
如果未找到，则无法恢复会话。`callback()`必须调用
没有`sessionData`以便握手可以继续，新会话可以
被创建。可以调用`callback(err)`终止传入
连接并销毁套接字。

侦听此事件只会对已建立的连接产生影响
在添加事件侦听器之后。

下面说明了如何恢复 TLS 会话：

```js
const tlsSessionStore = {};
server.on('newSession', (id, data, cb) => {
  tlsSessionStore[id.toString('hex')] = data;
  cb();
});
server.on('resumeSession', (id, cb) => {
  cb(null, tlsSessionStore[id.toString('hex')] || null);
});
```

### 事件：`'secureConnection'`

<!-- YAML
added: v0.3.2
-->

这`'secureConnection'`事件在 的握手过程之后发出
新连接已成功完成。侦听器回调传递一个
调用时的单个参数：

*   `tlsSocket`{tls.TLSSocket} 已建立的 TLS 套接字。

这`tlsSocket.authorized`属性是一个`boolean`指示
客户端已由提供的证书颁发机构之一验证
服务器。如果`tlsSocket.authorized`是`false`然后`socket.authorizationError`
设置为描述授权如何失败。取决于设置
的 TLS 服务器中，仍可能接受未经授权的连接。

这`tlsSocket.alpnProtocol`属性是一个字符串，其中包含选定的
ALPN 协议。当 ALPN 没有选择的实验方案时，`tlsSocket.alpnProtocol`
等于`false`.

这`tlsSocket.servername`属性是包含服务器名称的字符串
通过 SNI 请求。

### 事件：`'tlsClientError'`

<!-- YAML
added: v6.0.0
-->

这`'tlsClientError'`事件在安全之前发生错误时发出
连接已建立。侦听器回调在以下情况下传递两个参数：
叫：

*   `exception`{错误}这`Error`描述错误的对象
*   `tlsSocket`{tls.TLSSocket} The`tls.TLSSocket`实例，其中
    错误源自。

### `server.addContext(hostname, context)`

<!-- YAML
added: v0.5.3
-->

*   `hostname`{字符串}SNI 主机名或通配符（例如`'*'`)
*   `context`{对象}包含任何可能属性的对象
    从[`tls.createSecureContext()`][tls.createSecureContext()] `options`参数（例如`key`,
    `cert`,`ca`等）。

这`server.addContext()`方法添加一个安全上下文，该上下文将在以下情况下使用：
客户端请求的 SNI 名称与提供的`hostname`（或通配符）。

当有多个匹配的上下文时，最近添加的上下文是
使用。

### `server.address()`

<!-- YAML
added: v0.6.0
-->

*   返回： {对象}

返回 绑定地址、地址系列名称和端口
操作系统报告的服务器。看[`net.Server.address()`][net.Server.address()]为
更多信息。

### `server.close([callback])`

<!-- YAML
added: v0.3.2
-->

*   `callback`{函数}将注册以侦听的侦听器回调
    对于服务器实例的`'close'`事件。
*   返回：{tls。服务器}

这`server.close()`方法阻止服务器接受新连接。

此函数以异步方式运行。这`'close'`将发出事件
当服务器不再有打开的连接时。

### `server.getTicketKeys()`

<!-- YAML
added: v3.0.0
-->

*   返回：{Buffer} 包含会话票证密钥的 48 字节缓冲区。

返回会话票证密钥。

看[续会][Session Resumption]了解更多信息。

### `server.listen()`

启动侦听加密连接的服务器。
此方法与[`server.listen()`][server.listen()]从[`net.Server`][net.Server].

### `server.setSecureContext(options)`

<!-- YAML
added: v11.0.0
-->

*   `options`{对象}包含以下位置中任何可能属性的对象
    这[`tls.createSecureContext()`][tls.createSecureContext()] `options`参数（例如`key`,`cert`,
    `ca`等）。

这`server.setSecureContext()`方法替换
现有服务器。与服务器的现有连接不会中断。

### `server.setTicketKeys(keys)`

<!-- YAML
added: v3.0.0
-->

*   `keys`{缓冲区|TypedArray|DataView} 包含会话的 48 字节缓冲区
    票证密钥。

设置会话票证密钥。

对票证密钥的更改仅对将来的服务器连接有效。
现有或当前挂起的服务器连接将使用前面的密钥。

看[续会][Session Resumption]了解更多信息。

## 类：`tls.TLSSocket`

<!-- YAML
added: v0.11.4
-->

*   扩展：{net.插座}

对写入数据和所有必需的 TLS 执行透明加密
谈判。

的实例`tls.TLSSocket`实现双工[流][Stream]接口。

返回 TLS 连接元数据的方法（例如
[`tls.TLSSocket.getPeerCertificate()`][tls.TLSSocket.getPeerCertificate()]将只返回数据，而
连接已打开。

### `new tls.TLSSocket(socket[, options])`

<!-- YAML
added: v0.11.4
changes:
  - version: v12.2.0
    pr-url: https://github.com/nodejs/node/pull/27497
    description: The `enableTrace` option is now supported.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2564
    description: ALPN options are supported now.
-->

*   `socket`{网.套接字|流。双工}
    在服务器端，任何`Duplex`流。在客户端，任何
    的实例[`net.Socket`][net.Socket]（对于通用`Duplex`流支持
    在客户端，[`tls.connect()`][tls.connect()]必须使用）。
*   `options`{对象}
    *   `enableTrace`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `isServer`：SSL/TLS 协议是不对称的，TLSSockets 必须知道
        它们将充当服务器或客户端。如果`true`TLS 套接字将是
        实例化为服务器。**违约：** `false`.
    *   `server`{网.服务器} A[`net.Server`][net.Server]实例。
    *   `requestCert`：是否通过请求
        证书。客户端始终请求服务器证书。服务器
        (`isServer`为真）可设置`requestCert`更改为 true 以请求客户端
        证书。
    *   `rejectUnauthorized`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `ALPNProtocols`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `SNICallback`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `session`{缓冲区}一个`Buffer`包含 TLS 会话的实例。
    *   `requestOCSP`{布尔值}如果`true`，指定 OCSP 状态请求
        扩展将被添加到客户端问候和`'OCSPResponse'`事件
        将在建立安全通信之前在套接字上发出
    *   `secureContext`：使用
        [`tls.createSecureContext()`][tls.createSecureContext()].如果`secureContext`是*不*提供，一个
        将通过传递整个`options`对象
        `tls.createSecureContext()`.
    *   ...:[`tls.createSecureContext()`][tls.createSecureContext()]在以下情况下使用的选项：
        `secureContext`选项丢失。否则，它们将被忽略。

构建一个新的`tls.TLSSocket`来自现有 TCP 套接字的对象。

### 事件：`'keylog'`

<!-- YAML
added:
 - v12.3.0
 - v10.20.0
-->

*   `line`{缓冲区}ASCII 文本行，在 NSS 中`SSLKEYLOGFILE`格式。

这`keylog`事件在`tls.TLSSocket`当关键材料
由套接字生成或接收。此键控材料可以存储
用于调试，因为它允许对捕获的 TLS 流量进行解密。它可能
在握手完成之前或之后多次发出。

一个典型的用例是将接收的行附加到一个通用的文本文件中，该文件
后来被软件（如Wireshark）用于解密流量：

```js
const logFile = fs.createWriteStream('/tmp/ssl-keys.log', { flags: 'a' });
// ...
tlsSocket.on('keylog', (line) => logFile.write(line));
```

### 事件：`'OCSPResponse'`

<!-- YAML
added: v0.11.13
-->

这`'OCSPResponse'`如果`requestOCSP`选项已设置
当`tls.TLSSocket`已创建并已收到 OCSP 响应。
调用侦听器回调时传递单个参数：

*   `response`{缓冲区}服务器的 OCSP 响应

通常，`response`是来自服务器 CA 的数字签名对象，该对象
包含有关服务器的证书吊销状态的信息。

### 事件：`'secureConnect'`

<!-- YAML
added: v0.11.4
-->

这`'secureConnect'`事件在新的握手过程之后发出
连接已成功完成。将调用侦听器回调
无论服务器的证书是否已获得授权。它
是客户的责任检查`tlsSocket.authorized`属性到
确定服务器证书是否由指定的 CA 之一签名。如果
`tlsSocket.authorized === false`，然后可以通过检查
`tlsSocket.authorizationError`财产。如果使用 ALPN，则
`tlsSocket.alpnProtocol`可以检查属性以确定协商
协议。

这`'secureConnect'`当 {tls.TLSSocket} 已创建
使用`new tls.TLSSocket()`构造 函数。

### 事件：`'session'`

<!-- YAML
added: v11.10.0
-->

*   `session`{缓冲区}

这`'session'`事件在客户端上发出`tls.TLSSocket`当新会话
或 TLS 票证可用。这可能是也可能不是在握手之前
完成，具体取决于协商的 TLS 协议版本。活动
未在服务器上发出，或者如果未创建新会话，例如，
当连接恢复时。对于某些 TLS 协议版本，事件可能是
发出多次，在这种情况下，所有会话都可用于
恢复。

在客户端上，`session`可以提供给`session`选项
[`tls.connect()`][tls.connect()]以恢复连接。

看[续会][Session Resumption]了解更多信息。

对于 TLSv1.2 及更低版本，[`tls.TLSSocket.getSession()`][tls.TLSSocket.getSession()]可调用一次
握手完成。对于 TLSv1.3，仅允许基于票证的恢复
通过协议，发送多个票证，并且直到
握手完成后。因此，有必要等待
`'session'`事件以获取可恢复的会话。应用
应使用`'session'`事件而不是`getSession()`以确保
它们将适用于所有 TLS 版本。仅期望的应用程序
get 或使用一个会话应仅侦听一次此事件：

```js
tlsSocket.once('session', (session) => {
  // The session can be used immediately or later.
  tls.connect({
    session: session,
    // Other connect options...
  });
});
```

### `tlsSocket.address()`

<!-- YAML
added: v0.11.4
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43054
    description: The `family` property now returns a string instead of a number.
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41431
    description: The `family` property now returns a number instead of a string.
-->

*   返回： {对象}

返回绑定`address`，地址`family`名称，以及`port`的
操作系统报告的基础套接字：
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`.

### `tlsSocket.authorizationError`

<!-- YAML
added: v0.11.4
-->

返回未验证对等方证书的原因。这
仅当以下情况时才设置属性`tlsSocket.authorized === false`.

### `tlsSocket.authorized`

<!-- YAML
added: v0.11.4
-->

*   {布尔值}

此属性是`true`如果对等证书由其中一个 CA 签名
在创建`tls.TLSSocket`实例，否则`false`.

### `tlsSocket.disableRenegotiation()`

<!-- YAML
added: v8.4.0
-->

为此禁用 TLS 重新协商`TLSSocket`实例。调用后，尝试
重新谈判将触发`'error'`事件`TLSSocket`.

### `tlsSocket.enableTrace()`

<!-- YAML
added: v12.2.0
-->

启用后，TLS 数据包跟踪信息将写入`stderr`.这可以是
用于调试 TLS 连接问题。

输出的格式与
`openssl s_client -trace`或`openssl s_server -trace`.虽然它是由
OpenSSL's`SSL_trace()`功能，格式未记录，可以更改
恕不另行通知，也不应依赖。

### `tlsSocket.encrypted`

<!-- YAML
added: v0.11.4
-->

总是返回`true`.这可用于区分 TLS 套接字和常规套接字
`net.Socket`实例。

### `tlsSocket.exportKeyingMaterial(length, label[, context])`

<!-- YAML
added:
 - v13.10.0
 - v12.17.0
-->

*   `length`{number} 要从键控材料中检索的字节数

*   `label`{string} 特定于应用程序的标签，通常这将是一个
    值从
    [伊安娜出口商标签登记处](https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#exporter-labels).

*   `context`{缓冲区}（可选）提供上下文。

*   返回：{Buffer} 键控材料的请求字节

键控材料用于验证，以防止不同类型的攻击
网络协议，例如IEEE 802.1X的规范。

例

```js
const keyingMaterial = tlsSocket.exportKeyingMaterial(
  128,
  'client finished');

/*
 Example return value of keyingMaterial:
 <Buffer 76 26 af 99 c5 56 8e 42 09 91 ef 9f 93 cb ad 6c 7b 65 f8 53 f1 d8 d9
    12 5a 33 b8 b5 25 df 7b 37 9f e0 e2 4f b8 67 83 a3 2f cd 5d 41 42 4c 91
    74 ef 2c ... 78 more bytes>
*/
```

查看 OpenSSL[`SSL_export_keying_material`][SSL_export_keying_material]文档以获取更多信息
信息。

### `tlsSocket.getCertificate()`

<!-- YAML
added: v11.2.0
-->

*   返回： {对象}

返回表示本地证书的对象。返回的对象具有
与证书字段对应的某些属性。

看[`tls.TLSSocket.getPeerCertificate()`][tls.TLSSocket.getPeerCertificate()]有关证书的示例
结构。

如果没有本地证书，则将返回一个空对象。如果
套接字已被破坏，`null`将被退回。

### `tlsSocket.getCipher()`

<!-- YAML
added: v0.11.4
changes:
  - version:
     - v13.4.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30637
    description: Return the IETF cipher name as `standardName`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26625
    description: Return the minimum cipher version, instead of a fixed string
      (`'TLSv1/SSLv3'`).
-->

*   返回： {对象}
    *   `name`{字符串}密码套件的 OpenSSL 名称。
    *   `standardName`{字符串}密码套件的 IETF 名称。
    *   `version`{字符串}此密码支持的最低 TLS 协议版本
        套房。有关实际协商的协议，请参阅[`tls.TLSSocket.getProtocol()`][tls.TLSSocket.getProtocol()].

返回一个对象，其中包含有关协商的密码套件的信息。

例如，具有 AES256-SHA 密码的 TLSv1.2 协议：

```json
{
    "name": "AES256-SHA",
    "standardName": "TLS_RSA_WITH_AES_256_CBC_SHA",
    "version": "SSLv3"
}
```

看
[SSL_CIPHER_get_name](https://www.openssl.org/docs/man1.1.1/man3/SSL_CIPHER_get_name.html)
了解更多信息。

### `tlsSocket.getEphemeralKeyInfo()`

<!-- YAML
added: v5.0.0
-->

*   返回： {对象}

返回一个对象，该对象表示 参数的类型、名称和大小
中短暂的密钥交换[完美的前向保密][perfect forward secrecy]在客户端上
连接。当密钥交换不是
短暂的。因为这仅在客户端套接字上受支持;`null`返回
如果在服务器套接字上调用。支持的类型包括`'DH'`和`'ECDH'`.这
`name`仅当类型为`'ECDH'`.

例如：`{ type: 'ECDH', name: 'prime256v1', size: 256 }`.

### `tlsSocket.getFinished()`

<!-- YAML
added: v9.9.0
-->

*   返回： {缓冲区|未定义} 最新`Finished`已
    作为 SSL/TLS 握手的一部分发送到套接字，或者`undefined`如果
    不`Finished`消息已发送。

作为`Finished`消息是完整握手的消息摘要
（TLS 1.0 总共有 192 位，SSL 3.0 有更多位），它们可以
用于外部身份验证过程时进行身份验证
SSL/TLS 提供的是不可取的，或者是不够的。

对应于`SSL_get_finished`OpenSSL 中的例程，可以使用
实现`tls-unique`通道绑定自[RFC 5929][].

### `tlsSocket.getPeerCertificate([detailed])`

<!-- YAML
added: v0.11.4
-->

*   `detailed`{布尔值}在以下情况下包括完整的证书链：`true`否则
    仅包括对等方的证书。
*   返回：{对象} 证书对象。

返回表示对等方证书的对象。如果对等方没有
提供证书，将返回空对象。如果套接字已
摧毁`null`将被退回。

如果请求了完整的证书链，则每个证书将包括
`issuerCertificate`属性，其中包含表示其颁发者的对象
证书。

#### 证书对象

<!-- YAML
changes:
  - version:
      - v17.2.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/39809
    description: Add fingerprint512.
  - version: v11.4.0
    pr-url: https://github.com/nodejs/node/pull/24358
    description: Support Elliptic Curve public key info.
-->

证书对象具有与
证书。

*   `raw`{缓冲区}DER 编码的 X.509 证书数据。
*   `subject`{对象}证书主题，按以下方式描述
    国家 （`C`）， StateOrProvince （`ST`），位置（`L`）、组织 （`O`),
    组织单位 （`OU`）和公用名 （`CN`).公用名通常为
    具有 TLS 证书的 DNS 名称。例：
    `{C: 'UK', ST: 'BC', L: 'Metro', O: 'Node Fans', OU: 'Docs', CN: 'example.com'}`.
*   `issuer`{对象}证书颁发者，其描述术语与
    `subject`.
*   `valid_from`{字符串}证书的有效起始日期。
*   `valid_to`{字符串}证书的有效日期。
*   `serialNumber`{字符串}证书序列号，以十六进制字符串的形式出现。
    例：`'B9B0D332A1AA5635'`.
*   `fingerprint`{字符串}DER 编码证书的 SHA-1 摘要。是的
    作为 返回`:`分隔的十六进制字符串。例：`'2A:7A:C2:DD:...'`.
*   `fingerprint256`{字符串}DER 编码证书的 SHA-256 摘要。
    它作为`:`分隔的十六进制字符串。例：
    `'2A:7A:C2:DD:...'`.
*   `fingerprint512`{字符串}DER 编码证书的 SHA-512 摘要。
    它作为`:`分隔的十六进制字符串。例：
    `'2A:7A:C2:DD:...'`.
*   `ext_key_usage`{数组}（可选）扩展密钥用法，一组 OID。
*   `subjectaltname`{字符串}（可选）包含串联名称的字符串
    对于主题，替代`subject`名字。
*   `infoAccess`{数组}（可选）描述 AuthorityInfoAccesss 的数组，
    与 OCSP 一起使用。
*   `issuerCertificate`{对象}（可选）颁发者证书对象。为
    自签名证书，这可能是一个循环引用。

证书可能包含有关公钥的信息，具体取决于
密钥类型。

对于 RSA 密钥，可以定义以下属性：

*   `bits`{数字}RSA 位大小。例：`1024`.
*   `exponent`{字符串}RSA 指数，以十六进制数的形式表示
    表示法。例：`'0x010001'`.
*   `modulus`{字符串}RSA 模量，作为十六进制字符串。例：
    `'B56CE45CB7...'`.
*   `pubkey`{缓冲区}公钥。

对于 EC 密钥，可以定义以下属性：

*   `pubkey`{缓冲区}公钥。
*   `bits`{数字}密钥大小（以位为单位）。例：`256`.
*   `asn1Curve`{字符串}（可选）椭圆的 OID 的 ASN.1 名称
    曲线。已知曲线由 OID 标识。虽然这是不寻常的，但它是
    该曲线可能通过其数学性质来识别，其中
    如果它不会有 OID。例：`'prime256v1'`.
*   `nistCurve`{字符串}（可选）椭圆曲线的 NIST 名称，如果
    有一个（并非所有已知曲线都被NIST指定了名称）。例：
    `'P-256'`.

示例证书：

<!-- eslint-skip -->

```js
{ subject:
   { OU: [ 'Domain Control Validated', 'PositiveSSL Wildcard' ],
     CN: '*.nodejs.org' },
  issuer:
   { C: 'GB',
     ST: 'Greater Manchester',
     L: 'Salford',
     O: 'COMODO CA Limited',
     CN: 'COMODO RSA Domain Validation Secure Server CA' },
  subjectaltname: 'DNS:*.nodejs.org, DNS:nodejs.org',
  infoAccess:
   { 'CA Issuers - URI':
      [ 'http://crt.comodoca.com/COMODORSADomainValidationSecureServerCA.crt' ],
     'OCSP - URI': [ 'http://ocsp.comodoca.com' ] },
  modulus: 'B56CE45CB740B09A13F64AC543B712FF9EE8E4C284B542A1708A27E82A8D151CA178153E12E6DDA15BF70FFD96CB8A88618641BDFCCA03527E665B70D779C8A349A6F88FD4EF6557180BD4C98192872BCFE3AF56E863C09DDD8BC1EC58DF9D94F914F0369102B2870BECFA1348A0838C9C49BD1C20124B442477572347047506B1FCD658A80D0C44BCC16BC5C5496CFE6E4A8428EF654CD3D8972BF6E5BFAD59C93006830B5EB1056BBB38B53D1464FA6E02BFDF2FF66CD949486F0775EC43034EC2602AEFBF1703AD221DAA2A88353C3B6A688EFE8387811F645CEED7B3FE46E1F8B9F59FAD028F349B9BC14211D5830994D055EEA3D547911E07A0ADDEB8A82B9188E58720D95CD478EEC9AF1F17BE8141BE80906F1A339445A7EB5B285F68039B0F294598A7D1C0005FC22B5271B0752F58CCDEF8C8FD856FB7AE21C80B8A2CE983AE94046E53EDE4CB89F42502D31B5360771C01C80155918637490550E3F555E2EE75CC8C636DDE3633CFEDD62E91BF0F7688273694EEEBA20C2FC9F14A2A435517BC1D7373922463409AB603295CEB0BB53787A334C9CA3CA8B30005C5A62FC0715083462E00719A8FA3ED0A9828C3871360A73F8B04A4FC1E71302844E9BB9940B77E745C9D91F226D71AFCAD4B113AAF68D92B24DDB4A2136B55A1CD1ADF39605B63CB639038ED0F4C987689866743A68769CC55847E4A06D6E2E3F1',
  exponent: '0x10001',
  pubkey: <Buffer ... >,
  valid_from: 'Aug 14 00:00:00 2017 GMT',
  valid_to: 'Nov 20 23:59:59 2019 GMT',
  fingerprint: '01:02:59:D9:C3:D2:0D:08:F7:82:4E:44:A4:B4:53:C5:E2:3A:87:4D',
  fingerprint256: '69:AE:1A:6A:D4:3D:C6:C1:1B:EA:C6:23:DE:BA:2A:14:62:62:93:5C:7A:EA:06:41:9B:0B:BC:87:CE:48:4E:02',
  fingerprint512: '19:2B:3E:C3:B3:5B:32:E8:AE:BB:78:97:27:E4:BA:6C:39:C9:92:79:4F:31:46:39:E2:70:E5:5F:89:42:17:C9:E8:64:CA:FF:BB:72:56:73:6E:28:8A:92:7E:A3:2A:15:8B:C2:E0:45:CA:C3:BC:EA:40:52:EC:CA:A2:68:CB:32',
  ext_key_usage: [ '1.3.6.1.5.5.7.3.1', '1.3.6.1.5.5.7.3.2' ],
  serialNumber: '66593D57F20CBC573E433381B5FEC280',
  raw: <Buffer ... > }
```

### `tlsSocket.getPeerFinished()`

<!-- YAML
added: v9.9.0
-->

*   返回： {缓冲区|未定义} 最新`Finished`预期的消息
    或者实际上已作为 SSL/TLS 握手的一部分从套接字接收，
    或`undefined`如果没有`Finished`消息到目前为止。

作为`Finished`消息是完整握手的消息摘要
（TLS 1.0 总共有 192 位，SSL 3.0 有更多位），它们可以
用于外部身份验证过程时进行身份验证
SSL/TLS 提供的是不可取的，或者是不够的。

对应于`SSL_get_peer_finished`OpenSSL 中的例程，可以使用
实现`tls-unique`通道绑定自[RFC 5929][].

### `tlsSocket.getPeerX509Certificate()`

<!-- YAML
added: v15.9.0
-->

*   返回： {X509Certificate}

将对等证书作为 {X509Certificate} 对象返回。

如果没有对等证书，或者套接字已被销毁，
`undefined`将被退回。

### `tlsSocket.getProtocol()`

<!-- YAML
added: v5.7.0
-->

*   返回：{字符串|null}

返回一个字符串，其中包含 协商的 SSL/TLS 协议版本的
当前连接。价值`'unknown'`将返回连接
尚未完成握手过程的套接字。价值`null`将
对于服务器套接字或断开连接的客户端套接字，将返回。

协议版本包括：

*   `'SSLv3'`
*   `'TLSv1'`
*   `'TLSv1.1'`
*   `'TLSv1.2'`
*   `'TLSv1.3'`

查看 OpenSSL[`SSL_get_version`][SSL_get_version]文档以获取更多信息。

### `tlsSocket.getSession()`

<!-- YAML
added: v0.11.4
-->

*   {缓冲区}

返回 TLS 会话数据或`undefined`如果没有会话
谈判。在客户端上，可以将数据提供给`session`选项
[`tls.connect()`][tls.connect()]以恢复连接。在服务器上，它可能很有用
用于调试。

看[续会][Session Resumption]了解更多信息。

注意：`getSession()`仅适用于 TLSv1.2 及更低版本。对于 TLSv1.3，应用程序
必须使用[`'session'`]['session']事件（它也适用于 TLSv1.2 及更低版本）。

### `tlsSocket.getSharedSigalgs()`

<!-- YAML
added: v12.11.0
-->

*   返回：{数组} 服务器和之间共享的签名算法列表
    客户端按优先级递减的顺序排列。

看
[SSL_get_shared_sigalgs](https://www.openssl.org/docs/man1.1.1/man3/SSL_get_shared_sigalgs.html)
了解更多信息。

### `tlsSocket.getTLSTicket()`

<!-- YAML
added: v0.11.4
-->

*   {缓冲区}

对于客户端，返回 TLS 会话票证（如果有），或者
`undefined`.对于服务器，始终返回`undefined`.

它可能对调试有用。

看[续会][Session Resumption]了解更多信息。

### `tlsSocket.getX509Certificate()`

<!-- YAML
added: v15.9.0
-->

*   返回： {X509Certificate}

将本地证书作为 {X509Certificate} 对象返回。

如果没有本地证书，或者套接字已被销毁，
`undefined`将被退回。

### `tlsSocket.isSessionReused()`

<!-- YAML
added: v0.5.6
-->

*   返回：{布尔值}`true`如果会话被重用，`false`否则。

看[续会][Session Resumption]了解更多信息。

### `tlsSocket.localAddress`

<!-- YAML
added: v0.11.4
-->

*   {字符串}

返回本地 IP 地址的字符串表示形式。

### `tlsSocket.localPort`

<!-- YAML
added: v0.11.4
-->

*   {整数}

返回本地端口的数字表示形式。

### `tlsSocket.remoteAddress`

<!-- YAML
added: v0.11.4
-->

*   {字符串}

返回远程 IP 地址的字符串表示形式。例如
`'74.125.127.100'`或`'2001:4860:a005::68'`.

### `tlsSocket.remoteFamily`

<!-- YAML
added: v0.11.4
-->

*   {字符串}

返回远程 IP 系列的字符串表示形式。`'IPv4'`或`'IPv6'`.

### `tlsSocket.remotePort`

<!-- YAML
added: v0.11.4
-->

*   {整数}

返回远程端口的数字表示形式。例如`443`.

### `tlsSocket.renegotiate(options, callback)`

<!-- YAML
added: v0.11.8
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `options`{对象}
    *   `rejectUnauthorized`{布尔值}如果不是`false`，则服务器证书为
        根据提供的 CA 列表进行验证。一`'error'`在以下情况下发出事件：
        验证失败;`err.code`包含 OpenSSL 错误代码。**违约：**
        `true`.
    *   `requestCert`

*   `callback`{函数}如果`renegotiate()`返回`true`，回调为
    附加到 一次`'secure'`事件。如果`renegotiate()`返回`false`,
    `callback`将在下一个价格变动中被调用并显示错误，除非
    `tlsSocket`已销毁，在这种情况下`callback`不会被调用
    完全。

*   返回：{布尔值}`true`如果重新谈判已启动，`false`否则。

这`tlsSocket.renegotiate()`方法启动 TLS 重新协商进程。
完成后，`callback`函数将传递单个参数
要么是`Error`（如果请求失败）或`null`.

此方法可用于请求对等方的证书后的安全
已建立连接。

作为服务器运行时，套接字将在以下情况下被销毁并显示错误
`handshakeTimeout`超时。

对于 TLSv1.3，重新协商无法启动，它不受
协议。

### `tlsSocket.setMaxSendFragment(size)`

<!-- YAML
added: v0.11.11
-->

*   `size`{数字}最大 TLS 片段大小。最大值为`16384`.
    **违约：** `16384`.
*   返回：{布尔值}

这`tlsSocket.setMaxSendFragment()`方法设置最大 TLS 片段大小。
返回`true`如果设置限制成功;`false`否则。

较小的片段大小可减少客户端上的缓冲延迟：较大
片段由 TLS 层缓冲，直到接收到整个片段
并验证其完整性;大片段可以跨越多个往返
并且它们的处理可能会因数据包丢失或重新排序而延迟。然而
较小的片段会增加额外的 TLS 成帧字节和 CPU 开销，这可能会
降低总体服务器吞吐量。

## `tls.checkServerIdentity(hostname, cert)`

<!-- YAML
added: v0.8.4
changes:
  - version:
      - v17.3.1
      - v16.13.2
      - v14.18.3
      - v12.22.9
    pr-url: https://github.com/nodejs-private/node-private/pull/300
    description: Support for `uniformResourceIdentifier` subject alternative
                 names has been disabled in response to CVE-2021-44531.
-->

*   `hostname`{字符串}用于验证证书的主机名或 IP 地址
    对。
*   `cert`{对象}一个[证书对象][certificate object]表示对等方的证书。
*   返回值：{错误|未定义}

验证证书`cert`颁发给`hostname`.

返回 {Error} 对象，并填充该对象`reason`,`host`和`cert`上
失败。成功时，返回 {未定义}。

此功能旨在与
`checkServerIdentity`可以传递给的选项[`tls.connect()`][tls.connect()]和作为
此类操作[证书对象][certificate object].对于其他目的，请考虑使用
[`x509.checkHost()`][x509.checkHost()]相反。

可以通过提供替代函数作为
`options.checkServerIdentity`传递给的选项`tls.connect()`.这
覆盖函数可以调用`tls.checkServerIdentity()`当然，要增加
通过额外验证完成的检查。

仅当证书通过了所有其他检查（例如
由受信任的 CA （`options.ca`).

早期版本的 Node.js给定的证书被错误地接受
`hostname`如果匹配`uniformResourceIdentifier`使用者备用名称
存在（请参见[CVE-2021-44531][]).希望接受的申请
`uniformResourceIdentifier`使用者替代名称可以使用自定义名称
`options.checkServerIdentity`实现所需行为的函数。

## `tls.connect(options[, callback])`

<!-- YAML
added: v0.11.3
changes:
  - version:
      - v15.1.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/35753
    description: Added `onread` option.
  - version:
      - v14.1.0
      - v13.14.0
    pr-url: https://github.com/nodejs/node/pull/32786
    description: The `highWaterMark` option is accepted now.
  - version:
      - v13.6.0
      - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/23188
    description: The `pskCallback` option is now supported.
  - version: v12.9.0
    pr-url: https://github.com/nodejs/node/pull/27836
    description: Support the `allowHalfOpen` option.
  - version: v12.4.0
    pr-url: https://github.com/nodejs/node/pull/27816
    description: The `hints` option is now supported.
  - version: v12.2.0
    pr-url: https://github.com/nodejs/node/pull/27497
    description: The `enableTrace` option is now supported.
  - version:
      - v11.8.0
      - v10.16.0
    pr-url: https://github.com/nodejs/node/pull/25517
    description: The `timeout` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12839
    description: The `lookup` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11984
    description: The `ALPNProtocols` option can be a `TypedArray` or
     `DataView` now.
  - version:
      - v5.3.0
      - v4.7.0
    pr-url: https://github.com/nodejs/node/pull/4246
    description: The `secureContext` option is supported now.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2564
    description: ALPN options are supported now.
-->

*   `options`{对象}
    *   `enableTrace`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `host`{字符串}客户端应连接到的主机。**违约：**
        `'localhost'`.
    *   `port`{数字}客户端应连接到的端口。
    *   `path`{字符串}创建与路径的 Unix 套接字连接。如果此选项是
        指定`host`和`port`将被忽略。
    *   `socket`{流。双工} 在给定套接字上建立安全连接
        而不是创建一个新的套接字。通常，这是
        [`net.Socket`][net.Socket]，但任何`Duplex`允许流。
        如果指定此选项，`path`,`host`和`port`被忽略，
        证书验证除外。通常，套接字已连接
        当传递给`tls.connect()`，但以后可以连接。
        连接/断开/销毁`socket`是用户的
        责任;叫`tls.connect()`不会造成`net.connect()`成为
        叫。
    *   `allowHalfOpen`{布尔值}如果设置为`false`，则套接字将
        当可读侧结束时，自动结束可写侧。如果
        `socket`选项已设置，此选项不起作用。查看`allowHalfOpen`
        选项[`net.Socket`][net.Socket]了解详情。**违约：** `false`.
    *   `rejectUnauthorized`{布尔值}如果不是`false`，则服务器证书为
        根据提供的 CA 列表进行验证。一`'error'`在以下情况下发出事件：
        验证失败;`err.code`包含 OpenSSL 错误代码。**违约：**
        `true`.
    *   `pskCallback`{函数}

        *   hint：{string} 从服务器发送到帮助客户端的可选消息
            确定在协商期间要使用的标识。
            总是`null`如果使用 TLS 1.3。
        *   返回：窗体中的 {对象}
            `{ psk: <Buffer|TypedArray|DataView>, identity: <string> }`
            或`null`以停止谈判过程。`psk`必须是
            与所选密码的摘要兼容。
            `identity`必须使用 UTF-8 编码。

        协商 TLS-PSK（预共享密钥）时，此函数称为
        具有可选标识`hint`由服务器提供或`null`
        在TLS 1.3的情况下，其中`hint`已删除。
        有必要提供自定义`tls.checkServerIdentity()`
        作为默认连接，将尝试检查主机名/ IP
        的服务器与证书，但这不适用于 PSK
        因为不会有证书存在。
        更多信息可以在[RFC 4279][].
    *   `ALPNProtocols`： {字符串\[]|缓冲器\[]|TypedArray\[]|数据视图\[]|缓冲区|
        TypedArray|数据视图}
        字符串数组，`Buffer`s,`TypedArray`s，或`DataView`s，或
        单`Buffer`,`TypedArray`或`DataView`包含受支持的 ALPN
        协议。`Buffer`s 应具有以下格式`[len][name][len][name]...`
        例如：`'\x08http/1.1\x08http/1.0'`，其中`len`字节是
        下一个协议名称。传递数组通常要简单得多，例如
        `['http/1.1', 'http/1.0']`.列表中较前面的协议具有更高的协议
        比后来的那些更受欢迎。
    *   `servername`：{字符串} SNI（服务器名称指示）TLS 的服务器名称
        外延。它是要连接到的主机的名称，并且必须是主机
        名称，而不是 IP 地址。多宿主服务器可以使用它来
        选择要呈现给客户端的正确证书，请参阅
        `SNICallback`选项[`tls.createServer()`][tls.createServer()].
    *   `checkServerIdentity(servername, cert)`{函数}回调函数
        要使用（而不是内置`tls.checkServerIdentity()`函数）
        检查服务器的主机名（或提供的`servername`什么时候
        显式设置）针对证书。这应该返回 {错误} 如果
        验证失败。该方法应返回`undefined`如果`servername`
        和`cert`已验证。
    *   `session`{缓冲区}一个`Buffer`实例，包含 TLS 会话。
    *   `minDHSize`{数字}DH 参数的最小大小（以位为单位）以接受
        TLS 连接。当服务器提供大小较小的 DH 参数时
        比`minDHSize`，则 TLS 连接被破坏并引发错误。
        **违约：** `1024`.
    *   `highWaterMark`：{数字} 与可读流一致`highWaterMark`参数。
        **违约：** `16 * 1024`.
    *   `secureContext`：使用
        [`tls.createSecureContext()`][tls.createSecureContext()].如果`secureContext`是*不*提供，一个
        将通过传递整个`options`对象
        `tls.createSecureContext()`.
    *   `onread`{对象}如果`socket`选项丢失，传入数据为
        存储在单个`buffer`并传递给提供的`callback`什么时候
        数据到达套接字，否则将忽略该选项。查看
        `onread`选项[`net.Socket`][net.Socket]了解详情。
    *   ...:[`tls.createSecureContext()`][tls.createSecureContext()]在以下情况下使用的选项：
        `secureContext`选项丢失，否则将忽略它们。
    *   ...： 任何[`socket.connect()`][socket.connect()]选项尚未列出。
*   `callback`{函数}
*   返回：{tls。TLSSocket}

这`callback`函数（如果指定）将被添加为
[`'secureConnect'`]['secureConnect']事件。

`tls.connect()`返回[`tls.TLSSocket`][tls.TLSSocket]对象。

与`https`应用程序接口`tls.connect()`不启用
默认情况下SNI（服务器名称指示）扩展，这可能会导致一些
服务器返回不正确的证书或拒绝连接
完全。要启用 SNI，请将`servername`另外选项
自`host`.

下面说明了 echo 服务器示例的客户端，来自
[`tls.createServer()`][tls.createServer()]:

```js
// Assumes an echo server that is listening on port 8000.
const tls = require('node:tls');
const fs = require('node:fs');

const options = {
  // Necessary only if the server requires client certificate authentication.
  key: fs.readFileSync('client-key.pem'),
  cert: fs.readFileSync('client-cert.pem'),

  // Necessary only if the server uses a self-signed certificate.
  ca: [ fs.readFileSync('server-cert.pem') ],

  // Necessary only if the server's cert isn't for "localhost".
  checkServerIdentity: () => { return null; },
};

const socket = tls.connect(8000, options, () => {
  console.log('client connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  process.stdin.pipe(socket);
  process.stdin.resume();
});
socket.setEncoding('utf8');
socket.on('data', (data) => {
  console.log(data);
});
socket.on('end', () => {
  console.log('server ends connection');
});
```

## `tls.connect(path[, options][, callback])`

<!-- YAML
added: v0.11.3
-->

*   `path`{字符串}的默认值`options.path`.
*   `options`{对象}看[`tls.connect()`][tls.connect()].
*   `callback`{函数}看[`tls.connect()`][tls.connect()].
*   返回：{tls。TLSSocket}

与 相同[`tls.connect()`][tls.connect()]除了`path`可提供
作为参数而不是选项。

路径选项（如果指定）将优先于 path 参数。

## `tls.connect(port[, host][, options][, callback])`

<!-- YAML
added: v0.11.3
-->

*   `port`{数字}的默认值`options.port`.
*   `host`{字符串}的默认值`options.host`.
*   `options`{对象}看[`tls.connect()`][tls.connect()].
*   `callback`{函数}看[`tls.connect()`][tls.connect()].
*   返回：{tls。TLSSocket}

与 相同[`tls.connect()`][tls.connect()]除了`port`和`host`可提供
作为参数而不是选项。

端口或主机选项（如果指定）将优先于任何端口或主机
论点。

## `tls.createSecureContext([options])`

<!-- YAML
added: v0.11.13
changes:
  - version: v12.12.0
    pr-url: https://github.com/nodejs/node/pull/28973
    description: Added `privateKeyIdentifier` and `privateKeyEngine` options
                 to get private key from an OpenSSL engine.
  - version: v12.11.0
    pr-url: https://github.com/nodejs/node/pull/29598
    description: Added `sigalgs` option to override supported signature
                 algorithms.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26209
    description: TLSv1.3 support added.
  - version: v11.5.0
    pr-url: https://github.com/nodejs/node/pull/24733
    description: The `ca:` option now supports `BEGIN TRUSTED CERTIFICATE`.
  - version:
     - v11.4.0
     - v10.16.0
    pr-url: https://github.com/nodejs/node/pull/24405
    description: The `minVersion` and `maxVersion` can be used to restrict
                 the allowed TLS protocol versions.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19794
    description: The `ecdhCurve` cannot be set to `false` anymore due to a
                 change in OpenSSL.
  - version: v9.3.0
    pr-url: https://github.com/nodejs/node/pull/14903
    description: The `options` parameter can now include `clientCertEngine`.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15206
    description: The `ecdhCurve` option can now be multiple `':'` separated
                 curve names or `'auto'`.
  - version: v7.3.0
    pr-url: https://github.com/nodejs/node/pull/10294
    description: If the `key` option is an array, individual entries do not
                 need a `passphrase` property anymore. `Array` entries can also
                 just be `string`s or `Buffer`s now.
  - version: v5.2.0
    pr-url: https://github.com/nodejs/node/pull/4099
    description: The `ca` option can now be a single string containing multiple
                 CA certificates.
-->

*   `options`{对象}
    *   `ca`{字符串|字符串\[]|缓冲区|Buffer\[]} （可选）覆盖受信任的 CA
        证书。默认为信任由 Mozilla 策划的知名 CA。
        当明确指定 CA 时，Mozilla 的 CA 将被完全替换
        使用此选项。该值可以是字符串或`Buffer`，或`Array`之
        字符串和/或`Buffer`s.任何字符串或`Buffer`可以包含多个 PEM
        CA 连接在一起。对等方的证书必须可链接到 CA
        受服务器信任，以便对连接进行身份验证。使用时
        无法链接到已知 CA（证书的 CA）的证书
        必须显式指定为受信任，否则连接将无法
        证实。
        如果对等方使用的证书与
        默认 CA，请使用`ca`选项以提供对等体的 CA 证书
        证书可以匹配或链接到。
        对于自签名证书，证书是其自己的 CA，并且必须是
        提供。
        对于 PEM 编码证书，支持的类型为“受信任的证书”，
        “X509 证书”和“证书”。
        另请参见[`tls.rootCertificates`][tls.rootCertificates].
    *   `cert`{字符串|字符串\[]|缓冲区|Buffer\[]} PEM 格式的证书链。一
        应按私钥提供证书链。每个证书链应
        由提供的私有的 PEM 格式证书组成`key`,
        后跟 PEM 格式的中间证书（如果有），按顺序排列，
        并且不包括根 CA（根 CA 必须预先为对等方所知，
        看`ca`).提供多个证书链时，它们不必位于
        与他们的私钥在 中的顺序相同`key`.如果中间
        未提供证书，对等方将无法验证
        证书，握手将失败。
    *   `sigalgs`{字符串}支持的签名算法的冒号分隔列表。
        该列表可以包含摘要算法 （`SHA256`,`MD5`等），公钥
        算法 （`RSA-PSS`,`ECDSA`等），两者的组合（例如
        'RSA+SHA384'） 或 TLS v1.3 方案名称（例如`rsa_pss_pss_sha512`).
        看[OpenSSL 手册页](https://www.openssl.org/docs/man1.1.1/man3/SSL_CTX_set1\_sigalgs_list.html)
        了解更多信息。
    *   `ciphers`{字符串}密码套件规范，替换默认值。为
        更多信息，请参阅[修改默认 TLS 密码套件][Modifying the default TLS cipher suite].允许
        密码可以通过以下方式获得[`tls.getCiphers()`][tls.getCiphers()].密码名称必须为
        大写，以便 OpenSSL 接受它们。
    *   `clientCertEngine`{字符串}OpenSSL 引擎的名称，它可以提供
        客户端证书。
    *   `crl`{字符串|字符串\[]|缓冲区|Buffer\[]} PEM 格式的 CRL （Certificate
        吊销列表）。
    *   `dhparam`{字符串|Buffer} Diffie-Hellman 参数，必需于
        [完美的前向保密][perfect forward secrecy].用`openssl dhparam`以创建参数。
        密钥长度必须大于或等于 1024 位，否则会出现错误
        将被抛出。尽管允许 1024 位，但请使用 2048 位或更大
        实现更强的安全性。如果省略或无效，参数将以静默方式显示
        丢弃的 DHE 密码将不可用。
    *   `ecdhCurve`{字符串}描述命名曲线或分隔冒号的字符串
        曲线 NID 或名称列表，例如`P-521:P-384:P-256`，用于
        ECDH关键协议。设置为`auto`以选择
        自动弯曲。用[`crypto.getCurves()`][crypto.getCurves()]获取列表
        可用的曲线名称。在最近的版本中，`openssl ecparam -list_curves`
        还将显示每个可用椭圆曲线的名称和描述。
        **违约：** [`tls.DEFAULT_ECDH_CURVE`][tls.DEFAULT_ECDH_CURVE].
    *   `honorCipherOrder`{布尔值}尝试使用服务器的密码套件
        偏好而不是客户的偏好。什么时候`true`原因
        `SSL_OP_CIPHER_SERVER_PREFERENCE`设置于`secureOptions`看
        [OpenSSL Options][]了解更多信息。
    *   `key`{字符串|字符串\[]|缓冲区|缓冲器\[]|对象\[]} PEM 中的私钥
        格式。PEM 允许选择对私钥进行加密。加密
        密钥将解密`options.passphrase`.多个键使用
        不同的算法可以作为未加密密钥的数组提供
        字符串或缓冲区，或格式中的对象数组
        `{pem: <string|buffer>[, passphrase: <string>]}`.对象窗体只能
        在数组中出现。`object.passphrase`是可选的。加密密钥将是
        解密`object.passphrase`如果提供，或`options.passphrase`如果
        事实并非如此。
    *   `privateKeyEngine`{字符串}用于获取私钥的 OpenSSL 引擎的名称
        从。应与`privateKeyIdentifier`.
    *   `privateKeyIdentifier`{字符串}由 管理的私钥的标识符
        一个OpenSSL引擎。应与`privateKeyEngine`.
        不应与`key`，因为这两个选项都定义了
        以不同的方式获取私钥。
    *   `maxVersion`{字符串}（可选）设置允许的最大 TLS 版本。一
        之`'TLSv1.3'`,`'TLSv1.2'`,`'TLSv1.1'`或`'TLSv1'`.无法指定
        以及`secureProtocol`选项;使用其中之一。
        **违约：** [`tls.DEFAULT_MAX_VERSION`][tls.DEFAULT_MAX_VERSION].
    *   `minVersion`{字符串}（可选）设置允许的最低 TLS 版本。一
        之`'TLSv1.3'`,`'TLSv1.2'`,`'TLSv1.1'`或`'TLSv1'`.无法指定
        以及`secureProtocol`选项;使用其中之一。避免
        设置为小于 TLSv1.2，但可能需要
        互操作性。
        **违约：** [`tls.DEFAULT_MIN_VERSION`][tls.DEFAULT_MIN_VERSION].
    *   `passphrase`{字符串}用于单个私钥和/或
        a PFX。
    *   `pfx`{字符串|字符串\[]|缓冲区|缓冲器\[]|Object\[]} PFX 或 PKCS12 编码
        私钥和证书链。`pfx`是提供的替代方法
        `key`和`cert`单独。PFX通常是加密的，如果是，
        `passphrase`将用于解密它。可提供多个 PFX
        作为未加密的 PFX 缓冲区数组，或以下形式的对象数组
        `{buf: <string|buffer>[, passphrase: <string>]}`.对象窗体只能
        在数组中出现。`object.passphrase`是可选的。加密的 PFX 将是
        解密`object.passphrase`如果提供，或`options.passphrase`如果
        事实并非如此。
    *   `secureOptions`{数字}（可选）影响 OpenSSL 协议行为，
        这通常不是必需的。如果有的话，这应该小心使用！
        值是`SSL_OP_*`选项从
        [OpenSSL Options][].
    *   `secureProtocol`{字符串}用于选择 TLS 协议的旧机制
        版本使用，它不支持独立控制的最小值和
        最高版本，并且不支持将协议限制为 TLSv1.3。
        `minVersion`和`maxVersion`相反。可能的值列为
        [SSL_METHODS][SSL_METHODS]中，将函数名称用作字符串。例如
        用`'TLSv1_1_method'`强制 TLS 版本 1.1，或`'TLS_method'`以允许
        任何 TLS 协议版本，最高可达 TLSv1.3。不建议使用 TLS
        版本小于 1.2，但互操作性可能需要它。
        **违约：**无，请参阅`minVersion`.
    *   `sessionIdContext`{字符串}服务器使用的不透明标识符，以确保
        会话状态不在应用程序之间共享。客户端未使用。
    *   `ticketKeys`：{Buffer} 48 字节的加密强伪随机
        数据。看[续会][Session Resumption]了解更多信息。
    *   `sessionTimeout`{数字}TLS 会话之后的秒数
        由服务器创建的内容将不再可恢复。看
        [续会][Session Resumption]了解更多信息。**违约：** `300`.

[`tls.createServer()`][tls.createServer()]设置`honorCipherOrder`选择
自`true`，则创建安全上下文的其他 API 将其保留为未设置状态。

[`tls.createServer()`][tls.createServer()]使用生成的 128 位截断 SHA1 哈希值
从`process.argv`作为 的默认值`sessionIdContext`选项，其他
创建安全上下文的 API 没有默认值。

这`tls.createSecureContext()`方法创建一个`SecureContext`对象。是的
可用作多个参数`tls`API，例如[`tls.createServer()`][tls.createServer()]
和[`server.addContext()`][server.addContext()]，但没有公共方法。

密钥是*必填*对于使用证书的密码。也`key`或
`pfx`可用于提供它。

如果`ca`选项未给出，则 Node.js 将默认使用
[Mozilla 公开信任的 CA 列表][Mozilla's publicly trusted list of CAs].

## `tls.createSecurePair([context][, isServer][, requestCert][, rejectUnauthorized][, options])`

<!-- YAML
added: v0.3.2
deprecated: v0.11.3
changes:
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2564
    description: ALPN options are supported now.
-->

> 稳定性：0 - 已弃用：使用[`tls.TLSSocket`][tls.TLSSocket]相反。

*   `context`{对象}返回的安全上下文对象
    `tls.createSecureContext()`
*   `isServer`{布尔值}`true`以指定此 TLS 连接应为
    作为服务器打开。
*   `requestCert`{布尔值}`true`以指定服务器是否应请求
    来自连接客户端的证书。仅适用于以下情况`isServer`是`true`.
*   `rejectUnauthorized`{布尔值}如果不是`false`服务器自动拒绝
    证书无效的客户端。仅适用于以下情况`isServer`是`true`.
*   `options`
    *   `enableTrace`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `secureContext`：来自 的 TLS 上下文对象[`tls.createSecureContext()`][tls.createSecureContext()]
    *   `isServer`： 如果`true`TLS 套接字将在服务器模式下实例化。
        **违约：** `false`.
    *   `server`{网.服务器} A[`net.Server`][net.Server]实例
    *   `requestCert`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `rejectUnauthorized`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `ALPNProtocols`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `SNICallback`： 请参阅[`tls.createServer()`][tls.createServer()]
    *   `session`{缓冲区}一个`Buffer`包含 TLS 会话的实例。
    *   `requestOCSP`{布尔值}如果`true`，指定 OCSP 状态请求
        扩展将被添加到客户端问候和`'OCSPResponse'`事件
        将在建立安全通信之前在套接字上发出。

创建具有两个流的新安全对对象，其中一个流读取和写入
加密数据，另一个读取和写入明文数据。
通常，加密流通过管道传输到/从传入的加密数据传出
流和明文一用作初始加密的替代品
流。

`tls.createSecurePair()`返回`tls.SecurePair`对象`cleartext`和
`encrypted`流属性。

用`cleartext`具有与 相同的 API[`tls.TLSSocket`][tls.TLSSocket].

这`tls.createSecurePair()`方法现在已被弃用，取而代之的是
`tls.TLSSocket()`.例如，代码：

```js
pair = tls.createSecurePair(/* ... */);
pair.encrypted.pipe(socket);
socket.pipe(pair.encrypted);
```

可以替换为：

```js
secureSocket = tls.TLSSocket(socket, options);
```

哪里`secureSocket`具有与 相同的 API`pair.cleartext`.

## `tls.createServer([options][, secureConnectionListener])`

<!-- YAML
added: v0.3.2
changes:
  - version: v12.3.0
    pr-url: https://github.com/nodejs/node/pull/27665
    description: The `options` parameter now supports `net.createServer()`
                 options.
  - version: v9.3.0
    pr-url: https://github.com/nodejs/node/pull/14903
    description: The `options` parameter can now include `clientCertEngine`.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11984
    description: The `ALPNProtocols` option can be a `TypedArray` or
     `DataView` now.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2564
    description: ALPN options are supported now.
-->

*   `options`{对象}
    *   `ALPNProtocols`： {字符串\[]|缓冲器\[]|TypedArray\[]|数据视图\[]|缓冲区|
        TypedArray|数据视图}
        字符串数组，`Buffer`s,`TypedArray`s，或`DataView`s，或单个
        `Buffer`,`TypedArray`或`DataView`包含受支持的 ALPN
        协议。`Buffer`s 应具有以下格式`[len][name][len][name]...`
        例如：`0x05hello0x05world`，其中第一个字节是下一个字节的长度
        协议名称。传递数组通常要简单得多，例如
        `['hello', 'world']`.（协议应按其优先级排序。
    *   `clientCertEngine`{字符串}OpenSSL 引擎的名称，它可以提供
        客户端证书。
    *   `enableTrace`{布尔值}如果`true`,[`tls.TLSSocket.enableTrace()`][tls.TLSSocket.enableTrace()]将是
        在新连接上调用。在安全保护后可以启用跟踪
        连接已建立，但必须使用此选项来跟踪安全
        连接设置。**违约：** `false`.
    *   `handshakeTimeout`{数字}如果 SSL/TLS 握手，则中止连接
        不在指定的毫秒数内完成。
        一个`'tlsClientError'`在 上发出`tls.Server`对象无论何时
        握手超时。**违约：** `120000`（120 秒）。
    *   `rejectUnauthorized`{布尔值}如果不是`false`服务器将拒绝任何
        未通过提供的 CA 列表授权的连接。这
        选项仅在以下情况下有效`requestCert`是`true`.**违约：** `true`.
    *   `requestCert`{布尔值}如果`true`服务器将从
        连接并尝试验证该证书的客户端。**违约：**
        `false`.
    *   `sessionTimeout`{数字}TLS 会话之后的秒数
        由服务器创建的内容将不再可恢复。看
        [续会][Session Resumption]了解更多信息。**违约：** `300`.
    *   `SNICallback(servername, callback)`{函数}一个函数，它将是
        如果客户端支持 SNI TLS 扩展，则调用。两个参数将是
        调用时传递：`servername`和`callback`.`callback`是一个
        采用两个可选参数的错误优先回调：`error`和`ctx`.
        `ctx`，如果提供，则为`SecureContext`实例。
        [`tls.createSecureContext()`][tls.createSecureContext()]可以用来得到一个适当的`SecureContext`.
        如果`callback`被叫出一个假的`ctx`参数，默认安全
        将使用服务器的上下文。如果`SNICallback`未提供
        将使用具有高级 API 的默认回调（见下文）。
    *   `ticketKeys`：{Buffer} 48 字节的加密强伪随机
        数据。看[续会][Session Resumption]了解更多信息。
    *   `pskCallback`{函数}

        *   套接字： {tls.TLSSocket} 服务器[`tls.TLSSocket`][tls.TLSSocket]的实例
            此连接。
        *   标识：从客户端发送的 {字符串} 标识参数。
        *   返回值：{缓冲区|TypedArray|DataView} 预共享密钥，该密钥必须是
            缓冲区或`null`以停止谈判过程。返回的 PSK 必须是
            与所选密码的摘要兼容。

        协商 TLS-PSK（预共享密钥）时，此函数称为
        使用客户端提供的标识。
        如果返回值为`null`谈判进程将停止，并且
        “unknown_psk_identity”警报消息将发送给另一方。
        如果服务器希望隐藏 PSK 身份未知的事实，
        回调必须提供一些随机数据，如`psk`以建立连接
        在协商完成之前，以“decrypt_error”失败。
        默认情况下禁用 PSK 密码，因此使用 TLS-PSK
        需要显式指定密码套件，其中包含`ciphers`选择。
        更多信息可以在[RFC 4279][].
    *   `pskIdentityHint`{string} 可选项提示，以发送给客户端以帮助
        在 TLS-PSK 协商期间选择标识。将被忽略
        在 TLS 1.3 中。在未能设置 pskIdentityHint 时`'tlsClientError'`将是
        发出方式`'ERR_TLS_PSK_SET_IDENTIY_HINT_FAILED'`法典。
    *   ...： 任何[`tls.createSecureContext()`][tls.createSecureContext()]可以提供选项。为
        服务器，标识选项 （`pfx`,`key`/`cert`或`pskCallback`)
        通常是必需的。
    *   ...： 任何[`net.createServer()`][net.createServer()]可以提供选项。
*   `secureConnectionListener`{函数}
*   返回：{tls。服务器}

创建新的[`tls.Server`][tls.Server].这`secureConnectionListener`，如果提供，则为
自动设置为[`'secureConnection'`]['secureConnection']事件。

这`ticketKeys`选项在`node:cluster`模块
工人。

下面说明了一个简单的 echo 服务器：

```js
const tls = require('node:tls');
const fs = require('node:fs');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),

  // This is necessary only if using client certificate authentication.
  requestCert: true,

  // This is necessary only if the client uses a self-signed certificate.
  ca: [ fs.readFileSync('client-cert.pem') ]
};

const server = tls.createServer(options, (socket) => {
  console.log('server connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  socket.write('welcome!\n');
  socket.setEncoding('utf8');
  socket.pipe(socket);
});
server.listen(8000, () => {
  console.log('server bound');
});
```

可以通过使用示例客户端连接到服务器来测试服务器
[`tls.connect()`][tls.connect()].

## `tls.getCiphers()`

<!-- YAML
added: v0.10.2
-->

*   返回：{字符串\[]}

返回一个数组，其中包含受支持的 TLS 密码的名称。这些名称是
由于历史原因，小写，但必须大写才能用于
这`ciphers`选项[`tls.createSecureContext()`][tls.createSecureContext()].

默认情况下，并非所有受支持的密码都处于启用状态。看
[修改默认 TLS 密码套件][Modifying the default TLS cipher suite].

以 开头的密码名称`'tls_'`适用于 TLSv1.3，所有其他用于
TLSv1.2 及更低版本。

```js
console.log(tls.getCiphers()); // ['aes128-gcm-sha256', 'aes128-sha', ...]
```

## `tls.rootCertificates`

<!-- YAML
added: v12.3.0
-->

*   {字符串\[]}

表示根证书的不可变字符串数组（PEM 格式）
从当前 Node.js 版本提供的捆绑 Mozilla CA 存储中。

由 Node.js 提供的捆绑 CA 存储是 Mozilla CA 商店的快照。
在发布时已修复。它在所有支持的平台上都是相同的。

## `tls.DEFAULT_ECDH_CURVE`

<!-- YAML
added: v0.11.13
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16853
    description: Default value changed to `'auto'`.
-->

用于 tls 服务器中的 ECDH 密钥协议的默认曲线名称。这
默认值为`'auto'`.看[`tls.createSecureContext()`][tls.createSecureContext()]进一步
信息。

## `tls.DEFAULT_MAX_VERSION`

<!-- YAML
added: v11.4.0
-->

*   {字符串}的默认值`maxVersion`选项
    [`tls.createSecureContext()`][tls.createSecureContext()].可以为其分配任何受支持的 TLS
    协议版本，`'TLSv1.3'`,`'TLSv1.2'`,`'TLSv1.1'`或`'TLSv1'`.
    **违约：** `'TLSv1.3'`，除非使用 CLI 选项进行更改。用
    `--tls-max-v1.2`将默认值设置为`'TLSv1.2'`.用`--tls-max-v1.3`集
    默认值`'TLSv1.3'`.如果提供了多个选项，则
    使用最高最大值。

## `tls.DEFAULT_MIN_VERSION`

<!-- YAML
added: v11.4.0
-->

*   {字符串}的默认值`minVersion`选项
    [`tls.createSecureContext()`][tls.createSecureContext()].可以为其分配任何受支持的 TLS
    协议版本，`'TLSv1.3'`,`'TLSv1.2'`,`'TLSv1.1'`或`'TLSv1'`.
    **违约：** `'TLSv1.2'`，除非使用 CLI 选项进行更改。用
    `--tls-min-v1.0`将默认值设置为`'TLSv1'`.用`--tls-min-v1.1`集
    默认值`'TLSv1.1'`.用`--tls-min-v1.3`将默认值设置为
    `'TLSv1.3'`.如果提供了多个选项，则最低值为
    使用。

[CVE-2021-44531]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44531

[Chrome's 'modern cryptography' setting]: https://www.chromium.org/Home/chromium-security/education/tls#TOC-Cipher-Suites

[DHE]: https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange

[ECDHE]: https://en.wikipedia.org/wiki/Elliptic_curve_Diffie%E2%80%93Hellman

[Modifying the default TLS cipher suite]: #modifying-the-default-tls-cipher-suite

[Mozilla's publicly trusted list of CAs]: https://hg.mozilla.org/mozilla-central/raw-file/tip/security/nss/lib/ckfw/builtins/certdata.txt

[OCSP request]: https://en.wikipedia.org/wiki/OCSP_stapling

[OpenSSL Options]: crypto.md#openssl-options

[RFC 2246]: https://www.ietf.org/rfc/rfc2246.txt

[RFC 4086]: https://tools.ietf.org/html/rfc4086

[RFC 4279]: https://tools.ietf.org/html/rfc4279

[RFC 5077]: https://tools.ietf.org/html/rfc5077

[RFC 5929]: https://tools.ietf.org/html/rfc5929

[SSL_METHODS]: https://www.openssl.org/docs/man1.1.1/man7/ssl.html#Dealing-with-Protocol-Methods

[Session Resumption]: #session-resumption

[Stream]: stream.md#stream

[TLS recommendations]: https://wiki.mozilla.org/Security/Server_Side_TLS

[`'newSession'`]: #event-newsession

[`'resumeSession'`]: #event-resumesession

[`'secureConnect'`]: #event-secureconnect

[`'secureConnection'`]: #event-secureconnection

[`'session'`]: #event-session

[`--tls-cipher-list`]: cli.md#--tls-cipher-listlist

[`Duplex`]: stream.md#class-streamduplex

[`NODE_OPTIONS`]: cli.md#node_optionsoptions

[`SSL_export_keying_material`]: https://www.openssl.org/docs/man1.1.1/man3/SSL_export_keying_material.html

[`SSL_get_version`]: https://www.openssl.org/docs/man1.1.1/man3/SSL_get_version.html

[`crypto.getCurves()`]: crypto.md#cryptogetcurves

[`import()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import

[`net.Server.address()`]: net.md#serveraddress

[`net.Server`]: net.md#class-netserver

[`net.Socket`]: net.md#class-netsocket

[`net.createServer()`]: net.md#netcreateserveroptions-connectionlistener

[`server.addContext()`]: #serveraddcontexthostname-context

[`server.getTicketKeys()`]: #servergetticketkeys

[`server.listen()`]: net.md#serverlisten

[`server.setTicketKeys()`]: #serversetticketkeyskeys

[`socket.connect()`]: net.md#socketconnectoptions-connectlistener

[`tls.DEFAULT_ECDH_CURVE`]: #tlsdefault_ecdh_curve

[`tls.DEFAULT_MAX_VERSION`]: #tlsdefault_max_version

[`tls.DEFAULT_MIN_VERSION`]: #tlsdefault_min_version

[`tls.Server`]: #class-tlsserver

[`tls.TLSSocket.enableTrace()`]: #tlssocketenabletrace

[`tls.TLSSocket.getPeerCertificate()`]: #tlssocketgetpeercertificatedetailed

[`tls.TLSSocket.getProtocol()`]: #tlssocketgetprotocol

[`tls.TLSSocket.getSession()`]: #tlssocketgetsession

[`tls.TLSSocket.getTLSTicket()`]: #tlssocketgettlsticket

[`tls.TLSSocket`]: #class-tlstlssocket

[`tls.connect()`]: #tlsconnectoptions-callback

[`tls.createSecureContext()`]: #tlscreatesecurecontextoptions

[`tls.createSecurePair()`]: #tlscreatesecurepaircontext-isserver-requestcert-rejectunauthorized-options

[`tls.createServer()`]: #tlscreateserveroptions-secureconnectionlistener

[`tls.getCiphers()`]: #tlsgetciphers

[`tls.rootCertificates`]: #tlsrootcertificates

[`x509.checkHost()`]: crypto.md#x509checkhostname-options

[asn1.js]: https://www.npmjs.com/package/asn1.js

[certificate object]: #certificate-object

[cipher list format]: https://www.openssl.org/docs/man1.1.1/man1/ciphers.html#CIPHER-LIST-FORMAT

[forward secrecy]: https://en.wikipedia.org/wiki/Perfect_forward_secrecy

[perfect forward secrecy]: #perfect-forward-secrecy
