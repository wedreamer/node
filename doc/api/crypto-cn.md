# 加密

<!--introduced_in=v0.3.6-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/crypto.js -->

这`node:crypto`模块提供加密功能，其中包括
用于 OpenSSL 的哈希、HMAC、密码、解密、签名和验证的包装器集
功能。

```mjs
const { createHmac } = await import('node:crypto');

const secret = 'abcdefg';
const hash = createHmac('sha256', secret)
               .update('I love cupcakes')
               .digest('hex');
console.log(hash);
// Prints:
//   c0fa1bc00531bd78ef38c628449c5102aeabd49b5dc3a2a516ea6ea959d6658e
```

```cjs
const crypto = require('node:crypto');

const secret = 'abcdefg';
const hash = crypto.createHmac('sha256', secret)
                   .update('I love cupcakes')
                   .digest('hex');
console.log(hash);
// Prints:
//   c0fa1bc00531bd78ef38c628449c5102aeabd49b5dc3a2a516ea6ea959d6658e
```

## 确定加密支持是否不可用

.js 可以在不包含对
`node:crypto`模块。在这种情况下，尝试`import`从`crypto`或
叫`require('node:crypto')`将导致引发错误。

使用 CommonJS 时，可以使用 try/catch 捕获引发的错误：

<!-- eslint-skip -->

```cjs
let crypto;
try {
  crypto = require('node:crypto');
} catch (err) {
  console.log('crypto support is disabled!');
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
let crypto;
try {
  crypto = await import('node:crypto');
} catch (err) {
  console.log('crypto support is disabled!');
}
```

## 类：`Certificate`

<!-- YAML
added: v0.11.8
-->

SPKAC 是一种证书签名请求机制，最初由
Netscape，并被正式指定为[HTML5的`keygen`元素][HTML5's keygen element].

`<keygen>`已弃用，因为[HTML 5.2][]和新项目
不应再使用此元素。

这`node:crypto`模块提供`Certificate`用于处理 SPKAC 的类
数据。最常见的用法是处理HTML5生成的输出
`<keygen>`元素。节点.js用途[OpenSSL的SPKAC实现][OpenSSL's SPKAC implementation]内部。

### 静态方法：`Certificate.exportChallenge(spkac[, encoding])`

<!-- YAML
added: v9.0.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The spkac argument can be an ArrayBuffer. Limited the size of
                 the spkac argument to a maximum of 2**31 - 1 bytes.
-->

*   `spkac`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`spkac`字符串。
*   返回：{缓冲区} 质询组件`spkac`数据结构，其中
    包括一个公钥和一个挑战。

```mjs
const { Certificate } = await import('node:crypto');
const spkac = getSpkacSomehow();
const challenge = Certificate.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Prints: the challenge as a UTF8 string
```

```cjs
const { Certificate } = require('node:crypto');
const spkac = getSpkacSomehow();
const challenge = Certificate.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Prints: the challenge as a UTF8 string
```

### 静态方法：`Certificate.exportPublicKey(spkac[, encoding])`

<!-- YAML
added: v9.0.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The spkac argument can be an ArrayBuffer. Limited the size of
                 the spkac argument to a maximum of 2**31 - 1 bytes.
-->

*   `spkac`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`spkac`字符串。
*   返回： {Buffer}`spkac`数据结构
    其中包括公钥和挑战。

```mjs
const { Certificate } = await import('node:crypto');
const spkac = getSpkacSomehow();
const publicKey = Certificate.exportPublicKey(spkac);
console.log(publicKey);
// Prints: the public key as <Buffer ...>
```

```cjs
const { Certificate } = require('node:crypto');
const spkac = getSpkacSomehow();
const publicKey = Certificate.exportPublicKey(spkac);
console.log(publicKey);
// Prints: the public key as <Buffer ...>
```

### 静态方法：`Certificate.verifySpkac(spkac[, encoding])`

<!-- YAML
added: v9.0.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The spkac argument can be an ArrayBuffer. Added encoding.
                 Limited the size of the spkac argument to a maximum of
                 2**31 - 1 bytes.
-->

*   `spkac`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`spkac`字符串。
*   返回：{布尔值}`true`如果给定`spkac`数据结构有效，
    `false`否则。

```mjs
import { Buffer } from 'node:buffer';
const { Certificate } = await import('node:crypto');

const spkac = getSpkacSomehow();
console.log(Certificate.verifySpkac(Buffer.from(spkac)));
// Prints: true or false
```

```cjs
const { Certificate } = require('node:crypto');
const { Buffer } = require('node:buffer');

const spkac = getSpkacSomehow();
console.log(Certificate.verifySpkac(Buffer.from(spkac)));
// Prints: true or false
```

### 旧版 API

> 稳定性：0 - 已弃用

作为旧接口，可以创建新的实例
这`crypto.Certificate`类，如下面的示例所示。

#### `new crypto.Certificate()`

的实例`Certificate`类可以使用`new`关键词
或致电`crypto.Certificate()`作为功能：

```mjs
const { Certificate } = await import('node:crypto');

const cert1 = new Certificate();
const cert2 = Certificate();
```

```cjs
const { Certificate } = require('node:crypto');

const cert1 = new Certificate();
const cert2 = Certificate();
```

#### `certificate.exportChallenge(spkac[, encoding])`

<!-- YAML
added: v0.11.8
-->

*   `spkac`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`spkac`字符串。
*   返回：{缓冲区} 质询组件`spkac`数据结构，其中
    包括一个公钥和一个挑战。

```mjs
const { Certificate } = await import('node:crypto');
const cert = Certificate();
const spkac = getSpkacSomehow();
const challenge = cert.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Prints: the challenge as a UTF8 string
```

```cjs
const { Certificate } = require('node:crypto');
const cert = Certificate();
const spkac = getSpkacSomehow();
const challenge = cert.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Prints: the challenge as a UTF8 string
```

#### `certificate.exportPublicKey(spkac[, encoding])`

<!-- YAML
added: v0.11.8
-->

*   `spkac`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`spkac`字符串。
*   返回： {Buffer}`spkac`数据结构
    其中包括公钥和挑战。

```mjs
const { Certificate } = await import('node:crypto');
const cert = Certificate();
const spkac = getSpkacSomehow();
const publicKey = cert.exportPublicKey(spkac);
console.log(publicKey);
// Prints: the public key as <Buffer ...>
```

```cjs
const { Certificate } = require('node:crypto');
const cert = Certificate();
const spkac = getSpkacSomehow();
const publicKey = cert.exportPublicKey(spkac);
console.log(publicKey);
// Prints: the public key as <Buffer ...>
```

#### `certificate.verifySpkac(spkac[, encoding])`

<!-- YAML
added: v0.11.8
-->

*   `spkac`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`spkac`字符串。
*   返回：{布尔值}`true`如果给定`spkac`数据结构有效，
    `false`否则。

```mjs
import { Buffer } from 'node:buffer';
const { Certificate } = await import('node:crypto');

const cert = Certificate();
const spkac = getSpkacSomehow();
console.log(cert.verifySpkac(Buffer.from(spkac)));
// Prints: true or false
```

```cjs
const { Certificate } = require('node:crypto');
const { Buffer } = require('node:buffer');

const cert = Certificate();
const spkac = getSpkacSomehow();
console.log(cert.verifySpkac(Buffer.from(spkac)));
// Prints: true or false
```

## 类：`Cipher`

<!-- YAML
added: v0.1.94
-->

*   扩展：{流。转换}

的实例`Cipher`类用于加密数据。该类可以是
以下列两种方式之一使用：

*   作为[流][stream]既可读又可写，其中纯未加密
    写入数据以在可读端生成加密数据，或者
*   使用[`cipher.update()`][cipher.update()]和[`cipher.final()`][cipher.final()]生产方法
    加密的数据。

这[`crypto.createCipher()`][crypto.createCipher()]或[`crypto.createCipheriv()`][crypto.createCipheriv()]方法是
用于创建`Cipher`实例。`Cipher`不创建对象
直接使用`new`关键词。

示例：使用`Cipher`对象作为流：

```mjs
const {
  scrypt,
  randomFill,
  createCipheriv
} = await import('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';

// First, we'll generate the key. The key length is dependent on the algorithm.
// In this case for aes192, it is 24 bytes (192 bits).
scrypt(password, 'salt', 24, (err, key) => {
  if (err) throw err;
  // Then, we'll generate a random initialization vector
  randomFill(new Uint8Array(16), (err, iv) => {
    if (err) throw err;

    // Once we have the key and iv, we can create and use the cipher...
    const cipher = createCipheriv(algorithm, key, iv);

    let encrypted = '';
    cipher.setEncoding('hex');

    cipher.on('data', (chunk) => encrypted += chunk);
    cipher.on('end', () => console.log(encrypted));

    cipher.write('some clear text data');
    cipher.end();
  });
});
```

```cjs
const {
  scrypt,
  randomFill,
  createCipheriv
} = require('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';

// First, we'll generate the key. The key length is dependent on the algorithm.
// In this case for aes192, it is 24 bytes (192 bits).
scrypt(password, 'salt', 24, (err, key) => {
  if (err) throw err;
  // Then, we'll generate a random initialization vector
  randomFill(new Uint8Array(16), (err, iv) => {
    if (err) throw err;

    // Once we have the key and iv, we can create and use the cipher...
    const cipher = createCipheriv(algorithm, key, iv);

    let encrypted = '';
    cipher.setEncoding('hex');

    cipher.on('data', (chunk) => encrypted += chunk);
    cipher.on('end', () => console.log(encrypted));

    cipher.write('some clear text data');
    cipher.end();
  });
});
```

示例：使用`Cipher`和管道流：

```mjs
import {
  createReadStream,
  createWriteStream,
} from 'node:fs';

import {
  pipeline
} from 'node:stream';

const {
  scrypt,
  randomFill,
  createCipheriv
} = await import('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';

// First, we'll generate the key. The key length is dependent on the algorithm.
// In this case for aes192, it is 24 bytes (192 bits).
scrypt(password, 'salt', 24, (err, key) => {
  if (err) throw err;
  // Then, we'll generate a random initialization vector
  randomFill(new Uint8Array(16), (err, iv) => {
    if (err) throw err;

    const cipher = createCipheriv(algorithm, key, iv);

    const input = createReadStream('test.js');
    const output = createWriteStream('test.enc');

    pipeline(input, cipher, output, (err) => {
      if (err) throw err;
    });
  });
});
```

```cjs
const {
  createReadStream,
  createWriteStream,
} = require('node:fs');

const {
  pipeline
} = require('node:stream');

const {
  scrypt,
  randomFill,
  createCipheriv,
} = require('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';

// First, we'll generate the key. The key length is dependent on the algorithm.
// In this case for aes192, it is 24 bytes (192 bits).
scrypt(password, 'salt', 24, (err, key) => {
  if (err) throw err;
  // Then, we'll generate a random initialization vector
  randomFill(new Uint8Array(16), (err, iv) => {
    if (err) throw err;

    const cipher = createCipheriv(algorithm, key, iv);

    const input = createReadStream('test.js');
    const output = createWriteStream('test.enc');

    pipeline(input, cipher, output, (err) => {
      if (err) throw err;
    });
  });
});
```

示例：使用[`cipher.update()`][cipher.update()]和[`cipher.final()`][cipher.final()]方法：

```mjs
const {
  scrypt,
  randomFill,
  createCipheriv
} = await import('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';

// First, we'll generate the key. The key length is dependent on the algorithm.
// In this case for aes192, it is 24 bytes (192 bits).
scrypt(password, 'salt', 24, (err, key) => {
  if (err) throw err;
  // Then, we'll generate a random initialization vector
  randomFill(new Uint8Array(16), (err, iv) => {
    if (err) throw err;

    const cipher = createCipheriv(algorithm, key, iv);

    let encrypted = cipher.update('some clear text data', 'utf8', 'hex');
    encrypted += cipher.final('hex');
    console.log(encrypted);
  });
});
```

```cjs
const {
  scrypt,
  randomFill,
  createCipheriv,
} = require('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';

// First, we'll generate the key. The key length is dependent on the algorithm.
// In this case for aes192, it is 24 bytes (192 bits).
scrypt(password, 'salt', 24, (err, key) => {
  if (err) throw err;
  // Then, we'll generate a random initialization vector
  randomFill(new Uint8Array(16), (err, iv) => {
    if (err) throw err;

    const cipher = createCipheriv(algorithm, key, iv);

    let encrypted = cipher.update('some clear text data', 'utf8', 'hex');
    encrypted += cipher.final('hex');
    console.log(encrypted);
  });
});
```

### `cipher.final([outputEncoding])`

<!-- YAML
added: v0.1.94
-->

*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串} 任何剩余的加密内容。
    如果`outputEncoding`指定，字符串为
    返回。如果`outputEncoding`未提供，则[`Buffer`][Buffer]返回。

一旦`cipher.final()`方法已被调用，`Cipher`对象不能否
用于加密数据的时间更长。尝试呼叫`cipher.final()`超过
一次将导致抛出错误。

### `cipher.getAuthTag()`

<!-- YAML
added: v1.0.0
-->

*   返回：{缓冲区} 使用经过身份验证的加密模式 （`GCM`,`CCM`,
    `OCB`和`chacha20-poly1305`当前支持），则
    `cipher.getAuthTag()`方法返回
    [`Buffer`][Buffer]包含*身份验证标记*已从以下位置计算得出
    给定的数据。

这`cipher.getAuthTag()`方法只应在加密后调用
已使用[`cipher.final()`][cipher.final()]方法。

如果`authTagLength`选项是在`cipher`实例的创建，
此函数将完全返回`authTagLength`字节。

### `cipher.setAAD(buffer[, options])`

<!-- YAML
added: v1.0.0
-->

*   `buffer`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
    *   `plaintextLength`{数字}
    *   `encoding`{字符串}在以下情况下要使用的字符串编码`buffer`是一个字符串。
*   返回：{密码}用于方法链接。

使用经过身份验证的加密模式 （`GCM`,`CCM`,`OCB`和
`chacha20-poly1305`是
当前支持），`cipher.setAAD()`方法设置用于
*其他经过身份验证的数据*（AAD） 输入参数。

这`plaintextLength`选项对于`GCM`和`OCB`.使用时`CCM`,
这`plaintextLength`选项必须指定，并且其值必须与
纯文本的长度（以字节为单位）。看[断续器模式][CCM mode].

这`cipher.setAAD()`方法必须在之前调用[`cipher.update()`][cipher.update()].

### `cipher.setAutoPadding([autoPadding])`

<!-- YAML
added: v0.7.1
-->

*   `autoPadding`{布尔值}**违约：** `true`
*   返回：{密码}用于方法链接。

使用块加密算法时，`Cipher`类将自动
将输入数据的填充添加到适当的块大小。禁用
默认填充调用`cipher.setAutoPadding(false)`.

什么时候`autoPadding`是`false`，整个输入数据的长度必须为
密码块大小的倍数或[`cipher.final()`][cipher.final()]将引发错误。
禁用自动填充对于非标准填充很有用，例如
用`0x0`而不是 PKCS 填充。

这`cipher.setAutoPadding()`方法必须在之前调用
[`cipher.final()`][cipher.final()].

### `cipher.update(data[, inputEncoding][, outputEncoding])`

<!-- YAML
added: v0.1.94
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的数据。
*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

更新密码`data`.如果`inputEncoding`参数被给出，
这`data`
参数是使用指定编码的字符串。如果`inputEncoding`
参数未给出，`data`必须是[`Buffer`][Buffer],`TypedArray`或
`DataView`.如果`data`是一个[`Buffer`][Buffer],`TypedArray`或`DataView`然后
`inputEncoding`被忽略。

这`outputEncoding`指定加密的输出格式
数据。如果`outputEncoding`
，则返回使用指定编码的字符串。如果不是
`outputEncoding`提供，一个[`Buffer`][Buffer]返回。

这`cipher.update()`可以使用新数据多次调用方法，直到
[`cipher.final()`][cipher.final()]被调用。叫`cipher.update()`后
[`cipher.final()`][cipher.final()]将导致引发错误。

## 类：`Decipher`

<!-- YAML
added: v0.1.94
-->

*   扩展：{流。转换}

的实例`Decipher`类用于解密数据。该类可以是
以下列两种方式之一使用：

*   作为[流][stream]既可读又可写，其中纯加密
    写入数据以在可读端生成未加密的数据，或者
*   使用[`decipher.update()`][decipher.update()]和[`decipher.final()`][decipher.final()]方法
    生成未加密的数据。

这[`crypto.createDecipher()`][crypto.createDecipher()]或[`crypto.createDecipheriv()`][crypto.createDecipheriv()]方法是
用于创建`Decipher`实例。`Decipher`不创建对象
直接使用`new`关键词。

示例：使用`Decipher`对象作为流：

```mjs
import { Buffer } from 'node:buffer';
const {
  scryptSync,
  createDecipheriv
} = await import('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Key length is dependent on the algorithm. In this case for aes192, it is
// 24 bytes (192 bits).
// Use the async `crypto.scrypt()` instead.
const key = scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = createDecipheriv(algorithm, key, iv);

let decrypted = '';
decipher.on('readable', () => {
  let chunk;
  while (null !== (chunk = decipher.read())) {
    decrypted += chunk.toString('utf8');
  }
});
decipher.on('end', () => {
  console.log(decrypted);
  // Prints: some clear text data
});

// Encrypted with same algorithm, key and iv.
const encrypted =
  'e5f79c5915c02171eec6b212d5520d44480993d7d622a7c4c2da32f6efda0ffa';
decipher.write(encrypted, 'hex');
decipher.end();
```

```cjs
const {
  scryptSync,
  createDecipheriv,
} = require('node:crypto');
const { Buffer } = require('node:buffer');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Key length is dependent on the algorithm. In this case for aes192, it is
// 24 bytes (192 bits).
// Use the async `crypto.scrypt()` instead.
const key = scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = createDecipheriv(algorithm, key, iv);

let decrypted = '';
decipher.on('readable', () => {
  let chunk;
  while (null !== (chunk = decipher.read())) {
    decrypted += chunk.toString('utf8');
  }
});
decipher.on('end', () => {
  console.log(decrypted);
  // Prints: some clear text data
});

// Encrypted with same algorithm, key and iv.
const encrypted =
  'e5f79c5915c02171eec6b212d5520d44480993d7d622a7c4c2da32f6efda0ffa';
decipher.write(encrypted, 'hex');
decipher.end();
```

示例：使用`Decipher`和管道流：

```mjs
import {
  createReadStream,
  createWriteStream,
} from 'node:fs';
import { Buffer } from 'node:buffer';
const {
  scryptSync,
  createDecipheriv
} = await import('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Use the async `crypto.scrypt()` instead.
const key = scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = createDecipheriv(algorithm, key, iv);

const input = createReadStream('test.enc');
const output = createWriteStream('test.js');

input.pipe(decipher).pipe(output);
```

```cjs
const {
  createReadStream,
  createWriteStream,
} = require('node:fs');
const {
  scryptSync,
  createDecipheriv,
} = require('node:crypto');
const { Buffer } = require('node:buffer');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Use the async `crypto.scrypt()` instead.
const key = scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = createDecipheriv(algorithm, key, iv);

const input = createReadStream('test.enc');
const output = createWriteStream('test.js');

input.pipe(decipher).pipe(output);
```

示例：使用[`decipher.update()`][decipher.update()]和[`decipher.final()`][decipher.final()]方法：

```mjs
import { Buffer } from 'node:buffer';
const {
  scryptSync,
  createDecipheriv
} = await import('node:crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Use the async `crypto.scrypt()` instead.
const key = scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = createDecipheriv(algorithm, key, iv);

// Encrypted using same algorithm, key and iv.
const encrypted =
  'e5f79c5915c02171eec6b212d5520d44480993d7d622a7c4c2da32f6efda0ffa';
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
// Prints: some clear text data
```

```cjs
const {
  scryptSync,
  createDecipheriv,
} = require('node:crypto');
const { Buffer } = require('node:buffer');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Use the async `crypto.scrypt()` instead.
const key = scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = createDecipheriv(algorithm, key, iv);

// Encrypted using same algorithm, key and iv.
const encrypted =
  'e5f79c5915c02171eec6b212d5520d44480993d7d622a7c4c2da32f6efda0ffa';
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
// Prints: some clear text data
```

### `decipher.final([outputEncoding])`

<!-- YAML
added: v0.1.94
-->

*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串} 任何剩余的解密内容。
    如果`outputEncoding`指定，字符串为
    返回。如果`outputEncoding`未提供，则[`Buffer`][Buffer]返回。

一旦`decipher.final()`方法已被调用，`Decipher`对象可以
不再用于解密数据。尝试呼叫`decipher.final()`更多
超过一次将导致抛出错误。

### `decipher.setAAD(buffer[, options])`

<!-- YAML
added: v1.0.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The buffer argument can be a string or ArrayBuffer and is
                limited to no more than 2 ** 31 - 1 bytes.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9398
    description: This method now returns a reference to `decipher`.
-->

*   `buffer`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
    *   `plaintextLength`{数字}
    *   `encoding`{字符串}字符串编码在以下情况下使用`buffer`是一个字符串。
*   返回：{解密}用于方法链接。

使用经过身份验证的加密模式 （`GCM`,`CCM`,`OCB`和
`chacha20-poly1305`是
当前支持），`decipher.setAAD()`方法设置用于
*其他经过身份验证的数据*（AAD） 输入参数。

这`options`参数对于`GCM`.使用时`CCM`这
`plaintextLength`选项，并且其值必须与长度匹配
的密文（以字节为单位）。看[断续器模式][CCM mode].

这`decipher.setAAD()`方法必须在之前调用[`decipher.update()`][decipher.update()].

将字符串作为传递时`buffer`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

### `decipher.setAuthTag(buffer[, encoding])`

<!-- YAML
added: v1.0.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The buffer argument can be a string or ArrayBuffer and is
                limited to no more than 2 ** 31 - 1 bytes.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/17825
    description: This method now throws if the GCM tag length is invalid.
  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9398
    description: This method now returns a reference to `decipher`.
-->

*   `buffer`{字符串|缓冲区|ArrayBuffer|TypedArray|数据视图}
*   `encoding`{字符串}字符串编码在以下情况下使用`buffer`是一个字符串。
*   返回：{解密}用于方法链接。

