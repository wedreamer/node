# 事件

<!--introduced_in=v0.10.0-->

> 稳定性： 2 - 稳定

<!--type=module-->

<!-- source_link=lib/events.js -->

Node.js核心API的大部分都是围绕惯用异步构建的。
事件驱动的体系结构，其中某些类型的对象（称为“发射器”）
发出导致`Function`要调用的对象（“侦听器”）。

例如：一个[`net.Server`][net.Server]对象每次对等体时发出一个事件
连接到它;一个[`fs.ReadStream`][fs.ReadStream]打开文件时发出事件;
一个[流][stream]每当有数据可供读取时，就会发出一个事件。

发出事件的所有对象都是`EventEmitter`类。这些
对象公开`eventEmitter.on()`允许一个或多个函数
要附加到对象发出的命名事件的函数。通常
事件名称是驼峰大小写的字符串，但任何有效的 JavaScript 属性键
可以使用。

当`EventEmitter`对象发出一个事件，所有附加的函数
到该特定事件称为*同步*.返回的任何值
被调用的侦听器是*忽视*并被丢弃。

以下示例显示了一个简单的`EventEmitter`具有单个实例的实例
听者。这`eventEmitter.on()`方法用于注册侦听器，而
这`eventEmitter.emit()`方法用于触发事件。

```mjs
import { EventEmitter } from 'node:events';

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
myEmitter.emit('event');
```

```cjs
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
myEmitter.emit('event');
```

## 传递参数和`this`给听众

这`eventEmitter.emit()`方法允许任意参数集
传递给侦听器函数。请记住，当
一个普通的监听器函数叫做，标准`this`关键词
有意设置为引用`EventEmitter`实例
附加了侦听器。

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', function(a, b) {
  console.log(a, b, this, this === myEmitter);
  // Prints:
  //   a b MyEmitter {
  //     domain: null,
  //     _events: { event: [Function] },
  //     _eventsCount: 1,
  //     _maxListeners: undefined } true
});
myEmitter.emit('event', 'a', 'b');
```

可以使用ES6箭头函数作为侦听器，但是，在这样做时，
这`this`关键字将不再引用`EventEmitter`实例：

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  console.log(a, b, this);
  // Prints: a b {}
});
myEmitter.emit('event', 'a', 'b');
```

## 异步与同步

这`EventEmitter`按以下顺序同步调用所有侦听器：
他们已经注册。这确保了
事件，并帮助避免争用条件和逻辑错误。在适当的时候，
侦听器函数可以使用
这`setImmediate()`或`process.nextTick()`方法：

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('this happens asynchronously');
  });
});
myEmitter.emit('event', 'a', 'b');
```

## 只处理一次事件

当侦听器使用`eventEmitter.on()`方法，即
调用侦听器*每次*将发出命名事件。

```js
const myEmitter = new MyEmitter();
let m = 0;
myEmitter.on('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
// Prints: 1
myEmitter.emit('event');
// Prints: 2
```

使用`eventEmitter.once()`方法，可以注册一个监听器
对于特定事件，最多调用一次。发出事件后，
侦听器未注册，并且*然后*叫。

```js
const myEmitter = new MyEmitter();
let m = 0;
myEmitter.once('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
// Prints: 1
myEmitter.emit('event');
// Ignored
```

## 错误事件

当`EventEmitter`实例，典型操作是
对于`'error'`要发出的事件。这些被视为特殊情况
在节点.js。

如果`EventEmitter`做*不*至少有一个侦听器注册了
`'error'`事件，以及`'error'`发出事件，抛出错误，a
将打印堆栈跟踪，并且节点.js进程将退出。

```js
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));
// Throws and crashes Node.js
```

要防止节点崩溃.js处理[`domain`][domain]模块可以是
使用。（但请注意，`node:domain`模块已弃用。

作为最佳实践，应始终将侦听器添加到`'error'`事件。

```js
const myEmitter = new MyEmitter();
myEmitter.on('error', (err) => {
  console.error('whoops! there was an error');
});
myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```

可以监控`'error'`事件而不消耗发出的错误
通过使用符号安装侦听器`events.errorMonitor`.

```mjs
import { EventEmitter, errorMonitor } from 'node:events';

const myEmitter = new EventEmitter();
myEmitter.on(errorMonitor, (err) => {
  MyMonitoringTool.log(err);
});
myEmitter.emit('error', new Error('whoops!'));
// Still throws and crashes Node.js
```

```cjs
const { EventEmitter, errorMonitor } = require('node:events');

const myEmitter = new EventEmitter();
myEmitter.on(errorMonitor, (err) => {
  MyMonitoringTool.log(err);
});
myEmitter.emit('error', new Error('whoops!'));
// Still throws and crashes Node.js
```

## 捕获承诺被拒绝的情况

用`async`具有事件处理程序的函数是有问题的，因为它
在抛出异常的情况下，可能导致未处理的拒绝：

```js
const ee = new EventEmitter();
ee.on('something', async (value) => {
  throw new Error('kaboom');
});
```

这`captureRejections`选项`EventEmitter`构造函数或全局
设置更改此行为，安装`.then(undefined, handler)`
上的处理程序`Promise`.此处理程序路由异常
异步到[`Symbol.for('nodejs.rejection')`][rejection]方法
如果有，或者[`'error'`][error]事件处理程序（如果没有）。

```js
const ee1 = new EventEmitter({ captureRejections: true });
ee1.on('something', async (value) => {
  throw new Error('kaboom');
});

ee1.on('error', console.log);

const ee2 = new EventEmitter({ captureRejections: true });
ee2.on('something', async (value) => {
  throw new Error('kaboom');
});

ee2[Symbol.for('nodejs.rejection')] = console.log;
```

设置`events.captureRejections = true`将更改所有默认值
的新实例`EventEmitter`.

```mjs
import { EventEmitter } from 'node:events';

EventEmitter.captureRejections = true;
const ee1 = new EventEmitter();
ee1.on('something', async (value) => {
  throw new Error('kaboom');
});

ee1.on('error', console.log);
```

```cjs
const events = require('node:events');
events.captureRejections = true;
const ee1 = new events.EventEmitter();
ee1.on('something', async (value) => {
  throw new Error('kaboom');
});

