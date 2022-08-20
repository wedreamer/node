# 模块： ECMAScript 模块

<!--introduced_in=v8.5.0-->

<!-- type=misc -->

<!-- YAML
added: v8.5.0
changes:
  - version:
    - v18.6.0
    pr-url: https://github.com/nodejs/node/pull/42623
    description: Add support for chaining loaders.
  - version:
    - v17.1.0
    - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/40250
    description: Add support for import assertions.
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/37468
    description:
      Consolidate loader hooks, removed `getFormat`, `getSource`,
      `transformSource`, and `getGlobalPreloadCode` hooks
      added `load` and `globalPreload` hooks
      allowed returning `format` from either `resolve` or `load` hooks.
  - version:
    - v15.3.0
    - v14.17.0
    - v12.22.0
    pr-url: https://github.com/nodejs/node/pull/35781
    description: Stabilize modules implementation.
  - version:
    - v14.13.0
    - v12.20.0
    pr-url: https://github.com/nodejs/node/pull/35249
    description: Support for detection of CommonJS named exports.
  - version: v14.8.0
    pr-url: https://github.com/nodejs/node/pull/34558
    description: Unflag Top-Level Await.
  - version:
    - v14.0.0
    - v13.14.0
    - v12.20.0
    pr-url: https://github.com/nodejs/node/pull/31974
    description: Remove experimental modules warning.
  - version:
    - v13.2.0
    - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/29866
    description: Loading ECMAScript modules no longer requires a command-line flag.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26745
    description:
      Add support for ES modules using `.js` file extension via `package.json`
      `"type"` field.
-->

> 稳定性： 2 - 稳定

## 介绍

<!--name=esm-->

ECMAScript 模块是[官方标准格式][the official standard format]打包 JavaScript
用于重用的代码。使用各种[`import`][import]和
[`export`][export]语句。

以下 ES 模块示例导出函数：

```js
// addTwo.mjs
function addTwo(num) {
  return num + 2;
}

export { addTwo };
```

以下 ES 模块示例从`addTwo.mjs`:

```js
// app.mjs
import { addTwo } from './addTwo.mjs';

// Prints: 6
console.log(addTwo(4));
```

Node.js 完全支持当前指定的 ECMAScript 模块和
提供它们与其原始模块格式之间的互操作性，
[CommonJS][].

<!-- Anchors to make sure old links find a target -->

<i id="esm_package_json_type_field"></i><i id="esm_package_scope_and_file_extensions"></i><i id="esm_input_type_flag"></i>

## 使

<!-- type=misc -->

Node.js有两个模块系统：[CommonJS][]模块和 ECMAScript 模块。

作者可以告诉 Node.js使用 ECMAScript 模块加载程序
通过`.mjs`文件扩展名，`package.json` [`"type"`]["type"]字段，或
[`--input-type`][--input-type]旗。在这些情况下，Node.js将使用CommonJS。
模块加载程序。看[确定模块系统][Determining module system]了解更多详情。

<!-- Anchors to make sure old links find a target -->

<i id="esm_package_entry_points"></i><i id="esm_main_entry_point_export"></i><i id="esm_subpath_exports"></i><i id="esm_package_exports_fallbacks"></i><i id="esm_exports_sugar"></i><i id="esm_conditional_exports"></i><i id="esm_nested_conditions"></i><i id="esm_self_referencing_a_package_using_its_name"></i><i id="esm_internal_package_imports"></i><i id="esm_dual_commonjs_es_module_packages"></i><i id="esm_dual_package_hazard"></i><i id="esm_writing_dual_packages_while_avoiding_or_minimizing_hazards"></i><i id="esm_approach_1_use_an_es_module_wrapper"></i><i id="esm_approach_2_isolate_state"></i>

## 包

此部分已移至[模块： 包](packages.md).

## `import`说明符

### 术语

这*规范*的`import`语句是`from`关键词
例如：`'node:path'`在`import { sep } from 'node:path'`.说明符也是
用于`export from`语句，并作为参数`import()`
表达。

有三种类型的说明符：

*   *相对说明符*喜欢`'./startup.js'`或`'../config.mjs'`.他们提到
    到相对于导入文件位置的路径。*文件扩展名
    对于这些总是必要的。*

*   *裸指定符*喜欢`'some-package'`或`'some-package/shuffle'`.他们可以
    通过包名称引用包的主入口点，或
    包中的特定功能模块，以包名称为前缀，如下所示
    分别是示例。*仅需要包含文件扩展名
    对于没有[`"exports"`]["exports"]田。*

*   *绝对说明符*喜欢`'file:///opt/nodejs/config.js'`.他们提到
    直接显式地连接到完整路径。

裸说明符分辨率由[节点.js模块分辨率
算法][Node.js module resolution
algorithm].所有其他说明符分辨率始终仅使用
标准相对[网址][URL]解析语义。

与在 CommonJS 中一样，可以通过附加一个
包名称的路径，除非包的[`package.json`][package.json]包含一个
[`"exports"`]["exports"]字段，在这种情况下，只能访问包中的文件
通过 中定义的路径[`"exports"`]["exports"].

有关适用于裸说明符的这些包解析规则的详细信息，请参见
节点.js模块分辨率，请参阅[软件包文档](packages.md).

### 必需的文件扩展名

使用 时必须提供文件扩展名`import`要解决的关键字
相对或绝对说明符。目录索引（例如`'./startup/index.js'`)
还必须完全指定。

此行为与方式匹配`import`在浏览器环境中的行为，假设
通常配置的服务器。

### 网址

ES 模块被解析并缓存为 URL。这意味着特殊字符
必须是[百分比编码][percent-encoded]如`#`跟`%23`和`?`跟`%3F`.

`file:`,`node:`和`data:`支持 URL 方案。一个说明符，如
`'https://example.com/app.js'`在 Node 中不受本机支持.js除非使用
一个[自定义 HTTPS 加载程序][custom HTTPS loader].

#### `file:`网址

模块被加载多次，如果`import`用于解析的说明符
它们具有不同的查询或片段。

```js
import './foo.mjs?query=1'; // loads ./foo.mjs with query of "?query=1"
import './foo.mjs?query=2'; // loads ./foo.mjs with query of "?query=2"
```

卷根可以通过以下方式引用`/`,`//`或`file:///`.鉴于
差异[网址][URL]和路径分辨率（如百分比编码
细节），建议使用[url.pathToFileURL][]导入路径时。

#### `data:`进口

<!-- YAML
added: v12.10.0
-->

[`data:`网址][data: URLs]支持使用以下 MIME 类型导入：

*   `text/javascript`用于 ES 模块
*   `application/json`对于 JSON
*   `application/wasm`对于瓦斯姆

```js
import 'data:text/javascript,console.log("hello!");';
import _ from 'data:application/json,"world!"' assert { type: 'json' };
```

`data:`仅解析网址[裸说明符][Terminology]用于内置模块
和[绝对说明符][Terminology].解决
[相对说明符][Terminology]不起作用，因为`data:`不是
[特殊方案][special scheme].例如，尝试加载`./foo`
从`data:text/javascript,import "./foo";`无法解决，因为有
是 没有相对分辨率的概念`data:`网址。

#### `node:`进口

