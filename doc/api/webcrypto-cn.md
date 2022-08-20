# Web Crypto API

<!-- YAML
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43310
    description: Removed proprietary `'node.keyObject'` import/export format.
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/43310
    description: Removed proprietary `'NODE-DSA'`, `'NODE-DH'`,
      and `'NODE-SCRYPT'` algorithms.
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'Ed25519'`, `'Ed448'`, `'X25519'`, and `'X448'`
      algorithms.
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Removed proprietary `'NODE-ED25519'` and `'NODE-ED448'`
      algorithms.
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Removed proprietary `'NODE-X25519'` and `'NODE-X448'` named
      curves from the `'ECDH'` algorithm.
-->

<!-- introduced_in=v15.0.0 -->

> Stability: 1 - Experimental

Node.js 提供了标准 [Web Crypto API][] 的实现.

使用 `require('node:crypto').webcrypto` 来访问这个模块.

```js
const { subtle } = require('node:crypto').webcrypto;

(async function() {

  const key = await subtle.generateKey({
    name: 'HMAC',
    hash: 'SHA-256',
    length: 256
  }, true, ['sign', 'verify']);

  const enc = new TextEncoder();
  const message = enc.encode('I love cupcakes');

  const digest = await subtle.sign({
    name: 'HMAC'
  }, key, message);

})();
```

## Examples

### Generating keys

{SubtleCrypto} 类可用于生成对称（秘密）密钥或非对称密钥对（公钥和私钥）.

#### AES keys

```js
const { subtle } = require('node:crypto').webcrypto;

async function generateAesKey(length = 256) {
  const key = await subtle.generateKey({
    name: 'AES-CBC',
    length
  }, true, ['encrypt', 'decrypt']);

  return key;
}
```

#### ECDSA key pairs

```js
const { subtle } = require('node:crypto').webcrypto;

async function generateEcKey(namedCurve = 'P-521') {
  const {
    publicKey,
    privateKey
  } = await subtle.generateKey({
    name: 'ECDSA',
    namedCurve,
  }, true, ['sign', 'verify']);

  return { publicKey, privateKey };
}
```

#### Ed25519/Ed448/X25519/X448 key pairs

> Stability: 1 - Experimental

```js
const { subtle } = require('node:crypto').webcrypto;

async function generateEd25519Key() {
  return subtle.generateKey({
    name: 'Ed25519',
  }, true, ['sign', 'verify']);
}

async function generateX25519Key() {
  return subtle.generateKey({
    name: 'X25519',
  }, true, ['deriveKey']);
}
```

#### HMAC keys

```js
const { subtle } = require('node:crypto').webcrypto;

async function generateHmacKey(hash = 'SHA-256') {
  const key = await subtle.generateKey({
    name: 'HMAC',
    hash
  }, true, ['sign', 'verify']);

  return key;
}
```

#### RSA key pairs

```js
const { subtle } = require('node:crypto').webcrypto;
const publicExponent = new Uint8Array([1, 0, 1]);

async function generateRsaKey(modulusLength = 2048, hash = 'SHA-256') {
  const {
    publicKey,
    privateKey
  } = await subtle.generateKey({
    name: 'RSASSA-PKCS1-v1_5',
    modulusLength,
    publicExponent,
    hash,
  }, true, ['sign', 'verify']);

  return { publicKey, privateKey };
}
```

### Encryption and decryption

```js
const crypto = require('node:crypto').webcrypto;

async function aesEncrypt(plaintext) {
  const ec = new TextEncoder();
  const key = await generateAesKey();
  const iv = crypto.getRandomValues(new Uint8Array(16));

  const ciphertext = await crypto.subtle.encrypt({
    name: 'AES-CBC',
    iv,
  }, key, ec.encode(plaintext));

  return {
    key,
    iv,
    ciphertext
  };
}

async function aesDecrypt(ciphertext, key, iv) {
  const dec = new TextDecoder();
  const plaintext = await crypto.subtle.decrypt({
    name: 'AES-CBC',
    iv,
  }, key, ciphertext);

  return dec.decode(plaintext);
}
```

### Exporting and importing keys

```js
const { subtle } = require('node:crypto').webcrypto;

async function generateAndExportHmacKey(format = 'jwk', hash = 'SHA-512') {
  const key = await subtle.generateKey({
    name: 'HMAC',
    hash
  }, true, ['sign', 'verify']);

  return subtle.exportKey(format, key);
}

async function importHmacKey(keyData, format = 'jwk', hash = 'SHA-512') {
  const key = await subtle.importKey(format, keyData, {
    name: 'HMAC',
    hash
  }, true, ['sign', 'verify']);

  return key;
}
```

### Wrapping and unwrapping keys

```js
const { subtle } = require('node:crypto').webcrypto;

async function generateAndWrapHmacKey(format = 'jwk', hash = 'SHA-512') {
  const [
    key,
    wrappingKey,
  ] = await Promise.all([
    subtle.generateKey({
      name: 'HMAC', hash
    }, true, ['sign', 'verify']),
    subtle.generateKey({
      name: 'AES-KW',
      length: 256
    }, true, ['wrapKey', 'unwrapKey']),
  ]);

  const wrappedKey = await subtle.wrapKey(format, key, wrappingKey, 'AES-KW');

  return { wrappedKey, wrappingKey };
}

