# Timers

<!--introduced_in=v0.10.0-->

> Stability: 2 - Stable

<!-- source_link=lib/timers.js -->

`timer` 模块公开了一个全局 API，用于调度要在未来某个时间段调用的函数。因为定时器函数是全局的，所以不需要调用 `require('node:timers')` 来使用 API.

Node.js 中的计时器函数实现了与 Web 浏览器提供的计时器 API 类似的 API，但使用了围绕 Node.js [事件循环] [] 构建的不同内部实现.

## Class: `Immediate`

该对象在内部创建，并从 [`setImmediate()`][] 返回。它可以传递给 [`clearImmediate()`][] 以取消计划的操作.

默认情况下，当调度立即数时，只要立即数处于活动状态，Node.js 事件循环就会继续运行。 [`setImmediate()`][] 返回的 `Immediate` 对象同时导出 `immediate.ref()` 和 `immediate.unref()` 函数，可用于控制此默认行为.

### `immediate.hasRef()`

<!-- YAML
added: v11.0.0
-->

* Returns: {boolean}

如果为 true，`Immediate` 对象将保持 Node.js 事件循环处于活动状态.

### `immediate.ref()`

<!-- YAML
added: v9.7.0
-->

* Returns: {Immediate} a reference to `immediate`

调用时，只要“立即”处于活动状态，就请求 Node.js 事件循环_not_退出。多次调用 `immediate.ref()` 将无效.

默认情况下，所有 `Immediate` 对象都是“引用”的，因此通常不需要调用 `immediate.ref()`，除非之前调用过 `immediate.unref()`.

### `immediate.unref()`

<!-- YAML
added: v9.7.0
-->

* Returns: {Immediate} a reference to `immediate`

调用时，活动的“立即”对象将不需要 Node.js 事件循环保持活动状态。如果没有其他活动保持事件循环运行，则进程可能会在调用“立即”对象的回调之前退出。多次调用 `immediate.unref()` 将无效.

## Class: `Timeout`

该对象在内部创建，并从 [`setTimeout()`][] 和 [`setInterval()`][] 返回。它可以传递给 [`clearTimeout()`][] 或 [`clearInterval()`][] 以取消计划的操作.

默认情况下，当使用 [`setTimeout()`][] 或 [`setInterval()`][] 安排计时器时，只要计时器处于活动状态，Node.js 事件循环就会继续运行。这些函数返回的每个 `Timeout` 对象都导出 `timeout.ref()` 和 `timeout.unref()` 函数，可用于控制此默认行为.

### `timeout.close()`

<!-- YAML
added: v0.9.1
-->

> Stability: 3 - Legacy: Use [`clearTimeout()`][] instead.

* Returns: {Timeout} a reference to `timeout`

取消超时.

### `timeout.hasRef()`

<!-- YAML
added: v11.0.0
-->

* Returns: {boolean}

如果为 true，`Timeout` 对象将保持 Node.js 事件循环处于活动状态.

### `timeout.ref()`

<!-- YAML
added: v0.9.1
-->

* Returns: {Timeout} a reference to `timeout`

调用时，只要“超时”处于活动状态，就请求 Node.js 事件循环_not_退出。多次调用 `timeout.ref()` 将无效.

默认情况下，所有 `Timeout` 对象都是“引用”的，因此通常不需要调用 `timeout.ref()`，除非之前调用过 `timeout.unref()`.

### `timeout.refresh()`

<!-- YAML
added: v10.2.0
-->

* Returns: {Timeout} a reference to `timeout`

将计时器的开始时间设置为当前时间，并重新安排计时器以在先前指定的调整为当前时间的持续时间调用其回调。这对于在不分配新 JavaScript 对象的情况下刷新计时器很有用.

在已经调用其回调的计时器上使用它将重新激活计时器.

### `timeout.unref()`

<!-- YAML
added: v0.9.1
-->

* Returns: {Timeout} a reference to `timeout`

调用时，活动的“超时”对象将不需要 Node.js 事件循环保持活动状态。如果没有其他活动保持事件循环运行, 该进程可能会在调用 `Timeout` 对象的回调之前退出。多次调用 `timeout.unref()` 将无效.

### `timeout[Symbol.toPrimitive]()`

<!-- YAML
added:
  - v14.9.0
  - v12.19.0
-->

* Returns: {integer} a number that can be used to reference this `timeout`

将“超时”强制为原语。该原语可用于清除“超时”。该原语只能在创建超时的同一线程中使用。因此，要跨 [`worker_threads`][] 使用它，必须首先将其传递给正确的线程。这允许增强与浏览器 `setTimeout()` 和 `setInterval()` 实现的兼容性.

## Scheduling timers

Node.js 中的计时器是一个内部构造，它在一段时间后调用给定的函数。何时调用计时器的函数取决于用于创建计时器的方法以及 Node.js 事件循环正在执行的其他工作.

### `setImmediate(callback[, ...args])`

<!-- YAML
added: v0.9.1
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

* `callback` {Function} The function to call at the end of this turn of
  the Node.js [Event Loop][]
* `...args` {any} Optional arguments to pass when the `callback` is called.
* Returns: {Immediate} for use with [`clearImmediate()`][]

