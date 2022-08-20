# 路径

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/path.js -->

这`node:path`模块提供了用于处理文件和目录的实用程序
路径。可以使用以下命令访问它：

```js
const path = require('node:path');
```

## Windows vs. POSIX

的默认操作`node:path`模块因操作而异
系统，其中运行了 Node.js 应用程序。具体来说，当运行时
一个 Windows 操作系统，`node:path`模块将假定
正在使用窗口样式的路径。

所以使用`path.basename()`可能会在 POSIX 和 Windows 上产生不同的结果：

在 POSIX 上：

```js
path.basename('C:\\temp\\myfile.html');
// Returns: 'C:\\temp\\myfile.html'
```

在视窗上：

```js
path.basename('C:\\temp\\myfile.html');
// Returns: 'myfile.html'
```

在任何设备上使用 Windows 文件路径时获得一致的结果
操作系统， 使用[`path.win32`][path.win32]:

在 POSIX 和 Windows 上：

```js
path.win32.basename('C:\\temp\\myfile.html');
// Returns: 'myfile.html'
```

在任何设备上使用 POSIX 文件路径时获得一致的结果
操作系统， 使用[`path.posix`][path.posix]:

在 POSIX 和 Windows 上：

```js
path.posix.basename('/tmp/myfile.html');
// Returns: 'myfile.html'
```

在 Windows 节点上.js遵循每个驱动器工作目录的概念。
使用不带反斜杠的驱动器路径时，可以观察到此行为。为
例`path.resolve('C:\\')`可能返回与以下结果不同的结果
`path.resolve('C:')`.有关详细信息，请参阅
[此 MSDN 页面][MSDN-Rel-Path].

## `path.basename(path[, ext])`

<!-- YAML
added: v0.1.25
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

*   `path`{字符串}
*   `ext`{字符串}可选文件扩展名
*   返回：{字符串}

这`path.basename()`方法返回`path`似
Unix`basename`命令。尾随目录分隔符将被忽略，请参阅
[`path.sep`][path.sep].

```js
path.basename('/foo/bar/baz/asdf/quux.html');
// Returns: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// Returns: 'quux'
```

尽管 Windows 通常将文件名（包括文件扩展名）视为
不区分大小写的方式，此功能不。例如`C:\\foo.html`和
`C:\\foo.HTML`引用同一文件，但`basename`将扩展视为
区分大小写的字符串：

```js
path.win32.basename('C:\\foo.html', '.html');
// Returns: 'foo'

path.win32.basename('C:\\foo.HTML', '.html');
// Returns: 'foo.HTML'
```

一个[`TypeError`][TypeError]在以下情况下被抛出`path`不是字符串，或者如果`ext`被赋予
并且不是字符串。

## `path.delimiter`

<!-- YAML
added: v0.9.3
-->

*   {字符串}

提供特定于平台的路径分隔符：

*   `;`对于视窗
*   `:`对于 POSIX

例如，在 POSIX 上：

```js
console.log(process.env.PATH);
// Prints: '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

process.env.PATH.split(path.delimiter);
// Returns: ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
```

在视窗上：

```js
console.log(process.env.PATH);
// Prints: 'C:\Windows\system32;C:\Windows;C:\Program Files\node\'

process.env.PATH.split(path.delimiter);
// Returns ['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\node\\']
```

## `path.dirname(path)`

<!-- YAML
added: v0.1.16
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

*   `path`{字符串}
*   返回：{字符串}

这`path.dirname()`方法返回`path`似
Unix`dirname`命令。尾随目录分隔符将被忽略，请参阅
[`path.sep`][path.sep].

```js
path.dirname('/foo/bar/baz/asdf/quux');
// Returns: '/foo/bar/baz/asdf'
```

一个[`TypeError`][TypeError]在以下情况下被抛出`path`不是字符串。

## `path.extname(path)`

<!-- YAML
added: v0.1.25
changes:
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

*   `path`{字符串}
*   返回：{字符串}

这`path.extname()`方法返回`path`，从上一个
的出现次数`.`（句点）字符到字符串末尾的最后一部分
这`path`.如果没有`.`在的最后一部分`path`，或者如果
没有`.`除 第一个字符以外的字符
的基名`path`（请参见`path.basename()`） ，则返回一个空字符串。

```js
path.extname('index.html');
// Returns: '.html'

path.extname('index.coffee.md');
// Returns: '.md'

path.extname('index.');
// Returns: '.'

path.extname('index');
// Returns: ''

path.extname('.index');
// Returns: ''

path.extname('.index.md');
// Returns: '.md'
```

一个[`TypeError`][TypeError]在以下情况下被抛出`path`不是字符串。

## `path.format(pathObject)`

<!-- YAML
added: v0.11.15
-->

*   `pathObject`{对象}具有以下属性的任何 JavaScript 对象：
    *   `dir`{字符串}
    *   `root`{字符串}
    *   `base`{字符串}
    *   `name`{字符串}
    *   `ext`{字符串}
*   返回：{字符串}