使用经过身份验证的加密模式 （`GCM`,`CCM`,`OCB`和
`chacha20-poly1305`是
当前支持），`decipher.setAuthTag()`方法用于传入
收到*身份验证标记*.如果未提供标记，或者如果密文
已被篡改，[`decipher.final()`][decipher.final()]将抛出，指示
由于身份验证失败，应丢弃密码文本。如果标签长度
根据[NIST SP 800-38D][]或与 的值不匹配
`authTagLength`选择`decipher.setAuthTag()`将引发错误。

这`decipher.setAuthTag()`方法必须在之前调用[`decipher.update()`][decipher.update()]
为`CCM`模式或之前[`decipher.final()`][decipher.final()]为`GCM`和`OCB`模式和
`chacha20-poly1305`.
`decipher.setAuthTag()`只能调用一次。

当传递字符串作为身份验证标记时，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

### `decipher.setAutoPadding([autoPadding])`

<!-- YAML
added: v0.7.1
-->

*   `autoPadding`{布尔值}**违约：** `true`
*   返回：{解密}用于方法链接。

当数据在没有标准块填充的情况下加密时，调用
`decipher.setAutoPadding(false)`将禁用自动填充以防止
[`decipher.final()`][decipher.final()]从检查和删除填充。

仅当输入数据的长度为
密码块大小的倍数。

这`decipher.setAutoPadding()`方法必须在之前调用
[`decipher.final()`][decipher.final()].

### `decipher.update(data[, inputEncoding][, outputEncoding])`

<!-- YAML
added: v0.1.94
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`data`字符串。
*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

更新破译`data`.如果`inputEncoding`参数被给出，
这`data`
参数是使用指定编码的字符串。如果`inputEncoding`
参数未给出，`data`必须是[`Buffer`][Buffer].如果`data`是一个
[`Buffer`][Buffer]然后`inputEncoding`被忽略。

这`outputEncoding`指定加密的输出格式
数据。如果`outputEncoding`
，则返回使用指定编码的字符串。如果不是
`outputEncoding`提供，一个[`Buffer`][Buffer]返回。

这`decipher.update()`可以使用新数据多次调用方法，直到
[`decipher.final()`][decipher.final()]被调用。叫`decipher.update()`后
[`decipher.final()`][decipher.final()]将导致引发错误。

## 类：`DiffieHellman`

<!-- YAML
added: v0.5.0
-->

这`DiffieHellman`类是用于创建Diffie-Hellman键的实用程序
交流。

的实例`DiffieHellman`类可以使用
[`crypto.createDiffieHellman()`][crypto.createDiffieHellman()]功能。

```mjs
import assert from 'node:assert';

const {
  createDiffieHellman
} = await import('node:crypto');

// Generate Alice's keys...
const alice = createDiffieHellman(2048);
const aliceKey = alice.generateKeys();

// Generate Bob's keys...
const bob = createDiffieHellman(alice.getPrime(), alice.getGenerator());
const bobKey = bob.generateKeys();

// Exchange and generate the secret...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

// OK
assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
```

```cjs
const assert = require('node:assert');

const {
  createDiffieHellman,
} = require('node:crypto');

// Generate Alice's keys...
const alice = createDiffieHellman(2048);
const aliceKey = alice.generateKeys();

// Generate Bob's keys...
const bob = createDiffieHellman(alice.getPrime(), alice.getGenerator());
const bobKey = bob.generateKeys();

// Exchange and generate the secret...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

// OK
assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
```

### `diffieHellman.computeSecret(otherPublicKey[, inputEncoding][, outputEncoding])`

<!-- YAML
added: v0.5.0
-->

*   `otherPublicKey`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`otherPublicKey`字符串。
*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

使用`otherPublicKey`作为另一个
参与方的公钥，并返回计算出的共享机密。提供的
键使用指定的`inputEncoding`，而秘密是
使用指定的编码`outputEncoding`.
如果`inputEncoding`莫
提供`otherPublicKey`预计是[`Buffer`][Buffer],
`TypedArray`或`DataView`.

如果`outputEncoding`给定一个字符串返回;否则，一个
[`Buffer`][Buffer]返回。

### `diffieHellman.generateKeys([encoding])`

<!-- YAML
added: v0.5.0
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

生成私有和公共 Diffie-Hellman 键值，并返回
指定中的公钥`encoding`.此密钥应为
转让给另一方。
如果`encoding`提供返回字符串;否则
[`Buffer`][Buffer]返回。

### `diffieHellman.getGenerator([encoding])`

<!-- YAML
added: v0.5.0
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

返回指定中的 Diffie-Hellman 生成器`encoding`.
如果`encoding`提供字符串为
返回;否则[`Buffer`][Buffer]返回。

### `diffieHellman.getPrime([encoding])`

<!-- YAML
added: v0.5.0
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

返回指定中的迪菲-赫尔曼素数`encoding`.
如果`encoding`提供字符串为
返回;否则[`Buffer`][Buffer]返回。

### `diffieHellman.getPrivateKey([encoding])`

<!-- YAML
added: v0.5.0
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

返回指定地址中的 Diffie-Hellman 私钥`encoding`.
如果`encoding`提供了一个
返回字符串;否则[`Buffer`][Buffer]返回。

### `diffieHellman.getPublicKey([encoding])`

<!-- YAML
added: v0.5.0
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

返回指定中的 Diffie-Hellman 公钥`encoding`.
如果`encoding`提供了一个
返回字符串;否则[`Buffer`][Buffer]返回。

### `diffieHellman.setPrivateKey(privateKey[, encoding])`

<!-- YAML
added: v0.5.0
-->

*   `privateKey`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`privateKey`字符串。

设置 Diffie-Hellman 私钥。如果`encoding`提供参数，
`privateKey`预计
成为字符串。如果不是`encoding`提供，`privateKey`预计
成为[`Buffer`][Buffer],`TypedArray`或`DataView`.

### `diffieHellman.setPublicKey(publicKey[, encoding])`

<!-- YAML
added: v0.5.0
-->

*   `publicKey`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`publicKey`字符串。

设置 Diffie-Hellman 公钥。如果`encoding`提供参数，
`publicKey`预计
成为字符串。如果不是`encoding`提供，`publicKey`预计
成为[`Buffer`][Buffer],`TypedArray`或`DataView`.

### `diffieHellman.verifyError`

<!-- YAML
added: v0.11.12
-->

包含由检查导致的任何警告和/或错误的位字段
在 初始化期间执行`DiffieHellman`对象。

以下值对此属性有效（如`node:constants`模块）：

*   `DH_CHECK_P_NOT_SAFE_PRIME`
*   `DH_CHECK_P_NOT_PRIME`
*   `DH_UNABLE_TO_CHECK_GENERATOR`
*   `DH_NOT_SUITABLE_GENERATOR`

## 类：`DiffieHellmanGroup`

<!-- YAML
added: v0.7.5
-->

这`DiffieHellmanGroup`class 将一个众所周知的 modp 组作为其参数。
它的工作原理与`DiffieHellman`，除非它不允许更改
创建后的密钥。换句话说，它不实现`setPublicKey()`
或`setPrivateKey()`方法。

```mjs
const { createDiffieHellmanGroup } = await import('node:crypto');
const dh = createDiffieHellmanGroup('modp1');
```

```cjs
const { createDiffieHellmanGroup } = require('node:crypto');
const dh = createDiffieHellmanGroup('modp1');
```

支持以下组：

*   `'modp1'`（768 位，[RFC 2409][]第6.1节）
*   `'modp2'`（1024 位，[RFC 2409][]第6.2节）
*   `'modp5'`（1536 位，[RFC 3526][]第2节）
*   `'modp14'`（2048 位，[RFC 3526][]第3节）
*   `'modp15'`（3072 位，[RFC 3526][]第4节）
*   `'modp16'`（4096 位，[RFC 3526][]第5节）
*   `'modp17'`（6144 位，[RFC 3526][]第6节）
*   `'modp18'`（8192 位，[RFC 3526][]第7节）

## 类：`ECDH`

<!-- YAML
added: v0.11.14
-->

这`ECDH`类是用于创建椭圆曲线 Diffie-Hellman （ECDH） 的实用程序
密钥交换。

的实例`ECDH`类可以使用
[`crypto.createECDH()`][crypto.createECDH()]功能。

```mjs
import assert from 'node:assert';

const {
  createECDH
} = await import('node:crypto');

// Generate Alice's keys...
const alice = createECDH('secp521r1');
const aliceKey = alice.generateKeys();

// Generate Bob's keys...
const bob = createECDH('secp521r1');
const bobKey = bob.generateKeys();

// Exchange and generate the secret...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
// OK
```

```cjs
const assert = require('node:assert');

const {
  createECDH,
} = require('node:crypto');

// Generate Alice's keys...
const alice = createECDH('secp521r1');
const aliceKey = alice.generateKeys();

// Generate Bob's keys...
const bob = createECDH('secp521r1');
const bobKey = bob.generateKeys();

// Exchange and generate the secret...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
// OK
```

### 静态方法：`ECDH.convertKey(key, curve[, inputEncoding[, outputEncoding[, format]]])`

<!-- YAML
added: v10.0.0
-->

*   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `curve`{字符串}
*   `inputEncoding`{字符串}这[编码][encoding]的`key`字符串。
*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   `format`{字符串}**违约：** `'uncompressed'`
*   返回：{缓冲区|字符串}

转换由`key`和`curve`到
由 指定的格式`format`.这`format`参数指定点编码
并且可以是`'compressed'`,`'uncompressed'`或`'hybrid'`.提供的密钥是
使用指定的`inputEncoding`，并对返回的密钥进行编码
使用指定的`outputEncoding`.

用[`crypto.getCurves()`][crypto.getCurves()]以获取可用曲线名称的列表。
在最近的OpenSSL版本中，`openssl ecparam -list_curves`还将显示
每个可用椭圆曲线的名称和描述。

如果`format`未指定点将在`'uncompressed'`
格式。

如果`inputEncoding`未提供，`key`预计是[`Buffer`][Buffer],
`TypedArray`或`DataView`.

示例（解压缩密钥）：

```mjs
const {
  createECDH,
  ECDH
} = await import('node:crypto');

const ecdh = createECDH('secp256k1');
ecdh.generateKeys();

const compressedKey = ecdh.getPublicKey('hex', 'compressed');

const uncompressedKey = ECDH.convertKey(compressedKey,
                                        'secp256k1',
                                        'hex',
                                        'hex',
                                        'uncompressed');

// The converted key and the uncompressed public key should be the same
console.log(uncompressedKey === ecdh.getPublicKey('hex'));
```

```cjs
const {
  createECDH,
  ECDH,
} = require('node:crypto');

const ecdh = createECDH('secp256k1');
ecdh.generateKeys();

const compressedKey = ecdh.getPublicKey('hex', 'compressed');

const uncompressedKey = ECDH.convertKey(compressedKey,
                                        'secp256k1',
                                        'hex',
                                        'hex',
                                        'uncompressed');

// The converted key and the uncompressed public key should be the same
console.log(uncompressedKey === ecdh.getPublicKey('hex'));
```

### `ecdh.computeSecret(otherPublicKey[, inputEncoding][, outputEncoding])`

<!-- YAML
added: v0.11.14
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16849
    description: Changed error format to better support invalid public key
                 error.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `otherPublicKey`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`otherPublicKey`字符串。
*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

使用`otherPublicKey`作为另一个
参与方的公钥，并返回计算出的共享机密。提供的
键使用指定的`inputEncoding`，以及返回的秘密
使用指定的编码`outputEncoding`.
如果`inputEncoding`莫
提供`otherPublicKey`预计是[`Buffer`][Buffer],`TypedArray`或
`DataView`.

如果`outputEncoding`给定一个字符串将被返回;否则
[`Buffer`][Buffer]返回。

`ecdh.computeSecret`将抛出一个
`ERR_CRYPTO_ECDH_INVALID_PUBLIC_KEY`错误时`otherPublicKey`
位于椭圆曲线之外。因为`otherPublicKey`是
通常由远程用户通过不安全的网络提供，
请务必相应地处理此异常。

### `ecdh.generateKeys([encoding[, format]])`

<!-- YAML
added: v0.11.14
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   `format`{字符串}**违约：** `'uncompressed'`
*   返回：{缓冲区|字符串}

生成私有和公有 EC Diffie-Hellman 键值，并返回
指定中的公钥`format`和`encoding`.此密钥应为
转让给另一方。

这`format`参数指定点编码，并且可以`'compressed'`或
`'uncompressed'`.如果`format`未指定，该点将在
`'uncompressed'`格式。

如果`encoding`提供返回字符串;否则[`Buffer`][Buffer]
返回。

### `ecdh.getPrivateKey([encoding])`

<!-- YAML
added: v0.11.14
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串} 指定的 EC Diffie-Hellman`encoding`.

如果`encoding`指定，返回字符串;否则[`Buffer`][Buffer]是
返回。

### `ecdh.getPublicKey([encoding][, format])`

<!-- YAML
added: v0.11.14
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   `format`{字符串}**违约：** `'uncompressed'`
*   返回：{缓冲区|字符串} 指定中的 EC Diffie-Hellman 公钥
    `encoding`和`format`.

这`format`参数指定点编码，并且可以`'compressed'`或
`'uncompressed'`.如果`format`未指定点将在
`'uncompressed'`格式。

如果`encoding`指定，返回字符串;否则[`Buffer`][Buffer]是
返回。

### `ecdh.setPrivateKey(privateKey[, encoding])`

<!-- YAML
added: v0.11.14
-->

*   `privateKey`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`privateKey`字符串。

设置 EC Diffie-Hellman 私钥。
如果`encoding`提供，`privateKey`预计
成为字符串;否则`privateKey`预计是[`Buffer`][Buffer],
`TypedArray`或`DataView`.

如果`privateKey`对于在`ECDH`对象是
已创建，则引发错误。设置私钥后，关联的
公共点（密钥）也在`ECDH`对象。

### `ecdh.setPublicKey(publicKey[, encoding])`

<!-- YAML
added: v0.11.14
deprecated: v5.2.0
-->

> 稳定性：0 - 已弃用

*   `publicKey`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}这[编码][encoding]的`publicKey`字符串。

设置 EC Diffie-Hellman 公钥。
如果`encoding`提供`publicKey`预计
是一个字符串;否则[`Buffer`][Buffer],`TypedArray`或`DataView`是预期的。

通常没有理由调用此方法，因为`ECDH`
只需要一个私钥和另一方的公钥来计算
共享密钥。通常[`ecdh.generateKeys()`][ecdh.generateKeys()]或
[`ecdh.setPrivateKey()`][ecdh.setPrivateKey()]将被调用。这[`ecdh.setPrivateKey()`][ecdh.setPrivateKey()]方法
尝试生成与私钥关联的公共点/密钥
设置。

示例（获取共享密钥）：

```mjs
const {
  createECDH,
  createHash
} = await import('node:crypto');

const alice = createECDH('secp256k1');
const bob = createECDH('secp256k1');

// This is a shortcut way of specifying one of Alice's previous private
// keys. It would be unwise to use such a predictable private key in a real
// application.
alice.setPrivateKey(
  createHash('sha256').update('alice', 'utf8').digest()
);

// Bob uses a newly generated cryptographically strong
// pseudorandom key pair
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

// aliceSecret and bobSecret should be the same shared secret value
console.log(aliceSecret === bobSecret);
```

```cjs
const {
  createECDH,
  createHash,
} = require('node:crypto');

const alice = createECDH('secp256k1');
const bob = createECDH('secp256k1');

// This is a shortcut way of specifying one of Alice's previous private
// keys. It would be unwise to use such a predictable private key in a real
// application.
alice.setPrivateKey(
  createHash('sha256').update('alice', 'utf8').digest()
);

// Bob uses a newly generated cryptographically strong
// pseudorandom key pair
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

// aliceSecret and bobSecret should be the same shared secret value
console.log(aliceSecret === bobSecret);
```

## 类：`Hash`

<!-- YAML
added: v0.1.92
-->

*   扩展：{流。转换}

这`Hash`class 是用于创建数据的哈希摘要的实用程序。它可以是
以下列两种方式之一使用：

*   作为[流][stream]既可读又可写，其中写入数据
    在可读端生成计算哈希摘要，或
*   使用[`hash.update()`][hash.update()]和[`hash.digest()`][hash.digest()]生成的方法
    计算哈希。

这[`crypto.createHash()`][crypto.createHash()]方法用于创建`Hash`实例。`Hash`
对象不能直接使用`new`关键词。

示例：使用`Hash`对象作为流：

```mjs
const {
  createHash
} = await import('node:crypto');

const hash = createHash('sha256');

hash.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = hash.read();
  if (data) {
    console.log(data.toString('hex'));
    // Prints:
    //   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
  }
});

hash.write('some data to hash');
hash.end();
```

```cjs
const {
  createHash,
} = require('node:crypto');

const hash = createHash('sha256');

hash.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = hash.read();
  if (data) {
    console.log(data.toString('hex'));
    // Prints:
    //   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
  }
});

hash.write('some data to hash');
hash.end();
```

示例：使用`Hash`和管道流：

```mjs
import { createReadStream } from 'node:fs';
import { stdout } from 'node:process';
const { createHash } = await import('node:crypto');

const hash = createHash('sha256');

const input = createReadStream('test.js');
input.pipe(hash).setEncoding('hex').pipe(stdout);
```

```cjs
const { createReadStream } = require('node:fs');
const { createHash } = require('node:crypto');
const { stdout } = require('node:process');

const hash = createHash('sha256');

const input = createReadStream('test.js');
input.pipe(hash).setEncoding('hex').pipe(stdout);
```

示例：使用[`hash.update()`][hash.update()]和[`hash.digest()`][hash.digest()]方法：

```mjs
const {
  createHash
} = await import('node:crypto');

const hash = createHash('sha256');

hash.update('some data to hash');
console.log(hash.digest('hex'));
// Prints:
//   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
```

```cjs
const {
  createHash,
} = require('node:crypto');

const hash = createHash('sha256');

hash.update('some data to hash');
console.log(hash.digest('hex'));
// Prints:
//   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
```

### `hash.copy([options])`

<!-- YAML
added: v13.1.0
-->

*   `options`{对象}[`stream.transform`选项][stream.transform options]
*   返回值：{哈希}

创建新的`Hash`包含内部状态的深层副本的对象
的电流`Hash`对象。

可选`options`参数控制流行为。对于 XOF 哈希
函数，例如`'shake256'`这`outputLength`选项可用于
指定所需的输出长度（以字节为单位）。

尝试复制`Hash`对象之后
其[`hash.digest()`][hash.digest()]方法已被调用。

```mjs
// Calculate a rolling hash.
const {
  createHash
} = await import('node:crypto');

const hash = createHash('sha256');

hash.update('one');
console.log(hash.copy().digest('hex'));

hash.update('two');
console.log(hash.copy().digest('hex'));

hash.update('three');
console.log(hash.copy().digest('hex'));

// Etc.
```

```cjs
// Calculate a rolling hash.
const {
  createHash,
} = require('node:crypto');

const hash = createHash('sha256');

hash.update('one');
console.log(hash.copy().digest('hex'));

hash.update('two');
console.log(hash.copy().digest('hex'));

hash.update('three');
console.log(hash.copy().digest('hex'));

// Etc.
```

### `hash.digest([encoding])`

<!-- YAML
added: v0.1.92
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

计算传递到要进行哈希处理的所有数据的摘要（使用
[`hash.update()`][hash.update()]方法）。
如果`encoding`提供将返回字符串;否则
一个[`Buffer`][Buffer]返回。

这`Hash`对象不能在以后再次使用`hash.digest()`方法已
叫。多个调用将导致引发错误。

### `hash.update(data[, inputEncoding])`

<!-- YAML
added: v0.1.92
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`data`字符串。

使用给定的更新哈希内容`data`，其编码
给出于`inputEncoding`.
如果`encoding`未提供，并且`data`是一个字符串，一个
编码`'utf8'`强制执行。如果`data`是一个[`Buffer`][Buffer],`TypedArray`或
`DataView`然后`inputEncoding`被忽略。

在流式传输新数据时，可以使用新数据多次调用它。

## 类：`Hmac`

<!-- YAML
added: v0.1.94
-->

*   扩展：{流。转换}

这`Hmac`class 是用于创建加密 HMAC 摘要的实用程序。它可以
以以下两种方式之一使用：

*   作为[流][stream]既可读又可写，其中写入数据
    在可读端生成计算的 HMAC 摘要，或
*   使用[`hmac.update()`][hmac.update()]和[`hmac.digest()`][hmac.digest()]生成的方法
    计算 HMAC 摘要。

这[`crypto.createHmac()`][crypto.createHmac()]方法用于创建`Hmac`实例。`Hmac`
对象不能直接使用`new`关键词。

示例：使用`Hmac`对象作为流：

```mjs
const {
  createHmac
} = await import('node:crypto');

const hmac = createHmac('sha256', 'a secret');

hmac.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = hmac.read();
  if (data) {
    console.log(data.toString('hex'));
    // Prints:
    //   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
  }
});

hmac.write('some data to hash');
hmac.end();
```

```cjs
const {
  createHmac,
} = require('node:crypto');

const hmac = createHmac('sha256', 'a secret');

hmac.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = hmac.read();
  if (data) {
    console.log(data.toString('hex'));
    // Prints:
    //   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
  }
});

hmac.write('some data to hash');
hmac.end();
```

示例：使用`Hmac`和管道流：

```mjs
import { createReadStream } from 'node:fs';
import { stdout } from 'node:process';
const {
  createHmac
} = await import('node:crypto');

const hmac = createHmac('sha256', 'a secret');

const input = createReadStream('test.js');
input.pipe(hmac).pipe(stdout);
```

```cjs
const {
  createReadStream,
} = require('node:fs');
const {
  createHmac,
} = require('node:crypto');
const { stdout } = require('node:process');

const hmac = createHmac('sha256', 'a secret');

const input = createReadStream('test.js');
input.pipe(hmac).pipe(stdout);
```

示例：使用[`hmac.update()`][hmac.update()]和[`hmac.digest()`][hmac.digest()]方法：

```mjs
const {
  createHmac
} = await import('node:crypto');

const hmac = createHmac('sha256', 'a secret');

hmac.update('some data to hash');
console.log(hmac.digest('hex'));
// Prints:
//   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
```

