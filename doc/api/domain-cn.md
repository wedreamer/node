# 域

<!-- YAML
deprecated: v1.4.2
changes:
  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15695
    description: Any `Promise`s created in VM contexts no longer have a
                 `.domain` property. Their handlers are still executed in the
                 proper domain, however, and `Promise`s created in the main
                 context still possess a `.domain` property.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12489
    description: Handlers for `Promise`s are now invoked in the domain in which
                 the first promise of a chain was created.
-->

<!--introduced_in=v0.10.0-->

> 稳定性：0 - 已弃用

<!-- source_link=lib/domain.js -->

**此模块正在等待弃用。**一旦替换 API 已
最终确定，此模块将被完全弃用。大多数开发人员应该
**不**有理由使用此模块。绝对必须拥有的用户
域提供的功能可能暂时依赖于它
但应该期望必须迁移到其他解决方案
在未来。

域提供了一种将多个不同的 IO 操作作为
单个组。如果任何事件发射器或回调注册到
域发出`'error'`事件，或引发错误，然后是域对象
将收到通知，而不是丢失错误上下文中的
`process.on('uncaughtException')`处理程序，或使程序
立即退出并显示错误代码。

## 警告：不要忽略错误！

<!-- type=misc -->

域错误处理程序不能替代关闭
发生错误时的进程。

按其本质说明[`throw`][throw]在JavaScript中工作，几乎有
从来没有任何方法可以安全地“从它离开的地方捡起”，而不会泄漏
引用，或创建某种其他类型的未定义脆性状态。

响应抛出错误的最安全方法是关闭
过程。当然，在普通的Web服务器中，可能有很多
打开连接，突然关闭这些连接是不合理的
因为错误是由其他人触发的。

更好的方法是向请求发送错误响应
触发了错误，同时让其他人以正常的方式完成
时间，并停止侦听该工作线程中的新请求。

通过这种方式，`domain`使用与集群模块齐头并进，
因为主要流程可以在工作线程时分叉新工作线程
遇到错误。对于 Node.js扩展到多个的程序
机器、终止代理或服务注册表可以记下
失败，并做出相应的反应。

例如，这不是一个好主意：

```js
// XXX WARNING! BAD IDEA!

const d = require('node:domain').create();
d.on('error', (er) => {
  // The error won't crash the process, but what it does is worse!
  // Though we've prevented abrupt process restarting, we are leaking
  // a lot of resources if this ever happens.
  // This is no better than process.on('uncaughtException')!
  console.log(`error, but oh well ${er.message}`);
});
d.run(() => {
  require('node:http').createServer((req, res) => {
    handleRequest(req, res);
  }).listen(PORT);
});
```

通过使用域的上下文，以及分离我们的弹性
程序化为多个工作进程，我们可以反应更多
适当地，并以更高的安全性处理错误。

```js
// Much better!

const cluster = require('node:cluster');
const PORT = +process.env.PORT || 1337;

if (cluster.isPrimary) {
  // A more realistic scenario would have more than 2 workers,
  // and perhaps not put the primary and worker in the same file.
  //
  // It is also possible to get a bit fancier about logging, and
  // implement whatever custom logic is needed to prevent DoS
  // attacks and other bad behavior.
  //
  // See the options in the cluster documentation.
  //
  // The important thing is that the primary does very little,
  // increasing our resilience to unexpected errors.

  cluster.fork();
  cluster.fork();

  cluster.on('disconnect', (worker) => {
    console.error('disconnect!');
    cluster.fork();
  });

} else {
  // the worker
  //
  // This is where we put our bugs!

  const domain = require('node:domain');

  // See the cluster documentation for more details about using
  // worker processes to serve requests. How it works, caveats, etc.

  const server = require('node:http').createServer((req, res) => {
    const d = domain.create();
    d.on('error', (er) => {
      console.error(`error ${er.stack}`);

      // We're in dangerous territory!
      // By definition, something unexpected occurred,
      // which we probably didn't want.
      // Anything can happen now! Be very careful!

      try {
        // Make sure we close down within 30 seconds
        const killtimer = setTimeout(() => {
          process.exit(1);
        }, 30000);
        // But don't keep the process open just for that!
        killtimer.unref();

        // Stop taking new requests.
        server.close();

        // Let the primary know we're dead. This will trigger a
        // 'disconnect' in the cluster primary, and then it will fork
        // a new worker.
        cluster.worker.disconnect();

        // Try to send an error to the request that triggered the problem
        res.statusCode = 500;
        res.setHeader('content-type', 'text/plain');
        res.end('Oops, there was a problem!\n');
      } catch (er2) {
        // Oh well, not much we can do at this point.
        console.error(`Error sending 500! ${er2.stack}`);
      }
    });

    // Because req and res were created before this domain existed,
    // we need to explicitly add them.
    // See the explanation of implicit vs explicit binding below.
    d.add(req);
    d.add(res);

    // Now run the handler function in the domain.
    d.run(() => {
      handleRequest(req, res);
    });
  });
  server.listen(PORT);
}

// This part is not important. Just an example routing thing.
// Put fancy application logic here.
function handleRequest(req, res) {
  switch (req.url) {
    case '/error':
      // We do some async stuff, and then...
      setTimeout(() => {
        // Whoops!
        flerb.bark();
      }, timeout);
      break;
    default:
      res.end('ok');
  }
}
```

