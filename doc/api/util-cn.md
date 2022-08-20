# 利用

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!-- source_link=lib/util.js -->

这`node:util`模块支持 Node.js内部 API 的需求。许多
实用程序对于应用程序和模块开发人员也很有用。访问
它：

```js
const util = require('node:util');
```

## `util.callbackify(original)`

<!-- YAML
added: v8.2.0
-->

*   `original`{函数}一`async`功能
*   返回：{函数} 回调样式函数

采取`async`函数（或返回`Promise`） 并返回
遵循错误优先回调样式的函数，即
一`(err, value) => ...`回调作为最后一个参数。在回调中，
第一个参数将是拒绝原因（或`null`如果`Promise`
已解析），第二个参数将是已解析的值。

```js
const util = require('node:util');

async function fn() {
  return 'hello world';
}
const callbackFunction = util.callbackify(fn);

callbackFunction((err, ret) => {
  if (err) throw err;
  console.log(ret);
});
```

将打印：

```text
hello world
```

回调是异步执行的，并且将具有有限的堆栈跟踪。
如果回调抛出，进程将发出一个[`'uncaughtException'`]['uncaughtException']
事件，如果不处理，将退出。

因为`null`具有作为回调的第一个参数的特殊含义，如果
包装的函数拒绝`Promise`以虚假值为理由，该值
被包裹在`Error`原始值存储在名为
`reason`.

```js
function fn() {
  return Promise.reject(null);
}
const callbackFunction = util.callbackify(fn);

callbackFunction((err, ret) => {
  // When the Promise was rejected with `null` it is wrapped with an Error and
  // the original value is stored in `reason`.
  err && Object.hasOwn(err, 'reason') && err.reason === null;  // true
});
```

## `util.debuglog(section[, callback])`

<!-- YAML
added: v0.11.3
-->

*   `section`{字符串}标识应用程序的部分的字符串
    其中`debuglog`正在创建函数。
*   `callback`{函数}第一次调用日志记录函数的回调
    使用函数参数调用，该参数是更优化的日志记录函数。
*   返回：{函数} 日志记录函数

这`util.debuglog()`方法用于创建有条件的函数
将调试消息写入`stderr`基于`NODE_DEBUG`
环境变量。如果`section`名称出现在该值内
环境变量，则返回的函数的操作类似于
[`console.error()`][console.error()].如果不是，则返回的函数为 no-op。

```js
const util = require('node:util');
const debuglog = util.debuglog('foo');

debuglog('hello from foo [%d]', 123);
```

如果此程序运行`NODE_DEBUG=foo`在环境中，然后
它将输出如下内容：

```console
FOO 3245: hello from foo [123]
```

哪里`3245`是进程 ID。如果它没有运行
设置环境变量，则不会打印任何内容。

这`section`还支持通配符：

```js
const util = require('node:util');
const debuglog = util.debuglog('foo-bar');

debuglog('hi there, it\'s foo-bar [%d]', 2333);
```

如果它与 一起运行`NODE_DEBUG=foo*`在环境中，然后它将输出
像这样：

```console
FOO-BAR 3257: hi there, it's foo-bar [2333]
```

多个逗号分隔`section`名称可以在`NODE_DEBUG`
环境变量：`NODE_DEBUG=fs,net,tls`.

可选`callback`参数可用于替换日志记录函数
使用没有任何初始化的不同函数，或者
不必要的包装。

```js
const util = require('node:util');
let debuglog = util.debuglog('internals', (debug) => {
  // Replace with a logging function that optimizes out
  // testing if the section is enabled
  debuglog = debug;
});
```

### `debuglog().enabled`

<!-- YAML
added: v14.9.0
-->

*   {布尔值}

这`util.debuglog().enabled`getter 用于创建可以使用的测试
在基于`NODE_DEBUG`环境变量。
如果`section`name 出现在该环境变量的值中，
则返回的值将为`true`.如果不是，则返回值将为
`false`.

```js
const util = require('node:util');
const enabled = util.debuglog('foo').enabled;
if (enabled) {
  console.log('hello from foo [%d]', 123);
}
```

如果此程序运行`NODE_DEBUG=foo`在环境中，那么它将
输出如下内容：

```console
hello from foo [123]
```

## `util.debug(section)`

<!-- YAML
added: v14.9.0
-->

的别名`util.debuglog`.使用允许可读性并不意味着
仅使用时记录`util.debuglog().enabled`.

## `util.deprecate(fn, msg[, code])`

<!-- YAML
added: v0.8.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16393
    description: Deprecation warnings are only emitted once for each code.
-->

*   `fn`{函数}正在弃用的函数。
*   `msg`{字符串}当已弃用的函数处于
    调用。
*   `code`{字符串}弃用代码。查看[已弃用的 API 列表][list of deprecated APIs]对于
    代码列表。
*   返回：{函数} 已弃用的函数包装以发出警告。

这`util.deprecate()`方法包装`fn`（可以是函数或类）
这样一种方式，它被标记为已弃用。

```js
const util = require('node:util');

exports.obsoleteFunction = util.deprecate(() => {
  // Do something here.
}, 'obsoleteFunction() is deprecated. Use newShinyFunction() instead.');
```

当被调用时，`util.deprecate()`将返回一个函数，该函数将发出
`DeprecationWarning`使用[`'warning'`]['warning']事件。警告将
发出并打印到`stderr`第一次返回的函数是
叫。发出警告后，调用包装的函数而不带
发出警告。

如果相同的可选`code`在多个调用中提供`util.deprecate()`,
警告只会发出一次`code`.

```js
const util = require('node:util');

const fn1 = util.deprecate(someFunction, someMessage, 'DEP0001');
const fn2 = util.deprecate(someOtherFunction, someOtherMessage, 'DEP0001');
fn1(); // Emits a deprecation warning with code DEP0001
fn2(); // Does not emit a deprecation warning because it has the same code
```

如果`--no-deprecation`或`--no-warnings`命令行标志是
二手的，或者如果`process.noDeprecation`属性设置为`true` *事先*自
第一个弃用警告，`util.deprecate()`方法不执行任何操作。

如果`--trace-deprecation`或`--trace-warnings`设置命令行标志，
或`process.traceDeprecation`属性设置为`true`、警告和
堆栈跟踪被打印到`stderr`第一次弃用的函数是
叫。

如果`--throw-deprecation`设置了命令行标志，或者
`process.throwDeprecation`属性设置为`true`，则异常将是
在调用已弃用的函数时抛出。

这`--throw-deprecation`命令行标志和`process.throwDeprecation`
属性优先于`--trace-deprecation`和
`process.traceDeprecation`.

## `util.format(format[, ...args])`

<!-- YAML
added: v0.5.3
changes:
  - version: v12.11.0
    pr-url: https://github.com/nodejs/node/pull/29606
    description: The `%c` specifier is ignored now.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23162
    description: The `format` argument is now only taken as such if it actually
                 contains format specifiers.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/23162
    description: If the `format` argument is not a format string, the output
                 string's formatting is no longer dependent on the type of the
                 first argument. This change removes previously present quotes
                 from strings that were being output when the first argument
                 was not a string.
  - version: v11.4.0
    pr-url: https://github.com/nodejs/node/pull/23708
    description: The `%d`, `%f`, and `%i` specifiers now support Symbols
                 properly.
  - version: v11.4.0
    pr-url: https://github.com/nodejs/node/pull/24806
    description: The `%o` specifier's `depth` has default depth of 4 again.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/17907
    description: The `%o` specifier's `depth` option will now fall back to the
                 default depth.
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/22097
    description: The `%d` and `%i` specifiers now support BigInt.
  - version: v8.4.0
    pr-url: https://github.com/nodejs/node/pull/14558
    description: The `%o` and `%O` specifiers are supported now.
-->

*   `format`{字符串}一个`printf`-like 格式字符串。

这`util.format()`方法使用第一个参数返回格式化字符串
作为`printf`-like format string，可以包含零个或多个格式
说明符。每个说明符都替换为
相应的参数。支持的说明符包括：

*   `%s`:`String`将用于转换除`BigInt`,`Object`
    和`-0`.`BigInt`值将用`n`和对象
    未定义用户`toString`使用以下命令检查函数`util.inspect()`
    与选项`{ depth: 0, colors: false, compact: 3 }`.
*   `%d`:`Number`将用于转换除`BigInt`和
    `Symbol`.
*   `%i`:`parseInt(value, 10)`用于除以下所有值之外的所有值`BigInt`和
    `Symbol`.
*   `%f`:`parseFloat(value)`用于预期的所有值`Symbol`.
*   `%j`：JSON。替换为字符串`'[Circular]'`如果参数包含
    循环引用。
*   `%o`:`Object`.具有通用 JavaScript 的对象的字符串表示形式
    对象格式设置。似`util.inspect()`与选项
    `{ showHidden: true, showProxy: true }`.这将显示完整的对象
    包括不可枚举的属性和代理。
*   `%O`:`Object`.具有通用 JavaScript 的对象的字符串表示形式
    对象格式设置。似`util.inspect()`没有选项。这将显示
    不包含不可枚举属性和代理的完整对象。
*   `%c`:`CSS`.此说明符将被忽略，并将跳过传入的任何 CSS。
*   `%%`：单百分号（`'%'`).这不会消耗参数。
*   返回：{字符串} 格式化字符串

如果说明符没有相应的参数，则不会替换它：

```js
util.format('%s:%s', 'foo');
// Returns: 'foo:%s'
```

不属于格式字符串的值的格式设置为
`util.inspect()`如果它们的类型不是`string`.

