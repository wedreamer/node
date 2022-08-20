# 模块： 包

<!--introduced_in=v12.20.0-->

<!-- type=misc -->

<!-- YAML
changes:
  - version:
    - v14.13.0
    - v12.20.0
    pr-url: https://github.com/nodejs/node/pull/34718
    description: Add support for `"exports"` patterns.
  - version:
    - v14.6.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34117
    description: Add package `"imports"` field.
  - version:
    - v13.7.0
    - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/29866
    description: Unflag conditional exports.
  - version:
    - v13.7.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31001
    description: Remove the `--experimental-conditional-exports` option. In 12.16.0, conditional exports are still behind `--experimental-modules`.
  - version:
    - v13.6.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31002
    description: Unflag self-referencing a package using its name.
  - version: v12.7.0
    pr-url: https://github.com/nodejs/node/pull/28568
    description:
      Introduce `"exports"` `package.json` field as a more powerful alternative
      to the classic `"main"` field.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/26745
    description:
      Add support for ES modules using `.js` file extension via `package.json`
      `"type"` field.
-->

## 介绍

包是由`package.json`文件。套餐
由包含`package.json`文件和所有子文件夹
直到下一个包含另一个文件夹`package.json`文件或文件夹
叫`node_modules`.

本页为包作者编写提供指导`package.json`文件
以及[`package.json`][package.json]由节点定义的字段.js。

## 确定模块系统

节点.js将以下内容视为[电磁脉冲模块][ES modules]当传递给`node`作为
初始输入，或当引用`import`语句或`import()`
表达 式：

*   具有`.mjs`外延。

*   带有`.js`当最近的父项时扩展`package.json`文件
    包含顶级[`"type"`]["type"]值为`"module"`.

*   作为参数传入的字符串`--eval`，或通过管道传输到`node`通过`STDIN`,
    带旗`--input-type=module`.

节点.js将被视为[CommonJS][]所有其他形式的输入，例如`.js`文件
其中最接近的父项`package.json`文件不包含顶级`"type"`
字段，或不带标志的字符串输入`--input-type`.此行为是
保持向后兼容性。但是，现在 Node.js 同时支持这两种功能。
常见的JS和ES模块，最好尽可能明确。节点.js
将以下内容视为 CommonJS 时传递给`node`作为初始输入，
或当引用时`import`语句`import()`表达式，或
`require()`表达 式：

*   带有`.cjs`外延。

*   带有`.js`当最近的父项时扩展`package.json`文件
    包含顶级字段[`"type"`]["type"]值为`"commonjs"`.

*   作为参数传入的字符串`--eval`或`--print`，或通过管道传输到`node`
    通过`STDIN`，带有旗帜`--input-type=commonjs`.

包作者应包括[`"type"`]["type"]字段，即使在以下情况下的包中
所有来源都是CommonJS。明确说明`type`的包将
使软件包适应未来，以防 Node.js 的默认类型发生变化，以及
它还将使构建工具和加载程序更容易确定如何
包中的文件应该被解释。

### 模块装载机

Node.js有两个系统用于解析说明符和加载模块。

有CommonJS模块加载器：

*   它是完全同步的。
*   它负责处理`require()`调用。
*   它是猴子可修补的。
*   它支持[文件夹作为模块][folders as modules].
*   解析指定符时，如果未找到完全匹配项，它将尝试添加
    扩展 （`.js`,`.json`，最后`.node`），然后尝试解析
    [文件夹作为模块][folders as modules].
*   它治疗`.json`作为 JSON 文本文件。
*   `.node`文件被解释为已编译的插件模块加载
    `process.dlopen()`.
*   它处理所有缺少的文件`.json`或`.node`作为 JavaScript 的扩展
    文本文件。
*   它不能用于加载 ECMAScript 模块（尽管可以
    [从 CommonJS 模块加载 ECMASCript 模块][load ECMASCript modules from CommonJS modules]).当用于加载
    不是 ECMAScript 模块的 JavaScript 文本文件，它将其作为
    通用JS模块。

有 ECMAScript 模块加载程序：

*   它是异步的。
*   它负责处理`import`语句和`import()`表达 式。
*   它不是猴子可修补的，可以使用[装载机吊钩][loader hooks].
*   它不支持文件夹作为模块，目录索引（例如
    `'./startup/index.js'`） 必须完全指定。
*   它不执行扩展名搜索。必须提供文件扩展名
    当说明符是相对或绝对文件 URL 时。
*   它可以加载 JSON 模块，但需要导入断言。
*   它只接受`.js`,`.mjs`和`.cjs`JavaScript 文本的扩展
    文件。
*   它可以用来加载JavaScript CommonJS模块。此类模块
    通过`es-module-lexer`尝试标识命名导出，
    如果可以通过静态分析确定它们，则可用。
    导入的 CommonJS 模块的 URL 已转换为绝对值
    路径，然后通过 CommonJS 模块加载程序加载。

### `package.json`和文件扩展名

在包中，[`package.json`][package.json] [`"type"`]["type"]字段定义如何
节点.js应该解释`.js`文件。如果`package.json`文件没有
`"type"`田`.js`文件被视为[CommonJS][].

一个`package.json` `"type"`的值`"module"`告诉节点.js解释`.js`
文件作为使用[电磁模块][ES module]语法。

这`"type"`字段不仅适用于初始入口点 （`node my-app.js`)
但也引用的文件`import`语句和`import()`表达 式。