<!-- YAML
added:
  - v14.13.1
  - v12.20.0
changes:
  - version:
      - v16.0.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37246
    description: Added `node:` import support to `require(...)`.
-->

`node:`支持将 URL 作为加载 Node.js 内置的替代方法
模块。此 URL 方案允许有效引用内置模块
绝对网址字符串。

```js
import fs from 'node:fs/promises';
```

## 导入断言

<!-- YAML
added:
  - v17.1.0
  - v16.14.0
-->

> 稳定性： 1 - 实验

这[导入断言建议][Import Assertions proposal]为模块导入添加内联语法
语句，用于将更多信息与模块说明符一起传递。

```js
import fooData from './foo.json' assert { type: 'json' };

const { default: barData } =
  await import('./bar.json', { assert: { type: 'json' } });
```

节点.js支持以下`type`值，其断言为
命令的：

|断言`type`||需要
|---------------- |---------------- |
|`'json'`|[JSON 模块][JSON modules]|

## 内置模块

[核心模块][Core modules]提供其公共 API 的命名导出。一个
还提供了默认导出，这是 CommonJS 导出的值。
除其他事项外，默认导出可用于修改
出口。内置模块的命名导出只能通过调用来更新
[`module.syncBuiltinESMExports()`][module.syncBuiltinESMExports()].

```js
import EventEmitter from 'node:events';
const e = new EventEmitter();
```

```js
import { readFile } from 'node:fs';
readFile('./foo.txt', (err, source) => {
  if (err) {
    console.error(err);
  } else {
    console.log(source);
  }
});
```

```js
import fs, { readFileSync } from 'node:fs';
import { syncBuiltinESMExports } from 'node:module';
import { Buffer } from 'node:buffer';

fs.readFileSync = () => Buffer.from('Hello, ESM');
syncBuiltinESMExports();

fs.readFileSync === readFileSync;
```

## `import()`表达 式

[动态`import()`][Dynamic import()]在 CommonJS 和 ES 模块中均受支持。在 CommonJS 中
模块 它可以用来加载ES模块。

## `import.meta`

*   {对象}

这`import.meta`元属性是一个`Object`包含以下内容
性能。

### `import.meta.url`

*   {字符串}绝对`file:`模块的 URL。

这的定义与在提供
当前模块文件。

这将启用有用的模式，例如相对文件加载：

```js
import { readFileSync } from 'node:fs';
const buffer = readFileSync(new URL('./data.proto', import.meta.url));
```

### `import.meta.resolve(specifier[, parent])`

<!--
added:
  - v13.9.0
  - v12.16.2
changes:
  - version:
      - v16.2.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/38587
    description: Add support for WHATWG `URL` object to `parentURL` parameter.
-->

> 稳定性： 1 - 实验

此功能仅适用于`--experimental-import-meta-resolve`
命令标志已启用。

*   `specifier`{字符串}要相对于`parent`.
*   `parent`{字符串|URL} 要从中解析的绝对父模块 URL。如果没有
    指定，值`import.meta.url`用作默认值。
*   返回： {承诺}

提供作用域为每个模块的模块相对分辨率函数，返回
网址字符串。

<!-- eslint-skip -->

```js
const dependencyAsset = await import.meta.resolve('component-lib/asset.css');
```

`import.meta.resolve`还接受第二个参数，即父模块
从中解析：

<!-- eslint-skip -->

```js
await import.meta.resolve('./dep', import.meta.url);
```

此函数是异步的，因为 Node.js 中的 ES 模块解析器是
允许异步。

## 与 CommonJS 的互操作性

### `import`语句

一`import`语句可以引用 ES 模块或 CommonJS 模块。
`import`语句只允许在 ES 模块中使用，但允许动态[`import()`][import()]
CommonJS 中支持用于加载 ES 模块的表达式。

