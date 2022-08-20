# C++ addons

<!--introduced_in=v0.10.0-->

<!-- type=misc -->

_Addons_ 是用 C++ 编写的动态链接共享对象。 [`require()`][require] 函数可以将插件加载为普通的 Node.js 模块.
插件提供 JavaScript 和 C/C++ 库之间的接口.

实现插件有三个选项：Node-API、nan，或直接使用内部 V8、libuv 和 Node.js 库。除非需要直接访问 Node-API 未公开的功能，否则请使用 Node-API.
有关 Node-API 的更多信息，请参阅 [使用 Node-API 的 C/C++ 插件](n-api.md).

不使用 Node-API 时，实现插件很复杂，涉及到几个组件和 API 的知识:

* [V8][]：Node.js 用于提供 JavaScript 实现的 C++ 库。 V8 提供了创建对象、调用函数等的机制。V8 的 API 主要记录在 `v8.h` 头文件（Node.js 源代码树中的`deps/v8/include/v8.h`）中，即也可用 [在线][v8-docs].

* [libuv][]：实现 Node.js 事件循环、它的工作线程和平台的所有异步行为的 C 库。它还用作跨平台抽象库，为所有主要操作系统提供简单的、类似 POSIX 的访问，以访问许多常见的系统任务，例如与文件系统、套接字、计时器和系统事件交互。 libuv 还为需要超越标准事件循环的更复杂的异步插件提供类似于 POSIX 线程的线程抽象。插件作者应避免使用 I/O 或其他时间密集型任务阻塞事件循环，方法是通过 libuv 将工作卸载到非阻塞系统操作、工作线程或自定义使用 libuv 线程.

* 内部 Node.js 库。 Node.js 本身导出了插件可以使用的 C++ API，其中最重要的是 `node::ObjectWrap` 类.

* Node.js 包括其他静态链接库，包括 OpenSSL。这些其他库位于 Node.js 源代码树的 `deps/` 目录中。只有 libuv、OpenSSL、V8 和 zlib 符号被 Node.js 有目的地重新导出，并且可以被插件在不同程度上使用。有关更多信息，请参阅 [链接到 Node.js 中包含的库][].

以下所有示例都可用于 [download][] 并可用作插件的起点.

## Hello world

这个“Hello world”示例是一个简单的插件，用 C++ 编写，相当于以下 JavaScript 代码:

```js
module.exports.hello = () => 'world';
```

First, create the file `hello.cc`:

```cpp
// hello.cc
#include <node.h>

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void Method(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  args.GetReturnValue().Set(String::NewFromUtf8(
      isolate, "world").ToLocalChecked());
}

void Initialize(Local<Object> exports) {
  NODE_SET_METHOD(exports, "hello", Method);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize)

}  // namespace demo
```

所有 Node.js 插件都必须按照模式导出初始化函数:

```cpp
void Initialize(Local<Object> exports);
NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize)
```

`NODE MODULE` 后面没有分号，因为它不是函数（参见 `node.h`）.

`module_name` 必须与最终二进制文件的文件名匹配（不包括 `.node` 后缀）.

那么在`hello.cc`例子中，初始化函数是`Initialize`，插件模块名是`addon`.

当使用 `node-gyp` 构建插件时，使用宏 `NODE_GYP_MODULE_NAME` 作为 `NODE_MODULE()` 的第一个参数将确保最终二进制文件的名称将传递给 `NODE_MODULE()`.

### Context-aware addons

在某些环境中，可能需要在多个上下文中多次加载 Node.js 插件。例如，[Electron][] 运行时在单个进程中运行多个 Node.js 实例。每个实例都有自己的 `require()` 缓存，因此每个实例在通过 `require()` 加载时都需要一个本机插件才能正常运行。这意味着插件必须支持多个初始化.

可以使用宏“NODE_MODULE_INITIALIZER”构建上下文感知插件，该宏扩展为 Node.js 在加载插件时期望找到的函数的名称。因此，可以像以下示例一样初始化插件:

```cpp
using namespace v8;

extern "C" NODE_MODULE_EXPORT void
NODE_MODULE_INITIALIZER(Local<Object> exports,
                        Local<Value> module,
                        Local<Context> context) {
  /* Perform addon initialization steps here. */
}
```