如果有更多参数传递给`util.format()`方法比
说明符的数量，多余的参数连接到返回的
字符串，以空格分隔：

```js
util.format('%s:%s', 'foo', 'bar', 'baz');
// Returns: 'foo:bar baz'
```

如果第一个参数不包含有效的格式说明符，`util.format()`
返回一个字符串，该字符串是用空格分隔的所有参数的串联：

```js
util.format(1, 2, 3);
// Returns: '1 2 3'
```

如果只有一个参数传递给`util.format()`，则按原样返回
没有任何格式：

```js
util.format('%% %s');
// Returns: '%% %s'
```

`util.format()`是一种同步方法，旨在用作调试工具。
某些输入值可能具有显著的性能开销，可能会阻止
事件循环。请谨慎使用此函数，切勿在热代码路径中使用。

## `util.formatWithOptions(inspectOptions, format[, ...args])`

<!-- YAML
added: v10.0.0
-->

*   `inspectOptions`{对象}
*   `format`{字符串}

此功能与[`util.format()`][util.format()]，除了它需要
一`inspectOptions`参数，该参数指定传递给
[`util.inspect()`][util.inspect()].

```js
util.formatWithOptions({ colors: true }, 'See object %O', { foo: 42 });
// Returns 'See object { foo: 42 }', where `42` is colored as a number
// when printed to a terminal.
```

## `util.getSystemErrorName(err)`

<!-- YAML
added: v9.7.0
-->

*   `err`{数字}
*   返回：{字符串}

返回来自 Node.js API 的数字错误代码的字符串名称。
错误代码和错误名称之间的映射取决于平台。
看[常见系统错误][Common System Errors]以获取常见错误的名称。

```js
fs.access('file/that/does/not/exist', (err) => {
  const name = util.getSystemErrorName(err.errno);
  console.error(name);  // ENOENT
});
```

## `util.getSystemErrorMap()`

<!-- YAML
added:
  - v16.0.0
  - v14.17.0
-->

*   返回： {地图}

返回 Node.js API 中提供的所有系统错误代码的 Map。
错误代码和错误名称之间的映射取决于平台。
看[常见系统错误][Common System Errors]以获取常见错误的名称。

```js
fs.access('file/that/does/not/exist', (err) => {
  const errorMap = util.getSystemErrorMap();
  const name = errorMap.get(err.errno);
  console.error(name);  // ENOENT
});
```

## `util.inherits(constructor, superConstructor)`

<!-- YAML
added: v0.3.0
changes:
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3455
    description: The `constructor` parameter can refer to an ES6 class now.
-->

> 稳定性：3 - 旧版：使用 ES2015 类语法和`extends`关键字。

*   `constructor`{函数}
*   `superConstructor`{函数}

用法`util.inherits()`不鼓励。请使用 ES6`class`和
`extends`关键字以获取语言级别的继承支持。另请注意
这两种样式是[语义上不兼容][semantically incompatible].

从一个方法继承原型方法[构造 函数][constructor]进入另一个。这
原型`constructor`将设置为从
`superConstructor`.

这主要在
`Object.setPrototypeOf(constructor.prototype, superConstructor.prototype)`.
为了提供额外的便利，`superConstructor`将可访问
通过`constructor.super_`财产。

```js
const util = require('node:util');
const EventEmitter = require('node:events');

function MyStream() {
  EventEmitter.call(this);
}

util.inherits(MyStream, EventEmitter);

MyStream.prototype.write = function(data) {
  this.emit('data', data);
};

const stream = new MyStream();

console.log(stream instanceof EventEmitter); // true
console.log(MyStream.super_ === EventEmitter); // true

stream.on('data', (data) => {
  console.log(`Received data: "${data}"`);
});
stream.write('It works!'); // Received data: "It works!"
```

ES6 示例使用`class`和`extends`:

```js
const EventEmitter = require('node:events');

class MyStream extends EventEmitter {
  write(data) {
    this.emit('data', data);
  }
}

const stream = new MyStream();

stream.on('data', (data) => {
  console.log(`Received data: "${data}"`);
});
stream.write('With ES6');
```

## `util.inspect(object[, options])`

## `util.inspect(object[, showHidden[, depth[, colors]]])`

<!-- YAML
added: v0.3.0
changes:
  - version: REPLACEME
    pr-url: https://github.com/nodejs/node/pull/43576
    description: add support for `maxArrayLength` when inspecting `Set` and `Map`.
  - version:
    - v17.3.0
    - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41003
    description: The `numericSeparator` option is supported now.
  - version:
    - v14.6.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/33690
    description: If `object` is from a different `vm.Context` now, a custom
                 inspection function on it will not receive context-specific
                 arguments anymore.
  - version:
     - v13.13.0
     - v12.17.0
    pr-url: https://github.com/nodejs/node/pull/32392
    description: The `maxStringLength` option is supported now.
  - version:
     - v13.5.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/30768
    description: User defined prototype properties are inspected in case
                 `showHidden` is `true`.
  - version: v13.0.0
    pr-url: https://github.com/nodejs/node/pull/27685
    description: Circular references now include a marker to the reference.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27109
    description: The `compact` options default is changed to `3` and the
                 `breakLength` options default is changed to `80`.
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/24971
    description: Internal properties no longer appear in the context argument
                 of a custom inspection function.
  - version: v11.11.0
    pr-url: https://github.com/nodejs/node/pull/26269
    description: The `compact` option accepts numbers for a new output mode.
  - version: v11.7.0
    pr-url: https://github.com/nodejs/node/pull/25006
    description: ArrayBuffers now also show their binary contents.
  - version: v11.5.0
    pr-url: https://github.com/nodejs/node/pull/24852
    description: The `getters` option is supported now.
  - version: v11.4.0
    pr-url: https://github.com/nodejs/node/pull/24326
    description: The `depth` default changed back to `2`.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22846
    description: The `depth` default changed to `20`.
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22756
    description: The inspection output is now limited to about 128 MiB. Data
                 above that size will not be fully inspected.
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/22788
    description: The `sorted` option is supported now.
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/20725
    description: Inspecting linked lists and similar objects is now possible
                 up to the maximum call stack size.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19259
    description: The `WeakMap` and `WeakSet` entries can now be inspected
                 as well.
  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/17576
    description: The `compact` option is supported now.
  - version: v6.6.0
    pr-url: https://github.com/nodejs/node/pull/8174
    description: Custom inspection functions can now return `this`.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/7499
    description: The `breakLength` option is supported now.
  - version: v6.1.0
    pr-url: https://github.com/nodejs/node/pull/6334
    description: The `maxArrayLength` option is supported now; in particular,
                 long arrays are truncated by default.
  - version: v6.1.0
    pr-url: https://github.com/nodejs/node/pull/6465
    description: The `showProxy` option is supported now.
-->

*   `object`{任何}任何 JavaScript 原语或`Object`.
*   `options`{对象}
    *   `showHidden`{布尔值}如果`true`,`object`的不可枚举符号和
        属性包含在格式化的结果中。[`WeakMap`][WeakMap]和
        [`WeakSet`][WeakSet]还包括条目以及用户定义的原型
        属性（不包括方法属性）。**违约：** `false`.
    *   `depth`{数字}指定格式化时要递归的次数
        `object`.这对于检查大型对象非常有用。递归到
        最大调用堆栈大小传递`Infinity`或`null`.
        **违约：** `2`.
    *   `colors`{布尔值}如果`true`，则输出使用 ANSI 颜色设置样式
        代码。颜色是可定制的。看[定制`util.inspect`颜色][Customizing util.inspect colors].
        **违约：** `false`.
    *   `customInspect`{布尔值}如果`false`,
        `[util.inspect.custom](depth, opts, inspect)`不调用函数。
        **违约：** `true`.
    *   `showProxy`{布尔值}如果`true`,`Proxy`检查包括
        这[`target`和`handler`][target and handler]对象。**违约：** `false`.
    *   `maxArrayLength`{整数}指定最大数量`Array`,
        [`TypedArray`][TypedArray],[`Map`][Map],[`Set`][Set],[`WeakMap`][WeakMap],
        和[`WeakSet`][WeakSet]元素， 以设置格式时要包含的元素。
        设置为`null`或`Infinity`以显示所有元素。设置为`0`或
        负数表示不显示任何元素。**违约：** `100`.
    *   `maxStringLength`{整数}指定最大字符数
        格式化时包含。设置为`null`或`Infinity`以显示所有元素。
        设置为`0`或负数表示不显示任何字符。**违约：** `10000`.
    *   `breakLength`{整数}输入值拆分的长度
        多行。设置为`Infinity`将输入格式设置为单行
        （结合`compact`设置为`true`或任何数字 >=`1`).
        **违约：** `80`.
    *   `compact`{布尔值|integer}将其设置为`false`导致每个对象键
        以显示在新行上。它将在文本中的新行上中断，该文本是
        长度超过`breakLength`.如果设置为数字，则最`n`内部元素
        只要所有属性都适合
        `breakLength`.短数组元素也组合在一起。更多
        信息，请参阅下面的示例。**违约：** `3`.
    *   `sorted`{布尔值|函数} 如果设置为`true`或函数，所有属性
        的对象，以及`Set`和`Map`条目在生成的
        字符串。如果设置为`true`这[默认排序][default sort]使用。如果设置为函数，
        它被用作[比较函数][compare function].
    *   `getters`{布尔值|字符串}如果设置为`true`，则检查 getter。如果设置
        自`'get'`，则仅检查没有相应二传手的采集器。如果
        设置为`'set'`，则仅检查具有相应二传手的 getter。
        这可能会导致副作用，具体取决于 getter 函数。
        **违约：** `false`.
    *   `numericSeparator`{布尔值}如果设置为`true`，下划线用于
        在所有大数字和数字中每三位数字分开一次。
        **违约：** `false`.