这`path.format()`方法从对象返回路径字符串。这是
相反[`path.parse()`][path.parse()].

向`pathObject`请记住，有
一个属性优先于另一个属性的组合：

*   `pathObject.root`在以下情况下被忽略：`pathObject.dir`提供
*   `pathObject.ext`和`pathObject.name`在以下情况下被忽略：`pathObject.base`存在

例如，在 POSIX 上：

```js
// If `dir`, `root` and `base` are provided,
// `${dir}${path.sep}${base}`
// will be returned. `root` is ignored.
path.format({
  root: '/ignored',
  dir: '/home/user/dir',
  base: 'file.txt'
});
// Returns: '/home/user/dir/file.txt'

// `root` will be used if `dir` is not specified.
// If only `root` is provided or `dir` is equal to `root` then the
// platform separator will not be included. `ext` will be ignored.
path.format({
  root: '/',
  base: 'file.txt',
  ext: 'ignored'
});
// Returns: '/file.txt'

// `name` + `ext` will be used if `base` is not specified.
path.format({
  root: '/',
  name: 'file',
  ext: '.txt'
});
// Returns: '/file.txt'
```

在视窗上：

```js
path.format({
  dir: 'C:\\path\\dir',
  base: 'file.txt'
});
// Returns: 'C:\\path\\dir\\file.txt'
```

## `path.isAbsolute(path)`

<!-- YAML
added: v0.11.2
-->

*   `path`{字符串}
*   返回：{布尔值}

这`path.isAbsolute()`方法确定是否`path`是绝对路径。

如果给定`path`是一个长度为零的字符串，`false`将被退回。

例如，在 POSIX 上：

```js
path.isAbsolute('/foo/bar'); // true
path.isAbsolute('/baz/..');  // true
path.isAbsolute('qux/');     // false
path.isAbsolute('.');        // false
```

在视窗上：

```js
path.isAbsolute('//server');    // true
path.isAbsolute('\\\\server');  // true
path.isAbsolute('C:/foo/..');   // true
path.isAbsolute('C:\\foo\\..'); // true
path.isAbsolute('bar\\baz');    // false
path.isAbsolute('bar/baz');     // false
path.isAbsolute('.');           // false
```

一个[`TypeError`][TypeError]在以下情况下被抛出`path`不是字符串。

## `path.join([...paths])`

<!-- YAML
added: v0.1.16
-->

*   `...paths`{字符串}路径段序列
*   返回：{字符串}

这`path.join()`方法连接全部给定`path`使用
将特定于平台的分隔符作为分隔符，然后规范化生成的路径。

