# VM (executing JavaScript)

<!--introduced_in=v0.10.0-->

> Stability: 2 - Stable

<!--name=vm-->

<!-- source_link=lib/vm.js -->

`node:vm` 模块支持在 V8 虚拟机上下文中编译和运行代码.

<strong class="critical">`node:vm` 模块不是安全机制。不要使用它来运行不受信任的代码.</strong>

JavaScript 代码可以立即编译和运行，也可以在以后编译、保存和运行.

一个常见的用例是在不同的 V8 上下文中运行代码。这意味着被调用的代码具有与调用代码不同的全局对象.

可以通过 [_contextifying_][contextified] 一个对象来提供上下文。调用的代码将上下文中的任何属性都视为全局变量。由调用代码引起的对全局变量的任何更改都会反映在上下文对象中.

```js
const vm = require('node:vm');

const x = 1;

const context = { x: 2 };
vm.createContext(context); // Contextify the object.

const code = 'x += 40; var y = 17;';
// `x` and `y` are global variables in the context.
// Initially, x has the value 2 because that is the value of context.x.
vm.runInContext(code, context);

console.log(context.x); // 42
console.log(context.y); // 17

console.log(x); // 1; y is not defined.
```

## Class: `vm.Script`

<!-- YAML
added: v0.3.1
-->

`vm.Script` 类的实例包含可以在特定上下文中执行的预编译脚本.

### `new vm.Script(code[, options])`

<!-- YAML
added: v0.3.1
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/40249
    description: Added support for import assertions to the
                 `importModuleDynamically` parameter.
  - version: v10.6.0
    pr-url: https://github.com/nodejs/node/pull/20300
    description: The `produceCachedData` is deprecated in favour of
                 `script.createCachedData()`.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4777
    description: The `cachedData` and `produceCachedData` options are
                 supported now.
-->

* `code` {string} The JavaScript code to compile.
* `options` {Object|string}
  * `filename` {string} Specifies the filename used in stack traces produced
    by this script. **Default:** `'evalmachine.<anonymous>'`.
  * `lineOffset` {number} Specifies the line number offset that is displayed
    in stack traces produced by this script. **Default:** `0`.
  * `columnOffset` {number} Specifies the first-line column number offset that
    is displayed in stack traces produced by this script. **Default:** `0`.
  * `cachedData` {Buffer|TypedArray|DataView} Provides an optional `Buffer` or
    `TypedArray`, or `DataView` with V8's code cache data for the supplied
    source. When supplied, the `cachedDataRejected` value will be set to
    either `true` or `false` depending on acceptance of the data by V8.
  * `produceCachedData` {boolean} When `true` and no `cachedData` is present, V8
    will attempt to produce code cache data for `code`. Upon success, a
    `Buffer` with V8's code cache data will be produced and stored in the
    `cachedData` property of the returned `vm.Script` instance.
    The `cachedDataProduced` value will be set to either `true` or `false`
    depending on whether code cache data is produced successfully.
    This option is **deprecated** in favor of `script.createCachedData()`.
    **Default:** `false`.
  * `importModuleDynamically` {Function} Called during evaluation of this module
    when `import()` is called. If this option is not specified, calls to
    `import()` will reject with [`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`][].
    This option is part of the experimental modules API. We do not recommend
    using it in a production environment.
    * `specifier` {string} specifier passed to `import()`
    * `script` {vm.Script}
    * `importAssertions` {Object} The `"assert"` value passed to the
      [`optionsExpression`][] optional parameter, or an empty object if no value
      was provided.
    * Returns: {Module Namespace Object|vm.Module} Returning a `vm.Module` is
      recommended in order to take advantage of error tracking, and to avoid
      issues with namespaces that contain `then` function exports.

如果 `options` 是一个字符串，那么它指定文件名.

创建一个新的 `vm.Script` 对象会编译 `code` 但不会运行它。编译后的 `vm.Script` 可以在以后多次运行。 `code` 没有绑定到任何全局对象；相反，它是在每次运行之前绑定的，只是为了那个运行.

### `script.createCachedData()`

<!-- YAML
added: v10.6.0
-->

* Returns: {Buffer}

创建可与 `Script` 构造函数的 `cachedData` 选项一起使用的代码缓存。返回一个“缓冲区”。这个方法可以在任何时间和任意次数被调用.

```js
const script = new vm.Script(`
function add(a, b) {
  return a + b;
}

const x = add(1, 2);
`);

const cacheWithoutX = script.createCachedData();

script.runInThisContext();

const cacheWithX = script.createCachedData();
```

### `script.runInContext(contextifiedObject[, options])`

<!-- YAML
added: v0.3.1
changes:
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6635
    description: The `breakOnSigint` option is supported now.
-->

* `contextifiedObject` {Object} A [contextified][] object as returned by the
  `vm.createContext()` method.
* `options` {Object}
  * `displayErrors` {boolean} When `true`, if an [`Error`][] occurs
    while compiling the `code`, the line of code causing the error is attached
    to the stack trace. **Default:** `true`.
  * `timeout` {integer} Specifies the number of milliseconds to execute `code`
    before terminating execution. If execution is terminated, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
* Returns: {any} the result of the very last statement executed in the script.

在给定的 `contextifiedObject` 中运行 `vm.Script` 对象包含的编译代码并返回结果。运行代码无权访问本地范围.

下面的示例编译代码，增加一个全局变量，设置另一个全局变量的值，然后多次执行代码。
全局变量包含在 `context` 对象中.

```js
const vm = require('node:vm');

const context = {
  animal: 'cat',
  count: 2
};

const script = new vm.Script('count += 1; name = "kitty";');

vm.createContext(context);
for (let i = 0; i < 10; ++i) {
  script.runInContext(context);
}

console.log(context);
// Prints: { animal: 'cat', count: 12, name: 'kitty' }
```

使用 `timeout` 或 `breakOnSigint` 选项将导致新的事件循环和相应的线程被启动，这具有非零的性能开销.

### `script.runInNewContext([contextObject[, options]])`

<!-- YAML
added: v0.3.1
changes:
  - version: v14.6.0
    pr-url: https://github.com/nodejs/node/pull/34023
    description: The `microtaskMode` option is supported now.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19016
    description: The `contextCodeGeneration` option is supported now.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6635
    description: The `breakOnSigint` option is supported now.
-->

* `contextObject` {Object} An object that will be [contextified][]. If
  `undefined`, a new object will be created.