*   返回值：{字符串} 的表示形式`object`.

这`util.inspect()`方法返回字符串表示形式`object`那是
用于调试。输出`util.inspect`可能随时更改
并且不应以编程方式依赖。附加`options`可能
通过，改变结果。
`util.inspect()`将使用构造函数的名称和/或`@@toStringTag`制作
已检查值的可识别标记。

```js
class Foo {
  get [Symbol.toStringTag]() {
    return 'bar';
  }
}

class Bar {}

const baz = Object.create(null, { [Symbol.toStringTag]: { value: 'foo' } });

util.inspect(new Foo()); // 'Foo [bar] {}'
util.inspect(new Bar()); // 'Bar {}'
util.inspect(baz);       // '[foo] {}'
```

循环引用通过使用引用索引指向其锚点：

```js
const { inspect } = require('node:util');

const obj = {};
obj.a = [obj];
obj.b = {};
obj.b.inner = obj.b;
obj.b.obj = obj;

console.log(inspect(obj));
// <ref *1> {
//   a: [ [Circular *1] ],
//   b: <ref *2> { inner: [Circular *2], obj: [Circular *1] }
// }
```

下面的示例检查`util`对象：

```js
const util = require('node:util');

console.log(util.inspect(util, { showHidden: true, depth: null }));
```

下面的示例突出显示了`compact`选择：

```js
const util = require('node:util');

const o = {
  a: [1, 2, [[
    'Lorem ipsum dolor sit amet,\nconsectetur adipiscing elit, sed do ' +
      'eiusmod \ntempor incididunt ut labore et dolore magna aliqua.',
    'test',
    'foo']], 4],
  b: new Map([['za', 1], ['zb', 'test']])
};
console.log(util.inspect(o, { compact: true, depth: 5, breakLength: 80 }));

// { a:
//   [ 1,
//     2,
//     [ [ 'Lorem ipsum dolor sit amet,\nconsectetur [...]', // A long line
//           'test',
//           'foo' ] ],
//     4 ],
//   b: Map(2) { 'za' => 1, 'zb' => 'test' } }

// Setting `compact` to false or an integer creates more reader friendly output.
console.log(util.inspect(o, { compact: false, depth: 5, breakLength: 80 }));

// {
//   a: [
//     1,
//     2,
//     [
//       [
//         'Lorem ipsum dolor sit amet,\n' +
//           'consectetur adipiscing elit, sed do eiusmod \n' +
//           'tempor incididunt ut labore et dolore magna aliqua.',
//         'test',
//         'foo'
//       ]
//     ],
//     4
//   ],
//   b: Map(2) {
//     'za' => 1,
//     'zb' => 'test'
//   }
// }

// Setting `breakLength` to e.g. 150 will print the "Lorem ipsum" text in a
// single line.
```

这`showHidden`选项允许[`WeakMap`][WeakMap]和[`WeakSet`][WeakSet]条目
检查。如果条目数多于`maxArrayLength`，则没有
保证显示哪些条目。这意味着检索相同的内容
[`WeakSet`][WeakSet]条目两次可能会导致不同的输出。此外，条目
没有剩余的强引用可以随时被垃圾回收。

```js
const { inspect } = require('node:util');

const obj = { a: 1 };
const obj2 = { b: 2 };
const weakSet = new WeakSet([obj, obj2]);

console.log(inspect(weakSet, { showHidden: true }));
// WeakSet { { a: 1 }, { b: 2 } }
```

这`sorted`选项可确保对象的属性插入顺序不
影响结果`util.inspect()`.

```js
const { inspect } = require('node:util');
const assert = require('node:assert');

const o1 = {
  b: [2, 3, 1],
  a: '`a` comes before `b`',
  c: new Set([2, 3, 1])
};
console.log(inspect(o1, { sorted: true }));
// { a: '`a` comes before `b`', b: [ 2, 3, 1 ], c: Set(3) { 1, 2, 3 } }
console.log(inspect(o1, { sorted: (a, b) => b.localeCompare(a) }));
// { c: Set(3) { 3, 2, 1 }, b: [ 2, 3, 1 ], a: '`a` comes before `b`' }

const o2 = {
  c: new Set([2, 1, 3]),
  a: '`a` comes before `b`',
  b: [2, 3, 1]
};
assert.strict.equal(
  inspect(o1, { sorted: true }),
  inspect(o2, { sorted: true })
);
```

这`numericSeparator`选项将每三位数字下划线添加到所有
数字。

```js
const { inspect } = require('node:util');

const thousand = 1_000;
const million = 1_000_000;
const bigNumber = 123_456_789n;
const bigDecimal = 1_234.123_45;

console.log(thousand, million, bigNumber, bigDecimal);
// 1_000 1_000_000 123_456_789n 1_234.123_45
```

`util.inspect()`是用于调试的同步方法。其最大值
输出长度约为 128 MiB。导致更长输出的输入将
被截断。

### 定制`util.inspect`颜色

<!-- type=misc -->

颜色输出（如果启用）的`util.inspect`可全局定制
通过`util.inspect.styles`和`util.inspect.colors`性能。

`util.inspect.styles`是将样式名称与颜色关联的地图，从
`util.inspect.colors`.

默认样式和关联的颜色为：

*   `bigint`:`yellow`
*   `boolean`:`yellow`
*   `date`:`magenta`
*   `module`:`underline`
*   `name`：（无样式）
*   `null`:`bold`
*   `number`:`yellow`
*   `regexp`:`red`
*   `special`:`cyan`（例如，`Proxies`)
*   `string`:`green`
*   `symbol`:`green`
*   `undefined`:`grey`

颜色样式使用 ANSI 控制代码，并非所有颜色都受支持
终端。验证颜色支持使用[`tty.hasColors()`][tty.hasColors()].

下面列出了预定义的控制代码（分组为“修饰符”，“前景”
颜色“和”背景色”）。

#### 修饰 符

修改器支持因不同的终端而异。他们大多是
如果不支持，则忽略。

*   `reset`- 将所有（颜色）修饰符重置为其默认值
*   **大胆**- 使文本加粗
*   *斜体的*- 使文本斜体
*   <span style="border-bottom: 1px;">下划线</span>- 使文本带有下划线
*   \~~删除线~~ - 将一条水平线穿过文本的中心
    （别名：`strikeThrough`,`crossedout`,`crossedOut`)
*   `hidden`- 打印文本，但使其不可见（别名：隐藏）
*   <span style="opacity: 0.5;">䵨</span>- 颜色强度降低（别名：
    `faint`)
*   <span style="border-top: 1px">带上划线</span>- 使文本上划线
*   闪烁 - 在某个时间间隔内隐藏和显示文本
*   <span style="filter: invert(100%)">逆</span>- 交换前景和
    背景色（别名：`swapcolors`,`swapColors`)
*   <span style="border-bottom: 1px double;">双下线</span>- 制作文本
    双下划线（别名：`doubleUnderline`)
*   <span style="border: 1px">陷害</span>- 在文本周围绘制一个框架

#### 前景色

*   `black`
*   `red`
*   `green`
*   `yellow`
*   `blue`
*   `magenta`
*   `cyan`
*   `white`
*   `gray`（别名：`grey`,`blackBright`)
*   `redBright`
*   `greenBright`
*   `yellowBright`
*   `blueBright`
*   `magentaBright`
*   `cyanBright`
*   `whiteBright`

#### 背景颜色

*   `bgBlack`
*   `bgRed`
*   `bgGreen`
*   `bgYellow`
*   `bgBlue`
*   `bgMagenta`
*   `bgCyan`
*   `bgWhite`
*   `bgGray`（别名：`bgGrey`,`bgBlackBright`)
*   `bgRedBright`
*   `bgGreenBright`
*   `bgYellowBright`
*   `bgBlueBright`
*   `bgMagentaBright`
*   `bgCyanBright`
*   `bgWhiteBright`

### 对象的自定义检查功能

<!-- type=misc -->

<!-- YAML
added: v0.1.97
changes:
  - version:
      - v17.3.0
      - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41019
    description: The inspect argument is added for more interoperability.
-->

对象也可以定义自己的对象
[`[util.inspect.custom](depth, opts, inspect)`][util.inspect.custom]功能
哪`util.inspect()`将调用和使用检查时的结果
对象。

```js
const util = require('node:util');

class Box {
  constructor(value) {
    this.value = value;
  }

  [util.inspect.custom](depth, options, inspect) {
    if (depth < 0) {
      return options.stylize('[Box]', 'special');
    }

    const newOptions = Object.assign({}, options, {
      depth: options.depth === null ? null : options.depth - 1
    });

    // Five space padding because that's the size of "Box< ".
    const padding = ' '.repeat(5);
    const inner = inspect(this.value, newOptions)
                  .replace(/\n/g, `\n${padding}`);
    return `${options.stylize('Box', 'special')}< ${inner} >`;
  }
}

const box = new Box(true);

util.inspect(box);
// Returns: "Box< true >"
```

习惯`[util.inspect.custom](depth, opts, inspect)`函数通常返回
一个字符串，但可以返回任何类型的值，该值将相应地设置格式
由`util.inspect()`.

```js
const util = require('node:util');

const obj = { foo: 'this will not show up in the inspect() output' };
obj[util.inspect.custom] = (depth) => {
  return { bar: 'baz' };
};

util.inspect(obj);
// Returns: "{ bar: 'baz' }"
```

