# 测试运行程序

<!--introduced_in=v18.0.0-->

> 稳定性： 1 - 实验

<!-- source_link=lib/test.js -->

这`node:test`模块有助于创建 JavaScript 测试
报告结果[水龙头][TAP]格式。要访问它：

```mjs
import test from 'node:test';
```

```cjs
const test = require('node:test');
```

此模块仅在`node:`方案。以下不会
工作：

```mjs
import test from 'test';
```

```cjs
const test = require('test');
```

通过`test`模块由单个函数组成，该函数是
以以下三种方式之一进行处理：

1.  一个同步函数，如果它引发异常，则认为该函数失败，
    否则被视为通过。
2.  返回`Promise`如果
    `Promise`拒绝，如果`Promise`解决。
3.  接收回调函数的函数。如果回调收到任何
    真实值作为其第一个参数，测试被认为是失败的。如果
    falsy 值作为第一个参数传递给回调，测试为
    考虑通过。如果测试函数收到回调函数和
    还返回一个`Promise`，测试将失败。

下面的示例演示如何使用
`test`模块。

```js
test('synchronous passing test', (t) => {
  // This test passes because it does not throw an exception.
  assert.strictEqual(1, 1);
});

test('synchronous failing test', (t) => {
  // This test fails because it throws an exception.
  assert.strictEqual(1, 2);
});

test('asynchronous passing test', async (t) => {
  // This test passes because the Promise returned by the async
  // function is not rejected.
  assert.strictEqual(1, 1);
});

test('asynchronous failing test', async (t) => {
  // This test fails because the Promise returned by the async
  // function is rejected.
  assert.strictEqual(1, 2);
});

test('failing test using Promises', (t) => {
  // Promises can be used directly as well.
  return new Promise((resolve, reject) => {
    setImmediate(() => {
      reject(new Error('this will cause the test to fail'));
    });
  });
});

test('callback passing test', (t, done) => {
  // done() is the callback function. When the setImmediate() runs, it invokes
  // done() with no arguments.
  setImmediate(done);
});

test('callback failing test', (t, done) => {
  // When the setImmediate() runs, done() is invoked with an Error object and
  // the test fails.
  setImmediate(() => {
    done(new Error('callback failure'));
  });
});
```

当测试文件执行时，TAP 将写入节点的标准输出.js
过程。此输出可由任何了解
TAP 格式。如果任何测试失败，则进程退出代码设置为`1`.

## 子测试

测试上下文的`test()`方法允许创建子测试。此方法
行为与顶层相同`test()`功能。以下示例
演示如何创建具有两个子测试的顶级测试。

```js
test('top level test', async (t) => {
  await t.test('subtest 1', (t) => {
    assert.strictEqual(1, 1);
  });

  await t.test('subtest 2', (t) => {
    assert.strictEqual(2, 2);
  });
});
```

在此示例中，`await`用于确保两个子测试都已完成。
这是必要的，因为父测试不会等待其子测试
完成。当其父项完成时仍未完成的任何子测试
被取消并被视为失败。任何子测试失败都会导致父级
测试失败。

## 跳过测试

通过`skip`选项到测试，或由
调用测试上下文的`skip()`方法。这两个选项都支持
包括一条显示在 TAP 输出中的消息，如
以下示例。

```js
// The skip option is used, but no message is provided.
test('skip option', { skip: true }, (t) => {
  // This code is never executed.
});

// The skip option is used, and a message is provided.
test('skip option with message', { skip: 'this is skipped' }, (t) => {
  // This code is never executed.
});

test('skip() method', (t) => {
  // Make sure to return here as well if the test contains additional logic.
  t.skip();
});

test('skip() method with message', (t) => {
  // Make sure to return here as well if the test contains additional logic.
  t.skip('this is skipped');
});
```

## `describe`/`it`语法

运行测试也可以使用`describe`申报套房
和`it`以声明测试。
套件用于将相关测试组织和分组在一起。
`it`是 的别名`test`，除非没有通过测试上下文，
因为嵌套是使用套件完成的。