```cjs
const {
  createHmac,
} = require('node:crypto');

const hmac = createHmac('sha256', 'a secret');

hmac.update('some data to hash');
console.log(hmac.digest('hex'));
// Prints:
//   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
```

### `hmac.digest([encoding])`

<!-- YAML
added: v0.1.94
-->

*   `encoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

计算使用 方法传递的所有数据的 HMAC 摘要[`hmac.update()`][hmac.update()].
如果`encoding`是
前提是返回字符串;否则[`Buffer`][Buffer]返回;

这`Hmac`对象不能在以后再次使用`hmac.digest()`已经
叫。多次调用`hmac.digest()`将导致引发错误。

### `hmac.update(data[, inputEncoding])`

<!-- YAML
added: v0.1.94
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`data`字符串。

更新`Hmac`给定的内容`data`，其编码
给出于`inputEncoding`.
如果`encoding`未提供，并且`data`是一个字符串，一个
编码`'utf8'`强制执行。如果`data`是一个[`Buffer`][Buffer],`TypedArray`或
`DataView`然后`inputEncoding`被忽略。

在流式传输新数据时，可以使用新数据多次调用它。

## 类：`KeyObject`

<!-- YAML
added: v11.6.0
changes:
  - version:
    - v14.5.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33360
    description: Instances of this class can now be passed to worker threads
                 using `postMessage`.
  - version: v11.13.0
    pr-url: https://github.com/nodejs/node/pull/26438
    description: This class is now exported.
-->

节点.js使用`KeyObject`类来表示对称或非对称密钥，
并且每种键都公开了不同的功能。这
[`crypto.createSecretKey()`][crypto.createSecretKey()],[`crypto.createPublicKey()`][crypto.createPublicKey()]和
[`crypto.createPrivateKey()`][crypto.createPrivateKey()]方法用于创建`KeyObject`
实例。`KeyObject`对象不能直接使用`new`
关键词。

大多数应用程序应考虑使用新的`KeyObject`使用 API 而不是
将键作为字符串传递，或`Buffer`s 由于改进了安全功能。

`KeyObject`实例可以通过以下方式传递到其他线程[`postMessage()`][postMessage()].
接收器获得克隆`KeyObject`，以及`KeyObject`不需要
列在`transferList`论点。

### 静态方法：`KeyObject.from(key)`

<!-- YAML
added: v15.0.0
-->

*   `key`{CryptoKey}
*   返回：{键对象}

示例：转换`CryptoKey`实例到`KeyObject`:

```mjs
const { webcrypto, KeyObject } = await import('node:crypto');
const { subtle } = webcrypto;

const key = await subtle.generateKey({
  name: 'HMAC',
  hash: 'SHA-256',
  length: 256
}, true, ['sign', 'verify']);

const keyObject = KeyObject.from(key);
console.log(keyObject.symmetricKeySize);
// Prints: 32 (symmetric key size in bytes)
```

```cjs
const {
  webcrypto: {
    subtle,
  },
  KeyObject,
} = require('node:crypto');

(async function() {
  const key = await subtle.generateKey({
    name: 'HMAC',
    hash: 'SHA-256',
    length: 256
  }, true, ['sign', 'verify']);

  const keyObject = KeyObject.from(key);
  console.log(keyObject.symmetricKeySize);
  // Prints: 32 (symmetric key size in bytes)
})();
```

### `keyObject.asymmetricKeyDetails`

<!-- YAML
added: v15.7.0
changes:
  - version: v16.9.0
    pr-url: https://github.com/nodejs/node/pull/39851
    description: Expose `RSASSA-PSS-params` sequence parameters
                 for RSA-PSS keys.
-->

*   {对象}
    *   `modulusLength`：{数字} 密钥大小（以位为单位）（RSA、DSA）。
    *   `publicExponent`： {bigint} Public exponent （RSA）.
    *   `hashAlgorithm`：{字符串} 消息摘要的名称 （RSA-PSS）。
    *   `mgf1HashAlgorithm`：{字符串} 消息摘要的名称
        MGF1 （RSA-PSS）.
    *   `saltLength`：{数字} 最小盐长度（以字节为单位） （RSA-PSS）。
    *   `divisorLength`： {数字} 大小`q`以位为单位 （DSA）。
    *   `namedCurve`：{字符串} 曲线名称 （EC）。

此属性仅存在于非对称密钥上。根据密钥的类型，
此对象包含有关密钥的信息。未获得任何信息
通过此属性可用于唯一标识密钥或危害
密钥的安全性。

对于 RSA-PSS 密钥，如果密钥材料包含`RSASSA-PSS-params`序列
这`hashAlgorithm`,`mgf1HashAlgorithm`和`saltLength`属性将是
设置。

其他关键详细信息可能通过此 API 使用其他属性公开。

### `keyObject.asymmetricKeyType`

<!-- YAML
added: v11.6.0
changes:
  - version:
     - v13.9.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31178
    description: Added support for `'dh'`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26960
    description: Added support for `'rsa-pss'`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26786
    description: This property now returns `undefined` for KeyObject
                 instances of unrecognized type instead of aborting.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26774
    description: Added support for `'x25519'` and `'x448'`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26319
    description: Added support for `'ed25519'` and `'ed448'`.
-->

*   {字符串}

对于非对称密钥，此属性表示密钥的类型。支持的密钥
类型有：

*   `'rsa'`（OID 1.2.840.113549.1.1.1）
*   `'rsa-pss'`（OID 1.2.840.113549.1.1.10）
*   `'dsa'`（OID 1.2.840.10040.4.1）
*   `'ec'`（OID 1.2.840.10045.2.1）
*   `'x25519'`（OID 1.3.101.110）
*   `'x448'`（OID 1.3.101.111）
*   `'ed25519'`（OID 1.3.101.112）
*   `'ed448'`（OID 1.3.101.113）
*   `'dh'`（OID 1.2.840.113549.1.3.1）

此属性是`undefined`对于未识别`KeyObject`类型和对称
钥匙。

### `keyObject.export([options])`

<!-- YAML
added: v11.6.0
changes:
  - version: v15.9.0
    pr-url: https://github.com/nodejs/node/pull/37081
    description: Added support for `'jwk'` format.
-->

*   `options`： {对象}
*   返回：{字符串|缓冲区|对象}

对于对称密钥，可以使用以下编码选项：

*   `format`：{字符串} 必须`'buffer'`（默认值）或`'jwk'`.

对于公钥，可以使用以下编码选项：

*   `type`：{字符串} 必须是`'pkcs1'`（仅限 RSA）或`'spki'`.
*   `format`：{字符串} 必须`'pem'`,`'der'`或`'jwk'`.

对于私钥，可以使用以下编码选项：

*   `type`：{字符串} 必须是`'pkcs1'`（仅限 RSA），`'pkcs8'`或
    `'sec1'`（仅限欧共体）。
*   `format`：{字符串} 必须`'pem'`,`'der'`或`'jwk'`.
*   `cipher`：{字符串} 如果指定，私钥将加密
    给定的`cipher`和`passphrase`使用基于 PKCS#5 v2.0 密码
    加密。
*   `passphrase`：{字符串|缓冲区} 用于加密的密码，请参阅
    `cipher`.

结果类型取决于所选的编码格式，当 PEM
结果是一个字符串，当DER时，它将是一个包含数据的缓冲区
编码为 DER，当[断续器][JWK]它将是一个对象。

什么时候[断续器][JWK]已选择编码格式，所有其他编码选项均为
忽视。

PKCS#1、SEC1 和 PKCS#8 类型的密钥可以通过使用以下各项的组合进行加密：
这`cipher`和`format`选项。The PKCS#8`type`可与任何
`format`通过指定
`cipher`.PKCS#1 和 SEC1 只能通过指定`cipher`
当PEM`format`使用。为了获得最大的兼容性，请使用 PKCS#8
加密的私钥。由于PKCS#8定义了自己的
加密机制，加密时不支持 PEM 级加密
一个 PKCS#8 密钥。看[RFC 5208][]用于 PKCS#8 加密和[RFC 1421][]为
PKCS#1 和 SEC1 加密。

### `keyObject.equals(otherKeyObject)`

<!-- YAML
added:
  - v17.7.0
  - v16.15.0
-->

*   `otherKeyObject`： {KeyObject} A`KeyObject`用它
    比较`keyObject`.
*   返回：{布尔值}

返回`true`或`false`取决于密钥是否完全相同
类型、值和参数。此方法不是
[恒定时间](https://en.wikipedia.org/wiki/Timing_attack).

### `keyObject.symmetricKeySize`

<!-- YAML
added: v11.6.0
-->

*   {数字}

对于密钥，此属性表示密钥的大小（以字节为单位）。这
属性是`undefined`用于非对称密钥。

### `keyObject.type`

<!-- YAML
added: v11.6.0
-->

*   {字符串}

取决于此类型`KeyObject`，则此属性为
`'secret'`对于秘密（对称）密钥，`'public'`对于公共（非对称）密钥
或`'private'`用于私有（非对称）密钥。

## 类：`Sign`

<!-- YAML
added: v0.1.92
-->

*   扩展：{流。可写}

这`Sign`类是用于生成签名的实用程序。它可以用于一个
的两种方式：

*   作为可写[流][stream]，其中写入要签名的数据，并且
    [`sign.sign()`][sign.sign()]方法用于生成和返回签名，或者
*   使用[`sign.update()`][sign.update()]和[`sign.sign()`][sign.sign()]生成的方法
    签名。

这[`crypto.createSign()`][crypto.createSign()]方法用于创建`Sign`实例。这
参数是要使用的哈希函数的字符串名称。`Sign`对象不是
直接使用`new`关键词。

示例：使用`Sign`和[`Verify`][Verify]对象作为流：

```mjs
const {
  generateKeyPairSync,
  createSign,
  createVerify
} = await import('node:crypto');

const { privateKey, publicKey } = generateKeyPairSync('ec', {
  namedCurve: 'sect239k1'
});

const sign = createSign('SHA256');
sign.write('some data to sign');
sign.end();
const signature = sign.sign(privateKey, 'hex');

const verify = createVerify('SHA256');
verify.write('some data to sign');
verify.end();
console.log(verify.verify(publicKey, signature, 'hex'));
// Prints: true
```

```cjs
const {
  generateKeyPairSync,
  createSign,
  createVerify,
} = require('node:crypto');

const { privateKey, publicKey } = generateKeyPairSync('ec', {
  namedCurve: 'sect239k1'
});

const sign = createSign('SHA256');
sign.write('some data to sign');
sign.end();
const signature = sign.sign(privateKey, 'hex');

const verify = createVerify('SHA256');
verify.write('some data to sign');
verify.end();
console.log(verify.verify(publicKey, signature, 'hex'));
// Prints: true
```

示例：使用[`sign.update()`][sign.update()]和[`verify.update()`][verify.update()]方法：

```mjs
const {
  generateKeyPairSync,
  createSign,
  createVerify
} = await import('node:crypto');

const { privateKey, publicKey } = generateKeyPairSync('rsa', {
  modulusLength: 2048,
});

const sign = createSign('SHA256');
sign.update('some data to sign');
sign.end();
const signature = sign.sign(privateKey);

const verify = createVerify('SHA256');
verify.update('some data to sign');
verify.end();
console.log(verify.verify(publicKey, signature));
// Prints: true
```

```cjs
const {
  generateKeyPairSync,
  createSign,
  createVerify,
} = require('node:crypto');

const { privateKey, publicKey } = generateKeyPairSync('rsa', {
  modulusLength: 2048,
});

const sign = createSign('SHA256');
sign.update('some data to sign');
sign.end();
const signature = sign.sign(privateKey);

const verify = createVerify('SHA256');
verify.update('some data to sign');
verify.end();
console.log(verify.verify(publicKey, signature));
// Prints: true
```

### `sign.sign(privateKey[, outputEncoding])`

<!-- YAML
added: v0.1.92
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The privateKey can also be an ArrayBuffer and CryptoKey.
  - version:
     - v13.2.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/29292
    description: This function now supports IEEE-P1363 DSA and ECDSA signatures.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26960
    description: This function now supports RSA-PSS keys.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: This function now supports key objects.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11705
    description: Support for RSASSA-PSS and additional options was added.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `privateKey`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
    *   `dsaEncoding`{字符串}
    *   `padding`{整数}
    *   `saltLength`{整数}
*   `outputEncoding`{字符串}这[编码][encoding]的返回值。
*   返回：{缓冲区|字符串}

<!--lint enable maximum-line-length remark-lint-->

计算通过的所有数据的签名，使用
[`sign.update()`][sign.update()]或[`sign.write()`][stream-writable-write].

如果`privateKey`不是[`KeyObject`][KeyObject]，则此函数的行为就像
`privateKey`已传递到[`crypto.createPrivateKey()`][crypto.createPrivateKey()].如果是
对象，则可以传递以下附加属性：

*   `dsaEncoding`{字符串}对于 DSA 和 ECDSA，此选项指定
    生成的签名的格式。它可以是以下之一：
    *   `'der'`（默认值）：DER 编码的 ASN.1 签名结构编码`(r, s)`.
    *   `'ieee-p1363'`：签名格式`r || s`如IEEE-P1363中所建议。
*   `padding`{整数}RSA 的可选填充值，如下所示之一：

    *   `crypto.constants.RSA_PKCS1_PADDING`（默认值）
    *   `crypto.constants.RSA_PKCS1_PSS_PADDING`

    `RSA_PKCS1_PSS_PADDING`将使用具有相同哈希函数的 MGF1
    用于按照 第 3.1 节中指定的消息进行签名[RFC 4055][]除非
    MGF1 哈希函数已被指定为密钥的一部分，符合
    第 3.3 节[RFC 4055][].
*   `saltLength`{整数}填充时的盐长度为
    `RSA_PKCS1_PSS_PADDING`.特殊值
    `crypto.constants.RSA_PSS_SALTLEN_DIGEST`将盐的长度设置为消化液
    大小`crypto.constants.RSA_PSS_SALTLEN_MAX_SIGN`（默认值）将其设置为
    最大允许值。

如果`outputEncoding`提供返回字符串;否则[`Buffer`][Buffer]
返回。

这`Sign`对象不能在以后再次使用`sign.sign()`方法已
叫。多次调用`sign.sign()`将导致引发错误。

### `sign.update(data[, inputEncoding])`

<!-- YAML
added: v0.1.92
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`data`字符串。

更新`Sign`给定的内容`data`，其编码
给出于`inputEncoding`.
如果`encoding`未提供，并且`data`是一个字符串，一个
编码`'utf8'`强制执行。如果`data`是一个[`Buffer`][Buffer],`TypedArray`或
`DataView`然后`inputEncoding`被忽略。

在流式传输新数据时，可以使用新数据多次调用它。

## 类：`Verify`

<!-- YAML
added: v0.1.92
-->

*   扩展：{流。可写}

这`Verify`类是用于验证签名的实用程序。它可以用于一个
的两种方式：

*   作为可写[流][stream]其中，写入的数据用于根据
    提供的签名，或
*   使用[`verify.update()`][verify.update()]和[`verify.verify()`][verify.verify()]验证方法
    签名。

这[`crypto.createVerify()`][crypto.createVerify()]方法用于创建`Verify`实例。
`Verify`对象不能直接使用`new`关键词。

看[`Sign`][Sign]例如。

### `verify.update(data[, inputEncoding])`

<!-- YAML
added: v0.1.92
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

*   `data`{字符串|缓冲区|TypedArray|数据视图}
*   `inputEncoding`{字符串}这[编码][encoding]的`data`字符串。

更新`Verify`给定的内容`data`，其编码
给出于`inputEncoding`.
如果`inputEncoding`未提供，并且`data`是一个字符串，一个
编码`'utf8'`强制执行。如果`data`是一个[`Buffer`][Buffer],`TypedArray`或
`DataView`然后`inputEncoding`被忽略。

在流式传输新数据时，可以使用新数据多次调用它。

### `verify.verify(object, signature[, signatureEncoding])`

<!-- YAML
added: v0.1.92
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The object can also be an ArrayBuffer and CryptoKey.
  - version:
     - v13.2.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/29292
    description: This function now supports IEEE-P1363 DSA and ECDSA signatures.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26960
    description: This function now supports RSA-PSS keys.
  - version: v11.7.0
    pr-url: https://github.com/nodejs/node/pull/25217
    description: The key can now be a private key.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11705
    description: Support for RSASSA-PSS and additional options was added.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `object`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
    *   `dsaEncoding`{字符串}
    *   `padding`{整数}
    *   `saltLength`{整数}
*   `signature`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `signatureEncoding`{字符串}这[编码][encoding]的`signature`字符串。
*   返回：{布尔值}`true`或`false`取决于的有效性
    数据和公钥的签名。

<!--lint enable maximum-line-length remark-lint-->

使用给定的数据验证提供的数据`object`和`signature`.

如果`object`不是[`KeyObject`][KeyObject]，则此函数的行为就像
`object`已传递到[`crypto.createPublicKey()`][crypto.createPublicKey()].如果是
对象，则可以传递以下附加属性：

*   `dsaEncoding`{字符串}对于 DSA 和 ECDSA，此选项指定
    签名的格式。它可以是以下之一：
    *   `'der'`（默认值）：DER 编码的 ASN.1 签名结构编码`(r, s)`.
    *   `'ieee-p1363'`：签名格式`r || s`如IEEE-P1363中所建议。
*   `padding`{整数}RSA 的可选填充值，如下所示之一：

    *   `crypto.constants.RSA_PKCS1_PADDING`（默认值）
    *   `crypto.constants.RSA_PKCS1_PSS_PADDING`

    `RSA_PKCS1_PSS_PADDING`将使用具有相同哈希函数的 MGF1
    用于验证在 的 3.1 节中指定的消息[RFC 4055][]除非
    MGF1 哈希函数已被指定为密钥的一部分，符合
    第 3.3 节[RFC 4055][].
*   `saltLength`{整数}填充时的盐长度为
    `RSA_PKCS1_PSS_PADDING`.特殊值
    `crypto.constants.RSA_PSS_SALTLEN_DIGEST`将盐的长度设置为消化液
    大小`crypto.constants.RSA_PSS_SALTLEN_AUTO`（默认值）导致它是
    自动确定。

这`signature`参数是以前计算的数据签名，在
这`signatureEncoding`.
如果`signatureEncoding`指定，`signature`预计是
字符串;否则`signature`预计是[`Buffer`][Buffer],
`TypedArray`或`DataView`.

这`verify`对象不能在以后再次使用`verify.verify()`已经
叫。多次调用`verify.verify()`将导致错误
扔。

由于公钥可以从私钥派生，因此私钥可以
而不是公钥。

## 类：`X509Certificate`

<!-- YAML
added: v15.6.0
-->

封装 X509 证书并提供只读访问权限
其信息。

```mjs
const { X509Certificate } = await import('node:crypto');

const x509 = new X509Certificate('{... pem encoded cert ...}');

console.log(x509.subject);
```

```cjs
const { X509Certificate } = require('node:crypto');

const x509 = new X509Certificate('{... pem encoded cert ...}');