另一种选择是使用宏 `NODE_MODULE_INIT()`，它还将构建一个上下文感知插件。与 `NODE_MODULE()` 不同，后者用于
围绕给定的插件初始化函数构造一个插件，`NODE_MODULE_INIT()` 用作这种初始化器的声明，后面跟着一个函数体.

在调用“NODE_MODULE_INIT()”之后，可以在函数体内使用以下三个变量:

* `Local<Object> exports`,
* `Local<Value> module`, and
* `Local<Context> context`

构建上下文感知插件的选择带有仔细管理全局静态数据的责任。由于插件可能会被多次加载，甚至可能来自不同的线程，因此存储在插件中的任何全局静态数据都必须得到适当的保护，并且不得包含对 JavaScript 对象的任何持久引用。这样做的原因是 JavaScript 对象仅在一个上下文中有效，并且当从错误的上下文或从与创建它们的线程不同的线程访问时可能会导致崩溃.

通过执行以下步骤，可以构建上下文感知插件以避免全局静态数据:

* 定义一个类，该类将保存每个插件实例的数据并具有表单的静态成员
  ```cpp
  static void DeleteInstance(void* data) {
    // Cast `data` to an instance of the class and delete it.
  }
  ```
* 在插件初始化器中堆分配此类的一个实例。这可以使用 `new` 关键字来完成.
* 调用 `node::AddEnvironmentCleanupHook()`，将上面创建的实例和指向 `DeleteInstance()` 的指针传递给它。这将确保在环境被拆除时删除实例.
* 将类的实例存储在 `v8::External` 中，并且
* 将 `v8::External` 传递给所有暴露给 JavaScript 的方法，方法是将其传递给 `v8::FunctionTemplate::New()` 或 `v8::Function::New()` 以创建原生支持的 JavaScript 函数。 `v8::FunctionTemplate::New()` 或 `v8::Function::New()` 的第三个参数接受 `v8::External` 并使用 `v8::FunctionCallbackInfo 使其在本机回调中可用::Data()` 方法.

这将确保每个插件实例的数据到达每个可以从 JavaScript 调用的绑定。每个插件实例的数据也必须传递到插件可能创建的任何异步回调中.

以下示例说明了上下文感知插件的实现:

```cpp
#include <node.h>

using namespace v8;

class AddonData {
 public:
  explicit AddonData(Isolate* isolate):
      call_count(0) {
    // Ensure this per-addon-instance data is deleted at environment cleanup.
    node::AddEnvironmentCleanupHook(isolate, DeleteInstance, this);
  }

  // Per-addon data.
  int call_count;

  static void DeleteInstance(void* data) {
    delete static_cast<AddonData*>(data);
  }
};

static void Method(const v8::FunctionCallbackInfo<v8::Value>& info) {
  // Retrieve the per-addon-instance data.
  AddonData* data =
      reinterpret_cast<AddonData*>(info.Data().As<External>()->Value());
  data->call_count++;
  info.GetReturnValue().Set((double)data->call_count);
}

// Initialize this addon to be context-aware.
NODE_MODULE_INIT(/* exports, module, context */) {
  Isolate* isolate = context->GetIsolate();

  // Create a new instance of `AddonData` for this instance of the addon and
  // tie its life cycle to that of the Node.js environment.
  AddonData* data = new AddonData(isolate);

  // Wrap the data in a `v8::External` so we can pass it to the method we
  // expose.
  Local<External> external = External::New(isolate, data);

  // Expose the method `Method` to JavaScript, and make sure it receives the
  // per-addon-instance data we created above by passing `external` as the
  // third parameter to the `FunctionTemplate` constructor.
  exports->Set(context,
               String::NewFromUtf8(isolate, "method").ToLocalChecked(),
               FunctionTemplate::New(isolate, Method, external)
                  ->GetFunction(context).ToLocalChecked()).FromJust();
}
```

#### Worker support

<!-- YAML
changes:
  - version:
    - v14.8.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34572
    description: Cleanup hooks may now be asynchronous.