async function unwrapHmacKey(
  wrappedKey,
  wrappingKey,
  format = 'jwk',
  hash = 'SHA-512') {

  const key = await subtle.unwrapKey(
    format,
    wrappedKey,
    wrappingKey,
    'AES-KW',
    { name: 'HMAC', hash },
    true,
    ['sign', 'verify']);

  return key;
}
```

### Sign and verify

```js
const { subtle } = require('node:crypto').webcrypto;

async function sign(key, data) {
  const ec = new TextEncoder();
  const signature =
    await subtle.sign('RSASSA-PKCS1-v1_5', key, ec.encode(data));
  return signature;
}

async function verify(key, signature, data) {
  const ec = new TextEncoder();
  const verified =
    await subtle.verify(
      'RSASSA-PKCS1-v1_5',
      key,
      signature,
      ec.encode(data));
  return verified;
}
```

### Deriving bits and keys

```js
const { subtle } = require('node:crypto').webcrypto;

async function pbkdf2(pass, salt, iterations = 1000, length = 256) {
  const ec = new TextEncoder();
  const key = await subtle.importKey(
    'raw',
    ec.encode(pass),
    'PBKDF2',
    false,
    ['deriveBits']);
  const bits = await subtle.deriveBits({
    name: 'PBKDF2',
    hash: 'SHA-512',
    salt: ec.encode(salt),
    iterations
  }, key, length);
  return bits;
}

async function pbkdf2Key(pass, salt, iterations = 1000, length = 256) {
  const ec = new TextEncoder();
  const keyMaterial = await subtle.importKey(
    'raw',
    ec.encode(pass),
    'PBKDF2',
    false,
    ['deriveKey']);
  const key = await subtle.deriveKey({
    name: 'PBKDF2',
    hash: 'SHA-512',
    salt: ec.encode(salt),
    iterations
  }, keyMaterial, {
    name: 'AES-GCM',
    length: 256
  }, true, ['encrypt', 'decrypt']);
  return key;
}
```

### Digest

```js
const { subtle } = require('node:crypto').webcrypto;