```js
// my-app.js, treated as an ES module because there is a package.json
// file in the same folder with "type": "module".

import './startup/init.js';
// Loaded as ES module since ./startup contains no package.json file,
// and therefore inherits the "type" value from one level up.

import 'commonjs-package';
// Loaded as CommonJS since ./node_modules/commonjs-package/package.json
// lacks a "type" field or contains "type": "commonjs".

import './node_modules/commonjs-package/index.js';
// Loaded as CommonJS since ./node_modules/commonjs-package/package.json
// lacks a "type" field or contains "type": "commonjs".
```

以 结尾的文件`.mjs`始终加载为[电磁脉冲模块][ES modules]无论
最近的父级`package.json`.

以 结尾的文件`.cjs`始终加载为[CommonJS][]无论
最近的父项`package.json`.

```js
import './legacy-file.cjs';
// Loaded as CommonJS since .cjs is always loaded as CommonJS.

import 'commonjs-package/src/index.mjs';
// Loaded as ES module since .mjs is always loaded as ES module.
```

这`.mjs`和`.cjs`扩展可用于混合同一类型中的类型
包：

*   在`"type": "module"`包，节点.js可以被指示为
    将特定文件解释为[CommonJS][]通过用`.cjs`
    扩展（因为两者兼而有之`.js`和`.mjs`文件被视为 ES 模块
    一个`"module"`包）。

*   在`"type": "commonjs"`包，节点.js可以被指示为
    将特定文件解释为[电磁模块][ES module]通过用`.mjs`
    扩展（因为两者兼而有之`.js`和`.cjs`文件被视为 CommonJS 在
    `"commonjs"`包）。

### `--input-type`旗

<!-- YAML
added: v12.0.0
-->

作为参数传入的字符串`--eval`（或`-e`），或通过管道传送到`node`通过
`STDIN`，则视为[电磁脉冲模块][ES modules]当`--input-type=module`旗
已设置。

```bash
node --input-type=module --eval "import { sep } from 'node:path'; console.log(sep);"

echo "import { sep } from 'node:path'; console.log(sep);" | node --input-type=module
```

为了完整性，还有`--input-type=commonjs`，用于显式运行
字符串输入作为 CommonJS。这是默认行为，如果`--input-type`是
未指定。

## 确定包管理器

> 稳定性： 1 - 实验

虽然所有 Node.js 项目都应可由所有包安装
经理一旦发布，他们的开发团队经常被要求使用一个
特定的包管理器。为了使此过程更容易，Node.js附带了
工具称为[核心包][Corepack]旨在使所有包管理器透明化
在您的环境中可用 - 前提是安装了 Node.js。

默认情况下，Corepack 不会强制执行任何特定的包管理器，并且将使用
与每个 Node 相关的通用“最近一次已知良好”版本.js版本，
但是，您可以通过设置[`"packageManager"`]["packageManager"]田
在项目的`package.json`.

## 包入口点

在包的`package.json`文件中，两个字段可以定义
包：[`"main"`]["main"]和[`"exports"`]["exports"].这两个字段都适用于两个 ES 模块
和 CommonJS 模块入口点。

这[`"main"`]["main"]字段在所有版本的 Node.js中都受支持，但其
功能是有限的：它只定义了包的主入口点。

这[`"exports"`]["exports"]提供了现代替代方案[`"main"`]["main"]允许
多个入口点待定义，支持条件入口解析
在环境之间，以及**防止除这些入口点以外的任何其他入口点
定义于[`"exports"`]["exports"]**.此封装允许模块作者
清楚地定义其包的公共接口。

对于面向当前支持的 Node.js 版本的新包，
[`"exports"`]["exports"]建议使用字段。对于支持 Node 的包.js 10 和
下面，[`"main"`]["main"]字段为必填字段。如果两者兼而有之[`"exports"`]["exports"]和
[`"main"`]["main"]已定义，[`"exports"`]["exports"]字段优先于
[`"main"`]["main"]在受支持的 Node.js 版本中。

[有条件出口][Conditional exports]可在以下范围内使用[`"exports"`]["exports"]定义不同的
每个环境的包入口点，包括包是否
引用方式`require`或通过`import`.有关支持的更多信息
CommonJS 和 ES 模块都在一个软件包中，请咨询
[双通用JS/ES模块包部分][the dual CommonJS/ES module packages section].

现有软件包介绍[`"exports"`]["exports"]现场会阻止消费者
使用任何未定义的入口点，包括
[`package.json`][package.json]（例如`require('your-package/package.json')`.**这将
可能是一个重大更改。**

要做介绍[`"exports"`]["exports"]不间断，确保每一项
导出以前支持的入口点。最好明确指定
入口点，以便包的公共 API 得到明确定义。例如
以前导出的项目`main`,`lib`,
`feature`，以及`package.json`可以使用以下内容`package.exports`:

```json
{
  "name": "my-package",
  "exports": {
    ".": "./lib/index.js",
    "./lib": "./lib/index.js",
    "./lib/index": "./lib/index.js",
    "./lib/index.js": "./lib/index.js",
    "./feature": "./feature/index.js",
    "./feature/index": "./feature/index.js",
    "./feature/index.js": "./feature/index.js",
    "./package.json": "./package.json"
  }
}
```

或者，项目可以选择导出整个文件夹，同时使用 和
不使用导出模式的扩展子路径：