导入时[通用JS模块](#commonjs-namespaces)这
`module.exports`对象作为默认导出提供。命名导出可能是
可用，由静态分析提供，以方便更好的生态系统
兼容性。

### `require`

CommonJS 模块`require`始终将它引用的文件视为 CommonJS。

用`require`不支持加载 ES 模块，因为 ES 模块具有
异步执行。相反，请使用[`import()`][import()]加载 ES 模块
从 CommonJS 模块。

### CommonJS Namespaces

CommonJS模块由一个`module.exports`对象可以是任何类型。

导入 CommonJS 模块时，可以使用 ES 可靠地导入该模块
模块默认导入或其相应的糖语法：

<!-- eslint-disable no-duplicate-imports -->

```js
import { default as cjs } from 'cjs';

// The following import statement is "syntax sugar" (equivalent but sweeter)
// for `{ default as cjsSugar }` in the above import statement:
import cjsSugar from 'cjs';

console.log(cjs);
console.log(cjs === cjsSugar);
// Prints:
//   <module.exports>
//   true
```

CommonJS 模块的 ECMAScript 模块命名空间表示形式始终是
具有`default`导出指向 CommonJS 的密钥
`module.exports`价值。

此模块命名空间奇异对象可以在使用时直接观察
`import * as m from 'cjs'`或动态导入：

<!-- eslint-skip -->

```js
import * as m from 'cjs';
console.log(m);
console.log(m === await import('cjs'));
// Prints:
//   [Module] { default: <module.exports> }
//   true
```

为了更好地与JS生态系统中的现有用法兼容，Node.js
此外，尝试确定每个导入的CommonJS命名导出
CommonJS 模块将它们作为单独的 ES 模块导出提供，使用静态
分析过程。

例如，考虑编写一个CommonJS模块：

```cjs
// cjs.cjs
exports.name = 'exported';
```

前面的模块支持 ES 模块中的命名导入：

<!-- eslint-disable no-duplicate-imports -->

```js
import { name } from './cjs.cjs';
console.log(name);
// Prints: 'exported'

import cjs from './cjs.cjs';
console.log(cjs);
// Prints: { name: 'exported' }

import * as m from './cjs.cjs';
console.log(m);
// Prints: [Module] { default: { name: 'exported' }, name: 'exported' }
```

从模块命名空间奇异对象的最后一个示例中可以看出
已记录，`name`导出是从`module.exports`对象和集
导入模块时，直接在 ES 模块命名空间上。

实时绑定更新或添加到的新导出`module.exports`未检测到
对于这些命名的导出。

命名导出的检测基于常见的语法模式，但不基于
始终正确检测命名导出。在这些情况下，使用默认值
上面描述的导入表单可能是一个更好的选择。

命名导出检测涵盖许多常见的导出模式，重新导出模式
并构建工具和转译器输出。看[cjs-module-lexer][]对于确切的
语义已实现。

### ES模块和CommonJS之间的差异

#### 不`require`,`exports`或`module.exports`

在大多数情况下，ES 模块`import`可用于加载 CommonJS 模块。

如果需要，一个`require`函数可以在ES模块中构建，使用
[`module.createRequire()`][module.createRequire()].

#### 不`__filename`或`__dirname`

这些 CommonJS 变量在 ES 模块中不可用。

`__filename`和`__dirname`用例可以通过以下方式复制
[`import.meta.url`][import.meta.url].

#### 无需加载本机模块

ES 模块导入当前不支持本机模块。

相反，它们可以加载[`module.createRequire()`][module.createRequire()]或
[`process.dlopen`][process.dlopen].

#### 不`require.resolve`

相对分辨率可以通过以下方式处理`new URL('./local', import.meta.url)`.

对于完整的`require.resolve`替换，有标记的实验
[`import.meta.resolve`][import.meta.resolve]应用程序接口。

或者`module.createRequire()`可以使用。

#### 不`NODE_PATH`

`NODE_PATH`不是解析的一部分`import`说明符。请使用符号链接
如果需要此行为。

#### 不`require.extensions`

`require.extensions`未由`import`.期望是加载器
钩子可以在将来提供此工作流。

#### 不`require.cache`

`require.cache`未由`import`因为ES模块加载器有自己的
单独的缓存。

<i id="esm_experimental_json_modules"></i>

## JSON 模块

> 稳定性： 1 - 实验

JSON 文件可以由`import`:

```js
import packageConfig from './package.json' assert { type: 'json' };
```

这`assert { type: 'json' }`语法是强制性的;看[导入断言][Import Assertions].

导入的 JSON 仅公开一个`default`出口。不支持命名
出口。在 CommonJS 缓存中创建缓存条目以避免重复。
如果 JSON 模块已经
从同一路径导入。

<i id="esm_experimental_wasm_modules"></i>

## 瓦斯姆模块

> 稳定性： 1 - 实验

导入 WebAssembly 模块在
`--experimental-wasm-modules`标志，允许任何`.wasm`文件
作为普通模块导入，同时还支持其模块导入。

这种集成符合
[WebAssembly 的 ES 模块集成建议][ES Module Integration Proposal for WebAssembly].

例如，一个`index.mjs`含：

```js
import * as M from './module.wasm';
console.log(M);
```

执行依据：

```bash
node --experimental-wasm-modules index.mjs
```

将提供导出接口，用于实例化`module.wasm`.

<i id="esm_experimental_top_level_await"></i>

## 顶级`await`

<!-- YAML
added: v14.8.0
-->

这`await`关键字可以在 ECMAScript 模块的顶级正文中使用。

假设`a.mjs`跟

```js
export const five = await Promise.resolve(5);
```

和`b.mjs`跟

```js
import { five } from './a.mjs';

console.log(five); // Logs `5`
```

```bash
node b.mjs # works
```

如果是顶级`await`表达式永远不会解析，`node`进程将退出
与`13` [状态代码][status code].

```js
import { spawn } from 'node:child_process';
import { execPath } from 'node:process';

spawn(execPath, [
  '--input-type=module',
  '--eval',
  // Never-resolving Promise:
  'await new Promise(() => {})',
]).once('exit', (code) => {
  console.log(code); // Logs `13`
});
```

## HTTPS 和 HTTP 导入

> 稳定性： 1 - 实验

导入基于网络的模块`https:`和`http:`受以下项支持
这`--experimental-network-imports`旗。这允许类似 Web 浏览器的导入
在 Node 中工作.js由于应用程序稳定性和
在特权环境中运行时不同的安全问题
而不是浏览器沙箱。

### 导入限制为 HTTP/1

尚不支持 HTTP/2 和 HTTP/3 的自动协议协商。

### HTTP 仅限于环回地址

`http:`容易受到中间人攻击，并且不允许
用于 IPv4 地址之外的地址`127.0.0.0/8`(`127.0.0.1`自
`127.255.255.255`） 和 IPv6 地址`::1`.支持`http:`是有意的
用于本地开发。

### 身份验证永远不会发送到目标服务器。

`Authorization`,`Cookie`和`Proxy-Authorization`标头不会发送到
服务器。避免在导入的 URL 的部分中包含用户信息。安全模型
以便在正在处理的服务器上安全地使用这些内容。

### 从不在目标服务器上检查 CORS

CORS 旨在允许服务器将 API 的使用者限制为
特定的主机集。这不受支持，因为它对
基于服务器的实现。

### 无法加载非网络依赖项

这些模块无法访问未结束的其他模块`http:`或`https:`.
要在避免安全问题的同时仍访问本地模块，请转入
对本地依赖项的引用：

```mjs
// file.mjs
import worker_threads from 'node:worker_threads';
import { configure, resize } from 'https://example.com/imagelib.mjs';
configure({ worker_threads });
```

```mjs
// https://example.com/imagelib.mjs
let worker_threads;
export function configure(opts) {
  worker_threads = opts.worker_threads;
}
export function resize(img, size) {
  // Perform resizing in worker_thread to avoid main thread blocking
}
```

### 默认情况下不启用基于网络的加载

目前，`--experimental-network-imports`标志是启用加载所必需的
资源超过`http:`或`https:`.将来，一种不同的机制将是
用于强制执行此命令。需要选择加入以防止传递依赖关系
无意中使用了可能影响可靠性的潜在可变状态
节点.js应用程序。

<i id="esm_experimental_loaders"></i>

## 装载 机

<!-- YAML
added: v8.8.0
changes:
  - version:
    - v18.6.0
    pr-url: https://github.com/nodejs/node/pull/42623
    description: Add support for chaining loaders.
  - version: v16.12.0
    pr-url: https://github.com/nodejs/node/pull/37468
    description: Removed `getFormat`, `getSource`, `transformSource`, and
                 `globalPreload`; added `load` hook and `getGlobalPreload` hook.
-->

> 稳定性： 1 - 实验

> 此 API 目前正在重新设计，并且仍将更改。

<!-- type=misc -->

要自定义默认模块分辨率，可以选择将加载程序挂钩设置为
通过`--experimental-loader ./loader-name.mjs`参数到节点.js。

当使用钩子时，它们适用于入口点和所有`import`调用。他们
不适用于`require`呼叫;那些仍然跟随[CommonJS][]规则。

装载机遵循以下模式`--require`:

```console
node \
  --experimental-loader unpkg \
  --experimental-loader http-to-https \
  --experimental-loader cache-buster
```

它们按以下顺序调用：`cache-buster`调用
`http-to-https`哪个调用`unpkg`.

### 钩

钩子是链的一部分，即使该链只包含一个自定义
（用户提供的）钩子和默认钩子，它始终存在。钩
函数嵌套：每个函数必须始终返回一个普通对象，并且发生链接
作为每个函数调用的结果`next<hookName>()`，这是一个参考
到后续加载器的挂钩。

如果钩子返回缺少必需属性的值，则会触发异常。
返回而不调用的钩子`next<hookName>()` *和*不返回
`shortCircuit: true`还会触发异常。这些错误是有帮助的
防止链条中的意外断裂。

#### `resolve(specifier, context, nextResolve)`

<!-- YAML
changes:
  - version: v18.6.0
    pr-url: https://github.com/nodejs/node/pull/42623
    description: Add support for chaining resolve hooks. Each hook must either
      call `nextResolve()` or include a `shortCircuit` property set to `true`
      in its return.
  - version:
    - v17.1.0
    - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/40250
    description: Add support for import assertions.
-->

> 加载程序 API 正在重新设计。此钩子可能会消失或其
> 签名可能会更改。不要依赖下面描述的 API。

*   `specifier`{字符串}
*   `context`{对象}
    *   `conditions`{字符串\[]}出口条件相关`package.json`
    *   `importAssertions`{对象}
    *   `parentURL`{字符串|未定义}导入此模块的模块，或未定义的模块
        如果这是节点.js入口点
*   `nextResolve`{函数}后续`resolve`链条中的钩子，或
    节点.js默认值`resolve`最后一个用户提供的挂钩`resolve`钩
    *   `specifier`{字符串}
    *   `context`{对象}
*   返回： {对象}
    *   `format`{string|null|undefined}对负载挂钩的提示（可能是
        忽略）
        `'builtin' | 'commonjs' | 'json' | 'module' | 'wasm'`
    *   `shortCircuit`{undefined|boolean}此钩子打算的信号
        终止链`resolve`钩。**违约：** `false`
    *   `url`{字符串}此输入解析到的绝对 URL

这`resolve`钩子链负责解析给定的文件 URL
模块说明符和父 URL，以及可选的格式（如`'module'`)
作为对`load`钩。如果指定了格式，则`load`钩子是
最终负责提供最终`format`值（并且它是免费的
忽略 提供的提示`resolve`);如果`resolve`提供了一个`format`一个
习惯`load`即使只是将值传递给节点，也需要 hook.js
违约`load`钩。