async function digest(data, algorithm = 'SHA-512') {
  const ec = new TextEncoder();
  const digest = await subtle.digest(algorithm, ec.encode(data));
  return digest;
}
```

## Algorithm matrix

该表详细介绍了 Node.js Web Crypto API 实现支持的算法以及每种算法支持的 API:

| Algorithm             | `generateKey` | `exportKey` | `importKey` | `encrypt` | `decrypt` | `wrapKey` | `unwrapKey` | `deriveBits` | `deriveKey` | `sign` | `verify` | `digest` |
| --------------------- | ------------- | ----------- | ----------- | --------- | --------- | --------- | ----------- | ------------ | ----------- | ------ | -------- | -------- |
| `'RSASSA-PKCS1-v1_5'` | ✔             | ✔           | ✔           |           |           |           |             |              |             | ✔      | ✔        |          |
| `'RSA-PSS'`           | ✔             | ✔           | ✔           |           |           |           |             |              |             | ✔      | ✔        |          |
| `'RSA-OAEP'`          | ✔             | ✔           | ✔           | ✔         | ✔         | ✔         | ✔           |              |             |        |          |          |
| `'ECDSA'`             | ✔             | ✔           | ✔           |           |           |           |             |              |             | ✔      | ✔        |          |
| `'Ed25519'`[^1]       | ✔             | ✔           | ✔           |           |           |           |             |              |             | ✔      | ✔        |          |
| `'Ed448'`[^1]         | ✔             | ✔           | ✔           |           |           |           |             |              |             | ✔      | ✔        |          |
| `'ECDH'`              | ✔             | ✔           | ✔           |           |           |           |             | ✔            | ✔           |        |          |          |
| `'X25519'`[^1]        | ✔             | ✔           | ✔           |           |           |           |             | ✔            | ✔           |        |          |          |
| `'X448'`[^1]          | ✔             | ✔           | ✔           |           |           |           |             | ✔            | ✔           |        |          |          |
| `'AES-CTR'`           | ✔             | ✔           | ✔           | ✔         | ✔         | ✔         | ✔           |              |             |        |          |          |
| `'AES-CBC'`           | ✔             | ✔           | ✔           | ✔         | ✔         | ✔         | ✔           |              |             |        |          |          |
| `'AES-GCM'`           | ✔             | ✔           | ✔           | ✔         | ✔         | ✔         | ✔           |              |             |        |          |          |
| `'AES-KW'`            | ✔             | ✔           | ✔           |           |           | ✔         | ✔           |              |             |        |          |          |
| `'HMAC'`              | ✔             | ✔           | ✔           |           |           |           |             |              |             | ✔      | ✔        |          |
| `'HKDF'`              |               | ✔           | ✔           |           |           |           |             | ✔            | ✔           |        |          |          |
| `'PBKDF2'`            |               | ✔           | ✔           |           |           |           |             | ✔            | ✔           |        |          |          |
| `'SHA-1'`             |               |             |             |           |           |           |             |              |             |        |          | ✔        |
| `'SHA-256'`           |               |             |             |           |           |           |             |              |             |        |          | ✔        |
| `'SHA-384'`           |               |             |             |           |           |           |             |              |             |        |          | ✔        |
| `'SHA-512'`           |               |             |             |           |           |           |             |              |             |        |          | ✔        |

## Class: `Crypto`

<!-- YAML
added: v15.0.0
-->

调用 `require('node:crypto').webcrypto` 返回 `Crypto` 类的实例。 `Crypto` 是一个单例，提供对加密 API 其余部分的访问.

### `crypto.subtle`

<!-- YAML
added: v15.0.0
-->

* Type: {SubtleCrypto}

提供对“SubtleCrypto”API 的访问.

### `crypto.getRandomValues(typedArray)`

<!-- YAML
added: v15.0.0
-->

* `typedArray` {Buffer|TypedArray}
* Returns: {Buffer|TypedArray}

生成加密的强随机值。给定的 `typedArray` 填充随机值，并返回对 `typedArray` 的引用.

给定的 `typedArray` 必须是基于整数的 {TypedArray} 实例，即不接受 `Float32Array` 和 `Float64Array`.

如果给定的 `typedArray` 大于 65,536 字节，将引发错误.

### `crypto.randomUUID()`

<!-- YAML
added: v16.7.0
-->

* Returns: {string}

生成一个随机的 [RFC 4122][] 版本 4 UUID。 UUID 是使用加密伪随机数生成器生成的.

## Class: `CryptoKey`

<!-- YAML
added: v15.0.0
-->

### `cryptoKey.algorithm`

<!-- YAML
added: v15.0.0
-->

<!--lint disable maximum-line-length remark-lint-->

* Type: {AesKeyGenParams|RsaHashedKeyGenParams|EcKeyGenParams|HmacKeyGenParams}

<!--lint enable maximum-line-length remark-lint-->

一个对象，详细说明可以使用密钥的算法以及其他特定于算法的参数.

Read-only.

### `cryptoKey.extractable`

<!-- YAML
added: v15.0.0
-->

* Type: {boolean}

当 `true` 时，可以使用 `subtleCrypto.exportKey()` 或 `subtleCrypto.wrapKey()` 提取 {CryptoKey}.

Read-only.

### `cryptoKey.type`

<!-- YAML
added: v15.0.0
-->

* Type: {string} One of `'secret'`, `'private'`, or `'public'`.

标识密钥是对称（`'secret'`）还是非对称（`'private'` 或 `'public'`）密钥的字符串.

### `cryptoKey.usages`

<!-- YAML
added: v15.0.0
-->

* Type: {string\[]}

标识可以使用密钥的操作的字符串数组.

The possible usages are:

* `'encrypt'` - The key may be used to encrypt data.
* `'decrypt'` - The key may be used to decrypt data.
* `'sign'` - The key may be used to generate digital signatures.
* `'verify'` - The key may be used to verify digital signatures.
* `'deriveKey'` - The key may be used to derive a new key.
* `'deriveBits'` - The key may be used to derive bits.
* `'wrapKey'` - The key may be used to wrap another key.
* `'unwrapKey'` - The key may be used to unwrap another key.

Valid key usages depend on the key algorithm (identified by `cryptokey.algorithm.name`).

| Key Type              | `'encrypt'` | `'decrypt'` | `'sign'` | `'verify'` | `'deriveKey'` | `'deriveBits'` | `'wrapKey'` | `'unwrapKey'` |
| --------------------- | ----------- | ----------- | -------- | ---------- | ------------- | -------------- | ----------- | ------------- |
| `'AES-CBC'`           | ✔           | ✔           |          |            |               |                | ✔           | ✔             |
| `'AES-CTR'`           | ✔           | ✔           |          |            |               |                | ✔           | ✔             |
| `'AES-GCM'`           | ✔           | ✔           |          |            |               |                | ✔           | ✔             |
| `'AES-KW'`            |             |             |          |            |               |                | ✔           | ✔             |
| `'ECDH'`              |             |             |          |            | ✔             | ✔              |             |               |
| `'X25519'`[^1]        |             |             |          |            | ✔             | ✔              |             |               |
| `'X448'`[^1]          |             |             |          |            | ✔             | ✔              |             |               |
| `'ECDSA'`             |             |             | ✔        | ✔          |               |                |             |               |
| `'Ed25519'`[^1]       |             |             | ✔        | ✔          |               |                |             |               |
| `'Ed448'`[^1]         |             |             | ✔        | ✔          |               |                |             |               |
| `'HDKF'`              |             |             |          |            | ✔             | ✔              |             |               |
| `'HMAC'`              |             |             | ✔        | ✔          |               |                |             |               |
| `'PBKDF2'`            |             |             |          |            | ✔             | ✔              |             |               |
| `'RSA-OAEP'`          | ✔           | ✔           |          |            |               |                | ✔           | ✔             |
| `'RSA-PSS'`           |             |             | ✔        | ✔          |               |                |             |               |
| `'RSASSA-PKCS1-v1_5'` |             |             | ✔        | ✔          |               |                |             |               |

## Class: `CryptoKeyPair`

<!-- YAML
added: v15.0.0
-->

`CryptoKeyPair` 是一个简单的字典对象，具有 `publicKey` 和 `privateKey` 属性，表示非对称密钥对.

### `cryptoKeyPair.privateKey`

<!-- YAML
added: v15.0.0
-->

* Type: {CryptoKey} A {CryptoKey} whose `type` will be `'private'`.

### `cryptoKeyPair.publicKey`

<!-- YAML
added: v15.0.0
-->

* Type: {CryptoKey} A {CryptoKey} whose `type` will be `'public'`.

## Class: `SubtleCrypto`

<!-- YAML
added: v15.0.0
-->

### `subtle.decrypt(algorithm, key, data)`

<!-- YAML
added: v15.0.0
-->

* `algorithm`: {RsaOaepParams|AesCtrParams|AesCbcParams|AesGcmParams}
* `key`: {CryptoKey}
* `data`: {ArrayBuffer|TypedArray|DataView|Buffer}
* Returns: {Promise} containing {ArrayBuffer}

使用 `algorithm` 中指定的方法和参数以及 `key` 提供的密钥材料，`subtle.decrypt()` 尝试破译提供的 `data`。如果成功，返回的 Promise 将使用包含明文结果的 {ArrayBuffer} 解析.

目前支持的算法包括:

* `'RSA-OAEP'`
* `'AES-CTR'`
* `'AES-CBC'`
* `'AES-GCM`'