```js
describe('A thing', () => {
  it('should work', () => {
    assert.strictEqual(1, 1);
  });

  it('should be ok', () => {
    assert.strictEqual(2, 2);
  });

  describe('a nested thing', () => {
    it('should work', () => {
      assert.strictEqual(3, 3);
    });
  });
});
```

`describe`和`it`从`node:test`模块。

```mjs
import { describe, it } from 'node:test';
```

```cjs
const { describe, it } = require('node:test');
```

### `only`测试

如果 Node.js 是使用[`--test-only`][--test-only]命令行选项，它是
可以通过以下方法跳过除所选子集之外的所有顶级测试
这`only`选项添加到应运行的测试。当测试与`only`
运行选项集，也运行所有子测试。测试上下文的`runOnly()`
方法可用于在子测试级别实现相同的行为。

```js
// Assume Node.js is run with the --test-only command-line option.
// The 'only' option is set, so this test is run.
test('this test is run', { only: true }, async (t) => {
  // Within this test, all subtests are run by default.
  await t.test('running subtest');

  // The test context can be updated to run subtests with the 'only' option.
  t.runOnly(true);
  await t.test('this subtest is now skipped');
  await t.test('this subtest is run', { only: true });

  // Switch the context back to execute all tests.
  t.runOnly(false);
  await t.test('this subtest is now run');

  // Explicitly do not run these tests.
  await t.test('skipped subtest 3', { only: false });
  await t.test('skipped subtest 4', { skip: true });
});

// The 'only' option is not set, so this test is skipped.
test('this test is not run', () => {
  // This code is not run.
  throw new Error('fail');
});
```

## 无关的异步活动

测试函数执行完成后，TAP结果将尽快输出
尽可能同时保持测试的顺序。但是，这是可能的
用于测试函数生成比测试寿命更长的异步活动
本身。测试运行程序处理此类活动，但不会延迟
报告测试结果以适应它。

在下面的示例中，测试以两个完成`setImmediate()`
操作仍然悬而未决。第一个`setImmediate()`尝试创建
新的子测试。因为父测试已经完成并输出其
结果，新的子测试立即标记为失败，并在
文件的 TAP 输出的顶级。

第二个`setImmediate()`创建一个`uncaughtException`事件。
`uncaughtException`和`unhandledRejection`源自 已完成的事件
测试由`test`模块，并在 中报告为诊断警告
文件的 TAP 输出的顶层。

```js
test('a test that creates asynchronous activity', (t) => {
  setImmediate(() => {
    t.test('subtest that is created too late', (t) => {
      throw new Error('error1');
    });
  });

  setImmediate(() => {
    throw new Error('error2');
  });

  // The test finishes after this line.
});
```

## 从命令行运行测试

节点.js测试运行程序可以从命令行调用，方法是传递
[`--test`][--test]旗：

```bash
node --test
```

默认情况下，Node.js 将以递归方式在当前目录中搜索
与特定命名约定匹配的 JavaScript 源文件。匹配文件
作为测试文件执行。有关预期测试文件命名的详细信息
约定和行为可以在[测试运行程序执行模型][test runner execution model]
部分。

或者，可以提供一个或多个路径作为最终参数
“节点.js命令，如下所示。

```bash
node --test test1.js test2.mjs custom_test_dir/
```

在此示例中，测试运行程序将执行文件`test1.js`和
`test2.mjs`.测试运行程序还将以递归方式搜索
`custom_test_dir/`要执行的测试文件的目录。

### 测试运行程序执行模型

搜索要执行的测试文件时，测试运行程序的行为如下所示：

*   将执行用户显式提供的任何文件。
*   如果用户未显式指定任何路径，则当前工作
    目录以递归方式搜索以下指定文件
    步骤。
*   `node_modules`除非
    用户。