在 I/O 事件的回调之后安排“立即”执行“回调”.

当多次调用 `setImmediate()` 时，`callback` 函数会按照它们创建的顺序排队等待执行。每次事件循环迭代都会处理整个回调队列。如果即时计时器从正在执行的回调中排队，则在下一次事件循环迭代之前不会触发该计时器.

如果 `callback` 不是函数，则会抛出 [`TypeError`][].

此方法具有可使用 [`timersPromises.setImmediate()`][] 的承诺的自定义变体.

### `setInterval(callback[, delay[, ...args]])`

<!-- YAML
added: v0.0.1
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

* `callback` {Function} The function to call when the timer elapses.
* `delay` {number} The number of milliseconds to wait before calling the
  `callback`. **Default:** `1`.
* `...args` {any} Optional arguments to pass when the `callback` is called.
* Returns: {Timeout} for use with [`clearInterval()`][]

安排每隔 `delay` 毫秒重复执行`callback`.

当 `delay` 大于 `2147483647` 或小于 `1` 时，`delay` 将被设置为 `1`。非整数延迟被截断为整数.

如果 `callback` 不是函数，则会抛出 [`TypeError`][].

此方法具有可使用 [`timersPromises.setInterval()`][] 的承诺的自定义变体.

### `setTimeout(callback[, delay[, ...args]])`

<!-- YAML
added: v0.0.1
changes:
  - version: v18.0.0
    pr-url: https://github.com/nodejs/node/pull/41678
    description: Passing an invalid callback to the `callback` argument
                 now throws `ERR_INVALID_ARG_TYPE` instead of
                 `ERR_INVALID_CALLBACK`.
-->

* `callback` {Function} The function to call when the timer elapses.
* `delay` {number} The number of milliseconds to wait before calling the
  `callback`. **Default:** `1`.
* `...args` {any} Optional arguments to pass when the `callback` is called.
* Returns: {Timeout} for use with [`clearTimeout()`][]

安排在 `delay` 毫秒后执行一次性`callback`.

`callback` 可能不会在精确的 `delay` 毫秒内被调用.
Node.js 不保证回调何时触发的确切时间，也不保证它们的顺序。回调将被调用尽可能接近指定的时间.

当 `delay` 大于 `2147483647` 或小于 `1` 时，`delay` 将被设置为 `1`。非整数延迟被截断为整数.

如果 `callback` 不是函数，则会抛出 [`TypeError`][].

此方法具有可使用 [`timersPromises.setTimeout()`][] 的承诺的自定义变体.

## Cancelling timers

[`setImmediate()`][]、[`setInterval()`][] 和 [`setTimeout()`][] 方法均返回表示计划计时器的对象。这些可用于取消计时器并防止其触发.

对于 [`setImmediate()`][] 和 [`setTimeout()`][] 的承诺变体，可以使用 [`AbortController`][] 来取消计时器。取消后，返回的 Promise 将被拒绝并返回 `'AbortError'`.

For `setImmediate()`:

```js
const { setImmediate: setImmediatePromise } = require('node:timers/promises');

const ac = new AbortController();
const signal = ac.signal;

setImmediatePromise('foobar', { signal })
  .then(console.log)
  .catch((err) => {
    if (err.name === 'AbortError')
      console.log('The immediate was aborted');
  });

ac.abort();
```

For `setTimeout()`:

```js
const { setTimeout: setTimeoutPromise } = require('node:timers/promises');

const ac = new AbortController();
const signal = ac.signal;

setTimeoutPromise(1000, 'foobar', { signal })
  .then(console.log)
  .catch((err) => {
    if (err.name === 'AbortError')
      console.log('The timeout was aborted');
  });

ac.abort();
```

### `clearImmediate(immediate)`

<!-- YAML
added: v0.9.1
-->

* `immediate` {Immediate} An `Immediate` object as returned by
  [`setImmediate()`][].

取消由 [`setImmediate()`][] 创建的 `Immediate` 对象.

### `clearInterval(timeout)`

<!-- YAML
added: v0.0.1
-->

* `timeout` {Timeout|string|number} A `Timeout` object as returned by [`setInterval()`][]
  or the [primitive][] of the `Timeout` object as a string or a number.

取消由 [`setInterval()`][] 创建的 `Timeout` 对象.

### `clearTimeout(timeout)`

<!-- YAML
added: v0.0.1
-->

* `timeout` {Timeout|string|number} A `Timeout` object as returned by [`setTimeout()`][]
  or the [primitive][] of the `Timeout` object as a string or a number.

取消由 [`setTimeout()`][] 创建的 `Timeout` 对象.

## Timers Promises API

<!-- YAML
added: v15.0.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/38112
    description: Graduated from experimental.
-->

`timers/promises` API 提供了一组替代的定时器函数，它们返回 `Promise` 对象。该 API 可通过 `require('node:timers/promises')` 访问.

```mjs
import {
  setTimeout,
  setImmediate,
  setInterval,
} from 'timers/promises';
```

```cjs
const {
  setTimeout,
  setImmediate,
  setInterval,
} = require('node:timers/promises');
```