-->

为了从多个 Node.js 环境（例如主线程和工作线程）加载，附加组件需要:

* 成为 Node-API 插件，或者
* 如上所述使用“NODE_MODULE_INIT()”声明为上下文感知

为了支持 [`Worker`][] 线程，插件需要在存在此类线程时清理它们可能分配的任何资源。这可以通过使用 `AddEnvironmentCleanupHook()` 函数来实现:

```cpp
void AddEnvironmentCleanupHook(v8::Isolate* isolate,
                               void (*fun)(void* arg),
                               void* arg);
```

该函数添加了一个钩子，该钩子将在给定的 Node.js 实例关闭之前运行。如有必要，可以使用具有相同签名的“RemoveEnvironmentCleanupHook()”在运行之前删除此类挂钩。回调按照后进先出的顺序运行.

如有必要，还有一对额外的 `AddEnvironmentCleanupHook()` 和 `RemoveEnvironmentCleanupHook()` 重载，其中清理挂钩采用回调函数。这可用于关闭异步资源,
例如插件注册的任何 libuv 句柄.

以下 `addon.cc` 使用 `AddEnvironmentCleanupHook`:

```cpp
// addon.cc
#include <node.h>
#include <assert.h>
#include <stdlib.h>

using node::AddEnvironmentCleanupHook;
using v8::HandleScope;
using v8::Isolate;
using v8::Local;
using v8::Object;

// Note: In a real-world application, do not rely on static/global data.
static char cookie[] = "yum yum";
static int cleanup_cb1_called = 0;
static int cleanup_cb2_called = 0;

static void cleanup_cb1(void* arg) {
  Isolate* isolate = static_cast<Isolate*>(arg);
  HandleScope scope(isolate);
  Local<Object> obj = Object::New(isolate);
  assert(!obj.IsEmpty());  // assert VM is still alive
  assert(obj->IsObject());
  cleanup_cb1_called++;
}

static void cleanup_cb2(void* arg) {
  assert(arg == static_cast<void*>(cookie));
  cleanup_cb2_called++;
}

static void sanity_check(void*) {
  assert(cleanup_cb1_called == 1);
  assert(cleanup_cb2_called == 1);
}

// Initialize this addon to be context-aware.
NODE_MODULE_INIT(/* exports, module, context */) {
  Isolate* isolate = context->GetIsolate();

  AddEnvironmentCleanupHook(isolate, sanity_check, nullptr);
  AddEnvironmentCleanupHook(isolate, cleanup_cb2, cookie);
  AddEnvironmentCleanupHook(isolate, cleanup_cb1, isolate);
}
```

通过运行在 JavaScript 中进行测试:

```js
// test.js
require('./build/Release/addon');
```

### Building

编写源代码后，必须将其编译到二进制“addon.node”文件中。为此，请在项目的顶层创建一个名为 `binding.gyp` 的文件，使用类似 JSON 的格式描述模块的构建配置。该文件由 [node-gyp][] 使用，这是一个专门为编译 Node.js 插件而编写的工具.

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "hello.cc" ]
    }
  ]
}
```

`node-gyp` 实用程序的一个版本与 Node.js 捆绑并分发，作为 `npm` 的一部分。此版本不直接供开发人员使用，仅用于支持使用 `npm install` 命令编译和安装插件的能力。希望直接使用 `node-gyp` 的开发人员可以使用命令 `npm install -g node-gyp` 安装它。有关更多信息，包括特定于平台的要求，请参阅 `node-gyp` [安装说明][].

创建 `binding.gyp` 文件后，使用 `node-gyp configure` 为当前平台生成适当的项目构建文件。这将在 `build/` 目录中生成 `Makefile`（在 Unix 平台上）或 `vcxproj` 文件（在 Windows 上）.

接下来，调用 `node-gyp build` 命令生成编译后的 `addon.node` 文件。这将被放入`build/Release/`目录.

当使用 `npm install` 安装 Node.js 插件时，npm 使用它自己的 `node-gyp` 捆绑版本来执行相同的一组操作，根据需要为用户的平台生成插件的编译版本.

构建完成后，可以通过将 [`require()`][require] 指向构建的 `addon.node` 模块在 Node.js 中使用二进制插件:

```js
// hello.js
const addon = require('./build/Release/addon');