零长度`path`段将被忽略。如果联接的路径字符串是
零长度字符串，然后`'.'`将返回，表示当前
工作目录。

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// Returns: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar');
// Throws 'TypeError: Path must be a string. Received {}'
```

一个[`TypeError`][TypeError]如果任何路径段不是字符串，则抛出。

## `path.normalize(path)`

<!-- YAML
added: v0.1.23
-->

*   `path`{字符串}
*   返回：{字符串}

这`path.normalize()`方法规范化给定`path`解决`'..'`和
`'.'`段。

当找到多个顺序路径段分隔字符时（例如
`/`在 POSIX 上，并且`\`或`/`在Windows上），它们被替换为单个
特定于平台的路径段分隔符的实例 （`/`在 POSIX 上，并且
`\`在视窗上）。将保留尾随分隔符。

如果`path`是一个长度为零的字符串，`'.'`返回，表示
当前工作目录。

例如，在 POSIX 上：

```js
path.normalize('/foo/bar//baz/asdf/quux/..');
// Returns: '/foo/bar/baz/asdf'
```

在视窗上：

```js
path.normalize('C:\\temp\\\\foo\\bar\\..\\');
// Returns: 'C:\\temp\\foo\\'
```

由于 Windows 可识别多个路径分隔符，因此两个分隔符都将是
替换为 Windows 首选分隔符 （`\`):

```js
path.win32.normalize('C:////temp\\\\/\\/\\/foo/bar');
// Returns: 'C:\\temp\\foo\\bar'
```

一个[`TypeError`][TypeError]在以下情况下被抛出`path`不是字符串。

## `path.parse(path)`

<!-- YAML
added: v0.11.15
-->

*   `path`{字符串}
*   返回： {对象}

这`path.parse()`方法返回一个对象，其属性表示
的重要元素`path`.尾随目录分隔符将被忽略，
看[`path.sep`][path.sep].

返回的对象将具有以下属性：

*   `dir`{字符串}
*   `root`{字符串}
*   `base`{字符串}
*   `name`{字符串}
*   `ext`{字符串}

例如，在 POSIX 上：

```js
path.parse('/home/user/dir/file.txt');
// Returns:
// { root: '/',
//   dir: '/home/user/dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file' }
```

```text
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
"  /    home/user/dir / file  .txt "
└──────┴──────────────┴──────┴─────┘
(All spaces in the "" line should be ignored. They are purely for formatting.)
```

在视窗上：

```js
path.parse('C:\\path\\dir\\file.txt');
// Returns:
// { root: 'C:\\',
//   dir: 'C:\\path\\dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file' }
```

```text
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
" C:\      path\dir   \ file  .txt "
└──────┴──────────────┴──────┴─────┘
(All spaces in the "" line should be ignored. They are purely for formatting.)
```

一个[`TypeError`][TypeError]在以下情况下被抛出`path`不是字符串。

## `path.posix`

<!-- YAML
added: v0.11.15
changes:
  - version: v15.3.0
    pr-url: https://github.com/nodejs/node/pull/34962
    description: Exposed as `require('path/posix')`.
-->

*   {对象}

这`path.posix`属性提供对 POSIX 特定实现的访问
的`path`方法。

该 API 可通过以下方式访问`require('node:path').posix`或`require('node:path/posix')`.

## `path.relative(from, to)`

<!-- YAML
added: v0.5.0
changes:
  - version: v6.8.0
    pr-url: https://github.com/nodejs/node/pull/8523
    description: On Windows, the leading slashes for UNC paths are now included
                 in the return value.
-->

*   `from`{字符串}
*   `to`{字符串}
*   返回：{字符串}

这`path.relative()`方法返回相对路径`from`自`to`基于
在当前工作目录中。如果`from`和`to`每个解析为相同
路径（调用后`path.resolve()`），则返回一个长度为零的字符串。

如果将长度为零的字符串传递为`from`或`to`，当前工作
将使用目录代替长度为零的字符串。

例如，在 POSIX 上：

```js
path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb');
// Returns: '../../impl/bbb'
```

在视窗上：

```js
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb');
// Returns: '..\\..\\impl\\bbb'
```

一个[`TypeError`][TypeError]如果`from`或`to`不是字符串。

## `path.resolve([...paths])`

<!-- YAML
added: v0.3.4
-->

*   `...paths`{字符串}路径或路径段序列
*   返回：{字符串}

这`path.resolve()`方法将一系列路径或路径段解析为
绝对路径。

给定的路径序列从右到左处理，每个
随后的`path`在构造绝对路径之前一直处于前面。
例如，给定路径段的顺序：`/foo`,`/bar`,`baz`,
叫`path.resolve('/foo', '/bar', 'baz')`会再来的`/bar/baz`
因为`'baz'`不是绝对的路径，而是`'/bar' + '/' + 'baz'`是。

如果，处理完所有给出`path`段，绝对路径尚未
已生成，则使用当前工作目录。

将规范化生成的路径并删除尾部斜杠，除非
路径解析为根目录。

零长度`path`段将被忽略。

如果不是`path`段被传递，`path.resolve()`将返回绝对路径
的当前工作目录。

```js
path.resolve('/foo/bar', './baz');
// Returns: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/');
// Returns: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
// If the current working directory is /home/myself/node,
// this returns '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

一个[`TypeError`][TypeError]如果任何参数不是字符串，则抛出。

## `path.sep`

<!-- YAML
added: v0.7.9
-->

*   {字符串}

提供特定于平台的路径段分隔符：

*   `\`在视窗上
*   `/`在 POSIX 上

例如，在 POSIX 上：

```js
'foo/bar/baz'.split(path.sep);
// Returns: ['foo', 'bar', 'baz']
```

在视窗上：

```js
'foo\\bar\\baz'.split(path.sep);
// Returns: ['foo', 'bar', 'baz']
```

在 Windows 上，正斜杠 （`/`） 和反斜杠 （`\`） 被接受
作为路径段分隔符;但是，`path`方法仅向后添加
斜杠 （`\`).

## `path.toNamespacedPath(path)`

<!-- YAML
added: v9.0.0
-->

*   `path`{字符串}
*   返回：{字符串}

仅在 Windows 系统上，返回等效项[命名空间前缀路径][namespace-prefixed path]为
给定的`path`.如果`path`不是字符串，`path`将在没有
修改。

此方法仅在 Windows 系统上有意义。在 POSIX 系统上，
方法非操作且始终返回`path`无需修改。

## `path.win32`

<!-- YAML
added: v0.11.15
changes:
  - version: v15.3.0
    pr-url: https://github.com/nodejs/node/pull/34962
    description: Exposed as `require('path/win32')`.
-->

*   {对象}

这`path.win32`属性提供对特定于 Windows 的实现的访问
的`path`方法。

该 API 可通过以下方式访问`require('node:path').win32`或`require('node:path/win32')`.

[MSDN-Rel-Path]: https://docs.microsoft.com/en-us/windows/desktop/FileIO/naming-a-file#fully-qualified-vs-relative-paths

[`TypeError`]: errors.md#class-typeerror

[`path.parse()`]: #pathparsepath

[`path.posix`]: #pathposix

[`path.sep`]: #pathsep

[`path.win32`]: #pathwin32

[namespace-prefixed path]: https://docs.microsoft.com/en-us/windows/desktop/FileIO/naming-a-file#namespaces