console.log(x509.subject);
```

### `new X509Certificate(buffer)`

<!-- YAML
added: v15.6.0
-->

*   `buffer`{字符串|TypedArray|缓冲区|DataView} PEM 或 DER 编码
    X509 证书。

### `x509.ca`

<!-- YAML
added: v15.6.0
-->

*   类型： {布尔值} 将是`true`如果这是证书颁发机构 （CA）
    证书。

### `x509.checkEmail(email[, options])`

<!-- YAML
added: v15.6.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41600
    description: The subject option now defaults to `'default'`.
  - version:
      - v17.5.0
      - v16.14.1
    pr-url: https://github.com/nodejs/node/pull/41599
    description: The `wildcards`, `partialWildcards`, `multiLabelWildcards`, and
                 `singleLabelSubdomains` options have been removed since they
                 had no effect.
  - version:
    - v17.5.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41569
    description: The subject option can now be set to `'default'`.
-->

*   `email`{字符串}
*   `options`{对象}
    *   `subject`{字符串}`'default'`,`'always'`或`'never'`.
        **违约：** `'default'`.
*   返回：{字符串|未定义} 返回`email`如果证书匹配，
    `undefined`如果没有。

检查证书是否与给定的电子邮件地址匹配。

如果`'subject'`选项未定义或设置为`'default'`、证书
仅当使用者备用文件扩展名执行以下任一情况时，才考虑使用者
不存在或不包含任何电子邮件地址。

如果`'subject'`选项设置为`'always'`如果主题替代
文件扩展名不存在或不包含匹配的电子邮件
地址，则考虑证书主体。

如果`'subject'`选项设置为`'never'`，则证书使用者从不
考虑，即使证书不包含使用者替代名称。

### `x509.checkHost(name[, options])`

<!-- YAML
added: v15.6.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41600
    description: The subject option now defaults to `'default'`.
  - version:
    - v17.5.0
    - v16.15.0
    pr-url: https://github.com/nodejs/node/pull/41569
    description: The subject option can now be set to `'default'`.
-->

*   `name`{字符串}
*   `options`{对象}
    *   `subject`{字符串}`'default'`,`'always'`或`'never'`.
        **违约：** `'default'`.
    *   `wildcards`{布尔值}**违约：** `true`.
    *   `partialWildcards`{布尔值}**违约：** `true`.
    *   `multiLabelWildcards`{布尔值}**违约：** `false`.
    *   `singleLabelSubdomains`{布尔值}**违约：** `false`.
*   返回：{字符串|未定义} 返回与`name`,
    或`undefined`如果没有匹配的使用者名称`name`.

检查证书是否与给定的主机名匹配。

如果证书与给定的主机名匹配，则匹配的使用者名称为
返回。返回的名称可能完全匹配（例如，`foo.example.com`)
或者它可能包含通配符（例如，`*.example.com`).因为主机名
比较不区分大小写，返回的使用者名称也可能不同
从给定`name`大写。

如果`'subject'`选项未定义或设置为`'default'`、证书
仅当使用者备用文件扩展名执行以下任一情况时，才考虑使用者
不存在或不包含任何 DNS 名称。此行为符合
[RFC 2818][]（“HTTP over TLS”）。

如果`'subject'`选项设置为`'always'`如果主题替代
名称扩展不存在或不包含匹配的 DNS 名称，
考虑证书主题。

如果`'subject'`选项设置为`'never'`，则证书使用者从不
考虑，即使证书不包含使用者替代名称。

### `x509.checkIP(ip)`

<!-- YAML
added: v15.6.0
changes:
  - version:
      - v17.5.0
      - v16.14.1
    pr-url: https://github.com/nodejs/node/pull/41571
    description: The `options` argument has been removed since it had no effect.
-->

*   `ip`{字符串}
*   返回：{字符串|未定义} 返回`ip`如果证书匹配，
    `undefined`如果没有。

检查证书是否与给定的 IP 地址（IPv4 或 IPv6）匹配。

只[RFC 5280][] `iPAddress`考虑主题替代名称，并且它们
必须与给定的匹配`ip`地址完全正确。其他主题替代名称
以及证书的主题字段将被忽略。

### `x509.checkIssued(otherCert)`

<!-- YAML
added: v15.6.0
-->

*   `otherCert`{X509证书}
*   返回：{布尔值}

检查此证书是否由给定的颁发`otherCert`.

### `x509.checkPrivateKey(privateKey)`

<!-- YAML
added: v15.6.0
-->

*   `privateKey`{键对象}私钥。
*   返回：{布尔值}

检查此证书的公钥是否与
给定的私钥。

### `x509.fingerprint`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书的 SHA-1 指纹。

因为 SHA-1 在加密方面被破坏了，并且因为 SHA-1 的安全性
明显比通常用于签名的算法差得多
证书，请考虑使用[`x509.fingerprint256`][x509.fingerprint256]相反。

### `x509.fingerprint256`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书的 SHA-256 指纹。

### `x509.fingerprint512`

<!-- YAML
added:
  - v17.2.0
  - v16.14.0
-->

*   类型： {字符串}

此证书的 SHA-512 指纹。

因为计算SHA-256指纹通常更快，而且因为它是
只有SHA-512指纹的一半大小，[`x509.fingerprint256`][x509.fingerprint256]可能
一个更好的选择。虽然SHA-512可能在以下方面提供了更高级别的安全性
通常，SHA-256的安全性与大多数算法的安全性相匹配
通常用于对证书进行签名。

### `x509.infoAccess`

<!-- YAML
added: v15.6.0
changes:
  - version:
      - v17.3.1
      - v16.13.2
    pr-url: https://github.com/nodejs-private/node-private/pull/300
    description: Parts of this string may be encoded as JSON string literals
                 in response to CVE-2021-44532.
-->

*   类型： {字符串}

证书颁发机构信息访问的文本表示形式
外延。

这是一个以换行符分隔的访问说明列表。每行以 开头
访问方法和访问位置的类型，后跟冒号和
与访问位置关联的值。

在表示访问方法和访问位置类型的前缀之后，
每行的其余部分可以括在引号中，以指示
值是 JSON 字符串文本。为了向后兼容，Node.js 仅使用
此属性中的 JSON 字符串文本（如有必要）以避免歧义。
应准备第三方代码来处理这两种可能的输入格式。

### `x509.issuer`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书中包含的颁发者标识。

### `x509.issuerCertificate`

<!-- YAML
added: v15.9.0
-->

*   类型： {X509证书}

颁发者证书或`undefined`如果颁发者证书不是
可用。

### `x509.keyUsage`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串\[]}

详细介绍此证书的密钥用法的数组。

### `x509.publicKey`

<!-- YAML
added: v15.6.0
-->

*   类型： {键对象}

此证书的公钥 {KeyObject}。

### `x509.raw`

<!-- YAML
added: v15.6.0
-->

*   类型： {缓冲区}

一个`Buffer`包含此证书的 DER 编码。

### `x509.serialNumber`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书的序列号。

序列号由证书颁发机构分配，而不是唯一的
标识证书。考虑使用[`x509.fingerprint256`][x509.fingerprint256]作为一个独特的
而是标识符。

### `x509.subject`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书的完整主题。

### `x509.subjectAltName`

<!-- YAML
added: v15.6.0
changes:
  - version:
      - v17.3.1
      - v16.13.2
    pr-url: https://github.com/nodejs-private/node-private/pull/300
    description: Parts of this string may be encoded as JSON string literals
                 in response to CVE-2021-44532.
-->

*   类型： {字符串}

为此证书指定的使用者备用名称。

这是一个以逗号分隔的使用者备用名称列表。每个条目开始
带有一个字符串，标识使用者替代名称的类型，后跟
冒号和与条目关联的值。

早期版本的 Node.js错误地认为拆分此内容是安全的
属性位于双字符序列中`', '`（请参见[CVE-2021-44532][]).然而
恶意证书和合法证书都可以包含使用者备用名称
当表示为字符串时，包含此序列。

在表示条目类型的前缀之后，每个条目的其余部分
可能用引号括起来，以指示该值是 JSON 字符串文本。
为了向后兼容，Node.js 仅在此中使用 JSON 字符串文本
属性（如有必要，以避免歧义）。应准备第三方代码
以处理两种可能的输入格式。

### `x509.toJSON()`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

X509 证书没有标准的 JSON 编码。这
`toJSON()`方法返回包含 PEM 编码的字符串
证书。

### `x509.toLegacyObject()`

<!-- YAML
added: v15.6.0
-->

*   类型： {对象}

使用旧版返回有关此证书的信息
[证书对象][certificate object]编码。

### `x509.toString()`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

返回 PEM 编码的证书。

### `x509.validFrom`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书被视为有效的日期/时间。

### `x509.validTo`

<!-- YAML
added: v15.6.0
-->

*   类型： {字符串}

此证书被视为有效的日期/时间。

### `x509.verify(publicKey)`

<!-- YAML
added: v15.6.0
-->

*   `publicKey`{键对象}公钥。
*   返回：{布尔值}

验证此证书是否由给定的公钥签名。
不对证书执行任何其他验证检查。

## `node:crypto`模块方法和属性

### `crypto.constants`

<!-- YAML
added: v6.3.0
-->

*   返回：{对象} 包含加密和
    与安全相关的操作。当前定义的特定常量是
    描述于[加密常量][Crypto constants].

### `crypto.DEFAULT_ENCODING`

<!-- YAML
added: v0.9.3
deprecated: v10.0.0
-->

> 稳定性：0 - 已弃用

用于可以采用任一字符串的函数的默认编码
或[缓冲区][`Buffer`].默认值为`'buffer'`，这就产生了方法
默认为[`Buffer`][Buffer]对象。

这`crypto.DEFAULT_ENCODING`提供机制以实现向后兼容性
使用期望的遗留程序`'latin1'`作为默认编码。

新应用程序应预期默认值为`'buffer'`.

此属性已弃用。

### `crypto.fips`

<!-- YAML
added: v6.0.0
deprecated: v10.0.0
-->

> 稳定性：0 - 已弃用

用于检查和控制是否符合 FIPS 的加密提供程序的属性
当前正在使用中。设置为 true 需要 Node.js 的 FIPS 内部版本。

此属性已弃用。请使用`crypto.setFips()`和
`crypto.getFips()`相反。

### `crypto.checkPrime(candidate[, options[, callback]])`

<!-- YAML
added: v15.8.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `candidate`{ArrayBuffer|SharedArrayBuffer|TypedArray|缓冲区|DataView|bigint}
    编码为任意大端八位字节序序列的可能素数
    长度。
*   `options`{对象}
    *   `checks`{数字}米勒-拉宾概率素数
        要执行的迭代。当值为`0`（零），多次检查
        用于产生最多 2 的误报率<sup>-64</sup>为
        随机输入。选择多个检查时必须小心。指
        到 OpenSSL 文档，了解[`BN_is_prime_ex`][BN_is_prime_ex]功能`nchecks`
        选项以获取更多详细信息。**违约：** `0`
*   `callback`{函数}
    *   `err`{错误}如果在检查期间发生错误，则设置为 {Error} 对象。
    *   `result`{布尔值}`true`如果候选者是有错误的素数
        概率小于`0.25 ** options.checks`.

检查`candidate`.

### `crypto.checkPrimeSync(candidate[, options])`

<!-- YAML
added: v15.8.0
-->

*   `candidate`{ArrayBuffer|SharedArrayBuffer|TypedArray|缓冲区|DataView|bigint}
    编码为任意大端八位字节序序列的可能素数
    长度。
*   `options`{对象}
    *   `checks`{数字}米勒-拉宾概率素数
        要执行的迭代。当值为`0`（零），多次检查
        用于产生最多 2 的误报率<sup>-64</sup>为
        随机输入。选择多个检查时必须小心。指
        到 OpenSSL 文档，了解[`BN_is_prime_ex`][BN_is_prime_ex]功能`nchecks`
        选项以获取更多详细信息。**违约：** `0`
*   返回：{布尔值}`true`如果候选者是有错误的素数
    概率小于`0.25 ** options.checks`.

检查`candidate`.

### `crypto.createCipher(algorithm, password[, options])`

<!-- YAML
added: v0.1.94
deprecated: v10.0.0
changes:
  - version: v17.9.0
    pr-url: https://github.com/nodejs/node/pull/42427
    description: The `authTagLength` option is now optional when using the
                 `chacha20-poly1305` cipher and defaults to 16 bytes.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The password argument can be an ArrayBuffer and is limited to
                 a maximum of 2 ** 31 - 1 bytes.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/21447
    description: Ciphers in OCB mode are now supported.
  - version: v10.2.0
    pr-url: https://github.com/nodejs/node/pull/20235
    description: The `authTagLength` option can now be used to produce shorter
                 authentication tags in GCM mode and defaults to 16 bytes.
-->

> 稳定性：0 - 已弃用：使用[`crypto.createCipheriv()`][crypto.createCipheriv()]相反。

*   `algorithm`{字符串}
*   `password`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
*   返回： {密码}

创建并返回`Cipher`使用给定对象`algorithm`和
`password`.

这`options`参数控制流行为，并且是可选的，除非
CCM 或 OCB 模式下的密码（例如`'aes-128-ccm'`） 被使用。在这种情况下，
`authTagLength`选项是必需的，并指定
身份验证标记（以字节为单位），请参阅[断续器模式][CCM mode].在 GCM 模式下，`authTagLength`
选项不是必需的，但可用于设置身份验证的长度
标记中，该标记将由`getAuthTag()`，默认值为 16 个字节。
为`chacha20-poly1305`这`authTagLength`选项默认为 16 个字节。

这`algorithm`依赖于 OpenSSL，例如`'aes192'`等。上
最近的OpenSSL版本，`openssl list -cipher-algorithms`将
显示可用的密码算法。

这`password`用于派生密码密钥和初始化向量 （IV）。
该值必须为`'latin1'`编码字符串，一个[`Buffer`][Buffer]一个
`TypedArray`，或`DataView`.

实施`crypto.createCipher()`使用 OpenSSL 派生密钥
功能[`EVP_BytesToKey`][EVP_BytesToKey]摘要算法设置为 MD5 时，一个
迭代，没有盐。缺乏盐允许字典攻击相同
密码始终创建相同的密钥。低迭代次数和
非加密安全的哈希算法允许非常测试密码
迅速地。

根据OpenSSL的建议，使用更现代的算法而不是
[`EVP_BytesToKey`][EVP_BytesToKey]建议开发人员派生一个密钥，并在
自己的使用[`crypto.scrypt()`][crypto.scrypt()]并使用[`crypto.createCipheriv()`][crypto.createCipheriv()]
以创建`Cipher`对象。用户不应将密码与计数器模式结合使用
（例如 CTR、GCM 或 CCM）在`crypto.createCipher()`.在以下情况下发出警告：
使用它们是为了避免IV重复使用的风险，导致
漏洞。对于在GCM中重复使用IV的情况，请参见[不尊重
对手][Nonce-Disrespecting
Adversaries]了解详情。

### `crypto.createCipheriv(algorithm, key, iv[, options])`

<!-- YAML
added: v0.1.94
changes:
  - version: v17.9.0
    pr-url: https://github.com/nodejs/node/pull/42427
    description: The `authTagLength` option is now optional when using the
                 `chacha20-poly1305` cipher and defaults to 16 bytes.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The password and iv arguments can be an ArrayBuffer and are
                 each limited to a maximum of 2 ** 31 - 1 bytes.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: The `key` argument can now be a `KeyObject`.
  - version:
     - v11.2.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/24081
    description: The cipher `chacha20-poly1305` (the IETF variant of
                 ChaCha20-Poly1305) is now supported.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/21447
    description: Ciphers in OCB mode are now supported.
  - version: v10.2.0
    pr-url: https://github.com/nodejs/node/pull/20235
    description: The `authTagLength` option can now be used to produce shorter
                 authentication tags in GCM mode and defaults to 16 bytes.
  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/18644
    description: The `iv` parameter may now be `null` for ciphers which do not
                 need an initialization vector.
-->

*   `algorithm`{字符串}
*   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
*   `iv`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView|null}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
*   返回： {密码}

创建并返回`Cipher`对象，具有给定的`algorithm`,`key`和
初始化向量 （`iv`).

这`options`参数控制流行为，并且是可选的，除非
CCM 或 OCB 模式下的密码（例如`'aes-128-ccm'`） 被使用。在这种情况下，
`authTagLength`选项是必需的，并指定
身份验证标记（以字节为单位），请参阅[断续器模式][CCM mode].在 GCM 模式下，`authTagLength`
选项不是必需的，但可用于设置身份验证的长度
标记中，该标记将由`getAuthTag()`，默认值为 16 个字节。
为`chacha20-poly1305`这`authTagLength`选项默认为 16 个字节。

这`algorithm`依赖于 OpenSSL，例如`'aes192'`等。上
最近的OpenSSL版本，`openssl list -cipher-algorithms`将
显示可用的密码算法。

这`key`是`algorithm`和`iv`是一个
[初始化向量][initialization vector].两个参数都必须是`'utf8'`编码字符串，
[缓冲区][`Buffer`],`TypedArray`或`DataView`s.这`key`可选
一个[`KeyObject`][KeyObject]类型`secret`.如果密码不需要
初始化向量，`iv`可能`null`.

传递 字符串时`key`或`iv`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

初始化向量应该是不可预测和唯一的;理想情况下，它们将是
加密随机。它们不必是秘密的：IV通常只是
添加到未加密的密文消息中。这听起来可能自相矛盾
有些东西必须是不可预测和独特的，但不一定是秘密的;
请记住，攻击者一定无法提前预测
给定 IV 将是。

### `crypto.createDecipher(algorithm, password[, options])`

<!-- YAML
added: v0.1.94
deprecated: v10.0.0
changes:
  - version: v17.9.0
    pr-url: https://github.com/nodejs/node/pull/42427
    description: The `authTagLength` option is now optional when using the
                 `chacha20-poly1305` cipher and defaults to 16 bytes.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/21447
    description: Ciphers in OCB mode are now supported.
-->

> 稳定性：0 - 已弃用：使用[`crypto.createDecipheriv()`][crypto.createDecipheriv()]相反。

*   `algorithm`{字符串}
*   `password`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
*   返回：{解密}

创建并返回`Decipher`使用给定对象`algorithm`和
`password`（键）。

这`options`参数控制流行为，并且是可选的，除非
CCM 或 OCB 模式下的密码（例如`'aes-128-ccm'`） 被使用。在这种情况下，
`authTagLength`选项是必需的，并指定
身份验证标记（以字节为单位），请参阅[断续器模式][CCM mode].
为`chacha20-poly1305`这`authTagLength`选项默认为 16 个字节。

实施`crypto.createDecipher()`使用 OpenSSL 派生密钥
功能[`EVP_BytesToKey`][EVP_BytesToKey]摘要算法设置为 MD5 时，一个
迭代，没有盐。缺乏盐允许字典攻击相同
密码始终创建相同的密钥。低迭代次数和
非加密安全的哈希算法允许非常测试密码
迅速地。

根据OpenSSL的建议，使用更现代的算法而不是
[`EVP_BytesToKey`][EVP_BytesToKey]建议开发人员派生一个密钥，并在
自己的使用[`crypto.scrypt()`][crypto.scrypt()]并使用[`crypto.createDecipheriv()`][crypto.createDecipheriv()]
以创建`Decipher`对象。

### `crypto.createDecipheriv(algorithm, key, iv[, options])`

<!-- YAML
added: v0.1.94
changes:
  - version: v17.9.0
    pr-url: https://github.com/nodejs/node/pull/42427
    description: The `authTagLength` option is now optional when using the
                 `chacha20-poly1305` cipher and defaults to 16 bytes.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: The `key` argument can now be a `KeyObject`.
  - version:
     - v11.2.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/24081
    description: The cipher `chacha20-poly1305` (the IETF variant of
                 ChaCha20-Poly1305) is now supported.
  - version: v10.10.0
    pr-url: https://github.com/nodejs/node/pull/21447
    description: Ciphers in OCB mode are now supported.
  - version: v10.2.0
    pr-url: https://github.com/nodejs/node/pull/20039
    description: The `authTagLength` option can now be used to restrict accepted
                 GCM authentication tag lengths.
  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/18644
    description: The `iv` parameter may now be `null` for ciphers which do not
                 need an initialization vector.
-->

*   `algorithm`{字符串}
*   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
*   `iv`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView|null}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
*   返回：{解密}

创建并返回`Decipher`使用给定对象`algorithm`,`key`
和初始化向量 （`iv`).

这`options`参数控制流行为，并且是可选的，除非
CCM 或 OCB 模式下的密码（例如`'aes-128-ccm'`） 被使用。在这种情况下，
`authTagLength`选项是必需的，并指定
身份验证标记（以字节为单位），请参阅[断续器模式][CCM mode].在 GCM 模式下，`authTagLength`
选项不是必需的，但可用于限制接受的身份验证标记
到具有指定长度的那些。
为`chacha20-poly1305`这`authTagLength`选项默认为 16 个字节。

这`algorithm`依赖于 OpenSSL，例如`'aes192'`等。上
最近的OpenSSL版本，`openssl list -cipher-algorithms`将
显示可用的密码算法。

这`key`是`algorithm`和`iv`是一个
[初始化向量][initialization vector].两个参数都必须是`'utf8'`编码字符串，
[缓冲区][`Buffer`],`TypedArray`或`DataView`s.这`key`可选
一个[`KeyObject`][KeyObject]类型`secret`.如果密码不需要
初始化向量，`iv`可能`null`.

传递 字符串时`key`或`iv`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

初始化向量应该是不可预测和唯一的;理想情况下，它们将是
加密随机。它们不必是秘密的：IV通常只是
添加到未加密的密文消息中。这听起来可能自相矛盾
有些东西必须是不可预测和独特的，但不一定是秘密的;
请记住，攻击者一定无法提前预测给定的内容
四会的。

### `crypto.createDiffieHellman(prime[, primeEncoding][, generator][, generatorEncoding])`

<!-- YAML
added: v0.11.12
changes:
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `prime` argument can be any `TypedArray` or `DataView` now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11983
    description: The `prime` argument can be a `Uint8Array` now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default for the encoding parameters changed
                 from `binary` to `utf8`.
-->

*   `prime`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `primeEncoding`{字符串}这[编码][encoding]的`prime`字符串。
*   `generator`{数字|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
    **违约：** `2`
*   `generatorEncoding`{字符串}这[编码][encoding]的`generator`字符串。
*   Returns： {DiffieHellman}

创建`DiffieHellman`密钥交换对象使用提供的`prime`和
可选特定`generator`.

这`generator`参数可以是数字、字符串或[`Buffer`][Buffer].如果
`generator`未指定，值`2`使用。

如果`primeEncoding`指定，`prime`应为字符串;否则
一个[`Buffer`][Buffer],`TypedArray`或`DataView`是预期的。

如果`generatorEncoding`指定，`generator`应为字符串;
否则为数字，[`Buffer`][Buffer],`TypedArray`或`DataView`是预期的。

### `crypto.createDiffieHellman(primeLength[, generator])`

<!-- YAML
added: v0.5.0
-->

*   `primeLength`{数字}
*   `generator`{数字}**违约：** `2`
*   Returns： {DiffieHellman}

创建`DiffieHellman`密钥交换对象并生成素数
`primeLength`使用可选特定数字的位`generator`.
如果`generator`未指定，值`2`使用。

### `crypto.createDiffieHellmanGroup(name)`

<!-- YAML
added: v0.9.3
-->

*   `name`{字符串}
*   Returns： {DiffieHellmanGroup}

的别名[`crypto.getDiffieHellman()`][crypto.getDiffieHellman()]

### `crypto.createECDH(curveName)`

<!-- YAML
added: v0.11.14
-->

*   `curveName`{字符串}
*   返回： {ECDH}

创建椭圆曲线 Diffie-Hellman （`ECDH`） 密钥交换对象使用
由`curveName`字符串。用
[`crypto.getCurves()`][crypto.getCurves()]以获取可用曲线名称的列表。关于最近
OpenSSL 发布，`openssl ecparam -list_curves`还会显示名称
和每个可用椭圆曲线的描述。

### `crypto.createHash(algorithm[, options])`

<!-- YAML
added: v0.1.92
changes:
  - version: v12.8.0
    pr-url: https://github.com/nodejs/node/pull/28805
    description: The `outputLength` option was added for XOF hash functions.
-->

*   `algorithm`{字符串}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
*   返回值：{哈希}

创建并返回`Hash`可用于生成哈希摘要的对象
使用给定的`algorithm`.自选`options`参数控件流
行为。对于 XOF 哈希函数，例如`'shake256'`这`outputLength`选择
可用于指定所需的输出长度（以字节为单位）。

这`algorithm`取决于
平台上OpenSSL的版本。例如`'sha256'`,`'sha512'`等。
在最近发布的OpenSSL上，`openssl list -digest-algorithms`将
显示可用的摘要算法。

示例：生成文件的 sha256 总和

```mjs
import {
  createReadStream
} from 'node:fs';
import { argv } from 'node:process';
const {
  createHash
} = await import('node:crypto');

const filename = argv[2];

const hash = createHash('sha256');

const input = createReadStream(filename);
input.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = input.read();
  if (data)
    hash.update(data);
  else {
    console.log(`${hash.digest('hex')} ${filename}`);
  }
});
```

```cjs
const {
  createReadStream,
} = require('node:fs');
const {
  createHash,
} = require('node:crypto');
const { argv } = require('node:process');

const filename = argv[2];

const hash = createHash('sha256');