### `subtle.deriveBits(algorithm, baseKey, length)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'X25519'`, and `'X448'` algorithms.
-->

<!--lint disable maximum-line-length remark-lint-->

* `algorithm`: {AlgorithmIdentifier|EcdhKeyDeriveParams|HkdfParams|Pbkdf2Params}
* `baseKey`: {CryptoKey}
* `length`: {number}
* Returns: {Promise} containing {ArrayBuffer}

<!--lint enable maximum-line-length remark-lint-->

使用 `algorithm` 中指定的方法和参数以及 `baseKey` 提供的密钥材料，`subtle.deriveBits()` 尝试生成 `length` 位。 Node.js 实现要求 `length` 是 `8` 的倍数。如果成功，返回的 Promise 将使用包含生成数据的 {ArrayBuffer} 解析.

目前支持的算法包括:

* `'ECDH'`
* `'HKDF'`
* `'PBKDF2'`

### `subtle.deriveKey(algorithm, baseKey, derivedKeyAlgorithm, extractable, keyUsages)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'X25519'`, and `'X448'` algorithms.
-->

<!--lint disable maximum-line-length remark-lint-->

* `algorithm`: {AlgorithmIdentifier|EcdhKeyDeriveParams|HkdfParams|Pbkdf2Params}
* `baseKey`: {CryptoKey}
* `derivedKeyAlgorithm`: {HmacKeyGenParams|AesKeyGenParams}
* `extractable`: {boolean}
* `keyUsages`: {string\[]} See [Key usages][].
* Returns: {Promise} containing {CryptoKey}

<!--lint enable maximum-line-length remark-lint-->

使用 `algorithm` 中指定的方法和参数，以及 `baseKey` 提供的密钥材料，`subtle.deriveKey()` 尝试根据 `derivedKeyAlgorithm` 中的方法和参数生成一个新的 {CryptoKey}.

调用 `subtle.deriveKey()` 等效于调用 `subtle.deriveBits()` 来生成原始键控材料，然后使用 `deriveKeyAlgorithm`、`extractable` 和`keyUsages` 参数作为输入.

目前支持的算法包括:

* `'ECDH'`
* `'HKDF'`
* `'PBKDF2'`

### `subtle.digest(algorithm, data)`

<!-- YAML
added: v15.0.0
-->

* `algorithm`: {string|Object}
* `data`: {ArrayBuffer|TypedArray|DataView|Buffer}
* Returns: {Promise} containing {ArrayBuffer}

使用 `algorithm` 标识的方法，`subtle.digest()` 尝试生成 `data` 的摘要。如果成功，返回的 Promise 将使用包含计算摘要的 {ArrayBuffer} 解析.

如果 `algorithm` 作为 {string} 提供，它必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果 `algorithm` 作为 {Object} 提供，它必须有一个 `name` 属性，其值为上述之一.

### `subtle.encrypt(algorithm, key, data)`

<!-- YAML
added: v15.0.0
-->

* `algorithm`: {RsaOaepParams|AesCtrParams|AesCbcParams|AesGcmParams}
* `key`: {CryptoKey}
* Returns: {Promise} containing {ArrayBuffer}

使用 `algorithm` 指定的方法和参数以及 `key` 提供的密钥材料，`subtle.encrypt()` 尝试加密 `data`.
如果成功，返回的 Promise 将使用包含加密结果的 {ArrayBuffer} 解析.

目前支持的算法包括:

* `'RSA-OAEP'`
* `'AES-CTR'`
* `'AES-CBC'`
* `'AES-GCM`'

### `subtle.exportKey(format, key)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'Ed25519'`, `'Ed448'`, `'X25519'`, and `'X448'`
      algorithms.
  - version: v15.9.0
    pr-url: https://github.com/nodejs/node/pull/37203
    description: Removed `'NODE-DSA'` JWK export.
-->

* `format`: {string} Must be one of `'raw'`, `'pkcs8'`, `'spki'`, or `'jwk'`.
* `key`: {CryptoKey}
* Returns: {Promise} containing {ArrayBuffer}.

