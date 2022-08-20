# 模块：`node:module`应用程序接口

<!--introduced_in=v12.20.0-->

<!-- YAML
added: v0.3.7
-->

## 这`Module`对象

*   {对象}

提供与 实例交互时的常规实用工具方法
`Module`这[`module`][module]变量常见于[CommonJS][]模块。访问
通过`import 'node:module'`或`require('node:module')`.

### `module.builtinModules`

<!-- YAML
added:
  - v9.3.0
  - v8.10.0
  - v6.13.0
-->

*   {字符串\[]}

Node.js 提供的所有模块的名称列表。可用于验证
模块是否由第三方维护。

`module`在此上下文中不是提供的同一对象
由[模块包装器][module wrapper].要访问它，需要`Module`模块：

```mjs
// module.mjs
// In an ECMAScript module
import { builtinModules as builtin } from 'node:module';
```

```cjs
// module.cjs
// In a CommonJS module
const builtin = require('node:module').builtinModules;
```

### `module.createRequire(filename)`

<!-- YAML
added: v12.2.0
-->

*   `filename`{字符串|URL} 用于构造要求的文件名
    功能。必须是文件 URL 对象、文件 URL 字符串或绝对路径
    字符串。
*   返回：{require} Require 函数

```mjs
import { createRequire } from 'node:module';
const require = createRequire(import.meta.url);

// sibling-module.js is a CommonJS module.
const siblingModule = require('./sibling-module');
```

### `module.isBuiltin(moduleName)`

<!-- YAML
added: v18.6.0
-->

*   `moduleName`{字符串} 模块的名称
*   返回：{布尔值} 如果模块是内置的，则返回 true，否则返回 false

```mjs
import { isBuiltin } from 'node:module';
isBuiltin('node:fs'); // true
isBuiltin('fs'); // true
isBuiltin('wss'); // false
```

### `module.syncBuiltinESMExports()`

<!-- YAML
added: v12.12.0
-->

这`module.syncBuiltinESMExports()`方法更新 的所有实时绑定
内置[电子模块][ES Modules]以匹配 的属性[CommonJS][]出口。它
不会在[电子模块][ES Modules].

```js
const fs = require('node:fs');
const assert = require('node:assert');
const { syncBuiltinESMExports } = require('node:module');

fs.readFile = newAPI;

delete fs.readFileSync;

function newAPI() {
  // ...
}

fs.newAPI = newAPI;

syncBuiltinESMExports();

import('node:fs').then((esmFS) => {
  // It syncs the existing readFile property with the new value
  assert.strictEqual(esmFS.readFile, newAPI);
  // readFileSync has been deleted from the required fs
  assert.strictEqual('readFileSync' in fs, false);
  // syncBuiltinESMExports() does not remove readFileSync from esmFS
  assert.strictEqual('readFileSync' in esmFS, true);
  // syncBuiltinESMExports() does not add names
  assert.strictEqual(esmFS.newAPI, undefined);
});
```

## 源映射 v3 支持

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
-->

> 稳定性： 1 - 实验

用于与源地图缓存交互的帮助程序。此缓存是
在启用源映射解析时填充，并且
[源映射包含指令][source map include directives]位于模块的页脚中。

要启用源映射解析，必须使用标志运行 Node.js
[`--enable-source-maps`][--enable-source-maps]，或通过设置启用代码覆盖率
[`NODE_V8_COVERAGE=dir`][NODE_V8_COVERAGE=dir].

```mjs
// module.mjs
// In an ECMAScript module
import { findSourceMap, SourceMap } from 'node:module';
```

```cjs
// module.cjs
// In a CommonJS module
const { findSourceMap, SourceMap } = require('node:module');
```

<!-- Anchors to make sure old links find a target -->

<a id="module_module_findsourcemap_path_error"></a>

### `module.findSourceMap(path)`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
-->

*   `path`{字符串}
*   返回：{模块。源地图}

`path`是文件的解析路径，其具有相应的源映射
应该被获取。

### 类：`module.SourceMap`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
-->

#### `new SourceMap(payload)`

*   `payload`{对象}

创建新的`sourceMap`实例。

`payload`是一个对象，其键与[源映射 v3 格式][Source map v3 format]:

*   `file`：{字符串}
*   `version`： {数字}
*   `sources`： {字符串\[]}
*   `sourcesContent`： {字符串\[]}
*   `names`： {字符串\[]}
*   `mappings`：{字符串}
*   `sourceRoot`：{字符串}

#### `sourceMap.payload`

*   返回： {对象}

用于构造的有效负载的 Getter[`SourceMap`][SourceMap]实例。

#### `sourceMap.findEntry(lineNumber, columnNumber)`

*   `lineNumber`{数字}
*   `columnNumber`{数字}
*   返回： {对象}

给定生成的源文件中的行号和列号，返回
表示原始文件中位置的对象。返回的对象
由以下键组成：

*   生成行： {数字}
*   生成的列：{数字}
*   原始来源： {字符串}
*   原始行： {数字}
*   原始列：{数字}
*   名称： {字符串}

[CommonJS]: modules.md

[ES Modules]: esm.md

[Source map v3 format]: https://sourcemaps.info/spec.html#h.mofvlxcwqzej

[`--enable-source-maps`]: cli.md#--enable-source-maps

[`NODE_V8_COVERAGE=dir`]: cli.md#node_v8_coveragedir

[`SourceMap`]: #class-modulesourcemap

[`module`]: modules.md#the-module-object

[module wrapper]: modules.md#the-module-wrapper

[source map include directives]: https://sourcemaps.info/spec.html#h.lmz475t4mvbx