```json
{
  "name": "my-package",
  "exports": {
    ".": "./lib/index.js",
    "./lib": "./lib/index.js",
    "./lib/*": "./lib/*.js",
    "./lib/*.js": "./lib/*.js",
    "./feature": "./feature/index.js",
    "./feature/*": "./feature/*.js",
    "./feature/*.js": "./feature/*.js",
    "./package.json": "./package.json"
  }
}
```

由于上述内容为任何次要软件包版本提供了向后兼容性，
包装的未来重大变化可以适当地限制出口
仅显示特定功能导出：

```json
{
  "name": "my-package",
  "exports": {
    ".": "./lib/index.js",
    "./feature/*.js": "./feature/*.js",
    "./feature/internal/*": null
  }
}
```

### 主要入口点出口

编写新包时，建议使用[`"exports"`]["exports"]田：

```json
{
  "exports": "./index.js"
}
```

当[`"exports"`]["exports"]字段已定义，包的所有子路径均为
封装，不再提供给进口商。例如
`require('pkg/subpath.js')`抛出一个[`ERR_PACKAGE_PATH_NOT_EXPORTED`][ERR_PACKAGE_PATH_NOT_EXPORTED]
错误。

这种出口的封装提供了更可靠的保证
关于工具的软件包接口以及处理 semver 升级时
包。它不是一个强大的封装，因为直接要求任何
包的绝对子路径，例如
`require('/path/to/node_modules/pkg/subpath.js')`仍将加载`subpath.js`.

所有当前支持的 Node.js 和现代构建工具版本都支持
`"exports"`田。对于使用旧版 Node 的项目.js或相关
构建工具，兼容性可以通过包含`"main"`田
旁边`"exports"`指向同一模块：

```json
{
  "main": "./index.js",
  "exports": "./index.js"
}
```

### 子路径导出

<!-- YAML
added: v12.7.0
-->

使用[`"exports"`]["exports"]字段，可以沿自定义子路径进行定义
将主入口点视为主入口点
`"."`子路径：

```json
{
  "exports": {
    ".": "./index.js",
    "./submodule.js": "./src/submodule.js"
  }
}
```

现在只有定义的子路径[`"exports"`]["exports"]可由使用者导入：

```js
import submodule from 'es-module-package/submodule.js';
// Loads ./node_modules/es-module-package/src/submodule.js
```

而其他子路径将出错：

```js
import submodule from 'es-module-package/private-module.js';
// Throws ERR_PACKAGE_PATH_NOT_EXPORTED
```

#### 子路径中的扩展

包作者应提供扩展 （`import 'pkg/subpath.js'`） 或
无延伸（`import 'pkg/subpath'`） 其导出中的子路径。这确保了
每个导出的模块只有一个子路径，以便所有依赖
导入相同的一致说明符，使包合同保持清晰
使用者和简化包子路径完成。

传统上，软件包倾向于使用无扩展样式，它具有
可读性和屏蔽文件真实路径的好处
包。

跟[导入地图][import maps]现在为浏览器中的包分辨率提供标准
和其他JavaScript运行时，使用无扩展样式可以导致
臃肿的导入映射定义。显式文件扩展名可以通过以下方式避免此问题
使导入映射能够利用[包文件夹映射][packages folder mapping]以映射多个
尽可能使用子路径，而不是每个包子路径使用单独的映射条目
出口。这也反映了使用的要求[完整的说明符路径][the full specifier path]
在相对和绝对导入说明符中。

### 出口糖

<!-- YAML
added: v12.11.0
-->

如果`"."`导出是唯一的导出，[`"exports"`]["exports"]田地提供糖
对于这种情况是直接的[`"exports"`]["exports"]字段值。

```json
{
  "exports": {
    ".": "./index.js"
  }
}
```

可以写成：

```json
{
  "exports": "./index.js"
}
```

### 子路径导入

<!-- YAML
added:
  - v14.6.0
  - v12.19.0
-->

除了[`"exports"`]["exports"]字段，有一个包`"imports"`田
以创建仅适用于从
包本身。

中的条目`"imports"`字段必须始终以 开头`#`以确保他们是
已从外部包说明符中消除歧义。

例如，导入字段可用于获得有条件的好处
内部模块的导出：

```json
// package.json
{
  "imports": {
    "#dep": {
      "node": "dep-node-native",
      "default": "./dep-polyfill.js"
    }
  },
  "dependencies": {
    "dep-node-native": "^1.0.0"
  }
}
```

哪里`import '#dep'`无法获得外部软件包的分辨率
`dep-node-native`（包括其出口依次），而是获得本地
文件`./dep-polyfill.js`相对于其他环境中的包。

与`"exports"`字段，`"imports"`现场许可映射到外部
包。

导入字段的解析规则在其他方面类似于
导出字段。

### 子路径模式

<!-- YAML
added:
  - v14.13.0
  - v12.20.0
changes:
  - version:
    - v16.10.0
    - v14.19.0
    pr-url: https://github.com/nodejs/node/pull/40041
    description: Support pattern trailers in "imports" field.
  - version:
    - v16.9.0
    - v14.19.0
    pr-url: https://github.com/nodejs/node/pull/39635
    description: Support pattern trailers.
-->

对于出口或进口数量较少的包裹，我们建议
显式列出每个导出子路径条目。但对于具有以下条件的软件包
大量子路径，这可能会导致`package.json`膨胀和
维护问题。