如果支持，将给定密钥导出为指定格式.

如果 {CryptoKey} 不可提取，返回的 Promise 将拒绝.

当 `format` 是 `'pkcs8'` 或 `'spki'` 并且导出成功时，返回的 Promise 将使用包含导出的关键数据的 {ArrayBuffer} 解析.

当 `format` 为 `'jwk'` 且导出成功时，返回的 Promise 将使用符合 [JSON Web Key][] 的 JavaScript 对象解析
规格.

| Key Type              | `'spki'` | `'pkcs8'` | `'jwk'` | `'raw'` |
| --------------------- | -------- | --------- | ------- | ------- |
| `'AES-CBC'`           |          |           | ✔       | ✔       |
| `'AES-CTR'`           |          |           | ✔       | ✔       |
| `'AES-GCM'`           |          |           | ✔       | ✔       |
| `'AES-KW'`            |          |           | ✔       | ✔       |
| `'ECDH'`              | ✔        | ✔         | ✔       | ✔       |
| `'ECDSA'`             | ✔        | ✔         | ✔       | ✔       |
| `'Ed25519'`[^1]       | ✔        | ✔         | ✔       | ✔       |
| `'Ed448'`[^1]         | ✔        | ✔         | ✔       | ✔       |
| `'HDKF'`              |          |           |         |         |
| `'HMAC'`              |          |           | ✔       | ✔       |
| `'PBKDF2'`            |          |           |         |         |
| `'RSA-OAEP'`          | ✔        | ✔         | ✔       |         |
| `'RSA-PSS'`           | ✔        | ✔         | ✔       |         |
| `'RSASSA-PKCS1-v1_5'` | ✔        | ✔         | ✔       |         |

### `subtle.generateKey(algorithm, extractable, keyUsages)`

<!-- YAML
added: v15.0.0
-->

<!--lint disable maximum-line-length remark-lint-->

* `algorithm`: {AlgorithmIdentifier|RsaHashedKeyGenParams|EcKeyGenParams|HmacKeyGenParams|AesKeyGenParams}

<!--lint enable maximum-line-length remark-lint-->

* `extractable`: {boolean}
* `keyUsages`: {string\[]} See [Key usages][].
* Returns: {Promise} containing {CryptoKey|CryptoKeyPair}

使用 `algorithm` 中提供的方法和参数，`subtle.generateKey()` 尝试生成新的密钥材料。根据使用的方法，该方法可以生成单个 {CryptoKey} 或 {CryptoKeyPair}.

支持的 {CryptoKeyPair}（公钥和私钥）生成算法包括:

* `'RSASSA-PKCS1-v1_5'`
* `'RSA-PSS'`
* `'RSA-OAEP'`
* `'ECDSA'`
* `'Ed25519'`[^1]
* `'Ed448'`[^1]
* `'ECDH'`
* `'X25519'`[^1]
* `'X448'`[^1]

支持的 {CryptoKey}（密钥）生成算法包括:

* `'HMAC'`
* `'AES-CTR'`
* `'AES-CBC'`
* `'AES-GCM'`
* `'AES-KW'`

### `subtle.importKey(format, keyData, algorithm, extractable, keyUsages)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'Ed25519'`, `'Ed448'`, `'X25519'`, and `'X448'`
      algorithms.
  - version: v15.9.0
    pr-url: https://github.com/nodejs/node/pull/37203
    description: Removed `'NODE-DSA'` JWK import.
-->

* `format`: {string} Must be one of `'raw'`, `'pkcs8'`, `'spki'`, or `'jwk'`.
* `keyData`: {ArrayBuffer|TypedArray|DataView|Buffer|KeyObject}

<!--lint disable maximum-line-length remark-lint-->

* `algorithm`: {AlgorithmIdentifier|RsaHashedImportParams|EcKeyImportParams|HmacImportParams}

<!--lint enable maximum-line-length remark-lint-->

* `extractable`: {boolean}
* `keyUsages`: {string\[]} See [Key usages][].
* Returns: {Promise} containing {CryptoKey}

`subtle.importKey()` 方法尝试将提供的 `keyData` 解释为给定的 `format` 以使用提供的创建 {CryptoKey} 实例
`algorithm`、`extractable` 和 `keyUsages` 参数。如果导入成功，返回的 Promise 将使用创建的 {CryptoKey} 解析.

如果导入 `'PBKDF2'` 密钥，`extractable` 必须为 `false`.

目前支持的算法包括:

| Key Type              | `'spki'` | `'pkcs8'` | `'jwk'` | `'raw'` |
| --------------------- | -------- | --------- | ------- | ------- |
| `'AES-CBC'`           |          |           | ✔       | ✔       |
| `'AES-CTR'`           |          |           | ✔       | ✔       |
| `'AES-GCM'`           |          |           | ✔       | ✔       |
| `'AES-KW'`            |          |           | ✔       | ✔       |
| `'ECDH'`              | ✔        | ✔         | ✔       | ✔       |
| `'X25519'`[^1]        | ✔        | ✔         | ✔       | ✔       |
| `'X448'`[^1]          | ✔        | ✔         | ✔       | ✔       |
| `'ECDSA'`             | ✔        | ✔         | ✔       | ✔       |
| `'Ed25519'`[^1]       | ✔        | ✔         | ✔       | ✔       |
| `'Ed448'`[^1]         | ✔        | ✔         | ✔       | ✔       |
| `'HDKF'`              |          |           |         | ✔       |
| `'HMAC'`              |          |           | ✔       | ✔       |
| `'PBKDF2'`            |          |           |         | ✔       |
| `'RSA-OAEP'`          | ✔        | ✔         | ✔       |         |
| `'RSA-PSS'`           | ✔        | ✔         | ✔       |         |
| `'RSASSA-PKCS1-v1_5'` | ✔        | ✔         | ✔       |         |