### `timersPromises.setTimeout([delay[, value[, options]]])`

<!-- YAML
added: v15.0.0
-->

* `delay` {number} The number of milliseconds to wait before fulfilling the
  promise. **Default:** `1`.
* `value` {any} A value with which the promise is fulfilled.
* `options` {Object}
  * `ref` {boolean} Set to `false` to indicate that the scheduled `Timeout`
    should not require the Node.js event loop to remain active.
    **Default:** `true`.
  * `signal` {AbortSignal} An optional `AbortSignal` that can be used to
    cancel the scheduled `Timeout`.

```mjs
import {
  setTimeout,
} from 'timers/promises';

const res = await setTimeout(100, 'result');

console.log(res);  // Prints 'result'
```

```cjs
const {
  setTimeout,
} = require('node:timers/promises');

setTimeout(100, 'result').then((res) => {
  console.log(res);  // Prints 'result'
});
```

### `timersPromises.setImmediate([value[, options]])`

<!-- YAML
added: v15.0.0
-->

* `value` {any} A value with which the promise is fulfilled.
* `options` {Object}
  * `ref` {boolean} Set to `false` to indicate that the scheduled `Immediate`
    should not require the Node.js event loop to remain active.
    **Default:** `true`.
  * `signal` {AbortSignal} An optional `AbortSignal` that can be used to
    cancel the scheduled `Immediate`.

```mjs
import {
  setImmediate,
} from 'timers/promises';

const res = await setImmediate('result');

console.log(res);  // Prints 'result'
```

```cjs
const {
  setImmediate,
} = require('node:timers/promises');

setImmediate('result').then((res) => {
  console.log(res);  // Prints 'result'
});
```

### `timersPromises.setInterval([delay[, value[, options]]])`

<!-- YAML
added: v15.9.0
-->

返回一个异步迭代器，它以“延迟”毫秒的间隔生成值.

* `delay` {number} The number of milliseconds to wait between iterations.
  **Default:** `1`.
* `value` {any} A value with which the iterator returns.
* `options` {Object}
  * `ref` {boolean} Set to `false` to indicate that the scheduled `Timeout`
    between iterations should not require the Node.js event loop to
    remain active.
    **Default:** `true`.
  * `signal` {AbortSignal} An optional `AbortSignal` that can be used to
    cancel the scheduled `Timeout` between operations.

```mjs
import {
  setInterval,
} from 'timers/promises';

const interval = 100;
for await (const startTime of setInterval(interval, Date.now())) {
  const now = Date.now();
  console.log(now);
  if ((now - startTime) > 1000)
    break;
}
console.log(Date.now());
```

```cjs
const {
  setInterval,
} = require('node:timers/promises');
const interval = 100;

(async function() {
  for await (const startTime of setInterval(interval, Date.now())) {
    const now = Date.now();
    console.log(now);
    if ((now - startTime) > 1000)
      break;
  }
  console.log(Date.now());
})();
```

### `timersPromises.scheduler.wait(delay[, options])`

<!-- YAML
added:
  - v17.3.0
  - v16.14.0
-->

> Stability: 1 - Experimental

* `delay` {number} The number of milliseconds to wait before resolving the
  promise.
* `options` {Object}
  * `signal` {AbortSignal} An optional `AbortSignal` that can be used to
    cancel waiting.
* Returns: {Promise}

[Scheduling APIs][] 草案规范定义的实验性 API，正在开发为标准 Web 平台 API.

调用 `timersPromises.scheduler.wait(delay, options)` 大致等同于调用 `timersPromises.setTimeout(delay, undefined, options)`，只是不支持 `ref` 选项.

```mjs
import { scheduler } from 'node:timers/promises';

await scheduler.wait(1000); // Wait one second before continuing
```

### `timersPromises.scheduler.yield()`

<!-- YAML
added:
  - v17.3.0
  - v16.14.0
-->

> Stability: 1 - Experimental

* Returns: {Promise}

[Scheduling APIs][] 草案规范定义的实验性 API，正在开发为标准 Web 平台 API.

调用 `timersPromises.scheduler.yield()` 等同于调用 `timersPromises.setImmediate()` 不带参数.

[Event Loop]: https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#setimmediate-vs-settimeout
[Scheduling APIs]: https://github.com/WICG/scheduling-apis
[`AbortController`]: globals.md#class-abortcontroller
[`TypeError`]: errors.md#class-typeerror
[`clearImmediate()`]: #clearimmediateimmediate
[`clearInterval()`]: #clearintervaltimeout
[`clearTimeout()`]: #cleartimeouttimeout
[`setImmediate()`]: #setimmediatecallback-args
[`setInterval()`]: #setintervalcallback-delay-args
[`setTimeout()`]: #settimeoutcallback-delay-args
[`timersPromises.setImmediate()`]: #timerspromisessetimmediatevalue-options
[`timersPromises.setInterval()`]: #timerspromisessetintervaldelay-value-options
[`timersPromises.setTimeout()`]: #timerspromisessettimeoutdelay-value-options
[`worker_threads`]: worker_threads.md
[primitive]: #timeoutsymboltoprimitive