*   如果目录名为`test`遇到，测试运行程序将搜索它
    递归地适用于所有人`.js`,`.cjs`和`.mjs`文件。所有这些文件
    被视为测试文件，不需要匹配特定的命名
    约定详见下文。这是为了适应将
    他们的测试在一个单`test`目录。
*   在所有其他目录中，`.js`,`.cjs`和`.mjs`与 匹配的文件
    以下模式被视为测试文件：
    *   `^test$`- 基名为字符串的文件`'test'`.例子：
        `test.js`,`test.cjs`,`test.mjs`.
    *   `^test-.+`- 基名以字符串开头的文件`'test-'`
        后跟一个或多个字符。例子：`test-example.js`,
        `test-another-example.mjs`.
    *   `.+[\.\-\_]test$`- 基本名称以结尾的文件`.test`,`-test`或
        `_test`，前面有一个或多个字符。例子：`example.test.js`,
        `example-test.cjs`,`example_test.mjs`.
    *   Node 可以理解的其他文件类型.js例如`.node`和`.json`不是
        由测试运行程序自动执行，但如果显式支持
        在命令行上提供。

每个匹配的测试文件都在单独的子进程中执行。如果孩子
进程以退出代码 0 结束，测试被视为通过。
否则，测试将被视为失败。测试文件必须是
由 Node.js可执行，但不需要使用`node:test`模块
内部。

## `test([name][, options][, fn])`

<!-- YAML
added: v18.0.0
changes:
  - version: REPLACEME
    pr-url: https://github.com/nodejs/node/pull/43554
    description: Add a `signal` option.
  - version: v18.7.0
    pr-url: https://github.com/nodejs/node/pull/43505
    description: Add a `timeout` option.
-->

*   `name`{字符串}测试的名称，在报告测试时显示
    结果。**违约：**这`name`的属性`fn`或`'<anonymous>'`如果`fn`
    没有名称。
*   `options`{对象}测试的配置选项。以下
    支持的属性：
    *   `concurrency`{数字|布尔值}如果提供了数字，
        然后，许多测试将并行运行。
        如果属实，它将运行（CPU内核数 - 1）
        并行测试。
        对于子测试，它将是`Infinity`并行测试。
        如果是虚假的，它一次只能运行一个测试。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `false`.
    *   `only`{布尔值}如果属实，并且测试上下文配置为运行
        `only`测试，然后将运行此测试。否则，将跳过测试。
        **违约：** `false`.
    *   `signal`{中止信号}允许中止正在进行的测试。
    *   `skip`{布尔值|字符串}如果属实，则跳过测试。如果字符串是
        提供，则该字符串显示在测试结果中作为原因
        跳过测试。**违约：** `false`.
    *   `todo`{布尔值|字符串}如果属实，则测试标记为`TODO`.如果字符串
        ，则该字符串显示在测试结果中作为原因
        测试是`TODO`.**违约：** `false`.
    *   `timeout`{数字}测试将在几毫秒后失败。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.
*   `fn`{功能|异步函数} 被测函数。第一个参数
    到这个函数是一个[`TestContext`][TestContext]对象。如果测试使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   返回：{承诺} 解析方式`undefined`测试完成后。

这`test()`函数是从`test`模块。每
调用此函数会导致在TAP中创建测试点
输出。

这`TestContext`传递给 的对象`fn`参数可用于执行
与当前测试相关的操作。示例包括跳过测试、添加
其他 TAP 诊断信息，或创建子测试。

`test()`返回`Promise`一旦测试完成，就会解决。回归
对于顶级测试，通常可以丢弃值。但是，返回值
应使用子测试来防止父测试首先完成
并取消子测试，如以下示例所示。

```js
test('top level test', async (t) => {
  // The setTimeout() in the following subtest would cause it to outlive its
  // parent test if 'await' is removed on the next line. Once the parent test
  // completes, it will cancel any outstanding subtests.
  await t.test('longer running subtest', async (t) => {
    return new Promise((resolve, reject) => {
      setTimeout(resolve, 1000);
    });
  });
});
```