const input = createReadStream(filename);
input.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = input.read();
  if (data)
    hash.update(data);
  else {
    console.log(`${hash.digest('hex')} ${filename}`);
  }
});
```

### `crypto.createHmac(algorithm, key[, options])`

<!-- YAML
added: v0.1.94
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The key can also be an ArrayBuffer or CryptoKey. The
                 encoding option was added. The key cannot contain
                 more than 2 ** 32 - 1 bytes.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: The `key` argument can now be a `KeyObject`.
-->

*   `algorithm`{字符串}
*   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
*   `options`{对象}[`stream.transform`选项][stream.transform options]
    *   `encoding`{字符串}在以下情况下要使用的字符串编码`key`是一个字符串。
*   返回： {Hmac}

创建并返回`Hmac`使用给定对象`algorithm`和`key`.
自选`options`参数控制流行为。

这`algorithm`取决于
平台上OpenSSL的版本。例如`'sha256'`,`'sha512'`等。
在最近发布的OpenSSL上，`openssl list -digest-algorithms`将
显示可用的摘要算法。

这`key`是用于生成加密 HMAC 哈希的 HMAC 密钥。如果是
一个[`KeyObject`][KeyObject]，其类型必须为`secret`.

示例：生成文件的 sha256 HMAC

```mjs
import {
  createReadStream
} from 'node:fs';
import { argv } from 'node:process';
const {
  createHmac
} = await import('node:crypto');

const filename = argv[2];

const hmac = createHmac('sha256', 'a secret');

const input = createReadStream(filename);
input.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = input.read();
  if (data)
    hmac.update(data);
  else {
    console.log(`${hmac.digest('hex')} ${filename}`);
  }
});
```

```cjs
const {
  createReadStream,
} = require('node:fs');
const {
  createHmac,
} = require('node:crypto');
const { argv } = require('node:process');

const filename = argv[2];

const hmac = createHmac('sha256', 'a secret');

const input = createReadStream(filename);
input.on('readable', () => {
  // Only one element is going to be produced by the
  // hash stream.
  const data = input.read();
  if (data)
    hmac.update(data);
  else {
    console.log(`${hmac.digest('hex')} ${filename}`);
  }
});
```

### `crypto.createPrivateKey(key)`

<!-- YAML
added: v11.6.0
changes:
  - version: v15.12.0
    pr-url: https://github.com/nodejs/node/pull/37254
    description: The key can also be a JWK object.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The key can also be an ArrayBuffer. The encoding option was
                 added. The key cannot contain more than 2 ** 32 - 1 bytes.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `key`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
    *   `key`：{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|对象} 键
        材料，采用 PEM、DER 或 JWK 格式。
    *   `format`：{字符串} 必须`'pem'`,`'der'`，或 '`'jwk'`.
        **违约：** `'pem'`.
    *   `type`：{字符串} 必须`'pkcs1'`,`'pkcs8'`或`'sec1'`.此选项是
        仅当`format`是`'der'`否则忽略。
    *   `passphrase`：{字符串|缓冲区} 用于解密的密码。
    *   `encoding`：{字符串} 在以下情况下要使用的字符串编码`key`是一个字符串。
*   返回：{键对象}

<!--lint enable maximum-line-length remark-lint-->

创建并返回包含私钥的新密钥对象。如果`key`是一个
字符串或`Buffer`,`format`假定为`'pem'`;否则`key`
必须是具有上述属性的对象。

如果私钥已加密，则`passphrase`必须指定。长度
的密码限制为 1024 字节。

### `crypto.createPublicKey(key)`

<!-- YAML
added: v11.6.0
changes:
  - version: v15.12.0
    pr-url: https://github.com/nodejs/node/pull/37254
    description: The key can also be a JWK object.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The key can also be an ArrayBuffer. The encoding option was
                 added. The key cannot contain more than 2 ** 32 - 1 bytes.
  - version: v11.13.0
    pr-url: https://github.com/nodejs/node/pull/26278
    description: The `key` argument can now be a `KeyObject` with type
                 `private`.
  - version: v11.7.0
    pr-url: https://github.com/nodejs/node/pull/25217
    description: The `key` argument can now be a private key.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `key`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
    *   `key`：{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|对象} 键
        材料，采用 PEM、DER 或 JWK 格式。
    *   `format`：{字符串} 必须`'pem'`,`'der'`或`'jwk'`.
        **违约：** `'pem'`.
    *   `type`：{字符串} 必须`'pkcs1'`或`'spki'`.此选项是
        仅当`format`是`'der'`否则忽略。
    *   `encoding`{字符串}在以下情况下要使用的字符串编码`key`是一个字符串。
*   返回：{键对象}

<!--lint enable maximum-line-length remark-lint-->

创建并返回包含公钥的新密钥对象。如果`key`是一个
字符串或`Buffer`,`format`假定为`'pem'`;如果`key`是一个`KeyObject`
带类型`'private'`，公钥派生自给定的私钥;
否则`key`必须是具有上述属性的对象。

如果格式为`'pem'`这`'key'`也可能是 X.509 证书。

由于公钥可以从私钥派生，因此私钥可能是
已传递而不是公钥。在这种情况下，此函数的行为就像
[`crypto.createPrivateKey()`][crypto.createPrivateKey()]已被调用，除了的类型
返回`KeyObject`将是`'public'`并且私钥不能是
从返回的中提取`KeyObject`.同样，如果`KeyObject`带类型
`'private'`给出了，一个新的`KeyObject`带类型`'public'`将被退回
并且不可能从返回的对象中提取私钥。

### `crypto.createSecretKey(key[, encoding])`

<!-- YAML
added: v11.6.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The key can also be an ArrayBuffer or string. The encoding
                 argument was added. The key cannot contain more than
                 2 ** 32 - 1 bytes.
-->

*   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `encoding`{字符串}字符串编码时`key`是一个字符串。
*   返回：{键对象}

创建并返回一个新的密钥对象，其中包含用于对称的密钥
加密或`Hmac`.

### `crypto.createSign(algorithm[, options])`

<!-- YAML
added: v0.1.92
-->

*   `algorithm`{字符串}
*   `options`{对象}[`stream.Writable`选项][stream.Writable options]
*   返回： {符号}

创建并返回`Sign`使用给定对象`algorithm`.用
[`crypto.getHashes()`][crypto.getHashes()]以获取可用摘要算法的名称。
自选`options`参数控制`stream.Writable`行为。

在某些情况下，一个`Sign`可以使用签名的名称创建实例
算法，例如`'RSA-SHA256'`，而不是摘要算法。这将使用
相应的摘要算法。这不适用于所有签名
算法，例如`'ecdsa-with-SHA256'`，所以最好始终使用摘要
算法名称。

### `crypto.createVerify(algorithm[, options])`

<!-- YAML
added: v0.1.92
-->

*   `algorithm`{字符串}
*   `options`{对象}[`stream.Writable`选项][stream.Writable options]
*   返回：{验证}

创建并返回`Verify`使用给定算法的对象。
用[`crypto.getHashes()`][crypto.getHashes()]以获取可用名称的数组
签名算法。自选`options`参数控制
`stream.Writable`行为。

在某些情况下，一个`Verify`可以使用签名的名称创建实例
算法，例如`'RSA-SHA256'`，而不是摘要算法。这将使用
相应的摘要算法。这不适用于所有签名
算法，例如`'ecdsa-with-SHA256'`，所以最好始终使用摘要
算法名称。

### `crypto.diffieHellman(options)`

<!-- YAML
added:
 - v13.9.0
 - v12.17.0
-->

*   `options`： {对象}
    *   `privateKey`： {键对象}
    *   `publicKey`： {键对象}
*   返回：{缓冲区}

计算迪菲-赫尔曼的秘密基于`privateKey`和`publicKey`.
两个键必须具有相同的`asymmetricKeyType`，它必须是`'dh'`
（对于迪菲-赫尔曼），`'ec'`（对于幼儿发展部），`'x448'`或`'x25519'`（适用于 ECDH-ES）。

### `crypto.generateKey(type, options, callback)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `type`：{字符串} 生成的密钥的预期用途。现在
    接受的值为`'hmac'`和`'aes'`.
*   `options`： {对象}
    *   `length`：{数字} 要生成的密钥的位长度。这必须是
        值大于 0。
        *   如果`type`是`'hmac'`，最小值为 8，最大长度为
            2<sup>31</sup>-1.如果该值不是 8 的倍数，则生成的
            键将被截断为`Math.floor(length / 8)`.
        *   如果`type`是`'aes'`，则长度必须为之一`128`,`192`或`256`.
*   `callback`： {函数}
    *   `err`：{错误}
    *   `key`： {键对象}

异步生成给定的新随机密钥`length`.这
`type`将确定将在`length`.

```mjs
const {
  generateKey
} = await import('node:crypto');

generateKey('hmac', { length: 64 }, (err, key) => {
  if (err) throw err;
  console.log(key.export().toString('hex'));  // 46e..........620
});
```

```cjs
const {
  generateKey,
} = require('node:crypto');

generateKey('hmac', { length: 64 }, (err, key) => {
  if (err) throw err;
  console.log(key.export().toString('hex'));  // 46e..........620
});
```

### `crypto.generateKeyPair(type, options, callback)`

<!-- YAML
added: v10.12.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/39927
    description: Add ability to define `RSASSA-PSS-params` sequence parameters
                 for RSA-PSS keys pairs.
  - version:
     - v13.9.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31178
    description: Add support for Diffie-Hellman.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26960
    description: Add support for RSA-PSS key pairs.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26774
    description: Add ability to generate X25519 and X448 key pairs.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26554
    description: Add ability to generate Ed25519 and Ed448 key pairs.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: The `generateKeyPair` and `generateKeyPairSync` functions now
                 produce key objects if no encoding was specified.
-->

*   `type`：{字符串} 必须`'rsa'`,`'rsa-pss'`,`'dsa'`,`'ec'`,`'ed25519'`,
    `'ed448'`,`'x25519'`,`'x448'`或`'dh'`.
*   `options`： {对象}
    *   `modulusLength`：{数字} 密钥大小（以位为单位）（RSA、DSA）。
    *   `publicExponent`：{数字} 公共指数 （RSA）。**违约：** `0x10001`.
    *   `hashAlgorithm`：{字符串} 消息摘要的名称 （RSA-PSS）。
    *   `mgf1HashAlgorithm`：{字符串} 消息摘要的名称
        MGF1 （RSA-PSS）.
    *   `saltLength`：{数字} 最小盐长度（以字节为单位） （RSA-PSS）。
    *   `divisorLength`： {数字} 大小`q`以位为单位 （DSA）。
    *   `namedCurve`：{字符串} 要使用的曲线的名称 （EC）。
    *   `prime`：{缓冲区} 质数参数 （DH）。
    *   `primeLength`：{数字} 以位为单位的质数长度 （DH）。
    *   `generator`：{数字} 自定义生成器 （DH）。**违约：** `2`.
    *   `groupName`：{string} Diffie-Hellman group name （DH）.看
        [`crypto.getDiffieHellman()`][crypto.getDiffieHellman()].
    *   `publicKeyEncoding`： {对象} 查看[`keyObject.export()`][keyObject.export()].
    *   `privateKeyEncoding`： {对象} 查看[`keyObject.export()`][keyObject.export()].
*   `callback`： {函数}
    *   `err`：{错误}
    *   `publicKey`：{字符串|缓冲区|键对象}
    *   `privateKey`：{字符串|缓冲区|键对象}

生成给定的新非对称密钥对`type`.RSA， RSA-PSS， DSA， EC，
目前支持 Ed25519、Ed448、X25519、X448 和 DH。

如果`publicKeyEncoding`或`privateKeyEncoding`已指定，此函数
表现得好像[`keyObject.export()`][keyObject.export()]有人呼吁其结果。否则
密钥的相应部分作为[`KeyObject`][KeyObject].

建议将公钥编码为`'spki'`和私钥作为
`'pkcs8'`使用加密进行长期存储：

```mjs
const {
  generateKeyPair
} = await import('node:crypto');

generateKeyPair('rsa', {
  modulusLength: 4096,
  publicKeyEncoding: {
    type: 'spki',
    format: 'pem'
  },
  privateKeyEncoding: {
    type: 'pkcs8',
    format: 'pem',
    cipher: 'aes-256-cbc',
    passphrase: 'top secret'
  }
}, (err, publicKey, privateKey) => {
  // Handle errors and use the generated key pair.
});
```

```cjs
const {
  generateKeyPair,
} = require('node:crypto');

generateKeyPair('rsa', {
  modulusLength: 4096,
  publicKeyEncoding: {
    type: 'spki',
    format: 'pem'
  },
  privateKeyEncoding: {
    type: 'pkcs8',
    format: 'pem',
    cipher: 'aes-256-cbc',
    passphrase: 'top secret'
  }
}, (err, publicKey, privateKey) => {
  // Handle errors and use the generated key pair.
});
```

完成后，`callback`将调用`err`设置为`undefined`和
`publicKey`/`privateKey`表示生成的密钥对。

如果调用此方法作为其[`util.promisify()`][util.promisify()]ed 版本，它返回
一个`Promise`对于`Object`跟`publicKey`和`privateKey`性能。

### `crypto.generateKeyPairSync(type, options)`

<!-- YAML
added: v10.12.0
changes:
  - version: v16.10.0
    pr-url: https://github.com/nodejs/node/pull/39927
    description: Add ability to define `RSASSA-PSS-params` sequence parameters
                 for RSA-PSS keys pairs.
  - version:
     - v13.9.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/31178
    description: Add support for Diffie-Hellman.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26960
    description: Add support for RSA-PSS key pairs.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26774
    description: Add ability to generate X25519 and X448 key pairs.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26554
    description: Add ability to generate Ed25519 and Ed448 key pairs.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: The `generateKeyPair` and `generateKeyPairSync` functions now
                 produce key objects if no encoding was specified.
-->

*   `type`：{字符串} 必须`'rsa'`,`'rsa-pss'`,`'dsa'`,`'ec'`,`'ed25519'`,
    `'ed448'`,`'x25519'`,`'x448'`或`'dh'`.
*   `options`： {对象}
    *   `modulusLength`：{数字} 密钥大小（以位为单位）（RSA、DSA）。
    *   `publicExponent`：{数字} 公共指数 （RSA）。**违约：** `0x10001`.
    *   `hashAlgorithm`：{字符串} 消息摘要的名称 （RSA-PSS）。
    *   `mgf1HashAlgorithm`：{字符串} 消息摘要的名称
        MGF1 （RSA-PSS）.
    *   `saltLength`：{数字} 最小盐长度（以字节为单位） （RSA-PSS）。
    *   `divisorLength`： {数字} 大小`q`以位为单位 （DSA）。
    *   `namedCurve`：{字符串} 要使用的曲线的名称 （EC）。
    *   `prime`：{缓冲区} 质数参数 （DH）。
    *   `primeLength`：{数字} 以位为单位的质数长度 （DH）。
    *   `generator`：{数字} 自定义生成器 （DH）。**违约：** `2`.
    *   `groupName`：{string} Diffie-Hellman group name （DH）.看
        [`crypto.getDiffieHellman()`][crypto.getDiffieHellman()].
    *   `publicKeyEncoding`： {对象} 查看[`keyObject.export()`][keyObject.export()].
    *   `privateKeyEncoding`： {对象} 查看[`keyObject.export()`][keyObject.export()].
*   返回： {对象}
    *   `publicKey`：{字符串|缓冲区|键对象}
    *   `privateKey`：{字符串|缓冲区|键对象}

生成给定的新非对称密钥对`type`.RSA， RSA-PSS， DSA， EC，
目前支持 Ed25519、Ed448、X25519、X448 和 DH。

如果`publicKeyEncoding`或`privateKeyEncoding`已指定，此函数
表现得好像[`keyObject.export()`][keyObject.export()]有人呼吁其结果。否则
密钥的相应部分作为[`KeyObject`][KeyObject].

编码公钥时，建议使用`'spki'`.编码时
私钥，建议使用`'pkcs8'`使用强密码，
并对密码保密。

```mjs
const {
  generateKeyPairSync
} = await import('node:crypto');

const {
  publicKey,
  privateKey,
} = generateKeyPairSync('rsa', {
  modulusLength: 4096,
  publicKeyEncoding: {
    type: 'spki',
    format: 'pem'
  },
  privateKeyEncoding: {
    type: 'pkcs8',
    format: 'pem',
    cipher: 'aes-256-cbc',
    passphrase: 'top secret'
  }
});
```

```cjs
const {
  generateKeyPairSync,
} = require('node:crypto');

const {
  publicKey,
  privateKey,
} = generateKeyPairSync('rsa', {
  modulusLength: 4096,
  publicKeyEncoding: {
    type: 'spki',
    format: 'pem'
  },
  privateKeyEncoding: {
    type: 'pkcs8',
    format: 'pem',
    cipher: 'aes-256-cbc',
    passphrase: 'top secret'
  }
});
```

返回值`{ publicKey, privateKey }`表示生成的密钥对。
选择 PEM 编码时，相应的键将为字符串，否则
它将是一个包含编码为DER的数据的缓冲区。

### `crypto.generateKeySync(type, options)`

<!-- YAML
added: v15.0.0
-->

*   `type`：{字符串} 生成的密钥的预期用途。现在
    接受的值为`'hmac'`和`'aes'`.
*   `options`： {对象}
    *   `length`：{数字} 要生成的密钥的位长度。
        *   如果`type`是`'hmac'`，最小值为 8，最大长度为
            2<sup>31</sup>-1.如果该值不是 8 的倍数，则生成的
            键将被截断为`Math.floor(length / 8)`.
        *   如果`type`是`'aes'`，则长度必须为之一`128`,`192`或`256`.
*   返回：{键对象}

同步生成给定的新随机密钥`length`.这
`type`将确定将在`length`.

```mjs
const {
  generateKeySync
} = await import('node:crypto');

const key = generateKeySync('hmac', { length: 64 });
console.log(key.export().toString('hex'));  // e89..........41e
```

```cjs
const {
  generateKeySync,
} = require('node:crypto');

const key = generateKeySync('hmac', { length: 64 });
console.log(key.export().toString('hex'));  // e89..........41e
```

### `crypto.generatePrime(size[, options[, callback]])`

<!-- YAML
added: v15.8.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `size`{数字}要生成的素数的大小（以位为单位）。
*   `options`{对象}
    *   `add`{ArrayBuffer|SharedArrayBuffer|TypedArray|缓冲区|DataView|bigint}
    *   `rem`{ArrayBuffer|SharedArrayBuffer|TypedArray|缓冲区|DataView|bigint}
    *   `safe`{布尔值}**违约：** `false`.
    *   `bigint`{布尔值}什么时候`true`，则返回生成的素数
        奥斯纳`bigint`.
*   `callback`{函数}
    *   `err`{错误}
    *   `prime`{ArrayBuffer|bigint}

生成伪随机素数`size`位。

如果`options.safe`是`true`，素数将是一个安全的素数 -- 也就是说，
`(prime - 1) / 2`也将是一个质数。

这`options.add`和`options.rem`参数可用于强制执行其他
要求，例如，对于 Diffie-Hellman：

*   如果`options.add`和`options.rem`两者都设置，素数将满足
    条件是`prime % add = rem`.
*   苟`options.add`已设置和`options.safe`莫`true`，质数意志
    满足以下条件：`prime % add = 1`.
*   苟`options.add`已设置和`options.safe`设置为`true`，素数
    而是将满足以下条件：`prime % add = 3`.这是必要的
    因为`prime % add = 1`为`options.add > 2`将与条件相矛盾
    执行者`options.safe`.
*   `options.rem`在以下情况下被忽略：`options.add`未给出。

双`options.add`和`options.rem`必须编码为大端序列
如果作为`ArrayBuffer`,`SharedArrayBuffer`,`TypedArray`,`Buffer`或
`DataView`.

默认情况下，素数被编码为八位字节的大端序列
在 {ArrayBuffer} 中。如果`bigint`选项是`true`，然后是 {bigint}
提供。

### `crypto.generatePrimeSync(size[, options])`

<!-- YAML
added: v15.8.0
-->

*   `size`{数字}要生成的素数的大小（以位为单位）。
*   `options`{对象}
    *   `add`{ArrayBuffer|SharedArrayBuffer|TypedArray|缓冲区|DataView|bigint}
    *   `rem`{ArrayBuffer|SharedArrayBuffer|TypedArray|缓冲区|DataView|bigint}
    *   `safe`{布尔值}**违约：** `false`.
    *   `bigint`{布尔值}什么时候`true`，则返回生成的素数
        奥斯纳`bigint`.
*   返回： {ArrayBuffer|bigint}

生成伪随机素数`size`位。

如果`options.safe`是`true`，素数将是一个安全的素数 -- 也就是说，
`(prime - 1) / 2`也将是一个质数。

这`options.add`和`options.rem`参数可用于强制执行其他
要求，例如，对于 Diffie-Hellman：

*   如果`options.add`和`options.rem`两者都设置，素数将满足
    条件是`prime % add = rem`.
*   苟`options.add`已设置和`options.safe`莫`true`，质数意志
    满足以下条件：`prime % add = 1`.
*   苟`options.add`已设置和`options.safe`设置为`true`，素数
    而是将满足以下条件：`prime % add = 3`.这是必要的
    因为`prime % add = 1`为`options.add > 2`将与条件相矛盾
    执行者`options.safe`.
*   `options.rem`在以下情况下被忽略：`options.add`未给出。

双`options.add`和`options.rem`必须编码为大端序列
如果作为`ArrayBuffer`,`SharedArrayBuffer`,`TypedArray`,`Buffer`或
`DataView`.

默认情况下，素数被编码为八位字节的大端序列
在 {ArrayBuffer} 中。如果`bigint`选项是`true`，然后是 {bigint}
提供。

### `crypto.getCipherInfo(nameOrNid[, options])`

<!-- YAML
added: v15.0.0
-->

*   `nameOrNid`：{字符串|编号} 要查询的密码的名称或 nid。
*   `options`： {对象}
    *   `keyLength`：{数字} 测试密钥长度。
    *   `ivLength`：{数字} A 测试 IV 长度。
