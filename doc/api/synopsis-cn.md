# Usage and example

## Usage

<!--introduced_in=v0.10.0-->

<!--type=misc-->

`node [options] [V8 options] [script.js | -e "script" | - ] [arguments]`

请参阅 [命令行选项][] 文档了解更多信息.

## Example

一个用 Node.js 编写的 [web server][] 示例，它响应“Hello, World!”:

本文档中的命令以 `$` 或 `>` 开头，以复制它们在用户终端中的显示方式。不要包含 `$` 和 `>` 字符。他们在那里显示每个命令的开始.

不以 `$` 或 `>` 字符开头的行显示上一个命令的输出.

首先，确保已经下载并安装了 Node.js。有关更多安装信息，请参阅 [通过包管理器安装 Node.js][].

现在，创建一个名为“projects”的空项目文件夹，然后导航到它.

Linux and Mac:

```console
$ mkdir ~/projects
$ cd ~/projects
```

Windows CMD:

```console
> mkdir %USERPROFILE%\projects
> cd %USERPROFILE%\projects
```

Windows PowerShell:

```console
> mkdir $env:USERPROFILE\projects
> cd $env:USERPROFILE\projects
```

接下来，在 `projects` 文件夹中创建一个新的源文件并命名为 `hello-world.js`.

在任何首选的文本编辑器中打开 `hello-world.js` 并粘贴以下内容:

```js
const http = require('node:http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

保存文件，回到终端窗口，输入以下命令:

```console
$ node hello-world.js
```

像这样的输出应该出现在终端中:

```console
Server running at http://127.0.0.1:3000/
```

现在，打开任何首选的 Web 浏览器并访问 `http://127.0.0.1:3000`.

如果浏览器显示字符串 `Hello, World!`，则表明服务器正在工作.

[Command-line options]: cli.md#options
[Installing Node.js via package manager]: https://nodejs.org/en/download/package-manager/
[web server]: http.md