* `options` {Object}
  * `displayErrors` {boolean} When `true`, if an [`Error`][] occurs
    while compiling the `code`, the line of code causing the error is attached
    to the stack trace. **Default:** `true`.
  * `timeout` {integer} Specifies the number of milliseconds to execute `code`
    before terminating execution. If execution is terminated, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
  * `contextName` {string} Human-readable name of the newly created context.
    **Default:** `'VM Context i'`, where `i` is an ascending numerical index of
    the created context.
  * `contextOrigin` {string} [Origin][origin] corresponding to the newly
    created context for display purposes. The origin should be formatted like a
    URL, but with only the scheme, host, and port (if necessary), like the
    value of the [`url.origin`][] property of a [`URL`][] object. Most notably,
    this string should omit the trailing slash, as that denotes a path.
    **Default:** `''`.
  * `contextCodeGeneration` {Object}
    * `strings` {boolean} If set to false any calls to `eval` or function
      constructors (`Function`, `GeneratorFunction`, etc) will throw an
      `EvalError`. **Default:** `true`.
    * `wasm` {boolean} If set to false any attempt to compile a WebAssembly
      module will throw a `WebAssembly.CompileError`. **Default:** `true`.
  * `microtaskMode` {string} If set to `afterEvaluate`, microtasks (tasks
    scheduled through `Promise`s and `async function`s) will be run immediately
    after the script has run. They are included in the `timeout` and
    `breakOnSigint` scopes in that case.
* Returns: {any} the result of the very last statement executed in the script.

首先将给定的 `contextObject` 上下文化，在创建的上下文中运行 `vm.Script` 对象包含的编译代码，并返回结果.
运行代码无权访问本地范围.

以下示例编译设置全局变量的代码，然后在不同的上下文中多次执行该代码。全局变量设置并包含在每个单独的“上下文”中.

```js
const vm = require('node:vm');

const script = new vm.Script('globalVar = "set"');

const contexts = [{}, {}, {}];
contexts.forEach((context) => {
  script.runInNewContext(context);
});

console.log(contexts);
// Prints: [{ globalVar: 'set' }, { globalVar: 'set' }, { globalVar: 'set' }]
```

### `script.runInThisContext([options])`

<!-- YAML
added: v0.3.1
changes:
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6635
    description: The `breakOnSigint` option is supported now.
-->

* `options` {Object}
  * `displayErrors` {boolean} When `true`, if an [`Error`][] occurs
    while compiling the `code`, the line of code causing the error is attached
    to the stack trace. **Default:** `true`.
  * `timeout` {integer} Specifies the number of milliseconds to execute `code`
    before terminating execution. If execution is terminated, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
* Returns: {any} the result of the very last statement executed in the script.

在当前 `global` 对象的上下文中运行 `vm.Script` 包含的编译代码。运行代码无权访问本地范围，但_does_有权访问当前的`global`对象.

下面的示例编译增加一个`global`变量的代码，然后多次执行该代码:

```js
const vm = require('node:vm');

global.globalVar = 0;

const script = new vm.Script('globalVar += 1', { filename: 'myfile.vm' });

for (let i = 0; i < 1000; ++i) {
  script.runInThisContext();
}

console.log(globalVar);

// 1000
```

## Class: `vm.Module`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
-->

> Stability: 1 - Experimental

此功能仅在启用 `--experimental-vm-modules` 命令标志的情况下可用.

`vm.Module` 类为在​​ VM 上下文中使用 ECMAScript 模块提供了一个低级接口。它是 `vm.Script` 类的对应物，与 ECMAScript 规范中定义的 [Module Record][]s 密切相关.

然而，与 `vm.Script` 不同的是，每个 `vm.Module` 对象从创建时就绑定到一个上下文。 `vm.Module` 对象上的操作本质上是异步的，与 `vm.Script` 对象的同步性质相反。使用“异步”函数可以帮助操作“vm.Module”对象.

使用 `vm.Module` 对象需要三个不同的步骤：创建/解析、链接和评估。这三个步骤如下图所示例子.

此实现位于 [ECMAScript Module loader][] 的较低级别。虽然计划提供支持，但目前还无法与 Loader 交互.

```mjs
import vm from 'node:vm';

const contextifiedObject = vm.createContext({
  secret: 42,
  print: console.log,
});

// Step 1
//
// Create a Module by constructing a new `vm.SourceTextModule` object. This
// parses the provided source text, throwing a `SyntaxError` if anything goes
// wrong. By default, a Module is created in the top context. But here, we
// specify `contextifiedObject` as the context this Module belongs to.
//
// Here, we attempt to obtain the default export from the module "foo", and
// put it into local binding "secret".

const bar = new vm.SourceTextModule(`
  import s from 'foo';
  s;
  print(s);
`, { context: contextifiedObject });

// Step 2
//
// "Link" the imported dependencies of this Module to it.
//
// The provided linking callback (the "linker") accepts two arguments: the
// parent module (`bar` in this case) and the string that is the specifier of
// the imported module. The callback is expected to return a Module that
// corresponds to the provided specifier, with certain requirements documented
// in `module.link()`.
//
// If linking has not started for the returned Module, the same linker
// callback will be called on the returned Module.
//
// Even top-level Modules without dependencies must be explicitly linked. The
// callback provided would never be called, however.
//
// The link() method returns a Promise that will be resolved when all the
// Promises returned by the linker resolve.
//
// Note: This is a contrived example in that the linker function creates a new
// "foo" module every time it is called. In a full-fledged module system, a
// cache would probably be used to avoid duplicated modules.

async function linker(specifier, referencingModule) {
  if (specifier === 'foo') {
    return new vm.SourceTextModule(`
      // The "secret" variable refers to the global variable we added to
      // "contextifiedObject" when creating the context.
      export default secret;
    `, { context: referencingModule.context });

    // Using `contextifiedObject` instead of `referencingModule.context`
    // here would work as well.
  }
  throw new Error(`Unable to resolve dependency: ${specifier}`);
}
await bar.link(linker);

// Step 3
//
// Evaluate the Module. The evaluate() method returns a promise which will
// resolve after the module has finished evaluating.

// Prints 42.
await bar.evaluate();
```