### `subtle.sign(algorithm, key, data)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'Ed25519'`, and `'Ed448'` algorithms.
-->

<!--lint disable maximum-line-length remark-lint-->

* `algorithm`: {AlgorithmIdentifier|RsaPssParams|EcdsaParams|Ed448Params}
* `key`: {CryptoKey}
* `data`: {ArrayBuffer|TypedArray|DataView|Buffer}
* Returns: {Promise} containing {ArrayBuffer}

<!--lint enable maximum-line-length remark-lint-->

使用 `algorithm` 给出的方法和参数以及 `key` 提供的密钥材料，`subtle.sign()` 尝试生成 `data` 的加密签名。如果成功，返回的 Promise 将使用包含生成签名的 {ArrayBuffer} 解析.

目前支持的算法包括:

* `'RSASSA-PKCS1-v1_5'`
* `'RSA-PSS'`
* `'ECDSA'`
* `'Ed25519'`[^1]
* `'Ed448'`[^1]
* `'HMAC'`

### `subtle.unwrapKey(format, wrappedKey, unwrappingKey, unwrapAlgo, unwrappedKeyAlgo, extractable, keyUsages)`

<!-- YAML
added: v15.0.0
-->

* `format`: {string} Must be one of `'raw'`, `'pkcs8'`, `'spki'`, or `'jwk'`.
* `wrappedKey`: {ArrayBuffer|TypedArray|DataView|Buffer}
* `unwrappingKey`: {CryptoKey}

<!--lint disable maximum-line-length remark-lint-->

* `unwrapAlgo`: {AlgorithmIdentifier|RsaOaepParams|AesCtrParams|AesCbcParams|AesGcmParams}
* `unwrappedKeyAlgo`: {AlgorithmIdentifier|RsaHashedImportParams|EcKeyImportParams|HmacImportParams}

<!--lint enable maximum-line-length remark-lint-->

* `extractable`: {boolean}
* `keyUsages`: {string\[]} See [Key usages][].
* Returns: {Promise} containing {CryptoKey}