这`timeout`选项可用于使测试失败，如果它花费的时间超过
`timeout`毫秒级完成。但是，它不是一种可靠的机制
取消测试，因为正在运行的测试可能会阻塞应用程序线程，并且
从而防止预定的取消。

## `describe([name][, options][, fn])`

*   `name`{字符串}套件的名称，在报告测试时显示
    结果。**违约：**这`name`的属性`fn`或`'<anonymous>'`如果`fn`
    没有名称。
*   `options`{对象}套件的配置选项。
    支持与`test([name][, options][, fn])`.
*   `fn`{功能|AsyncFunction} Suite 下的函数
    声明所有子测试和子建议。
    此函数的第一个参数是[`SuiteContext`][SuiteContext]对象。
    **违约：**无操作函数。
*   返回：`undefined`.

这`describe()`从`node:test`模块。每
调用此函数会导致创建子测试
和 TAP 输出中的测试点。
调用顶级后`describe`功能
将执行所有顶级测试和套件。

## `describe.skip([name][, options][, fn])`

跳过套件的简写，与[`describe([name], { skip: true }[, fn])`][describe options].

## `describe.todo([name][, options][, fn])`

将套件标记为`TODO`，与 相同
[`describe([name], { todo: true }[, fn])`][describe options].

## `it([name][, options][, fn])`

*   `name`{字符串}测试的名称，在报告测试时显示
    结果。**违约：**这`name`的属性`fn`或`'<anonymous>'`如果`fn`
    没有名称。
*   `options`{对象}套件的配置选项。
    支持与`test([name][, options][, fn])`.
*   `fn`{功能|异步函数} 被测函数。
    如果测试使用回调，则回调函数作为参数传递。
    **违约：**无操作函数。
*   返回：`undefined`.

这`it()`函数是从`node:test`模块。
每次调用此函数都会在
抽头输出。

## `it.skip([name][, options][, fn])`

跳过测试的简写，
与[`it([name], { skip: true }[, fn])`][it options].

## `it.todo([name][, options][, fn])`

将测试标记为`TODO`,
与[`it([name], { todo: true }[, fn])`][it options].

### `before([, fn][, options])`

<!-- YAML
added: REPLACEME
-->

*   `fn`{功能|AsyncFunction} 钩子函数。
    如果钩子使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   `options`{对象}挂钩的配置选项。以下
    支持的属性：
    *   `signal`{中止信号}允许中止正在进行的挂钩。
    *   `timeout`{数字}钩子将在几毫秒后失效。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.

此函数用于在运行套件之前创建运行钩子。

```js
describe('tests', async () => {
  before(() => console.log('about to run some test'));
  it('is a subtest', () => {
    assert.ok('some relevant assertion here');
  });
});
```

### `after([, fn][, options])`

<!-- YAML
added: REPLACEME
-->

*   `fn`{功能|AsyncFunction} 钩子函数。
    如果钩子使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   `options`{对象}挂钩的配置选项。以下
    支持的属性：
    *   `signal`{中止信号}允许中止正在进行的挂钩。
    *   `timeout`{数字}钩子将在几毫秒后失效。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.

此函数用于创建在运行套件后运行的钩子。

```js
describe('tests', async () => {
  after(() => console.log('finished running tests'));
  it('is a subtest', () => {
    assert.ok('some relevant assertion here');
  });
});
```

### `beforeEach([, fn][, options])`

<!-- YAML
added: REPLACEME
-->

*   `fn`{功能|AsyncFunction} 钩子函数。
    如果钩子使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   `options`{对象}挂钩的配置选项。以下
    支持的属性：
    *   `signal`{中止信号}允许中止正在进行的挂钩。
    *   `timeout`{数字}钩子将在几毫秒后失效。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.

此函数用于创建运行钩子
在当前套件的每个子测试之前。