ee1.on('error', console.log);
```

这`'error'`由 生成的事件`captureRejections`行为
没有 catch 处理程序来避免无限的错误循环：
建议是**不使用`async`函数为`'error'`事件处理程序**.

## 类：`EventEmitter`

<!-- YAML
added: v0.1.26
changes:
  - version:
     - v13.4.0
     - v12.16.0
    pr-url: https://github.com/nodejs/node/pull/27867
    description: Added captureRejections option.
-->

这`EventEmitter`类由 定义和公开`node:events`模块：

```mjs
import { EventEmitter } from 'node:events';
```

```cjs
const EventEmitter = require('node:events');
```

都`EventEmitter`s 发出事件`'newListener'`当新侦听器是
已添加和`'removeListener'`删除现有侦听器时。

它支持以下选项：

*   `captureRejections`{布尔值}它能
    [自动捕获承诺拒绝][capturerejections].
    **违约：** `false`.

### 事件：`'newListener'`

<!-- YAML
added: v0.1.26
-->

*   `eventName`{字符串|符号}正在侦听的事件的名称
*   `listener`{函数}事件处理程序函数

这`EventEmitter`实例将发出自己的`'newListener'`事件*以前*
侦听器被添加到其内部侦听器数组中。

已注册的侦听器`'newListener'`事件已传递事件
名称和对要添加的侦听器的引用。

在添加侦听器之前触发事件的事实有一个微妙的事实
但重要的副作用：任何*附加*注册到同一个侦听器
`name` *在*这`'newListener'`插入回调*以前*这
正在添加的侦听器。

```js
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
// Only do this once so we don't loop forever
myEmitter.once('newListener', (event, listener) => {
  if (event === 'event') {
    // Insert a new listener in front
    myEmitter.on('event', () => {
      console.log('B');
    });
  }
});
myEmitter.on('event', () => {
  console.log('A');
});
myEmitter.emit('event');
// Prints:
//   B
//   A
```

### 事件：`'removeListener'`

<!-- YAML
added: v0.9.3
changes:
  - version:
    - v6.1.0
    - v4.7.0
    pr-url: https://github.com/nodejs/node/pull/6394
    description: For listeners attached using `.once()`, the `listener` argument
                 now yields the original listener function.
-->

*   `eventName`{字符串|符号}事件名称
*   `listener`{函数}事件处理程序函数

这`'removeListener'`发出事件*后*这`listener`已被删除。

### `emitter.addListener(eventName, listener)`

<!-- YAML
added: v0.1.26
-->

*   `eventName`{字符串|符号}
*   `listener`{函数}

的别名`emitter.on(eventName, listener)`.

### `emitter.emit(eventName[, ...args])`

<!-- YAML
added: v0.1.26
-->

*   `eventName`{字符串|符号}
*   `...args`{任何}
*   返回：{布尔值}

同步调用为名为
`eventName`，按照它们的注册顺序，传递提供的参数
到每个。

返回`true`如果事件有侦听器，`false`否则。

```mjs
import { EventEmitter } from 'node:events';
const myEmitter = new EventEmitter();

// First listener
myEmitter.on('event', function firstListener() {
  console.log('Helloooo! first listener');
});
// Second listener
myEmitter.on('event', function secondListener(arg1, arg2) {
  console.log(`event with parameters ${arg1}, ${arg2} in second listener`);
});
// Third listener
myEmitter.on('event', function thirdListener(...args) {
  const parameters = args.join(', ');
  console.log(`event with parameters ${parameters} in third listener`);
});

console.log(myEmitter.listeners('event'));

myEmitter.emit('event', 1, 2, 3, 4, 5);

// Prints:
// [
//   [Function: firstListener],
//   [Function: secondListener],
//   [Function: thirdListener]
// ]
// Helloooo! first listener
// event with parameters 1, 2 in second listener
// event with parameters 1, 2, 3, 4, 5 in third listener
```

```cjs
const EventEmitter = require('node:events');
const myEmitter = new EventEmitter();

// First listener
myEmitter.on('event', function firstListener() {
  console.log('Helloooo! first listener');
});
// Second listener
myEmitter.on('event', function secondListener(arg1, arg2) {
  console.log(`event with parameters ${arg1}, ${arg2} in second listener`);
});
// Third listener
myEmitter.on('event', function thirdListener(...args) {
  const parameters = args.join(', ');
  console.log(`event with parameters ${parameters} in third listener`);
});

console.log(myEmitter.listeners('event'));

myEmitter.emit('event', 1, 2, 3, 4, 5);

// Prints:
// [
//   [Function: firstListener],
//   [Function: secondListener],
//   [Function: thirdListener]
// ]
// Helloooo! first listener
// event with parameters 1, 2 in second listener
// event with parameters 1, 2, 3, 4, 5 in third listener
```

### `emitter.eventNames()`

<!-- YAML
added: v6.0.0
-->

*   返回： {数组}

返回一个数组，其中列出了发射器已注册的事件
听众。数组中的值为字符串或`Symbol`s.

```mjs
import { EventEmitter } from 'node:events';

const myEE = new EventEmitter();
myEE.on('foo', () => {});
myEE.on('bar', () => {});

const sym = Symbol('symbol');
myEE.on(sym, () => {});

console.log(myEE.eventNames());
// Prints: [ 'foo', 'bar', Symbol(symbol) ]
```

```cjs
const EventEmitter = require('node:events');

const myEE = new EventEmitter();
myEE.on('foo', () => {});
myEE.on('bar', () => {});

const sym = Symbol('symbol');
myEE.on(sym, () => {});