对于这些用例，可以改用子路径导出模式：

```json
// ./node_modules/es-module-package/package.json
{
  "exports": {
    "./features/*.js": "./src/features/*.js"
  },
  "imports": {
    "#internal/*.js": "./src/internal/*.js"
  }
}
```

**`*`映射公开嵌套的子路径，因为它是字符串替换语法
只。**

的所有实例`*`在右侧将被替换为
值，包括如果它包含任何值`/`分隔符。

```js
import featureX from 'es-module-package/features/x.js';
// Loads ./node_modules/es-module-package/src/features/x.js

import featureY from 'es-module-package/features/y/y.js';
// Loads ./node_modules/es-module-package/src/features/y/y.js

import internalZ from '#internal/z.js';
// Loads ./node_modules/es-module-package/src/internal/z.js
```

这是一种直接的静态匹配和更换，无需任何特殊处理
对于文件扩展名。包括`"*.js"`在映射的两侧
将公开的包导出限制为仅 JS 文件。

导出静态可枚举的属性与导出一起保持
由于包的单个导出可以通过以下方式确定模式：
将右侧目标模式视为`**`根据列表进行球形
包中的文件。因为`node_modules`出口中禁止路径
目标，此扩展仅依赖于包本身的文件。

要从模式中排除私有子文件夹，`null`目标可用于：

```json
// ./node_modules/es-module-package/package.json
{
  "exports": {
    "./features/*.js": "./src/features/*.js",
    "./features/private-internal/*": null
  }
}
```

```js
import featureInternal from 'es-module-package/features/private-internal/m.js';
// Throws: ERR_PACKAGE_PATH_NOT_EXPORTED

import featureX from 'es-module-package/features/x.js';
// Loads ./node_modules/es-module-package/src/features/x.js
```

### 有条件出口

<!-- YAML
added:
  - v13.2.0
  - v12.16.0
changes:
  - version:
    - v13.7.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31001
    description: Unflag conditional exports.
-->

条件导出提供了一种映射到不同路径的方法，具体取决于
某些条件。CommonJS 和 ES 模块导入都支持它们。

例如，一个包想要为
`require()`和`import`可以写成：

```json
// package.json
{
  "exports": {
    "import": "./index-module.js",
    "require": "./index-require.cjs"
  },
  "type": "module"
}
```

Node.js实现以下条件，按大多数条件的顺序列出
特定到最不具体的条件应定义：

*   `"node-addons"`- 类似于`"node"`并匹配任何节点.js环境。
    此条件可用于提供使用本机C++
    插件，而不是更通用且不依赖的入口点
    在本机插件上。可以通过
    [`--no-addons`旗][--no-addons flag].
*   `"node"`- 匹配任何节点.js环境。可以是 CommonJS 或 ES
    模块文件。*在大多数情况下，显式调用 Node.js平台是
    没必要。*
*   `"import"`- 当包通过加载时匹配`import`或
    `import()`，或通过
    ECMAScript module loader.无论 模块格式如何，都适用
    目标文件。*始终相互排斥`"require"`.*
*   `"require"`- 当包通过加载时匹配`require()`.这
    引用的文件应该可以使用`require()`虽然条件
    匹配，而不考虑目标文件的模块格式。预期
    格式包括 CommonJS、JSON 和本机插件，但不包括 ES 模块
    `require()`不支持它们。*始终相互排斥
    `"import"`.*
*   `"default"`- 始终匹配的通用回退。可以是通用JS
    或 ES 模块文件。*此条件应始终排在最后。*

在[`"exports"`]["exports"]对象，键序很重要。在条件期间
匹配，较早的条目具有更高的优先级，并且优先于以后的条目
条目。*一般规则是，条件应从最具体到
对象顺序中的最低特定*.

使用`"import"`和`"require"`条件可能导致一些危害，
进一步解释在[双通用JS/ES模块包部分][the dual CommonJS/ES module packages section].

这`"node-addons"`条件可用于提供入口点
使用本机C++插件。但是，可以通过
[`--no-addons`旗][--no-addons flag].使用时`"node-addons"`，建议治疗
`"default"`作为一种增强功能，提供更通用的入口点，例如
使用WebAssembly而不是本机插件。

条件导出也可以扩展到导出子路径，例如：

```json
{
  "exports": {
    ".": "./index.js",
    "./feature.js": {
      "node": "./feature-node.js",
      "default": "./feature.js"
    }
  }
}
```

定义一个包，其中`require('pkg/feature.js')`和
`import 'pkg/feature.js'`可以提供不同的实现
Node.js和其他JS环境。

使用环境分支时，请始终包含`"default"`条件，其中
可能。提供`"default"`条件确保任何未知的JS
环境能够使用这种通用实现，这有助于避免
这些JS环境不必假装是现有环境
以支持有条件导出的包。因此，使用
`"node"`和`"default"`条件分支通常优于使用
`"node"`和`"browser"`条件分支。

### 嵌套条件

除了直接映射之外，Node.js还支持嵌套条件对象。

例如，要定义一个仅具有 双模式入口点的包
在 Node.js 但不在浏览器中使用：

```json
{
  "exports": {
    "node": {
      "import": "./feature-node.mjs",
      "require": "./feature-node.cjs"
    },
    "default": "./feature.mjs"
  }
}
```

与平坦条件一样，条件继续按顺序匹配。如果
嵌套条件没有任何映射，它将继续检查
父条件的其余条件。以这种方式嵌套
条件的行为类似于嵌套的 JavaScript`if`语句。

### 解决用户条件

<!-- YAML
added:
  - v14.9.0
  - v12.19.0
-->

运行 Node.js 时，可以使用
`--conditions`旗：

```bash
node --conditions=development index.js
```

然后，这将解决`"development"`包装进口条件和
导出，同时解析现有的`"node"`,`"node-addons"`,`"default"`,
`"import"`和`"require"`条件酌情而定。

可以使用重复标志设置任意数量的自定义条件。

### 社区条件定义

条件字符串`"import"`,`"require"`,`"node"`,
`"node-addons"`和`"default"`条件
[在 Node.js 核心中实现](#conditional-exports)默认情况下忽略。

其他平台可以实现其他条件，用户条件可以是
在 Node 中启用.js通过[`--conditions`/`-C`旗][--conditions / -C flag].

由于自定义包装条件需要明确的定义以确保正确
用法，常见已知包条件及其严格定义的列表
下面提供，以协助生态系统协调。

*   `"types"`- 可以通过键入系统来解析键入文件
    给定的导出。*应始终首先包含此条件。*
*   `"deno"`- 表示 Deno 平台的变体。
*   `"browser"`- 任何网络浏览器环境。
*   `"development"`- 可用于定义仅开发环境
    入口点，例如提供额外的调试上下文，例如
    在开发模式下运行时的错误消息更好。*必须始终
    相互排斥`"production"`.*
*   `"production"`- 可用于定义生产环境条目
    点。*必须始终与相互排斥`"development"`.*

通过创建拉取请求，可以将新的条件定义添加到此列表中
到[本节的节点.js文档][Node.js documentation for this section].上市要求
这里的一个新的条件定义是：

*   对于所有实施者来说，定义应该是清晰和明确的。
*   为什么需要条件的用例应该清楚地证明是合理的。
*   应该有足够的现有实现用法。
*   条件名称不应与其他条件定义冲突，或者
    广泛使用的条件。
*   条件定义的列表应提供协调
    为生态系统带来好处，否则是不可能的。例如
    对于公司特定的或
    特定于应用程序的条件。

上述定义可移至专用条件登记处
课程。

### 使用包的名称自引用包

<!-- YAML
added:
  - v13.1.0
  - v12.16.0
changes:
  - version:
    - v13.6.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31002
    description: Unflag self-referencing a package using its name.
-->

在包中，包中定义的值
`package.json` [`"exports"`]["exports"]字段可以通过包的名称进行引用。
例如，假设`package.json`是：

```json
// package.json
{
  "name": "a-package",
  "exports": {
    ".": "./index.mjs",
    "./foo.js": "./foo.js"
  }
}
```

然后是任何模块*在该包中*可以在包本身中引用导出：

```js
// ./a-module.mjs
import { something } from 'a-package'; // Imports "something" from ./index.mjs.
```

仅当以下情况时，自引用才可用`package.json`有[`"exports"`]["exports"]和
将只允许导入什么[`"exports"`]["exports"]（在`package.json`)
允许。因此，给定上一个包，下面的代码将生成一个运行时
错误：

```js
// ./another-module.mjs

// Imports "another" from ./m.mjs. Fails because
// the "package.json" "exports" field
// does not provide an export named "./m.mjs".
import { another } from 'a-package/m.mjs';
```

使用时也可使用自引用`require`，两者都在 ES 模块中，
和在CommonJS中。例如，此代码也适用于：

```cjs
// ./a-module.js
const { something } = require('a-package/foo.js'); // Loads from ./foo.js.
```

最后，自引用也适用于作用域包。例如，这个
代码也将工作：

```json
// package.json
{
  "name": "@my/package",
  "exports": "./index.js"
}
```

```cjs
// ./index.js
module.exports = 42;
```

```cjs
// ./other.js
console.log(require('@my/package'));
```

```console
$ node other.js
42
```

## 双通用JS/ES模块包

在 Node.js 中引入对 ES 模块的支持之前，它是一种常见的
包作者的模式，包括 CommonJS 和 ES 模块 JavaScript
源在其包中，具有`package.json` [`"main"`]["main"]指定
CommonJS 入口点和`package.json` `"module"`指定 ES 模块
入口点。
此启用的 Node.js在构建工具（如
捆绑器使用ES模块入口点，因为Node.js忽略（并且仍然
忽略） 顶层`"module"`田。

Node.js现在可以运行 ES 模块入口点，并且包可以同时包含两者
通用JS和ES模块入口点（通过单独的说明符，例如
`'pkg'`和`'pkg/es-module'`，或两者位于同一说明符处，通过[有條件的
出口][Conditional
exports]).与以下场景不同`"module"`仅由捆绑器使用，
或者 ES 模块文件在评估之前动态转译到 CommonJS 中
节点.js，ES 模块入口点引用的文件被评估为 ES
模块。

### 双重包装危险

当应用程序使用同时提供 CommonJS 和 ES 模块的包时
源，如果两个版本的软件包都得到，则存在某些错误的风险
加载。这种潜力来自这样一个事实：`pkgInstance`创建者
`const pkgInstance = require('pkg')`与`pkgInstance`
创建者`import pkgInstance from 'pkg'`（或替代主要路径，如
`'pkg/module'`).这是“双重包装危险”，其中两个版本的
可以在同一运行时环境中加载相同的包。虽然它是
应用程序或包不太可能有意加载这两个版本
直接地说，应用程序在依赖关系时加载一个版本是很常见的
的应用程序加载其他版本。这种危险可能发生，因为
Node.js支持混合CommonJS和ES模块，并可能导致意外
行为。

如果包主导出是构造函数，则`instanceof`比较
由两个版本创建的实例返回`false`，并且如果导出是
对象，添加到一个属性（如`pkgInstance.foo = 3`） 不存在于
另一个。这与方式不同`import`和`require`语句工作在
所有通用JS或全ES模块环境，分别是
让用户感到惊讶。它也不同于用户熟悉的行为
通过以下工具使用转译时[巴别塔][Babel]或[`esm`][esm].

### 编写双包装，同时避免或最大限度地减少危险

首先，上一节中描述的危险发生在包装时
包含 CommonJS 和 ES 模块源代码，并且这两个源代码都针对
在 Node.js中使用，通过单独的主入口点或导出的路径。一个
包可能被写入任何版本的 Node.js 仅接收
通用 JS 源，以及包可能包含的任何单独的 ES 模块源
仅适用于其他环境，如浏览器。这样的一揽子计划
任何版本的 Node.js 都可以使用，因为`import`可以参考CommonJS
文件;但它不会提供使用ES模块语法的任何优点。

包还可以在[打破
改变](https://semver.org/)版本颠簸。这有一个缺点，
最新版本的软件包只能在ES模块支持中使用
节点的版本.js。

每种模式都有权衡，但有两种广泛的方法可以满足
以下情况：

1.  该软件包可通过两种方式使用`require`和`import`.
2.  该软件包可在当前 Node.js 和旧版本的 Node 中使用.js
    缺乏对ES模块的支持。
3.  包装主入口点，例如`'pkg'`可由两者使用`require`自
    解析为 CommonJS 文件并由`import`解析为 ES 模块文件。
    （同样适用于导出的路径，例如`'pkg/feature'`.)
4.  该软件包提供命名导出，例如`import { name } from 'pkg'`而
    比`import pkg from 'pkg'; pkg.name`.
5.  该软件包可能在其他ES模块环境中使用，例如
    浏览器。
6.  避免或最小化上一节中描述的危害。

#### 方法#1：使用ES模块包装器

将包写入 CommonJS 或将 ES 模块源代码转译为 CommonJS，以及
创建一个定义命名导出的 ES 模块包装器文件。用
[有条件出口][Conditional exports]，则 ES 模块包装器用于`import`和
通用 JS 入口点`require`.

```json
// ./node_modules/pkg/package.json
{
  "type": "module",
  "exports": {
    "import": "./wrapper.mjs",
    "require": "./index.cjs"
  }
}
```