```cjs
const vm = require('node:vm');

const contextifiedObject = vm.createContext({
  secret: 42,
  print: console.log,
});

(async () => {
  // Step 1
  //
  // Create a Module by constructing a new `vm.SourceTextModule` object. This
  // parses the provided source text, throwing a `SyntaxError` if anything goes
  // wrong. By default, a Module is created in the top context. But here, we
  // specify `contextifiedObject` as the context this Module belongs to.
  //
  // Here, we attempt to obtain the default export from the module "foo", and
  // put it into local binding "secret".

  const bar = new vm.SourceTextModule(`
    import s from 'foo';
    s;
    print(s);
  `, { context: contextifiedObject });

  // Step 2
  //
  // "Link" the imported dependencies of this Module to it.
  //
  // The provided linking callback (the "linker") accepts two arguments: the
  // parent module (`bar` in this case) and the string that is the specifier of
  // the imported module. The callback is expected to return a Module that
  // corresponds to the provided specifier, with certain requirements documented
  // in `module.link()`.
  //
  // If linking has not started for the returned Module, the same linker
  // callback will be called on the returned Module.
  //
  // Even top-level Modules without dependencies must be explicitly linked. The
  // callback provided would never be called, however.
  //
  // The link() method returns a Promise that will be resolved when all the
  // Promises returned by the linker resolve.
  //
  // Note: This is a contrived example in that the linker function creates a new
  // "foo" module every time it is called. In a full-fledged module system, a
  // cache would probably be used to avoid duplicated modules.

  async function linker(specifier, referencingModule) {
    if (specifier === 'foo') {
      return new vm.SourceTextModule(`
        // The "secret" variable refers to the global variable we added to
        // "contextifiedObject" when creating the context.
        export default secret;
      `, { context: referencingModule.context });

      // Using `contextifiedObject` instead of `referencingModule.context`
      // here would work as well.
    }
    throw new Error(`Unable to resolve dependency: ${specifier}`);
  }
  await bar.link(linker);

  // Step 3
  //
  // Evaluate the Module. The evaluate() method returns a promise which will
  // resolve after the module has finished evaluating.

  // Prints 42.
  await bar.evaluate();
})();
```

### `module.dependencySpecifiers`

* {string\[]}

此模块的所有依赖项的说明符。返回的数组被冻结以禁止对其进行任何更改.

对应 ECMAScript 规范中 [Cyclic Module Record][]s 的 `[[RequestedModules]]` 字段.

### `module.error`

* {any}

如果 `module.status` 为 `'errored'`，则此属性包含模块在评估期间抛出的异常。如果状态是其他，访问该属性将导致抛出异常.

值 `undefined` 不能用于没有抛出异常的情况，因为可能与 `throw undefined;` 有歧义。.

对应 ECMAScript 规范中 [Cyclic Module Record][]s 的 `[[EvaluationError]]` 字段.

### `module.evaluate([options])`

* `options` {Object}
  * `timeout` {integer} Specifies the number of milliseconds to evaluate
    before terminating execution. If execution is interrupted, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
* Returns: {Promise} Fulfills with `undefined` upon success.

评估模块.

这必须在模块链接后调用；否则它将拒绝.
当模块已经被评估时，它也可以被调用，在这种情况下，如果初始评估成功结束（`module.status` 是 `'evaluated'`），它要么什么都不做，要么重新抛出异常初始评估导致（`module.status` is `'errored'`）.

在评估模块时无法调用此方法（`module.status` 是 `'evalating'`）.

对应ECMAScript规范中[Cyclic Module Record][]s的[Evaluate()具体方法][]字段.

### `module.identifier`

* {string}

当前模块的标识符，在构造函数中设置.

### `module.link(linker)`

* `linker` {Function}
  * `specifier` {string} The specifier of the requested module:
    ```mjs
    import foo from 'foo';
    //              ^^^^^ the module specifier
    ```

  * `referencingModule` {vm.Module} The `Module` object `link()` is called on.

  * `extra` {Object}
    * `assert` {Object} The data from the assertion:
      <!-- eslint-skip -->
      ```js
      import foo from 'foo' assert { name: 'value' };
      //                           ^^^^^^^^^^^^^^^^^ the assertion
      ```
      根据 ECMA-262，主机应该忽略它们不支持的断言，而不是，例如，如果存在不受支持的断言则触发错误.

  * Returns: {vm.Module|Promise}
* Returns: {Promise}

链接模块依赖项。该方法必须在求值前调用，每个模块只能调用一次.

该函数应返回一个“Module”对象或一个“Promise”，最终解析为一个“Module”对象。返回的“模块”必须满足
遵循两个不变量:

* 它必须与父模块属于同一上下文.
* 它的`status`不能是`'errored'`.

如果返回的 `Module` 的 `status` 是 `'unlinked'`，则该方法将在返回的 `Module` 上递归调用，并提供相同的 `linker` 函数.

`link()` 返回一个 `Promise`，当所有链接实例解析为有效的 `Module` 时，该 Promise 将被解析，或者如果链接器函数抛出异常或返回无效的 `Module` 则被拒绝.

链接器函数大致对应于 ECMAScript 规范中实现定义的 [HostResolveImportedModule][] 抽象操作，有几个关键区别:

* 当 [HostResolveImportedModule][] 是同步的时，链接器函数允许是异步的.

在模块链接期间使用的实际 [HostResolveImportedModule][] 实现是返回链接期间链接的模块的实现。由于此时所有模块都已完全链接，因此 [HostResolveImportedModule][] 实现按照规范完全同步.

对应 ECMAScript 规范中 [Cyclic Module Record][]s 的[Link()具体方法][]字段.

### `module.namespace`

* {Object}

The namespace object of the module. This is only available after linking (`module.link()`) has completed.

Corresponds to the [GetModuleNamespace][] abstract operation in the ECMAScript specification.

### `module.status`

* {string}

模块的当前状态。将是其中之一:

* `'unlinked'`: `module.link()` 尚未被调用.

* `'linking'`: `module.link()` 已被调用，但尚未解决链接器函数返回的所有 Promise.

* `'linked'`: 模块已链接成功，并且其所有依赖项都已链接，但尚未调用 `module.evaluate()`.

* `'evaluating'`: 模块正在通过自身或父模块上的“module.evaluate()”进行评估.

* `'evaluated'`: 该模块已成功评估.

* `'errored'`: 模块已评估，但抛出异常.

除了 `'errored'`，此状态字符串对应于规范的 [Cyclic Module Record][] 的 `[[Status]]` 字段。 `'errored'` 对应于规范中的 `'evaluated'`，但 `[[EvaluationError]]` 设置为非 `undefined` 的值.

## Class: `vm.SourceTextModule`

<!-- YAML
added: v9.6.0
-->

> Stability: 1 - Experimental

此功能仅在启用 `--experimental-vm-modules` 命令标志的情况下可用.

* Extends: {vm.Module}

`vm.SourceTextModule` 类提供了 ECMAScript 规范中定义的 [源文本模块记录][].

### `new vm.SourceTextModule(code[, options])`

<!-- YAML
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/40249
    description: Added support for import assertions to the
                 `importModuleDynamically` parameter.
-->

* `code` {string} JavaScript Module code to parse
* `options`
  * `identifier` {string} String used in stack traces.
    **Default:** `'vm:module(i)'` where `i` is a context-specific ascending
    index.
  * `cachedData` {Buffer|TypedArray|DataView} Provides an optional `Buffer` or
    `TypedArray`, or `DataView` with V8's code cache data for the supplied
    source. The `code` must be the same as the module from which this
    `cachedData` was created.
  * `context` {Object} The [contextified][] object as returned by the
    `vm.createContext()` method, to compile and evaluate this `Module` in.
  * `lineOffset` {integer} Specifies the line number offset that is displayed
    in stack traces produced by this `Module`. **Default:** `0`.
  * `columnOffset` {integer} Specifies the first-line column number offset that
    is displayed in stack traces produced by this `Module`. **Default:** `0`.
  * `initializeImportMeta` {Function} Called during evaluation of this `Module`
    to initialize the `import.meta`.
    * `meta` {import.meta}
    * `module` {vm.SourceTextModule}
  * `importModuleDynamically` {Function} Called during evaluation of this module
    when `import()` is called. If this option is not specified, calls to
    `import()` will reject with [`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`][].
    * `specifier` {string} specifier passed to `import()`
    * `module` {vm.Module}
    * `importAssertions` {Object} The `"assert"` value passed to the
      [`optionsExpression`][] optional parameter, or an empty object if no value
      was provided.
    * Returns: {Module Namespace Object|vm.Module} Returning a `vm.Module` is
      recommended in order to take advantage of error tracking, and to avoid
      issues with namespaces that contain `then` function exports.

创建一个新的 `SourceTextModule` 实例.

分配给“import.meta”对象的属性可以允许模块访问指定“上下文”之外的信息。使用 `vm.runInContext()` 在特定上下文中创建对象.

```mjs
import vm from 'node:vm';

const contextifiedObject = vm.createContext({ secret: 42 });

const module = new vm.SourceTextModule(
  'Object.getPrototypeOf(import.meta.prop).secret = secret;',
  {
    initializeImportMeta(meta) {
      // Note: this object is created in the top context. As such,
      // Object.getPrototypeOf(import.meta.prop) points to the
      // Object.prototype in the top context rather than that in
      // the contextified object.
      meta.prop = {};
    }
  });
// Since module has no dependencies, the linker function will never be called.
await module.link(() => {});
await module.evaluate();

// Now, Object.prototype.secret will be equal to 42.
//
// To fix this problem, replace
//     meta.prop = {};
// above with
//     meta.prop = vm.runInContext('{}', contextifiedObject);
```

```cjs
const vm = require('node:vm');
const contextifiedObject = vm.createContext({ secret: 42 });
(async () => {
  const module = new vm.SourceTextModule(
    'Object.getPrototypeOf(import.meta.prop).secret = secret;',
    {
      initializeImportMeta(meta) {
        // Note: this object is created in the top context. As such,
        // Object.getPrototypeOf(import.meta.prop) points to the
        // Object.prototype in the top context rather than that in
        // the contextified object.
        meta.prop = {};
      }
    });
  // Since module has no dependencies, the linker function will never be called.
  await module.link(() => {});
  await module.evaluate();
  // Now, Object.prototype.secret will be equal to 42.
  //
  // To fix this problem, replace
  //     meta.prop = {};
  // above with
  //     meta.prop = vm.runInContext('{}', contextifiedObject);
})();
```

### `sourceTextModule.createCachedData()`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
-->

* Returns: {Buffer}

创建可与 `SourceTextModule` 构造函数的 `cachedData` 选项一起使用的代码缓存。返回一个“缓冲区”。在评估模块之前，可以多次调用此方法.

```js
// Create an initial module
const module = new vm.SourceTextModule('const a = 1;');

// Create cached data from this module
const cachedData = module.createCachedData();

// Create a new module using the cached data. The code must be the same.
const module2 = new vm.SourceTextModule('const a = 1;', { cachedData });
```

## Class: `vm.SyntheticModule`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
-->

> Stability: 1 - Experimental

此功能仅在启用 `--experimental-vm-modules` 命令标志的情况下可用.

* Extends: {vm.Module}

`vm.SyntheticModule` 类提供了 WebIDL 规范中定义的 [Synthetic Module Record][]。合成模块的目的是提供一个通用接口，用于将非 JavaScript 源代码暴露给 ECMAScript 模块图.

```js
const vm = require('node:vm');

const source = '{ "a": 1 }';
const module = new vm.SyntheticModule(['default'], function() {
  const obj = JSON.parse(source);
  this.setExport('default', obj);
});

// Use `module` in linking...
```

### `new vm.SyntheticModule(exportNames, evaluateCallback[, options])`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
-->

* `exportNames` {string\[]} Array of names that will be exported from the
  module.
* `evaluateCallback` {Function} Called when the module is evaluated.
* `options`
  * `identifier` {string} String used in stack traces.
    **Default:** `'vm:module(i)'` where `i` is a context-specific ascending
    index.
  * `context` {Object} The [contextified][] object as returned by the
    `vm.createContext()` method, to compile and evaluate this `Module` in.

创建一个新的 `SyntheticModule` 实例.

分配给此实例的导出的对象可能允许模块的导入器访问指定的“上下文”之外的信息。使用 `vm.runInContext()` 在特定上下文中创建对象.

### `syntheticModule.setExport(name, value)`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
-->

* `name` {string} Name of the export to set.
* `value` {any} The value to set the export to.

此方法在模块链接后使用，用于设置导出的值。如果在模块链接之前调用它，会抛出 [`ERR_VM_MODULE_STATUS`][] 错误.

```mjs
import vm from 'node:vm';

const m = new vm.SyntheticModule(['x'], () => {
  m.setExport('x', 1);
});

await m.link(() => {});
await m.evaluate();

assert.strictEqual(m.namespace.x, 1);
```

```cjs
const vm = require('node:vm');
(async () => {
  const m = new vm.SyntheticModule(['x'], () => {
    m.setExport('x', 1);
  });
  await m.link(() => {});
  await m.evaluate();
  assert.strictEqual(m.namespace.x, 1);
})();
```

## `vm.compileFunction(code[, params[, options]])`