### `util.inspect.custom`

<!-- YAML
added: v6.6.0
changes:
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/20857
    description: This is now defined as a shared symbol.
-->

*   {符号}，可用于声明自定义检查函数。

除了通过以下方式访问`util.inspect.custom`这
符号是[全球注册][global symbol registry]并且可以是
在任何环境中访问为`Symbol.for('nodejs.util.inspect.custom')`.

使用这个允许以可移植的方式编写代码，以便自定义
检查函数在 Node.js环境中使用，在浏览器中被忽略。
这`util.inspect()`函数本身作为第三个参数传递给自定义
检查功能以允许进一步的可移植性。

```js
const customInspectSymbol = Symbol.for('nodejs.util.inspect.custom');

class Password {
  constructor(value) {
    this.value = value;
  }

  toString() {
    return 'xxxxxxxx';
  }

  [customInspectSymbol](depth, inspectOptions, inspect) {
    return `Password <${this.toString()}>`;
  }
}

const password = new Password('r0sebud');
console.log(password);
// Prints Password <xxxxxxxx>
```

看[对象上的自定义检查函数][Custom inspection functions on Objects]了解更多详情。

### `util.inspect.defaultOptions`

<!-- YAML
added: v6.4.0
-->

这`defaultOptions`值允许自定义
`util.inspect`.这对于以下功能很有用：`console.log`或
`util.format`它隐式调用`util.inspect`.它应设置为
包含一个或多个有效值的对象[`util.inspect()`][util.inspect()]选项。设置
还直接支持选项属性。

```js
const util = require('node:util');
const arr = Array(101).fill(0);

console.log(arr); // Logs the truncated array
util.inspect.defaultOptions.maxArrayLength = null;
console.log(arr); // logs the full array
```

## `util.isDeepStrictEqual(val1, val2)`

<!-- YAML
added: v9.0.0
-->

*   `val1`{任何}
*   `val2`{任何}
*   返回：{布尔值}

返回`true`如果两者之间有深度的严格平等`val1`和`val2`.
否则，返回`false`.

看[`assert.deepStrictEqual()`][assert.deepStrictEqual()]有关深度严格的更多信息
平等。

## `util.parseArgs([config])`

<!-- YAML
added: v18.3.0
changes:
  - version: v18.7.0
    pr-url: https://github.com/nodejs/node/pull/43459
    description: add support for returning detailed parse information
                 using `tokens` in input `config` and returned properties.
-->

> 稳定性： 1 - 实验

*   `config`{对象}用于提供用于解析的参数和配置
    解析器。`config`支持以下属性：
    *   `args`{字符串\[]} 参数字符串的数组。**违约：** `process.argv`
        跟`execPath`和`filename`删除。
    *   `options`{对象}用于描述解析器已知的参数。
        的键`options`是选项的长名称，值是
        {对象} 接受以下属性：
        *   `type`{字符串}参数的类型，该类型必须是`boolean`或`string`.
        *   `multiple`{布尔值}此选项是否可以提供多个
            次。如果`true`，则所有值都将收集在一个数组中。如果
            `false`，则该选项的值是最后一次获胜。**违约：** `false`.
        *   `short`{字符串}选项的单个字符别名。
    *   `strict`{布尔值}当未知参数时是否应引发错误
        ，或者当传递的参数与
        `type`配置于`options`.
        **违约：** `true`.
    *   `allowPositionals`{布尔值}此命令是否接受位置
        参数。
        **违约：** `false`如果`strict`是`true`否则`true`.
    *   `tokens`{布尔值}返回已分析的令牌。这对于扩展
        内置行为，从添加其他检查到重新处理
        以不同的方式标记。
        **违约：** `false`.

*   返回：{对象} 解析的命令行参数：
    *   `values`{对象}已分析选项名称及其 {string} 的映射
        或 {布尔} 值。
    *   `positionals`{字符串\[]}位置参数。
    *   `tokens`{Object\[] | undefined}看[parseArgs tokens](#parseargs-tokens)
        部分。仅当以下情况返回时`config`包括`tokens: true`.

为命令行参数解析提供比交互更高级别的 API
跟`process.argv`径直。采用预期参数的规范
并返回一个结构化对象，其中包含已解析的选项和位置。

```mjs
import { parseArgs } from 'node:util';
const args = ['-f', '--bar', 'b'];
const options = {
  foo: {
    type: 'boolean',
    short: 'f'
  },
  bar: {
    type: 'string'
  }
};
const {
  values,
  positionals
} = parseArgs({ args, options });
console.log(values, positionals);
// Prints: [Object: null prototype] { foo: true, bar: 'b' } []
```

```cjs
const { parseArgs } = require('node:util');
const args = ['-f', '--bar', 'b'];
const options = {
  foo: {
    type: 'boolean',
    short: 'f'
  },
  bar: {
    type: 'string'
  }
};
const {
  values,
  positionals
} = parseArgs({ args, options });
console.log(values, positionals);
// Prints: [Object: null prototype] { foo: true, bar: 'b' } []
```

`util.parseArgs`是实验性的，行为可能会改变。加入
对话[pkgjs/parseargs][]为设计做出贡献。

### `parseArgs` `tokens`

详细的解析信息可用于通过以下方式添加自定义行为
指定`tokens: true`在配置中。
返回的令牌具有描述以下内容的属性：

*   所有代币
    *   `kind`{字符串}“选项”、“位置”或“选项终止符”之一。
    *   `index`{数字}元素索引`args`包含令牌。所以
        令牌的源参数为`args[token.index]`.
*   期权代币
    *   `name`{字符串}选项的长名称。
    *   `rawName`{字符串}如何在参数中使用选项，例如`-f`之`--foo`.
    *   `value`{字符串 | 未定义}在参数中指定的选项值。
        未定义布尔选项。
    *   `inlineValue`{布尔|未定义}是否以内联方式指定选项值，
        喜欢`--foo=bar`.
*   位置令牌
    *   `value`{字符串}args 中位置参数的值（即`args[index]`).
*   选项终止符令牌

返回的标记按输入参数中遇到的顺序排列。选项
在 args 中多次出现，每次使用都会生成一个令牌。短选项
组如`-xy`展开为每个选项的令牌。所以`-xxx`生产
三个令牌。

例如，使用返回的标记添加对否定选项的支持
喜欢`--no-color`，可以重新处理令牌以更改存储的值
对于否定选项。

```mjs
import { parseArgs } from 'node:util';

const options = {
  'color': { type: 'boolean' },
  'no-color': { type: 'boolean' },
  'logfile': { type: 'string' },
  'no-logfile': { type: 'boolean' },
};
const { values, tokens } = parseArgs({ options, tokens: true });

// Reprocess the option tokens and overwrite the returned values.
tokens
  .filter((token) => token.kind === 'option')
  .forEach((token) => {
    if (token.name.startsWith('no-')) {
      // Store foo:false for --no-foo
      const positiveName = token.name.slice(3);
      values[positiveName] = false;
      delete values[token.name];
    } else {
      // Resave value so last one wins if both --foo and --no-foo.
      values[token.name] = token.value ?? true;
    }
  });

const color = values.color;
const logfile = values.logfile ?? 'default.log';

console.log({ logfile, color });
```

```cjs
const { parseArgs } = require('node:util');

const options = {
  'color': { type: 'boolean' },
  'no-color': { type: 'boolean' },
  'logfile': { type: 'string' },
  'no-logfile': { type: 'boolean' },
};
const { values, tokens } = parseArgs({ options, tokens: true });

// Reprocess the option tokens and overwrite the returned values.
tokens
  .filter((token) => token.kind === 'option')
  .forEach((token) => {
    if (token.name.startsWith('no-')) {
      // Store foo:false for --no-foo
      const positiveName = token.name.slice(3);
      values[positiveName] = false;
      delete values[token.name];
    } else {
      // Resave value so last one wins if both --foo and --no-foo.
      values[token.name] = token.value ?? true;
    }
  });

const color = values.color;
const logfile = values.logfile ?? 'default.log';

console.log({ logfile, color });
```

显示否定选项以及何时使用选项的示例用法
多种方式，然后最后一个获胜。

```console
$ node negate.js
{ logfile: 'default.log', color: undefined }
$ node negate.js --no-logfile --no-color
{ logfile: false, color: false }
$ node negate.js --logfile=test.log --color
{ logfile: 'test.log', color: true }
$ node negate.js --no-logfile --logfile=test.log --color --no-color
{ logfile: 'test.log', color: false }
```

## `util.promisify(original)`

<!-- YAML
added: v8.0.0
-->

*   `original`{函数}
*   返回：{函数}

采用遵循常见错误优先回调样式的函数，即
一`(err, value) => ...`回调作为最后一个参数，并返回一个版本
返回承诺。

```js
const util = require('node:util');
const fs = require('node:fs');

const stat = util.promisify(fs.stat);
stat('.').then((stats) => {
  // Do something with `stats`
}).catch((error) => {
  // Handle the error.
});
```

或者，等效地使用`async function`s:

```js
const util = require('node:util');
const fs = require('node:fs');

const stat = util.promisify(fs.stat);

async function callStat() {
  const stats = await stat('.');
  console.log(`This directory is owned by ${stats.uid}`);
}
```

如果有`original[util.promisify.custom]`存在属性，`promisify`
将返回其值，请参阅[自定义过期函数][Custom promisified functions].