*   返回： {对象}
    *   `name`{字符串}密码的名称
    *   `nid`{数字}密码的 nid
    *   `blockSize`{数字}密码的块大小（以字节为单位）。此属性
        在以下情况下省略`mode`是`'stream'`.
    *   `ivLength`{数字}中的预期或默认初始化向量长度
        字节。如果密码不使用初始化，则省略此属性
        向量。
    *   `keyLength`{数字}预期或默认密钥长度（以字节为单位）。
    *   `mode`{字符串}密码模式。其中之一`'cbc'`,`'ccm'`,`'cfb'`,`'ctr'`,
        `'ecb'`,`'gcm'`,`'ocb'`,`'ofb'`,`'stream'`,`'wrap'`,`'xts'`.

返回有关给定密码的信息。

某些密码接受可变长度键和初始化向量。默认情况下，
这`crypto.getCipherInfo()`方法将返回这些的默认值
密码。测试给定的密钥长度或 iv 长度对于给定的密钥长度是否可接受
密码，使用`keyLength`和`ivLength`选项。如果给定值为
无法接受`undefined`将被退回。

### `crypto.getCiphers()`

<!-- YAML
added: v0.9.3
-->

*   返回：{string\[]} 具有受支持密码名称的数组
    算法。

```mjs
const {
  getCiphers
} = await import('node:crypto');

console.log(getCiphers()); // ['aes-128-cbc', 'aes-128-ccm', ...]
```

```cjs
const {
  getCiphers,
} = require('node:crypto');

console.log(getCiphers()); // ['aes-128-cbc', 'aes-128-ccm', ...]
```

### `crypto.getCurves()`

<!-- YAML
added: v2.3.0
-->

*   返回：{string\[]} 一个数组，其中包含支持的椭圆曲线的名称。

```mjs
const {
  getCurves
} = await import('node:crypto');

console.log(getCurves()); // ['Oakley-EC2N-3', 'Oakley-EC2N-4', ...]
```

```cjs
const {
  getCurves,
} = require('node:crypto');

console.log(getCurves()); // ['Oakley-EC2N-3', 'Oakley-EC2N-4', ...]
```

### `crypto.getDiffieHellman(groupName)`

<!-- YAML
added: v0.7.5
-->

*   `groupName`{字符串}
*   Returns： {DiffieHellmanGroup}

创建预定义的`DiffieHellmanGroup`密钥交换对象。这
支持的组包括：`'modp1'`,`'modp2'`,`'modp5'`（定义于
[RFC 2412][]，但请参阅[警告][Caveats]） 和`'modp14'`,`'modp15'`,
`'modp16'`,`'modp17'`,`'modp18'`（定义于[RFC 3526][]).这
返回的对象模仿由
[`crypto.createDiffieHellman()`][crypto.createDiffieHellman()]，但不允许更改
键（带有[`diffieHellman.setPublicKey()`][diffieHellman.setPublicKey()]），例如）。这
使用这种方法的优点是当事人不必
事先生成或交换组模，节省两个处理器
和通信时间。

示例（获取共享密钥）：

```mjs
const {
  getDiffieHellman
} = await import('node:crypto');
const alice = getDiffieHellman('modp14');
const bob = getDiffieHellman('modp14');

alice.generateKeys();
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

/* aliceSecret and bobSecret should be the same */
console.log(aliceSecret === bobSecret);
```

```cjs
const {
  getDiffieHellman,
} = require('node:crypto');

const alice = getDiffieHellman('modp14');
const bob = getDiffieHellman('modp14');

alice.generateKeys();
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

/* aliceSecret and bobSecret should be the same */
console.log(aliceSecret === bobSecret);
```

### `crypto.getFips()`

<!-- YAML
added: v10.0.0
-->

*   返回值：{数字}`1`当且仅当符合 FIPS 标准的加密提供程序是
    目前正在使用，`0`否则。未来的主要版本可能会更改
    此 API 的返回类型为 {布尔值}。

### `crypto.getHashes()`

<!-- YAML
added: v0.9.3
-->

*   返回：{string\[]} 受支持哈希算法名称的数组，
    如`'RSA-SHA256'`.哈希算法也称为“摘要”算法。

```mjs
const {
  getHashes
} = await import('node:crypto');

console.log(getHashes()); // ['DSA', 'DSA-SHA', 'DSA-SHA1', ...]
```

```cjs
const {
  getHashes,
} = require('node:crypto');

console.log(getHashes()); // ['DSA', 'DSA-SHA', 'DSA-SHA1', ...]
```

### `crypto.getRandomValues(typedArray)`

<!-- YAML
added: v17.4.0
-->

*   `typedArray`{缓冲区|TypedArray|数据视图|ArrayBuffer}
*   返回值：{缓冲区|TypedArray|数据视图|ArrayBuffer} Returns`typedArray`.

方便的别名[`crypto.webcrypto.getRandomValues()`][crypto.webcrypto.getRandomValues()].这
实现不符合 Web 加密规范，要编写
与 Web 兼容的代码使用[`crypto.webcrypto.getRandomValues()`][crypto.webcrypto.getRandomValues()]相反。

### `crypto.hkdf(digest, ikm, salt, info, keylen, callback)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `digest`{字符串}要使用的摘要算法。
*   `ikm`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象} 输入
    键控材料。它的长度必须至少为一个字节。
*   `salt`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 盐值。必须
    提供，但长度可以为零。
*   `info`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图} 附加信息值。
    必须提供，但长度可以为零，并且不能超过 1024 个字节。
*   `keylen`{数字}要生成的密钥的长度。必须大于 0。
    最大允许值为`255`乘以生成的字节数
    选定的摘要函数（例如`sha512`生成 64 字节哈希，使
    最大 HKDF 输出 16320 字节）。
*   `callback`{函数}
    *   `err`{错误}
    *   `derivedKey`{ArrayBuffer}

HKDF 是 RFC 5869 中定义的一个简单的密钥派生函数。给定的`ikm`,
`salt`和`info`与`digest`派生一个密钥`keylen`字节。

提供的`callback`函数使用两个参数调用：`err`和
`derivedKey`.如果在派生密钥时发生错误，`err`将被设置;
否则`err`将是`null`.已成功生成`derivedKey`将
作为 {ArrayBuffer} 传递给回调。如果存在错误，将引发错误
的输入参数指定无效值或类型。

```mjs
import { Buffer } from 'node:buffer';
const {
  hkdf
} = await import('node:crypto');

hkdf('sha512', 'key', 'salt', 'info', 64, (err, derivedKey) => {
  if (err) throw err;
  console.log(Buffer.from(derivedKey).toString('hex'));  // '24156e2...5391653'
});
```

```cjs
const {
  hkdf,
} = require('node:crypto');
const { Buffer } = require('node:buffer');

hkdf('sha512', 'key', 'salt', 'info', 64, (err, derivedKey) => {
  if (err) throw err;
  console.log(Buffer.from(derivedKey).toString('hex'));  // '24156e2...5391653'
});
```

### `crypto.hkdfSync(digest, ikm, salt, info, keylen)`

<!-- YAML
added: v15.0.0
-->

*   `digest`{字符串}要使用的摘要算法。
*   `ikm`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象} 输入
    键控材料。它的长度必须至少为一个字节。
*   `salt`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 盐值。必须
    提供，但长度可以为零。
*   `info`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图} 附加信息值。
    必须提供，但长度可以为零，并且不能超过 1024 个字节。
*   `keylen`{数字}要生成的密钥的长度。必须大于 0。
    最大允许值为`255`乘以生成的字节数
    选定的摘要函数（例如`sha512`生成 64 字节哈希，使
    最大 HKDF 输出 16320 字节）。
*   返回： {ArrayBuffer}

提供 RFC 5869 中定义的同步 HKDF 密钥派生函数。这
鉴于`ikm`,`salt`和`info`与`digest`派生一个密钥
`keylen`字节。

已成功生成`derivedKey`将作为 {ArrayBuffer} 返回。

如果任何输入参数指定了无效值或
类型，或者如果无法生成派生键。

```mjs
import { Buffer } from 'node:buffer';
const {
  hkdfSync
} = await import('node:crypto');

const derivedKey = hkdfSync('sha512', 'key', 'salt', 'info', 64);
console.log(Buffer.from(derivedKey).toString('hex'));  // '24156e2...5391653'
```

```cjs
const {
  hkdfSync,
} = require('node:crypto');
const { Buffer } = require('node:buffer');

const derivedKey = hkdfSync('sha512', 'key', 'salt', 'info', 64);
console.log(Buffer.from(derivedKey).toString('hex'));  // '24156e2...5391653'
```

### `crypto.pbkdf2(password, salt, iterations, keylen, digest, callback)`

<!-- YAML
added: v0.5.5
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The password and salt arguments can also be ArrayBuffer
                 instances.
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/30578
    description: The `iterations` parameter is now restricted to positive
                 values. Earlier releases treated other values as one.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11305
    description: The `digest` parameter is always required now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4047
    description: Calling this function without passing the `digest` parameter
                 is deprecated now and will emit a warning.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default encoding for `password` if it is a string changed
                 from `binary` to `utf8`.
-->

*   `password`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `salt`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `iterations`{数字}
*   `keylen`{数字}
*   `digest`{字符串}
*   `callback`{函数}
    *   `err`{错误}
    *   `derivedKey`{缓冲区}

提供基于密码的异步密钥派生函数 2 （PBKDF2）
实现。由 指定的选定 HMAC 摘要算法`digest`是
应用于派生所请求字节长度 （`keylen`） 从
`password`,`salt`和`iterations`.

提供的`callback`函数使用两个参数调用：`err`和
`derivedKey`.如果在派生密钥时发生错误，`err`将被设置;
否则`err`将是`null`.默认情况下，已成功生成
`derivedKey`将作为[`Buffer`][Buffer].错误将是
如果任何输入参数指定了无效值或类型，则抛出。

如果`digest`是`null`,`'sha1'`将使用。此行为已弃用，
请指定`digest`明确地。

这`iterations`参数必须是设置得尽可能高的数字。这
迭代次数越多，派生密钥就越安全，
但需要更长的时间才能完成。

这`salt`应尽可能唯一。建议盐是
随机且长度至少为 16 个字节。看[NIST SP 800-132][]了解详情。

传递 字符串时`password`或`salt`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

```mjs
const {
  pbkdf2
} = await import('node:crypto');

pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
```

```cjs
const {
  pbkdf2,
} = require('node:crypto');

pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
```

这`crypto.DEFAULT_ENCODING`属性可用于更改
`derivedKey`传递给回调。但是，此属性已
弃用，应避免使用。

```mjs
import crypto from 'node:crypto';
crypto.DEFAULT_ENCODING = 'hex';
crypto.pbkdf2('secret', 'salt', 100000, 512, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey);  // '3745e48...aa39b34'
});
```

```cjs
const crypto = require('node:crypto');
crypto.DEFAULT_ENCODING = 'hex';
crypto.pbkdf2('secret', 'salt', 100000, 512, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey);  // '3745e48...aa39b34'
});
```

可以使用以下命令检索支持的摘要函数数组
[`crypto.getHashes()`][crypto.getHashes()].

此 API 使用 libuv 的 threadpool，它可能具有令人惊讶的和
对某些应用程序的性能产生负面影响;查看
[`UV_THREADPOOL_SIZE`][UV_THREADPOOL_SIZE]文档以获取更多信息。

### `crypto.pbkdf2Sync(password, salt, iterations, keylen, digest)`

<!-- YAML
added: v0.9.3
changes:
  - version: v14.0.0
    pr-url: https://github.com/nodejs/node/pull/30578
    description: The `iterations` parameter is now restricted to positive
                 values. Earlier releases treated other values as one.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4047
    description: Calling this function without passing the `digest` parameter
                 is deprecated now and will emit a warning.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default encoding for `password` if it is a string changed
                 from `binary` to `utf8`.
-->

*   `password`{字符串|缓冲区|TypedArray|数据视图}
*   `salt`{字符串|缓冲区|TypedArray|数据视图}
*   `iterations`{数字}
*   `keylen`{数字}
*   `digest`{字符串}
*   返回：{缓冲区}

提供基于密码的同步密钥派生函数 2 （PBKDF2）
实现。由 指定的选定 HMAC 摘要算法`digest`是
应用于派生所请求字节长度 （`keylen`） 从
`password`,`salt`和`iterations`.

如果发生错误`Error`将被抛出，否则派生的密钥将为
作为 返回[`Buffer`][Buffer].

如果`digest`是`null`,`'sha1'`将使用。此行为已弃用，
请指定`digest`明确地。

这`iterations`参数必须是设置得尽可能高的数字。这
迭代次数越多，派生密钥就越安全，
但需要更长的时间才能完成。

这`salt`应尽可能唯一。建议盐是
随机且长度至少为 16 个字节。看[NIST SP 800-132][]了解详情。

传递 字符串时`password`或`salt`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

```mjs
const {
  pbkdf2Sync
} = await import('node:crypto');

const key = pbkdf2Sync('secret', 'salt', 100000, 64, 'sha512');
console.log(key.toString('hex'));  // '3745e48...08d59ae'
```

```cjs
const {
  pbkdf2Sync,
} = require('node:crypto');

const key = pbkdf2Sync('secret', 'salt', 100000, 64, 'sha512');
console.log(key.toString('hex'));  // '3745e48...08d59ae'
```

这`crypto.DEFAULT_ENCODING`属性可用于更改
`derivedKey`返回。但是，此属性已弃用，请使用
应避免使用。

```mjs
import crypto from 'node:crypto';
crypto.DEFAULT_ENCODING = 'hex';
const key = crypto.pbkdf2Sync('secret', 'salt', 100000, 512, 'sha512');
console.log(key);  // '3745e48...aa39b34'
```

```cjs
const crypto = require('node:crypto');
crypto.DEFAULT_ENCODING = 'hex';
const key = crypto.pbkdf2Sync('secret', 'salt', 100000, 512, 'sha512');
console.log(key);  // '3745e48...aa39b34'
```

可以使用以下命令检索支持的摘要函数数组
[`crypto.getHashes()`][crypto.getHashes()].

### `crypto.privateDecrypt(privateKey, buffer)`

<!-- YAML
added: v0.11.14
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: Added string, ArrayBuffer, and CryptoKey as allowable key
                 types. The oaepLabel can be an ArrayBuffer. The buffer can
                 be a string or ArrayBuffer. All types that accept buffers
                 are limited to a maximum of 2 ** 31 - 1 bytes.
  - version: v12.11.0
    pr-url: https://github.com/nodejs/node/pull/29489
    description: The `oaepLabel` option was added.
  - version: v12.9.0
    pr-url: https://github.com/nodejs/node/pull/28335
    description: The `oaepHash` option was added.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: This function now supports key objects.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `privateKey`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
    *   `oaepHash`{字符串}用于 OAEP 填充和 MGF1 的哈希函数。
        **违约：** `'sha1'`
    *   `oaepLabel`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 标签
        用于 OAEP 填充。如果未指定，则不使用标签。
    *   `padding`{crypto.constants} 在 中定义的可选填充值
        `crypto.constants`，可能是：`crypto.constants.RSA_NO_PADDING`,
        `crypto.constants.RSA_PKCS1_PADDING`或
        `crypto.constants.RSA_PKCS1_OAEP_PADDING`.
*   `buffer`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   返回： {缓冲区} 一个新的`Buffer`与解密的内容。

<!--lint enable maximum-line-length remark-lint-->

解密`buffer`跟`privateKey`.`buffer`以前使用
相应的公钥，例如使用[`crypto.publicEncrypt()`][crypto.publicEncrypt()].

如果`privateKey`不是[`KeyObject`][KeyObject]，则此函数的行为就像
`privateKey`已传递到[`crypto.createPrivateKey()`][crypto.createPrivateKey()].如果是
对象，`padding`属性可以传递。否则，此函数使用
`RSA_PKCS1_OAEP_PADDING`.

### `crypto.privateEncrypt(privateKey, buffer)`

<!-- YAML
added: v1.1.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: Added string, ArrayBuffer, and CryptoKey as allowable key
                 types. The passphrase can be an ArrayBuffer. The buffer can
                 be a string or ArrayBuffer. All types that accept buffers
                 are limited to a maximum of 2 ** 31 - 1 bytes.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: This function now supports key objects.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `privateKey`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
    *   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
        PEM 编码的私钥。
    *   `passphrase`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 可选
        私钥的密码。
    *   `padding`{crypto.constants} 在 中定义的可选填充值
        `crypto.constants`，可能是：`crypto.constants.RSA_NO_PADDING`或
        `crypto.constants.RSA_PKCS1_PADDING`.
    *   `encoding`{字符串}在以下情况下要使用的字符串编码`buffer`,`key`,
        或`passphrase`是字符串。
*   `buffer`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   返回： {缓冲区} 一个新的`Buffer`与加密的内容。

<!--lint enable maximum-line-length remark-lint-->

加密`buffer`跟`privateKey`.返回的数据可以使用以下命令解密
相应的公钥，例如使用[`crypto.publicDecrypt()`][crypto.publicDecrypt()].

如果`privateKey`不是[`KeyObject`][KeyObject]，则此函数的行为就像
`privateKey`已传递到[`crypto.createPrivateKey()`][crypto.createPrivateKey()].如果是
对象，`padding`属性可以传递。否则，此函数使用
`RSA_PKCS1_PADDING`.

### `crypto.publicDecrypt(key, buffer)`

<!-- YAML
added: v1.1.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: Added string, ArrayBuffer, and CryptoKey as allowable key
                 types. The passphrase can be an ArrayBuffer. The buffer can
                 be a string or ArrayBuffer. All types that accept buffers
                 are limited to a maximum of 2 ** 31 - 1 bytes.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: This function now supports key objects.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `key`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
    *   `passphrase`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 可选
        私钥的密码。
    *   `padding`{crypto.constants} 在 中定义的可选填充值
        `crypto.constants`，可能是：`crypto.constants.RSA_NO_PADDING`或
        `crypto.constants.RSA_PKCS1_PADDING`.
    *   `encoding`{字符串}在以下情况下要使用的字符串编码`buffer`,`key`,
        或`passphrase`是字符串。
*   `buffer`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   返回： {缓冲区} 一个新的`Buffer`与解密的内容。

<!--lint enable maximum-line-length remark-lint-->

解密`buffer`跟`key`.`buffer`以前使用
相应的私钥，例如使用[`crypto.privateEncrypt()`][crypto.privateEncrypt()].

如果`key`不是[`KeyObject`][KeyObject]，则此函数的行为就像
`key`已传递到[`crypto.createPublicKey()`][crypto.createPublicKey()].如果是
对象，`padding`属性可以传递。否则，此函数使用
`RSA_PKCS1_PADDING`.

由于 RSA 公钥可以从私钥派生，因此私钥可以
而不是公钥。

### `crypto.publicEncrypt(key, buffer)`

<!-- YAML
added: v0.11.14
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: Added string, ArrayBuffer, and CryptoKey as allowable key
                 types. The oaepLabel and passphrase can be ArrayBuffers. The
                 buffer can be a string or ArrayBuffer. All types that accept
                 buffers are limited to a maximum of 2 ** 31 - 1 bytes.
  - version: v12.11.0
    pr-url: https://github.com/nodejs/node/pull/29489
    description: The `oaepLabel` option was added.
  - version: v12.9.0
    pr-url: https://github.com/nodejs/node/pull/28335
    description: The `oaepHash` option was added.
  - version: v11.6.0
    pr-url: https://github.com/nodejs/node/pull/24234
    description: This function now supports key objects.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `key`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
    *   `key`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
        PEM 编码的公钥或私钥{KeyObject}或{CryptoKey}。
    *   `oaepHash`{字符串}用于 OAEP 填充和 MGF1 的哈希函数。
        **违约：** `'sha1'`
    *   `oaepLabel`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 标签
        用于 OAEP 填充。如果未指定，则不使用标签。
    *   `passphrase`{字符串|ArrayBuffer|缓冲区|TypedArray|DataView} 可选
        私钥的密码。
    *   `padding`{crypto.constants} 在 中定义的可选填充值
        `crypto.constants`，可能是：`crypto.constants.RSA_NO_PADDING`,
        `crypto.constants.RSA_PKCS1_PADDING`或
        `crypto.constants.RSA_PKCS1_OAEP_PADDING`.
    *   `encoding`{字符串}在以下情况下要使用的字符串编码`buffer`,`key`,
        `oaepLabel`或`passphrase`是字符串。
*   `buffer`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   返回： {缓冲区} 一个新的`Buffer`与加密的内容。

<!--lint enable maximum-line-length remark-lint-->

加密的内容`buffer`跟`key`并返回新的
[`Buffer`][Buffer]包含加密内容。返回的数据可以使用以下命令解密
相应的私钥，例如使用[`crypto.privateDecrypt()`][crypto.privateDecrypt()].

如果`key`不是[`KeyObject`][KeyObject]，则此函数的行为就像
`key`已传递到[`crypto.createPublicKey()`][crypto.createPublicKey()].如果是
对象，`padding`属性可以传递。否则，此函数使用
`RSA_PKCS1_OAEP_PADDING`.

由于 RSA 公钥可以从私钥派生，因此私钥可以
而不是公钥。

### `crypto.randomBytes(size[, callback])`

<!-- YAML
added: v0.5.8
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/16454
    description: Passing `null` as the `callback` argument now throws
                 `ERR_INVALID_CALLBACK`.
-->

*   `size`{数字}要生成的字节数。 这`size`必须
    不大于`2**31 - 1`.
*   `callback`{函数}
    *   `err`{错误}
    *   `buf`{缓冲区}
*   返回：{缓冲区}，如果`callback`未提供函数。

生成加密强伪随机数据。这`size`论点
是一个数字，指示要生成的字节数。

如果`callback`提供函数，字节异步生成
和`callback`函数使用两个参数调用：`err`和`buf`.
如果发生错误，`err`将是一个`Error`对象;否则它是`null`.这
`buf`参数是一个[`Buffer`][Buffer]包含生成的字节。

```mjs
// Asynchronous
const {
  randomBytes
} = await import('node:crypto');

randomBytes(256, (err, buf) => {
  if (err) throw err;
  console.log(`${buf.length} bytes of random data: ${buf.toString('hex')}`);
});
```

```cjs
// Asynchronous
const {
  randomBytes,
} = require('node:crypto');

randomBytes(256, (err, buf) => {
  if (err) throw err;
  console.log(`${buf.length} bytes of random data: ${buf.toString('hex')}`);
});
```