console.log(addon.hello());
// Prints: 'world'
```

因为编译的插件二进制文件的确切路径可能会因编译方式而异（即有时它可能在 `./build/Debug/` 中），所以插件可以使用 [bindings][] 包来加载已编译的模块.

虽然 `bindings` 包实现在定位插件模块方面更加复杂，但它本质上是使用类似于 `try...catch` 的模式:

```js
try {
  return require('./build/Release/addon.node');
} catch (err) {
  return require('./build/Debug/addon.node');
}
```

### Linking to libraries included with Node.js

Node.js 使用静态链接库，例如 V8、libuv 和 OpenSSL。所有插件都需要链接到 V8，也可以链接到任何其他依赖项。通常，这就像包含适当的 `#include <...>` 语句（例如 `#include <v8.h>`）一样简单，并且 `node-gyp` 将自动定位适当的标头。但是，有一些注意事项需要注意:

* 当 `node-gyp` 运行时，它将检测 Node.js 的特定发行版本并下载完整的源代码压缩包或仅下载标头。如果下载了完整的源代码，插件将拥有对全套 Node.js 依赖项的完全访问权限。但是，如果仅下载 Node.js 标头，则只有 Node.js 导出的符号可用.

* `node-gyp` 可以使用指向本地 Node.js 源图像的 `--nodedir` 标志运行。使用此选项，插件将有权访问完整的依赖项集.

### Loading addons using `require()`

已编译的插件二进制文件的文件扩展名是 `.node`（相对于 `.dll` 或 `.so`）。 [`require()`][require] 函数用于查找具有 `.node` 文件扩展名的文件并将其初始化为动态链接库.

当调用 [`require()`][require] 时，`.node` 扩展通常可以省略，Node.js 仍然会找到并初始化插件。然而，需要注意的是，Node.js 将首先尝试定位和加载碰巧共享相同基本名称的模块或 JavaScript 文件。例如，如果在与二进制文件 `addon.node` 相同的目录中有一个文件 `addon.js`，则 [`require('addon')`][require] 将优先于 `addon.js`文件并加载它.

## Native abstractions for Node.js

本文档中说明的每个示例都直接使用 Node.js 和 V8 API 来实现插件。 V8 API 可以并且已经从一个 V8 版本到下一个版本（以及一个主要的 Node.js 版本到下一个版本）发生了巨大变化。随着每次更改，插件可能需要更新和重新编译才能继续运行。 Node.js 发布计划旨在最大限度地减少此类更改的频率和影响，但 Node.js 几乎无法确保 V8 API 的稳定性.

[Native Abstractions for Node.js][]（或 `nan`）提供了一组工具，建议插件开发人员使用这些工具来保持 V8 和 Node.js 过去和未来版本之间的兼容性。有关如何使用它的说明，请参阅 `nan` [examples][].

## Node-API

> Stability: 2 - Stable

Node-API 是用于构建原生插件的 API。它独立于底层 JavaScript 运行时（例如 V8），并作为 Node.js 本身的一部分进行维护。此 API 将是跨 Node.js 版本稳定的应用程序二进制接口 (ABI)。它旨在将插件与底层 JavaScript 引擎中的更改隔离开来，并允许为一个版本编译的模块在更高版本的 Node.js 上运行而无需重新编译。插件使用本文档中概述的相同方法/工具（node-gyp 等）构建/打包。唯一的区别是本机代码使用的 API 集。不使用 V8 或 [Native Abstractions for Node.js][] API，而是使用 Node-API 中可用的函数.

创建和维护一个受益于 Node-API 提供的 ABI 稳定性的插件带有某些 [实施注意事项] [].

要在上面的“Hello world”示例中使用 Node-API，请将 `hello.cc` 的内容替换为以下内容。所有其他指令保持不变.

```cpp
// hello.cc using Node-API
#include <node_api.h>

namespace demo {

napi_value Method(napi_env env, napi_callback_info args) {
  napi_value greeting;
  napi_status status;

  status = napi_create_string_utf8(env, "world", NAPI_AUTO_LENGTH, &greeting);
  if (status != napi_ok) return nullptr;
  return greeting;
}

napi_value init(napi_env env, napi_value exports) {
  napi_status status;
  napi_value fn;

  status = napi_create_function(env, nullptr, 0, Method, nullptr, &fn);
  if (status != napi_ok) return nullptr;

  status = napi_set_named_property(env, exports, "hello", fn);
  if (status != napi_ok) return nullptr;
  return exports;
}

NAPI_MODULE(NODE_GYP_MODULE_NAME, init)

}  // namespace demo
```

可用的功能以及如何使用它们记录在 [C/C++ addons with Node-API](n-api.md).

## Addon examples

以下是一些旨在帮助开发人员入门的示例插件。这些示例使用 V8 API。有关各种 V8 调用的帮助，请参阅在线 [V8 参考][v8-docs]，并参阅 V8 的 [Embedder's Guide][] 以了解使用的几个概念，例如句柄、作用域、函数模板等.

这些示例中的每一个都使用以下 `binding.gyp` 文件:

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "addon.cc" ]
    }
  ]
}
```

如果有多个 `.cc` 文件，只需将附加文件名添加到 `sources` 数组:

```json
"sources": ["addon.cc", "myexample.cc"]
```

一旦 `binding.gyp` 文件准备好，可以使用 `node-gyp` 配置和构建示例插件:

```console
$ node-gyp configure build
```

### Function arguments

插件通常会公开可以从 Node.js 中运行的 JavaScript 访问的对象和函数。当从 JavaScript 调用函数时,
输入参数和返回值必须与 C/C++ 代码相互映射.

以下示例说明如何读取从 JavaScript 传递的函数参数以及如何返回结果:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Exception;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::String;
using v8::Value;

// This is the implementation of the "add" method
// Input arguments are passed using the
// const FunctionCallbackInfo<Value>& args struct
void Add(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  // Check the number of arguments passed.
  if (args.Length() < 2) {
    // Throw an Error that is passed back to JavaScript
    isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate,
                            "Wrong number of arguments").ToLocalChecked()));
    return;
  }

  // Check the argument types
  if (!args[0]->IsNumber() || !args[1]->IsNumber()) {
    isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate,
                            "Wrong arguments").ToLocalChecked()));
    return;
  }

  // Perform the operation
  double value =
      args[0].As<Number>()->Value() + args[1].As<Number>()->Value();
  Local<Number> num = Number::New(isolate, value);

  // Set the return value (using the passed in
  // FunctionCallbackInfo<Value>&)
  args.GetReturnValue().Set(num);
}

void Init(Local<Object> exports) {
  NODE_SET_METHOD(exports, "add", Add);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

编译后，可以在 Node.js 中需要和使用示例插件:

```js
// test.js
const addon = require('./build/Release/addon');

console.log('This should be eight:', addon.add(3, 5));
```

### Callbacks

插件中的常见做法是将 JavaScript 函数传递给 C++ 函数并从那里执行它们。以下示例说明了如何调用此类回调:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Null;
using v8::Object;
using v8::String;
using v8::Value;

void RunCallback(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();
  Local<Function> cb = Local<Function>::Cast(args[0]);
  const unsigned argc = 1;
  Local<Value> argv[argc] = {
      String::NewFromUtf8(isolate,
                          "hello world").ToLocalChecked() };
  cb->Call(context, Null(isolate), argc, argv).ToLocalChecked();
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", RunCallback);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

此示例使用 `Init()` 的双参数形式，它接收完整的 `module` 对象作为第二个参数。这允许插件使用单个函数完全覆盖 `exports`，而不是将函数添加为 `exports` 的属性.

要对其进行测试，请运行以下 JavaScript:

```js
// test.js
const addon = require('./build/Release/addon');

addon((msg) => {
  console.log(msg);
// Prints: 'hello world'
});
```

在这个例子中，回调函数是同步调用的.

### Object factory

插件可以从 C++ 函数中创建和返回新对象，如下例所示。创建一个对象并返回一个属性“msg”，该属性与传递给“createObject()”的字符串相呼应:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Context;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  Local<Object> obj = Object::New(isolate);
  obj->Set(context,
           String::NewFromUtf8(isolate,
                               "msg").ToLocalChecked(),
                               args[0]->ToString(context).ToLocalChecked())
           .FromJust();

  args.GetReturnValue().Set(obj);
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", CreateObject);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

To test it in JavaScript:

```js
// test.js
const addon = require('./build/Release/addon');

const obj1 = addon('hello');
const obj2 = addon('world');
console.log(obj1.msg, obj2.msg);
// Prints: 'hello world'
```

### Function factory

另一个常见的场景是创建包装 C++ 函数的 JavaScript 函数并将它们返回给 JavaScript:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void MyFunction(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  args.GetReturnValue().Set(String::NewFromUtf8(
      isolate, "hello world").ToLocalChecked());
}

void CreateFunction(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  Local<Context> context = isolate->GetCurrentContext();
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, MyFunction);
  Local<Function> fn = tpl->GetFunction(context).ToLocalChecked();

  // omit this to make it anonymous
  fn->SetName(String::NewFromUtf8(
      isolate, "theFunction").ToLocalChecked());

  args.GetReturnValue().Set(fn);
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", CreateFunction);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

To test:

```js
// test.js
const addon = require('./build/Release/addon');

const fn = addon();
console.log(fn());
// Prints: 'hello world'
```

### Wrapping C++ objects

也可以以允许使用 JavaScript `new` 运算符创建新实例的方式包装 C++ 对象/类:

```cpp
// addon.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using v8::Local;
using v8::Object;

void InitAll(Local<Object> exports) {
  MyObject::Init(exports);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

Then, in `myobject.h`, the wrapper class inherits from `node::ObjectWrap`:

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Local<v8::Object> exports);

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);

  double value_;
};

}  // namespace demo

#endif
```

在 `myobject.cc` 中，实现要公开的各种方法.
下面，通过将方法 `plusOne()` 添加到构造函数的原型中来公开它:

```cpp
// myobject.cc
#include "myobject.h"

namespace demo {

using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::ObjectTemplate;
using v8::String;
using v8::Value;

MyObject::MyObject(double value) : value_(value) {
}

MyObject::~MyObject() {
}

void MyObject::Init(Local<Object> exports) {
  Isolate* isolate = exports->GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  Local<ObjectTemplate> addon_data_tpl = ObjectTemplate::New(isolate);
  addon_data_tpl->SetInternalFieldCount(1);  // 1 field for the MyObject::New()
  Local<Object> addon_data =
      addon_data_tpl->NewInstance(context).ToLocalChecked();

  // Prepare constructor template
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New, addon_data);
  tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject").ToLocalChecked());
  tpl->InstanceTemplate()->SetInternalFieldCount(1);

  // Prototype
  NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

  Local<Function> constructor = tpl->GetFunction(context).ToLocalChecked();
  addon_data->SetInternalField(0, constructor);
  exports->Set(context, String::NewFromUtf8(
      isolate, "MyObject").ToLocalChecked(),
      constructor).FromJust();
}

void MyObject::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  if (args.IsConstructCall()) {
    // Invoked as constructor: `new MyObject(...)`
    double value = args[0]->IsUndefined() ?
        0 : args[0]->NumberValue(context).FromMaybe(0);
    MyObject* obj = new MyObject(value);
    obj->Wrap(args.This());
    args.GetReturnValue().Set(args.This());
  } else {
    // Invoked as plain function `MyObject(...)`, turn into construct call.
    const int argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Function> cons =
        args.Data().As<Object>()->GetInternalField(0).As<Function>();
    Local<Object> result =
        cons->NewInstance(context, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(result);
  }
}

void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
  obj->value_ += 1;

  args.GetReturnValue().Set(Number::New(isolate, obj->value_));
}

}  // namespace demo
```

要构建此示例，必须将 `myobject.cc` 文件添加到 `binding.gyp`:

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [
        "addon.cc",
        "myobject.cc"
      ]
    }
  ]
}
```

Test it with:

```js
// test.js
const addon = require('./build/Release/addon');

const obj = new addon.MyObject(10);
console.log(obj.plusOne());
// Prints: 11
console.log(obj.plusOne());
// Prints: 12
console.log(obj.plusOne());
// Prints: 13
```

包装对象的析构函数将在对象被垃圾回收时运行。对于析构函数测试，可以使用命令行标志来强制进行垃圾回收。这些标志由底层 V8 JavaScript 引擎提供。它们可能随时更改或删除。它们没有被 Node.js 或 V8 记录，它们不应该在测试之外使用.

在进程或工作线程关闭期间，JS 引擎不会调用析构函数。因此，用户有责任跟踪这些对象并确保正确销毁以避免资源泄漏.

### Factory of wrapped objects

或者，可以使用工厂模式来避免使用 JavaScript `new` 运算符显式创建对象实例:

```js
const obj = addon.createObject();
// instead of:
// const obj = new addon.Object();
```

First, the `createObject()` method is implemented in `addon.cc`:

```cpp
// addon.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  MyObject::NewInstance(args);
}

void InitAll(Local<Object> exports, Local<Object> module) {
  MyObject::Init(exports->GetIsolate());

  NODE_SET_METHOD(module, "exports", CreateObject);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

在 `myobject.h` 中，添加了静态方法 `NewInstance()` 来处理实例化对象。这个方法代替了在 JavaScript 中使用 `new`:

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Isolate* isolate);
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Global<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

`myobject.cc` 中的实现与前面的示例类似:

```cpp
// myobject.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using node::AddEnvironmentCleanupHook;
using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Global;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::String;
using v8::Value;

// Warning! This is not thread-safe, this addon cannot be used for worker
// threads.
Global<Function> MyObject::constructor;

MyObject::MyObject(double value) : value_(value) {
}

MyObject::~MyObject() {
}

void MyObject::Init(Isolate* isolate) {
  // Prepare constructor template
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
  tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject").ToLocalChecked());
  tpl->InstanceTemplate()->SetInternalFieldCount(1);

  // Prototype
  NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

  Local<Context> context = isolate->GetCurrentContext();
  constructor.Reset(isolate, tpl->GetFunction(context).ToLocalChecked());

  AddEnvironmentCleanupHook(isolate, [](void*) {
    constructor.Reset();
  }, nullptr);
}

void MyObject::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  if (args.IsConstructCall()) {
    // Invoked as constructor: `new MyObject(...)`
    double value = args[0]->IsUndefined() ?
        0 : args[0]->NumberValue(context).FromMaybe(0);
    MyObject* obj = new MyObject(value);
    obj->Wrap(args.This());
    args.GetReturnValue().Set(args.This());
  } else {
    // Invoked as plain function `MyObject(...)`, turn into construct call.
    const int argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Object> instance =
        cons->NewInstance(context, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(instance);
  }
}

void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  const unsigned argc = 1;
  Local<Value> argv[argc] = { args[0] };
  Local<Function> cons = Local<Function>::New(isolate, constructor);
  Local<Context> context = isolate->GetCurrentContext();
  Local<Object> instance =
      cons->NewInstance(context, argc, argv).ToLocalChecked();

  args.GetReturnValue().Set(instance);
}

void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
  obj->value_ += 1;

  args.GetReturnValue().Set(Number::New(isolate, obj->value_));
}

}  // namespace demo
```

再一次，要构建这个例子，必须将 `myobject.cc` 文件添加到 `binding.gyp`:

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [
        "addon.cc",
        "myobject.cc"
      ]
    }
  ]
}
```

Test it with:

```js
// test.js
const createObject = require('./build/Release/addon');

const obj = createObject(10);
console.log(obj.plusOne());
// Prints: 11
console.log(obj.plusOne());
// Prints: 12
console.log(obj.plusOne());
// Prints: 13

const obj2 = createObject(20);
console.log(obj2.plusOne());
// Prints: 21
console.log(obj2.plusOne());
// Prints: 22
console.log(obj2.plusOne());
// Prints: 23
```

### Passing wrapped objects around

除了包装和返回 C++ 对象之外，还可以通过使用 Node.js 帮助函数 `node::ObjectWrap::Unwrap` 展开包装的对象来传递包装的对象。以下示例显示了一个函数 `add()`，它可以将两个 `MyObject` 对象作为输入参数:

```cpp
// addon.cc
#include <node.h>
#include <node_object_wrap.h>
#include "myobject.h"

namespace demo {

using v8::Context;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  MyObject::NewInstance(args);
}

void Add(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  MyObject* obj1 = node::ObjectWrap::Unwrap<MyObject>(
      args[0]->ToObject(context).ToLocalChecked());
  MyObject* obj2 = node::ObjectWrap::Unwrap<MyObject>(
      args[1]->ToObject(context).ToLocalChecked());

  double sum = obj1->value() + obj2->value();
  args.GetReturnValue().Set(Number::New(isolate, sum));
}

void InitAll(Local<Object> exports) {
  MyObject::Init(exports->GetIsolate());

  NODE_SET_METHOD(exports, "createObject", CreateObject);
  NODE_SET_METHOD(exports, "add", Add);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

在 `myobject.h` 中，添加了一个新的公共方法以允许在展开对象后访问私有值.

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Isolate* isolate);
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);
  inline double value() const { return value_; }

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Global<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

`myobject.cc` 的实现与之前类似:

```cpp
// myobject.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using node::AddEnvironmentCleanupHook;
using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Global;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

// Warning! This is not thread-safe, this addon cannot be used for worker
// threads.
Global<Function> MyObject::constructor;

MyObject::MyObject(double value) : value_(value) {
}

MyObject::~MyObject() {
}

void MyObject::Init(Isolate* isolate) {
  // Prepare constructor template
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
  tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject").ToLocalChecked());
  tpl->InstanceTemplate()->SetInternalFieldCount(1);

  Local<Context> context = isolate->GetCurrentContext();
  constructor.Reset(isolate, tpl->GetFunction(context).ToLocalChecked());

  AddEnvironmentCleanupHook(isolate, [](void*) {
    constructor.Reset();
  }, nullptr);
}

void MyObject::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Context> context = isolate->GetCurrentContext();

  if (args.IsConstructCall()) {
    // Invoked as constructor: `new MyObject(...)`
    double value = args[0]->IsUndefined() ?
        0 : args[0]->NumberValue(context).FromMaybe(0);
    MyObject* obj = new MyObject(value);
    obj->Wrap(args.This());
    args.GetReturnValue().Set(args.This());
  } else {
    // Invoked as plain function `MyObject(...)`, turn into construct call.
    const int argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Object> instance =
        cons->NewInstance(context, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(instance);
  }
}

void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  const unsigned argc = 1;
  Local<Value> argv[argc] = { args[0] };
  Local<Function> cons = Local<Function>::New(isolate, constructor);
  Local<Context> context = isolate->GetCurrentContext();
  Local<Object> instance =
      cons->NewInstance(context, argc, argv).ToLocalChecked();

  args.GetReturnValue().Set(instance);
}

}  // namespace demo
```

Test it with:

```js
// test.js
const addon = require('./build/Release/addon');

const obj1 = addon.createObject(10);
const obj2 = addon.createObject(20);
const result = addon.add(obj1, obj2);

console.log(result);
// Prints: 30
```

[Electron]: https://electronjs.org/
[Embedder's Guide]: https://v8.dev/docs/embed
[Linking to libraries included with Node.js]: #linking-to-libraries-included-with-nodejs
[Native Abstractions for Node.js]: https://github.com/nodejs/nan
[V8]: https://v8.dev/
[`Worker`]: worker_threads.md#class-worker
[bindings]: https://github.com/TooTallNate/node-bindings
[download]: https://github.com/nodejs/node-addon-examples
[examples]: https://github.com/nodejs/nan/tree/HEAD/examples/
[implementation considerations]: n-api.md#implications-of-abi-stability
[installation instructions]: https://github.com/nodejs/node-gyp#installation
[libuv]: https://github.com/libuv/libuv
[node-gyp]: https://github.com/nodejs/node-gyp
[require]: modules.md#requireid
[v8-docs]: https://v8docs.nodesource.com/
