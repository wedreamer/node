# 调试器

<!--introduced_in=v0.9.12-->

> 稳定性：2 - 稳定

<!-- type=misc -->

Node.js包括一个命令行调试工具。Node.js的调试器客户端不是一个全功能的调试器，但可以进行简单的步进和检查。

要使用它，请在启动Node.js时使用`expect`参数，后面跟着要调试的脚本的路径。

```console
$ node inspect myscript.js
< Debugger listening on ws://127.0.0.1:9229/621111f9-ffcb-4e82-b718-48a145fa5db8
< For help, see: https://nodejs.org/en/docs/inspector
<
< Debugger attached.
<
 ok
Break on start in myscript.js:2
  1 // myscript.js
> 2 global.x = 5;
  3 setTimeout(() => {
  4   debugger;
debug>
```

调试器会在第一个可执行行自动中断。要代替运行到第一个断点（由`[debugger`]\[]语句指定），将`NODE_INSPECT_RESUME_ON_START`环境变量设置为`1`。

```console
$ cat myscript.js
// myscript.js
global.x = 5;
setTimeout(() => {
  debugger;
  console.log('world');
}, 1000);
console.log('hello');
$ NODE_INSPECT_RESUME_ON_START=1 node inspect myscript.js
< Debugger listening on ws://127.0.0.1:9229/f1ed133e-7876-495b-83ae-c32c6fc319c2
< For help, see: https://nodejs.org/en/docs/inspector
<
connecting to 127.0.0.1:9229 ... ok
< Debugger attached.
<
< hello
<
break in myscript.js:4
  2 global.x = 5;
  3 setTimeout(() => {
> 4   debugger;
  5   console.log('world');
  6 }, 1000);
debug> next
break in myscript.js:5
  3 setTimeout(() => {
  4   debugger;
> 5   console.log('world');
  6 }, 1000);
  7 console.log('hello');
debug> repl
Press Ctrl+C to leave debug repl
> x
5
> 2 + 2
4
debug> next
< world
<
break in myscript.js:6
  4   debugger;
  5   console.log('world');
> 6 }, 1000);
  7 console.log('hello');
  8
debug> .exit
$
```

`repl`命令允许对代码进行远程评估。`下一个`命令会步入下一行。键入`help`来查看还有哪些命令可用。

在没有输入命令的情况下按下`回车键`将重复前一个调试器命令。

## 监视者

在调试时可以观察表达式和变量的值。在每一个断点上，观察者列表中的每个表达式都会在当前上下文中被评估，并在断点的源代码列表前显示。

要开始观察一个表达式，输入`watch('my_expression')`。命令观察器将打印出活动的观察者。要删除一个监视者，请输入unwatch`('my_expression')`。

## 命令参考

### 步进

* `cont`,`c`:继续执行
* `next`,`n`:下一个步骤
* `step`,`s`:步入
* `out`,`o`:步出
* `暂停`。暂停正在运行的代码（就像开发工具中的暂停按钮）。

### 断点

* `setBreakpoint()`,`sb():`在当前行设置断点
* `setBreakpoint(line)`,`sb(line`):在特定的行上设置断点
* `setBreakpoint('fn()')`,`sb(...):`在函数正文的第一个语句上设置断点
* `setBreakpoint('script.js', 1)`,`sb`(...):在script`.js`的第一行设置断点
* `setBreakpoint('script.js', 1, 'num < 4')`, sb(...`)`。在`script`.js的第一行设置条件断点，只有当`num < 4evaluate`为`真时`才断掉。
* `clearBreakpoint('script.js', 1)`,`cb(...):`清除`script.json`第1行的断点

也可以在一个尚未加载的文件（模块）中设置断点。

```console
$ node inspect main.js
< Debugger listening on ws://127.0.0.1:9229/48a5b28a-550c-471b-b5e1-d13dd7165df9
< For help, see: https://nodejs.org/en/docs/inspector
<
< Debugger attached.
<
 ok
Break on start in main.js:1
> 1 const mod = require('./mod.js');
  2 mod.hello();
  3 mod.hello();
debug> setBreakpoint('mod.js', 22)
Warning: script 'mod.js' was not loaded yet.
debug> c
break in mod.js:22
 20 // USE OR OTHER DEALINGS IN THE SOFTWARE.
 21
>22 exports.hello = function() {
 23   return 'hello from module';
 24 };
debug>
```

也可以设置一个条件性断点，只在给定的表达式评估为`真时`才断。

```console
$ node inspect main.js
< Debugger listening on ws://127.0.0.1:9229/ce24daa8-3816-44d4-b8ab-8273c8a66d35
< For help, see: https://nodejs.org/en/docs/inspector
< Debugger attached.
Break on start in main.js:7
  5 }
  6
> 7 addOne(10);
  8 addOne(-1);
  9
debug> setBreakpoint('main.js', 4, 'num < 0')
  1 'use strict';
  2
  3 function addOne(num) {
> 4   return num + 1;
  5 }
  6
  7 addOne(10);
  8 addOne(-1);
  9
debug> cont
break in main.js:4
  2
  3 function addOne(num) {
> 4   return num + 1;
  5 }
  6
debug> exec('num')
-1
debug>
```

### 信息

* `backtrace`,`bt`:打印当前执行框架的回溯信息
* `list(5):`用5行上下文列出脚本的源代码（前后各5行）。
* `watch(expr)`。将表达式添加到监视列表中
* `unwatch(expr`):从观察列表中删除表达式
* `观察者`。列出所有观察者和它们的值（在每个断点上自动列出）。
* `repl`:打开调试器的rep，在调试脚本的上下文中进行评估
* `exec expr`,`p expr`:在调试脚本的上下文中执行一个表达式并打印其值

### 执行控制

* `运行`。运行脚本（在调试器启动时自动运行）。
* `重新启动`。重启脚本
* `kill`:杀死脚本

### 各种各样的

* `脚本`。列出所有加载的脚本
* `版本`。显示V8的版本

## 高级用法

### Node.js的V8检查器集成

V8 Inspector集成允许将Chrome DevTools附加到Node.jsinstances上，以便进行调试和分析。它使用\[Chrome DevTools协议]\[]。

在启动Node.js应用程序时，可以通过传递`--inspect`标志来启用V8 Inspector。也可以通过该标志提供一个自定义端口，例如`--inspect=9222`将接受端口 9222 的 DevTools 连接。

要在应用程序代码的第一行中断，请传递`--inspect-brkflag`而不是`--inspect`。

```console
$ node --inspect index.js
Debugger listening on ws://127.0.0.1:9229/dc9010dd-f8b8-4ac5-a510-c1a114ec7d29
For help, see: https://nodejs.org/en/docs/inspector
```

(在上面的例子中，URL末尾的UUID dc9010dd-f8b8-4ac5-a510-c1a114ec7d29是即时生成的，它在不同的调试会话中会有所不同）。

如果Chrome浏览器的版本超过66.0.3345.0，请使用`inspector.html`而不是上述URL中的`js_app.html`。

Chrome DevTools还不支持调试\[worker threads]\[]，可以用\[ndb]\[]来调试它们。

[Chrome DevTools Protocol]: https://chromedevtools.github.io/devtools-protocol/

[`debugger`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger

[ndb]: https://github.com/GoogleChromeLabs/ndb/

[worker threads]: worker_threads.md