<!-- YAML
added: v10.10.0
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/40249
    description: Added support for import assertions to the
                 `importModuleDynamically` parameter.
  - version: v15.9.0
    pr-url: https://github.com/nodejs/node/pull/35431
    description: Added `importModuleDynamically` option again.
  - version: v14.3.0
    pr-url: https://github.com/nodejs/node/pull/33364
    description: Removal of `importModuleDynamically` due to compatibility
                 issues.
  - version:
    - v14.1.0
    - v13.14.0
    pr-url: https://github.com/nodejs/node/pull/32985
    description: The `importModuleDynamically` option is now supported.
-->

* `code` {string} The body of the function to compile.
* `params` {string\[]} An array of strings containing all parameters for the
  function.
* `options` {Object}
  * `filename` {string} Specifies the filename used in stack traces produced
    by this script. **Default:** `''`.
  * `lineOffset` {number} Specifies the line number offset that is displayed
    in stack traces produced by this script. **Default:** `0`.
  * `columnOffset` {number} Specifies the first-line column number offset that
    is displayed in stack traces produced by this script. **Default:** `0`.
  * `cachedData` {Buffer|TypedArray|DataView} Provides an optional `Buffer` or
    `TypedArray`, or `DataView` with V8's code cache data for the supplied
    source.
  * `produceCachedData` {boolean} Specifies whether to produce new cache data.
    **Default:** `false`.
  * `parsingContext` {Object} The [contextified][] object in which the said
    function should be compiled in.
  * `contextExtensions` {Object\[]} An array containing a collection of context
    extensions (objects wrapping the current scope) to be applied while
    compiling. **Default:** `[]`.
  * `importModuleDynamically` {Function} Called during evaluation of this module
    when `import()` is called. If this option is not specified, calls to
    `import()` will reject with [`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`][].
    This option is part of the experimental modules API, and should not be
    considered stable.
    * `specifier` {string} specifier passed to `import()`
    * `function` {Function}
    * `importAssertions` {Object} The `"assert"` value passed to the
      [`optionsExpression`][] optional parameter, or an empty object if no value
      was provided.
    * Returns: {Module Namespace Object|vm.Module} Returning a `vm.Module` is
      recommended in order to take advantage of error tracking, and to avoid
      issues with namespaces that contain `then` function exports.
* Returns: {Function}

将给定的代码编译到提供的上下文中（如果没有提供上下文，则使用当前上下文），并将其包装在具有给定 `params` 的函数中.

## `vm.createContext([contextObject[, options]])`

<!-- YAML
added: v0.3.1
changes:
  - version: v14.6.0
    pr-url: https://github.com/nodejs/node/pull/34023
    description: The `microtaskMode` option is supported now.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19398
    description: The first argument can no longer be a function.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19016
    description: The `codeGeneration` option is supported now.
-->

* `contextObject` {Object}
* `options` {Object}
  * `name` {string} Human-readable name of the newly created context.
    **Default:** `'VM Context i'`, where `i` is an ascending numerical index of
    the created context.
  * `origin` {string} [Origin][origin] corresponding to the newly created
    context for display purposes. The origin should be formatted like a URL,
    but with only the scheme, host, and port (if necessary), like the value of
    the [`url.origin`][] property of a [`URL`][] object. Most notably, this
    string should omit the trailing slash, as that denotes a path.
    **Default:** `''`.
  * `codeGeneration` {Object}
    * `strings` {boolean} If set to false any calls to `eval` or function
      constructors (`Function`, `GeneratorFunction`, etc) will throw an
      `EvalError`. **Default:** `true`.
    * `wasm` {boolean} If set to false any attempt to compile a WebAssembly
      module will throw a `WebAssembly.CompileError`. **Default:** `true`.
  * `microtaskMode` {string} If set to `afterEvaluate`, microtasks (tasks
    scheduled through `Promise`s and `async function`s) will be run immediately
    after a script has run through [`script.runInContext()`][].
    They are included in the `timeout` and `breakOnSigint` scopes in that case.
* Returns: {Object} contextified object.

如果给定一个 `contextObject`，`vm.createContext()` 方法将 [准备该对象][contextified] 以便它可以在调用 [`vm.runInContext()`][] 或 [`script. runInContext()`][]。在此类脚本中，`contextObject` 将是全局对象，保留其所有现有属性，但也具有任何标准 [全局对象][] 具有的内置对象和功能。在 vm 模块运行的脚本之外，全局变量将保持不变.

```js
const vm = require('node:vm');

global.globalVar = 3;

const context = { globalVar: 1 };
vm.createContext(context);

vm.runInContext('globalVar *= 2;', context);

console.log(context);
// Prints: { globalVar: 2 }

console.log(global.globalVar);
// Prints: 3
```

如果 `contextObject` 被省略（或显式传递为 `undefined`），将返回一个新的空 [contextified][] 对象.

`vm.createContext()` 方法主要用于创建可用于运行多个脚本的单个上下文。例如，如果模拟 Web 浏览器，该方法可用于创建表示窗口全局对象的单个上下文，然后在该上下文中一起运行所有 `<script>` 标记.

提供的上下文的“名称”和“来源”通过 Inspector API 可见.

## `vm.isContext(object)`

<!-- YAML
added: v0.11.7
-->

* `object` {Object}
* Returns: {boolean}

如果给定的 `object` 对象已使用 [`vm.createContext()`][] [上下文化][]，则返回 `true`.

## `vm.measureMemory([options])`

<!-- YAML
added: v13.10.0
-->

> Stability: 1 - Experimental

测量 V8 已知并由当前 V8 隔离或主上下文已知的所有上下文使用的内存.

* `options` {Object} Optional.
  * `mode` {string} Either `'summary'` or `'detailed'`. In summary mode,
    only the memory measured for the main context will be returned. In
    detailed mode, the memory measured for all contexts known to the
    current V8 isolate will be returned.
    **Default:** `'summary'`
  * `execution` {string} Either `'default'` or `'eager'`. With default
    execution, the promise will not resolve until after the next scheduled
    garbage collection starts, which may take a while (or never if the program
    exits before the next GC). With eager execution, the GC will be started
    right away to measure the memory.
    **Default:** `'default'`
* Returns: {Promise} If the memory is successfully measured the promise will
  resolve with an object containing information about the memory usage.

返回的 Promise 可以解析的对象的格式特定于 V8 引擎，并且可能会从 V8 的一个版本更改为下一个版本.