模块说明符是`import`语句或
`import()`表达。

父 URL 是导入此模块的 URL，或者`undefined`
如果这是应用程序的主要入口点。

这`conditions`属性在`context`是条件数组
[包装出口条件][Conditional Exports]适用于本决议的内容
请求。它们可用于查找其他地方的条件映射或
在调用默认解析逻辑时修改列表。

当前[包装出口条件][Conditional Exports]始终处于
这`context.conditions`数组传递到钩子中。保证*违约
节点.js模块说明符解析行为*呼叫时`defaultResolve`这
`context.conditions`传递给它的数组*必须*包括*都*的元素
`context.conditions`数组最初传递到`resolve`钩。

```js
export async function resolve(specifier, context, nextResolve) {
  const { parentURL = null } = context;

  if (Math.random() > 0.5) { // Some condition.
    // For some or all specifiers, do some custom logic for resolving.
    // Always return an object of the form {url: <string>}.
    return {
      shortCircuit: true,
      url: parentURL ?
        new URL(specifier, parentURL).href :
        new URL(specifier).href,
    };
  }

  if (Math.random() < 0.5) { // Another condition.
    // When calling `defaultResolve`, the arguments can be modified. In this
    // case it's adding another value for matching conditional exports.
    return nextResolve(specifier, {
      ...context,
      conditions: [...context.conditions, 'another-condition'],
    });
  }

  // Defer to the next hook in the chain, which would be the
  // Node.js default resolve if this is the last user-specified loader.
  return nextResolve(specifier);
}
```

#### `load(url, context, nextLoad)`

<!-- YAML
changes:
  - version: v18.6.0
    pr-url: https://github.com/nodejs/node/pull/42623
    description: Add support for chaining load hooks. Each hook must either
      call `nextLoad()` or include a `shortCircuit` property set to `true` in
      its return.
-->

> 加载程序 API 正在重新设计。此钩子可能会消失或其
> 签名可能会更改。不要依赖下面描述的 API。

> 在此 API 的早期版本中，它被拆分为 3 个单独的 API，现在
> 已弃用，挂钩 （`getFormat`,`getSource`和`transformSource`).

*   `url`{字符串}返回的 URL`resolve`链
*   `context`{对象}
    *   `conditions`{字符串\[]}出口条件相关`package.json`
    *   `format`{string|null|undefined}可选的格式由
        `resolve`钩链
    *   `importAssertions`{对象}
*   `nextLoad`{函数}后续`load`链条中的钩子，或
    节点.js默认值`load`最后一个用户提供的挂钩`load`钩
    *   `specifier`{字符串}
    *   `context`{对象}
*   返回： {对象}
    *   `format`{字符串}
    *   `shortCircuit`{undefined|boolean}此钩子打算的信号
        终止链`resolve`钩。**违约：** `false`
    *   `source`{字符串|ArrayBuffer|TypedArray} 要评估的 Node.js 的源代码

这`load`hook 提供了一种定义自定义方法的方法，用于确定如何
应解释、检索和解析 URL。它还负责
验证导入断言。

的最终值`format`必须是下列之一：

|`format`|描述 |可接受的类型`source`返回者`load`|
|------------ |------------------------------ |----------------------------------------------------- |
|`'builtin'`|加载节点.js内置模块|不适用于|
|`'commonjs'`|加载节点.js CommonJS 模块|不适用于|
|`'json'`|加载 JSON 文件|{[`string`][string],[`ArrayBuffer`][ArrayBuffer],[`TypedArray`][TypedArray]} |
|`'module'`|加载 ES 模块|{[`string`][string],[`ArrayBuffer`][ArrayBuffer],[`TypedArray`][TypedArray]} |
|`'wasm'`|加载 WebAssembly 模块|{[`ArrayBuffer`][ArrayBuffer],[`TypedArray`][TypedArray]}               |

的价值`source`对于类型被忽略`'builtin'`因为目前它是
无法替换节点.js内置（核心）模块的值。价值
之`source`对于类型被忽略`'commonjs'`因为 CommonJS 模块加载程序
不提供 ES 模块加载程序覆盖
[通用JS模块返回值](#commonjs-namespaces).此限制可能是
在未来克服。

> **警告**： ESM`load`从 CommonJS 模块的挂接和命名空间导出
> 不兼容。尝试同时使用它们将导致空
> 对象从导入。这将在将来可能会得到解决。

> 这些类型都对应于 ECMAScript 中定义的类。

*   具体[`ArrayBuffer`][ArrayBuffer]对象是一个[`SharedArrayBuffer`][SharedArrayBuffer].
*   具体[`TypedArray`][TypedArray]对象是一个[`Uint8Array`][Uint8Array].

如果基于文本的格式的源值（即`'json'`,`'module'`)
不是字符串，而是使用[`util.TextDecoder`][util.TextDecoder].

这`load`hook 提供了一种定义自定义方法来检索
ES 模块说明符的源代码。这将允许加载器潜在地
避免从磁盘读取文件。它还可用于映射未识别的
格式为受支持的格式，例如`yaml`自`module`.

```js
export async function load(url, context, nextLoad) {
  const { format } = context;

  if (Math.random() > 0.5) { // Some condition
    /*
      For some or all URLs, do some custom logic for retrieving the source.
      Always return an object of the form {
        format: <string>,
        source: <string|buffer>,
      }.
    */
    return {
      format,
      shortCircuit: true,
      source: '...',
    };
  }

  // Defer to the next hook in the chain.
  return nextLoad(url);
}
```

在更高级的方案中，这还可用于转换不受支持的
源到受支持的源（请参阅[例子](#examples)下面）。

#### `globalPreload()`

<!-- YAML
changes:
  - version: v18.6.0
    pr-url: https://github.com/nodejs/node/pull/42623
    description: Add support for chaining globalPreload hooks.
-->

> 加载程序 API 正在重新设计。此钩子可能会消失或其
> 签名可能会更改。不要依赖下面描述的 API。

> 在此 API 的早期版本中，此钩子被命名为
> `getGlobalPreloadCode`.

*   `context`{对象}有助于预加载代码的信息
    *   `port`{消息端口}
*   返回：{字符串} 要在应用程序启动之前运行的代码

有时可能需要在同一全局变量中运行一些代码
应用程序运行范围。此钩子允许返回字符串
在启动时作为草率模式脚本运行。

与 CommonJS 包装器的工作方式类似，代码在隐式函数中运行
范围。唯一的参数是`require`-可用于加载的类似功能
内置的，如“fs”：`getBuiltin(request: string)`.

如果代码需要更高级`require`功能，它必须构建
自己`require`用`module.createRequire()`.

```js
export function globalPreload(context) {
  return `\
globalThis.someInjectedProperty = 42;
console.log('I just set some globals!');

const { createRequire } = getBuiltin('module');
const { cwd } = getBuiltin('process');

const require = createRequire(cwd() + '/<preload>');
// [...]
`;
}
```

为了允许应用程序和加载程序之间的通信，另一个
参数提供给预加载代码：`port`.这可用作
参数到加载程序挂钩和挂钩返回的源文本内部。
必须小心才能正确拨打电话[`port.ref()`][port.ref()]和
[`port.unref()`][port.unref()]以防止进程处于不会的状态
正常关闭。

```js
/**
 * This example has the application context send a message to the loader
 * and sends the message back to the application context
 */
export function globalPreload({ port }) {
  port.onmessage = (evt) => {
    port.postMessage(evt.data);
  };
  return `\
    port.postMessage('console.log("I went to the Loader and back");');
    port.onmessage = (evt) => {
      eval(evt.data);
    };
  `;
}
```

### 例子

各种装载机吊钩可以一起使用，以实现广泛的
节点的自定义.js代码加载和评估行为。

#### HTTPS loader

在当前节点.js中，说明符以`https://`是实验性的（参见
[HTTPS 和 HTTP 导入][HTTPS and HTTP imports]).

下面的加载程序注册钩子，以实现对此类操作的基本支持
说明符。虽然这似乎是对Node的重大改进.js核心
功能，实际使用此加载程序有很大的缺点：
性能比从磁盘加载文件慢得多，没有缓存，
而且没有安全性。

```js
// https-loader.mjs
import { get } from 'node:https';

export function resolve(specifier, context, nextResolve) {
  const { parentURL = null } = context;

  // Normally Node.js would error on specifiers starting with 'https://', so
  // this hook intercepts them and converts them into absolute URLs to be
  // passed along to the later hooks below.
  if (specifier.startsWith('https://')) {
    return {
      shortCircuit: true,
      url: specifier
    };
  } else if (parentURL && parentURL.startsWith('https://')) {
    return {
      shortCircuit: true,
      url: new URL(specifier, parentURL).href,
    };
  }

  // Let Node.js handle all other specifiers.
  return nextResolve(specifier);
}

export function load(url, context, nextLoad) {
  // For JavaScript to be loaded over the network, we need to fetch and
  // return it.
  if (url.startsWith('https://')) {
    return new Promise((resolve, reject) => {
      get(url, (res) => {
        let data = '';
        res.on('data', (chunk) => data += chunk);
        res.on('end', () => resolve({
          // This example assumes all network-provided JavaScript is ES module
          // code.
          format: 'module',
          shortCircuit: true,
          source: data,
        }));
      }).on('error', (err) => reject(err));
    });
  }

  // Let Node.js handle all other URLs.
  return nextLoad(url);
}
```

```js
// main.mjs
import { VERSION } from 'https://coffeescript.org/browser-compiler-modern/coffeescript.js';

console.log(VERSION);
```

使用前面的加载程序，运行
`node --experimental-loader ./https-loader.mjs ./main.mjs`
在 URL 处的每个模块打印当前版本的 CoffeeScript
`main.mjs`.

#### 转译器装载机

Node.js不理解的格式的源可以转换为
JavaScript 使用[`load`钩][load hook].在钩子被调用之前，
但是，一个[`resolve`钩][resolve hook]需要告诉Node.js不要
在未知文件类型上引发错误。

这比在运行之前转译源文件的性能要低
节点.js;转译器加载器应仅用于开发和测试
目的。

```js
// coffeescript-loader.mjs
import { readFile } from 'node:fs/promises';
import { dirname, extname, resolve as resolvePath } from 'node:path';
import { cwd } from 'node:process';
import { fileURLToPath, pathToFileURL } from 'node:url';
import CoffeeScript from 'coffeescript';

const baseURL = pathToFileURL(`${cwd()}/`).href;

// CoffeeScript files end in .coffee, .litcoffee, or .coffee.md.
const extensionsRegex = /\.coffee$|\.litcoffee$|\.coffee\.md$/;

export async function resolve(specifier, context, nextResolve) {
  if (extensionsRegex.test(specifier)) {
    const { parentURL = baseURL } = context;

    // Node.js normally errors on unknown file extensions, so return a URL for
    // specifiers ending in the CoffeeScript file extensions.
    return {
      shortCircuit: true,
      url: new URL(specifier, parentURL).href
    };
  }

  // Let Node.js handle all other specifiers.
  return nextResolve(specifier);
}

export async function load(url, context, nextLoad) {
  if (extensionsRegex.test(url)) {
    // Now that we patched resolve to let CoffeeScript URLs through, we need to
    // tell Node.js what format such URLs should be interpreted as. Because
    // CoffeeScript transpiles into JavaScript, it should be one of the two
    // JavaScript formats: 'commonjs' or 'module'.

    // CoffeeScript files can be either CommonJS or ES modules, so we want any
    // CoffeeScript file to be treated by Node.js the same as a .js file at the
    // same location. To determine how Node.js would interpret an arbitrary .js
    // file, search up the file system for the nearest parent package.json file
    // and read its "type" field.
    const format = await getPackageType(url);
    // When a hook returns a format of 'commonjs', `source` is be ignored.
    // To handle CommonJS files, a handler needs to be registered with
    // `require.extensions` in order to process the files with the CommonJS
    // loader. Avoiding the need for a separate CommonJS handler is a future
    // enhancement planned for ES module loaders.
    if (format === 'commonjs') {
      return {
        format,
        shortCircuit: true,
      };
    }

    const { source: rawSource } = await nextLoad(url, { ...context, format });
    // This hook converts CoffeeScript source code into JavaScript source code
    // for all imported CoffeeScript files.
    const transformedSource = coffeeCompile(rawSource.toString(), url);

    return {
      format,
      shortCircuit: true,
      source: transformedSource,
    };
  }

  // Let Node.js handle all other URLs.
  return nextLoad(url);
}

async function getPackageType(url) {
  // `url` is only a file path during the first iteration when passed the
  // resolved url from the load() hook
  // an actual file path from load() will contain a file extension as it's
  // required by the spec
  // this simple truthy check for whether `url` contains a file extension will
  // work for most projects but does not cover some edge-cases (such as
  // extensionless files or a url ending in a trailing space)
  const isFilePath = !!extname(url);
  // If it is a file path, get the directory it's in
  const dir = isFilePath ?
    dirname(fileURLToPath(url)) :
    url;
  // Compose a file path to a package.json in the same directory,
  // which may or may not exist
  const packagePath = resolvePath(dir, 'package.json');
  // Try to read the possibly nonexistent package.json
  const type = await readFile(packagePath, { encoding: 'utf8' })
    .then((filestring) => JSON.parse(filestring).type)
    .catch((err) => {
      if (err?.code !== 'ENOENT') console.error(err);
    });
  // Ff package.json existed and contained a `type` field with a value, voila
  if (type) return type;
  // Otherwise, (if not at the root) continue checking the next directory up
  // If at the root, stop and return false
  return dir.length > 1 && getPackageType(resolvePath(dir, '..'));
}
```