console.log(myEE.eventNames());
// Prints: [ 'foo', 'bar', Symbol(symbol) ]
```

### `emitter.getMaxListeners()`

<!-- YAML
added: v1.0.0
-->

*   返回：{整数}

返回`EventEmitter`这是
设置者[`emitter.setMaxListeners(n)`][emitter.setMaxListeners(n)]或默认为
[`events.defaultMaxListeners`][events.defaultMaxListeners].

### `emitter.listenerCount(eventName)`

<!-- YAML
added: v3.2.0
-->

*   `eventName`{字符串|符号}正在侦听的事件的名称
*   返回：{整数}

返回侦听名为`eventName`.

### `emitter.listeners(eventName)`

<!-- YAML
added: v0.1.26
changes:
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/6881
    description: For listeners attached using `.once()` this returns the
                 original listeners instead of wrapper functions now.
-->

*   `eventName`{字符串|符号}
*   返回： {函数\[]}

返回名为 的事件的侦听器数组的副本`eventName`.

```js
server.on('connection', (stream) => {
  console.log('someone connected!');
});
console.log(util.inspect(server.listeners('connection')));
// Prints: [ [Function] ]
```

### `emitter.off(eventName, listener)`

<!-- YAML
added: v10.0.0
-->

*   `eventName`{字符串|符号}
*   `listener`{函数}
*   返回： {事件发射器}

的别名[`emitter.removeListener()`][emitter.removeListener()].

### `emitter.on(eventName, listener)`

<!-- YAML
added: v0.1.101
-->

*   `eventName`{字符串|符号}事件的名称。
*   `listener`{函数}回调函数
*   返回： {事件发射器}

添加`listener`函数到侦听器数组的末尾，用于
事件命名`eventName`.不进行检查以查看`listener`有
已添加。多个呼叫传递相同的组合`eventName`
和`listener`将导致`listener`正在添加和调用多个
次。

```js
server.on('connection', (stream) => {
  console.log('someone connected!');
});
```

返回对`EventEmitter`，以便可以链接调用。

默认情况下，事件侦听器按其添加顺序调用。这
`emitter.prependListener()`方法可以用作添加
事件侦听器侦听器到侦听器数组的开头。

```js
const myEE = new EventEmitter();
myEE.on('foo', () => console.log('a'));
myEE.prependListener('foo', () => console.log('b'));
myEE.emit('foo');
// Prints:
//   b
//   a
```

### `emitter.once(eventName, listener)`

<!-- YAML
added: v0.3.0
-->

*   `eventName`{字符串|符号}事件的名称。
*   `listener`{函数}回调函数
*   返回： {事件发射器}

添加**一次性** `listener`名为 的事件的函数`eventName`.这
下次`eventName`触发时，将删除此侦听器，然后调用该侦听器。

```js
server.once('connection', (stream) => {
  console.log('Ah, we have our first user!');
});
```

返回对`EventEmitter`，以便可以链接调用。

默认情况下，事件侦听器按其添加顺序调用。这
`emitter.prependOnceListener()`方法可以用作添加
事件侦听器侦听器到侦听器数组的开头。

```js
const myEE = new EventEmitter();
myEE.once('foo', () => console.log('a'));
myEE.prependOnceListener('foo', () => console.log('b'));
myEE.emit('foo');
// Prints:
//   b
//   a
```

### `emitter.prependListener(eventName, listener)`

<!-- YAML
added: v6.0.0
-->

*   `eventName`{字符串|符号}事件的名称。
*   `listener`{函数}回调函数
*   返回： {事件发射器}

添加`listener`函数到*开始*的侦听器数组
事件命名`eventName`.不进行检查以查看`listener`有
已添加。多个呼叫传递相同的组合`eventName`
和`listener`将导致`listener`正在添加和调用多个
次。

```js
server.prependListener('connection', (stream) => {
  console.log('someone connected!');
});
```

返回对`EventEmitter`，以便可以链接调用。

### `emitter.prependOnceListener(eventName, listener)`

<!-- YAML
added: v6.0.0
-->

*   `eventName`{字符串|符号}事件的名称。
*   `listener`{函数}回调函数
*   返回： {事件发射器}

添加**一次性** `listener`名为 的事件的函数`eventName`到
*开始*的侦听器数组。下次`eventName`被触发，这
侦听器被删除，然后调用。

```js
server.prependOnceListener('connection', (stream) => {
  console.log('Ah, we have our first user!');
});
```

返回对`EventEmitter`，以便可以链接调用。

### `emitter.removeAllListeners([eventName])`

<!-- YAML
added: v0.1.26
-->

*   `eventName`{字符串|符号}
*   返回： {事件发射器}

删除所有侦听器或指定侦听器的侦听器`eventName`.

删除在代码中其他位置添加的侦听器是一种不好的做法，
特别是当`EventEmitter`实例是由其他某个人创建的
组件或模块（例如套接字或文件流）。

返回对`EventEmitter`，以便可以链接调用。

### `emitter.removeListener(eventName, listener)`

<!-- YAML
added: v0.1.26
-->

*   `eventName`{字符串|符号}
*   `listener`{函数}
*   返回： {事件发射器}

删除指定的`listener`从名为 的事件的侦听器数组
`eventName`.

```js
const callback = (stream) => {
  console.log('someone connected!');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```

`removeListener()`将最多从 中删除一个侦听器实例
侦听器数组。如果任何单个侦听器已多次添加到
指定侦听器数组`eventName`然后`removeListener()`必须是
调用多次以删除每个实例。

发出事件后，所有附加到该事件的侦听器都将在
发射时间按顺序调用。这意味着任何
`removeListener()`或`removeAllListeners()`调用*后*发射和
*以前*最后一个侦听器完成执行不会从中删除它们
`emit()`正在进行中。后续事件的行为符合预期。

```js
const myEmitter = new MyEmitter();

const callbackA = () => {
  console.log('A');
  myEmitter.removeListener('event', callbackB);
};

const callbackB = () => {
  console.log('B');
};

myEmitter.on('event', callbackA);

myEmitter.on('event', callbackB);

// callbackA removes listener callbackB but it will still be called.
// Internal listener array at time of emit [callbackA, callbackB]
myEmitter.emit('event');
// Prints:
//   A
//   B

// callbackB is now removed.
// Internal listener array [callbackA]
myEmitter.emit('event');
// Prints:
//   A
```

由于侦听器是使用内部数组管理的，因此调用此函数将
更改任何已注册侦听器的位置索引*后*听众
正在删除。这不会影响侦听器的调用顺序，
但这意味着返回的侦听器数组的任何副本由
这`emitter.listeners()`方法需要重新创建。

当单个函数已多次添加为单个函数的处理程序时
事件（如下面的示例所示），`removeListener()`将删除最多
最近添加的实例。在此示例中，`once('ping')`
侦听器被删除：

```js
const ee = new EventEmitter();

function pong() {
  console.log('pong');
}

ee.on('ping', pong);
ee.once('ping', pong);
ee.removeListener('ping', pong);

ee.emit('ping');
ee.emit('ping');
```

返回对`EventEmitter`，以便可以链接调用。

### `emitter.setMaxListeners(n)`

<!-- YAML
added: v0.3.5
-->

*   `n`{整数}
*   返回： {事件发射器}

默认情况下`EventEmitter`s 将打印警告，如果超过`10`侦听器是
为特定事件添加。这是一个有用的默认值，有助于查找
内存泄漏。这`emitter.setMaxListeners()`方法允许限制为
针对此特定进行了修改`EventEmitter`实例。该值可以设置为
`Infinity`（或`0`） 以指示无限数量的侦听器。

返回对`EventEmitter`，以便可以链接调用。

### `emitter.rawListeners(eventName)`

<!-- YAML
added: v9.4.0
-->

*   `eventName`{字符串|符号}
*   返回： {函数\[]}

返回名为 的事件的侦听器数组的副本`eventName`,
包括任何包装器（例如由`.once()`).

```js
const emitter = new EventEmitter();
emitter.once('log', () => console.log('log once'));

// Returns a new Array with a function `onceWrapper` which has a property
// `listener` which contains the original listener bound above
const listeners = emitter.rawListeners('log');
const logFnWrapper = listeners[0];

// Logs "log once" to the console and does not unbind the `once` event
logFnWrapper.listener();

// Logs "log once" to the console and removes the listener
logFnWrapper();

emitter.on('log', () => console.log('log persistently'));
// Will return a new Array with a single function bound by `.on()` above
const newListeners = emitter.rawListeners('log');

// Logs "log persistently" twice
newListeners[0]();
emitter.emit('log');
```

### `emitter[Symbol.for('nodejs.rejection')](err, eventName[, ...args])`

<!-- YAML
added:
 - v13.4.0
 - v12.16.0
changes:
  - version:
    - v17.4.0
    - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41267
    description: No longer experimental.
-->

*   `err`错误
*   `eventName`{字符串|符号}
*   `...args`{任何}

这`Symbol.for('nodejs.rejection')`在以下情况下调用方法：
承诺拒绝发生在发出事件时，并且
[`captureRejections`][capturerejections]在发射器上启用。
可以使用[`events.captureRejectionSymbol`][rejectionsymbol]在
地点`Symbol.for('nodejs.rejection')`.

```mjs
import { EventEmitter, captureRejectionSymbol } from 'node:events';

class MyClass extends EventEmitter {
  constructor() {
    super({ captureRejections: true });
  }

  [captureRejectionSymbol](err, event, ...args) {
    console.log('rejection happened for', event, 'with', err, ...args);
    this.destroy(err);
  }

  destroy(err) {
    // Tear the resource down here.
  }
}
```

```cjs
const { EventEmitter, captureRejectionSymbol } = require('node:events');

class MyClass extends EventEmitter {
  constructor() {
    super({ captureRejections: true });
  }

  [captureRejectionSymbol](err, event, ...args) {
    console.log('rejection happened for', event, 'with', err, ...args);
    this.destroy(err);
  }

  destroy(err) {
    // Tear the resource down here.
  }
}
```

## `events.defaultMaxListeners`

<!-- YAML
added: v0.11.2
-->

默认情况下，最大值为`10`监听器可以注册任何单个
事件。此限制可以针对个人进行更改`EventEmitter`实例
使用[`emitter.setMaxListeners(n)`][emitter.setMaxListeners(n)]方法。更改默认值
为*都* `EventEmitter`实例，`events.defaultMaxListeners`
属性可以使用。如果此值不是正数，则`RangeError`
被抛出。

设置`events.defaultMaxListeners`因为
更改影响*都* `EventEmitter`实例，包括之前创建的实例
进行更改。但是，呼叫[`emitter.setMaxListeners(n)`][emitter.setMaxListeners(n)]仍然有
优先于`events.defaultMaxListeners`.

这不是一个硬性限制。这`EventEmitter`实例将允许
要添加更多侦听器，但会向 stderr 输出跟踪警告，指示
检测到“可能的事件编辑者内存泄漏”。对于任何单身人士
`EventEmitter`这`emitter.getMaxListeners()`和`emitter.setMaxListeners()`
方法可用于暂时避免此警告：

```js
emitter.setMaxListeners(emitter.getMaxListeners() + 1);
emitter.once('event', () => {
  // do stuff
  emitter.setMaxListeners(Math.max(emitter.getMaxListeners() - 1, 0));
});
```

这[`--trace-warnings`][--trace-warnings]命令行标志可用于显示
此类警告的堆栈跟踪。

发出的警告可以使用以下命令进行检查[`process.on('warning')`][process.on('warning')]并将
具有附加功能`emitter`,`type`和`count`属性，指
事件发射器实例、事件的名称和附加的数目
分别是侦听器。
其`name`属性设置为`'MaxListenersExceededWarning'`.

## `events.errorMonitor`

<!-- YAML
added:
 - v13.6.0
 - v12.17.0
-->

此符号应用于安装仅用于监视的侦听器`'error'`
事件。使用此符号安装的侦听器在常规之前调用
`'error'`调用侦听器。

使用此符号安装侦听器不会在
`'error'`发出事件。因此，如果没有，该过程仍将崩溃
定期`'error'`已安装侦听器。

## `events.getEventListeners(emitterOrTarget, eventName)`

<!-- YAML
added:
 - v15.2.0
 - v14.17.0
-->

*   `emitterOrTarget`{事件发射器|事件目标}
*   `eventName`{字符串|符号}
*   返回： {函数\[]}

返回名为 的事件的侦听器数组的副本`eventName`.

为`EventEmitter`s 此行为与调用完全相同`.listeners`上
发射器。

为`EventTarget`s 这是获取事件侦听器的唯一方法
事件目标。这对于调试和诊断目的很有用。

```mjs
import { getEventListeners, EventEmitter } from 'node:events';

{
  const ee = new EventEmitter();
  const listener = () => console.log('Events are fun');
  ee.on('foo', listener);
  getEventListeners(ee, 'foo'); // [listener]
}
{
  const et = new EventTarget();
  const listener = () => console.log('Events are fun');
  et.addEventListener('foo', listener);
  getEventListeners(et, 'foo'); // [listener]
}
```

```cjs
const { getEventListeners, EventEmitter } = require('node:events');

{
  const ee = new EventEmitter();
  const listener = () => console.log('Events are fun');
  ee.on('foo', listener);
  getEventListeners(ee, 'foo'); // [listener]
}
{
  const et = new EventTarget();
  const listener = () => console.log('Events are fun');
  et.addEventListener('foo', listener);
  getEventListeners(et, 'foo'); // [listener]
}
```

## `events.once(emitter, name[, options])`

<!-- YAML
added:
 - v11.13.0
 - v10.16.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/34912
    description: The `signal` option is supported now.
-->

*   `emitter`{事件发射器}
*   `name`{字符串}
*   `options`{对象}
    *   `signal`{中止信号}可用于取消等待事件。
*   返回： {承诺}

创建`Promise`当`EventEmitter`发出给定的
事件或被拒绝的事件，如果`EventEmitter`发出`'error'`等待时。
这`Promise`将使用发出到 的所有参数的数组进行解析
给定事件。

此方法是有意通用的，适用于Web平台
[事件目标][WHATWG-EventTarget]接口，没有特殊
`'error'`事件语义和不侦听`'error'`事件。

```mjs
import { once, EventEmitter } from 'node:events';
import process from 'node:process';

const ee = new EventEmitter();

process.nextTick(() => {
  ee.emit('myevent', 42);
});

const [value] = await once(ee, 'myevent');
console.log(value);

const err = new Error('kaboom');
process.nextTick(() => {
  ee.emit('error', err);
});

try {
  await once(ee, 'myevent');
} catch (err) {
  console.log('error happened', err);
}
```

```cjs
const { once, EventEmitter } = require('node:events');

async function run() {
  const ee = new EventEmitter();

  process.nextTick(() => {
    ee.emit('myevent', 42);
  });

  const [value] = await once(ee, 'myevent');
  console.log(value);

  const err = new Error('kaboom');
  process.nextTick(() => {
    ee.emit('error', err);
  });

  try {
    await once(ee, 'myevent');
  } catch (err) {
    console.log('error happened', err);
  }
}

run();
```

特殊处理`'error'`事件仅在以下情况下使用`events.once()`
用于等待另一个事件。如果`events.once()`用于等待
'`error'`事件本身，那么它被视为任何其他类型的事件，没有
特殊处理：

```mjs
import { EventEmitter, once } from 'node:events';

const ee = new EventEmitter();

once(ee, 'error')
  .then(([err]) => console.log('ok', err.message))
  .catch((err) => console.log('error', err.message));

ee.emit('error', new Error('boom'));

// Prints: ok boom
```

```cjs
const { EventEmitter, once } = require('node:events');

const ee = new EventEmitter();

once(ee, 'error')
  .then(([err]) => console.log('ok', err.message))
  .catch((err) => console.log('error', err.message));

ee.emit('error', new Error('boom'));

// Prints: ok boom
```

{中止信号} 可用于取消等待事件：

```mjs
import { EventEmitter, once } from 'node:events';

const ee = new EventEmitter();
const ac = new AbortController();

async function foo(emitter, event, signal) {
  try {
    await once(emitter, event, { signal });
    console.log('event emitted!');
  } catch (error) {
    if (error.name === 'AbortError') {
      console.error('Waiting for the event was canceled!');
    } else {
      console.error('There was an error', error.message);
    }
  }
}

foo(ee, 'foo', ac.signal);
ac.abort(); // Abort waiting for the event
ee.emit('foo'); // Prints: Waiting for the event was canceled!
```

```cjs
const { EventEmitter, once } = require('node:events');

const ee = new EventEmitter();
const ac = new AbortController();

async function foo(emitter, event, signal) {
  try {
    await once(emitter, event, { signal });
    console.log('event emitted!');
  } catch (error) {
    if (error.name === 'AbortError') {
      console.error('Waiting for the event was canceled!');
    } else {
      console.error('There was an error', error.message);
    }
  }
}

foo(ee, 'foo', ac.signal);
ac.abort(); // Abort waiting for the event
ee.emit('foo'); // Prints: Waiting for the event was canceled!
```

### 等待 发出的多个事件`process.nextTick()`

使用时有一个值得注意的边缘情况`events.once()`功能
以等待在同一批中发出的多个事件`process.nextTick()`
操作，或当同步发出多个事件时。具体说来
因为`process.nextTick()`队列在`Promise`微任务
队列，并且因为`EventEmitter`同步发出所有事件，这是可能的
为`events.once()`错过活动。

```mjs
import { EventEmitter, once } from 'node:events';
import process from 'node:process';

const myEE = new EventEmitter();

async function foo() {
  await once(myEE, 'bar');
  console.log('bar');

  // This Promise will never resolve because the 'foo' event will
  // have already been emitted before the Promise is created.
  await once(myEE, 'foo');
  console.log('foo');
}

process.nextTick(() => {
  myEE.emit('bar');
  myEE.emit('foo');
});

foo().then(() => console.log('done'));
```

```cjs
const { EventEmitter, once } = require('node:events');

const myEE = new EventEmitter();

async function foo() {
  await once(myEE, 'bar');
  console.log('bar');

  // This Promise will never resolve because the 'foo' event will
  // have already been emitted before the Promise is created.
  await once(myEE, 'foo');
  console.log('foo');
}

process.nextTick(() => {
  myEE.emit('bar');
  myEE.emit('foo');
});

foo().then(() => console.log('done'));
```

要捕获这两个事件，请创建每个承诺*以前*等待
，然后可以使用`Promise.all()`,`Promise.race()`,
或`Promise.allSettled()`:

```mjs
import { EventEmitter, once } from 'node:events';
import process from 'node:process';

const myEE = new EventEmitter();

async function foo() {
  await Promise.all([once(myEE, 'bar'), once(myEE, 'foo')]);
  console.log('foo', 'bar');
}

process.nextTick(() => {
  myEE.emit('bar');
  myEE.emit('foo');
});

foo().then(() => console.log('done'));
```

```cjs
const { EventEmitter, once } = require('node:events');

const myEE = new EventEmitter();

async function foo() {
  await Promise.all([once(myEE, 'bar'), once(myEE, 'foo')]);
  console.log('foo', 'bar');
}

process.nextTick(() => {
  myEE.emit('bar');
  myEE.emit('foo');
});

foo().then(() => console.log('done'));
```

## `events.captureRejections`

<!-- YAML
added:
 - v13.4.0
 - v12.16.0
changes:
  - version:
    - v17.4.0
    - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41267
    description: No longer experimental.
-->

值：{布尔值}

更改默认值`captureRejections`所有新选项上的选项`EventEmitter`对象。

## `events.captureRejectionSymbol`

<!-- YAML
added:
  - v13.4.0
  - v12.16.0
changes:
  - version:
    - v17.4.0
    - v16.14.0
    pr-url: https://github.com/nodejs/node/pull/41267
    description: No longer experimental.
-->

价值：`Symbol.for('nodejs.rejection')`

了解如何编写自定义[拒绝处理程序][rejection].

## `events.listenerCount(emitter, eventName)`

<!-- YAML
added: v0.9.12
deprecated: v3.2.0
-->

> 稳定性：0 - 已弃用：使用[`emitter.listenerCount()`][emitter.listenerCount()]相反。

*   `emitter`{事件发射器}要查询的发射器
*   `eventName`{字符串|符号}事件名称

一个类方法，它返回给定的侦听器数`eventName`
在给定的`emitter`.

```mjs
import { EventEmitter, listenerCount } from 'node:events';

const myEmitter = new EventEmitter();
myEmitter.on('event', () => {});
myEmitter.on('event', () => {});
console.log(listenerCount(myEmitter, 'event'));
// Prints: 2
```

```cjs
const { EventEmitter, listenerCount } = require('node:events');

const myEmitter = new EventEmitter();
myEmitter.on('event', () => {});
myEmitter.on('event', () => {});
console.log(listenerCount(myEmitter, 'event'));
// Prints: 2
```

## `events.on(emitter, eventName[, options])`

<!-- YAML
added:
 - v13.6.0
 - v12.16.0
-->

*   `emitter`{事件发射器}
*   `eventName`{字符串|符号}正在侦听的事件的名称
*   `options`{对象}
    *   `signal`{中止信号}可用于取消等待的事件。
*   返回：{AsyncIterator} 用于迭代`eventName`发出的事件`emitter`

```mjs
import { on, EventEmitter } from 'node:events';
import process from 'node:process';

const ee = new EventEmitter();

// Emit later on
process.nextTick(() => {
  ee.emit('foo', 'bar');
  ee.emit('foo', 42);
});

for await (const event of on(ee, 'foo')) {
  // The execution of this inner block is synchronous and it
  // processes one event at a time (even with await). Do not use
  // if concurrent execution is required.
  console.log(event); // prints ['bar'] [42]
}
// Unreachable here
```

```cjs
const { on, EventEmitter } = require('node:events');

(async () => {
  const ee = new EventEmitter();

  // Emit later on
  process.nextTick(() => {
    ee.emit('foo', 'bar');
    ee.emit('foo', 42);
  });

  for await (const event of on(ee, 'foo')) {
    // The execution of this inner block is synchronous and it
    // processes one event at a time (even with await). Do not use
    // if concurrent execution is required.
    console.log(event); // prints ['bar'] [42]
  }
  // Unreachable here
})();
```

返回`AsyncIterator`迭代`eventName`事件。它会抛出
如果`EventEmitter`发出`'error'`.它会在以下情况下删除所有侦听器：
退出循环。这`value`每次迭代返回的是一个数组
由发出的事件参数组成。

{AbortSignal} 可用于取消等待事件：

```mjs
import { on, EventEmitter } from 'node:events';
import process from 'node:process';

const ac = new AbortController();

(async () => {
  const ee = new EventEmitter();

  // Emit later on
  process.nextTick(() => {
    ee.emit('foo', 'bar');
    ee.emit('foo', 42);
  });

  for await (const event of on(ee, 'foo', { signal: ac.signal })) {
    // The execution of this inner block is synchronous and it
    // processes one event at a time (even with await). Do not use
    // if concurrent execution is required.
    console.log(event); // prints ['bar'] [42]
  }
  // Unreachable here
})();

process.nextTick(() => ac.abort());
```

```cjs
const { on, EventEmitter } = require('node:events');

const ac = new AbortController();

(async () => {
  const ee = new EventEmitter();

  // Emit later on
  process.nextTick(() => {
    ee.emit('foo', 'bar');
    ee.emit('foo', 42);
  });

  for await (const event of on(ee, 'foo', { signal: ac.signal })) {
    // The execution of this inner block is synchronous and it
    // processes one event at a time (even with await). Do not use
    // if concurrent execution is required.
    console.log(event); // prints ['bar'] [42]
  }
  // Unreachable here
})();

process.nextTick(() => ac.abort());
```

## `events.setMaxListeners(n[, ...eventTargets])`

<!-- YAML
added: v15.4.0
-->

*   `n`{数字}非负数。每个侦听器的最大数量
    `EventTarget`事件。
*   `...eventsTargets`{事件目标\[]|EventEmitter\[]} 零个或多个 {EventTarget}
    或 {EventEmitter} 实例。如果未指定任何项，`n`设置为默认值
    max 表示所有新创建的 {EventTarget} 和 {EventEmitter} 对象。

```mjs
import { setMaxListeners, EventEmitter } from 'node:events';

const target = new EventTarget();
const emitter = new EventEmitter();

setMaxListeners(5, target, emitter);
```

```cjs
const {
  setMaxListeners,
  EventEmitter
} = require('node:events');

const target = new EventTarget();
const emitter = new EventEmitter();

setMaxListeners(5, target, emitter);
```

## 类：`events.EventEmitterAsyncResource extends EventEmitter`

<!-- YAML
added:
  - v17.4.0
  - v16.14.0
-->

集成`EventEmitter`与 {AsyncResource} 一起`EventEmitter`就是那个
需要手动异步跟踪。具体而言，实例发出的所有事件
之`events.EventEmitterAsyncResource`将在其内部运行[异步上下文][async context].

```mjs
import { EventEmitterAsyncResource } from 'node:events';
import { notStrictEqual, strictEqual } from 'node:assert';
import { executionAsyncId } from 'node:async_hooks';

// Async tracking tooling will identify this as 'Q'.
const ee1 = new EventEmitterAsyncResource({ name: 'Q' });

// 'foo' listeners will run in the EventEmitters async context.
ee1.on('foo', () => {
  strictEqual(executionAsyncId(), ee1.asyncId);
  strictEqual(triggerAsyncId(), ee1.triggerAsyncId);
});

const ee2 = new EventEmitter();

// 'foo' listeners on ordinary EventEmitters that do not track async
// context, however, run in the same async context as the emit().
ee2.on('foo', () => {
  notStrictEqual(executionAsyncId(), ee2.asyncId);
  notStrictEqual(triggerAsyncId(), ee2.triggerAsyncId);
});

Promise.resolve().then(() => {
  ee1.emit('foo');
  ee2.emit('foo');
});
```

```cjs
const { EventEmitterAsyncResource } = require('node:events');
const { notStrictEqual, strictEqual } = require('node:assert');
const { executionAsyncId } = require('node:async_hooks');

// Async tracking tooling will identify this as 'Q'.
const ee1 = new EventEmitterAsyncResource({ name: 'Q' });

// 'foo' listeners will run in the EventEmitters async context.
ee1.on('foo', () => {
  strictEqual(executionAsyncId(), ee1.asyncId);
  strictEqual(triggerAsyncId(), ee1.triggerAsyncId);
});

const ee2 = new EventEmitter();

// 'foo' listeners on ordinary EventEmitters that do not track async
// context, however, run in the same async context as the emit().
ee2.on('foo', () => {
  notStrictEqual(executionAsyncId(), ee2.asyncId);
  notStrictEqual(triggerAsyncId(), ee2.triggerAsyncId);
});

Promise.resolve().then(() => {
  ee1.emit('foo');
  ee2.emit('foo');
});
```

这`EventEmitterAsyncResource`类具有相同的方法，并采用
选项与`EventEmitter`和`AsyncResource`他们自己。

### `new events.EventEmitterAsyncResource(options)`

*   `options`{对象}
    *   `captureRejections`{布尔值}它能
        [自动捕获承诺拒绝][capturerejections].
        **违约：** `false`.
    *   `name`{字符串}异步事件的类型。**违约：：**
        [`new.target.name`][new.target.name].
    *   `triggerAsyncId`{数字}创建此值的执行上下文的 ID
        异步事件。**违约：** `executionAsyncId()`.
    *   `requireManualDestroy`{布尔值}如果设置为`true`禁用`emitDestroy`
        当对象被垃圾回收时。这通常不需要设置
        （即使`emitDestroy`手动调用），除非资源的`asyncId`
        被检索到，并且敏感的 API`emitDestroy`与它一起调用。
        设置为`false`这`emitDestroy`调用垃圾回收
        仅当至少有一个活动状态时才会发生`destroy`钩。
        **违约：** `false`.

### `eventemitterasyncresource.asyncId`

*   类型：{数字} 唯一`asyncId`分配给资源。

### `eventemitterasyncresource.asyncResource`

*   类型：基础 {AsyncResource}。

返回的`AsyncResource`对象具有附加`eventEmitter`财产
提供了对此的引用`EventEmitterAsyncResource`.

### `eventemitterasyncresource.emitDestroy()`

全部致电`destroy`钩。这应该只调用一次。错误将
如果它被调用多次，则被抛出。这**必须**手动调用。如果
资源由GC收集，然后`destroy`钩子将
永远不要被召唤。

### `eventemitterasyncresource.triggerAsyncId`

*   类型：{数字} 相同`triggerAsyncId`传递给
    `AsyncResource`构造 函数。

<a id="event-target-and-event-api"></a>

## `EventTarget`和`Event`应用程序接口

<!-- YAML
added: v14.5.0
changes:
  - version: v16.0.0
    pr-url: https://github.com/nodejs/node/pull/37237
    description: changed EventTarget error handling.
  - version: v15.4.0
    pr-url: https://github.com/nodejs/node/pull/35949
    description: No longer experimental.
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35496
    description:
      The `EventTarget` and `Event` classes are now available as globals.
-->

这`EventTarget`和`Event`对象是特定于节点.js实现
的[`EventTarget`网络接口][EventTarget Web API]由某些 Node.js 核心 API 公开。

```js
const target = new EventTarget();

target.addEventListener('foo', (event) => {
  console.log('foo event happened!');
});
```

### 节点.js`EventTarget`与 DOM`EventTarget`

节点之间有两个主要区别.js`EventTarget`和
[`EventTarget`网络接口][EventTarget Web API]:

1.  而 DOM`EventTarget`实例*五月*是分层的，没有
    Node.js中的层次结构和事件传播的概念。即事件
    已发送到`EventTarget`不通过
    嵌套的目标对象，每个对象可能都有自己的一组处理程序
    事件。
2.  在节点中.js`EventTarget`，如果事件侦听器是异步函数
    或返回`Promise`，并返回`Promise`拒绝，拒绝
    自动捕获和处理方式与侦听器相同
    同步抛出（请参见[`EventTarget`错误处理][EventTarget error handling]了解详情）。

### `NodeEventTarget`与。`EventEmitter`

这`NodeEventTarget`对象实现了
`EventEmitter`允许它紧密结合的 API*模仿*一`EventEmitter`在
某些情况。一个`NodeEventTarget`是*不*的实例`EventEmitter`
并且不能代替`EventEmitter`在大多数情况下。

1.  与`EventEmitter`，任何给定`listener`最多可以注册一次
    每个事件`type`.尝试注册`listener`多次是
    忽视。
2.  这`NodeEventTarget`不模仿完整`EventEmitter`应用程序接口。
    具体来说`prependListener()`,`prependOnceListener()`,
    `rawListeners()`,`setMaxListeners()`,`getMaxListeners()`和
    `errorMonitor`不模拟 API。这`'newListener'`和
    `'removeListener'`事件也不会发出。
3.  这`NodeEventTarget`不实现任何特殊的默认行为
    对于具有类型的事件`'error'`.
4.  这`NodeEventTarget`支持`EventListener`对象以及
    函数作为所有事件类型的处理程序。

### 事件侦听器

为事件注册的事件侦听器`type`可以是 JavaScript
函数或对象具有`handleEvent`其值为函数的属性。

在任一情况下，处理程序函数都使用`event`论点
传递给`eventTarget.dispatchEvent()`功能。

异步函数可用作事件侦听器。如果异步处理程序函数
拒绝，则捕获和处理拒绝，如 中所述
[`EventTarget`错误处理][EventTarget error handling].

一个处理程序函数引发的错误不会阻止其他处理程序
从被调用。

处理程序函数的返回值将被忽略。

处理程序始终按其添加顺序调用。

处理程序函数可能会改变`event`对象。

```js
function handler1(event) {
  console.log(event.type);  // Prints 'foo'
  event.a = 1;
}

async function handler2(event) {
  console.log(event.type);  // Prints 'foo'
  console.log(event.a);  // Prints 1
}

const handler3 = {
  handleEvent(event) {
    console.log(event.type);  // Prints 'foo'
  }
};

const handler4 = {
  async handleEvent(event) {
    console.log(event.type);  // Prints 'foo'
  }
};

const target = new EventTarget();

target.addEventListener('foo', handler1);
target.addEventListener('foo', handler2);
target.addEventListener('foo', handler3);
target.addEventListener('foo', handler4, { once: true });
```

### `EventTarget`错误处理

当已注册的事件侦听器引发（或返回拒绝的承诺）时，
默认情况下，该错误被视为未捕获的异常
`process.nextTick()`.这意味着`EventTarget`的遗嘱
默认情况下终止节点.js进程。

在事件侦听器中抛出将*不*停止其他已注册的处理程序
从被调用。

这`EventTarget`不实现任何特殊的默认处理`'error'`
类型事件，如`EventEmitter`.

当前，错误首先转发到`process.on('error')`事件
到达之前`process.on('uncaughtException')`.此行为是
已弃用，并将在将来的版本中进行更改以对齐`EventTarget`跟
其他节点.js API。任何依赖于`process.on('error')`事件应该
与新行为保持一致。

### 类：`Event`

<!-- YAML
added: v14.5.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35496
    description: The `Event` class is now available through the global object.
-->

这`Event`对象是[`Event`网络接口][Event Web API].实例
由 Node.js 在内部创建。

#### `event.bubbles`

<!-- YAML
added: v14.5.0
-->

*   类型：{布尔值} 始终返回`false`.

这在 Node.js中使用，提供它纯粹是为了完整性。

#### `event.cancelBubble()`

<!-- YAML
added: v14.5.0
-->

的别名`event.stopPropagation()`.这不在 Node 中使用.js并且是
纯粹为了完整性而提供。

#### `event.cancelable`

<!-- YAML
added: v14.5.0
-->

*   类型：{布尔值} True，如果事件是使用`cancelable`选择。

#### `event.composed`

<!-- YAML
added: v14.5.0
-->

*   类型：{布尔值} 始终返回`false`.

这在 Node.js中使用，提供它纯粹是为了完整性。

#### `event.composedPath()`

<!-- YAML
added: v14.5.0
-->

返回一个数组，其中包含`EventTarget`作为唯一的条目或
如果事件未调度，则为空。这未用于
Node.js，纯粹是为了完整性而提供的。

#### `event.currentTarget`

<!-- YAML
added: v14.5.0
-->

*   类型： {事件目标} The`EventTarget`调度事件。

的别名`event.target`.

#### `event.defaultPrevented`

<!-- YAML
added: v14.5.0
-->

*   类型： {布尔值}

是`true`如果`cancelable`是`true`和`event.preventDefault()`已经
叫。

#### `event.eventPhase`

<!-- YAML
added: v14.5.0
-->

*   类型：{数字} 返回`0`未调度事件时，`2`而
    它正在被调度。

这在 Node.js中使用，提供它纯粹是为了完整性。

#### `event.isTrusted`

<!-- YAML
added: v14.5.0
-->

*   类型： {布尔值}

{中止信号}`"abort"`事件发出`isTrusted`设置为`true`.这
值为`false`在所有其他情况下。

#### `event.preventDefault()`

<!-- YAML
added: v14.5.0
-->

设置`defaultPrevented`属性到`true`如果`cancelable`是`true`.

#### `event.returnValue`

<!-- YAML
added: v14.5.0
-->

*   类型：{布尔值} True（如果事件尚未取消）。

这在 Node.js中使用，提供它纯粹是为了完整性。

#### `event.srcElement`

<!-- YAML
added: v14.5.0
-->

*   类型： {事件目标} The`EventTarget`调度事件。

的别名`event.target`.

#### `event.stopImmediatePropagation()`

<!-- YAML
added: v14.5.0
-->

在当前事件侦听器完成后停止对事件侦听器的调用。

#### `event.stopPropagation()`

<!-- YAML
added: v14.5.0
-->

这在 Node.js中使用，提供它纯粹是为了完整性。

#### `event.target`

<!-- YAML
added: v14.5.0
-->

*   类型： {事件目标} The`EventTarget`调度事件。

#### `event.timeStamp`

<!-- YAML
added: v14.5.0
-->

*   类型： {数字}

毫秒时间戳`Event`对象已创建。

#### `event.type`

<!-- YAML
added: v14.5.0
-->

*   类型： {字符串}

事件类型标识符。

### 类：`EventTarget`

<!-- YAML
added: v14.5.0
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35496
    description:
      The `EventTarget` class is now available through the global object.
-->

#### `eventTarget.addEventListener(type, listener[, options])`

<!-- YAML
added: v14.5.0
changes:
  - version: v15.4.0
    pr-url: https://github.com/nodejs/node/pull/36258
    description: add support for `signal` option.
-->

*   `type`{字符串}
*   `listener`{功能|EventListener}
*   `options`{对象}
    *   `once`{布尔值}什么时候`true`，侦听器将自动删除
        首次调用时。**违约：** `false`.
    *   `passive`{布尔值}什么时候`true`，作为侦听器将
        不调用`Event`对象的`preventDefault()`方法。
        **违约：** `false`.
    *   `capture`{布尔值}节点不直接使用.js。已针对 API 添加
        完整性。**违约：** `false`.
    *   `signal`{中止信号}当给定的
        中止信号对象的`abort()`调用方法。

为`type`事件。任何给定`listener`已添加
每只一次`type`和每个`capture`选项值。

如果`once`选项是`true`这`listener`在
下次再来一次`type`事件被调度。

这`capture`选项未由 Node 使用.js以除
跟踪每个已注册事件侦听器`EventTarget`规范。
具体来说，`capture`选项在注册时用作密钥的一部分
一个`listener`.任何个人`listener`可添加一次
`capture = false`，并且一次`capture = true`.

```js
function handler(event) {}

const target = new EventTarget();
target.addEventListener('foo', handler, { capture: true });  // first
target.addEventListener('foo', handler, { capture: false }); // second

// Removes the second instance of handler
target.removeEventListener('foo', handler);

// Removes the first instance of handler
target.removeEventListener('foo', handler, { capture: true });
```

#### `eventTarget.dispatchEvent(event)`

<!-- YAML
added: v14.5.0
-->

*   `event`{事件}
*   返回：{布尔值}`true`如果任一事件的`cancelable`属性值为
    假或其`preventDefault()`未调用方法，否则`false`.

调度`event`到 的处理程序列表`event.type`.

已注册的事件侦听器按其顺序同步调用
已注册。

#### `eventTarget.removeEventListener(type, listener)`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}
*   `listener`{功能|EventListener}
*   `options`{对象}
    *   `capture`{布尔值}

删除`listener`从事件的处理程序列表中`type`.

### 类：`CustomEvent`

<!-- YAML
added: v18.7.0
-->

> 稳定性： 1 - 实验性。

*   扩展：{事件}

这`CustomEvent`对象是[`CustomEvent`网络接口][CustomEvent Web API].
实例由 Node.js 在内部创建。

#### `event.detail`

<!-- YAML
added: v18.7.0
-->

> 稳定性： 1 - 实验性。

*   类型：{any} 返回初始化时传递的自定义数据。

只读。

### 类：`NodeEventTarget`

<!-- YAML
added: v14.5.0
-->

*   扩展：{事件目标}

这`NodeEventTarget`是特定于节点.js扩展`EventTarget`
模拟`EventEmitter`应用程序接口。

#### `nodeEventTarget.addListener(type, listener[, options])`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   `listener`{功能|EventListener}

*   `options`{对象}
    *   `once`{布尔值}

*   返回： {事件目标} 此

特定于节点.js的扩展`EventTarget`类，该类模拟
等效`EventEmitter`应用程序接口。唯一的区别`addListener()`和
`addEventListener()`那是`addListener()`将返回对
`EventTarget`.

#### `nodeEventTarget.eventNames()`

<!-- YAML
added: v14.5.0
-->

*   返回：{字符串\[]}

特定于节点.js的扩展`EventTarget`返回数组的类
事件数`type`为其注册事件侦听器的名称。

#### `nodeEventTarget.listenerCount(type)`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   返回值：{数字}

特定于节点.js的扩展`EventTarget`返回数字的类
为`type`.

#### `nodeEventTarget.off(type, listener)`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   `listener`{功能|EventListener}

*   返回： {事件目标} 此

的节点.js特定别名`eventTarget.removeListener()`.

#### `nodeEventTarget.on(type, listener[, options])`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   `listener`{功能|EventListener}

*   `options`{对象}
    *   `once`{布尔值}

*   返回： {事件目标} 此

的节点.js特定别名`eventTarget.addListener()`.

#### `nodeEventTarget.once(type, listener[, options])`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   `listener`{功能|EventListener}

*   `options`{对象}

*   返回： {事件目标} 此

特定于节点.js的扩展`EventTarget`添加`once`
给定事件的侦听器`type`.这相当于调用`on`
与`once`选项设置为`true`.

#### `nodeEventTarget.removeAllListeners([type])`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   返回： {事件目标} 此

特定于节点.js的扩展`EventTarget`类。如果`type`指定，
删除 的所有已注册侦听器`type`，否则将删除所有已注册的内容
听众。

#### `nodeEventTarget.removeListener(type, listener)`

<!-- YAML
added: v14.5.0
-->

*   `type`{字符串}

*   `listener`{功能|EventListener}

*   返回： {事件目标} 此

特定于节点.js的扩展`EventTarget`类，该类删除
`listener`对于给定的`type`.唯一的区别`removeListener()`
和`removeEventListener()`那是`removeListener()`将返回引用
到`EventTarget`.

[WHATWG-EventTarget]: https://dom.spec.whatwg.org/#interface-eventtarget

[`--trace-warnings`]: cli.md#--trace-warnings

[`CustomEvent` Web API]: https://dom.spec.whatwg.org/#customevent

[`EventTarget` Web API]: https://dom.spec.whatwg.org/#eventtarget

[`EventTarget` error handling]: #eventtarget-error-handling

[`Event` Web API]: https://dom.spec.whatwg.org/#event

[`domain`]: domain.md

[`emitter.listenerCount()`]: #emitterlistenercounteventname

[`emitter.removeListener()`]: #emitterremovelistenereventname-listener

[`emitter.setMaxListeners(n)`]: #emittersetmaxlistenersn

[`events.defaultMaxListeners`]: #eventsdefaultmaxlisteners

[`fs.ReadStream`]: fs.md#class-fsreadstream

[`net.Server`]: net.md#class-netserver

[`new.target.name`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target

[`process.on('warning')`]: process.md#event-warning

[async context]: async_context.md

[capturerejections]: #capture-rejections-of-promises

[error]: #error-events

[rejection]: #emittersymbolfornodejsrejectionerr-eventname-args

[rejectionsymbol]: #eventscapturerejectionsymbol

[stream]: stream.md