返回的结果与 `v8.getHeapSpaceStatistics()` 返回的统计信息不同，`vm.measureMemory()` 测量 V8 引擎当前实例中每个 V8 特定上下文可访问的内存，而 `v8 .getHeapSpaceStatistics()`测量当前V8实例中每个堆空间占用的内存.

```js
const vm = require('node:vm');
// Measure the memory used by the main context.
vm.measureMemory({ mode: 'summary' })
  // This is the same as vm.measureMemory()
  .then((result) => {
    // The current format is:
    // {
    //   total: {
    //      jsMemoryEstimate: 2418479, jsMemoryRange: [ 2418479, 2745799 ]
    //    }
    // }
    console.log(result);
  });

const context = vm.createContext({ a: 1 });
vm.measureMemory({ mode: 'detailed', execution: 'eager' })
  .then((result) => {
    // Reference the context here so that it won't be GC'ed
    // until the measurement is complete.
    console.log(context.a);
    // {
    //   total: {
    //     jsMemoryEstimate: 2574732,
    //     jsMemoryRange: [ 2574732, 2904372 ]
    //   },
    //   current: {
    //     jsMemoryEstimate: 2438996,
    //     jsMemoryRange: [ 2438996, 2768636 ]
    //   },
    //   other: [
    //     {
    //       jsMemoryEstimate: 135736,
    //       jsMemoryRange: [ 135736, 465376 ]
    //     }
    //   ]
    // }
    console.log(result);
  });
```

## `vm.runInContext(code, contextifiedObject[, options])`

<!-- YAML
added: v0.3.1
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/40249
    description: Added support for import assertions to the
                 `importModuleDynamically` parameter.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6635
    description: The `breakOnSigint` option is supported now.
-->

* `code` {string} The JavaScript code to compile and run.
* `contextifiedObject` {Object} The [contextified][] object that will be used
  as the `global` when the `code` is compiled and run.
* `options` {Object|string}
  * `filename` {string} Specifies the filename used in stack traces produced
    by this script. **Default:** `'evalmachine.<anonymous>'`.
  * `lineOffset` {number} Specifies the line number offset that is displayed
    in stack traces produced by this script. **Default:** `0`.
  * `columnOffset` {number} Specifies the first-line column number offset that
    is displayed in stack traces produced by this script. **Default:** `0`.
  * `displayErrors` {boolean} When `true`, if an [`Error`][] occurs
    while compiling the `code`, the line of code causing the error is attached
    to the stack trace. **Default:** `true`.
  * `timeout` {integer} Specifies the number of milliseconds to execute `code`
    before terminating execution. If execution is terminated, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
  * `cachedData` {Buffer|TypedArray|DataView} Provides an optional `Buffer` or
    `TypedArray`, or `DataView` with V8's code cache data for the supplied
    source. When supplied, the `cachedDataRejected` value will be set to
    either `true` or `false` depending on acceptance of the data by V8.
  * `produceCachedData` {boolean} When `true` and no `cachedData` is present, V8
    will attempt to produce code cache data for `code`. Upon success, a
    `Buffer` with V8's code cache data will be produced and stored in the
    `cachedData` property of the returned `vm.Script` instance.
    The `cachedDataProduced` value will be set to either `true` or `false`
    depending on whether code cache data is produced successfully.
    This option is **deprecated** in favor of `script.createCachedData()`.
    **Default:** `false`.
  * `importModuleDynamically` {Function} Called during evaluation of this module
    when `import()` is called. If this option is not specified, calls to
    `import()` will reject with [`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`][].
    This option is part of the experimental modules API. We do not recommend
    using it in a production environment.
    * `specifier` {string} specifier passed to `import()`
    * `script` {vm.Script}
    * `importAssertions` {Object} The `"assert"` value passed to the
      [`optionsExpression`][] optional parameter, or an empty object if no value
      was provided.
    * Returns: {Module Namespace Object|vm.Module} Returning a `vm.Module` is
      recommended in order to take advantage of error tracking, and to avoid
      issues with namespaces that contain `then` function exports.
* Returns: {any} the result of the very last statement executed in the script.

`vm.runInContext()` 方法编译`code`，在`contextifiedObject` 的上下文中运行它，然后返回结果。运行代码无权访问本地范围。 `contextifiedObject` 对象_必须_ 之前已使用 [`vm.createContext()`][] 方法 [contextified][].

如果 `options` 是一个字符串，那么它指定文件名.

以下示例使用单个 [contextified][] 对象编译和执行不同的脚本:

```js
const vm = require('node:vm');

const contextObject = { globalVar: 1 };
vm.createContext(contextObject);

for (let i = 0; i < 10; ++i) {
  vm.runInContext('globalVar *= 2;', contextObject);
}
console.log(contextObject);
// Prints: { globalVar: 1024 }
```

## `vm.runInNewContext(code[, contextObject[, options]])`

<!-- YAML
added: v0.3.1
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/40249
    description: Added support for import assertions to the
                 `importModuleDynamically` parameter.
  - version: v14.6.0
    pr-url: https://github.com/nodejs/node/pull/34023
    description: The `microtaskMode` option is supported now.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19016
    description: The `contextCodeGeneration` option is supported now.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6635
    description: The `breakOnSigint` option is supported now.
-->

* `code` {string} The JavaScript code to compile and run.
* `contextObject` {Object} An object that will be [contextified][]. If
  `undefined`, a new object will be created.