```js
describe('tests', async () => {
  beforeEach(() => t.diagnostics('about to run a test'));
  it('is a subtest', () => {
    assert.ok('some relevant assertion here');
  });
});
```

### `afterEach([, fn][, options])`

<!-- YAML
added: REPLACEME
-->

*   `fn`{功能|AsyncFunction} 钩子函数。
    如果钩子使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   `options`{对象}挂钩的配置选项。以下
    支持的属性：
    *   `signal`{中止信号}允许中止正在进行的挂钩。
    *   `timeout`{数字}钩子将在几毫秒后失效。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.

此函数用于创建运行钩子
在当前测试的每个子测试之后。

```js
describe('tests', async () => {
  afterEach(() => t.diagnostics('about to run a test'));
  it('is a subtest', () => {
    assert.ok('some relevant assertion here');
  });
});
```

## 类：`TestContext`

<!-- YAML
added: v18.0.0
-->

的实例`TestContext`被传递给每个测试函数，以便
与测试运行程序交互。但是，`TestContext`构造函数不是
作为 API 的一部分公开。

### `context.beforeEach([, fn][, options])`

<!-- YAML
added: REPLACEME
-->

*   `fn`{功能|AsyncFunction} 钩子函数。第一个参数
    到这个函数是一个[`TestContext`][TestContext]对象。如果钩子使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   `options`{对象}挂钩的配置选项。以下
    支持的属性：
    *   `signal`{中止信号}允许中止正在进行的挂钩。
    *   `timeout`{数字}钩子将在几毫秒后失效。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.

此函数用于创建运行钩子
在当前测试的每个子测试之前。

```js
test('top level test', async (t) => {
  t.beforeEach((t) => t.diagnostics(`about to run ${t.name}`));
  await t.test(
    'This is a subtest',
    (t) => {
      assert.ok('some relevant assertion here');
    }
  );
});
```

### `context.afterEach([, fn][, options])`

<!-- YAML
added: REPLACEME
-->

*   `fn`{功能|AsyncFunction} 钩子函数。第一个参数
    到这个函数是一个[`TestContext`][TestContext]对象。如果钩子使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   `options`{对象}挂钩的配置选项。以下
    支持的属性：
    *   `signal`{中止信号}允许中止正在进行的挂钩。
    *   `timeout`{数字}钩子将在几毫秒后失效。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.

此函数用于创建运行钩子
在当前测试的每个子测试之后。

```js
test('top level test', async (t) => {
  t.afterEach((t) => t.diagnostics(`finished running ${t.name}`));
  await t.test(
    'This is a subtest',
    (t) => {
      assert.ok('some relevant assertion here');
    }
  );
});
```

### `context.diagnostic(message)`

<!-- YAML
added: v18.0.0
-->

*   `message`{字符串}要显示为TAP诊断的消息。

此函数用于将 TAP 诊断写入输出。任何诊断
信息包含在测试结果的末尾。此函数执行
不返回值。

```js
test('top level test', (t) => {
  t.diagnostic('A diagnostic message');
});
```

### `context.name`

<!-- YAML
added: REPLACEME
-->

测试的名称。

### `context.runOnly(shouldRunOnlyTests)`

<!-- YAML
added: v18.0.0
-->

*   `shouldRunOnlyTests`{布尔值}是否运行`only`测试。

如果`shouldRunOnlyTests`是实话，测试上下文将只运行以下测试
有`only`选项集。否则，将运行所有测试。如果节点.js不是
从[`--test-only`][--test-only]命令行选项，此函数是一个
无操作。

```js
test('top level test', (t) => {
  // The test context can be set to run subtests with the 'only' option.
  t.runOnly(true);
  return Promise.all([
    t.test('this subtest is now skipped'),
    t.test('this subtest is run', { only: true }),
  ]);
});
```

### `context.signal`

<!-- YAML
added: v18.7.0
-->

*   {中止信号}可用于在测试已
    中止。

```js
test('top level test', async (t) => {
  await fetch('some/uri', { signal: t.signal });
});
```