```coffee
# main.coffee
import { scream } from './scream.coffee'
console.log scream 'hello, world'

import { version } from 'node:process'
console.log "Brought to you by Node.js version #{version}"
```

```coffee
# scream.coffee
export scream = (str) -> str.toUpperCase()
```

使用前面的加载程序，运行
`node --experimental-loader ./coffeescript-loader.mjs main.coffee`
原因`main.coffee`在其源代码是
从磁盘加载，但在 Node 之前.js执行它;以此类推，适用于任何`.coffee`,
`.litcoffee`或`.coffee.md`引用的文件`import`任何的声明
加载的文件。

## 分辨率算法

### 特征

冲突解决程序具有以下属性：

*   ES 模块使用的基于文件 URL 的分辨率
*   支持内置模块加载
*   相对和绝对 URL 解析
*   无默认扩展名
*   无文件夹主目录
*   通过node_modules进行裸说明符包分辨率查找

### 旋转变压器算法

加载 ES 模块说明符的算法通过以下方式给出：
**ESM_RESOLVE**下面的方法。它返回
相对于父 URL 的模块说明符。

确定解析 URL 的模块格式的算法为
提供方**ESM_FORMAT**，返回唯一模块
任何文件的格式。这*“模块”*为 ECMAScript 返回格式
模块，而*“commonjs”*格式用于指示通过
遗留的 CommonJS 加载程序。其他格式，例如*“插件”*可以扩展于
未来的更新。

在以下算法中，所有子例程错误都作为错误传播
这些顶级例程，除非另有说明。

*默认条件*是条件环境名称数组，
`["node", "import"]`.

冲突解决程序可能会引发以下错误：

*   *无效的模块说明符*：模块说明符是无效的 URL，包名称
    或包子路径说明符。
*   *包配置无效*：package.json 配置无效或
    包含无效的配置。
*   *无效的包目标*：包导出或导入定义目标模块
    对于无效类型或字符串目标的包。
*   *未导出包路径*：包导出不定义或不允许目标
    给定模块的包中的子路径。
*   *未定义包导入*：包导入不定义说明符。
*   *未找到模块*：请求的包或模块不存在。
*   *不支持的目录导入*：解析的路径对应于一个目录，
    这不是模块导入的受支持目标。

### 旋转变压器算法规范

**ESM_RESOLVE**(*规范*,*父网址*)