## 对`Error`对象

<!-- type=misc -->

任何时候`Error`对象通过域路由，一些额外的字段
添加到其中。

*   `error.domain`首先处理错误的域。
*   `error.domainEmitter`发出的事件发射器`'error'`事件
    与错误对象一起使用。
*   `error.domainBound`绑定到
    域，并传递了一个错误作为其第一个参数。
*   `error.domainThrown`指示错误是否为
    抛出、发出或传递给绑定的回调函数。

## 隐式绑定

<!--type=misc-->

如果域正在使用中，则所有域**新增功能** `EventEmitter`对象（包括
流式传输对象、请求、响应等）将隐式绑定到
创建时的活动域。

此外，传递给低级事件循环请求的回调（例如
自`fs.open()`，或其他回调方法）将自动
绑定到活动域。如果他们抛出，那么域将捕获
错误。

为了防止过多的内存使用，`Domain`对象本身
不隐式添加为活动域的子级。如果他们
那么阻止请求和响应对象就太容易了
从被适当的垃圾收集。

嵌套`Domain`对象作为父项的子项`Domain`他们必须是
显式添加。

隐式绑定路由引发错误和`'error'`事件到
`Domain`的`'error'`事件，但不注册`EventEmitter`在
`Domain`.
隐式绑定仅处理引发的错误和`'error'`事件。

## 显式绑定

<!--type=misc-->

有时，正在使用的域不是应该用于
特定事件发射器。或者，事件发射器可能已创建
在一个域的上下文中，但应该绑定到一些域
其他域。

例如，可能有一个域正在用于 HTTP 服务器，但是
也许我们希望为每个请求使用一个单独的域。

这可以通过显式绑定来实现。

```js
// Create a top-level domain for the server
const domain = require('node:domain');
const http = require('node:http');
const serverDomain = domain.create();

serverDomain.run(() => {
  // Server is created in the scope of serverDomain
  http.createServer((req, res) => {
    // Req and res are also created in the scope of serverDomain
    // however, we'd prefer to have a separate domain for each request.
    // create it first thing, and add req and res to it.
    const reqd = domain.create();
    reqd.add(req);
    reqd.add(res);
    reqd.on('error', (er) => {
      console.error('Error', er, req.url);
      try {
        res.writeHead(500);
        res.end('Error occurred, sorry.');
      } catch (er2) {
        console.error('Error sending 500', er2, req.url);
      }
    });
  }).listen(1337);
});
```

## `domain.create()`

*   返回：{域}

## 类：`Domain`

*   扩展：{事件发射器}

这`Domain`类封装了路由错误和
活动未捕获的异常`Domain`对象。

要处理它捕获的错误，请侦听`'error'`事件。

### `domain.members`

*   {数组}

已显式添加的计时器和事件发射器的数组
到域。

### `domain.add(emitter)`

*   `emitter`{事件发射器|定时器} 发射器或要添加到域中的定时器

显式向域添加发射器。如果 调用的任何事件处理程序
发射器抛出错误，或者如果发射器发出`'error'`事件，它
将被路由到域的`'error'`事件，就像隐式一样
捆绑。

这也适用于从[`setInterval()`][setInterval()]和
[`setTimeout()`][setTimeout()].如果他们的回调函数抛出，它将被捕获
域`'error'`处理器。

如果计时器或`EventEmitter`已绑定到域，它将被删除
从那个，并绑定到这个。

### `domain.bind(callback)`

*   `callback`{函数}回调函数
*   返回：{函数} 绑定函数

返回的函数将是围绕所提供回调的包装器
功能。调用返回的函数时，任何错误
抛出的将被路由到域的`'error'`事件。