`promisify()`假设`original`是一个将回调作为其
所有情况下的最终参数。如果`original`不是函数，`promisify()`
将引发错误。如果`original`是一个函数，但其最后一个参数不是
一个错误优先的回调，它仍然会传递一个错误优先
回调作为其最后一个参数。

用`promisify()`在类方法或其他使用`this`可能不是
除非经过特殊处理，否则按预期工作：

```js
const util = require('node:util');

class Foo {
  constructor() {
    this.a = 42;
  }

  bar(callback) {
    callback(null, this.a);
  }
}

const foo = new Foo();

const naiveBar = util.promisify(foo.bar);
// TypeError: Cannot read property 'a' of undefined
// naiveBar().then(a => console.log(a));

naiveBar.call(foo).then((a) => console.log(a)); // '42'

const bindBar = naiveBar.bind(foo);
bindBar().then((a) => console.log(a)); // '42'
```

### 自定义过期函数

使用`util.promisify.custom`符号 1 可以覆盖返回值
[`util.promisify()`][util.promisify()]:

```js
const util = require('node:util');

function doSomething(foo, callback) {
  // ...
}

doSomething[util.promisify.custom] = (foo) => {
  return getPromiseSomehow();
};

const promisified = util.promisify(doSomething);
console.log(promisified === doSomething[util.promisify.custom]);
// prints 'true'
```

这对于原始函数不遵循
将错误优先回调作为最后一个参数的标准格式。

例如，使用接受的函数
`(foo, onSuccessCallback, onErrorCallback)`:

```js
doSomething[util.promisify.custom] = (foo) => {
  return new Promise((resolve, reject) => {
    doSomething(foo, resolve, reject);
  });
};
```

如果`promisify.custom`已定义但不是函数，`promisify()`将
引发错误。

### `util.promisify.custom`

<!-- YAML
added: v8.0.0
changes:
  - version:
      - v13.12.0
      - v12.16.2
    pr-url: https://github.com/nodejs/node/pull/31672
    description: This is now defined as a shared symbol.
-->

*   {symbol}，可用于声明函数的自定义变体，
    看[自定义过期函数][Custom promisified functions].

除了通过以下方式访问`util.promisify.custom`这
符号是[全球注册][global symbol registry]并且可以是
在任何环境中访问为`Symbol.for('nodejs.util.promisify.custom')`.

例如，使用接受的函数
`(foo, onSuccessCallback, onErrorCallback)`:

```js
const kCustomPromisifiedSymbol = Symbol.for('nodejs.util.promisify.custom');

doSomething[kCustomPromisifiedSymbol] = (foo) => {
  return new Promise((resolve, reject) => {
    doSomething(foo, resolve, reject);
  });
};
```

## `util.stripVTControlCharacters(str)`

<!-- YAML
added: v16.11.0
-->

*   `str`{字符串}
*   返回：{字符串}

返回`str`删除任何 ANSI 转义代码。

```js
console.log(util.stripVTControlCharacters('\u001B[4mvalue\u001B[0m'));
// Prints "value"
```

## 类：`util.TextDecoder`

<!-- YAML
added: v8.3.0
-->

实现[WHATWG 编码标准][WHATWG Encoding Standard] `TextDecoder`应用程序接口。

```js
const decoder = new TextDecoder();
const u8arr = new Uint8Array([72, 101, 108, 108, 111]);
console.log(decoder.decode(u8arr)); // Hello
```

### WHATWG 支持的编码

根据[WHATWG 编码标准][WHATWG Encoding Standard]，则支持的编码
`TextDecoder`下表概述了 API。对于每种编码，
可以使用一个或多个别名。

不同的 Node.js 构建配置支持不同的编码集。
（请参见[国际化][Internationalization])

#### 默认支持的编码（包含完整的 ICU 数据）

|编码|别名 |
|------------------ |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|`'ibm866'`|`'866'`,`'cp866'`,`'csibm866'`|
|`'iso-8859-2'`|`'csisolatin2'`,`'iso-ir-101'`,`'iso8859-2'`,`'iso88592'`,`'iso_8859-2'`,`'iso_8859-2:1987'`,`'l2'`,`'latin2'`|
|`'iso-8859-3'`|`'csisolatin3'`,`'iso-ir-109'`,`'iso8859-3'`,`'iso88593'`,`'iso_8859-3'`,`'iso_8859-3:1988'`,`'l3'`,`'latin3'`|
|`'iso-8859-4'`|`'csisolatin4'`,`'iso-ir-110'`,`'iso8859-4'`,`'iso88594'`,`'iso_8859-4'`,`'iso_8859-4:1988'`,`'l4'`,`'latin4'`|
|`'iso-8859-5'`|`'csisolatincyrillic'`,`'cyrillic'`,`'iso-ir-144'`,`'iso8859-5'`,`'iso88595'`,`'iso_8859-5'`,`'iso_8859-5:1988'`|
|`'iso-8859-6'`|`'arabic'`,`'asmo-708'`,`'csiso88596e'`,`'csiso88596i'`,`'csisolatinarabic'`,`'ecma-114'`,`'iso-8859-6-e'`,`'iso-8859-6-i'`,`'iso-ir-127'`,`'iso8859-6'`,`'iso88596'`,`'iso_8859-6'`,`'iso_8859-6:1987'`|
|`'iso-8859-7'`|`'csisolatingreek'`,`'ecma-118'`,`'elot_928'`,`'greek'`,`'greek8'`,`'iso-ir-126'`,`'iso8859-7'`,`'iso88597'`,`'iso_8859-7'`,`'iso_8859-7:1987'`,`'sun_eu_greek'`|
|`'iso-8859-8'`|`'csiso88598e'`,`'csisolatinhebrew'`,`'hebrew'`,`'iso-8859-8-e'`,`'iso-ir-138'`,`'iso8859-8'`,`'iso88598'`,`'iso_8859-8'`,`'iso_8859-8:1988'`,`'visual'`|
|`'iso-8859-8-i'`|`'csiso88598i'`,`'logical'`|
|`'iso-8859-10'`|`'csisolatin6'`,`'iso-ir-157'`,`'iso8859-10'`,`'iso885910'`,`'l6'`,`'latin6'`|
|`'iso-8859-13'`|`'iso8859-13'`,`'iso885913'`|
|`'iso-8859-14'`|`'iso8859-14'`,`'iso885914'`|
|`'iso-8859-15'`|`'csisolatin9'`,`'iso8859-15'`,`'iso885915'`,`'iso_8859-15'`,`'l9'`|
|`'koi8-r'`|`'cskoi8r'`,`'koi'`,`'koi8'`,`'koi8_r'`|
|`'koi8-u'`|`'koi8-ru'`|
|`'macintosh'`|`'csmacintosh'`,`'mac'`,`'x-mac-roman'`|
|`'windows-874'`|`'dos-874'`,`'iso-8859-11'`,`'iso8859-11'`,`'iso885911'`,`'tis-620'`|
|`'windows-1250'`|`'cp1250'`,`'x-cp1250'`|
|`'windows-1251'`|`'cp1251'`,`'x-cp1251'`|
|`'windows-1252'`|`'ansi_x3.4-1968'`,`'ascii'`,`'cp1252'`,`'cp819'`,`'csisolatin1'`,`'ibm819'`,`'iso-8859-1'`,`'iso-ir-100'`,`'iso8859-1'`,`'iso88591'`,`'iso_8859-1'`,`'iso_8859-1:1987'`,`'l1'`,`'latin1'`,`'us-ascii'`,`'x-cp1252'`|
|`'windows-1253'`|`'cp1253'`,`'x-cp1253'`|
|`'windows-1254'`|`'cp1254'`,`'csisolatin5'`,`'iso-8859-9'`,`'iso-ir-148'`,`'iso8859-9'`,`'iso88599'`,`'iso_8859-9'`,`'iso_8859-9:1989'`,`'l5'`,`'latin5'`,`'x-cp1254'`|
|`'windows-1255'`|`'cp1255'`,`'x-cp1255'`|
|`'windows-1256'`|`'cp1256'`,`'x-cp1256'`|
|`'windows-1257'`|`'cp1257'`,`'x-cp1257'`|
|`'windows-1258'`|`'cp1258'`,`'x-cp1258'`|
|`'x-mac-cyrillic'`|`'x-mac-ukrainian'`|
|`'gbk'`|`'chinese'`,`'csgb2312'`,`'csiso58gb231280'`,`'gb2312'`,`'gb_2312'`,`'gb_2312-80'`,`'iso-ir-58'`,`'x-gbk'`|
|`'gb18030'`|                                                                                                                                                                                                                                    |
|`'big5'`|`'big5-hkscs'`,`'cn-big5'`,`'csbig5'`,`'x-x-big5'`|
|`'euc-jp'`|`'cseucpkdfmtjapanese'`,`'x-euc-jp'`|
|`'iso-2022-jp'`|`'csiso2022jp'`|
|`'shift_jis'`|`'csshiftjis'`,`'ms932'`,`'ms_kanji'`,`'shift-jis'`,`'sjis'`,`'windows-31j'`,`'x-sjis'`|
|`'euc-kr'`|`'cseuckr'`,`'csksc56011987'`,`'iso-ir-149'`,`'korean'`,`'ks_c_5601-1987'`,`'ks_c_5601-1989'`,`'ksc5601'`,`'ksc_5601'`,`'windows-949'`|

#### 当 Node.js 是使用`small-icu`选择

|编码|别名 |
|------------ |------------------------------- |
|`'utf-8'`|`'unicode-1-1-utf-8'`,`'utf8'`|
|`'utf-16le'`|`'utf-16'`|
|`'utf-16be'`|                                |

