# C++嵌入器 API

<!--introduced_in=v12.19.0-->

Node.js提供了许多可用于执行 JavaScript 的C++ API
在 Node.js环境中，从其他C++软件。

这些 API 的文档可以在[src/node.h][]在节点中.js
源树。除了 Node.js 公开的 API 之外，还有一些必需的概念
由 V8 嵌入器 API 提供。

因为使用 Node.js 作为嵌入式库与编写代码不同
由 Node 执行.js，重大更改不遵循典型的 Node.js
[弃用策略][deprecation policy]并且可能在每个主要版本中发生，而无需事先
警告。

## 嵌入应用程序示例

以下各节将概述如何使用这些 API
从头开始创建一个应用程序，该应用程序将执行等效于
`node -e <code>`，即它将采用一段JavaScript并将其运行在
特定于节点.js环境。

可以找到完整的代码[在节点.js源树中][embedtest.cc].

### 设置每个进程的状态

Node.js 需要一些每个进程的状态管理才能运行：

*   节点的参数解析.js[CLI 选项][CLI options],
*   V8 每个进程的要求，例如`v8::Platform`实例。

下面的示例演示如何设置这些设置。一些类名来自
这`node`和`v8`分别C++命名空间。

```cpp
int main(int argc, char** argv) {
  argv = uv_setup_args(argc, argv);
  std::vector<std::string> args(argv, argv + argc);
  // Parse Node.js CLI options, and print any errors that have occurred while
  // trying to parse them.
  std::unique_ptr<node::InitializationResult> result =
      node::InitializeOncePerProcess(args, {
        node::ProcessInitializationFlags::kNoInitializeV8,
        node::ProcessInitializationFlags::kNoInitializeNodeV8Platform
      });

  for (const std::string& error : result->errors())
    fprintf(stderr, "%s: %s\n", args[0].c_str(), error.c_str());
  if (result->early_return() != 0) {
    return result->exit_code();
  }

  // Create a v8::Platform instance. `MultiIsolatePlatform::Create()` is a way
  // to create a v8::Platform instance that Node.js can use when creating
  // Worker threads. When no `MultiIsolatePlatform` instance is present,
  // Worker threads are disabled.
  std::unique_ptr<MultiIsolatePlatform> platform =
      MultiIsolatePlatform::Create(4);
  V8::InitializePlatform(platform.get());
  V8::Initialize();

  // See below for the contents of this function.
  int ret = RunNodeInstance(
      platform.get(), result->args(), result->exec_args());

  V8::Dispose();
  V8::DisposePlatform();

  node::TearDownOncePerProcess();
  return ret;
}
```

### 每个实例的状态

<!-- YAML
changes:
  - version: v15.0.0
    pr-url: https://github.com/nodejs/node/pull/35597
    description:
      The `CommonEnvironmentSetup` and `SpinEventLoop` utilities were added.
-->

Node.js具有“Node.js实例”的概念，通常被引用
作为`node::Environment`.每`node::Environment`与以下各项相关：

*   正好一个`v8::Isolate`，即一个 JS 引擎实例，
*   正好一个`uv_loop_t`，即一个事件循环，以及
*   一些`v8::Context`s，但正好是一个主要`v8::Context`.
*   一`node::IsolateData`包含以下信息的实例
    由多个共享`node::Environment`使用相同的`v8::Isolate`.
    目前，如果对此方案执行任何测试，则不会执行任何测试。

为了设置`v8::Isolate`一`v8::ArrayBuffer::Allocator`需要
提供。一个可能的选择是默认的 Node.js 分配器，它
可以通过以下方式创建`node::ArrayBufferAllocator::Create()`.使用节点.js
分配器允许在插件使用 Node 时进行少量性能优化.js
C++`Buffer`API，并且是必需的，以便跟踪`ArrayBuffer`内存
[`process.memoryUsage()`][process.memoryUsage()].

此外，每个`v8::Isolate`用于节点.js实例需要
在`MultiIsolatePlatform`实例，如果有
正在使用，以便平台知道要使用哪个事件循环
对于 由`v8::Isolate`.

这`node::NewIsolate()`帮助器函数创建一个`v8::Isolate`,
使用一些特定于节点.js钩子（例如Node.js错误处理程序）对其进行设置，
并自动将其注册到平台。

```cpp
int RunNodeInstance(MultiIsolatePlatform* platform,
                    const std::vector<std::string>& args,
                    const std::vector<std::string>& exec_args) {
  int exit_code = 0;

  // Setup up a libuv event loop, v8::Isolate, and Node.js Environment.
  std::vector<std::string> errors;
  std::unique_ptr<CommonEnvironmentSetup> setup =
      CommonEnvironmentSetup::Create(platform, &errors, args, exec_args);
  if (!setup) {
    for (const std::string& err : errors)
      fprintf(stderr, "%s: %s\n", args[0].c_str(), err.c_str());
    return 1;
  }

  Isolate* isolate = setup->isolate();
  Environment* env = setup->env();

  {
    Locker locker(isolate);
    Isolate::Scope isolate_scope(isolate);
    HandleScope handle_scope(isolate);
    // The v8::Context needs to be entered when node::CreateEnvironment() and
    // node::LoadEnvironment() are being called.
    Context::Scope context_scope(setup->context());

    // Set up the Node.js instance for execution, and run code inside of it.
    // There is also a variant that takes a callback and provides it with
    // the `require` and `process` objects, so that it can manually compile
    // and run scripts as needed.
    // The `require` function inside this script does *not* access the file
    // system, and can only load built-in Node.js modules.
    // `module.createRequire()` is being used to create one that is able to
    // load files from the disk, and uses the standard CommonJS file loader
    // instead of the internal-only `require` function.
    MaybeLocal<Value> loadenv_ret = node::LoadEnvironment(
        env,
        "const publicRequire ="
        "  require('node:module').createRequire(process.cwd() + '/');"
        "globalThis.require = publicRequire;"
        "require('node:vm').runInThisContext(process.argv[1]);");

    if (loadenv_ret.IsEmpty())  // There has been a JS exception.
      return 1;

    exit_code = node::SpinEventLoop(env).FromMaybe(1);

    // node::Stop() can be used to explicitly stop the event loop and keep
    // further JavaScript from running. It can be called from any thread,
    // and will act like worker.terminate() if called from another thread.
    node::Stop(env);
  }

  return exit_code;
}
```

[CLI options]: cli.md

[`process.memoryUsage()`]: process.md#processmemoryusage

[deprecation policy]: deprecations.md

[embedtest.cc]: https://github.com/nodejs/node/blob/HEAD/test/embedding/embedtest.cc

[src/node.h]: https://github.com/nodejs/node/blob/HEAD/src/node.h