```js
const d = domain.create();

function readSomeFile(filename, cb) {
  fs.readFile(filename, 'utf8', d.bind((er, data) => {
    // If this throws, it will also be passed to the domain.
    return cb(er, data ? JSON.parse(data) : null);
  }));
}

d.on('error', (er) => {
  // An error occurred somewhere. If we throw it now, it will crash the program
  // with the normal line number and stack message.
});
```

### `domain.enter()`

这`enter()`方法是管道使用的`run()`,`bind()`和
`intercept()`方法来设置活动域。它设置`domain.active`和
`process.domain`到域，并隐式将域推送到域上
由域模块管理的堆栈（请参见[`domain.exit()`][domain.exit()]有关
域堆栈）。调用`enter()`分隔链的开头
绑定到域的异步调用和 I/O 操作。

叫`enter()`仅更改活动域，而不更改域
本身。`enter()`和`exit()`可以在
单个域。

### `domain.exit()`

这`exit()`方法退出当前域，将其从域堆栈中弹出。
任何时候执行都会切换到不同链的上下文
异步调用，确保退出当前域非常重要。
调用`exit()`分隔链的末端或链的中断
绑定到域的异步调用和 I/O 操作。

如果有多个嵌套域绑定到当前执行上下文，
`exit()`将退出嵌套在此域中的任何域。

叫`exit()`仅更改活动域，而不更改域
本身。`enter()`和`exit()`可以在
单个域。

### `domain.intercept(callback)`

*   `callback`{函数}回调函数
*   返回：{函数} 截获的函数

此方法几乎与[`domain.bind(callback)`][domain.bind(callback)].但是，在
除了捕获抛出的错误外，它还会拦截[`Error`][Error]
对象作为函数的第一个参数发送。

这样，共同`if (err) return callback(err);`图案可以替换
在单个位置使用单个错误处理程序。

```js
const d = domain.create();

function readSomeFile(filename, cb) {
  fs.readFile(filename, 'utf8', d.intercept((data) => {
    // Note, the first argument is never passed to the
    // callback since it is assumed to be the 'Error' argument
    // and thus intercepted by the domain.

    // If this throws, it will also be passed to the domain
    // so the error-handling logic can be moved to the 'error'
    // event on the domain instead of being repeated throughout
    // the program.
    return cb(null, JSON.parse(data));
  }));
}

d.on('error', (er) => {
  // An error occurred somewhere. If we throw it now, it will crash the program
  // with the normal line number and stack message.
});
```

### `domain.remove(emitter)`

*   `emitter`{事件发射器|定时器} 发射器或定时器要从域中删除

相反[`domain.add(emitter)`][domain.add(emitter)].从 中删除域处理
指定的发射器。

### `domain.run(fn[, ...args])`

*   `fn`{函数}
*   `...args`{任何}

在域的上下文中隐式运行提供的函数
绑定所有事件发射器、计时器和低级别请求
在这种背景下创建。（可选）可以将参数传递给
函数。

这是使用域的最基本方法。

```js
const domain = require('node:domain');
const fs = require('node:fs');
const d = domain.create();
d.on('error', (er) => {
  console.error('Caught error!', er);
});
d.run(() => {
  process.nextTick(() => {
    setTimeout(() => { // Simulating some various async stuff
      fs.open('non-existent file', 'r', (er, fd) => {
        if (er) throw er;
        // proceed...
      });
    }, 100);
  });
});
```

在此示例中，`d.on('error')`处理程序将被触发，而不是
而不是使程序崩溃。

## 域名和承诺

从 Node.js 8.0.0 开始，promise 的处理程序在域内运行
调用`.then()`或`.catch()`本身是制作的：

```js
const d1 = domain.create();
const d2 = domain.create();

let p;
d1.run(() => {
  p = Promise.resolve(42);
});

d2.run(() => {
  p.then((v) => {
    // running in d2
  });
});
```

回调可以使用[`domain.bind(callback)`][domain.bind(callback)]绑定到特定域:

```js
const d1 = domain.create();
const d2 = domain.create();

let p;
d1.run(() => {
  p = Promise.resolve(42);
});

d2.run(() => {
  p.then(p.domain.bind((v) => {
    // running in d1
  }));
});
```

域不会干扰
承诺。换句话说，没有`'error'`对于未处理，将发出事件
`Promise`拒绝。

[`Error`]: errors.md#class-error

[`domain.add(emitter)`]: #domainaddemitter

[`domain.bind(callback)`]: #domainbindcallback

[`domain.exit()`]: #domainexit

[`setInterval()`]: timers.md#setintervalcallback-delay-args

[`setTimeout()`]: timers.md#settimeoutcallback-delay-args

[`throw`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw
