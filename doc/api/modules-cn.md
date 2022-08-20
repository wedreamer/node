# 模块：通用JS模块

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!--name=module-->

CommonJS模块是为Node.js打包JavaScript代码的原始方法。
节点.js还支持[ECMAScript 模块][ECMAScript modules]浏览器使用的标准
和其他 JavaScript 运行时。

在 Node.js 中，每个文件都被视为一个单独的模块。为
例如，考虑一个名为`foo.js`:

```js
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
```

在第一行，`foo.js`加载模块`circle.js`这是在同一
目录作为`foo.js`.

以下是`circle.js`:

```js
const { PI } = Math;

exports.area = (r) => PI * r ** 2;

exports.circumference = (r) => 2 * PI * r;
```

模块`circle.js`已导出函数`area()`和
`circumference()`.将函数和对象添加到模块的根目录
通过在特殊`exports`对象。

模块的局部变量将是私有的，因为模块是包装的
在 Node 的函数中.js（请参见[模块包装器](#the-module-wrapper)).
在此示例中，变量`PI`是私有的`circle.js`.

这`module.exports`可以为属性分配一个新值（如函数
或对象）。

下面`bar.js`利用`square`模块，它导出一个 Square 类：

```js
const Square = require('./square.js');
const mySquare = new Square(2);
console.log(`The area of mySquare is ${mySquare.area()}`);
```

这`square`模块定义于`square.js`:

```js
// Assigning to exports will not modify module, must use module.exports
module.exports = class Square {
  constructor(width) {
    this.width = width;
  }

  area() {
    return this.width ** 2;
  }
};
```

CommonJS模块系统在[`module`核心模块][module core module].

## 使

<!-- type=misc -->

Node.js有两个模块系统：CommonJS模块和[ECMAScript 模块][ECMAScript modules].

默认情况下，Node.js 会将以下内容视为 CommonJS 模块：

*   带有`.cjs`扩展;

*   带有`.js`当最近的父项时扩展`package.json`文件
    包含顶级字段[`"type"`]["type"]值为`"commonjs"`.

*   带有`.js`当最近的父项时扩展`package.json`文件
    不包含顶级字段[`"type"`]["type"].包作者应包括
    这[`"type"`]["type"]字段，即使在所有源都是 CommonJS 的包中也是如此。存在
    显式关于`type`的包将使事情更容易构建
    工具和加载程序，用于确定包中的文件应如何
    解释。

*   扩展名不是`.mjs`,`.cjs`,`.json`,`.node`或`.js`
    （当最接近的父母`package.json`文件包含顶级字段
    [`"type"`]["type"]值为`"module"`，这些文件将被识别为
    通用JS模块仅当它们正在`require`d、不用作
    程序的命令行入口点）。

看[确定模块系统][Determining module system]了解更多详情。

叫`require()`始终使用 CommonJS 模块加载程序。叫`import()`
始终使用 ECMAScript 模块加载程序。

## 访问主模块

<!-- type=misc -->

当文件直接从 Node 运行时.js，`require.main`设置为
`module`.这意味着可以确定文件是否已
通过测试直接运行`require.main === module`.

对于文件`foo.js`，这将是`true`如果运行方式`node foo.js`但
`false`如果由`require('./foo')`.

当入口点不是 CommonJS 模块时，`require.main`是`undefined`,
并且主模块遥不可及。

## 包管理器提示

<!-- type=misc -->

节点的语义.js`require()`功能被设计为通用
足以支持合理的目录结构。包管理器程序
如`dpkg`,`rpm`和`npm`希望能找到可以构建
来自 Node 的本机包.js模块，无需修改。

下面我们给出了一个可以工作的推荐目录结构：

假设我们希望将文件夹设在
`/usr/lib/node/<some-package>/<some-version>`保存的内容
包的特定版本。

包可以相互依赖。为了安装软件包`foo`它
可能需要安装特定版本的软件包`bar`.这`bar`
包本身可能有依赖关系，在某些情况下，这些甚至可能冲突
或形成循环依赖关系。

因为 Node.js 查找`realpath`它加载的任何模块（即
解析符号链接），然后[在 中查找它们的依赖关系`node_modules`文件夹](#loading-from-node_modules-folders),
这种情况可以通过以下体系结构解决：

*   `/usr/lib/node/foo/1.2.3/`： 内容`foo`包，版本 1.2.3。
*   `/usr/lib/node/bar/4.3.2/`： 内容`bar`包`foo`取决于
    上。
*   `/usr/lib/node/foo/1.2.3/node_modules/bar`：符号链接到
    `/usr/lib/node/bar/4.3.2/`.
*   `/usr/lib/node/bar/4.3.2/node_modules/*`：指向以下软件包的符号链接：
    `bar`取决于。

因此，即使遇到循环，或者存在依赖性
冲突，每个模块都能够获得其依赖项的一个版本
它可以使用。

当代码在`foo`包做`require('bar')`，它将得到
符号链接到 的版本`/usr/lib/node/foo/1.2.3/node_modules/bar`.
然后，当代码在`bar`包调用`require('quux')`，它会得到
符号链接到的版本
`/usr/lib/node/bar/4.3.2/node_modules/quux`.

此外，为了使模块查找过程更加优化，而不是
比直接放入包装`/usr/lib/node`，我们可以把它们放进去
`/usr/lib/node_modules/<name>/<version>`.那么Node.js就不会打扰了
在 中查找缺少的依赖项`/usr/node_modules`或`/node_modules`.

为了使模块可用于 Node.js REPL，它可能有助于
还添加`/usr/lib/node_modules`文件夹`$NODE_PATH`环境
变量。由于模块查找使用`node_modules`文件夹全部
相对的，并基于文件的真实路径进行调用
`require()`，则包本身可以在任何地方。

## 这`.mjs`外延

由于`require()`，则不能将其用于
加载 ECMAScript 模块文件。尝试这样做将抛出一个
[`ERR_REQUIRE_ESM`][ERR_REQUIRE_ESM]错误。用[`import()`][import()]相反。

这`.mjs`扩展名保留给[ECMAScript Modules][]这不能是
加载方式`require()`.看[确定模块系统][Determining module system]部分以获取更多信息
关于哪些文件被解析为 ECMAScript 模块。

## 一起

<!-- type=misc -->

要获取将在以下情况下加载的确切文件名`require()`被调用，使用
这`require.resolve()`功能。

将上述所有内容放在一起，这是高级算法
在伪代码中什么`require()`执行：

<pre>
require(X) from module at path Y
1. If X is a core module,
   a. return the core module
   b. STOP
2. If X begins with '/'
   a. set Y to be the filesystem root
3. If X begins with './' or '/' or '../'
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
   c. THROW "not found"
4. If X begins with '#'
   a. LOAD_PACKAGE_IMPORTS(X, dirname(Y))
5. LOAD_PACKAGE_SELF(X, dirname(Y))
6. LOAD_NODE_MODULES(X, dirname(Y))
7. THROW "not found"

LOAD_AS_FILE(X)
1. If X is a file, load X as its file extension format. STOP
2. If X.js is a file, load X.js as JavaScript text. STOP
3. If X.json is a file, parse X.json to a JavaScript Object. STOP
4. If X.node is a file, load X.node as binary addon. STOP

LOAD_INDEX(X)
1. If X/index.js is a file, load X/index.js as JavaScript text. STOP
2. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
3. If X/index.node is a file, load X/index.node as binary addon. STOP

LOAD_AS_DIRECTORY(X)
1. If X/package.json is a file,
   a. Parse X/package.json, and look for "main" field.
   b. If "main" is a falsy value, GOTO 2.
   c. let M = X + (json main field)
   d. LOAD_AS_FILE(M)
   e. LOAD_INDEX(M)
   f. LOAD_INDEX(X) DEPRECATED
   g. THROW "not found"
2. LOAD_INDEX(X)

LOAD_NODE_MODULES(X, START)
1. let DIRS = NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_PACKAGE_EXPORTS(X, DIR)
   b. LOAD_AS_FILE(DIR/X)
   c. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   b. DIR = path join(PARTS[0 .. I] + "node_modules")
   c. DIRS = DIR + DIRS
   d. let I = I - 1
5. return DIRS + GLOBAL_FOLDERS

LOAD_PACKAGE_IMPORTS(X, DIR)
1. Find the closest package scope SCOPE to DIR.
2. If no scope was found, return.
3. If the SCOPE/package.json "imports" is null or undefined, return.
4. let MATCH = PACKAGE_IMPORTS_RESOLVE(X, pathToFileURL(SCOPE),
  ["node", "require"]) <a href="esm.md#resolver-algorithm-specification">defined in the ESM resolver</a>.
5. RESOLVE_ESM_MATCH(MATCH).

LOAD_PACKAGE_EXPORTS(X, DIR)
1. Try to interpret X as a combination of NAME and SUBPATH where the name
   may have a @scope/ prefix and the subpath begins with a slash (`/`).
2. If X does not match this pattern or DIR/NAME/package.json is not a file,
   return.
3. Parse DIR/NAME/package.json, and look for "exports" field.
4. If "exports" is null or undefined, return.
5. let MATCH = PACKAGE_EXPORTS_RESOLVE(pathToFileURL(DIR/NAME), "." + SUBPATH,
   `package.json` "exports", ["node", "require"]) <a href="esm.md#resolver-algorithm-specification">defined in the ESM resolver</a>.
6. RESOLVE_ESM_MATCH(MATCH)

LOAD_PACKAGE_SELF(X, DIR)
1. Find the closest package scope SCOPE to DIR.
2. If no scope was found, return.
3. If the SCOPE/package.json "exports" is null or undefined, return.
4. If the SCOPE/package.json "name" is not the first segment of X, return.
5. let MATCH = PACKAGE_EXPORTS_RESOLVE(pathToFileURL(SCOPE),
   "." + X.slice("name".length), `package.json` "exports", ["node", "require"])
   <a href="esm.md#resolver-algorithm-specification">defined in the ESM resolver</a>.
6. RESOLVE_ESM_MATCH(MATCH)

RESOLVE_ESM_MATCH(MATCH)
1. let { RESOLVED, EXACT } = MATCH
2. let RESOLVED_PATH = fileURLToPath(RESOLVED)
3. If EXACT is true,
   a. If the file at RESOLVED_PATH exists, load RESOLVED_PATH as its extension
      format. STOP
4. Otherwise, if EXACT is false,
   a. LOAD_AS_FILE(RESOLVED_PATH)
   b. LOAD_AS_DIRECTORY(RESOLVED_PATH)
5. THROW "not found"
</pre>

## 缓存

<!--type=misc-->

模块在首次加载后进行缓存。这意味着（除其他外）
事物）每次调用`require('foo')`将获得完全相同的对象
返回，如果它将解析为同一文件。

提供`require.cache`未修改，多次调用`require('foo')`
不会导致模块代码多次执行。这是一个
重要功能。有了它，可以返回“部分完成”对象，因此
允许加载传递依赖项，即使它们会导致循环。

要让模块多次执行代码，请导出一个函数，然后调用该函数
功能。

### 模块缓存注意事项

<!--type=misc-->

模块根据其解析的文件名进行缓存。由于模块可以解析
根据调用模块的位置使用不同的文件名（加载
从`node_modules`文件夹），它不是*保证*那`require('foo')`将
如果将解析为不同的文件，则始终返回完全相同的对象。

此外，在不区分大小写的文件系统或操作系统上，不同的
解析的文件名可以指向同一文件，但缓存仍将处理
它们作为不同的模块，并将多次重新加载文件。例如
`require('./foo')`和`require('./FOO')`返回两个不同的对象，
无论是否`./foo`和`./FOO`是同一个文件。

## 核心模块

<!--type=misc-->

<!-- YAML
changes:
  - version:
      - v16.0.0
      - v14.18.0
    pr-url: https://github.com/nodejs/node/pull/37246
    description: Added `node:` import support to `require(...)`.
-->

Node.js有几个模块编译到二进制文件中。这些模块是
本文档其他部分对此进行了更详细的描述。

核心模块在 Node.js 源中定义，位于
`lib/`文件夹。

核心模块可以使用`node:`前缀，在这种情况下
它绕过了`require`缓存。例如`require('node:http')`将
始终返回内置的 HTTP 模块，即使有`require.cache`进入
通过这个名字。

某些核心模块始终优先加载，如果其标识符为
传递到`require()`.例如`require('http')`将永远
返回内置的 HTTP 模块，即使存在具有该名称的文件。列表
无需使用`node:`前缀已公开
如[`module.builtinModules`][module.builtinModules].

## 周期

<!--type=misc-->

当有圆形时`require()`调用，模块可能尚未完成
返回时执行。

请考虑以下情况：

`a.js`:

```js
console.log('a starting');
exports.done = false;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');
```

`b.js`:

```js
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');
```

`main.js`:

```js
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```

什么时候`main.js`负荷`a.js`然后`a.js`依次加载`b.js`.在那
点`b.js`尝试加载`a.js`.为了防止无限
循环，一个**未完成的副本**的`a.js`导出对象返回到
`b.js`模块。`b.js`然后完成加载，并且`exports`对象是
提供给`a.js`模块。

按时间`main.js`已经加载了两个模块，它们都完成了。
因此，该程序的输出将是：

```console
$ node main.js
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```

需要仔细规划以允许循环模块依赖关系正常工作
在应用程序中正确。

## 文件模块

<!--type=misc-->

如果未找到确切的文件名，则 Node.js 将尝试加载
带有添加扩展名的必需文件名：`.js`,`.json`，最后
`.node`.加载具有不同扩展名的文件时（例如`.cjs`），其
必须将全名传递给`require()`，包括其文件扩展名（例如
`require('./file.cjs')`).

`.json`文件被解析为 JSON 文本文件，`.node`文件被解释为
已编译的插件模块加载`process.dlopen()`.使用任何其他文件的文件
扩展名（或根本没有扩展名）被解析为JavaScript文本文件。指
这[确定模块系统][Determining module system]部分以了解解析目标是什么
使用。

以 前缀为前缀的必需模块`'/'`是文件的绝对路径。为
例`require('/home/marco/foo.js')`将在
`/home/marco/foo.js`.

以 前缀为前缀的必需模块`'./'`相对于文件调用
`require()`.那是`circle.js`必须与`foo.js`为
`require('./circle')`找到它。

没有引线`'/'`,`'./'`或`'../'`要指示文件，模块必须
要么是核心模块，要么是从`node_modules`文件夹。

如果给定路径不存在，`require()`将抛出一个
[`MODULE_NOT_FOUND`][MODULE_NOT_FOUND]错误。

## 文件夹作为模块

<!--type=misc-->

> 稳定性： 3 - 旧版： 使用[子路径导出][subpath exports]或[子路径导入][subpath imports]相反。

有三种方法可以将文件夹传递给`require()`如
一个参数。

第一种是创建一个[`package.json`][package.json]文件位于文件夹的根目录中，
它指定了`main`模块。示例[`package.json`][package.json]文件可能
看起来像这样：

```json
{ "name" : "some-library",
  "main" : "./lib/some-library.js" }
```

如果它位于`./some-library`然后
`require('./some-library')`将尝试加载
`./some-library/lib/some-library.js`.

如果没有[`package.json`][package.json]文件存在于目录中，或者如果
[`"main"`]["main"]条目丢失或无法解析，然后是 Node.js
将尝试加载`index.js`或`index.node`文件出来
目录。例如，如果没有[`package.json`][package.json]文件
示例，然后`require('./some-library')`将尝试加载：

*   `./some-library/index.js`
*   `./some-library/index.node`

如果这些尝试失败，则 Node.js 会将整个模块报告为缺失
默认错误：

```console
Error: Cannot find module 'some-library'
```

在上述所有三种情况下，一个`import('./some-library')`调用将导致
[`ERR_UNSUPPORTED_DIR_IMPORT`][ERR_UNSUPPORTED_DIR_IMPORT]错误。使用包[子路径导出][subpath exports]或
[子路径导入][subpath imports]可以提供与
文件夹作为模块，并同时适用于两者`require`和`import`.

## 加载自`node_modules`文件夹

<!--type=misc-->

如果模块标识符传递给`require()`不是
[核心](#core-modules)模块，并且不以 开头`'/'`,`'../'`或
`'./'`，然后 Node.js从当前模块的目录开始，以及
增加`/node_modules`，并尝试从该位置加载模块。
节点.js不会追加`node_modules`到已结束于
`node_modules`.

如果在那里找不到它，那么它将移动到父目录，因此
，直到到达文件系统的根目录。

例如，如果文件位于`'/home/ry/projects/foo.js'`叫
`require('bar.js')`，然后 Node.js 将在以下位置查找，在
此订单：

*   `/home/ry/projects/node_modules/bar.js`
*   `/home/ry/node_modules/bar.js`
*   `/home/node_modules/bar.js`
*   `/node_modules/bar.js`

这允许程序本地化其依赖项，以便它们不会
冲突。

可以要求使用
模块，方法是在模块名称后包含路径后缀。例如
`require('example-module/path/to/file')`会解决`path/to/file`
相对于位置`example-module`位于。后缀路径遵循
相同的模块解析语义。

## 从全局文件夹加载

<!-- type=misc -->

如果`NODE_PATH`环境变量设置为冒号分隔的列表
的绝对路径，则 Node.js 将在这些路径中搜索模块（如果它们）
在其他地方找不到。

在 Windows 上，`NODE_PATH`由分号 （`;`）， 而不是冒号。

`NODE_PATH`最初创建是为了支持从
电流前的路径变化[模块分辨率][module resolution]定义了算法。

`NODE_PATH`仍然受支持，但现在 Node 的必要性降低.js
生态系统已经确定了定位依赖模块的约定。
有时，部署依赖于`NODE_PATH`表现出令人惊讶的行为
当人们没有意识到`NODE_PATH`必须设置。有时
模块的依赖关系发生变化，导致版本不同（甚至
不同的模块）作为`NODE_PATH`被搜索。

此外，Node.js 将在以下GLOBAL_FOLDERS列表中进行搜索：

*   1:`$HOME/.node_modules`
*   2:`$HOME/.node_libraries`
*   3:`$PREFIX/lib/node`

哪里`$HOME`是用户的主目录，并且`$PREFIX`是节点.js
配置`node_prefix`.

这些主要是出于历史原因。

强烈建议将依赖项放在本地`node_modules`
文件夹。这些将加载得更快，更可靠。

## 模块包装器

<!-- type=misc -->

在执行模块的代码之前，Node.js将使用函数包装它
包装器如下所示：

```js
(function(exports, require, module, __filename, __dirname) {
// Module code actually lives in here
});
```

通过这样做，Node.js实现了以下几点：

*   它保留顶级变量（定义为`var`,`const`或`let`） 的作用域为
    模块而不是全局对象。
*   它有助于提供一些实际特定的全局变量
    到模块，例如：
    *   这`module`和`exports`实现器可用于导出的对象
        模块中的值。
    *   便利变量`__filename`和`__dirname`，包含
        模块的绝对文件名和目录路径。

## 模块范围

### `__dirname`

<!-- YAML
added: v0.1.27
-->

<!-- type=var -->

*   {字符串}

当前模块的目录名称。这与
[`path.dirname()`][path.dirname()]的[`__filename`][__filename].

示例：运行`node example.js`从`/Users/mjr`

```js
console.log(__dirname);
// Prints: /Users/mjr
console.log(path.dirname(__filename));
// Prints: /Users/mjr
```

### `__filename`

<!-- YAML
added: v0.0.1
-->

<!-- type=var -->

*   {字符串}

当前模块的文件名。这是当前模块文件的绝对值
已解析符号链接的路径。

对于主程序，这不一定与
命令行。

看[`__dirname`][__dirname]作为当前模块的目录名称。

例子：

运行`node example.js`从`/Users/mjr`

```js
console.log(__filename);
// Prints: /Users/mjr/example.js
console.log(__dirname);
// Prints: /Users/mjr
```

给定两个模块：`a`和`b`哪里`b`是 的依赖关系
`a`并且有一个目录结构：

*   `/Users/mjr/app/a.js`
*   `/Users/mjr/app/node_modules/b/b.js`

参考资料`__filename`在`b.js`会再来
`/Users/mjr/app/node_modules/b/b.js`而引用`__filename`在
`a.js`会再来`/Users/mjr/app/a.js`.

### `exports`

<!-- YAML
added: v0.1.12
-->

<!-- type=var -->

*   {对象}

对`module.exports`键入时间较短。
请参阅有关[导出快捷方式][exports shortcut]有关何时使用的详细信息
`exports`以及何时使用`module.exports`.

### `module`

<!-- YAML
added: v0.1.16
-->

<!-- type=var -->

*   {模块}

对当前模块的引用，请参阅有关
[`module`对象][module object].特别`module.exports`用于定义什么
模块导出并通过以下方式提供`require()`.

### `require(id)`

<!-- YAML
added: v0.1.13
-->

<!-- type=var -->

*   `id`{字符串} 模块名称或路径
*   返回：{任意} 导出的模块内容

用于导入模块，`JSON`和本地文件。模块可以导入
从`node_modules`.可以使用以下命令导入本地模块和 JSON 文件
相对路径（例如`./`,`./foo`,`./bar/baz`,`../foo`）， 这将是
已针对命名的目录进行解析[`__dirname`][__dirname]（如果已定义）或
当前工作目录。解决了 POSIX 样式的相对路径
以独立于操作系统的方式，这意味着上面的示例将适用于
Windows就像在Unix系统上一样。

```js
// Importing a local module with a path relative to the `__dirname` or current
// working directory. (On Windows, this would resolve to .\path\myLocalModule.)
const myLocalModule = require('./path/myLocalModule');

// Importing a JSON file:
const jsonData = require('./path/filename.json');

// Importing a module from node_modules or Node.js built-in module:
const crypto = require('node:crypto');
```

#### `require.cache`

<!-- YAML
added: v0.3.0
-->

*   {对象}

模块在需要时缓存在此对象中。通过删除密钥
值从此对象， 下一个`require`将重新加载模块。
这不适用于[原生插件][native addons]，则重新加载将导致
错误。

也可以添加或替换条目。此缓存在之前已检查
内置模块，如果将与内置模块匹配的名称添加到缓存中，
只`node:`-前缀 require 调用将接收内置模块。
小心使用！

<!-- eslint-disable node-core/no-duplicate-requires -->

```js
const assert = require('node:assert');
const realFs = require('node:fs');

const fakeFs = {};
require.cache.fs = { exports: fakeFs };

assert.strictEqual(require('node:fs'), fakeFs);
assert.strictEqual(require('node:fs'), realFs);
```

#### `require.extensions`

<!-- YAML
added: v0.3.0
deprecated: v0.10.6
-->

> 稳定性：0 - 已弃用

*   {对象}

指导`require`关于如何处理某些文件扩展名。

处理扩展名为`.sjs`如`.js`:

```js
require.extensions['.sjs'] = require.extensions['.js'];
```

**荒废的。**在过去，此列表已用于加载非 JavaScript
模块到 Node 中.js通过按需编译它们。但是，在实践中，有
是更好的方法来做到这一点，例如通过其他一些Node加载模块.js
程序，或提前将它们编译为 JavaScript。

避免使用`require.extensions`.使用可能会导致细微的错误并解决
每个已注册的扩展的扩展速度会变慢。

#### `require.main`

<!-- YAML
added: v0.1.17
-->

*   {module | undefined}

这`Module`对象表示节点加载时加载的入口脚本.js
进程已启动，或`undefined`如果程序的入口点不是
通用JS模块。
看[“访问主模块”](#accessing-the-main-module).

在`entry.js`脚本：

```js
console.log(require.main);
```

```bash
node entry.js
```

<!-- eslint-skip -->

```js
Module {
  id: '.',
  path: '/absolute/path/to',
  exports: {},
  filename: '/absolute/path/to/entry.js',
  loaded: false,
  children: [],
  paths:
   [ '/absolute/path/to/node_modules',
     '/absolute/path/node_modules',
     '/absolute/node_modules',
     '/node_modules' ] }
```

#### `require.resolve(request[, options])`

<!-- YAML
added: v0.3.0
changes:
  - version: v8.9.0
    pr-url: https://github.com/nodejs/node/pull/16397
    description: The `paths` option is now supported.
-->

*   `request`{字符串}要解析的模块路径。
*   `options`{对象}
    *   `paths`{字符串\[]}从中解析模块位置的路径。如果存在，这些
        使用路径而不是默认分辨率路径，但
        之[GLOBAL_FOLDERS][GLOBAL_FOLDERS]喜欢`$HOME/.node_modules`，它们是
        总是包括在内。这些路径中的每条都用作以下各项的起点：
        模块分辨率算法，意味着`node_modules`等级制度
        从此位置进行检查。
*   返回：{字符串}

使用内部`require()`机械查找模块的位置，
而不是加载模块，只需返回解析的文件名。

如果找不到该模块，则`MODULE_NOT_FOUND`抛出错误。

##### `require.resolve.paths(request)`

<!-- YAML
added: v8.9.0
-->

*   `request`{字符串}正在检索其查找路径的模块路径。
*   返回： {string\[]|null}

返回一个数组，其中包含在解析期间搜索的路径`request`或
`null`如果`request`字符串引用核心模块，例如`http`或
`fs`.

## 这`module`对象

<!-- YAML
added: v0.1.16
-->

<!-- type=var -->

<!-- name=module -->

*   {对象}

在每个模块中，`module`free 变量是对对象的引用
表示当前模块。为方便起见，`module.exports`是
也可通过`exports`模块-全局。`module`实际上不是
每个模块的全局但本地。

### `module.children`

<!-- YAML
added: v0.1.16
-->

*   {模块\[]}

这是第一次需要的模块对象。

### `module.exports`

<!-- YAML
added: v0.1.16
-->

*   {对象}

这`module.exports`对象由`Module`系统。有时这是
不可接受;许多人希望他们的模块是某个类的实例。待办事项
这将，将所需的导出对象分配给`module.exports`.分配
所需的对象`exports`将简单地重新绑定本地`exports`变量
这可能不是我们想要的。

例如，假设我们正在制作一个名为`a.js`:

```js
const EventEmitter = require('node:events');

module.exports = new EventEmitter();

// Do some work, and after some time emit
// the 'ready' event from the module itself.
setTimeout(() => {
  module.exports.emit('ready');
}, 1000);
```

然后在另一个文件中，我们可以执行以下操作：

```js
const a = require('./a');
a.on('ready', () => {
  console.log('module "a" is ready');
});
```

分配给`module.exports`必须立即完成。它不可能是
在任何回调中完成。这不起作用：

`x.js`:

```js
setTimeout(() => {
  module.exports = { a: 'hello' };
}, 0);
```

`y.js`:

```js
const x = require('./x');
console.log(x.a);
```

#### `exports`捷径

<!-- YAML
added: v0.1.16
-->

这`exports`变量在模块的文件级范围内可用，并且
已分配值`module.exports`在评估模块之前。

它允许快捷方式，以便`module.exports.f = ...`可以写得更多
简明扼要地作为`exports.f = ...`.但是，请注意，与任何变量一样，如果
新值赋给`exports`，则不再绑定到`module.exports`:

```js
module.exports.hello = true; // Exported from require of module
exports = { hello: false };  // Not exported, only available in the module
```

当`module.exports`属性正在被新的
对象，通常也重新分配`exports`:

<!-- eslint-disable func-name-matching -->

```js
module.exports = exports = function Constructor() {
  // ... etc.
};
```

为了说明行为，想象一下这个假设的实现
`require()`，这与实际操作非常相似`require()`:

```js
function require(/* ... */) {
  const module = { exports: {} };
  ((module, exports) => {
    // Module code here. In this example, define a function.
    function someFunc() {}
    exports = someFunc;
    // At this point, exports is no longer a shortcut to module.exports, and
    // this module will still export an empty default object.
    module.exports = someFunc;
    // At this point, the module will now export someFunc, instead of the
    // default object.
  })(module, module.exports);
  return module.exports;
}
```

### `module.filename`

<!-- YAML
added: v0.1.16
-->

*   {字符串}

模块的完全解析文件名。

### `module.id`

<!-- YAML
added: v0.1.16
-->

*   {字符串}

模块的标识符。通常，这是完全解决的
文件名。

### `module.isPreloading`

<!-- YAML
added:
  - v15.4.0
  - v14.17.0
-->

*   类型： {布尔值}`true`如果模块在节点期间运行.js预加载
    阶段。

### `module.loaded`

<!-- YAML
added: v0.1.16
-->

*   {布尔值}

模块是否已完成加载，或者是否正在
装载。

### `module.parent`

<!-- YAML
added: v0.1.16
deprecated:
  - v14.6.0
  - v12.19.0
-->

> 稳定性：0 - 已弃用：请使用[`require.main`][require.main]和
> [`module.children`][module.children]相反。

*   {模块 | 空|未定义}

首先需要此模块的模块，或`null`如果当前模块是
当前进程的入口点，或`undefined`如果模块由
不是CommonJS模块的东西（例如：REPL或`import`).

### `module.path`

<!-- YAML
added: v11.14.0
-->

*   {字符串}

模块的目录名称。这通常与
[`path.dirname()`][path.dirname()]的[`module.id`][module.id].

### `module.paths`

<!-- YAML
added: v0.4.0
-->

*   {字符串\[]}

模块的搜索路径。

### `module.require(id)`

<!-- YAML
added: v0.5.1
-->

*   `id`{字符串}
*   返回：{任意} 导出的模块内容

这`module.require()`方法提供了一种加载模块的方法，就像
`require()`是从原始模块调用的。

为此，有必要获取对`module`对象。
因为`require()`返回`module.exports`，以及`module`通常
*只*在特定模块的代码中可用，必须显式导出
为了被使用。

## 这`Module`对象

此部分已移至
[模块：`module`核心模块](module.md#the-module-object).

<!-- Anchors to make sure old links find a target -->

*   <a id="modules_module_builtinmodules" href="module.html#modulebuiltinmodules">`module.builtinModules`</a>
*   <a id="modules_module_createrequire_filename" href="module.html#modulecreaterequirefilename">`module.createRequire(filename)`</a>
*   <a id="modules_module_syncbuiltinesmexports" href="module.html#modulesyncbuiltinesmexports">`module.syncBuiltinESMExports()`</a>

## 源映射 v3 支持

此部分已移至
[模块：`module`核心模块](module.md#source-map-v3-support).

<!-- Anchors to make sure old links find a target -->

*   <a id="modules_module_findsourcemap_path_error" href="module.html#modulefindsourcemappath">`module.findSourceMap(path)`</a>
*   <a id="modules_class_module_sourcemap" href="module.html#class-modulesourcemap">类：`module.SourceMap`</a>
    *   <a id="modules_new_sourcemap_payload" href="module.html#new-sourcemappayload">`new SourceMap(payload)`</a>
    *   <a id="modules_sourcemap_payload" href="module.html#sourcemappayload">`sourceMap.payload`</a>
    *   <a id="modules_sourcemap_findentry_linenumber_columnnumber" href="module.html#sourcemapfindentrylinenumber-columnnumber">`sourceMap.findEntry(lineNumber, columnNumber)`</a>

[Determining module system]: packages.md#determining-module-system

[ECMAScript Modules]: esm.md

[GLOBAL_FOLDERS]: #loading-from-the-global-folders

[`"main"`]: packages.md#main

[`"type"`]: packages.md#type

[`ERR_REQUIRE_ESM`]: errors.md#err_require_esm

[`ERR_UNSUPPORTED_DIR_IMPORT`]: errors.md#err_unsupported_dir_import

[`MODULE_NOT_FOUND`]: errors.md#module_not_found

[`__dirname`]: #__dirname

[`__filename`]: #__filename

[`import()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import

[`module.builtinModules`]: module.md#modulebuiltinmodules

[`module.children`]: #modulechildren

[`module.id`]: #moduleid

[`module` core module]: module.md

[`module` object]: #the-module-object

[`package.json`]: packages.md#nodejs-packagejson-field-definitions

[`path.dirname()`]: path.md#pathdirnamepath

[`require.main`]: #requiremain

[exports shortcut]: #exports-shortcut

[module resolution]: #all-together

[native addons]: addons.md

[subpath exports]: packages.md#subpath-exports

[subpath imports]: packages.md#subpath-imports