如果`callback`未提供函数，生成随机字节
同步并作为 返回[`Buffer`][Buffer].如果出现以下情况，将引发错误
生成字节时出现问题。

```mjs
// Synchronous
const {
  randomBytes
} = await import('node:crypto');

const buf = randomBytes(256);
console.log(
  `${buf.length} bytes of random data: ${buf.toString('hex')}`);
```

```cjs
// Synchronous
const {
  randomBytes,
} = require('node:crypto');

const buf = randomBytes(256);
console.log(
  `${buf.length} bytes of random data: ${buf.toString('hex')}`);
```

这`crypto.randomBytes()`方法不会完成，直到有
足够的熵可用。
这通常不应超过几毫秒。唯一一次
当生成随机字节时，可以想象块更长的时间
时间在启动后，当整个系统的熵仍然很低时。

此 API 使用 libuv 的 threadpool，它可能具有令人惊讶的和
对某些应用程序的性能产生负面影响;查看
[`UV_THREADPOOL_SIZE`][UV_THREADPOOL_SIZE]文档以获取更多信息。

的异步版本`crypto.randomBytes()`在单个中执行
线程池请求。要最大限度地减少线程池任务长度变化，请分区
大`randomBytes`作为履行客户的一部分时的请求
请求。

### `crypto.randomFillSync(buffer[, offset][, size])`

<!-- YAML
added:
  - v7.10.0
  - v6.13.0
changes:
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15231
    description: The `buffer` argument may be any `TypedArray` or `DataView`.
-->

*   `buffer`{ArrayBuffer|缓冲区|TypedArray|必须提供 DataView}。这
    提供的大小`buffer`不得大于`2**31 - 1`.
*   `offset`{数字}**违约：** `0`
*   `size`{数字}**违约：** `buffer.length - offset`.这`size`必须
    不大于`2**31 - 1`.
*   返回值： {ArrayBuffer|缓冲区|TypedArray|DataView} 作为 传递的对象
    `buffer`论点。

同步版本[`crypto.randomFill()`][crypto.randomFill()].

```mjs
import { Buffer } from 'node:buffer';
const { randomFillSync } = await import('node:crypto');

const buf = Buffer.alloc(10);
console.log(randomFillSync(buf).toString('hex'));

randomFillSync(buf, 5);
console.log(buf.toString('hex'));

// The above is equivalent to the following:
randomFillSync(buf, 5, 5);
console.log(buf.toString('hex'));
```

```cjs
const { randomFillSync } = require('node:crypto');
const { Buffer } = require('node:buffer');

const buf = Buffer.alloc(10);
console.log(randomFillSync(buf).toString('hex'));

randomFillSync(buf, 5);
console.log(buf.toString('hex'));

// The above is equivalent to the following:
randomFillSync(buf, 5, 5);
console.log(buf.toString('hex'));
```

任何`ArrayBuffer`,`TypedArray`或`DataView`实例可以作为
`buffer`.

```mjs
import { Buffer } from 'node:buffer';
const { randomFillSync } = await import('node:crypto');

const a = new Uint32Array(10);
console.log(Buffer.from(randomFillSync(a).buffer,
                        a.byteOffset, a.byteLength).toString('hex'));

const b = new DataView(new ArrayBuffer(10));
console.log(Buffer.from(randomFillSync(b).buffer,
                        b.byteOffset, b.byteLength).toString('hex'));

const c = new ArrayBuffer(10);
console.log(Buffer.from(randomFillSync(c)).toString('hex'));
```

```cjs
const { randomFillSync } = require('node:crypto');
const { Buffer } = require('node:buffer');

const a = new Uint32Array(10);
console.log(Buffer.from(randomFillSync(a).buffer,
                        a.byteOffset, a.byteLength).toString('hex'));

const b = new DataView(new ArrayBuffer(10));
console.log(Buffer.from(randomFillSync(b).buffer,
                        b.byteOffset, b.byteLength).toString('hex'));

const c = new ArrayBuffer(10);
console.log(Buffer.from(randomFillSync(c)).toString('hex'));
```

### `crypto.randomFill(buffer[, offset][, size], callback)`

<!-- YAML
added:
  - v7.10.0
  - v6.13.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15231
    description: The `buffer` argument may be any `TypedArray` or `DataView`.
-->

*   `buffer`{ArrayBuffer|缓冲区|TypedArray|必须提供 DataView}。这
    提供的大小`buffer`不得大于`2**31 - 1`.
*   `offset`{数字}**违约：** `0`
*   `size`{数字}**违约：** `buffer.length - offset`.这`size`必须
    不大于`2**31 - 1`.
*   `callback`{函数}`function(err, buf) {}`.

此功能类似于[`crypto.randomBytes()`][crypto.randomBytes()]但需要第一个
参数成为[`Buffer`][Buffer]这将被填补。它还
要求传入回调。

如果`callback`未提供函数，将引发错误。

```mjs
import { Buffer } from 'node:buffer';
const { randomFill } = await import('node:crypto');

const buf = Buffer.alloc(10);
randomFill(buf, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

randomFill(buf, 5, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

// The above is equivalent to the following:
randomFill(buf, 5, 5, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});
```

```cjs
const { randomFill } = require('node:crypto');
const { Buffer } = require('node:buffer');

const buf = Buffer.alloc(10);
randomFill(buf, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

randomFill(buf, 5, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

// The above is equivalent to the following:
randomFill(buf, 5, 5, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});
```

任何`ArrayBuffer`,`TypedArray`或`DataView`实例可以作为
`buffer`.

虽然这包括以下实例：`Float32Array`和`Float64Array`这
函数不应用于生成随机浮点数。这
结果可能包含`+Infinity`,`-Infinity`和`NaN`，即使数组
仅包含有限数，它们不是从均匀随机数中抽取的
分布，并且没有有意义的下限或上限。

```mjs
import { Buffer } from 'node:buffer';
const { randomFill } = await import('node:crypto');

const a = new Uint32Array(10);
randomFill(a, (err, buf) => {
  if (err) throw err;
  console.log(Buffer.from(buf.buffer, buf.byteOffset, buf.byteLength)
    .toString('hex'));
});

const b = new DataView(new ArrayBuffer(10));
randomFill(b, (err, buf) => {
  if (err) throw err;
  console.log(Buffer.from(buf.buffer, buf.byteOffset, buf.byteLength)
    .toString('hex'));
});

const c = new ArrayBuffer(10);
randomFill(c, (err, buf) => {
  if (err) throw err;
  console.log(Buffer.from(buf).toString('hex'));
});
```

```cjs
const { randomFill } = require('node:crypto');
const { Buffer } = require('node:buffer');

const a = new Uint32Array(10);
randomFill(a, (err, buf) => {
  if (err) throw err;
  console.log(Buffer.from(buf.buffer, buf.byteOffset, buf.byteLength)
    .toString('hex'));
});

const b = new DataView(new ArrayBuffer(10));
randomFill(b, (err, buf) => {
  if (err) throw err;
  console.log(Buffer.from(buf.buffer, buf.byteOffset, buf.byteLength)
    .toString('hex'));
});

const c = new ArrayBuffer(10);
randomFill(c, (err, buf) => {
  if (err) throw err;
  console.log(Buffer.from(buf).toString('hex'));
});
```

此 API 使用 libuv 的 threadpool，它可能具有令人惊讶的和
对某些应用程序的性能产生负面影响;查看
[`UV_THREADPOOL_SIZE`][UV_THREADPOOL_SIZE]文档以获取更多信息。

的异步版本`crypto.randomFill()`在单个中执行
线程池请求。要最大限度地减少线程池任务长度变化，请分区
大`randomFill`作为履行客户的一部分时的请求
请求。

### `crypto.randomInt([min, ]max[, callback])`

<!-- YAML
added:
  - v14.10.0
  - v12.19.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

*   `min`{整数}随机范围的开始（包括随机范围）。**违约：** `0`.
*   `max`{整数}随机范围的结束（独占）。
*   `callback`{函数}`function(err, n) {}`.

返回随机整数`n`使得`min <= n < max`. 这
实现避免[模偏置][modulo bias].

范围 （`max - min`） 必须小于 2<sup>48</sup>.`min`和`max`必须
是[安全整数][safe integers].

如果`callback`未提供函数，随机整数为
同步生成。

```mjs
// Asynchronous
const {
  randomInt
} = await import('node:crypto');

randomInt(3, (err, n) => {
  if (err) throw err;
  console.log(`Random number chosen from (0, 1, 2): ${n}`);
});
```

```cjs
// Asynchronous
const {
  randomInt,
} = require('node:crypto');

randomInt(3, (err, n) => {
  if (err) throw err;
  console.log(`Random number chosen from (0, 1, 2): ${n}`);
});
```

```mjs
// Synchronous
const {
  randomInt
} = await import('node:crypto');

const n = randomInt(3);
console.log(`Random number chosen from (0, 1, 2): ${n}`);
```

```cjs
// Synchronous
const {
  randomInt,
} = require('node:crypto');

const n = randomInt(3);
console.log(`Random number chosen from (0, 1, 2): ${n}`);
```

```mjs
// With `min` argument
const {
  randomInt
} = await import('node:crypto');

const n = randomInt(1, 7);
console.log(`The dice rolled: ${n}`);
```

```cjs
// With `min` argument
const {
  randomInt,
} = require('node:crypto');

const n = randomInt(1, 7);
console.log(`The dice rolled: ${n}`);
```

### `crypto.randomUUID([options])`

<!-- YAML
added:
  - v15.6.0
  - v14.17.0
-->

*   `options`{对象}
    *   `disableEntropyCache`{布尔值}默认情况下，为了提高性能，
        节点.js生成并缓存足够的
        随机数据可生成多达 128 个随机 UUID。生成 UUID 的步骤
        不使用缓存，设置`disableEntropyCache`自`true`.
        **违约：** `false`.
*   返回：{字符串}

生成随机数[RFC 4122][]版本 4 UUID。UUID 是使用
加密伪随机数生成器。

### `crypto.scrypt(password, salt, keylen[, options], callback)`

<!-- YAML
added: v10.5.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The password and salt arguments can also be ArrayBuffer
                 instances.
  - version:
     - v12.8.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/28799
    description: The `maxmem` value can now be any safe integer.
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/21525
    description: The `cost`, `blockSize` and `parallelization` option names
                 have been added.
-->

*   `password`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `salt`{字符串|ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `keylen`{数字}
*   `options`{对象}
    *   `cost`{数字}CPU/内存成本参数。必须是两个更大的力量
        而不是一个。**违约：** `16384`.
    *   `blockSize`{数字}块大小参数。**违约：** `8`.
    *   `parallelization`{数字}并行化参数。**违约：** `1`.
    *   `N`{数字}的别名`cost`.只能指定两者中的一个。
    *   `r`{数字}的别名`blockSize`.只能指定两者中的一个。
    *   `p`{数字}的别名`parallelization`.只能指定两者中的一个。
    *   `maxmem`{数字}内存上限。当（大约）时，这是一个错误
        `128 * N * r > maxmem`.**违约：** `32 * 1024 * 1024`.
*   `callback`{函数}
    *   `err`{错误}
    *   `derivedKey`{缓冲区}

提供异步[地下室][scrypt]实现。Scrypt 是基于密码的
键派生函数，设计为计算成本高昂，并且
内存方面，以使暴力攻击没有回报。

这`salt`应尽可能唯一。建议盐是
随机且长度至少为 16 个字节。看[NIST SP 800-132][]了解详情。

传递 字符串时`password`或`salt`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

这`callback`函数使用两个参数调用：`err`和`derivedKey`.
`err`是键派生失败时的异常对象，否则`err`是
`null`.`derivedKey`作为传递给回调[`Buffer`][Buffer].

当任何输入参数指定无效值时，将引发异常
或类型。

```mjs
const {
  scrypt
} = await import('node:crypto');

// Using the factory defaults.
scrypt('password', 'salt', 64, (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
// Using a custom N parameter. Must be a power of two.
scrypt('password', 'salt', 64, { N: 1024 }, (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...aa39b34'
});
```

```cjs
const {
  scrypt,
} = require('node:crypto');

// Using the factory defaults.
scrypt('password', 'salt', 64, (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
// Using a custom N parameter. Must be a power of two.
scrypt('password', 'salt', 64, { N: 1024 }, (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...aa39b34'
});
```

### `crypto.scryptSync(password, salt, keylen[, options])`

<!-- YAML
added: v10.5.0
changes:
  - version:
     - v12.8.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/28799
    description: The `maxmem` value can now be any safe integer.
  - version: v10.9.0
    pr-url: https://github.com/nodejs/node/pull/21525
    description: The `cost`, `blockSize` and `parallelization` option names
                 have been added.
-->

*   `password`{字符串|缓冲区|TypedArray|数据视图}
*   `salt`{字符串|缓冲区|TypedArray|数据视图}
*   `keylen`{数字}
*   `options`{对象}
    *   `cost`{数字}CPU/内存成本参数。必须是两个更大的力量
        而不是一个。**违约：** `16384`.
    *   `blockSize`{数字}块大小参数。**违约：** `8`.
    *   `parallelization`{数字}并行化参数。**违约：** `1`.
    *   `N`{数字}的别名`cost`.只能指定两者中的一个。
    *   `r`{数字}的别名`blockSize`.只能指定两者中的一个。
    *   `p`{数字}的别名`parallelization`.只能指定两者中的一个。
    *   `maxmem`{数字}内存上限。当（大约）时，这是一个错误
        `128 * N * r > maxmem`.**违约：** `32 * 1024 * 1024`.
*   返回：{缓冲区}

提供同步[地下室][scrypt]实现。Scrypt 是基于密码的
键派生函数，设计为计算成本高昂，并且
内存方面，以使暴力攻击没有回报。

这`salt`应尽可能唯一。建议盐是
随机且长度至少为 16 个字节。看[NIST SP 800-132][]了解详情。

传递 字符串时`password`或`salt`，请考虑
[使用字符串作为加密 API 的输入时的警告][caveats when using strings as inputs to cryptographic APIs].

当键派生失败时，将引发异常，否则派生键为
作为 返回[`Buffer`][Buffer].

当任何输入参数指定无效值时，将引发异常
或类型。

```mjs
const {
  scryptSync
} = await import('node:crypto');
// Using the factory defaults.

const key1 = scryptSync('password', 'salt', 64);
console.log(key1.toString('hex'));  // '3745e48...08d59ae'
// Using a custom N parameter. Must be a power of two.
const key2 = scryptSync('password', 'salt', 64, { N: 1024 });
console.log(key2.toString('hex'));  // '3745e48...aa39b34'
```

```cjs
const {
  scryptSync,
} = require('node:crypto');
// Using the factory defaults.

const key1 = scryptSync('password', 'salt', 64);
console.log(key1.toString('hex'));  // '3745e48...08d59ae'
// Using a custom N parameter. Must be a power of two.
const key2 = scryptSync('password', 'salt', 64, { N: 1024 });
console.log(key2.toString('hex'));  // '3745e48...aa39b34'
```

### `crypto.secureHeapUsed()`

<!-- YAML
added: v15.6.0
-->

*   返回： {对象}
    *   `total`{数字}指定的已分配安全堆总大小
        使用`--secure-heap=n`命令行标志。
    *   `min`{数字}安全堆中的最小分配为
        使用`--secure-heap-min`命令行标志。
    *   `used`{数字}当前从 中分配的总字节数
        安全堆。
    *   `utilization`{数字}计算出的比率`used`自`total`
        分配的字节数。

### `crypto.setEngine(engine[, flags])`

<!-- YAML
added: v0.11.11
-->

*   `engine`{字符串}
*   `flags`{crypto.constants}**违约：** `crypto.constants.ENGINE_METHOD_ALL`

加载并设置`engine`对于部分或全部 OpenSSL 函数（由标志选择）。

`engine`可以是 id 或引擎共享库的路径。

可选`flags`参数用法`ENGINE_METHOD_ALL`默认情况下。这`flags`
是一个位字段，采用以下标志之一或混合（定义于
`crypto.constants`):

*   `crypto.constants.ENGINE_METHOD_RSA`
*   `crypto.constants.ENGINE_METHOD_DSA`
*   `crypto.constants.ENGINE_METHOD_DH`
*   `crypto.constants.ENGINE_METHOD_RAND`
*   `crypto.constants.ENGINE_METHOD_EC`
*   `crypto.constants.ENGINE_METHOD_CIPHERS`
*   `crypto.constants.ENGINE_METHOD_DIGESTS`
*   `crypto.constants.ENGINE_METHOD_PKEY_METHS`
*   `crypto.constants.ENGINE_METHOD_PKEY_ASN1_METHS`
*   `crypto.constants.ENGINE_METHOD_ALL`
*   `crypto.constants.ENGINE_METHOD_NONE`

下面的标志在 OpenSSL-1.1.0 中已弃用。

*   `crypto.constants.ENGINE_METHOD_ECDH`
*   `crypto.constants.ENGINE_METHOD_ECDSA`
*   `crypto.constants.ENGINE_METHOD_STORE`

### `crypto.setFips(bool)`

<!-- YAML
added: v10.0.0
-->

*   `bool`{布尔值}`true`以启用 FIPS 模式。

在启用了 FIPS 的节点.js版本中启用符合 FIPS 的加密提供程序。
如果 FIPS 模式不可用，则引发错误。

### `crypto.sign(algorithm, data, key[, callback])`

<!-- YAML
added: v12.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v15.12.0
    pr-url: https://github.com/nodejs/node/pull/37500
    description: Optional callback argument added.
  - version:
     - v13.2.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/29292
    description: This function now supports IEEE-P1363 DSA and ECDSA signatures.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `algorithm`{字符串 | 空值 | 未定义}