前面的示例使用显式扩展`.mjs`和`.cjs`.
如果您的文件使用`.js`外延`"type": "module"`将导致此类文件
被视为 ES 模块，就像`"type": "commonjs"`会导致他们
被视为 CommonJS。
看[使](esm.md#enabling).

```cjs
// ./node_modules/pkg/index.cjs
exports.name = 'value';
```

```js
// ./node_modules/pkg/wrapper.mjs
import cjsModule from './index.cjs';
export const name = cjsModule.name;
```

在此示例中，`name`从`import { name } from 'pkg'`是一样的
单例作为`name`从`const { name } = require('pkg')`.因此`===`
返回`true`比较两者时`name`s 和发散说明符危险
避免。

如果模块不仅仅是命名导出的列表，而是包含
独特的功能或对象导出，如`module.exports = function () { ... }`,
或者如果支持包装器中的`import pkg from 'pkg'`模式是需要的，
然后，将编写包装器以选择性地导出默认值
以及任何命名的导出：

```js
import cjsModule from './index.cjs';
export const name = cjsModule.name;
export default cjsModule;
```

此方法适用于以下任何用例：

*   该软件包目前是用CommonJS编写的，作者不希望
    将其重构为 ES 模块语法，但希望为
    ES 模块消费者。
*   该包具有依赖于它的其他包，最终用户可能会
    同时安装此包和其他包。例如`utilities`
    包直接在应用程序中使用，并且`utilities-plus`包
    将更多功能添加到`utilities`.因为包装器导出
    底层的 CommonJS 文件，如果`utilities-plus`写在
    通用JS或ES模块语法;无论哪种方式，它都可以工作。
*   包存储内部状态，包作者不希望
    重构包以隔离其状态管理。请参阅下一节。

这种方法的变体不需要消费者有条件出口，可以
添加导出，例如`"./module"`，指向全 ES 模块语法
包的版本。这可以通过以下方式使用`import 'pkg/module'`按用户
谁确定 CommonJS 版本不会加载到
应用程序，例如按依赖关系;或者是否可以加载 CommonJS 版本
但不影响 ES 模块版本（例如，因为包是
无状态）：

```json
// ./node_modules/pkg/package.json
{
  "type": "module",
  "exports": {
    ".": "./index.cjs",
    "./module": "./wrapper.mjs"
  }
}
```

#### 方法#2：隔离状态

一个[`package.json`][package.json]文件可以定义单独的 CommonJS 和 ES 模块条目
直接积分：

```json
// ./node_modules/pkg/package.json
{
  "type": "module",
  "exports": {
    "import": "./index.mjs",
    "require": "./index.cjs"
  }
}
```

如果软件包的 CommonJS 和 ES 模块版本都是
等价，例如，因为一个是另一个的转译输出;和
包的状态管理被仔细隔离（或者包是
无状态）。

状态之所以成为问题，是因为 CommonJS 和 ES 模块都是如此。
软件包的版本可能会在应用程序中使用;例如，
用户的应用程序代码可以`import`ES 模块版本，而依赖项
`require`s 的 CommonJS 版本。如果发生这种情况，则两个副本
包将加载到内存中，因此两个单独的状态将是
目前。这可能会导致难以排除故障的错误。

除了编写无状态包（如果JavaScript的`Math`是一个包，
例如，它将是无状态的，因为它的所有方法都是静态的），有
隔离状态的一些方法，以便在潜在加载者之间共享状态
包的通用JS和ES模块实例：

1.  如果可能，请包含实例化对象中的所有状态。JavaScript's
    `Date`，例如，需要实例化以包含状态;如果它是一个
    包，它将像这样使用：

    ```js
    import Date from 'date';
    const someDate = new Date();
    // someDate contains state; Date does not
    ```

    这`new`关键字不是必需的;包的函数可以返回新的
    对象，或修改传入的对象，以保持状态在
    包。

2.  隔离在
    该软件包的通用JS和ES模块版本。例如，如果 CommonJS
    和 ES 模块入口点是`index.cjs`和`index.mjs`分别：

    ```cjs
    // ./node_modules/pkg/index.cjs
    const state = require('./state.cjs');
    module.exports.state = state;
    ```

    ```js
    // ./node_modules/pkg/index.mjs
    import state from './state.cjs';
    export {
      state
    };
    ```

    便`pkg`通过两者使用`require`和`import`在应用程序中（用于
    示例，通过`import`在应用程序代码中，并通过`require`通过依赖关系）
    每个参考文献`pkg`将包含相同的状态;并修改
    来自任一模块系统的状态将同时应用于两者。

任何附加到包的单例的插件都需要单独
附加到 CommonJS 和 ES 模块单例。

此方法适用于以下任何用例：

*   包当前以 ES 模块语法和包作者编写
    希望在支持此类语法的任何地方使用该版本。
*   封装是无状态的，或者可以隔离其状态而不会太多
    困难。
*   该软件包不太可能有其他依赖于它的公共软件包，或者如果
    确实如此，包是无状态的或具有不需要在它们之间共享的状态
    依赖关系或与整个应用程序。

即使使用隔离状态，仍然存在可能的额外代码的成本
在包的 CommonJS 和 ES 模块版本之间执行。

与以前的方法一样，此方法的变体不需要
消费者的条件出口可以是增加出口，例如
`"./module"`，指向包的全 ES 模块语法版本：

```json
// ./node_modules/pkg/package.json
{
  "type": "module",
  "exports": {
    ".": "./index.cjs",
    "./module": "./index.mjs"
  }
}
```

## 节点.js`package.json`字段定义

本节介绍 Node.js 运行时使用的字段。其他工具（如
如[npm](https://docs.npmjs.com/cli/v8/configuring-npm/package-json)） 使用
其他字段被 Node 忽略.js，此处未记录。

中的以下字段`package.json`文件用于 Node.js：

*   [`"name"`]["name"]- 在包中使用命名导入时相关。也用过
    由包管理器作为包的名称。
*   [`"main"`]["main"]- 加载包时的默认模块，如果导出不是
    指定，并且在 Node 的版本中.js引入导出之前。
*   [`"packageManager"`]["packageManager"]- 贡献时推荐的包管理器
    包。杠杆由[核心包][Corepack]垫片。
*   [`"type"`]["type"]- 决定是否加载的包装类型`.js`文件作为
    通用JS或ES模块。
*   [`"exports"`]["exports"]- 包装出口和有条件出口。当存在时，
    限制可以从包中加载哪些子模块。
*   [`"imports"`]["imports"]- 包导入，供包中的模块使用
    本身。

### `"name"`

<!-- YAML
added:
  - v13.1.0
  - v12.16.0
changes:
  - version:
    - v13.6.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31002
    description: Remove the `--experimental-resolve-self` option.
-->

*   类型： {字符串}

```json
{
  "name": "package-name"
}
```

这`"name"`字段定义包的名称。发布到
*npm*注册表需要满足以下条件的名称
[某些要求](https://docs.npmjs.com/files/package.json#name).

这`"name"`字段可用于除[`"exports"`]["exports"]字段到
[自我参考][self-reference]使用其名称的包。

### `"main"`

<!-- YAML
added: v0.4.0
-->

*   类型： {字符串}

```json
{
  "main": "./index.js"
}
```

这`"main"`字段定义按名称导入时包的入口点
通过`node_modules`查找。 其值为路径。

当包具有[`"exports"`]["exports"]字段，这将优先于
`"main"`字段（按名称导入包时）。

它还定义了在[包目录已加载
通过`require()`](modules.md#folders-as-modules).

```cjs
// This resolves to ./path/to/directory/index.js.
require('./path/to/directory');
```

### `"packageManager"`

<!-- YAML
added:
  - v16.9.0
  - v14.19.0
-->

> 稳定性： 1 - 实验

*   类型： {字符串}

```json
{
  "packageManager": "<package manager name>@<version>"
}
```

这`"packageManager"`字段定义应为哪个包管理器
在处理当前项目时使用。它可以设置为
[支持的包管理器][supported package managers]，并将确保您的团队使用确切的
相同的包管理器版本，无需安装任何其他内容
节点.js。

该领域目前正在实验性，需要选择加入;检查
[核心包][Corepack]页，了解有关该过程的详细信息。

### `"type"`

<!-- YAML
added: v12.0.0
changes:
  - version:
    - v13.2.0
    - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/29866
    description: Unflag `--experimental-modules`.
-->

*   类型： {字符串}

这`"type"`字段定义 Node.js 用于所有
`.js`具有该文件的文件`package.json`文件作为其最近的父级。

以 结尾的文件`.js`当最接近的父级时，作为 ES 模块加载
`package.json`文件包含顶级字段`"type"`值为
`"module"`.

最近的父项`package.json`被定义为第一个`package.json`发现
在当前文件夹、该文件夹的父文件夹等中搜索时
直到到达node_modules文件夹或卷根目录。

```json
// package.json
{
  "type": "module"
}
```

```bash
# In same folder as preceding package.json
node my-app.js # Runs as ES module
```

如果最近的父项`package.json`缺少`"type"`字段，或包含
`"type": "commonjs"`,`.js`文件被视为[CommonJS][].如果卷
已到达根目录，但未到达`package.json`已找到，`.js`文件被视为
[CommonJS][].

`import`声明`.js`文件被视为 ES 模块，如果最接近
父母`package.json`包含`"type": "module"`.

```js
// my-app.js, part of the same example as above
import './startup.js'; // Loaded as ES module because of package.json
```

无论`"type"`田`.mjs`文件始终得到处理
作为 ES 模块和`.cjs`文件始终被视为 CommonJS。

### `"exports"`

<!-- YAML
added: v12.7.0
changes:
  - version:
    - v14.13.0
    - v12.20.0
    pr-url: https://github.com/nodejs/node/pull/34718
    description: Add support for `"exports"` patterns.
  - version:
    - v13.7.0
    - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/29866
    description: Unflag conditional exports.
  - version:
    - v13.7.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31008
    description: Implement logical conditional exports ordering.
  - version:
    - v13.7.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/31001
    description: Remove the `--experimental-conditional-exports` option. In 12.16.0, conditional exports are still behind `--experimental-modules`.
  - version:
    - v13.2.0
    - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/29978
    description: Implement conditional exports.
-->

*   类型： {对象} |{字符串} |{字符串\[]}

```json
{
  "exports": "./index.js"
}
```

这`"exports"`字段允许定义[入口点][entry points]的包在
按名称导入，通过`node_modules`查找或
[自我参考][self-reference]以自己的名字命名。它在 Node 中受支持.js 12+ 作为
替代[`"main"`]["main"]可以支持定义[子路径导出][subpath exports]
和[有条件出口][conditional exports]同时封装内部未导出的模块。

[有条件出口][Conditional Exports]也可用于`"exports"`定义不同的
每个环境的包入口点，包括包是否
引用方式`require`或通过`import`.

中定义的所有路径`"exports"`必须是相对文件 URL，开头为
`./`.

### `"imports"`

<!-- YAML
added:
 - v14.6.0
 - v12.19.0
-->

*   类型： {对象}

```json
// package.json
{
  "imports": {
    "#dep": {
      "node": "dep-node-native",
      "default": "./dep-polyfill.js"
    }
  },
  "dependencies": {
    "dep-node-native": "^1.0.0"
  }
}
```

导入字段中的条目必须是以下列开头的字符串`#`.

包导入允许映射到外部包。

此字段定义[子路径导入][subpath imports]对于当前包。

[Babel]: https://babeljs.io/

[CommonJS]: modules.md

[Conditional exports]: #conditional-exports

[Corepack]: corepack.md

[ES module]: esm.md

[ES modules]: esm.md

[Node.js documentation for this section]: https://github.com/nodejs/node/blob/HEAD/doc/api/packages.md#conditions-definitions

[`"exports"`]: #exports

[`"imports"`]: #imports

[`"main"`]: #main

[`"name"`]: #name

[`"packageManager"`]: #packagemanager

[`"type"`]: #type

[`--conditions` / `-C` flag]: #resolving-user-conditions

[`--no-addons` flag]: cli.md#--no-addons

[`ERR_PACKAGE_PATH_NOT_EXPORTED`]: errors.md#err_package_path_not_exported

[`esm`]: https://github.com/standard-things/esm#readme

[`package.json`]: #nodejs-packagejson-field-definitions

[entry points]: #package-entry-points

[folders as modules]: modules.md#folders-as-modules

[import maps]: https://github.com/WICG/import-maps

[load ECMASCript modules from CommonJS modules]: modules.md#the-mjs-extension

[loader hooks]: esm.md#loaders

[packages folder mapping]: https://github.com/WICG/import-maps#packages-via-trailing-slashes

[self-reference]: #self-referencing-a-package-using-its-name

[subpath exports]: #subpath-exports

[subpath imports]: #subpath-imports

[supported package managers]: corepack.md#supported-package-managers

[the dual CommonJS/ES module packages section]: #dual-commonjses-module-packages

[the full specifier path]: esm.md#mandatory-file-extensions