* `options` {Object|string}
  * `filename` {string} Specifies the filename used in stack traces produced
    by this script. **Default:** `'evalmachine.<anonymous>'`.
  * `lineOffset` {number} Specifies the line number offset that is displayed
    in stack traces produced by this script. **Default:** `0`.
  * `columnOffset` {number} Specifies the first-line column number offset that
    is displayed in stack traces produced by this script. **Default:** `0`.
  * `displayErrors` {boolean} When `true`, if an [`Error`][] occurs
    while compiling the `code`, the line of code causing the error is attached
    to the stack trace. **Default:** `true`.
  * `timeout` {integer} Specifies the number of milliseconds to execute `code`
    before terminating execution. If execution is terminated, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
  * `contextName` {string} Human-readable name of the newly created context.
    **Default:** `'VM Context i'`, where `i` is an ascending numerical index of
    the created context.
  * `contextOrigin` {string} [Origin][origin] corresponding to the newly
    created context for display purposes. The origin should be formatted like a
    URL, but with only the scheme, host, and port (if necessary), like the
    value of the [`url.origin`][] property of a [`URL`][] object. Most notably,
    this string should omit the trailing slash, as that denotes a path.
    **Default:** `''`.
  * `contextCodeGeneration` {Object}
    * `strings` {boolean} If set to false any calls to `eval` or function
      constructors (`Function`, `GeneratorFunction`, etc) will throw an
      `EvalError`. **Default:** `true`.
    * `wasm` {boolean} If set to false any attempt to compile a WebAssembly
      module will throw a `WebAssembly.CompileError`. **Default:** `true`.
  * `cachedData` {Buffer|TypedArray|DataView} Provides an optional `Buffer` or
    `TypedArray`, or `DataView` with V8's code cache data for the supplied
    source. When supplied, the `cachedDataRejected` value will be set to
    either `true` or `false` depending on acceptance of the data by V8.
  * `produceCachedData` {boolean} When `true` and no `cachedData` is present, V8
    will attempt to produce code cache data for `code`. Upon success, a
    `Buffer` with V8's code cache data will be produced and stored in the
    `cachedData` property of the returned `vm.Script` instance.
    The `cachedDataProduced` value will be set to either `true` or `false`
    depending on whether code cache data is produced successfully.
    This option is **deprecated** in favor of `script.createCachedData()`.
    **Default:** `false`.
  * `importModuleDynamically` {Function} Called during evaluation of this module
    when `import()` is called. If this option is not specified, calls to
    `import()` will reject with [`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`][].
    This option is part of the experimental modules API. We do not recommend
    using it in a production environment.
    * `specifier` {string} specifier passed to `import()`
    * `script` {vm.Script}
    * `importAssertions` {Object} The `"assert"` value passed to the
      [`optionsExpression`][] optional parameter, or an empty object if no value
      was provided.
    * Returns: {Module Namespace Object|vm.Module} Returning a `vm.Module` is
      recommended in order to take advantage of error tracking, and to avoid
      issues with namespaces that contain `then` function exports.
  * `microtaskMode` {string} If set to `afterEvaluate`, microtasks (tasks
    scheduled through `Promise`s and `async function`s) will be run immediately
    after the script has run. They are included in the `timeout` and
    `breakOnSigint` scopes in that case.
* Returns: {any} the result of the very last statement executed in the script.

`vm.runInNewContext()` 首先将给定的`contextObject` 上下文化（或者如果作为`undefined` 传递，则创建一个新的`contextObject`），编译`code`，在创建的上下文中运行它，然后返回结果。运行代码无权访问本地范围.

如果 `options` 是一个字符串，那么它指定文件名.

以下示例编译并执行递增全局变量并设置新变量的代码。这些全局变量包含在 `contextObject`.

```js
const vm = require('node:vm');

const contextObject = {
  animal: 'cat',
  count: 2
};

vm.runInNewContext('count += 1; name = "kitty"', contextObject);
console.log(contextObject);
// Prints: { animal: 'cat', count: 3, name: 'kitty' }
```

## `vm.runInThisContext(code[, options])`

<!-- YAML
added: v0.3.1
changes:
  - version:
    - v17.0.0
    - v16.12.0
    pr-url: https://github.com/nodejs/node/pull/40249
    description: Added support for import assertions to the
                 `importModuleDynamically` parameter.
  - version: v6.3.0
    pr-url: https://github.com/nodejs/node/pull/6635
    description: The `breakOnSigint` option is supported now.
-->

* `code` {string} The JavaScript code to compile and run.
* `options` {Object|string}
  * `filename` {string} Specifies the filename used in stack traces produced
    by this script. **Default:** `'evalmachine.<anonymous>'`.
  * `lineOffset` {number} Specifies the line number offset that is displayed
    in stack traces produced by this script. **Default:** `0`.
  * `columnOffset` {number} Specifies the first-line column number offset that
    is displayed in stack traces produced by this script. **Default:** `0`.
  * `displayErrors` {boolean} When `true`, if an [`Error`][] occurs
    while compiling the `code`, the line of code causing the error is attached
    to the stack trace. **Default:** `true`.
  * `timeout` {integer} Specifies the number of milliseconds to execute `code`
    before terminating execution. If execution is terminated, an [`Error`][]
    will be thrown. This value must be a strictly positive integer.
  * `breakOnSigint` {boolean} If `true`, receiving `SIGINT`
    (<kbd>Ctrl</kbd>+<kbd>C</kbd>) will terminate execution and throw an
    [`Error`][]. Existing handlers for the event that have been attached via
    `process.on('SIGINT')` are disabled during script execution, but continue to
    work after that. **Default:** `false`.
  * `cachedData` {Buffer|TypedArray|DataView} Provides an optional `Buffer` or
    `TypedArray`, or `DataView` with V8's code cache data for the supplied
    source. When supplied, the `cachedDataRejected` value will be set to
    either `true` or `false` depending on acceptance of the data by V8.
  * `produceCachedData` {boolean} When `true` and no `cachedData` is present, V8
    will attempt to produce code cache data for `code`. Upon success, a
    `Buffer` with V8's code cache data will be produced and stored in the
    `cachedData` property of the returned `vm.Script` instance.
    The `cachedDataProduced` value will be set to either `true` or `false`
    depending on whether code cache data is produced successfully.
    This option is **deprecated** in favor of `script.createCachedData()`.
    **Default:** `false`.
  * `importModuleDynamically` {Function} Called during evaluation of this module
    when `import()` is called. If this option is not specified, calls to
    `import()` will reject with [`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`][].
    This option is part of the experimental modules API. We do not recommend
    using it in a production environment.
    * `specifier` {string} specifier passed to `import()`
    * `script` {vm.Script}
    * `importAssertions` {Object} The `"assert"` value passed to the
      [`optionsExpression`][] optional parameter, or an empty object if no value
      was provided.
    * Returns: {Module Namespace Object|vm.Module} Returning a `vm.Module` is
      recommended in order to take advantage of error tracking, and to avoid
      issues with namespaces that contain `then` function exports.
* Returns: {any} the result of the very last statement executed in the script.

`vm.runInThisContext()` 编译 `code`，在当前 `global` 的上下文中运行它并返回结果。运行代码无权访问本地范围，但可以访问当前的“全局”对象.

如果 `options` 是一个字符串，那么它指定文件名.

以下示例说明了使用 `vm.runInThisContext()` 和 JavaScript [`eval()`][] 函数来运行相同的代码:

<!-- eslint-disable prefer-const -->