> 1.  让*解决*是**定义**.
> 2.  如果*规范*是有效的网址，则
>     1.  设置*解决*到解析和重序列化的结果
>         *规范*作为网址。
> 3.  否则，如果*规范*开头为*"/"*,*"./"*或*"../"*然后
>     1.  设置*解决*到网址分辨率*规范*相对于
>         *父网址*.
> 4.  否则，如果*规范*开头为*"#"*然后
>     1.  设置*解决*到结果
>         **PACKAGE_IMPORTS_RESOLVE**(*规范*,
>         *父网址*,*默认条件*).
> 5.  否则
>     1.  注意：*规范*现在是一个裸说明符。
>     2.  设置*解决*结果
>         **PACKAGE_RESOLVE**(*规范*,*父网址*).
> 6.  让*格式*是**定义**.
> 7.  如果*解决*是一个*“文件：”*网址，然后
>     1.  如果*解决*包含*"/"*或*"\\"*(*“%2F”*
>         和*“%5C”*），然后
>         1.  抛出一个*无效的模块说明符*错误。
>     2.  如果文件位于*解决*是一个目录，则
>         1.  抛出一个*不支持的目录导入*错误。
>     3.  如果文件位于*解决*不存在，则
>         1.  抛出一个*未找到模块*错误。
>     4.  设置*解决*到真正的路径*解决*，维护
>         相同的 URL 查询字符串和片段组件。
>     5.  设置*格式*到结果**ESM_FILE_FORMAT**(*解决*).
> 8.  否则
>     1.  设置*格式*与
>         网址*解决*.
> 9.  负荷*解决*作为模块格式，*格式*.

**PACKAGE_RESOLVE**(*包指定器*,*父网址*)

> 1.  让*包名称*是**定义**.
> 2.  如果*包指定器*是一个空字符串，则
>     1.  抛出一个*无效的模块说明符*错误。
> 3.  如果*包指定器*是节点.js内置模块名称，则
>     1.  返回字符串*“节点：”*串联于*包指定器*.
> 4.  如果*包指定器*不以 开头*"@"*然后
>     1.  设置*包名称*到 子字符串*包指定器*直到第一个
>         *"/"*分隔符或字符串的末尾。
> 5.  否则
>     1.  如果*包指定器*不包含*"/"*分隔符，然后
>         1.  抛出一个*无效的模块说明符*错误。
>     2.  设置*包名称*到 子字符串*包指定器*
>         直到第二个*"/"*分隔符或字符串的末尾。
> 6.  如果*包名称*开头为*"."*或包含*"\\"*或*"%"*然后
>     1.  抛出一个*无效的模块说明符*错误。
> 7.  让*包子路径*是*"."*与
>     *包指定器*从长度的位置*包名称*.
> 8.  如果*包子路径*结束于*"/"*然后
>     1.  抛出一个*无效的模块说明符*错误。
> 9.  让*自网址*是以下原因的结果
>     **PACKAGE_SELF_RESOLVE**(*包名称*,*包子路径*,*父网址*).
> 10. 如果*自网址*莫**定义**返回*自网址*.
> 11. 而*父网址*不是文件系统根目录，
>     1.  让*packageURL*是*“node_modules/”*
>         串联于*包指定器*，相对于*父网址*.
>     2.  设置*父网址*到 父文件夹 URL*父网址*.
>     3.  如果文件夹位于*packageURL*不存在，则
>         1.  继续下一个循环迭代。
>     4.  让*pjson*是以下原因的结果**READ_PACKAGE_JSON**(*packageURL*).
>     5.  如果*pjson*莫**零**和*pjson*.*出口*莫**零**或
>         **定义**然后
>         1.  返回结果**PACKAGE_EXPORTS_RESOLVE**(*packageURL*,
>             *包子路径*,*pjson.exports*,*默认条件*).
>     6.  否则，如果*包子路径*等于*"."*然后
>         1.  如果*pjson.main*是一个字符串，则
>             1.  返回网址分辨率*主要*在*packageURL*.
>     7.  否则
>         1.  返回网址分辨率*包子路径*在*packageURL*.
> 12. 抛出一个*未找到模块*错误。

**PACKAGE_SELF_RESOLVE**(*包名称*,*包子路径*,*父网址*)

> 1.  让*packageURL*是以下原因的结果**LOOKUP_PACKAGE_SCOPE**(*父网址*).
> 2.  如果*packageURL*是**零**然后
>     1.  返回**定义**.
> 3.  让*pjson*是以下原因的结果**READ_PACKAGE_JSON**(*packageURL*).
> 4.  如果*pjson*是**零**或者如果*pjson*.*出口*是**零**或
>     **定义**然后
>     1.  返回**定义**.
> 5.  如果*pjson.name*等于*包名称*然后
>     1.  返回结果**PACKAGE_EXPORTS_RESOLVE**(*packageURL*,
>         *包子路径*,*pjson.exports*,*默认条件*).
> 6.  否则，返回**定义**.

**PACKAGE_EXPORTS_RESOLVE**(*packageURL*,*子路径*,*出口*,*条件*)

> 1.  如果*出口*是一个对象，其两个键都以 开头*"."*和键不
>     从*"."*，抛出一个*包配置无效*错误。
> 2.  如果*子路径*等于*"."*然后
>     1.  让*主出口*是**定义**.
>     2.  如果*出口*是字符串或数组，或者不包含键的对象
>         从*"."*然后
>         1.  设置*主出口*自*出口*.
>     3.  否则，如果*出口*是一个包含*"."*属性，则
>         1.  设置*主出口*自*出口*\[*"."*].
>     4.  如果*主出口*莫**定义**然后
>         1.  让*解决*是以下原因的结果**PACKAGE_TARGET_RESOLVE**(
>             *packageURL*,*主出口*,*""*,**假**,**假**,
>             *条件*).
>         2.  如果*解决*莫**零**或**定义**返回*解决*.
> 3.  否则，如果*出口*是一个对象，并且*出口*入手
>     *"."*然后
>     1.  让*匹配键*成为字符串*"./"*串联于*子路径*.
>     2.  让*解决*是以下原因的结果**PACKAGE_IMPORTS_EXPORTS_RESOLVE**(
>         *匹配键*,*出口*,*packageURL*,**假**,*条件*).
>     3.  如果*解决*莫**零**或**定义**返回*解决*.
> 4.  抛出一个*未导出包路径*错误。

**PACKAGE_IMPORTS_RESOLVE**(*规范*,*父网址*,*条件*)

> 1.  断言：*规范*开头为*"#"*.
> 2.  如果*规范*正好等于*"#"*或以 开头*"#/"*然后
>     1.  抛出一个*无效的模块说明符*错误。
> 3.  让*packageURL*是以下原因的结果**LOOKUP_PACKAGE_SCOPE**(*父网址*).
> 4.  如果*packageURL*莫**零**然后
>     1.  让*pjson*是以下原因的结果**READ_PACKAGE_JSON**(*packageURL*).
>     2.  如果*pjson.imports*是一个非空对象，则
>         1.  让*解决*是以下原因的结果
>             **PACKAGE_IMPORTS_EXPORTS_RESOLVE**(
>             *规范*,*pjson.imports*,*packageURL*,**真**,*条件*).
>         2.  如果*解决*莫**零**或**定义**返回*解决*.
> 5.  抛出一个*未定义包导入*错误。

**PACKAGE_IMPORTS_EXPORTS_RESOLVE**(*匹配键*,*matchObj*,*packageURL*,
*isImports*,*条件*)

> 1.  如果*匹配键*是*matchObj*并且不包含*"\*"*然后
>     1.  让*目标*是的价值*matchObj*\[*匹配键*].
>     2.  返回结果**PACKAGE_TARGET_RESOLVE**(*packageURL*,
>         *目标*,*""*,**假**,*isImports*,*条件*).
> 2.  让*扩展密钥*是 的键列表*matchObj*仅包含
>     单*"\*"*，按排序功能排序**PATTERN_KEY_COMPARE**
>     哪些顺序按具体性降序排列。
> 3.  对于每个键*扩展键*在*扩展密钥*做
>     1.  让*模式基础*成为 的子字符串*扩展键*最多但不包括
>         第一个*"\*"*字符。
>     2.  如果*匹配键*开头为 但不等于*模式基础*然后
>         1.  让*模式拖车*成为 的子字符串*扩展键*从
>             索引后的第一个*"\*"*字符。
>         2.  如果*模式拖车*长度为零，或者如果*匹配键*结尾为
>             *模式拖车*和长度*匹配键*大于或
>             等于的长度*扩展键*然后
>             1.  让*目标*是的价值*matchObj*\[*扩展键*].
>             2.  让*子路径*成为 的子字符串*匹配键*从
>                 长度的索引*模式基础*最大长度
>                 *匹配键*减去长度*模式拖车*.
>             3.  返回结果**PACKAGE_TARGET_RESOLVE**(*packageURL*,
>                 *目标*,*子路径*,**真**,*isImports*,*条件*).
> 4.  返回**零**.

**PATTERN_KEY_COMPARE**(*键 A*,*键B*)

> 1.  断言：*键 A*结尾为*"/"*或仅包含单个*"\*"*.
> 2.  断言：*键B*结尾为*"/"*或仅包含单个*"\*"*.
> 3.  让*基本长度A*成为索引*"\*"*在*键 A*加一，如果*键 A*
>     包含*"\*"*，或长度*键 A*否则。
> 4.  让*基长度B*成为索引*"\*"*在*键B*加一，如果*键B*
>     包含*"\*"*，或长度*键B*否则。
> 5.  如果*基本长度A*大于*基长度B*，返回 -1。
> 6.  如果*基长度B*大于*基本长度A*，返回 1。
> 7.  如果*键 A*不包含*"\*"*，返回 1。
> 8.  如果*键B*不包含*"\*"*，返回 -1。
> 9.  如果长度*键 A*的长度大于*键B*，返回 -1。
> 10. 如果长度*键B*的长度大于*键 A*，返回 1。
> 11. 返回 0。

**PACKAGE_TARGET_RESOLVE**(*packageURL*,*目标*,*子路径*,*模式*,
*内部*,*条件*)

> 1.  如果*目标*是一个字符串，则
>     1.  如果*模式*是**假**,*子路径*具有非零长度和*目标*
>         不以 结尾*"/"*，抛出一个*无效的模块说明符*错误。
>     2.  如果*目标*不以 开头*"./"*然后
>         1.  如果*内部*是**真**和*目标*不以 开头*"../"*或
>             *"/"*并且不是有效的 URL，则
>             1.  如果*模式*是**真**然后
>                 1.  返回**PACKAGE_RESOLVE**(*目标*与的每个实例
>                     *"\*"*替换为*子路径*,*packageURL*+*"/"*).
>             2.  返回**PACKAGE_RESOLVE**(*目标*+*子路径*,
>                 *packageURL*+*"/"*).
>         2.  否则，抛出一个*无效的包目标*错误。
>     3.  如果*目标*拆分于*"/"*或*"\\"*包含任何*"."*,*".."*或
>         *“node_modules”*第一段之后的段，不区分大小写和
>         包括百分比编码的变体，抛出一个*无效的包目标*
>         错误。
>     4.  让*已解决目标*是串联的 URL 解析
>         *packageURL*和*目标*.
>     5.  断言：*已解决目标*包含在*packageURL*.
>     6.  如果*子路径*拆分于*"/"*或*"\\"*包含任何*"."*,*".."*或
>         *“node_modules”*段，不区分大小写，包括百分比
>         编码的变体，抛出一个*无效的模块说明符*错误。
>     7.  如果*模式*是**真**然后
>         1.  返回网址分辨率*已解决目标*与的每个实例
>             *"\*"*替换为*子路径*.
>     8.  否则
>         1.  返回 串联的 URL 解析*子路径*和
>             *已解决目标*.
> 2.  否则，如果*目标*是一个非空对象，则
>     1.  如果*出口*包含 ECMA-262 中定义的任何索引属性键
>         [6.1.7 数组索引][6.1.7 Array Index]，抛出一个*包配置无效*错误。
>     2.  对于每个属性*p*之*目标*，在对象插入顺序中为，
>         1.  如果*p*等于*“默认”*或*条件*包含的条目*p*,
>             然后
>             1.  让*目标值*是的价值*p*属性在*目标*.
>             2.  让*解决*是以下原因的结果**PACKAGE_TARGET_RESOLVE**(
>                 *packageURL*,*目标值*,*子路径*,*模式*,*内部*,
>                 *条件*).
>             3.  如果*解决*等于**定义**，继续循环。
>             4.  返回*解决*.
>     3.  返回**定义**.
> 3.  否则，如果*目标*是一个数组，则
>     1.  如果 \_target.length 为零，则返回**零**.
>     2.  对于每个项目*目标值*在*目标*做
>         1.  让*解决*是以下原因的结果**PACKAGE_TARGET_RESOLVE**(
>             *packageURL*,*目标值*,*子路径*,*模式*,*内部*,
>             *条件*），继续任何上的循环*无效的包目标*
>             错误。
>         2.  如果*解决*是**定义**，继续循环。
>         3.  返回*解决*.
>     3.  返回或抛出上次回退分辨率**零**返回或错误。
> 4.  否则，如果*目标*是*零*返回**零**.
> 5.  否则，抛出一个*无效的包目标*错误。

**ESM_FILE_FORMAT**(*网址*)

> 1.  断言：*网址*对应于现有文件。
> 2.  如果*网址*结束于*“.mjs”*然后
>     1.  返回*“模块”*.
> 3.  如果*网址*结束于*“.cjs”*然后
>     1.  返回*“commonjs”*.
> 4.  如果*网址*结束于*“.json”*然后
>     1.  返回*“json”*.
> 5.  让*packageURL*是以下原因的结果**LOOKUP_PACKAGE_SCOPE**(*网址*).
> 6.  让*pjson*是以下原因的结果**READ_PACKAGE_JSON**(*packageURL*).
> 7.  如果*pjson？.类型*存在并且是*“模块”*然后
>     1.  如果*网址*结束于*“.js”*然后
>         1.  返回*“模块”*.
>     2.  抛出一个*不支持的文件扩展名*错误。
> 8.  否则
>     1.  抛出一个*不支持的文件扩展名*错误。

**LOOKUP_PACKAGE_SCOPE**(*网址*)

> 1.  让*范围网址*是*网址*.
> 2.  而*范围网址*不是文件系统根目录，
>     1.  设置*范围网址*到 的父网址*范围网址*.
>     2.  如果*范围网址*以*“node_modules”*路径段， 返回**零**.
>     3.  让*pjsonURL*是分辨率*“package.json”*在
>         *范围网址*.
>     4.  如果文件位于*pjsonURL*存在，则
>         1.  返回*范围网址*.
> 3.  返回**零**.

**READ_PACKAGE_JSON**(*packageURL*)

> 1.  让*pjsonURL*是分辨率*“package.json”*在*packageURL*.
> 2.  如果文件位于*pjsonURL*不存在，则
>     1.  返回**零**.
> 3.  如果文件位于*packageURL*不解析为有效的 JSON，则
>     1.  抛出一个*包配置无效*错误。
> 4.  返回已解析的文件 JSON 源，地址为*pjsonURL*.

### 自定义 ESM 说明符解析算法

> 稳定性： 1 - 实验

> 不要依赖此标志。我们计划在
> [加载程序 API][Loaders API]已经发展到等效功能可以
> 通过定制装载机实现。

当前说明符分辨率不支持
CommonJS loader。行为差异之一是自动解决
的文件扩展名和导入具有索引的目录的功能
文件。

这`--experimental-specifier-resolution=[mode]`标志可用于自定义
扩展解析算法。默认模式为`explicit`哪
要求向加载程序提供模块的完整路径。要启用
自动扩展解析和从包含
索引文件使用`node`模式。

```console
$ node index.mjs
success!
$ node index # Failure!
Error: Cannot find module
$ node --experimental-specifier-resolution=node index
success!
```

<!-- Note: The cjs-module-lexer link should be kept in-sync with the deps version -->

[6.1.7 Array Index]: https://tc39.es/ecma262/#integer-index

[CommonJS]: modules.md

[Conditional exports]: packages.md#conditional-exports

[Core modules]: modules.md#core-modules

[Determining module system]: packages.md#determining-module-system

[Dynamic `import()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import

[ES Module Integration Proposal for WebAssembly]: https://github.com/webassembly/esm-integration

[HTTPS and HTTP imports]: #https-and-http-imports

[Import Assertions]: #import-assertions

[Import Assertions proposal]: https://github.com/tc39/proposal-import-assertions

[JSON modules]: #json-modules

[Loaders API]: #loaders

[Node.js Module Resolution Algorithm]: #resolver-algorithm-specification

[Terminology]: #terminology

[URL]: https://url.spec.whatwg.org/

[`"exports"`]: packages.md#exports

[`"type"`]: packages.md#type

[`--input-type`]: cli.md#--input-typetype

[`ArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

[`SharedArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer

[`TypedArray`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray

[`Uint8Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array

[`data:` URLs]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs

[`export`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export

[`import()`]: #import-expressions

[`import.meta.resolve`]: #importmetaresolvespecifier-parent

[`import.meta.url`]: #importmetaurl

[`import`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import

[`module.createRequire()`]: module.md#modulecreaterequirefilename

[`module.syncBuiltinESMExports()`]: module.md#modulesyncbuiltinesmexports

[`package.json`]: packages.md#nodejs-packagejson-field-definitions

[`port.ref()`]: https://nodejs.org/dist/latest-v17.x/docs/api/worker_threads.html#portref

[`port.unref()`]: https://nodejs.org/dist/latest-v17.x/docs/api/worker_threads.html#portunref

[`process.dlopen`]: process.md#processdlopenmodule-filename-flags

[`string`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String

[`util.TextDecoder`]: util.md#class-utiltextdecoder

[cjs-module-lexer]: https://github.com/nodejs/cjs-module-lexer/tree/1.2.2

[custom https loader]: #https-loader

[load hook]: #loadurl-context-nextload

[percent-encoded]: url.md#percent-encoding-in-urls

[resolve hook]: #resolvespecifier-context-nextresolve

[special scheme]: https://url.spec.whatwg.org/#special-scheme

[status code]: process.md#exit-codes

[the official standard format]: https://tc39.github.io/ecma262/#sec-modules

[url.pathToFileURL]: url.md#urlpathtofileurlpath