### `context.skip([message])`

<!-- YAML
added: v18.0.0
-->

*   `message`{字符串}要在 TAP 输出中显示的可选跳过消息。

此函数使测试的输出将测试指示为已跳过。如果
`message`提供时，它包含在 TAP 输出中。叫`skip()`做
不终止测试函数的执行。此函数不返回
价值。

```js
test('top level test', (t) => {
  // Make sure to return here as well if the test contains additional logic.
  t.skip('this is skipped');
});
```

### `context.todo([message])`

<!-- YAML
added: v18.0.0
-->

*   `message`{字符串}自选`TODO`要在 TAP 输出中显示的消息。

此函数添加一个`TODO`指令到测试的输出。如果`message`是
提供，它包含在TAP输出中。叫`todo()`不终止
执行测试函数。此函数不返回值。

```js
test('top level test', (t) => {
  // This test is marked as `TODO`
  t.todo('this is a todo');
});
```

### `context.test([name][, options][, fn])`

<!-- YAML
added: v18.0.0
changes:
  - version: REPLACEME
    pr-url: https://github.com/nodejs/node/pull/43554
    description: Add a `signal` option.
  - version: v18.7.0
    pr-url: https://github.com/nodejs/node/pull/43505
    description: Add a `timeout` option.
-->

*   `name`{字符串}子测试的名称，在报告时显示
    测试结果。**违约：**这`name`的属性`fn`或`'<anonymous>'`如果
    `fn`没有名称。
*   `options`{对象}子测试的配置选项。以下
    支持的属性：
    *   `concurrency`{数字}可以同时运行的测试数。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `1`.
    *   `only`{布尔值}如果属实，并且测试上下文配置为运行
        `only`测试，然后将运行此测试。否则，将跳过测试。
        **违约：** `false`.
    *   `signal`{中止信号}允许中止正在进行的测试。
    *   `skip`{布尔值|字符串}如果属实，则跳过测试。如果字符串是
        提供，则该字符串显示在测试结果中作为原因
        跳过测试。**违约：** `false`.
    *   `todo`{布尔值|字符串}如果属实，则测试标记为`TODO`.如果字符串
        ，则该字符串显示在测试结果中作为原因
        测试是`TODO`.**违约：** `false`.
    *   `timeout`{数字}测试将在几毫秒后失败。
        如果未指定，则子测试将从其父级继承此值。
        **违约：** `Infinity`.
*   `fn`{功能|异步函数} 被测函数。第一个参数
    到这个函数是一个[`TestContext`][TestContext]对象。如果测试使用回调，
    回调函数作为第二个参数传递。**违约：**无操作
    功能。
*   返回：{承诺} 解析方式`undefined`测试完成后。

此函数用于在当前测试下创建子测试。此函数
以与顶级相同的方式运行[`test()`][test()]功能。

```js
test('top level test', async (t) => {
  await t.test(
    'This is a subtest',
    { only: false, skip: false, concurrency: 1, todo: false },
    (t) => {
      assert.ok('some relevant assertion here');
    }
  );
});
```

## 类：`SuiteContext`

<!-- YAML
added: v18.7.0
-->

的实例`SuiteContext`被传递给每个套件功能，以便
与测试运行程序交互。但是，`SuiteContext`构造函数不是
作为 API 的一部分公开。

### `context.name`

<!-- YAML
added: REPLACEME
-->

套件的名称。

### `context.signal`

<!-- YAML
added: v18.7.0
-->

*   {中止信号}可用于在测试已
    中止。

[TAP]: https://testanything.org/

[`--test-only`]: cli.md#--test-only

[`--test`]: cli.md#--test

[`SuiteContext`]: #class-suitecontext

[`TestContext`]: #class-testcontext

[`test()`]: #testname-options-fn

[describe options]: #describename-options-fn

[it options]: #testname-options-fn

[test runner execution model]: #test-runner-execution-model