```js
const vm = require('node:vm');
let localVar = 'initial value';

const vmResult = vm.runInThisContext('localVar = "vm";');
console.log(`vmResult: '${vmResult}', localVar: '${localVar}'`);
// Prints: vmResult: 'vm', localVar: 'initial value'

const evalResult = eval('localVar = "eval";');
console.log(`evalResult: '${evalResult}', localVar: '${localVar}'`);
// Prints: evalResult: 'eval', localVar: 'eval'
```

因为 `vm.runInThisContext()` 无法访问本地范围，所以 `localVar` 没有改变。相反， [`eval()`][] _does_ 可以访问本地范围，因此值 `localVar` 被更改。这样，`vm.runInThisContext()` 很像 [indirect `eval()` call][]，例如`(0,eval)('code')`.

## Example: Running an HTTP server within a VM

当使用 [`script.runInThisContext()`][] 或 [`vm.runInThisContext()`][] 时，代码在当前 V8 全局上下文中执行。传递给此 VM 上下文的代码将具有自己的隔离范围.

为了使用 `node:http` 模块运行一个简单的 web 服务器，传递给上下文的代码必须自己调用`require('node:http')`，或者引用`node:http`模块传递给它。例如:

```js
'use strict';
const vm = require('node:vm');

const code = `
((require) => {
  const http = require('node:http');

  http.createServer((request, response) => {
    response.writeHead(200, { 'Content-Type': 'text/plain' });
    response.end('Hello World\\n');
  }).listen(8124);

  console.log('Server running at http://127.0.0.1:8124/');
})`;

vm.runInThisContext(code)(require);
```

上述情况下的 `require()` 与传递它的上下文共享状态。当执行不受信任的代码时，这可能会带来风险，例如以不需要的方式改变上下文中的对象.

## What does it mean to "contextify" an object?

在 Node.js 中执行的所有 JavaScript 都在“上下文”范围内运行.
根据 [V8 嵌入器指南][]:

> 在 V8 中，上下文是允许分离的、不相关的执行环境,
> 在 V8 的单个实例中运行的 JavaScript 应用程序。您必须明确
> 指定您希望运行任何 JavaScript 代码的上下文.

当调用 `vm.createContext()` 方法时，`contextObject` 参数（如果 `contextObject` 为 `undefined`，则为新创建的对象）在内部与 V8 上下文的新实例相关联。这个 V8 上下文为使用 `node:vm` 模块的方法运行的`code`提供了一个可以运行的隔离全局环境。创建 V8 上下文并将其与 `contextObject` 关联的过程是本文档所说的“上下文化”对象.

## Timeout interactions with asynchronous tasks and Promises

`Promise`s 和 `async function`s 可以异步调度 JavaScript 引擎运行的任务。默认情况下，这些任务在当前堆栈上的所有 JavaScript 函数执行完毕后运行.
这允许转义 `timeout` 和 `breakOnSigint` 选项的功能.

例如，由 `vm.runInNewContext()` 执行的以下代码，超时时间为 5 毫秒，会在 Promise 解决后安排无限循环运行。计划的循环永远不会被超时中断:

```js
const vm = require('node:vm');

function loop() {
  console.log('entering loop');
  while (1) console.log(Date.now());
}

vm.runInNewContext(
  'Promise.resolve().then(() => loop());',
  { loop, console },
  { timeout: 5 }
);
// This is printed *before* 'entering loop' (!)
console.log('done executing');
```

这可以通过将 `microtaskMode: 'afterEvaluate'` 传递给创建 `Context` 的代码来解决:

```js
const vm = require('node:vm');

function loop() {
  while (1) console.log(Date.now());
}

vm.runInNewContext(
  'Promise.resolve().then(() => loop());',
  { loop, console },
  { timeout: 5, microtaskMode: 'afterEvaluate' }
);
```

在这种情况下，通过 `promise.then()` 调度的微任务将在从 `vm.runInNewContext()` 返回之前运行，并会被 `timeout` 功能中断。这仅适用于在 `vm.Context` 中运行的代码，例如[`vm.runInThisContext()`][] 不采用此选项.

Promise 回调被输入到创建它们的上下文的微任务队列中。例如，如果在上面的例子中 `() => loop()` 被替换为 `loop`，那么 `loop` 将被推送到全局微任务队列中，因为它是来自外部（主）上下文的函数，因此也将能够逃脱超时.

如果异步调度函数，如 `process.nextTick()`、`queueMicrotask()`、`setTimeout()`、`setImmediate()` 等在 `vm.Context` 中可用，传递给它们的函数将被添加到由所有上下文共享的全局队列中。因此，传递给这些函数的回调也无法通过超时来控制.

[Cyclic Module Record]: https://tc39.es/ecma262/#sec-cyclic-module-records
[ECMAScript Module Loader]: esm.md#modules-ecmascript-modules
[Evaluate() concrete method]: https://tc39.es/ecma262/#sec-moduleevaluation
[GetModuleNamespace]: https://tc39.es/ecma262/#sec-getmodulenamespace
[HostResolveImportedModule]: https://tc39.es/ecma262/#sec-hostresolveimportedmodule
[Link() concrete method]: https://tc39.es/ecma262/#sec-moduledeclarationlinking
[Module Record]: https://www.ecma-international.org/ecma-262/#sec-abstract-module-records
[Source Text Module Record]: https://tc39.es/ecma262/#sec-source-text-module-records
[Synthetic Module Record]: https://heycam.github.io/webidl/#synthetic-module-records
[V8 Embedder's Guide]: https://v8.dev/docs/embed#contexts
[`ERR_VM_DYNAMIC_IMPORT_CALLBACK_MISSING`]: errors.md#err_vm_dynamic_import_callback_missing
[`ERR_VM_MODULE_STATUS`]: errors.md#err_vm_module_status
[`Error`]: errors.md#class-error
[`URL`]: url.md#class-url
[`eval()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval
[`optionsExpression`]: https://tc39.es/proposal-import-assertions/#sec-evaluate-import-call
[`script.runInContext()`]: #scriptrunincontextcontextifiedobject-options
[`script.runInThisContext()`]: #scriptruninthiscontextoptions
[`url.origin`]: url.md#urlorigin
[`vm.createContext()`]: #vmcreatecontextcontextobject-options
[`vm.runInContext()`]: #vmrunincontextcode-contextifiedobject-options
[`vm.runInThisContext()`]: #vmruninthiscontextcode-options
[contextified]: #what-does-it-mean-to-contextify-an-object
[global object]: https://es5.github.io/#x15.1
[indirect `eval()` call]: https://es5.github.io/#x10.4.2
[origin]: https://developer.mozilla.org/en-US/docs/Glossary/Origin