在密码学中，“包装密钥”是指导出然后加密密钥材料。 `subtle.unwrapKey()` 方法尝试解密包装的密钥并创建一个 {CryptoKey} 实例。这相当于首先对加密的密钥数据调用 `subtle.decrypt()`（使用 `wrappedKey`、`unwrapAlgo` 和 `unwrappingKey` 参数作为输入），然后将结果传递给 `subtle.importKey()使用 `unwrappedKeyAlgo`、`extractable` 和 `keyUsages` 参数作为输入的方法。如果成功，返回的 Promise 将使用 {CryptoKey} 对象解析.

当前支持的包装算法包括:

* `'RSA-OAEP'`
* `'AES-CTR'`
* `'AES-CBC'`
* `'AES-GCM'`
* `'AES-KW'`

支持的解包密钥算法包括:

* `'RSASSA-PKCS1-v1_5'`
* `'RSA-PSS'`
* `'RSA-OAEP'`
* `'ECDSA'`
* `'ECDH'`
* `'HMAC'`
* `'AES-CTR'`
* `'AES-CBC'`
* `'AES-GCM'`
* `'AES-KW'`

### `subtle.verify(algorithm, key, signature, data)`

<!-- YAML
added: v15.0.0
changes:
  - version: v18.4.0
    pr-url: https://github.com/nodejs/node/pull/42507
    description: Added `'Ed25519'`, and `'Ed448'` algorithms.
-->

<!--lint disable maximum-line-length remark-lint-->

* `algorithm`: {AlgorithmIdentifier|RsaPssParams|EcdsaParams|Ed448Params}
* `key`: {CryptoKey}
* `signature`: {ArrayBuffer|TypedArray|DataView|Buffer}
* `data`: {ArrayBuffer|TypedArray|DataView|Buffer}
* Returns: {Promise} containing {boolean}

<!--lint enable maximum-line-length remark-lint-->

使用 `algorithm` 中给出的方法和参数以及 `key` 提供的密钥材料，`subtle.verify()` 尝试验证 `signature` 是 `data` 的有效加密签名。返回的 Promise 用 `true` 或 `false` 解决.

目前支持的算法包括:

* `'RSASSA-PKCS1-v1_5'`
* `'RSA-PSS'`
* `'ECDSA'`
* `'Ed25519'`[^1]
* `'Ed448'`[^1]
* `'HMAC'`

### `subtle.wrapKey(format, key, wrappingKey, wrapAlgo)`

<!-- YAML
added: v15.0.0
-->

<!--lint disable maximum-line-length remark-lint-->

* `format`: {string} Must be one of `'raw'`, `'pkcs8'`, `'spki'`, or `'jwk'`.
* `key`: {CryptoKey}
* `wrappingKey`: {CryptoKey}
* `wrapAlgo`: {AlgorithmIdentifier|RsaOaepParams|AesCtrParams|AesCbcParams|AesGcmParams}
* Returns: {Promise} containing {ArrayBuffer}

<!--lint enable maximum-line-length remark-lint-->

在密码学中，“包装密钥”是指导出然后加密密钥材料。 `subtle.wrapKey()` 方法将密钥材料导出为 `format` 标识的格式，然后使用 `wrapAlgo` 指定的方法和参数以及 `wrappingKey` 提供的密钥材料对其进行加密。这相当于使用 `format` 和 `key` 作为参数调用 `subtle.exportKey()`，然后使用 `wrappingKey` 和 `wrapAlgo` 作为输入将结果传递给 `subtle.encrypt()` 方法。如果成功，返回的 Promise 将使用包含加密密钥数据的 {ArrayBuffer} 解析.

当前支持的包装算法包括:

* `'RSA-OAEP'`
* `'AES-CTR'`
* `'AES-CBC'`
* `'AES-GCM'`
* `'AES-KW'`

## Algorithm parameters

The algorithm parameter objects define the methods and parameters used by the various {SubtleCrypto} methods. While described here as "classes", they are simple JavaScript dictionary objects.

### Class: `AlgorithmIdentifier`

<!-- YAML
added: v18.4.0
-->

#### `algorithmIdentifier.name`

<!-- YAML
added: v18.4.0
-->

* Type: {string}

### Class: `AesCbcParams`

<!-- YAML
added: v15.0.0
-->

#### `aesCbcParams.iv`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

提供初始化向量。它的长度必须正好是 16 字节，并且应该是不可预测的和加密随机的.

#### `aesCbcParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'AES-CBC'`.

### Class: `AesCtrParams`

<!-- YAML
added: v15.0.0
-->

#### `aesCtrParams.counter`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

计数器块的初始值。这必须正好是 16 个字节长.

`AES-CTR` 方法使用块的最右边的 `length` 位作为计数器，其余位作为 nonce.

#### `aesCtrParams.length`

<!-- YAML
added: v15.0.0
-->

* Type: {number} The number of bits in the `aesCtrParams.counter` that are
  to be used as the counter.

#### `aesCtrParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'AES-CTR'`.

### Class: `AesGcmParams`

<!-- YAML
added: v15.0.0
-->

#### `aesGcmParams.additionalData`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer|undefined}

使用 AES-GCM 方法，`additionalData` 是额外的输入，未加密但包含在数据的身份验证中。 `additionalData` 的使用是可选的.

#### `aesGcmParams.iv`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

对于使用给定密钥的每个加密操作，初始化向量必须是唯一的.

理想情况下，这是一个确定性的 12 字节值，其计算方式确保它在使用相同密钥的所有调用中是唯一的.
或者，初始化向量可以由至少 12 个密码随机字节组成。有关为 AES-GCM 构造初始化向量的更多信息，请参阅 [NIST SP 800-38D][] 的第 8 节.

#### `aesGcmParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'AES-GCM'`.

#### `aesGcmParams.tagLength`

<!-- YAML
added: v15.0.0
-->

* Type: {number} The size in bits of the generated authentication tag.
  This values must be one of `32`, `64`, `96`, `104`, `112`, `120`, or
  `128`. **Default:** `128`.

### Class: `AesKeyGenParams`

<!-- YAML
added: v15.0.0
-->

#### `aesKeyGenParams.length`

<!-- YAML
added: v15.0.0
-->

* Type: {number}

要生成的 AES 密钥的长度。这必须是“128”、“192”或“256”.

#### `aesKeyGenParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'AES-CBC'`, `'AES-CTR'`, `'AES-GCM'`, or
  `'AES-KW'`

### Class: `EcdhKeyDeriveParams`

<!-- YAML
added: v15.0.0
-->

#### `ecdhKeyDeriveParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'ECDH'`, `'X25519'`, or `'X448'`.

#### `ecdhKeyDeriveParams.public`

<!-- YAML
added: v15.0.0
-->

* Type: {CryptoKey}

ECDH 密钥派生通过将一方的私钥和另一方的公钥作为输入来操作——使用两者来生成一个共同的共享秘密.
`ecdhKeyDeriveParams.public` 属性设置为对方公钥.

### Class: `EcdsaParams`

<!-- YAML
added: v15.0.0
-->

#### `ecdsaParams.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `ecdsaParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'ECDSA'`.

### Class: `EcKeyGenParams`

<!-- YAML
added: v15.0.0
-->

#### `ecKeyGenParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'ECDSA'` or `'ECDH'`.

#### `ecKeyGenParams.namedCurve`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'P-256'`, `'P-384'`, `'P-521'`.

### Class: `EcKeyImportParams`

<!-- YAML
added: v15.0.0
-->

#### `ecKeyImportParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'ECDSA'` or `'ECDH'`.

#### `ecKeyImportParams.namedCurve`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'P-256'`, `'P-384'`, `'P-521'`.

### Class: `Ed448Params`