#### 禁用 ICU 时支持的编码

|编码|别名 |
|------------ |------------------------------- |
|`'utf-8'`|`'unicode-1-1-utf-8'`,`'utf8'`|
|`'utf-16le'`|`'utf-16'`|

这`'iso-8859-16'`编码列在[WHATWG 编码标准][WHATWG Encoding Standard]
不受支持。

### `new TextDecoder([encoding[, options]])`

<!-- YAML
added: v8.3.0
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22281
    description: The class is now available on the global object.
-->

*   `encoding`{字符串}标识`encoding`那个`TextDecoder`实例
    支持。**违约：** `'utf-8'`.
*   `options`{对象}
    *   `fatal`{布尔值}`true`如果解码失败是致命的。
        禁用 ICU 时不支持此选项
        （请参见[国际化][Internationalization]).**违约：** `false`.
    *   `ignoreBOM`{布尔值}什么时候`true`这`TextDecoder`将包括字节
        解码结果中的订单标记。什么时候`false`，字节顺序标记将
        从输出中删除。此选项仅在以下情况下使用：`encoding`是
        `'utf-8'`,`'utf-16be'`或`'utf-16le'`.**违约：** `false`.

创建新的`TextDecoder`实例。这`encoding`可以指定
支持的编码或别名。

这`TextDecoder`类在全局对象上也可用。

### `textDecoder.decode([input[, options]])`

*   `input`{ArrayBuffer|数据视图|TypedArray} An`ArrayBuffer`,`DataView`或
    `TypedArray`包含编码数据的实例。
*   `options`{对象}
    *   `stream`{布尔值}`true`如果需要额外的数据块。
        **违约：** `false`.
*   返回：{字符串}

解码`input`并返回一个字符串。如果`options.stream`是`true`任何
在`input`已缓冲
在内部并在下次调用后发出`textDecoder.decode()`.

如果`textDecoder.fatal`是`true`，则发生的解码错误将导致
`TypeError`被抛出。

### `textDecoder.encoding`

*   {字符串}

支持的编码`TextDecoder`实例。

### `textDecoder.fatal`

*   {布尔值}

该值将为`true`如果解码错误导致`TypeError`存在
扔。

### `textDecoder.ignoreBOM`

*   {布尔值}

该值将为`true`如果解码结果将包括字节顺序
马克。

## 类：`util.TextEncoder`

<!-- YAML
added: v8.3.0
changes:
  - version: v11.0.0
    pr-url: https://github.com/nodejs/node/pull/22281
    description: The class is now available on the global object.
-->

实现[WHATWG 编码标准][WHATWG Encoding Standard] `TextEncoder`应用程序接口。都
的实例`TextEncoder`仅支持 UTF-8 编码。

```js
const encoder = new TextEncoder();
const uint8array = encoder.encode('this is some data');
```

这`TextEncoder`类在全局对象上也可用。

### `textEncoder.encode([input])`

*   `input`{字符串}要编码的文本。**违约：**空字符串。
*   返回： {Uint8Array}

UTF-8 对`input`字符串并返回`Uint8Array`包含
编码字节。

### `textEncoder.encodeInto(src, dest)`

*   `src`{字符串}要编码的文本。
*   `dest`{Uint8Array}用于保存编码结果的数组。
*   返回： {对象}
    *   `read`{数字}src 的读取 Unicode 代码单元。
    *   `written`{数字}写入的 DEST 的 UTF-8 字节。

UTF-8 对`src`字符串到`dest`Uint8数组并返回一个对象
包含读取的 Unicode 代码单元和写入的 UTF-8 字节。

```js
const encoder = new TextEncoder();
const src = 'this is some data';
const dest = new Uint8Array(10);
const { read, written } = encoder.encodeInto(src, dest);
```

### `textEncoder.encoding`

*   {字符串}

支持的编码`TextEncoder`实例。始终设置为`'utf-8'`.

## `util.toUSVString(string)`

<!-- YAML
added:
  - v16.8.0
  - v14.18.0
-->

*   `string`{字符串}

返回`string`替换任何代理代码点后
（或等效地，任何未配对的代理代码单元）与
Unicode “替换字符” U+FFFD.

## `util.transferableAbortController()`

<!-- YAML
added: REPLACEME
-->

> 稳定性： 1 - 实验

创建并返回一个 {AbortController} 实例，其 {AbortSignal} 已标记
作为可转让的，可以用于`structuredClone()`或`postMessage()`.

## `util.transferableAbortSignal(signal)`

<!-- YAML
added: REPLACEME
-->

> 稳定性： 1 - 实验

*   `signal`{中止信号}
*   返回： {中止信号}

将给定的 {AbortSignal} 标记为可转让，以便它可以与
`structuredClone()`和`postMessage()`.

```js
const signal = transferableAbortSignal(AbortSignal.timeout(100));
const channel = new MessageChannel();
channel.port2.postMessage(signal, [signal]);
```

## `util.types`

<!-- YAML
added: v10.0.0
changes:
  - version: v15.3.0
    pr-url: https://github.com/nodejs/node/pull/34055
    description: Exposed as `require('util/types')`.
-->

`util.types`为不同类型的内置对象提供类型检查。
与`instanceof`或`Object.prototype.toString.call(value)`，这些检查执行
不检查可从 JavaScript 访问的对象的属性（如
他们的原型），并且通常具有调用C++的开销。

结果一般不保证什么样的
值在 JavaScript 中公开的属性或行为。它们主要是
对于喜欢在JavaScript中进行类型检查的插件开发人员很有用。

该 API 可通过以下方式访问`require('node:util').types`或`require('node:util/types')`.

### `util.types.isAnyArrayBuffer(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`ArrayBuffer`][ArrayBuffer]或
[`SharedArrayBuffer`][SharedArrayBuffer]实例。

另请参见[`util.types.isArrayBuffer()`][util.types.isArrayBuffer()]和
[`util.types.isSharedArrayBuffer()`][util.types.isSharedArrayBuffer()].

```js
util.types.isAnyArrayBuffer(new ArrayBuffer());  // Returns true
util.types.isAnyArrayBuffer(new SharedArrayBuffer());  // Returns true
```

### `util.types.isArrayBufferView(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是[`ArrayBuffer`][ArrayBuffer]
视图，如类型化数组对象或[`DataView`][DataView].相当于
[`ArrayBuffer.isView()`][ArrayBuffer.isView()].

```js
util.types.isArrayBufferView(new Int8Array());  // true
util.types.isArrayBufferView(Buffer.from('hello world')); // true
util.types.isArrayBufferView(new DataView(new ArrayBuffer(16)));  // true
util.types.isArrayBufferView(new ArrayBuffer());  // false
```

### `util.types.isArgumentsObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值为`arguments`对象。

<!-- eslint-disable prefer-rest-params -->

```js
function foo() {
  util.types.isArgumentsObject(arguments);  // Returns true
}
```

### `util.types.isArrayBuffer(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`ArrayBuffer`][ArrayBuffer]实例。
这确实*不*包括[`SharedArrayBuffer`][SharedArrayBuffer]实例。通常，它是
希望对两者都进行测试;看[`util.types.isAnyArrayBuffer()`][util.types.isAnyArrayBuffer()]为此。

```js
util.types.isArrayBuffer(new ArrayBuffer());  // Returns true
util.types.isArrayBuffer(new SharedArrayBuffer());  // Returns false
```

### `util.types.isAsyncFunction(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值为[异步函数][async function].
这只报告了JavaScript引擎所看到的内容;
特别是，如果出现以下情况，则返回值可能与原始源代码不匹配：
使用了转译工具。

```js
util.types.isAsyncFunction(function foo() {});  // Returns false
util.types.isAsyncFunction(async function foo() {});  // Returns true
```

### `util.types.isBigInt64Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值为`BigInt64Array`实例。

```js
util.types.isBigInt64Array(new BigInt64Array());   // Returns true
util.types.isBigInt64Array(new BigUint64Array());  // Returns false
```

### `util.types.isBigUint64Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值为`BigUint64Array`实例。

```js
util.types.isBigUint64Array(new BigInt64Array());   // Returns false
util.types.isBigUint64Array(new BigUint64Array());  // Returns true
```

### `util.types.isBooleanObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是布尔对象，例如
由`new Boolean()`.

```js
util.types.isBooleanObject(false);  // Returns false
util.types.isBooleanObject(true);   // Returns false
util.types.isBooleanObject(new Boolean(false)); // Returns true
util.types.isBooleanObject(new Boolean(true));  // Returns true
util.types.isBooleanObject(Boolean(false)); // Returns false
util.types.isBooleanObject(Boolean(true));  // Returns false
```

### `util.types.isBoxedPrimitive(value)`

<!-- YAML
added: v10.11.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是任何盒装基元对象，例如
由`new Boolean()`,`new String()`或`Object(Symbol())`.

例如：

```js
util.types.isBoxedPrimitive(false); // Returns false
util.types.isBoxedPrimitive(new Boolean(false)); // Returns true
util.types.isBoxedPrimitive(Symbol('foo')); // Returns false
util.types.isBoxedPrimitive(Object(Symbol('foo'))); // Returns true
util.types.isBoxedPrimitive(Object(BigInt(5))); // Returns true
```

### `util.types.isCryptoKey(value)`

<!-- YAML
added: v16.2.0
-->

*   `value`{对象}
*   返回：{布尔值}

返回`true`如果`value`是 {CryptoKey}，`false`否则。

### `util.types.isDataView(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`DataView`][DataView]实例。