*   `data`{ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `key`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
*   `callback`{函数}
    *   `err`{错误}
    *   `signature`{缓冲区}
*   返回：{缓冲区}，如果`callback`未提供函数。

<!--lint enable maximum-line-length remark-lint-->

计算并返回 的签名`data`使用给定的私钥和
算法。如果`algorithm`是`null`或`undefined`，则算法为
取决于密钥类型（尤其是 Ed25519 和 Ed448）。

如果`key`不是[`KeyObject`][KeyObject]，则此函数的行为就像`key`一直
传递到[`crypto.createPrivateKey()`][crypto.createPrivateKey()].如果是对象，则
可以传递其他属性：

*   `dsaEncoding`{字符串}对于 DSA 和 ECDSA，此选项指定
    生成的签名的格式。它可以是以下之一：
    *   `'der'`（默认值）：DER 编码的 ASN.1 签名结构编码`(r, s)`.
    *   `'ieee-p1363'`：签名格式`r || s`如IEEE-P1363中所建议。
*   `padding`{整数}RSA 的可选填充值，如下所示之一：

    *   `crypto.constants.RSA_PKCS1_PADDING`（默认值）
    *   `crypto.constants.RSA_PKCS1_PSS_PADDING`

    `RSA_PKCS1_PSS_PADDING`将使用具有相同哈希函数的 MGF1
    用于按照 第 3.1 节中指定的消息进行签名[RFC 4055][].
*   `saltLength`{整数}填充时的盐长度为
    `RSA_PKCS1_PSS_PADDING`.特殊值
    `crypto.constants.RSA_PSS_SALTLEN_DIGEST`将盐的长度设置为消化液
    大小`crypto.constants.RSA_PSS_SALTLEN_MAX_SIGN`（默认值）将其设置为
    最大允许值。

如果`callback`函数提供此函数使用 libuv 的线程池。

### `crypto.subtle`

<!-- YAML
added: v17.4.0
-->

*   类型： {微妙的密码}

方便的别名[`crypto.webcrypto.subtle`][crypto.webcrypto.subtle].

### `crypto.timingSafeEqual(a, b)`

<!-- YAML
added: v6.6.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The a and b arguments can also be ArrayBuffer.
-->

*   `a`{ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `b`{ArrayBuffer|缓冲区|TypedArray|数据视图}
*   返回：{布尔值}

此函数比较表示给定
`ArrayBuffer`,`TypedArray`或`DataView`使用常量时间的实例
算法。

此函数不会泄漏计时信息
将允许攻击者猜测其中一个值。这适用于
比较 HMAC 摘要或机密值，如身份验证 Cookie 或
[功能网址](https://www.w3.org/TR/capability-urls/).

`a`和`b`必须两者都是`Buffer`s,`TypedArray`s，或`DataView`s，以及他们
必须具有相同的字节长度。在以下情况下会引发错误：`a`和`b`有
不同的字节长度。

如果至少一个`a`和`b`是一个`TypedArray`每字节数超过一个
条目，例如`Uint16Array`，则将使用平台计算结果
字节顺序。

<strong class="critical">当两个输入都`Float32Array`s 或
`Float64Array`s，此函数可能会由于 IEEE 754 而返回意外结果
浮点数的编码。特别是，两者都不是`x === y`也不
`Object.is(x, y)`意味着两个浮点的字节表示
数字`x`和`y`相等。</strong>

用途`crypto.timingSafeEqual`不保证*周围*法典
是定时安全的。应注意确保周围的代码确实
不会引入计时漏洞。

### `crypto.verify(algorithm, data, key, signature[, callback])`

<!-- YAML
added: v12.0.0
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
  - version: v15.12.0
    pr-url: https://github.com/nodejs/node/pull/37500
    description: Optional callback argument added.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35093
    description: The data, key, and signature arguments can also be ArrayBuffer.
  - version:
     - v13.2.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/29292
    description: This function now supports IEEE-P1363 DSA and ECDSA signatures.
-->

<!--lint disable maximum-line-length remark-lint-->

*   `algorithm`{string|null|undefined}
*   `data`{ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `key`{对象|字符串|ArrayBuffer|缓冲区|TypedArray|数据视图|键对象|CryptoKey}
*   `signature`{ArrayBuffer|缓冲区|TypedArray|数据视图}
*   `callback`{函数}
    *   `err`{错误}
    *   `result`{布尔值}
*   返回：{布尔值}`true`或`false`取决于的有效性
    数据和公钥的签名，如果`callback`函数不是
    提供。

<!--lint enable maximum-line-length remark-lint-->

验证 给定的签名`data`使用给定的密钥和算法。如果
`algorithm`是`null`或`undefined`，则算法依赖于
键类型（尤其是 Ed25519 和 Ed448）。

如果`key`不是[`KeyObject`][KeyObject]，则此函数的行为就像`key`一直
传递到[`crypto.createPublicKey()`][crypto.createPublicKey()].如果是对象，则
可以传递其他属性：

*   `dsaEncoding`{字符串}对于 DSA 和 ECDSA，此选项指定
    签名的格式。它可以是以下之一：
    *   `'der'`（默认值）：DER 编码的 ASN.1 签名结构编码`(r, s)`.
    *   `'ieee-p1363'`：签名格式`r || s`如IEEE-P1363中所建议。
*   `padding`{整数}RSA 的可选填充值，如下所示之一：

    *   `crypto.constants.RSA_PKCS1_PADDING`（默认值）
    *   `crypto.constants.RSA_PKCS1_PSS_PADDING`

    `RSA_PKCS1_PSS_PADDING`将使用具有相同哈希函数的 MGF1
    用于按照 第 3.1 节中指定的消息进行签名[RFC 4055][].
*   `saltLength`{整数}填充时的盐长度为
    `RSA_PKCS1_PSS_PADDING`.特殊值
    `crypto.constants.RSA_PSS_SALTLEN_DIGEST`将盐的长度设置为消化液
    大小`crypto.constants.RSA_PSS_SALTLEN_MAX_SIGN`（默认值）将其设置为
    最大允许值。

这`signature`参数是先前计算的`data`.

因为公钥可以从私钥、私钥或公钥派生
密钥可能传递`key`.

如果`callback`函数提供此函数使用 libuv 的线程池。

### `crypto.webcrypto`

<!-- YAML
added: v15.0.0
-->

类型：{Crypto} Web Crypto API 标准的实现。

查看[Web Crypto API 文档][Web Crypto API documentation]了解详情。

## 笔记

### 使用字符串作为加密 API 的输入

由于历史原因，Node提供的许多加密API.js接受
字符串作为输入，其中基础加密算法在字节上工作
序列。这些实例包括明文、密文、对称密钥、
初始化向量、密码短语、盐、身份验证标记、
和其他经过身份验证的数据。

将字符串传递到加密 API 时，请考虑以下因素。

*   并非所有字节序列都是有效的 UTF-8 字符串。因此，当一个字节
    长度序列`n`由字符串派生而来，其熵一般为
    低于随机或伪随机的熵`n`字节序列。
    例如，没有 UTF-8 字符串将导致字节序列`c0 af`.秘密
    键应该几乎完全是随机或伪随机字节序列。
*   同样，在将随机或伪随机字节序列转换为 UTF-8 时
    字符串、不表示有效代码点的子序列可能会被替换
    由 Unicode 替换字符 （`U+FFFD`).的字节表示形式
    因此，生成的 Unicode 字符串可能不等于字节序列
    字符串的创建自。

    ```js
    const original = [0xc0, 0xaf];
    const bytesAsString = Buffer.from(original).toString('utf8');
    const stringAsBytes = Buffer.from(bytesAsString, 'utf8');
    console.log(stringAsBytes);
    // Prints '<Buffer ef bf bd ef bf bd>'.
    ```

    密码、哈希函数、签名算法和密钥的输出
    派生函数是伪随机字节序列，不应是
    用作 Unicode 字符串。
*   当从用户输入中获取字符串时，某些 Unicode 字符可以是
    以多种等效方式表示，导致不同的字节
    序列。例如，将用户密码短语传递给密钥派生时
    函数，如 PBKDF2 或 scrypt，密钥派生函数的结果
    取决于字符串是使用组合字符还是分解字符。节点.js
    不规范化字符表示形式。开发人员应考虑使用
    [`String.prototype.normalize()`][String.prototype.normalize()]在将用户输入传递给
    加密接口。

### 传统流 API（节点之前.js 0.10）

加密模块被添加到Node.js之前，才有一个概念
统一的流 API，之前有[`Buffer`][Buffer]用于处理的对象
二进制数据。因此，许多`crypto`定义的类有方法没有
通常位于其他 Node.js实现[流][stream]
原料药（例如`update()`,`final()`或`digest()`).此外，接受许多方法
并返回`'latin1'`默认情况下编码字符串，而不是`Buffer`s.这
在 Node.js v0.8 之后更改了默认值以使用[`Buffer`][Buffer]默认对象
相反。

### 支持弱算法或受损算法

这`node:crypto`模块仍然支持一些已经
已泄露，目前不建议使用。该 API 还允许
使用密钥大小太弱而无法安全的小密码和哈希
用。

用户应全权负责选择加密货币
算法和密钥大小根据其安全要求。

根据[NIST SP 800-131A][]:

*   MD5 和 SHA-1 在抗碰撞性
    必需的，例如数字签名。
*   建议与 RSA、DSA 和 DH 算法一起使用的密钥具有
    至少2048位以及至少ECDSA和ECDH曲线的位
    224位，可安全使用数年。
*   卫生署集团`modp1`,`modp2`和`modp5`具有密钥大小
    小于 2048 位，不建议使用。

有关其他建议和详细信息，请参阅参考。

一些具有已知弱点且在
实践只能通过[旧版提供商][legacy provider]，这不是
默认情况下启用。

### 断续器模式

CCM 是受支持的[自动评估算法][AEAD algorithms].使用此应用程序的应用程序
模式在使用密码 API 时必须遵守某些限制：

*   身份验证标记长度必须在密码创建期间由
    设置`authTagLength`选项，并且必须是 4、6、8、10、12、14 或
    16 字节。
*   初始化向量的长度（随机数）`N`必须介于 7 和 13 之间
    字节 （`7 ≤ N ≤ 13`).
*   明文的长度限制为`2 ** (8 * (15 - N))`字节。
*   解密时，必须通过以下方式设置身份验证标记`setAuthTag()`以前
    叫`update()`.
    否则，解密将失败，并且`final()`将引发错误
    遵守[RFC 3610][].
*   使用流方法，例如`write(data)`,`end(data)`或`pipe()`在CCM中
    模式可能会失败，因为 CCM 无法处理每个实例的多个数据块。
*   传递其他经过身份验证的数据 （AAD） 时，实际长度
    必须将消息（以字节为单位）传递给`setAAD()`通过`plaintextLength`
    选择。
    许多加密库在密文中包含身份验证标记，
    这意味着它们产生长度的密文
    `plaintextLength + authTagLength`.节点.js不包括身份验证
    标记中，因此密文长度始终为`plaintextLength`.
    如果未使用 AAD，则不需要这样做。
*   当CCM一次处理整个消息时，`update()`必须完全调用
    一次。
*   即使打电话`update()`足以加密/解密消息，
    应用*必须*叫`final()`以计算或验证
    身份验证标记。

```mjs
import { Buffer } from 'node:buffer';
const {
  createCipheriv,
  createDecipheriv,
  randomBytes
} = await import('node:crypto');

const key = 'keykeykeykeykeykeykeykey';
const nonce = randomBytes(12);

const aad = Buffer.from('0123456789', 'hex');

const cipher = createCipheriv('aes-192-ccm', key, nonce, {
  authTagLength: 16
});
const plaintext = 'Hello world';
cipher.setAAD(aad, {
  plaintextLength: Buffer.byteLength(plaintext)
});
const ciphertext = cipher.update(plaintext, 'utf8');
cipher.final();
const tag = cipher.getAuthTag();

// Now transmit { ciphertext, nonce, tag }.

const decipher = createDecipheriv('aes-192-ccm', key, nonce, {
  authTagLength: 16
});
decipher.setAuthTag(tag);
decipher.setAAD(aad, {
  plaintextLength: ciphertext.length
});
const receivedPlaintext = decipher.update(ciphertext, null, 'utf8');

try {
  decipher.final();
} catch (err) {
  throw new Error('Authentication failed!', { cause: err });
}

console.log(receivedPlaintext);
```

```cjs
const { Buffer } = require('node:buffer');
const {
  createCipheriv,
  createDecipheriv,
  randomBytes,
} = require('node:crypto');

const key = 'keykeykeykeykeykeykeykey';
const nonce = randomBytes(12);

const aad = Buffer.from('0123456789', 'hex');

const cipher = createCipheriv('aes-192-ccm', key, nonce, {
  authTagLength: 16
});
const plaintext = 'Hello world';
cipher.setAAD(aad, {
  plaintextLength: Buffer.byteLength(plaintext)
});
const ciphertext = cipher.update(plaintext, 'utf8');
cipher.final();
const tag = cipher.getAuthTag();

// Now transmit { ciphertext, nonce, tag }.

const decipher = createDecipheriv('aes-192-ccm', key, nonce, {
  authTagLength: 16
});
decipher.setAuthTag(tag);
decipher.setAAD(aad, {
  plaintextLength: ciphertext.length
});
const receivedPlaintext = decipher.update(ciphertext, null, 'utf8');

try {
  decipher.final();
} catch (err) {
  throw new Error('Authentication failed!', { cause: err });
}

console.log(receivedPlaintext);
```

## 加密常量

以下常量导出的`crypto.constants`适用于各种用途
这`node:crypto`,`node:tls`和`node:https`模块和一般都是
特定于 OpenSSL。

### OpenSSL 选项

查看[SSL OP 标志列表][list of SSL OP Flags]了解详情。

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>SSL_OP_ALL</code></td>
    <td>Applies multiple bug workarounds within OpenSSL. See
    <a href="https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html">https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html</a>
    for detail.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_ALLOW_NO_DHE_KEX</code></td>
    <td>Instructs OpenSSL to allow a non-[EC]DHE-based key exchange mode
    for TLS v1.3</td>
  </tr>
  <tr>
    <td><code>SSL_OP_ALLOW_UNSAFE_LEGACY_RENEGOTIATION</code></td>
    <td>Allows legacy insecure renegotiation between OpenSSL and unpatched
    clients or servers. See
    <a href="https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html">https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html</a>.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_CIPHER_SERVER_PREFERENCE</code></td>
    <td>Attempts to use the server's preferences instead of the client's when
    selecting a cipher. Behavior depends on protocol version. See
    <a href="https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html">https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html</a>.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_CISCO_ANYCONNECT</code></td>
    <td>Instructs OpenSSL to use Cisco's "speshul" version of DTLS_BAD_VER.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_COOKIE_EXCHANGE</code></td>
    <td>Instructs OpenSSL to turn on cookie exchange.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_CRYPTOPRO_TLSEXT_BUG</code></td>
    <td>Instructs OpenSSL to add server-hello extension from an early version
    of the cryptopro draft.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_DONT_INSERT_EMPTY_FRAGMENTS</code></td>
    <td>Instructs OpenSSL to disable a SSL 3.0/TLS 1.0 vulnerability
    workaround added in OpenSSL 0.9.6d.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_EPHEMERAL_RSA</code></td>
    <td>Instructs OpenSSL to always use the tmp_rsa key when performing RSA
    operations.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_LEGACY_SERVER_CONNECT</code></td>
    <td>Allows initial connection to servers that do not support RI.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_MICROSOFT_BIG_SSLV3_BUFFER</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_MICROSOFT_SESS_ID_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_MSIE_SSLV2_RSA_PADDING</code></td>
    <td>Instructs OpenSSL to disable the workaround for a man-in-the-middle
    protocol-version vulnerability in the SSL 2.0 server implementation.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_CA_DN_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_CHALLENGE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_DEMO_CIPHER_CHANGE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_REUSE_CIPHER_CHANGE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_COMPRESSION</code></td>
    <td>Instructs OpenSSL to disable support for SSL/TLS compression.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_ENCRYPT_THEN_MAC</code></td>
    <td>Instructs OpenSSL to disable encrypt-then-MAC.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_QUERY_MTU</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_RENEGOTIATION</code></td>
    <td>Instructs OpenSSL to disable renegotiation.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_SESSION_RESUMPTION_ON_RENEGOTIATION</code></td>
    <td>Instructs OpenSSL to always start a new session when performing
    renegotiation.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_SSLv2</code></td>
    <td>Instructs OpenSSL to turn off SSL v2</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_SSLv3</code></td>
    <td>Instructs OpenSSL to turn off SSL v3</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TICKET</code></td>
    <td>Instructs OpenSSL to disable use of RFC4507bis tickets.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1</code></td>
    <td>Instructs OpenSSL to turn off TLS v1</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1_1</code></td>
    <td>Instructs OpenSSL to turn off TLS v1.1</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1_2</code></td>
    <td>Instructs OpenSSL to turn off TLS v1.2</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1_3</code></td>
    <td>Instructs OpenSSL to turn off TLS v1.3</td>
  </tr>
  <tr>
    <td><code>SSL_OP_PKCS1_CHECK_1</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_PKCS1_CHECK_2</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_PRIORITIZE_CHACHA</code></td>
    <td>Instructs OpenSSL server to prioritize ChaCha20-Poly1305
    when the client does.
    This option has no effect if
    <code>SSL_OP_CIPHER_SERVER_PREFERENCE</code>
    is not enabled.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_SINGLE_DH_USE</code></td>
    <td>Instructs OpenSSL to always create a new key when using
    temporary/ephemeral DH parameters.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_SINGLE_ECDH_USE</code></td>
    <td>Instructs OpenSSL to always create a new key when using
    temporary/ephemeral ECDH parameters.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_SSLEAY_080_CLIENT_DH_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_SSLREF2_REUSE_CERT_TYPE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_TLS_BLOCK_PADDING_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_TLS_D5_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_TLS_ROLLBACK_BUG</code></td>
    <td>Instructs OpenSSL to disable version rollback attack detection.</td>
  </tr>
</table>

### OpenSSL 引擎常量

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_RSA</code></td>
    <td>Limit engine usage to RSA</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_DSA</code></td>
    <td>Limit engine usage to DSA</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_DH</code></td>
    <td>Limit engine usage to DH</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_RAND</code></td>
    <td>Limit engine usage to RAND</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_EC</code></td>
    <td>Limit engine usage to EC</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_CIPHERS</code></td>
    <td>Limit engine usage to CIPHERS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_DIGESTS</code></td>
    <td>Limit engine usage to DIGESTS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_PKEY_METHS</code></td>
    <td>Limit engine usage to PKEY_METHDS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_PKEY_ASN1_METHS</code></td>
    <td>Limit engine usage to PKEY_ASN1_METHS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_ALL</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_NONE</code></td>
    <td></td>
  </tr>
</table>

### 其他 OpenSSL 常量

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>DH_CHECK_P_NOT_SAFE_PRIME</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>DH_CHECK_P_NOT_PRIME</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>DH_UNABLE_TO_CHECK_GENERATOR</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>DH_NOT_SUITABLE_GENERATOR</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>ALPN_ENABLED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PKCS1_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_SSLV23_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_NO_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PKCS1_OAEP_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_X931_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PKCS1_PSS_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PSS_SALTLEN_DIGEST</code></td>
    <td>Sets the salt length for <code>RSA_PKCS1_PSS_PADDING</code> to the
        digest size when signing or verifying.</td>
  </tr>
  <tr>
    <td><code>RSA_PSS_SALTLEN_MAX_SIGN</code></td>
    <td>Sets the salt length for <code>RSA_PKCS1_PSS_PADDING</code> to the
        maximum permissible value when signing data.</td>
  </tr>
  <tr>
    <td><code>RSA_PSS_SALTLEN_AUTO</code></td>
    <td>Causes the salt length for <code>RSA_PKCS1_PSS_PADDING</code> to be
        determined automatically when verifying a signature.</td>
  </tr>
  <tr>
    <td><code>POINT_CONVERSION_COMPRESSED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>POINT_CONVERSION_UNCOMPRESSED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>POINT_CONVERSION_HYBRID</code></td>
    <td></td>
  </tr>
</table>

### 节点.js加密常量

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>defaultCoreCipherList</code></td>
    <td>Specifies the built-in default cipher list used by Node.js.</td>
  </tr>
  <tr>
    <td><code>defaultCipherList</code></td>
    <td>Specifies the active default cipher list used by the current Node.js
    process.</td>
  </tr>
</table>

[AEAD algorithms]: https://en.wikipedia.org/wiki/Authenticated_encryption

[CCM mode]: #ccm-mode

[CVE-2021-44532]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44532

[Caveats]: #support-for-weak-or-compromised-algorithms

[Crypto constants]: #crypto-constants

[HTML 5.2]: https://www.w3.org/TR/html52/changes.html#features-removed

[HTML5's `keygen` element]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen

[JWK]: https://tools.ietf.org/html/rfc7517

[NIST SP 800-131A]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-131Ar1.pdf

[NIST SP 800-132]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf

[NIST SP 800-38D]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf

[Nonce-Disrespecting Adversaries]: https://github.com/nonce-disrespect/nonce-disrespect

[OpenSSL's SPKAC implementation]: https://www.openssl.org/docs/man1.1.0/apps/openssl-spkac.html

[RFC 1421]: https://www.rfc-editor.org/rfc/rfc1421.txt

[RFC 2409]: https://www.rfc-editor.org/rfc/rfc2409.txt

[RFC 2412]: https://www.rfc-editor.org/rfc/rfc2412.txt

[RFC 2818]: https://www.rfc-editor.org/rfc/rfc2818.txt

[RFC 3526]: https://www.rfc-editor.org/rfc/rfc3526.txt

[RFC 3610]: https://www.rfc-editor.org/rfc/rfc3610.txt

[RFC 4055]: https://www.rfc-editor.org/rfc/rfc4055.txt

[RFC 4122]: https://www.rfc-editor.org/rfc/rfc4122.txt

[RFC 5208]: https://www.rfc-editor.org/rfc/rfc5208.txt

[RFC 5280]: https://www.rfc-editor.org/rfc/rfc5280.txt

[Web Crypto API documentation]: webcrypto.md

[`BN_is_prime_ex`]: https://www.openssl.org/docs/man1.1.1/man3/BN_is_prime_ex.html

[`Buffer`]: buffer.md

[`EVP_BytesToKey`]: https://www.openssl.org/docs/man1.1.0/crypto/EVP_BytesToKey.html

[`KeyObject`]: #class-keyobject

[`Sign`]: #class-sign

[`String.prototype.normalize()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/normalize

[`UV_THREADPOOL_SIZE`]: cli.md#uv_threadpool_sizesize

[`Verify`]: #class-verify

[`cipher.final()`]: #cipherfinaloutputencoding

[`cipher.update()`]: #cipherupdatedata-inputencoding-outputencoding

[`crypto.createCipher()`]: #cryptocreatecipheralgorithm-password-options

[`crypto.createCipheriv()`]: #cryptocreatecipherivalgorithm-key-iv-options

[`crypto.createDecipher()`]: #cryptocreatedecipheralgorithm-password-options

[`crypto.createDecipheriv()`]: #cryptocreatedecipherivalgorithm-key-iv-options

[`crypto.createDiffieHellman()`]: #cryptocreatediffiehellmanprime-primeencoding-generator-generatorencoding

[`crypto.createECDH()`]: #cryptocreateecdhcurvename

[`crypto.createHash()`]: #cryptocreatehashalgorithm-options

[`crypto.createHmac()`]: #cryptocreatehmacalgorithm-key-options

[`crypto.createPrivateKey()`]: #cryptocreateprivatekeykey

[`crypto.createPublicKey()`]: #cryptocreatepublickeykey

[`crypto.createSecretKey()`]: #cryptocreatesecretkeykey-encoding

[`crypto.createSign()`]: #cryptocreatesignalgorithm-options

[`crypto.createVerify()`]: #cryptocreateverifyalgorithm-options

[`crypto.getCurves()`]: #cryptogetcurves

[`crypto.getDiffieHellman()`]: #cryptogetdiffiehellmangroupname

[`crypto.getHashes()`]: #cryptogethashes

[`crypto.privateDecrypt()`]: #cryptoprivatedecryptprivatekey-buffer

[`crypto.privateEncrypt()`]: #cryptoprivateencryptprivatekey-buffer

[`crypto.publicDecrypt()`]: #cryptopublicdecryptkey-buffer

[`crypto.publicEncrypt()`]: #cryptopublicencryptkey-buffer

[`crypto.randomBytes()`]: #cryptorandombytessize-callback

[`crypto.randomFill()`]: #cryptorandomfillbuffer-offset-size-callback

[`crypto.scrypt()`]: #cryptoscryptpassword-salt-keylen-options-callback

[`crypto.webcrypto.getRandomValues()`]: webcrypto.md#cryptogetrandomvaluestypedarray

[`crypto.webcrypto.subtle`]: webcrypto.md#class-subtlecrypto

[`decipher.final()`]: #decipherfinaloutputencoding

[`decipher.update()`]: #decipherupdatedata-inputencoding-outputencoding

[`diffieHellman.setPublicKey()`]: #diffiehellmansetpublickeypublickey-encoding

[`ecdh.generateKeys()`]: #ecdhgeneratekeysencoding-format

[`ecdh.setPrivateKey()`]: #ecdhsetprivatekeyprivatekey-encoding

[`hash.digest()`]: #hashdigestencoding

[`hash.update()`]: #hashupdatedata-inputencoding

[`hmac.digest()`]: #hmacdigestencoding

[`hmac.update()`]: #hmacupdatedata-inputencoding

[`import()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import

[`keyObject.export()`]: #keyobjectexportoptions

[`postMessage()`]: worker_threads.md#portpostmessagevalue-transferlist

[`sign.sign()`]: #signsignprivatekey-outputencoding

[`sign.update()`]: #signupdatedata-inputencoding

[`stream.Writable` options]: stream.md#new-streamwritableoptions

[`stream.transform` options]: stream.md#new-streamtransformoptions

[`util.promisify()`]: util.md#utilpromisifyoriginal

[`verify.update()`]: #verifyupdatedata-inputencoding

[`verify.verify()`]: #verifyverifyobject-signature-signatureencoding

[`x509.fingerprint256`]: #x509fingerprint256

[caveats when using strings as inputs to cryptographic APIs]: #using-strings-as-inputs-to-cryptographic-apis

[certificate object]: tls.md#certificate-object

[encoding]: buffer.md#buffers-and-character-encodings

[initialization vector]: https://en.wikipedia.org/wiki/Initialization_vector

[legacy provider]: cli.md#--openssl-legacy-provider

[list of SSL OP Flags]: https://wiki.openssl.org/index.php/List_of_SSL_OP_Flags#Table_of_Options

[modulo bias]: https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#Modulo_bias

[safe integers]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/isSafeInteger

[scrypt]: https://en.wikipedia.org/wiki/Scrypt

[stream]: stream.md

[stream-writable-write]: stream.md#writablewritechunk-encoding-callback