<!-- YAML
added: v15.0.0
-->

#### `ed448Params.name`

<!-- YAML
added: v18.4.0
-->

* Type: {string} Must be `'Ed448'`.

#### `ed448Params.context`

<!-- YAML
added: v18.4.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer|undefined}

`context` 成员表示与消息关联的可选上下文数据.
Node.js Web Crypto API 实现仅支持零长度上下文，这相当于根本不提供上下文.

### Class: `HkdfParams`

<!-- YAML
added: v15.0.0
-->

#### `hkdfParams.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `hkdfParams.info`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

为 HKDF 算法提供特定于应用程序的上下文输入.
这可以是零长度，但必须提供.

#### `hkdfParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'HKDF'`.

#### `hkdfParams.salt`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

salt值显着提高了HKDF算法的强度.
它应该是随机或伪随机的，并且应该与摘要函数的输出长度相同（例如，如果使用“SHA-256”作为摘要，则盐应该是 256 位随机数据​​）.

### Class: `HmacImportParams`

<!-- YAML
added: v15.0.0
-->

#### `hmacImportParams.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `hmacImportParams.length`

<!-- YAML
added: v15.0.0
-->

* Type: {number}

HMAC 密钥中的可选位数。这是可选的，在大多数情况下应该省略.

#### `hmacImportParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'HMAC'`.

### Class: `HmacKeyGenParams`

<!-- YAML
added: v15.0.0
-->

#### `hmacKeyGenParams.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `hmacKeyGenParams.length`

<!-- YAML
added: v15.0.0
-->

* Type: {number}

为 HMAC 密钥生成的位数。如果省略,
长度将由使用的哈希算法确定.
This is optional and should be omitted for most cases.

#### `hmacKeyGenParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'HMAC'`.

### Class: `Pbkdf2Params`

<!-- YAML
added: v15.0.0
-->

#### `pbkdb2Params.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `pbkdf2Params.iterations`

<!-- YAML
added: v15.0.0
-->

* Type: {number}

派生位时 PBKDF2 算法应进行的迭代次数.

#### `pbkdf2Params.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'PBKDF2'`.

#### `pbkdf2Params.salt`

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

Should be at least 16 random or pseudorandom bytes.

### Class: `RsaHashedImportParams`

<!-- YAML
added: v15.0.0
-->

#### `rsaHashedImportParams.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `rsaHashedImportParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'RSASSA-PKCS1-v1_5'`, `'RSA-PSS'`, or
  `'RSA-OAEP'`.

### Class: `RsaHashedKeyGenParams`

<!-- YAML
added: v15.0.0
-->

#### `rsaHashedKeyGenParams.hash`

<!-- YAML
added: v15.0.0
-->

* Type: {string|Object}

如果表示为 {string}，则值必须是以下之一:

* `'SHA-1'`
* `'SHA-256'`
* `'SHA-384'`
* `'SHA-512'`

如果表示为 {Object}，则该对象必须具有 `name` 属性，其值是上面列出的值之一.

#### `rsaHashedKeyGenParams.modulusLength`

<!-- YAML
added: v15.0.0
-->

* Type: {number}

The length in bits of the RSA modulus. As a best practice, this should be at least `2048`.

#### `rsaHashedKeyGenParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be one of `'RSASSA-PKCS1-v1_5'`, `'RSA-PSS'`, or
  `'RSA-OAEP'`.

#### `rsaHashedKeyGenParams.publicExponent`

<!-- YAML
added: v15.0.0
-->

* Type: {Uint8Array}

RSA 公共指数。这必须是一个 {Uint8Array} ，包含一个必须适合 32 位的大端、无符号整数。 {Uint8Array} 可以包含任意数量的前导零位。该值必须是质数。除非有理由使用不同的值，否则使用 `new Uint8Array([1, 0, 1])` (65537) 作为公共指数.

### Class: `RsaOaepParams`

<!-- YAML
added: v15.0.0
-->

#### rsaOaepParams.label

<!-- YAML
added: v15.0.0
-->

* Type: {ArrayBuffer|TypedArray|DataView|Buffer}

不会加密但将绑定到生成的密文的额外字节集合.

The `rsaOaepParams.label` parameter is optional.

#### rsaOaepParams.name

<!-- YAML
added: v15.0.0
-->

* Type: {string} must be `'RSA-OAEP'`.

### Class: `RsaPssParams`

<!-- YAML
added: v15.0.0
-->

#### `rsaPssParams.name`

<!-- YAML
added: v15.0.0
-->

* Type: {string} Must be `'RSA-PSS'`.

#### `rsaPssParams.saltLength`

<!-- YAML
added: v15.0.0
-->

* Type: {number}

The length (in bytes) of the random salt to use.

[^1]: An experimental implementation of
    [Secure Curves in the Web Cryptography API][] as of 05 May 2022

[JSON Web Key]: https://tools.ietf.org/html/rfc7517
[Key usages]: #cryptokeyusages
[NIST SP 800-38D]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf
[RFC 4122]: https://www.rfc-editor.org/rfc/rfc4122.txt
[Secure Curves in the Web Cryptography API]: https://wicg.github.io/webcrypto-secure-curves/
[Web Crypto API]: https://www.w3.org/TR/WebCryptoAPI/