```js
const ab = new ArrayBuffer(20);
util.types.isDataView(new DataView(ab));  // Returns true
util.types.isDataView(new Float64Array());  // Returns false
```

另请参见[`ArrayBuffer.isView()`][ArrayBuffer.isView()].

### `util.types.isDate(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Date`][Date]实例。

```js
util.types.isDate(new Date());  // Returns true
```

### `util.types.isExternal(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是本机值`External`价值。

本地人`External`值是一种特殊类型的对象，它包含
原始C++指针 （`void*`） 用于从本机代码访问，并且没有其他代码
性能。此类对象由 Node.js 内部或本机创建
插件。在JavaScript中，它们是[冷冻][`Object.freeze()`]具有
`null`原型。

```c
#include <js_native_api.h>
#include <stdlib.h>
napi_value result;
static napi_value MyNapi(napi_env env, napi_callback_info info) {
  int* raw = (int*) malloc(1024);
  napi_status status = napi_create_external(env, (void*) raw, NULL, NULL, &result);
  if (status != napi_ok) {
    napi_throw_error(env, NULL, "napi_create_external failed");
    return NULL;
  }
  return result;
}
...
DECLARE_NAPI_PROPERTY("myNapi", MyNapi)
...
```

```js
const native = require('napi_addon.node');
const data = native.myNapi();
util.types.isExternal(data); // returns true
util.types.isExternal(0); // returns false
util.types.isExternal(new String('foo')); // returns false
```

有关以下内容的更多信息`napi_create_external`指
[`napi_create_external()`][napi_create_external()].

### `util.types.isFloat32Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Float32Array`][Float32Array]实例。

```js
util.types.isFloat32Array(new ArrayBuffer());  // Returns false
util.types.isFloat32Array(new Float32Array());  // Returns true
util.types.isFloat32Array(new Float64Array());  // Returns false
```

### `util.types.isFloat64Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Float64Array`][Float64Array]实例。

```js
util.types.isFloat64Array(new ArrayBuffer());  // Returns false
util.types.isFloat64Array(new Uint8Array());  // Returns false
util.types.isFloat64Array(new Float64Array());  // Returns true
```

### `util.types.isGeneratorFunction(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是生成器函数。
这只报告了JavaScript引擎所看到的内容;
特别是，如果出现以下情况，则返回值可能与原始源代码不匹配：
使用了转译工具。

```js
util.types.isGeneratorFunction(function foo() {});  // Returns false
util.types.isGeneratorFunction(function* foo() {});  // Returns true
```

### `util.types.isGeneratorObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是从
内置发电机功能。
这只报告了JavaScript引擎所看到的内容;
特别是，如果出现以下情况，则返回值可能与原始源代码不匹配：
使用了转译工具。

```js
function* foo() {}
const generator = foo();
util.types.isGeneratorObject(generator);  // Returns true
```

### `util.types.isInt8Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Int8Array`][Int8Array]实例。

```js
util.types.isInt8Array(new ArrayBuffer());  // Returns false
util.types.isInt8Array(new Int8Array());  // Returns true
util.types.isInt8Array(new Float64Array());  // Returns false
```

### `util.types.isInt16Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Int16Array`][Int16Array]实例。

```js
util.types.isInt16Array(new ArrayBuffer());  // Returns false
util.types.isInt16Array(new Int16Array());  // Returns true
util.types.isInt16Array(new Float64Array());  // Returns false
```

### `util.types.isInt32Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Int32Array`][Int32Array]实例。

```js
util.types.isInt32Array(new ArrayBuffer());  // Returns false
util.types.isInt32Array(new Int32Array());  // Returns true
util.types.isInt32Array(new Float64Array());  // Returns false
```

### `util.types.isKeyObject(value)`

<!-- YAML
added: v16.2.0
-->

*   `value`{对象}
*   返回：{布尔值}

返回`true`如果`value`是 {KeyObject}，`false`否则。

### `util.types.isMap(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Map`][Map]实例。

```js
util.types.isMap(new Map());  // Returns true
```

### `util.types.isMapIterator(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是为内置值返回的迭代器
[`Map`][Map]实例。

```js
const map = new Map();
util.types.isMapIterator(map.keys());  // Returns true
util.types.isMapIterator(map.values());  // Returns true
util.types.isMapIterator(map.entries());  // Returns true
util.types.isMapIterator(map[Symbol.iterator]());  // Returns true
```

### `util.types.isModuleNamespaceObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是[模块命名空间对象][Module Namespace Object].

<!-- eslint-skip -->

```js
import * as ns from './a.js';

util.types.isModuleNamespaceObject(ns);  // Returns true
```

### `util.types.isNativeError(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的实例[`Error`][Error]类型。

```js
util.types.isNativeError(new Error());  // Returns true
util.types.isNativeError(new TypeError());  // Returns true
util.types.isNativeError(new RangeError());  // Returns true
```

### `util.types.isNumberObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是数字对象，例如
由`new Number()`.

```js
util.types.isNumberObject(0);  // Returns false
util.types.isNumberObject(new Number(0));   // Returns true
```

### `util.types.isPromise(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Promise`][Promise].

```js
util.types.isPromise(Promise.resolve(42));  // Returns true
```

### `util.types.isProxy(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值为[`Proxy`][Proxy]实例。

```js
const target = {};
const proxy = new Proxy(target, {});
util.types.isProxy(target);  // Returns false
util.types.isProxy(proxy);  // Returns true
```

### `util.types.isRegExp(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是正则表达式对象。

```js
util.types.isRegExp(/abc/);  // Returns true
util.types.isRegExp(new RegExp('abc'));  // Returns true
```

### `util.types.isSet(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Set`][Set]实例。

```js
util.types.isSet(new Set());  // Returns true
```

### `util.types.isSetIterator(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是为内置值返回的迭代器
[`Set`][Set]实例。

```js
const set = new Set();
util.types.isSetIterator(set.keys());  // Returns true
util.types.isSetIterator(set.values());  // Returns true
util.types.isSetIterator(set.entries());  // Returns true
util.types.isSetIterator(set[Symbol.iterator]());  // Returns true
```

### `util.types.isSharedArrayBuffer(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`SharedArrayBuffer`][SharedArrayBuffer]实例。
这确实*不*包括[`ArrayBuffer`][ArrayBuffer]实例。通常，它是
希望对两者都进行测试;看[`util.types.isAnyArrayBuffer()`][util.types.isAnyArrayBuffer()]为此。

```js
util.types.isSharedArrayBuffer(new ArrayBuffer());  // Returns false
util.types.isSharedArrayBuffer(new SharedArrayBuffer());  // Returns true
```

### `util.types.isStringObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是字符串对象，例如
由`new String()`.

```js
util.types.isStringObject('foo');  // Returns false
util.types.isStringObject(new String('foo'));   // Returns true
```

### `util.types.isSymbolObject(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果值是符号对象，则创建
致电`Object()`安娜`Symbol`原始。

```js
const symbol = Symbol('foo');
util.types.isSymbolObject(symbol);  // Returns false
util.types.isSymbolObject(Object(symbol));   // Returns true
```

### `util.types.isTypedArray(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`TypedArray`][TypedArray]实例。

```js
util.types.isTypedArray(new ArrayBuffer());  // Returns false
util.types.isTypedArray(new Uint8Array());  // Returns true
util.types.isTypedArray(new Float64Array());  // Returns true
```

另请参见[`ArrayBuffer.isView()`][ArrayBuffer.isView()].

### `util.types.isUint8Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Uint8Array`][Uint8Array]实例。

```js
util.types.isUint8Array(new ArrayBuffer());  // Returns false
util.types.isUint8Array(new Uint8Array());  // Returns true
util.types.isUint8Array(new Float64Array());  // Returns false
```

### `util.types.isUint8ClampedArray(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Uint8ClampedArray`][Uint8ClampedArray]实例。

```js
util.types.isUint8ClampedArray(new ArrayBuffer());  // Returns false
util.types.isUint8ClampedArray(new Uint8ClampedArray());  // Returns true
util.types.isUint8ClampedArray(new Float64Array());  // Returns false
```

### `util.types.isUint16Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Uint16Array`][Uint16Array]实例。

```js
util.types.isUint16Array(new ArrayBuffer());  // Returns false
util.types.isUint16Array(new Uint16Array());  // Returns true
util.types.isUint16Array(new Float64Array());  // Returns false
```

### `util.types.isUint32Array(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`Uint32Array`][Uint32Array]实例。

```js
util.types.isUint32Array(new ArrayBuffer());  // Returns false
util.types.isUint32Array(new Uint32Array());  // Returns true
util.types.isUint32Array(new Float64Array());  // Returns false
```

### `util.types.isWeakMap(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`WeakMap`][WeakMap]实例。

```js
util.types.isWeakMap(new WeakMap());  // Returns true
```

### `util.types.isWeakSet(value)`

<!-- YAML
added: v10.0.0
-->

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`WeakSet`][WeakSet]实例。

```js
util.types.isWeakSet(new WeakSet());  // Returns true
```

### `util.types.isWebAssemblyCompiledModule(value)`

<!-- YAML
added: v10.0.0
deprecated: v14.0.0
-->

> 稳定性：0 - 已弃用：使用`value instanceof WebAssembly.Module`相反。

*   `value`{任何}
*   返回：{布尔值}

返回`true`如果该值是内置的[`WebAssembly.Module`][WebAssembly.Module]实例。

```js
const module = new WebAssembly.Module(wasmBuffer);
util.types.isWebAssemblyCompiledModule(module);  // Returns true
```

## 已弃用的 API

以下 API 已弃用，不应再使用。现存
应更新应用程序和模块以找到替代方法。

### `util._extend(target, source)`

<!-- YAML
added: v0.7.5
deprecated: v6.0.0
-->

> 稳定性：0 - 已弃用：使用[`Object.assign()`][Object.assign()]相反。

*   `target`{对象}
*   `source`{对象}

这`util._extend()`方法从未打算在内部外部使用
节点.js模块。无论如何，社区发现并使用它。

它已弃用，不应在新代码中使用。JavaScript 附带非常
类似的内置功能通过以下方式[`Object.assign()`][Object.assign()].

### `util.isArray(object)`

<!-- YAML
added: v0.6.0
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用[`Array.isArray()`][Array.isArray()]相反。

*   `object`{任何}
*   返回：{布尔值}

的别名[`Array.isArray()`][Array.isArray()].

返回`true`如果给定`object`是一个`Array`.否则，返回`false`.

```js
const util = require('node:util');

util.isArray([]);
// Returns: true
util.isArray(new Array());
// Returns: true
util.isArray({});
// Returns: false
```

### `util.isBoolean(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`typeof value === 'boolean'`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`Boolean`.否则，返回`false`.

```js
const util = require('node:util');

util.isBoolean(1);
// Returns: false
util.isBoolean(0);
// Returns: false
util.isBoolean(false);
// Returns: true
```

### `util.isBuffer(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用[`Buffer.isBuffer()`][Buffer.isBuffer()]相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`Buffer`.否则，返回`false`.

```js
const util = require('node:util');

util.isBuffer({ length: 0 });
// Returns: false
util.isBuffer([]);
// Returns: false
util.isBuffer(Buffer.from('hello world'));
// Returns: true
```

### `util.isDate(object)`

<!-- YAML
added: v0.6.0
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用[`util.types.isDate()`][util.types.isDate()]相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`Date`.否则，返回`false`.

```js
const util = require('node:util');

util.isDate(new Date());
// Returns: true
util.isDate(Date());
// false (without 'new' returns a String)
util.isDate({});
// Returns: false
```

### `util.isError(object)`

<!-- YAML
added: v0.6.0
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用[`util.types.isNativeError()`][util.types.isNativeError()]相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个[`Error`][Error].否则，返回
`false`.

```js
const util = require('node:util');

util.isError(new Error());
// Returns: true
util.isError(new TypeError());
// Returns: true
util.isError({ name: 'Error', message: 'an error occurred' });
// Returns: false
```

此方法依赖于`Object.prototype.toString()`行为。是的
当`object`参数操作
`@@toStringTag`.

```js
const util = require('node:util');
const obj = { name: 'Error', message: 'an error occurred' };

util.isError(obj);
// Returns: false
obj[Symbol.toStringTag] = 'Error';
util.isError(obj);
// Returns: true
```

### `util.isFunction(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`typeof value === 'function'`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`Function`.否则，返回
`false`.

```js
const util = require('node:util');

function Foo() {}
const Bar = () => {};

util.isFunction({});
// Returns: false
util.isFunction(Foo);
// Returns: true
util.isFunction(Bar);
// Returns: true
```

### `util.isNull(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`value === null`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是严格`null`.否则，返回
`false`.

```js
const util = require('node:util');

util.isNull(0);
// Returns: false
util.isNull(undefined);
// Returns: false
util.isNull(null);
// Returns: true
```

### `util.isNullOrUndefined(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用
> `value === undefined || value === null`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是`null`或`undefined`.否则
返回`false`.

```js
const util = require('node:util');

util.isNullOrUndefined(0);
// Returns: false
util.isNullOrUndefined(undefined);
// Returns: true
util.isNullOrUndefined(null);
// Returns: true
```

### `util.isNumber(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`typeof value === 'number'`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`Number`.否则，返回`false`.

```js
const util = require('node:util');

util.isNumber(false);
// Returns: false
util.isNumber(Infinity);
// Returns: true
util.isNumber(0);
// Returns: true
util.isNumber(NaN);
// Returns: true
```

### `util.isObject(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：
> 用`value !== null && typeof value === 'object'`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是严格意义上的`Object` **和**不是
`Function`（即使函数是 JavaScript 中的对象）。
否则，返回`false`.

```js
const util = require('node:util');

util.isObject(5);
// Returns: false
util.isObject(null);
// Returns: false
util.isObject({});
// Returns: true
util.isObject(() => {});
// Returns: false
```

### `util.isPrimitive(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用
> `(typeof value !== 'object' && typeof value !== 'function') || value === null`
> 相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是基元类型。否则，返回
`false`.

```js
const util = require('node:util');

util.isPrimitive(5);
// Returns: true
util.isPrimitive('foo');
// Returns: true
util.isPrimitive(false);
// Returns: true
util.isPrimitive(null);
// Returns: true
util.isPrimitive(undefined);
// Returns: true
util.isPrimitive({});
// Returns: false
util.isPrimitive(() => {});
// Returns: false
util.isPrimitive(/^$/);
// Returns: false
util.isPrimitive(new Date());
// Returns: false
```

### `util.isRegExp(object)`

<!-- YAML
added: v0.6.0
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`RegExp`.否则，返回`false`.

```js
const util = require('node:util');

util.isRegExp(/some regexp/);
// Returns: true
util.isRegExp(new RegExp('another regexp'));
// Returns: true
util.isRegExp({});
// Returns: false
```

### `util.isString(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`typeof value === 'string'`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`string`.否则，返回`false`.

```js
const util = require('node:util');

util.isString('');
// Returns: true
util.isString('foo');
// Returns: true
util.isString(String('foo'));
// Returns: true
util.isString(5);
// Returns: false
```

### `util.isSymbol(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`typeof value === 'symbol'`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是一个`Symbol`.否则，返回`false`.

```js
const util = require('node:util');

util.isSymbol(5);
// Returns: false
util.isSymbol('foo');
// Returns: false
util.isSymbol(Symbol('foo'));
// Returns: true
```

### `util.isUndefined(object)`

<!-- YAML
added: v0.11.5
deprecated: v4.0.0
-->

> 稳定性：0 - 已弃用：使用`value === undefined`相反。

*   `object`{任何}
*   返回：{布尔值}

返回`true`如果给定`object`是`undefined`.否则，返回`false`.

```js
const util = require('node:util');

const foo = undefined;
util.isUndefined(5);
// Returns: false
util.isUndefined(foo);
// Returns: true
util.isUndefined(null);
// Returns: false
```

### `util.log(string)`

<!-- YAML
added: v0.3.0
deprecated: v6.0.0
-->

> 稳定性：0 - 已弃用：请改用第三方模块。

*   `string`{字符串}

这`util.log()`方法打印给定的`string`自`stdout`包含
时间戳。

```js
const util = require('node:util');

util.log('Timestamped message.');
```

[Common System Errors]: errors.md#common-system-errors

[Custom inspection functions on objects]: #custom-inspection-functions-on-objects

[Custom promisified functions]: #custom-promisified-functions

[Customizing `util.inspect` colors]: #customizing-utilinspect-colors

[Internationalization]: intl.md

[Module Namespace Object]: https://tc39.github.io/ecma262/#sec-module-namespace-exotic-objects

[WHATWG Encoding Standard]: https://encoding.spec.whatwg.org/

[`'uncaughtException'`]: process.md#event-uncaughtexception

[`'warning'`]: process.md#event-warning

[`Array.isArray()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray

[`ArrayBuffer.isView()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer/isView

[`ArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

[`Buffer.isBuffer()`]: buffer.md#static-method-bufferisbufferobj

[`DataView`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView

[`Date`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date

[`Error`]: errors.md#class-error

[`Float32Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float32Array

[`Float64Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float64Array

[`Int16Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int16Array

[`Int32Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int32Array

[`Int8Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int8Array

[`Map`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map

[`Object.assign()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign

[`Object.freeze()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze

[`Promise`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

[`Proxy`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy

[`Set`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set

[`SharedArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer

[`TypedArray`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray

[`Uint16Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint16Array

[`Uint32Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint32Array

[`Uint8Array`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array

[`Uint8ClampedArray`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray

[`WeakMap`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap

[`WeakSet`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet

[`WebAssembly.Module`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Module

[`assert.deepStrictEqual()`]: assert.md#assertdeepstrictequalactual-expected-message

[`console.error()`]: console.md#consoleerrordata-args

[`napi_create_external()`]: n-api.md#napi_create_external

[`target` and `handler`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#Terminology

[`tty.hasColors()`]: tty.md#writestreamhascolorscount-env

[`util.format()`]: #utilformatformat-args

[`util.inspect()`]: #utilinspectobject-options

[`util.promisify()`]: #utilpromisifyoriginal

[`util.types.isAnyArrayBuffer()`]: #utiltypesisanyarraybuffervalue

[`util.types.isArrayBuffer()`]: #utiltypesisarraybuffervalue

[`util.types.isDate()`]: #utiltypesisdatevalue

[`util.types.isNativeError()`]: #utiltypesisnativeerrorvalue

[`util.types.isSharedArrayBuffer()`]: #utiltypesissharedarraybuffervalue

[async function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function

[compare function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort#Parameters

[constructor]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor

[default sort]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort

[global symbol registry]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/for

[list of deprecated APIS]: deprecations.md#list-of-deprecated-apis

[pkgjs/parseargs]: https://github.com/pkgjs/parseargs

[semantically incompatible]: https://github.com/nodejs/node/issues/4179

[util.inspect.custom]: #utilinspectcustom
