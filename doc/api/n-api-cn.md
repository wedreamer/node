# Node-API

<!--introduced_in=v8.0.0-->

<!-- type=misc -->

> 稳定性： 2 - 稳定

Node-API（以前称为N-API）是用于构建本机插件的API。是的
独立于底层 JavaScript 运行时（例如，V8），并且是
作为 Node 本身的一部分进行维护.js。此 API 将是应用程序二进制文件
接口 （ABI） 在 Node.js 的版本中保持稳定。它旨在绝缘
底层 JavaScript 引擎中的更改中的插件，并允许模块
针对一个主要版本编译，以便在 Node 的更高主要版本上运行.js没有
重新 编译。这[ABI 稳定性][ABI Stability]指南提供了更深入的解释。

插件是使用本节中概述的相同方法/工具构建/打包的
题为[C++ 插件][C++ Addons].唯一的区别是使用的 API 集
本机代码。而不是使用 V8 或[Node 的本机抽象.js][Native Abstractions for Node.js]
API，使用Node-API中可用的函数。

Node-API 公开的 API 通常用于创建和操作
JavaScript 值。概念和操作通常映射到指定的想法
在 ECMA-262 语言规范中。这些 API 具有以下特性
性能：

*   所有 Node-API 调用都返回类型为类型的状态代码`napi_status`.这
    状态指示 API 调用是成功还是失败。
*   API 的返回值通过 out 参数传递。
*   所有 JavaScript 值都抽象在名为
    `napi_value`.
*   如果出现错误状态代码，可以获得其他信息
    用`napi_get_last_error_info`.可以在错误中找到更多信息
    处理部分[错误处理][Error handling].

Node-API 是一个 C API，可确保跨 Node.js 版本的 ABI 稳定性
和不同的编译器级别。C++ API 可以更易于使用。
为了支持使用C++，项目维护一个
C++包装模块称为[`node-addon-api`][node-addon-api].
此包装器提供可内联的C++ API。构建的二进制文件
跟`node-addon-api`将取决于基于节点 API C 的符号
由 Node 导出的函数.js。`node-addon-api`是一个更多
编写调用 Node-API 的代码的有效方法。例如，
以后`node-addon-api`法典。第一部分显示了
`node-addon-api`代码和第二部分显示了实际得到的内容
在插件中使用。

```cpp
Object obj = Object::New(env);
obj["foo"] = String::New(env, "bar");
```

```cpp
napi_status status;
napi_value object, string;
status = napi_create_object(env, &object);
if (status != napi_ok) {
  napi_throw_error(env, ...);
  return;
}

status = napi_create_string_utf8(env, "bar", NAPI_AUTO_LENGTH, &string);
if (status != napi_ok) {
  napi_throw_error(env, ...);
  return;
}

status = napi_set_named_property(env, object, "foo", string);
if (status != napi_ok) {
  napi_throw_error(env, ...);
  return;
}
```

最终结果是插件仅使用导出的 C API。因此，
它仍然受益于C API提供的ABI稳定性。

使用时`node-addon-api`而不是 C API，从 API 开始[文档][docs]
为`node-addon-api`.

这[节点 API 资源](https://nodejs.github.io/node-addon-examples/)提供
为刚入门的开发人员提供出色的方向和提示
Node-API 和`node-addon-api`.

## ABI稳定性的影响

尽管Node-API提供了ABI稳定性保证，但Node.js的其他部分确实如此。
不是，从插件使用的任何外部库都可能不会。特别
以下 API 均未在主要 API 中提供 ABI 稳定性保证
版本：

*   节点.js C++可通过以下任一方式获得的 API

    ```cpp
    #include <node.h>
    #include <node_buffer.h>
    #include <node_version.h>
    #include <node_object_wrap.h>
    ```

*   libuv API 也包含在 Node.js 中，可通过

    ```cpp
    #include <uv.h>
    ```

*   V8 API 可通过

    ```cpp
    #include <v8.h>
    ```

因此，为了使插件在Node.js主要版本之间保持ABI兼容，它
必须通过限制自身使用来独占使用 Node-API

```c
#include <node_api.h>
```

并且通过检查它使用的所有外部库，外部库
库使ABI稳定性保证类似于Node-API。

## 建筑

与用JavaScript编写的模块不同，开发和部署Node.js
使用 Node-API 的本机插件需要一组额外的工具。除了
为Node开发所需的基本工具.js，本机插件开发人员
需要一个可以将C编译并将代码C++二进制文件的工具链。在
另外，根据本机插件的部署方式，*用户*之
原生插件还需要安装C / C++工具链。

对于 Linux 开发人员来说，必要的 C/C++ 工具链包很容易
可用。[海湾合作委员会][GCC]广泛用于节点.js社区来构建和
跨各种平台进行测试。对于许多开发人员来说，[LLVM][]
编译器基础结构也是一个不错的选择。

对于 Mac 开发者来说，[Xcode][]提供所有必需的编译器工具。
但是，没有必要安装整个 Xcode IDE。以下
命令安装必要的工具链：

```bash
xcode-select --install
```

对于 Windows 开发人员，[Visual Studio][]提供所有必需的编译器
工具。但是，没有必要安装整个Visual Studio。
艾德。以下命令将安装必要的工具链：

```bash
npm install --global windows-build-tools
```

以下各节介绍了可用于开发的其他工具
并部署 Node.js本机插件。

### 构建工具

此处列出的两种工具都要求*用户*的本地人
插件安装了C / C++工具链才能成功安装
本机插件。

#### 节点-gyp

[节点-gyp][node-gyp]是一个基于[吉普-下一个][gyp-next]分叉
谷歌的[吉普][GYP]工具，并与 npm 捆绑在一起。GYP，因此节点-gyp，
要求安装 Python。

从历史上看，node-gyp一直是构建原生工具的首选工具。
插件。它具有广泛的采用和文档记录。但是，有些
开发人员在 node-gyp 中遇到了限制。

#### 咔嚓.js

[咔嚓.js][CMake.js]是一个基于[咔咔咔][CMake].

CMake.js对于已经使用CMake或用于的项目来说是一个不错的选择
受 node-gyp 限制影响的开发人员。

### 上载预编译的二进制文件

这里列出的三个工具允许本地插件开发人员和维护者
创建二进制文件并将其上载到公共或专用服务器。这些工具是
通常与 CI/CD 构建系统集成，如[特拉维斯·西][Travis CI]和
[AppVeyor][]为各种平台构建和上传二进制文件，以及
架构。然后，这些二进制文件可供以下用户下载：
不需要安装 C/C++ 工具链。

#### 节点预 gyp

[节点预 gyp][node-pre-gyp]是一种基于 node-gyp 的工具，它增加了
将二进制文件上载到开发人员选择的服务器。节点预 gyp 具有
特别支持将二进制文件上传到 Amazon S3。

#### 预构建

[预构建][prebuild]是一种支持使用 node-gyp 或
CMake.js. 与支持各种服务器的节点预 gyp 不同，预构建
仅将二进制文件上载到[GitHub 版本][GitHub releases].预构建是一个不错的选择
GitHub 项目使用 CMake.js。

#### 预构建

[预构建][prebuildify]是一个基于 node-gyp 的工具。预建的优点是
构建的二进制文件与本机插件捆绑在一起，当它是
已上载到 npm。二进制文件从 npm 下载并立即
在安装本机插件时可供模块用户使用。

## 用法

要使用 Node-API 函数，请包含该文件[`node_api.h`][node_api.h]哪
位于节点开发树的 src 目录中：

```c
#include <node_api.h>
```

这将选择加入默认值`NAPI_VERSION`对于节点的给定版本.js。
为了确保与特定版本的 Node-API 兼容，该版本
在包含标头时可以显式指定：

```c
#define NAPI_VERSION 3
#include <node_api.h>
```

这会将 Node-API 图面限制为仅可用的功能
在指定（和更早）版本中。

一些 Node-API 表面是实验性的，需要显式选择加入：

```c
#define NAPI_EXPERIMENTAL
#include <node_api.h>
```

在这种情况下，整个 API 图面（包括任何实验性 API）将为
可用于模块代码。

## 节点 API 版本矩阵

Node-API 版本是累加的，并且独立于 Node.js进行版本控制。
版本 4 是版本 3 的扩展，因为它具有所有 API
从版本3开始，并添加了一些内容。这意味着没有必要
重新编译新版本的 Node.js这些版本是
列为支持更高版本。

<!-- For accessibility purposes, this table needs row headers. That means we
     can't do it in markdown. Hence, the raw HTML. -->

<table>
  <tr>
    <td></td>
    <th scope="col">1</th>
    <th scope="col">2</th>
    <th scope="col">3</th>
  </tr>
  <tr>
    <th scope="row">v6.x</th>
    <td></td>
    <td></td>
    <td>v6.14.2*</td>
  </tr>
  <tr>
    <th scope="row">v8.x</th>
    <td>v8.6.0**</td>
    <td>v8.10.0*</td>
    <td>v8.11.2</td>
  </tr>
  <tr>
    <th scope="row">v9.x</th>
    <td>v9.0.0*</td>
    <td>v9.3.0*</td>
    <td>v9.11.0*</td>
  </tr>
  <tr>
    <th scope="row">≥ v10.x</th>
    <td>all releases</td>
    <td>all releases</td>
    <td>all releases</td>
  </tr>
</table>

<table>
  <tr>
    <td></td>
    <th scope="col">4</th>
    <th scope="col">5</th>
    <th scope="col">6</th>
    <th scope="col">7</th>
    <th scope="col">8</th>
  </tr>
  <tr>
    <th scope="row">v10.x</th>
    <td>v10.16.0</td>
    <td>v10.17.0</td>
    <td>v10.20.0</td>
    <td>v10.23.0</td>
    <td></td>
  </tr>
  <tr>
    <th scope="row">v11.x</th>
    <td>v11.8.0</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th scope="row">v12.x</th>
    <td>v12.0.0</td>
    <td>v12.11.0</td>
    <td>v12.17.0</td>
    <td>v12.19.0</td>
    <td>v12.22.0</td>
  </tr>
  <tr>
    <th scope="row">v13.x</th>
    <td>v13.0.0</td>
    <td>v13.0.0</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <th scope="row">v14.x</th>
    <td>v14.0.0</td>
    <td>v14.0.0</td>
    <td>v14.0.0</td>
    <td>v14.12.0</td>
    <td>v14.17.0</td>
  </tr>
  <tr>
    <th scope="row">v15.x</th>
    <td>v15.0.0</td>
    <td>v15.0.0</td>
    <td>v15.0.0</td>
    <td>v15.0.0</td>
    <td>v15.12.0</td>
  </tr>
  <tr>
    <th scope="row">v16.x</th>
    <td>v16.0.0</td>
    <td>v16.0.0</td>
    <td>v16.0.0</td>
    <td>v16.0.0</td>
    <td>v16.0.0</td>
  </tr>
</table>

\* Node-API是实验性的。

\*\* Node.js 8.0.0 包含 Node-API 作为实验性。它被释放为
Node-API 版本 1，但继续发展，直到 Node.js 8.6.0。该接口是
在 Node.js 8.6.0 之前的版本有所不同。我们建议使用 Node-API 版本 3 或
后。

为 Node-API 记录的每个 API 都将有一个名为`added in:`和 API
稳定的将具有额外的标头`Node-API version:`.
API 在使用支持 Node.js 版本时可直接使用
节点 API 版本显示在`Node-API version:`或更高。
使用节点时.js不支持
`Node-API version:`列出或如果没有`Node-API version:`上市
那么API只有在以下情况下才可用
`#define NAPI_EXPERIMENTAL`在包含之前`node_api.h`
或`js_native_api.h`.如果某个 API 似乎在 上不可用
Node 的一个版本.js它晚于`added in:`然后
这很可能是明显缺席的原因。

与从本机访问 ECMAScript 功能严格关联的 Node-API
代码可以在`js_native_api.h`和`js_native_api_types.h`.
这些标头中定义的 API 包含在`node_api.h`和
`node_api_types.h`.标头以这种方式构建，以便允许
Node-API 在 Node.js 之外的实现。对于这些实现，
节点.js特定的 API 可能不适用。

插件.js节点特定的部分可以与代码分离
向 JavaScript 环境公开实际功能，以便
后者可以与Node-API的多个实现一起使用。在示例中
下面`addon.c`和`addon.h`仅参考`js_native_api.h`.这确保了
那`addon.c`可以重用以针对节点进行编译.js
Node-API的实现或Node-API在Node.js之外的任何实现。

`addon_node.c`是一个单独的文件，其中包含 Node.js特定入口点
到插件，并通过调用`addon.c`当
插件被加载到节点.js环境中。

```c
// addon.h
#ifndef _ADDON_H_
#define _ADDON_H_
#include <js_native_api.h>
napi_value create_addon(napi_env env);
#endif  // _ADDON_H_
```

```c
// addon.c
#include "addon.h"

#define NAPI_CALL(env, call)                                      \
  do {                                                            \
    napi_status status = (call);                                  \
    if (status != napi_ok) {                                      \
      const napi_extended_error_info* error_info = NULL;          \
      napi_get_last_error_info((env), &error_info);               \
      const char* err_message = error_info->error_message;        \
      bool is_pending;                                            \
      napi_is_exception_pending((env), &is_pending);              \
      if (!is_pending) {                                          \
        const char* message = (err_message == NULL)               \
            ? "empty error message"                               \
            : err_message;                                        \
        napi_throw_error((env), NULL, message);                   \
        return NULL;                                              \
      }                                                           \
    }                                                             \
  } while(0)

static napi_value
DoSomethingUseful(napi_env env, napi_callback_info info) {
  // Do something useful.
  return NULL;
}

napi_value create_addon(napi_env env) {
  napi_value result;
  NAPI_CALL(env, napi_create_object(env, &result));

  napi_value exported_function;
  NAPI_CALL(env, napi_create_function(env,
                                      "doSomethingUseful",
                                      NAPI_AUTO_LENGTH,
                                      DoSomethingUseful,
                                      NULL,
                                      &exported_function));

  NAPI_CALL(env, napi_set_named_property(env,
                                         result,
                                         "doSomethingUseful",
                                         exported_function));

  return result;
}
```

```c
// addon_node.c
#include <node_api.h>
#include "addon.h"

NAPI_MODULE_INIT() {
  // This function body is expected to return a `napi_value`.
  // The variables `napi_env env` and `napi_value exports` may be used within
  // the body, as they are provided by the definition of `NAPI_MODULE_INIT()`.
  return create_addon(env);
}
```

## 环境生命周期 API

[第8.7节][Section 8.7]的[ECMAScript 语言规范][ECMAScript Language Specification]定义概念
将“代理”作为运行 JavaScript 代码的独立环境。
可以同时启动和终止多个此类代理，也可以在
按进程排序。

Node.js环境对应于 ECMAScript 代理。在主要过程中，
在启动时创建一个环境，并且可以创建其他环境
在单独的线程上用作[工作线程][worker threads].当节点.js嵌入到
另一个应用程序，该应用程序的主线程也可以构造和
在节点.js生命周期中多次破坏节点环境
应用程序进程使得每个 Node.js由
反过来，应用程序可能在其生命周期中创建和销毁其他
环境作为工作线程。

从本机插件的角度来看，这意味着它提供的绑定
可以调用多个，从多个上下文，甚至同时从
多个线程。

本机插件可能需要分配它们在
它们的整个生命周期使得状态对于每个实例都必须是唯一的
插件。

为此，Node-API提供了一种分配数据的方法，使其生命周期
与代理的生命周期相关联。

### `napi_set_instance_data`

<!-- YAML
added:
 - v12.8.0
 - v10.20.0
napiVersion: 6
-->

```c
napi_status napi_set_instance_data(napi_env env,
                                   void* data,
                                   napi_finalize finalize_cb,
                                   void* finalize_hint);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] data`：要提供给此实例的绑定的数据项。
*   `[in] finalize_cb`：环境被撕裂时要调用的函数
    下。该函数接收`data`这样它就可以释放它。
    [`napi_finalize`][napi_finalize]提供了更多详细信息。
*   `[in] finalize_hint`：在
    收集。

返回`napi_ok`如果 API 成功。

此 API 关联`data`使用当前正在运行的代理。`data`以后可以
检索使用`napi_get_instance_data()`.与 关联的任何现有数据
当前正在运行的代理，该代理是通过先前的调用设置为
`napi_set_instance_data()`将被覆盖。如果`finalize_cb`已提供
通过上一次调用，它不会被调用。

### `napi_get_instance_data`

<!-- YAML
added:
 - v12.8.0
 - v10.20.0
napiVersion: 6
-->

```c
napi_status napi_get_instance_data(napi_env env,
                                   void** data);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[out] data`：以前与当前
    通过调用`napi_set_instance_data()`.

返回`napi_ok`如果 API 成功。

此 API 检索以前与当前关联的数据
运行代理通过`napi_set_instance_data()`.如果未设置任何数据，则调用将
成功和`data`将设置为`NULL`.

## 基本节点 API 数据类型

Node-API 将以下基本数据类型公开为抽象，这些抽象是
由各种 API 消耗。这些 API 应被视为不透明，
只能通过其他 Node-API 调用进行内省。

### `napi_status`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

指示节点 API 调用成功或失败的整体状态代码。
目前，支持以下状态代码。

```c
typedef enum {
  napi_ok,
  napi_invalid_arg,
  napi_object_expected,
  napi_string_expected,
  napi_name_expected,
  napi_function_expected,
  napi_number_expected,
  napi_boolean_expected,
  napi_array_expected,
  napi_generic_failure,
  napi_pending_exception,
  napi_cancelled,
  napi_escape_called_twice,
  napi_handle_scope_mismatch,
  napi_callback_scope_mismatch,
  napi_queue_full,
  napi_closing,
  napi_bigint_expected,
  napi_date_expected,
  napi_arraybuffer_expected,
  napi_detachable_arraybuffer_expected,
  napi_would_deadlock,  /* unused */
} napi_status;
```

如果在 API 返回失败状态时需要其他信息，
它可以通过调用`napi_get_last_error_info`.

### `napi_extended_error_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
typedef struct {
  const char* error_message;
  void* engine_reserved;
  uint32_t engine_error_code;
  napi_status error_code;
} napi_extended_error_info;
```

*   `error_message`：UTF8 编码的字符串，其中包含
    错误。
*   `engine_reserved`：为特定于 VM 的错误详细信息保留。这是目前
    未针对任何 VM 实现。
*   `engine_error_code`：特定于虚拟机的错误代码。这是目前
    未针对任何 VM 实现。
*   `error_code`：源自上一个错误的节点 API 状态代码。

查看[错误处理][Error handling]部分以获取更多信息。

### `napi_env`

`napi_env`用于表示底层 Node-API 的上下文
实现可用于保留特定于 VM 的状态。此结构已传递
当调用本机函数时，它们被调用时，它必须被传递回去
进行节点 API 调用。具体来说，相同`napi_env`当传递时
调用的初始本机函数必须传递给任何后续函数
嵌套的 Node-API 调用。缓存`napi_env`为了一般再利用的目的，
并通过`napi_env`在 上运行的同一插件的实例之间
不同[`Worker`][Worker]不允许使用线程。这`napi_env`变得无效
当卸载本机插件的实例时。此事件的通知是
通过提供给[`napi_add_env_cleanup_hook`][napi_add_env_cleanup_hook]和
[`napi_set_instance_data`][napi_set_instance_data].

### `napi_value`

这是一个不透明的指针，用于表示 JavaScript 值。

### `napi_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

这是一个不透明的指针，表示一个JavaScript函数，它可以是
通过从多个线程异步调用
`napi_call_threadsafe_function()`.

### `napi_threadsafe_function_release_mode`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

要赋予的值`napi_release_threadsafe_function()`以指示是否
线程安全函数将立即关闭 （`napi_tsfn_abort`） 或
仅发布（`napi_tsfn_release`），因此可用于后续使用
`napi_acquire_threadsafe_function()`和`napi_call_threadsafe_function()`.

```c
typedef enum {
  napi_tsfn_release,
  napi_tsfn_abort
} napi_threadsafe_function_release_mode;
```

### `napi_threadsafe_function_call_mode`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

要赋予的值`napi_call_threadsafe_function()`以指示是否
每当队列与线程安全相关联时，调用应阻塞
功能已满。

```c
typedef enum {
  napi_tsfn_nonblocking,
  napi_tsfn_blocking
} napi_threadsafe_function_call_mode;
```

### 节点 API 内存管理类型

#### `napi_handle_scope`

这是一个抽象，用于控制和修改对象的生存期
在特定范围内创建。通常，创建节点 API 值
在句柄范围的上下文中。当从
JavaScript，一个默认的句柄作用域将存在。如果用户没有显式
创建新的句柄作用域，节点 API 值将在默认句柄中创建
范围。对于在执行本机方法之外的任何代码调用
（例如，在 libuv 回调调用期间），模块需要
在调用任何可能导致创建的函数之前创建作用域
的 JavaScript 值。

句柄作用域是使用[`napi_open_handle_scope`][napi_open_handle_scope]并被摧毁
用[`napi_close_handle_scope`][napi_close_handle_scope].关闭范围可以向GC指示
所有`napi_value`在句柄范围的生存期内创建的 s 是 no
从当前堆栈帧引用的更长。

有关更多详细信息，请查看[对象生存期管理][Object lifetime management].

#### `napi_escapable_handle_scope`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

可转义句柄作用域是一种特殊类型的句柄作用域，用于返回值
在父范围的特定句柄范围内创建。

#### `napi_ref`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

这是用于引用`napi_value`.这允许
用户来管理 JavaScript 值的生存期，包括定义它们的
显式的最小生存期。

有关更多详细信息，请查看[对象生存期管理][Object lifetime management].

#### `napi_type_tag`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
napiVersion: 8
-->

存储为两个无符号 64 位整数的 128 位值。它充当 UUID
用哪些 JavaScript 对象可以“标记”，以确保它们是
某种类型的。这是一个比[`napi_instanceof`][napi_instanceof]因为
如果对象的原型
数据处理。类型标记与[`napi_wrap`][napi_wrap]
因为它确保从包装对象检索的指针可以是
安全地转换为与已
以前应用于 JavaScript 对象。

```c
typedef struct {
  uint64_t lower;
  uint64_t upper;
} napi_type_tag;
```

#### `napi_async_cleanup_hook_handle`

<!-- YAML
added:
  - v14.10.0
  - v12.19.0
-->

返回的不透明值[`napi_add_async_cleanup_hook`][napi_add_async_cleanup_hook].它必须通过
自[`napi_remove_async_cleanup_hook`][napi_remove_async_cleanup_hook]当异步清理链
事件完成。

### 节点 API 回调类型

#### `napi_callback_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

传递给回调函数的不透明数据类型。它可用于
获取有关回调所在的上下文的其他信息
调用。

#### `napi_callback`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

用户提供的本机函数的函数指针类型，这些函数将
通过 Node-API 公开给 JavaScript。回调函数应满足
以下签名：

```c
typedef napi_value (*napi_callback)(napi_env, napi_callback_info);
```

除非出于中讨论的原因[对象生存期管理][Object Lifetime Management]，创建一个
句柄和/或回调作用域位于`napi_callback`不是必需的。

#### `napi_finalize`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

附加组件的函数指针类型，这些函数允许用户
当外部拥有的数据已准备好进行清理时，将收到通知，因为
与之关联的对象已被垃圾回收。用户
必须提供满足以下签名的函数，该函数将获得
调用对象的集合。现在`napi_finalize`可用于
找出何时收集具有外部数据的对象。

```c
typedef void (*napi_finalize)(napi_env env,
                              void* finalize_data,
                              void* finalize_hint);
```

除非出于中讨论的原因[对象生存期管理][Object Lifetime Management]，创建一个
函数体内的句柄和/或回调作用域不是必需的。

#### `napi_async_execute_callback`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

与支持异步的函数一起使用的函数指针
操作。回调函数必须满足以下签名：

```c
typedef void (*napi_async_execute_callback)(napi_env env, void* data);
```

此函数的实现必须避免执行 Node-API 调用
JavaScript 或与 JavaScript 对象交互。Node-API 调用应该在
`napi_async_complete_callback`相反。请勿使用`napi_env`参数作为
它可能会导致JavaScript的执行。

#### `napi_async_complete_callback`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

与支持异步的函数一起使用的函数指针
操作。回调函数必须满足以下签名：

```c
typedef void (*napi_async_complete_callback)(napi_env env,
                                             napi_status status,
                                             void* data);
```

除非出于中讨论的原因[对象生存期管理][Object Lifetime Management]，创建一个
函数体内的句柄和/或回调作用域不是必需的。

#### `napi_threadsafe_function_call_js`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

用于异步线程安全函数调用的函数指针。回调
将在主线程上调用。其目的是使用到达的数据项
通过其中一个辅助线程的队列来构造参数
对于调用 JavaScript 是必需的，通常通过`napi_call_function`然后
对 JavaScript 进行调用。

通过队列从辅助线程到达的数据在`data`
参数和要调用的 JavaScript 函数在`js_callback`
参数。

Node-API 在调用此回调之前设置环境，因此它是
足以通过以下方式调用 JavaScript 函数`napi_call_function`而不是
通过`napi_make_callback`.

回调函数必须满足以下签名：

```c
typedef void (*napi_threadsafe_function_call_js)(napi_env env,
                                                 napi_value js_callback,
                                                 void* context,
                                                 void* data);
```

*   `[in] env`：用于 API 调用的环境，或`NULL`如果线程安全
    函数正在被拆除，并且`data`可能需要释放。
*   `[in] js_callback`：要调用的 JavaScript 函数，或者`NULL`如果
    线程安全函数正在被拆除，并且`data`可能需要释放。它
    也可能是`NULL`如果线程安全函数是在没有
    `js_callback`.
*   `[in] context`：线程安全函数所在的可选数据
    创建。
*   `[in] data`：由辅助线程创建的数据。这是责任
    将此本机数据转换为 JavaScript 值的回调（使用 Node-API
    函数），在以下情况下可以作为参数传递`js_callback`被调用。
    此指针完全由线程和此回调管理。因此，这个
    回调应释放数据。

除非出于中讨论的原因[对象生存期管理][Object Lifetime Management]，创建一个
函数体内的句柄和/或回调作用域不是必需的。

#### `napi_async_cleanup_hook`

<!-- YAML
added:
  - v14.10.0
  - v12.19.0
-->

函数指针与[`napi_add_async_cleanup_hook`][napi_add_async_cleanup_hook].它将被称为
当环境被破坏时。

回调函数必须满足以下签名：

```c
typedef void (*napi_async_cleanup_hook)(napi_async_cleanup_hook_handle handle,
                                        void* data);
```

*   `[in] handle`：必须传递到的句柄
    [`napi_remove_async_cleanup_hook`][napi_remove_async_cleanup_hook]异步完成后
    清理。
*   `[in] data`：传递到的数据[`napi_add_async_cleanup_hook`][napi_add_async_cleanup_hook].

函数体应在
其结尾`handle`必须在调用中传递给
[`napi_remove_async_cleanup_hook`][napi_remove_async_cleanup_hook].

## 错误处理

Node-API 同时使用返回值和 JavaScript 异常进行错误处理。
以下各节介绍了每种情况的方法。

### 返回值

所有 Node-API 函数共享相同的错误处理模式。这
所有 API 函数的返回类型为`napi_status`.

返回值为`napi_ok`如果请求成功，并且
没有未捕获的 JavaScript 异常被抛出。如果发生错误，并且
抛出异常，`napi_status`错误的值
将被退回。如果抛出异常，但未发生错误，
`napi_pending_exception`将被退回。

在返回值以外的情况下`napi_ok`或
`napi_pending_exception`返回，[`napi_is_exception_pending`][napi_is_exception_pending]
必须调用以检查异常是否挂起。
有关更多详细信息，请参阅有关例外的部分。

全套可能`napi_status`值已定义
在`napi_api_types.h`.

这`napi_status`返回值提供独立于 VM 的表示形式
发生的错误。在某些情况下，能够获得
更详细的信息，包括表示错误的字符串以及
特定于 VM（引擎）的信息。

为了检索此信息[`napi_get_last_error_info`][napi_get_last_error_info]
提供，返回`napi_extended_error_info`结构。
的格式`napi_extended_error_info`结构如下：

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
typedef struct napi_extended_error_info {
  const char* error_message;
  void* engine_reserved;
  uint32_t engine_error_code;
  napi_status error_code;
};
```

*   `error_message`：所发生错误的文本表示形式。
*   `engine_reserved`：保留给引擎使用的不透明手柄。
*   `engine_error_code`：特定于虚拟机的错误代码。
*   `error_code`：上一个错误的节点 API 状态代码。

[`napi_get_last_error_info`][napi_get_last_error_info]返回最后一个的信息
已进行的节点 API 调用。

不要依赖任何扩展信息的内容或格式
不受 SemVer 的约束，并可能随时更改。它仅用于
日志记录目的。

#### `napi_get_last_error_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status
napi_get_last_error_info(napi_env env,
                         const napi_extended_error_info** result);
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`：`napi_extended_error_info`结构与更多
    有关错误的信息。

返回`napi_ok`如果 API 成功。

此 API 检索`napi_extended_error_info`包含信息的结构
关于上次发生的错误。

内容`napi_extended_error_info`返回仅在
节点 API 函数在同一个函数上被调用`env`.这包括调用
`napi_is_exception_pending`因此，通常可能需要制作副本
的信息，以便以后可以使用。返回的指针
在`error_message`指向静态定义的字符串，因此可以安全使用
该指针，如果您已将其从`error_message`字段（将
在调用另一个 Node-API 函数之前被覆盖）。

不要依赖任何扩展信息的内容或格式
不受 SemVer 的约束，并可能随时更改。它仅用于
日志记录目的。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

### 异常

任何 Node-API 函数调用都可能导致挂起的 JavaScript 异常。这是
任何 API 函数的情况，即使是那些可能不会导致
执行 JavaScript。

如果`napi_status`由函数返回的值为`napi_ok`那么没有
异常处于挂起状态，不需要其他操作。如果
`napi_status`返回的是除以下任何内容`napi_ok`或
`napi_pending_exception`，以便尝试恢复并继续
而不是简单地立即返回，[`napi_is_exception_pending`][napi_is_exception_pending]
必须调用才能确定异常是否挂起。

在许多情况下，当调用 Node-API 函数并且异常是
已挂起，该函数将立即返回
`napi_status`之`napi_pending_exception`.但是，事实并非如此
用于所有功能。Node-API 允许函数的子集
调用以允许在返回 JavaScript 之前进行一些最小的清理。
在这种情况下，`napi_status`将反映函数的状态。它
不会反映以前的挂起异常。为避免混淆，请检查
每次函数调用后的错误状态。

当异常挂起时，可以使用以下两种方法之一。

第一种方法是进行任何适当的清理，然后返回，以便
执行将返回到 JavaScript。作为过渡的一部分，返回到
JavaScript，异常将在 JavaScript 中的某个点被抛出
调用本机方法的代码。大多数节点 API 调用的行为
在挂起异常时未指定，许多人只会返回
`napi_pending_exception`，因此请尽可能少地执行，然后返回到
可以处理异常的 JavaScript。

第二种方法是尝试处理异常。会有案例
其中本机代码可以捕获异常，采取适当的操作，
，然后继续。建议仅在特定情况下使用此方法
已知可以安全处理异常。在这些
例[`napi_get_and_clear_last_exception`][napi_get_and_clear_last_exception]可用于获取和
清除异常。成功时，结果将包含句柄
最后一个 JavaScript`Object`扔。如果确定，则在
检索异常，毕竟无法处理异常
它可以用它重新抛出[`napi_throw`][napi_throw]其中错误是
要抛出的 JavaScript 值。

以下实用程序函数在本机代码的情况下也可用
需要引发异常或确定`napi_value`是一个实例
的 JavaScript`Error`对象：[`napi_throw_error`][napi_throw_error],
[`napi_throw_type_error`][napi_throw_type_error],[`napi_throw_range_error`][napi_throw_range_error],[`node_api_throw_syntax_error`][node_api_throw_syntax_error]和[`napi_is_error`][napi_is_error].

以下实用程序函数在本机情况下也可用
代码需要创建一个`Error`对象：[`napi_create_error`][napi_create_error],
[`napi_create_type_error`][napi_create_type_error],[`napi_create_range_error`][napi_create_range_error]和[`node_api_create_syntax_error`][node_api_create_syntax_error],
其中，结果是`napi_value`指的是新创建的
JavaScript`Error`对象。

Node.js项目正在向所有错误添加错误代码
在内部生成。目标是让应用程序使用这些
所有错误检查的错误代码。关联的错误消息
将保留，但仅用于日志记录和
显示时期望消息可以更改，而无需
SemVer applying.为了通过 Node-API 支持此模型，两者兼而有之
在内部功能和模块特定功能中
（作为其良好做法），`throw_`和`create_`功能
取一个可选的代码参数，该参数是代码的字符串
以添加到错误对象。如果可选参数为`NULL`
则不会与错误关联的代码。如果提供了代码，
与错误关联的名称也会更新为：

```text
originalName [code]
```

哪里`originalName`是与错误关联的原始名称
和`code`是提供的代码。例如，如果代码
是`'ERR_ERROR_1'`和`TypeError`正在创建的名称将是：

```text
TypeError [ERR_ERROR_1]
```

#### `napi_throw`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_throw(napi_env env, napi_value error);
```

*   `[in] env`：调用 API 的环境。
*   `[in] error`：要抛出的 JavaScript 值。

返回`napi_ok`如果 API 成功。

此 API 将引发提供的 JavaScript 值。

#### `napi_throw_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_throw_error(napi_env env,
                                         const char* code,
                                         const char* msg);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`：要在错误上设置的可选错误代码。
*   `[in] msg`：表示要与错误关联的文本的 C 字符串。

返回`napi_ok`如果 API 成功。

这个 API 抛出了一个 JavaScript`Error`提供文本。

#### `napi_throw_type_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_throw_type_error(napi_env env,
                                              const char* code,
                                              const char* msg);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`：要在错误上设置的可选错误代码。
*   `[in] msg`：表示要与错误关联的文本的 C 字符串。

返回`napi_ok`如果 API 成功。

这个 API 抛出了一个 JavaScript`TypeError`提供文本。

#### `napi_throw_range_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_throw_range_error(napi_env env,
                                               const char* code,
                                               const char* msg);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`：要在错误上设置的可选错误代码。
*   `[in] msg`：表示要与错误关联的文本的 C 字符串。

返回`napi_ok`如果 API 成功。

这个 API 抛出了一个 JavaScript`RangeError`提供文本。

#### `node_api_throw_syntax_error`

<!-- YAML
added:
  - v17.2.0
  - v16.14.0
-->

> 稳定性： 1 - 实验

````c
NAPI_EXTERN napi_status node_api_throw_syntax_error(napi_env env,
                                                    const char* code,
                                                    const char* msg);
```

* `[in] env`: The environment that the API is invoked under.
* `[in] code`: Optional error code to be set on the error.
* `[in] msg`: C string representing the text to be associated with the error.

Returns `napi_ok` if the API succeeded.

This API throws a JavaScript `SyntaxError` with the text provided.

#### `napi_is_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_is_error(napi_env env,
                                      napi_value value,
                                      bool* result);
````

*   `[in] env`：调用 API 的环境。
*   `[in] value`：`napi_value`进行检查。
*   `[out] result`：如果设置为 true 的布尔值，如果`napi_value`代表
    一个错误，否则为假。

返回`napi_ok`如果 API 成功。

此 API 查询`napi_value`以检查它是否表示错误对象。

#### `napi_create_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_create_error(napi_env env,
                                          napi_value code,
                                          napi_value msg,
                                          napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`： 可选`napi_value`错误代码的字符串为
    与错误相关联。
*   `[in] msg`:`napi_value`引用 JavaScript`string`用作
    的消息`Error`.
*   `[out] result`:`napi_value`表示创建的错误。

返回`napi_ok`如果 API 成功。

此 API 返回一个 JavaScript`Error`提供文本。

#### `napi_create_type_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_create_type_error(napi_env env,
                                               napi_value code,
                                               napi_value msg,
                                               napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`： 可选`napi_value`错误代码的字符串为
    与错误相关联。
*   `[in] msg`:`napi_value`引用 JavaScript`string`用作
    的消息`Error`.
*   `[out] result`:`napi_value`表示创建的错误。

返回`napi_ok`如果 API 成功。

此 API 返回一个 JavaScript`TypeError`提供文本。

#### `napi_create_range_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_create_range_error(napi_env env,
                                                napi_value code,
                                                napi_value msg,
                                                napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`： 可选`napi_value`错误代码的字符串为
    与错误相关联。
*   `[in] msg`:`napi_value`引用 JavaScript`string`用作
    的消息`Error`.
*   `[out] result`:`napi_value`表示创建的错误。

返回`napi_ok`如果 API 成功。

此 API 返回一个 JavaScript`RangeError`提供文本。

#### `node_api_create_syntax_error`

<!-- YAML
added:
  - v17.2.0
  - v16.14.0
-->

> 稳定性： 1 - 实验

```c
NAPI_EXTERN napi_status node_api_create_syntax_error(napi_env env,
                                                     napi_value code,
                                                     napi_value msg,
                                                     napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] code`： 可选`napi_value`错误代码的字符串为
    与错误相关联。
*   `[in] msg`:`napi_value`引用 JavaScript`string`用作
    的消息`Error`.
*   `[out] result`:`napi_value`表示创建的错误。

返回`napi_ok`如果 API 成功。

此 API 返回一个 JavaScript`SyntaxError`提供文本。

#### `napi_get_and_clear_last_exception`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_and_clear_last_exception(napi_env env,
                                              napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`：如果一个是挂起的，则为异常，`NULL`否则。

返回`napi_ok`如果 API 成功。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

#### `napi_is_exception_pending`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_is_exception_pending(napi_env env, bool* result);
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`：在异常挂起时设置为 true 的布尔值。

返回`napi_ok`如果 API 成功。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

#### `napi_fatal_exception`

<!-- YAML
added: v9.10.0
napiVersion: 3
-->

```c
napi_status napi_fatal_exception(napi_env env, napi_value err);
```

*   `[in] env`：调用 API 的环境。
*   `[in] err`：传递给`'uncaughtException'`.

触发`'uncaughtException'`在 JavaScript 中。如果异步
回调引发无法恢复的异常。

### 致命错误

如果本机插件中出现不可恢复的错误，则致命错误可能是
抛出以立即终止进程。

#### `napi_fatal_error`

<!-- YAML
added: v8.2.0
napiVersion: 1
-->

```c
NAPI_NO_RETURN void napi_fatal_error(const char* location,
                                     size_t location_len,
                                     const char* message,
                                     size_t message_len);
```

*   `[in] location`：发生错误的可选位置。
*   `[in] location_len`：位置的长度（以字节为单位），或
    `NAPI_AUTO_LENGTH`如果以空值终止。
*   `[in] message`：与错误关联的消息。
*   `[in] message_len`：消息的长度（以字节为单位），或`NAPI_AUTO_LENGTH`
    如果以空值终止。

函数调用不返回，进程将被终止。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

## 对象生存期管理

当进行 Node-API 调用时，对底层的堆中的对象进行句柄
VM 可能返回为`napi_values`.这些手柄必须容纳
对象“处于活动状态”，直到本机代码不再需要它们为止，
否则，可以在本机代码之前收集对象
完成使用它们。

当返回对象句柄时，它们与
“范围”。默认作用域的生存期与生存期相关
的本机方法调用。结果是，默认情况下，句柄
保持有效，与这些句柄关联的对象将为
在本机方法调用的生存期内保持实时。

但是，在许多情况下，句柄必须保持有效
比本机方法的寿命更短或更长。
以下各节介绍可以使用的 Node-API 函数
以更改句柄寿命从默认值。

### 使句柄寿命短于本机方法的使用寿命

通常需要使手柄的使用寿命短于
本机方法的生存期。例如，考虑一个本机方法
有一个循环，循环访问大型数组中的元素：

```c
for (int i = 0; i < 1000000; i++) {
  napi_value result;
  napi_status status = napi_get_element(env, object, i, &result);
  if (status != napi_ok) {
    break;
  }
  // do something with element
}
```

这将导致创建大量句柄，从而消耗
大量资源。此外，即使本机代码也只能
使用最新的句柄，所有关联的对象也将是
保持活动状态，因为它们都共享相同的范围。

为了处理这种情况，Node-API提供了建立新“范围”的能力
将关联哪些新创建的句柄。一旦这些句柄
不再需要，范围可以“关闭”，并且任何句柄关联
范围无效。可用于打开/关闭作用域的方法包括
[`napi_open_handle_scope`][napi_open_handle_scope]和[`napi_close_handle_scope`][napi_close_handle_scope].

Node-API 仅支持作用域的单个嵌套层次结构。只有一个
活动范围随时，所有新句柄都将与该操作相关联
活动时的作用域。必须按以下相反的顺序关闭作用域：
它们被打开。此外，在本机方法中创建的所有作用域
在从该方法返回之前必须关闭。

以前面的示例为例，将调用添加到[`napi_open_handle_scope`][napi_open_handle_scope]和
[`napi_close_handle_scope`][napi_close_handle_scope]将确保最多一个句柄
在整个循环执行过程中有效：

```c
for (int i = 0; i < 1000000; i++) {
  napi_handle_scope scope;
  napi_status status = napi_open_handle_scope(env, &scope);
  if (status != napi_ok) {
    break;
  }
  napi_value result;
  status = napi_get_element(env, object, i, &result);
  if (status != napi_ok) {
    break;
  }
  // do something with element
  status = napi_close_handle_scope(env, scope);
  if (status != napi_ok) {
    break;
  }
}
```

嵌套作用域时，在某些情况下，句柄来自
内部范围需要超出该范围的寿命。节点接口支持
一个“可转义的范围”，以支持这种情况。可转义的范围
允许一个句柄被“提升”，以便它“逃避”
当前范围和句柄的使用寿命与当前
范围到外部范围的范围。

可用于打开/关闭可转义作用域的方法包括
[`napi_open_escapable_handle_scope`][napi_open_escapable_handle_scope]和
[`napi_close_escapable_handle_scope`][napi_close_escapable_handle_scope].

提升句柄的请求通过以下方式发出[`napi_escape_handle`][napi_escape_handle]哪
只能调用一次。

#### `napi_open_handle_scope`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_open_handle_scope(napi_env env,
                                               napi_handle_scope* result);
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`:`napi_value`表示新范围。

返回`napi_ok`如果 API 成功。

此 API 将打开一个新作用域。

#### `napi_close_handle_scope`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_close_handle_scope(napi_env env,
                                                napi_handle_scope scope);
```

*   `[in] env`：调用 API 的环境。
*   `[in] scope`:`napi_value`表示要关闭的范围。

返回`napi_ok`如果 API 成功。

此 API 将关闭传入的作用域。必须在
创建它们的顺序相反。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

#### `napi_open_escapable_handle_scope`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status
    napi_open_escapable_handle_scope(napi_env env,
                                     napi_handle_scope* result);
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`:`napi_value`表示新范围。

返回`napi_ok`如果 API 成功。

此 API 将打开一个新作用域，从中可以提升一个对象
到外部范围。

#### `napi_close_escapable_handle_scope`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status
    napi_close_escapable_handle_scope(napi_env env,
                                      napi_handle_scope scope);
```

*   `[in] env`：调用 API 的环境。
*   `[in] scope`:`napi_value`表示要关闭的范围。

返回`napi_ok`如果 API 成功。

此 API 将关闭传入的作用域。必须在
创建它们的顺序相反。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

#### `napi_escape_handle`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_escape_handle(napi_env env,
                               napi_escapable_handle_scope scope,
                               napi_value escapee,
                               napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] scope`:`napi_value`表示当前范围。
*   `[in] escapee`:`napi_value`代表 JavaScript`Object`成为
    逃脱。
*   `[out] result`:`napi_value`表示转义的句柄`Object`
    在外部范围中。

返回`napi_ok`如果 API 成功。

此 API 将句柄提升到 JavaScript 对象，使其有效
对于外部作用域的生存期。每个作用域只能调用一次。
如果多次调用它，将返回错误。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

### 对生存期长于本机方法的对象的引用

在某些情况下，插件需要能够创建和引用对象
其生存期比单个本机方法调用的生存期更长。为
示例，创建构造函数并在以后使用该构造函数
在创建实例的请求中，必须能够引用
跨多个不同实例创建请求的构造函数对象。这
无法将正常句柄作为返回`napi_value`如
在前面的部分中进行了介绍。普通手柄的使用寿命为
由作用域管理，并且必须在本机结束之前关闭所有作用域
方法。

Node-API 提供了创建对对象的持久引用的方法。
每个持久引用都有一个关联的计数，值为 0
或更高。计数确定引用是否保留
相应的对象处于活动状态。计数为 0 的引用不
防止对象被收集，通常被称为“弱”
引用。任何大于 0 的计数都将阻止该对象
从被收集。

可以使用初始引用计数创建引用。计数可以
然后通过以下方式进行修改[`napi_reference_ref`][napi_reference_ref]和
[`napi_reference_unref`][napi_reference_unref].如果在计数时收集对象
对于一个引用是 0，所有后续调用
获取与引用关联的对象[`napi_get_reference_value`][napi_get_reference_value]
会再来`NULL`对于返回的`napi_value`.尝试调用
[`napi_reference_ref`][napi_reference_ref]用于已收集其对象的引用
导致错误。

一旦插件不再需要引用，就必须删除这些引用。什么时候
一个引用被删除，它将不再阻止相应的对象
正在收集。未能删除持久性引用会导致
“内存泄漏”，其中包含持久性引用的本机内存和
堆上的相应对象将永久保留。

可以创建多个持久引用，这些引用引用
对象，其中每个对象都将保持活动状态或不基于其
个人计数。对同一对象的多个持久引用
可能导致意外地保持活动本机内存。本机结构
对于持久性引用，必须保持活动状态，直到
执行引用的对象。如果创建新的持久性引用
对于同一对象，该对象的终结器将不会是
run 和早期持久性引用所指向的本机内存
不会被释放。这可以通过调用
`napi_delete_reference`除了`napi_reference_unref`如果可能。

#### `napi_create_reference`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_create_reference(napi_env env,
                                              napi_value value,
                                              uint32_t initial_refcount,
                                              napi_ref* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`代表`Object`我们想要一个
    参考。
*   `[in] initial_refcount`：新引用的初始引用计数。
*   `[out] result`:`napi_ref`指向新引用。

返回`napi_ok`如果 API 成功。

此 API 创建具有指定引用计数的新引用
到`Object`转会了。

#### `napi_delete_reference`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_delete_reference(napi_env env, napi_ref ref);
```

*   `[in] env`：调用 API 的环境。
*   `[in] ref`:`napi_ref`要删除。

返回`napi_ok`如果 API 成功。

此 API 将删除传入的引用。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

#### `napi_reference_ref`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_reference_ref(napi_env env,
                                           napi_ref ref,
                                           uint32_t* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] ref`:`napi_ref`其引用计数将递增。
*   `[out] result`：新的引用计数。

返回`napi_ok`如果 API 成功。

此 API 递增引用的引用计数
传入并返回生成的引用计数。

#### `napi_reference_unref`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_reference_unref(napi_env env,
                                             napi_ref ref,
                                             uint32_t* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] ref`:`napi_ref`其引用计数将递减。
*   `[out] result`：新的引用计数。

返回`napi_ok`如果 API 成功。

此 API 递减引用的引用计数
传入并返回生成的引用计数。

#### `napi_get_reference_value`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_get_reference_value(napi_env env,
                                                 napi_ref ref,
                                                 napi_value* result);
```

这`napi_value passed`这些方法的 in 或 out 是
与引用相关的对象。

*   `[in] env`：调用 API 的环境。
*   `[in] ref`:`napi_ref`为此，我们要求相应的`Object`.
*   `[out] result`：`napi_value`对于`Object`引用者
    `napi_ref`.

返回`napi_ok`如果 API 成功。

如果仍然有效，此 API 将返回`napi_value`代表
JavaScript`Object`与`napi_ref`.否则，结果
将是`NULL`.

### 当前节点退出时进行清理.js实例

虽然节点.js进程通常在退出时释放其所有资源，
Node.js或未来 Worker 支持的嵌入者可能需要插件来注册
清理将在当前 Node.js 实例退出后运行的挂接。

Node-API 提供了用于注册和取消注册此类回调的函数。
运行这些回调时，插件持有的所有资源
应该被释放。

#### `napi_add_env_cleanup_hook`

<!-- YAML
added: v10.2.0
napiVersion: 3
-->

```c
NODE_EXTERN napi_status napi_add_env_cleanup_hook(napi_env env,
                                                  void (*fun)(void* arg),
                                                  void* arg);
```

寄存 器`fun`作为要与`arg`参数一旦
当前节点.js环境退出。

一个功能可以安全地多次指定，具有不同
`arg`值。在这种情况下，它也将被多次调用。
提供相同的`fun`和`arg`不允许多次值
并将导致该过程中止。

钩子将以相反的顺序调用，即最近添加的钩子
将首先调用。

删除此钩子可以通过使用[`napi_remove_env_cleanup_hook`][napi_remove_env_cleanup_hook].
通常，当为其添加此挂接的资源时，会发生这种情况
无论如何都被拆除了。

对于异步清理，[`napi_add_async_cleanup_hook`][napi_add_async_cleanup_hook]可用。

#### `napi_remove_env_cleanup_hook`

<!-- YAML
added: v10.2.0
napiVersion: 3
-->

```c
NAPI_EXTERN napi_status napi_remove_env_cleanup_hook(napi_env env,
                                                     void (*fun)(void* arg),
                                                     void* arg);
```

取消注册`fun`作为要与`arg`参数一旦
当前节点.js环境退出。参数和函数值
需要完全匹配。

该函数必须最初已注册
跟`napi_add_env_cleanup_hook`，否则进程将中止。

#### `napi_add_async_cleanup_hook`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
napiVersion: 8
changes:
  - version:
    - v14.10.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34819
    description: Changed signature of the `hook` callback.
-->

```c
NAPI_EXTERN napi_status napi_add_async_cleanup_hook(
    napi_env env,
    napi_async_cleanup_hook hook,
    void* arg,
    napi_async_cleanup_hook_handle* remove_handle);
```

*   `[in] env`：调用 API 的环境。
*   `[in] hook`：在环境拆除时调用的函数指针。
*   `[in] arg`：要传递到的指针`hook`当它被调用时。
*   `[out] remove_handle`：引用异步清理的可选句柄
    钩。

寄存 器`hook`，这是类型的函数[`napi_async_cleanup_hook`][napi_async_cleanup_hook]如
要与`remove_handle`和`arg`参数一旦
当前节点.js环境退出。

与[`napi_add_env_cleanup_hook`][napi_add_env_cleanup_hook]，则允许钩子是异步的。

否则，行为通常与[`napi_add_env_cleanup_hook`][napi_add_env_cleanup_hook].

如果`remove_handle`莫`NULL`，不透明的值将存储在其中
以后必须传递给[`napi_remove_async_cleanup_hook`][napi_remove_async_cleanup_hook],
无论钩子是否已被调用。
通常，当为其添加此挂接的资源时，会发生这种情况
无论如何都被拆除了。

#### `napi_remove_async_cleanup_hook`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
changes:
  - version:
    - v14.10.0
    - v12.19.0
    pr-url: https://github.com/nodejs/node/pull/34819
    description: Removed `env` parameter.
-->

```c
NAPI_EXTERN napi_status napi_remove_async_cleanup_hook(
    napi_async_cleanup_hook_handle remove_handle);
```

*   `[in] remove_handle`：异步清理挂钩的句柄，该钩子是
    创建于[`napi_add_async_cleanup_hook`][napi_add_async_cleanup_hook].

取消注册对应于`remove_handle`.这将防止
钩子被执行，除非它已经开始执行。
这必须调用任何`napi_async_cleanup_hook_handle`获得的值
从[`napi_add_async_cleanup_hook`][napi_add_async_cleanup_hook].

## 模块注册

Node-API 模块的注册方式与其他模块类似
除了而不是使用`NODE_MODULE`宏以下
用于：

```c
NAPI_MODULE(NODE_GYP_MODULE_NAME, Init)
```

下一个区别是`Init`方法。对于 Node-API
模块如下：

```c
napi_value Init(napi_env env, napi_value exports);
```

返回值`Init`被视为`exports`对象。
这`Init`方法通过`exports`参数作为
方便。如果`Init`返回`NULL`，则参数传递为`exports`是
由模块导出。节点 API 模块无法修改`module`对象，但
可以将任何内容指定为`exports`模块的属性。

添加方法`hello`作为一个函数，以便它可以作为一个方法调用
由插件提供：

```c
napi_value Init(napi_env env, napi_value exports) {
  napi_status status;
  napi_property_descriptor desc = {
    "hello",
    NULL,
    Method,
    NULL,
    NULL,
    NULL,
    napi_writable | napi_enumerable | napi_configurable,
    NULL
  };
  status = napi_define_properties(env, exports, 1, &desc);
  if (status != napi_ok) return NULL;
  return exports;
}
```

要设置要由`require()`对于插件：

```c
napi_value Init(napi_env env, napi_value exports) {
  napi_value method;
  napi_status status;
  status = napi_create_function(env, "exports", NAPI_AUTO_LENGTH, Method, NULL, &method);
  if (status != napi_ok) return NULL;
  return method;
}
```

定义一个类，以便可以创建新实例（通常与
[对象环绕][Object wrap]):

```c
// NOTE: partial example, not all referenced code is included
napi_value Init(napi_env env, napi_value exports) {
  napi_status status;
  napi_property_descriptor properties[] = {
    { "value", NULL, NULL, GetValue, SetValue, NULL, napi_writable | napi_configurable, NULL },
    DECLARE_NAPI_METHOD("plusOne", PlusOne),
    DECLARE_NAPI_METHOD("multiply", Multiply),
  };

  napi_value cons;
  status =
      napi_define_class(env, "MyObject", New, NULL, 3, properties, &cons);
  if (status != napi_ok) return NULL;

  status = napi_create_reference(env, cons, 1, &constructor);
  if (status != napi_ok) return NULL;

  status = napi_set_named_property(env, exports, "MyObject", cons);
  if (status != napi_ok) return NULL;

  return exports;
}
```

您还可以使用`NAPI_MODULE_INIT`宏，用作速记
为`NAPI_MODULE`并定义`Init`功能：

```c
NAPI_MODULE_INIT() {
  napi_value answer;
  napi_status result;

  status = napi_create_int64(env, 42, &answer);
  if (status != napi_ok) return NULL;

  status = napi_set_named_property(env, exports, "answer", answer);
  if (status != napi_ok) return NULL;

  return exports;
}
```

所有 Node-API 插件都是上下文感知的，这意味着它们可以加载多个
次。声明此类模块时，有一些设计注意事项。
文档[上下文感知插件][context-aware addons]提供了更多详细信息。

变量`env`和`exports`将在函数体内部提供
在宏调用之后。

有关设置对象属性的更多详细信息，请参阅
[使用 JavaScript 属性][Working with JavaScript properties].

有关构建一般插件模块的更多详细信息，请参阅现有的
应用程序接口。

## 使用 JavaScript 值

Node-API 公开了一组 API 来创建所有类型的 JavaScript 值。
其中一些类型记录在[第6款][Section 6]
的[ECMAScript 语言规范][ECMAScript Language Specification].

从根本上说，这些 API 用于执行下列操作之一：

1.  创建新的 JavaScript 对象
2.  从基元 C 类型转换为节点 API 值
3.  从 Node-API 值转换为基元 C 类型
4.  获取全局实例，包括`undefined`和`null`

节点 API 值由类型表示`napi_value`.
任何需要 JavaScript 值的 Node-API 调用都会采用`napi_value`.
在某些情况下，API 会检查`napi_value`前期。
但是，为了获得更好的性能，调用方最好确保
这`napi_value`有问题的是API所期望的JavaScript类型。

### 枚举类型

#### `napi_key_collection_mode`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
 - v10.20.0
napiVersion: 6
-->

```c
typedef enum {
  napi_key_include_prototypes,
  napi_key_own_only
} napi_key_collection_mode;
```

描述`Keys/Properties`过滤器枚举：

`napi_key_collection_mode`限制收集的属性的范围。

`napi_key_own_only`将收集的属性限制为给定的属性
仅对象。`napi_key_include_prototypes`将包括所有键
对象的原型链也是如此。

#### `napi_key_filter`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
 - v10.20.0
napiVersion: 6
-->

```c
typedef enum {
  napi_key_all_properties = 0,
  napi_key_writable = 1,
  napi_key_enumerable = 1 << 1,
  napi_key_configurable = 1 << 2,
  napi_key_skip_strings = 1 << 3,
  napi_key_skip_symbols = 1 << 4
} napi_key_filter;
```

属性筛选器位。它们可以用于构建复合过滤器。

#### `napi_key_conversion`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
 - v10.20.0
napiVersion: 6
-->

```c
typedef enum {
  napi_key_keep_numbers,
  napi_key_numbers_to_strings
} napi_key_conversion;
```

`napi_key_numbers_to_strings`将整数索引转换为
字符串。`napi_key_keep_numbers`将返回整数的数字
指标。

#### `napi_valuetype`

```c
typedef enum {
  // ES6 types (corresponds to typeof)
  napi_undefined,
  napi_null,
  napi_boolean,
  napi_number,
  napi_string,
  napi_symbol,
  napi_object,
  napi_function,
  napi_external,
  napi_bigint,
} napi_valuetype;
```

描述 的类型`napi_value`.这通常对应于类型
描述于[第6.1节][Section 6.1]的 ECMAScript 语言规范。
除了该部分中的类型外，`napi_valuetype`也可以表示
`Function`s 和`Object`s 与外部数据。

类型的 JavaScript 值`napi_external`在 JavaScript 中显示为普通
对象，使得不能在其上设置任何属性，并且不能设置任何原型。

#### `napi_typedarray_type`

```c
typedef enum {
  napi_int8_array,
  napi_uint8_array,
  napi_uint8_clamped_array,
  napi_int16_array,
  napi_uint16_array,
  napi_int32_array,
  napi_uint32_array,
  napi_float32_array,
  napi_float64_array,
  napi_bigint64_array,
  napi_biguint64_array,
} napi_typedarray_type;
```

这表示`TypedArray`.
此枚举的元素对应于
[第22.2节][Section 22.2]的[ECMAScript 语言规范][ECMAScript Language Specification].

### 对象创建函数

#### `napi_create_array`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_array(napi_env env, napi_value* result)
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[out] result`： A`napi_value`表示 JavaScript`Array`.

返回`napi_ok`如果 API 成功。

此 API 返回与 JavaScript 对应的 Node-API 值`Array`类型。
JavaScript 数组在
[第22.1节][Section 22.1]的 ECMAScript 语言规范。

#### `napi_create_array_with_length`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_array_with_length(napi_env env,
                                          size_t length,
                                          napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] length`：初始长度`Array`.
*   `[out] result`： A`napi_value`表示 JavaScript`Array`.

返回`napi_ok`如果 API 成功。

此 API 返回与 JavaScript 对应的 Node-API 值`Array`类型。
这`Array`的长度属性设置为传入的长度参数。
但是，不保证 VM 预先分配基础缓冲区
创建数组时。该行为留给基础 VM 处理
实现。如果缓冲区必须是连续的内存块，则可以
通过C直接读取和/或写入，考虑使用
[`napi_create_external_arraybuffer`][napi_create_external_arraybuffer].

JavaScript 数组在
[第22.1节][Section 22.1]的 ECMAScript 语言规范。

#### `napi_create_arraybuffer`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_arraybuffer(napi_env env,
                                    size_t byte_length,
                                    void** data,
                                    napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] length`：要创建的数组缓冲区的长度（以字节为单位）。
*   `[out] data`：指向`ArrayBuffer`.
    `data`可以选择通过传递来忽略`NULL`.
*   `[out] result`： A`napi_value`表示 JavaScript`ArrayBuffer`.

返回`napi_ok`如果 API 成功。

此 API 返回与 JavaScript 对应的 Node-API 值`ArrayBuffer`.
`ArrayBuffer`s 用于表示固定长度的二进制数据缓冲区。他们是
通常用作`TypedArray`对象。
这`ArrayBuffer`分配将具有一个基础字节缓冲区，其大小为
由`length`传入的参数。
可以选择将基础缓冲区返回给调用方，以防
调用方想要直接操作缓冲区。此缓冲区只能是
直接从本机代码编写到。要从 JavaScript 写入此缓冲区，
类型化数组或`DataView`需要创建对象。

JavaScript`ArrayBuffer`对象描述于
[第24.1节][Section 24.1]的 ECMAScript 语言规范。

#### `napi_create_buffer`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_buffer(napi_env env,
                               size_t size,
                               void** data,
                               napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] size`：基础缓冲区的大小（以字节为单位）。
*   `[out] data`：指向基础缓冲区的原始指针。
    `data`可以选择通过传递来忽略`NULL`.
*   `[out] result`： A`napi_value`表示`node::Buffer`.

返回`napi_ok`如果 API 成功。

此 API 分配一个`node::Buffer`对象。虽然这仍然是一个
完全支持的数据结构，在大多数情况下使用`TypedArray`就足够了。

#### `napi_create_buffer_copy`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_buffer_copy(napi_env env,
                                    size_t length,
                                    const void* data,
                                    void** result_data,
                                    napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] size`：输入缓冲区的大小（以字节为单位）（应与大小相同）
    的新缓冲区）。
*   `[in] data`：指向要从中复制的基础缓冲区的原始指针。
*   `[out] result_data`：指向新`Buffer`的基础数据缓冲区。
    `result_data`可以选择通过传递来忽略`NULL`.
*   `[out] result`： A`napi_value`表示`node::Buffer`.

返回`napi_ok`如果 API 成功。

此 API 分配一个`node::Buffer`对象并使用复制的数据对其进行初始化
从传入的缓冲区。虽然这仍然是一个完全支持的数据
结构，在大多数情况下使用`TypedArray`就足够了。

#### `napi_create_date`

<!-- YAML
added:
 - v11.11.0
 - v10.17.0
napiVersion: 5
-->

```c
napi_status napi_create_date(napi_env env,
                             double time,
                             napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] time`：ECMAScript 自 1970 年 1 月 1 日 UTC 以来的时间值（以毫秒为单位）。
*   `[out] result`： A`napi_value`表示 JavaScript`Date`.

返回`napi_ok`如果 API 成功。

此 API 不观察闰秒;它们被忽略，因为
ECMAScript 与 POSIX 时间规范保持一致。

此 API 分配一个 JavaScript`Date`对象。

JavaScript`Date`对象描述于
[第20.3节][Section 20.3]的 ECMAScript 语言规范。

#### `napi_create_external`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_external(napi_env env,
                                 void* data,
                                 napi_finalize finalize_cb,
                                 void* finalize_hint,
                                 napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] data`：指向外部数据的原始指针。
*   `[in] finalize_cb`：可选回调，以便在外部值处于
    收集。[`napi_finalize`][napi_finalize]提供了更多详细信息。
*   `[in] finalize_hint`：在
    收集。
*   `[out] result`： A`napi_value`表示外部值。

返回`napi_ok`如果 API 成功。

此 API 分配一个 JavaScript 值，并附加外部数据。这
用于通过 JavaScript 代码传递外部数据，因此可以检索
以后通过本机代码使用[`napi_get_value_external`][napi_get_value_external].

该接口添加了一个`napi_finalize`在 JavaScript 时调用的回调
刚刚创建的对象已准备好进行垃圾回收。它类似于
`napi_wrap()`除了：

*   以后无法检索本机数据`napi_unwrap()`,
*   以后也不能使用`napi_remove_wrap()`和
*   由 API 创建的对象可用于`napi_wrap()`.

创建的值不是对象，因此不支持附加
性能。它被认为是一种不同的值类型：调用`napi_typeof()`跟
外部价值产生`napi_external`.

#### `napi_create_external_arraybuffer`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status
napi_create_external_arraybuffer(napi_env env,
                                 void* external_data,
                                 size_t byte_length,
                                 napi_finalize finalize_cb,
                                 void* finalize_hint,
                                 napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] external_data`：指向
    `ArrayBuffer`.
*   `[in] byte_length`：基础缓冲区的长度（以字节为单位）。
*   `[in] finalize_cb`：可选回调时调用`ArrayBuffer`正在
    收集。[`napi_finalize`][napi_finalize]提供了更多详细信息。
*   `[in] finalize_hint`：在
    收集。
*   `[out] result`： A`napi_value`表示 JavaScript`ArrayBuffer`.

返回`napi_ok`如果 API 成功。

此 API 返回与 JavaScript 对应的 Node-API 值`ArrayBuffer`.
的基础字节缓冲区`ArrayBuffer`是外部分配的，并且
管理。调用方必须确保字节缓冲区保持有效，直到
调用 finalize 回调。

该接口添加了一个`napi_finalize`在 JavaScript 时调用的回调
刚刚创建的对象已准备好进行垃圾回收。它类似于
`napi_wrap()`除了：

*   以后无法检索本机数据`napi_unwrap()`,
*   以后也不能使用`napi_remove_wrap()`和
*   由 API 创建的对象可用于`napi_wrap()`.

JavaScript`ArrayBuffer`s 描述于
[第24.1节][Section 24.1]的 ECMAScript 语言规范。

#### `napi_create_external_buffer`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_external_buffer(napi_env env,
                                        size_t length,
                                        void* data,
                                        napi_finalize finalize_cb,
                                        void* finalize_hint,
                                        napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] length`：输入缓冲区的大小（以字节为单位）应与
    新缓冲区的大小）。
*   `[in] data`：指向要向 JavaScript 公开的基础缓冲区的原始指针。
*   `[in] finalize_cb`：可选回调时调用`ArrayBuffer`正在
    收集。[`napi_finalize`][napi_finalize]提供了更多详细信息。
*   `[in] finalize_hint`：在
    收集。
*   `[out] result`： A`napi_value`表示`node::Buffer`.

返回`napi_ok`如果 API 成功。

此 API 分配一个`node::Buffer`对象并用数据初始化它
由传入的缓冲区支持。虽然这仍然是一个完全支持的数据
结构，在大多数情况下使用`TypedArray`就足够了。

该接口添加了一个`napi_finalize`在 JavaScript 时调用的回调
刚刚创建的对象已准备好进行垃圾回收。它类似于
`napi_wrap()`除了：

*   以后无法检索本机数据`napi_unwrap()`,
*   以后也不能使用`napi_remove_wrap()`和
*   由 API 创建的对象可用于`napi_wrap()`.

对于节点.js >=4`Buffers`是`Uint8Array`s.

#### `napi_create_object`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_object(napi_env env, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`： A`napi_value`表示 JavaScript`Object`.

返回`napi_ok`如果 API 成功。

此 API 分配一个默认的 JavaScript`Object`.
它相当于做`new Object()`在 JavaScript 中。

The JavaScript`Object`类型描述于[第6.1.7节][Section 6.1.7]的
ECMAScript 语言规范。

#### `napi_create_symbol`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_symbol(napi_env env,
                               napi_value description,
                               napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] description`： 可选`napi_value`它指的是JavaScript
    `string`设置为符号的描述。
*   `[out] result`： A`napi_value`表示 JavaScript`symbol`.

返回`napi_ok`如果 API 成功。

此 API 创建一个 JavaScript`symbol`UTF8 编码的 C 字符串中的值。

The JavaScript`symbol`类型描述于[第19.4节][Section 19.4]
的 ECMAScript 语言规范。

#### `node_api_symbol_for`

<!-- YAML
added:
  - v17.5.0
  - v16.15.0
-->

> 稳定性： 1 - 实验

```c
napi_status node_api_symbol_for(napi_env env,
                                const char* utf8description,
                                size_t length,
                                napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] utf8description`：UTF-8 C 字符串，表示要用作的文本
    符号的说明。
*   `[in] length`：描述字符串的长度（以字节为单位），或
    `NAPI_AUTO_LENGTH`如果以空值终止。
*   `[out] result`： A`napi_value`表示 JavaScript`symbol`.

返回`napi_ok`如果 API 成功。

此 API 在全局注册表中搜索具有给定符号的现有符号
描述。如果符号已经存在，它将被返回，否则将返回一个新的
符号将在注册表中创建。

The JavaScript`symbol`类型描述于[第19.4节][Section 19.4]的 ECMAScript
语言规范。

#### `napi_create_typedarray`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_typedarray(napi_env env,
                                   napi_typedarray_type type,
                                   size_t length,
                                   napi_value arraybuffer,
                                   size_t byte_offset,
                                   napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] type`：元素中的标量数据类型`TypedArray`.
*   `[in] length`：元素数`TypedArray`.
*   `[in] arraybuffer`:`ArrayBuffer`作为类型化数组的基础。
*   `[in] byte_offset`：字节偏移量`ArrayBuffer`从哪个到
    开始投影`TypedArray`.
*   `[out] result`： A`napi_value`表示 JavaScript`TypedArray`.

返回`napi_ok`如果 API 成功。

此 API 创建一个 JavaScript`TypedArray`对象覆盖现有对象
`ArrayBuffer`.`TypedArray`对象在
基础数据缓冲区，其中每个元素具有相同的基础二进制标量
数据类型。

需要`(length * size_of_element) + byte_offset`应该
< = 传入数组的大小（以字节为单位）。如果没有，则`RangeError`例外
被引发。

JavaScript`TypedArray`对象描述于
[第22.2节][Section 22.2]的 ECMAScript 语言规范。

#### `napi_create_dataview`

<!-- YAML
added: v8.3.0
napiVersion: 1
-->

```c
napi_status napi_create_dataview(napi_env env,
                                 size_t byte_length,
                                 napi_value arraybuffer,
                                 size_t byte_offset,
                                 napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] length`：元素数`DataView`.
*   `[in] arraybuffer`:`ArrayBuffer`基础`DataView`.
*   `[in] byte_offset`：字节偏移量`ArrayBuffer`从哪个到
    开始投影`DataView`.
*   `[out] result`： A`napi_value`表示 JavaScript`DataView`.

返回`napi_ok`如果 API 成功。

此 API 创建一个 JavaScript`DataView`对象覆盖现有对象`ArrayBuffer`.
`DataView`对象在底层数据缓冲区上提供类似数组的视图，
但是一个允许不同大小和类型的项目在`ArrayBuffer`.

要求`byte_length + byte_offset`小于或等于
大小（以传入的数组的字节数为单位）。如果没有，则`RangeError`异常是
提出。

JavaScript`DataView`对象描述于
[第24.3节][Section 24.3]的 ECMAScript 语言规范。

### 从 C 类型转换为 Node-API 的函数

#### `napi_create_int32`

<!-- YAML
added: v8.4.0
napiVersion: 1
-->

```c
napi_status napi_create_int32(napi_env env, int32_t value, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要用 JavaScript 表示的整数值。
*   `[out] result`： A`napi_value`表示 JavaScript`number`.

返回`napi_ok`如果 API 成功。

此 API 用于从 C 转换`int32_t`键入到 JavaScript
`number`类型。

The JavaScript`number`类型描述于
[第6.1.6节][Section 6.1.6]的 ECMAScript 语言规范。

#### `napi_create_uint32`

<!-- YAML
added: v8.4.0
napiVersion: 1
-->

```c
napi_status napi_create_uint32(napi_env env, uint32_t value, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：用 JavaScript 表示的无符号整数值。
*   `[out] result`： A`napi_value`表示 JavaScript`number`.

返回`napi_ok`如果 API 成功。

此 API 用于从 C 转换`uint32_t`键入到 JavaScript
`number`类型。

The JavaScript`number`类型描述于
[第6.1.6节][Section 6.1.6]的 ECMAScript 语言规范。

#### `napi_create_int64`

<!-- YAML
added: v8.4.0
napiVersion: 1
-->

```c
napi_status napi_create_int64(napi_env env, int64_t value, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要用 JavaScript 表示的整数值。
*   `[out] result`： A`napi_value`表示 JavaScript`number`.

返回`napi_ok`如果 API 成功。

此 API 用于从 C 转换`int64_t`键入到 JavaScript
`number`类型。

The JavaScript`number`类型描述于[第6.1.6节][Section 6.1.6]
的 ECMAScript 语言规范。注意完整的系列`int64_t`
不能在 JavaScript 中以完全精确的方式表示。整数值
超出范围[`Number.MIN_SAFE_INTEGER`][Number.MIN_SAFE_INTEGER] `-(2**53 - 1)`-
[`Number.MAX_SAFE_INTEGER`][Number.MAX_SAFE_INTEGER] `(2**53 - 1)`将失去精度。

#### `napi_create_double`

<!-- YAML
added: v8.4.0
napiVersion: 1
-->

```c
napi_status napi_create_double(napi_env env, double value, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：用 JavaScript 表示的双精度值。
*   `[out] result`： A`napi_value`表示 JavaScript`number`.

返回`napi_ok`如果 API 成功。

此 API 用于从 C 转换`double`键入到 JavaScript
`number`类型。

The JavaScript`number`类型描述于
[第6.1.6节][Section 6.1.6]的 ECMAScript 语言规范。

#### `napi_create_bigint_int64`

<!-- YAML
added: v10.7.0
napiVersion: 6
-->

```c
napi_status napi_create_bigint_int64(napi_env env,
                                     int64_t value,
                                     napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要用 JavaScript 表示的整数值。
*   `[out] result`： A`napi_value`表示 JavaScript`BigInt`.

返回`napi_ok`如果 API 成功。

此 API 转换 C`int64_t`键入到 JavaScript`BigInt`类型。

#### `napi_create_bigint_uint64`

<!-- YAML
added: v10.7.0
napiVersion: 6
-->

```c
napi_status napi_create_bigint_uint64(napi_env env,
                                      uint64_t value,
                                      napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：用 JavaScript 表示的无符号整数值。
*   `[out] result`： A`napi_value`表示 JavaScript`BigInt`.

返回`napi_ok`如果 API 成功。

此 API 转换 C`uint64_t`键入到 JavaScript`BigInt`类型。

#### `napi_create_bigint_words`

<!-- YAML
added: v10.7.0
napiVersion: 6
-->

```c
napi_status napi_create_bigint_words(napi_env env,
                                     int sign_bit,
                                     size_t word_count,
                                     const uint64_t* words,
                                     napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] sign_bit`：确定结果是否`BigInt`将为阳性或
    阴性。
*   `[in] word_count`：长度`words`数组。
*   `[in] words`：数组`uint64_t`小端 64 位字。
*   `[out] result`： A`napi_value`表示 JavaScript`BigInt`.

返回`napi_ok`如果 API 成功。

此 API 将无符号 64 位字数组转换为单个`BigInt`
价值。

结果`BigInt`计算公式为：（–1）<sup>`sign_bit`</sup>(`words[0]`
× （2）<sup>64</sup>)<sup>0</sup>+`words[1]`× （2）<sup>64</sup>)<sup>1</sup>+ ...)

#### `napi_create_string_latin1`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_string_latin1(napi_env env,
                                      const char* str,
                                      size_t length,
                                      napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] str`：表示 ISO-8859-1 编码字符串的字符缓冲区。
*   `[in] length`：字符串的长度（以字节为单位），或`NAPI_AUTO_LENGTH`如果它
    以空值终止。
*   `[out] result`： A`napi_value`表示 JavaScript`string`.

返回`napi_ok`如果 API 成功。

此 API 创建一个 JavaScript`string`来自 ISO-8859-1 编码 C 的值
字符串。复制本机字符串。

The JavaScript`string`类型描述于
[第6.1.4节][Section 6.1.4]的 ECMAScript 语言规范。

#### `napi_create_string_utf16`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_string_utf16(napi_env env,
                                     const char16_t* str,
                                     size_t length,
                                     napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] str`：表示 UTF16-LE 编码字符串的字符缓冲区。
*   `[in] length`：以双字节代码单位表示的字符串长度，或
    `NAPI_AUTO_LENGTH`如果以空值终止。
*   `[out] result`： A`napi_value`表示 JavaScript`string`.

返回`napi_ok`如果 API 成功。

此 API 创建一个 JavaScript`string`UTF16-LE 编码的 C 字符串中的值。
复制本机字符串。

The JavaScript`string`类型描述于
[第6.1.4节][Section 6.1.4]的 ECMAScript 语言规范。

#### `napi_create_string_utf8`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_string_utf8(napi_env env,
                                    const char* str,
                                    size_t length,
                                    napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] str`：表示 UTF8 编码字符串的字符缓冲区。
*   `[in] length`：字符串的长度（以字节为单位），或`NAPI_AUTO_LENGTH`如果它
    以空值终止。
*   `[out] result`： A`napi_value`表示 JavaScript`string`.

返回`napi_ok`如果 API 成功。

此 API 创建一个 JavaScript`string`UTF8 编码的 C 字符串中的值。
复制本机字符串。

The JavaScript`string`类型描述于
[第6.1.4节][Section 6.1.4]的 ECMAScript 语言规范。

### 从 Node-API 转换为 C 类型的函数

#### `napi_get_array_length`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_array_length(napi_env env,
                                  napi_value value,
                                  uint32_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`代表 JavaScript`Array`其长度为
    正在查询。
*   `[out] result`:`uint32`表示数组的长度。

返回`napi_ok`如果 API 成功。

此 API 返回数组的长度。

`Array`长度描述于[部分 22.1.4.1][Section 22.1.4.1]的 ECMAScript 语言
规范。

#### `napi_get_arraybuffer_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_arraybuffer_info(napi_env env,
                                      napi_value arraybuffer,
                                      void** data,
                                      size_t* byte_length)
```

*   `[in] env`：调用 API 的环境。
*   `[in] arraybuffer`:`napi_value`代表`ArrayBuffer`正在查询。
*   `[out] data`：的基础数据缓冲区`ArrayBuffer`.如果byte_length
    是`0`，这可能是`NULL`或任何其他指针值。
*   `[out] byte_length`：基础数据缓冲区的长度（以字节为单位）。

返回`napi_ok`如果 API 成功。

此 API 用于检索`ArrayBuffer`和
它的长度。

*警告*：使用此 API 时要小心。基础数据的生存期
缓冲区由`ArrayBuffer`即使它被退回。一个
使用此 API 的可能安全方法是与
[`napi_create_reference`][napi_create_reference]，可用于保证对
的生命周期`ArrayBuffer`.使用返回的数据缓冲区也是安全的
在同一回调中，只要没有对可能
触发 GC。

#### `napi_get_buffer_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_buffer_info(napi_env env,
                                 napi_value value,
                                 void** data,
                                 size_t* length)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`代表`node::Buffer`正在查询。
*   `[out] data`：的基础数据缓冲区`node::Buffer`.
    如果长度为`0`，这可能是`NULL`或任何其他指针值。
*   `[out] length`：基础数据缓冲区的长度（以字节为单位）。

返回`napi_ok`如果 API 成功。

此 API 用于检索`node::Buffer`
及其长度。

*警告*：使用此 API 时要小心，因为基础数据缓冲区的
如果生存期由 VM 管理，则无法保证生存期。

#### `napi_get_prototype`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_prototype(napi_env env,
                               napi_value object,
                               napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] object`:`napi_value`表示 JavaScript`Object`谁的原型
    返回。这返回的等效值`Object.getPrototypeOf`（这是
    与函数的不相同`prototype`属性）。
*   `[out] result`:`napi_value`表示给定对象的原型。

返回`napi_ok`如果 API 成功。

#### `napi_get_typedarray_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_typedarray_info(napi_env env,
                                     napi_value typedarray,
                                     napi_typedarray_type* type,
                                     size_t* length,
                                     void** data,
                                     napi_value* arraybuffer,
                                     size_t* byte_offset)
```

*   `[in] env`：调用 API 的环境。
*   `[in] typedarray`:`napi_value`代表`TypedArray`谁的
    要查询的属性。
*   `[out] type`：元素中的标量数据类型`TypedArray`.
*   `[out] length`：元素的数量`TypedArray`.
*   `[out] data`：底层的数据缓冲区`TypedArray`调整者
    这`byte_offset`值，以便它指向
    `TypedArray`.如果数组的长度为`0`，这可能是`NULL`或
    任何其他指针值。
*   `[out] arraybuffer`：`ArrayBuffer`基础`TypedArray`.
*   `[out] byte_offset`：底层本机数组中的字节偏移量
    数组的第一个元素所在的位置。数据的值
    参数已调整，以便数据指向第一个元素
    在数组中。因此，本机数组的第一个字节位于
    `data - byte_offset`.

返回`napi_ok`如果 API 成功。

此 API 返回类型化数组的各种属性。

任何 out 参数可以是`NULL`如果不需要该属性。

*警告*：使用此 API 时要小心，因为底层数据缓冲区
由 VM 管理。

#### `napi_get_dataview_info`

<!-- YAML
added: v8.3.0
napiVersion: 1
-->

```c
napi_status napi_get_dataview_info(napi_env env,
                                   napi_value dataview,
                                   size_t* byte_length,
                                   void** data,
                                   napi_value* arraybuffer,
                                   size_t* byte_offset)
```

*   `[in] env`：调用 API 的环境。
*   `[in] dataview`:`napi_value`代表`DataView`谁的
    要查询的属性。
*   `[out] byte_length`：字节数`DataView`.
*   `[out] data`：底层的数据缓冲区`DataView`.
    如果byte_length是`0`，这可能是`NULL`或任何其他指针值。
*   `[out] arraybuffer`:`ArrayBuffer`基础`DataView`.
*   `[out] byte_offset`：数据缓冲区内的字节偏移量，从
    开始投影`DataView`.

返回`napi_ok`如果 API 成功。

任何 out 参数可以是`NULL`如果不需要该属性。

此 API 返回`DataView`.

#### `napi_get_date_value`

<!-- YAML
added:
 - v11.11.0
 - v10.17.0
napiVersion: 5
-->

```c
napi_status napi_get_date_value(napi_env env,
                                napi_value value,
                                double* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`Date`.
*   `[out] result`：时间值作为`double`表示为毫秒
    1970 年 1 月 1 日 UTC 开始的午夜。

此 API 不观察闰秒;它们被忽略，因为
ECMAScript 与 POSIX 时间规范保持一致。

返回`napi_ok`如果 API 成功。如果非日期`napi_value`已通过
在其中返回`napi_date_expected`.

此 API 返回给定 JavaScript 的时间值的 C 双基元
`Date`.

#### `napi_get_value_bool`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_bool(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`Boolean`.
*   `[out] result`：C 布尔基元，相当于给定的 JavaScript
    `Boolean`.

返回`napi_ok`如果 API 成功。如果非布尔值`napi_value`是
传入的返回`napi_boolean_expected`.

此 API 返回给定 JavaScript 的 C 布尔基元等效值
`Boolean`.

#### `napi_get_value_double`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_double(napi_env env,
                                  napi_value value,
                                  double* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`number`.
*   `[out] result`：C 双原语，相当于给定的 JavaScript
    `number`.

返回`napi_ok`如果 API 成功。如果非数字`napi_value`已通过
在其中返回`napi_number_expected`.

此 API 返回 C 双基元，等效于给定的 JavaScript
`number`.

#### `napi_get_value_bigint_int64`

<!-- YAML
added: v10.7.0
napiVersion: 6
-->

```c
napi_status napi_get_value_bigint_int64(napi_env env,
                                        napi_value value,
                                        int64_t* result,
                                        bool* lossless);
```

*   `[in] env`：调用 API 的环境
*   `[in] value`:`napi_value`表示 JavaScript`BigInt`.
*   `[out] result`： C`int64_t`原始等效于给定的 JavaScript
    `BigInt`.
*   `[out] lossless`：指示`BigInt`值已转换
    无损。

返回`napi_ok`如果 API 成功。如果非-`BigInt`在其中传递
返回`napi_bigint_expected`.

此 API 返回 C`int64_t`原始等效于给定的 JavaScript
`BigInt`.如果需要，它将截断该值，设置`lossless`自`false`.

#### `napi_get_value_bigint_uint64`

<!-- YAML
added: v10.7.0
napiVersion: 6
-->

```c
napi_status napi_get_value_bigint_uint64(napi_env env,
                                        napi_value value,
                                        uint64_t* result,
                                        bool* lossless);
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`BigInt`.
*   `[out] result`： C`uint64_t`原始等效于给定的 JavaScript
    `BigInt`.
*   `[out] lossless`：指示`BigInt`值已转换
    无损。

返回`napi_ok`如果 API 成功。如果非-`BigInt`在其中传递
返回`napi_bigint_expected`.

此 API 返回 C`uint64_t`原始等效于给定的 JavaScript
`BigInt`.如果需要，它将截断该值，设置`lossless`自`false`.

#### `napi_get_value_bigint_words`

<!-- YAML
added: v10.7.0
napiVersion: 6
-->

```c
napi_status napi_get_value_bigint_words(napi_env env,
                                        napi_value value,
                                        int* sign_bit,
                                        size_t* word_count,
                                        uint64_t* words);
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`BigInt`.
*   `[out] sign_bit`：表示如果 JavaScript 的整数`BigInt`为阳性
    或阴性。
*   `[in/out] word_count`：必须初始化为`words`
    数组。返回后，它将设置为实际字数
    需要存储此`BigInt`.
*   `[out] words`：指向预分配的 64 位字数组的指针。

返回`napi_ok`如果 API 成功。

此 API 转换单个`BigInt`值转换为符号位，64 位小端序
数组，以及数组中的元素数。`sign_bit`和`words`可能
两者都设置为`NULL`，以便仅获取`word_count`.

#### `napi_get_value_external`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_external(napi_env env,
                                    napi_value value,
                                    void** result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript 外部值。
*   `[out] result`：指向由 JavaScript 外部值包装的数据的指针。

返回`napi_ok`如果 API 成功。如果非外部`napi_value`是
传入的返回`napi_invalid_arg`.

此 API 检索先前传递给的外部数据指针
`napi_create_external()`.

#### `napi_get_value_int32`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_int32(napi_env env,
                                 napi_value value,
                                 int32_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`number`.
*   `[out] result`： C`int32`原始等效于给定的 JavaScript
    `number`.

返回`napi_ok`如果 API 成功。如果非数字`napi_value`
传入`napi_number_expected`.

此 API 返回 C`int32`原始等效项
给定的 JavaScript`number`.

如果数字超过 32 位整数的范围，则结果为
截断为相当于底部 32 位。这可能导致
如果值> 2，则正数变为负数<sup>31</sup>- 1.

非有限数值 （`NaN`,`+Infinity`或`-Infinity`） 设置
结果为零。

#### `napi_get_value_int64`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_int64(napi_env env,
                                 napi_value value,
                                 int64_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`number`.
*   `[out] result`： C`int64`原始等效于给定的 JavaScript
    `number`.

返回`napi_ok`如果 API 成功。如果非数字`napi_value`
在中传递，返回`napi_number_expected`.

此 API 返回 C`int64`原始等效于给定的 JavaScript
`number`.

`number`超出范围的值[`Number.MIN_SAFE_INTEGER`][Number.MIN_SAFE_INTEGER]
`-(2**53 - 1)`-[`Number.MAX_SAFE_INTEGER`][Number.MAX_SAFE_INTEGER] `(2**53 - 1)`将损失
精度。

非有限数值 （`NaN`,`+Infinity`或`-Infinity`） 设置
结果为零。

#### `napi_get_value_string_latin1`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_string_latin1(napi_env env,
                                         napi_value value,
                                         char* buf,
                                         size_t bufsize,
                                         size_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript 字符串。
*   `[in] buf`：用于将 ISO-8859-1 编码的字符串写入其中的缓冲区。如果`NULL`是
    传入，字符串的长度（以字节为单位），并排除空终止符
    返回于`result`.
*   `[in] bufsize`：目标缓冲区的大小。当此值为
    不足时，返回的字符串将被截断并以空值终止。
*   `[out] result`：复制到缓冲区的字节数，不包括 null
    终结者。

返回`napi_ok`如果 API 成功。如果非-`string` `napi_value`
在中传递，返回`napi_string_expected`.

此 API 返回与传递的值对应的 ISO-8859-1 编码字符串
在。

#### `napi_get_value_string_utf8`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_string_utf8(napi_env env,
                                       napi_value value,
                                       char* buf,
                                       size_t bufsize,
                                       size_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript 字符串。
*   `[in] buf`：用于将 UTF8 编码的字符串写入其中的缓冲区。如果`NULL`已通过
    in 中，字符串的长度（以字节为单位，不包括空终止符）为
    返回于`result`.
*   `[in] bufsize`：目标缓冲区的大小。当此值为
    不足时，返回的字符串将被截断并以空值终止。
*   `[out] result`：复制到缓冲区的字节数，不包括 null
    终结者。

返回`napi_ok`如果 API 成功。如果非-`string` `napi_value`
在中传递，返回`napi_string_expected`.

此 API 返回与传入的值对应的 UTF8 编码字符串。

#### `napi_get_value_string_utf16`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_string_utf16(napi_env env,
                                        napi_value value,
                                        char16_t* buf,
                                        size_t bufsize,
                                        size_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript 字符串。
*   `[in] buf`：用于将 UTF16-LE 编码的字符串写入其中的缓冲区。如果`NULL`是
    传入，字符串的长度以 2 字节代码单位为单位，不包括
    返回空终止符。
*   `[in] bufsize`：目标缓冲区的大小。当此值为
    不足时，返回的字符串将被截断并以空值终止。
*   `[out] result`：复制到缓冲区中的 2 字节代码单元数，不包括
    空终止符。

返回`napi_ok`如果 API 成功。如果非-`string` `napi_value`
在中传递，返回`napi_string_expected`.

此 API 返回与传入的值对应的 UTF16 编码字符串。

#### `napi_get_value_uint32`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_value_uint32(napi_env env,
                                  napi_value value,
                                  uint32_t* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`:`napi_value`表示 JavaScript`number`.
*   `[out] result`：C 原语，相当于给定的`napi_value`奥斯纳
    `uint32_t`.

返回`napi_ok`如果 API 成功。如果非数字`napi_value`
在中传递，返回`napi_number_expected`.

此 API 返回给定的 C 原语等效项`napi_value`奥斯纳
`uint32_t`.

### 获取全局实例的函数

#### `napi_get_boolean`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_boolean(napi_env env, bool value, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检索的布尔值。
*   `[out] result`:`napi_value`表示 JavaScript`Boolean`单例到
    取回。

返回`napi_ok`如果 API 成功。

此 API 用于返回 JavaScript 单例对象，该对象用于
表示给定的布尔值。

#### `napi_get_global`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_global(napi_env env, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`:`napi_value`表示 JavaScript`global`对象。

返回`napi_ok`如果 API 成功。

此 API 返回`global`对象。

#### `napi_get_null`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_null(napi_env env, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`:`napi_value`表示 JavaScript`null`对象。

返回`napi_ok`如果 API 成功。

此 API 返回`null`对象。

#### `napi_get_undefined`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_undefined(napi_env env, napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`:`napi_value`表示 JavaScript 未定义值。

返回`napi_ok`如果 API 成功。

此 API 返回未定义的对象。

## 使用 JavaScript 值和抽象操作

Node-API公开了一组API，用于在JavaScript上执行一些抽象操作
值。其中一些操作记录在[第7款][Section 7]
的[ECMAScript 语言规范][ECMAScript Language Specification].

这些 API 支持执行下列操作之一：

1.  将 JavaScript 值强制为特定的 JavaScript 类型（例如`number`或
    `string`).
2.  检查 JavaScript 值的类型。
3.  检查两个 JavaScript 值之间的相等性。

### `napi_coerce_to_bool`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_coerce_to_bool(napi_env env,
                                napi_value value,
                                napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要强制的 JavaScript 值。
*   `[out] result`:`napi_value`表示被强制的 JavaScript`Boolean`.

返回`napi_ok`如果 API 成功。

此 API 实现抽象操作`ToBoolean()`如 中所定义
[第7.1.2节][Section 7.1.2]的 ECMAScript 语言规范。

### `napi_coerce_to_number`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_coerce_to_number(napi_env env,
                                  napi_value value,
                                  napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要强制的 JavaScript 值。
*   `[out] result`:`napi_value`表示被强制的 JavaScript`number`.

返回`napi_ok`如果 API 成功。

此 API 实现抽象操作`ToNumber()`如 中所定义
[第7.1.3节][Section 7.1.3]的 ECMAScript 语言规范。
如果传入的值是
对象。

### `napi_coerce_to_object`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_coerce_to_object(napi_env env,
                                  napi_value value,
                                  napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要强制的 JavaScript 值。
*   `[out] result`:`napi_value`表示被强制的 JavaScript`Object`.

返回`napi_ok`如果 API 成功。

此 API 实现抽象操作`ToObject()`如 中所定义
[部分 7.1.13][Section 7.1.13]的 ECMAScript 语言规范。

### `napi_coerce_to_string`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_coerce_to_string(napi_env env,
                                  napi_value value,
                                  napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要强制的 JavaScript 值。
*   `[out] result`:`napi_value`表示被强制的 JavaScript`string`.

返回`napi_ok`如果 API 成功。

此 API 实现抽象操作`ToString()`如 中所定义
[部分 7.1.13][Section 7.1.13]的 ECMAScript 语言规范。
如果传入的值是
对象。

### `napi_typeof`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_typeof(napi_env env, napi_value value, napi_valuetype* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要查询其类型的 JavaScript 值。
*   `[out] result`：JavaScript 值的类型。

返回`napi_ok`如果 API 成功。

*   `napi_invalid_arg`如果的类型`value`不是已知的 ECMAScript 类型，并且
    `value`不是外部值。

此 API 表示类似于调用`typeof`操作员打开
中定义的对象[部分 12.5.5][Section 12.5.5]的 ECMAScript 语言
规范。但是，存在一些差异：

1.  它支持检测外部值。
2.  它检测`null`作为单独的类型，而 ECMAScript`typeof`会检测
    `object`.

如果`value`具有无效的类型，则返回错误。

### `napi_instanceof`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_instanceof(napi_env env,
                            napi_value object,
                            napi_value constructor,
                            bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] object`：要检查的 JavaScript 值。
*   `[in] constructor`：构造函数的 JavaScript 函数对象
    以检查。
*   `[out] result`：如果设置为 true 的布尔值`object instanceof constructor`
    是真的。

返回`napi_ok`如果 API 成功。

此 API 表示调用`instanceof`对象上的运算符为
定义于[部分 12.10.4][Section 12.10.4]的 ECMAScript 语言规范。

### `napi_is_array`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_is_array(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：给定对象是否为数组。

返回`napi_ok`如果 API 成功。

此 API 表示调用`IsArray`对对象执行的操作
如 中所定义[第7.2.2节][Section 7.2.2]的 ECMAScript 语言规范。

### `napi_is_arraybuffer`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_is_arraybuffer(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：给定对象是否为`ArrayBuffer`.

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入的是数组缓冲区。

### `napi_is_buffer`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_is_buffer(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：是否给定`napi_value`表示`node::Buffer`
    对象。

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入的是缓冲区。

### `napi_is_date`

<!-- YAML
added:
 - v11.11.0
 - v10.17.0
napiVersion: 5
-->

```c
napi_status napi_is_date(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：是否给定`napi_value`表示一个 JavaScript`Date`
    对象。

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入是一个日期。

### `napi_is_error`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_is_error(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：是否给定`napi_value`表示`Error`对象。

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入是一个`Error`.

### `napi_is_typedarray`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_is_typedarray(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：是否给定`napi_value`表示`TypedArray`.

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入的是一个类型化数组。

### `napi_is_dataview`

<!-- YAML
added: v8.3.0
napiVersion: 1
-->

```c
napi_status napi_is_dataview(napi_env env, napi_value value, bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的 JavaScript 值。
*   `[out] result`：是否给定`napi_value`表示`DataView`.

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入是一个`DataView`.

### `napi_strict_equals`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_strict_equals(napi_env env,
                               napi_value lhs,
                               napi_value rhs,
                               bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] lhs`：要检查的 JavaScript 值。
*   `[in] rhs`：要检查的 JavaScript 值。
*   `[out] result`：是否两者`napi_value`对象相等。

返回`napi_ok`如果 API 成功。

此 API 表示对严格相等算法的调用
定义于[部分 7.2.14][Section 7.2.14]的 ECMAScript 语言规范。

### `napi_detach_arraybuffer`

<!-- YAML
added:
 - v13.0.0
 - v12.16.0
 - v10.22.0
napiVersion: 7
-->

```c
napi_status napi_detach_arraybuffer(napi_env env,
                                    napi_value arraybuffer)
```

*   `[in] env`：调用 API 的环境。
*   `[in] arraybuffer`： JavaScript`ArrayBuffer`要分离。

返回`napi_ok`如果 API 成功。如果是不可拆卸的`ArrayBuffer`是
传入的返回`napi_detachable_arraybuffer_expected`.

通常，`ArrayBuffer`如果之前已分离过，则不可拆卸。
发动机可能会对`ArrayBuffer`是
可拆卸。例如，V8 要求`ArrayBuffer`是外部的，
也就是说，创建[`napi_create_external_arraybuffer`][napi_create_external_arraybuffer].

此 API 表示对`ArrayBuffer`分离操作作为
定义于[部分 24.1.1.3][Section 24.1.1.3]的 ECMAScript 语言规范。

### `napi_is_detached_arraybuffer`

<!-- YAML
added:
 - v13.3.0
 - v12.16.0
 - v10.22.0
napiVersion: 7
-->

```c
napi_status napi_is_detached_arraybuffer(napi_env env,
                                         napi_value arraybuffer,
                                         bool* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] arraybuffer`： JavaScript`ArrayBuffer`进行检查。
*   `[out] result`：是否`arraybuffer`已分离。

返回`napi_ok`如果 API 成功。

这`ArrayBuffer`如果其内部数据为`null`.

此 API 表示对`ArrayBuffer` `IsDetachedBuffer`
操作如 中定义[部分 24.1.1.2][Section 24.1.1.2]的 ECMAScript 语言
规范。

## 使用 JavaScript 属性

Node-API 公开了一组 API 来获取和设置 JavaScript 上的属性
对象。其中一些类型记录在[第7款][Section 7]的
[ECMAScript 语言规范][ECMAScript Language Specification].

JavaScript 中的属性表示为键和值的元组。
从根本上说，Node-API 中的所有属性键都可以表示为
以下形式：

*   命名：一个简单的 UTF8 编码字符串
*   整数索引：由`uint32_t`
*   JavaScript 值：这些在 Node-API 中由`napi_value`.这可以
    德发`napi_value`表示`string`,`number`或`symbol`.

节点 API 值由类型表示`napi_value`.
任何需要 JavaScript 值的 Node-API 调用都会采用`napi_value`.
但是，呼叫者有责任确保
`napi_value`有问题的是API所期望的JavaScript类型。

本节中介绍的 API 提供了一个简单的接口，用于
获取并设置任意 JavaScript 对象的属性，表示
`napi_value`.

例如，考虑以下 JavaScript 代码片段：

```js
const obj = {};
obj.myProp = 123;
```

可以使用具有以下代码段的 Node-API 值来完成等效项：

```c
napi_status status = napi_generic_failure;

// const obj = {}
napi_value obj, value;
status = napi_create_object(env, &obj);
if (status != napi_ok) return status;

// Create a napi_value for 123
status = napi_create_int32(env, 123, &value);
if (status != napi_ok) return status;

// obj.myProp = 123
status = napi_set_named_property(env, obj, "myProp", value);
if (status != napi_ok) return status;
```

可以采用类似的方式设置索引属性。请考虑以下事项
JavaScript 片段：

```js
const arr = [];
arr[123] = 'hello';
```

可以使用具有以下代码段的 Node-API 值来完成等效项：

```c
napi_status status = napi_generic_failure;

// const arr = [];
napi_value arr, value;
status = napi_create_array(env, &arr);
if (status != napi_ok) return status;

// Create a napi_value for 'hello'
status = napi_create_string_utf8(env, "hello", NAPI_AUTO_LENGTH, &value);
if (status != napi_ok) return status;

// arr[123] = 'hello';
status = napi_set_element(env, arr, 123, value);
if (status != napi_ok) return status;
```

可以使用本节中介绍的 API 检索属性。
请考虑以下 JavaScript 代码段：

```js
const arr = [];
const value = arr[123];
```

以下是 Node-API 对应项的近似等效项：

```c
napi_status status = napi_generic_failure;

// const arr = []
napi_value arr, value;
status = napi_create_array(env, &arr);
if (status != napi_ok) return status;

// const value = arr[123]
status = napi_get_element(env, arr, 123, &value);
if (status != napi_ok) return status;
```

最后，还可以在一个对象上定义多个属性以提高性能
原因。考虑以下 JavaScript：

```js
const obj = {};
Object.defineProperties(obj, {
  'foo': { value: 123, writable: true, configurable: true, enumerable: true },
  'bar': { value: 456, writable: true, configurable: true, enumerable: true }
});
```

以下是 Node-API 对应项的近似等效项：

```c
napi_status status = napi_status_generic_failure;

// const obj = {};
napi_value obj;
status = napi_create_object(env, &obj);
if (status != napi_ok) return status;

// Create napi_values for 123 and 456
napi_value fooValue, barValue;
status = napi_create_int32(env, 123, &fooValue);
if (status != napi_ok) return status;
status = napi_create_int32(env, 456, &barValue);
if (status != napi_ok) return status;

// Set the properties
napi_property_descriptor descriptors[] = {
  { "foo", NULL, NULL, NULL, NULL, fooValue, napi_writable | napi_configurable, NULL },
  { "bar", NULL, NULL, NULL, NULL, barValue, napi_writable | napi_configurable, NULL }
}
status = napi_define_properties(env,
                                obj,
                                sizeof(descriptors) / sizeof(descriptors[0]),
                                descriptors);
if (status != napi_ok) return status;
```

### 结构

#### `napi_property_attributes`

<!-- YAML
changes:
 - version: v14.12.0
   pr-url: https://github.com/nodejs/node/pull/35214
   description: added `napi_default_method` and `napi_default_property`.
-->

```c
typedef enum {
  napi_default = 0,
  napi_writable = 1 << 0,
  napi_enumerable = 1 << 1,
  napi_configurable = 1 << 2,

  // Used with napi_define_class to distinguish static properties
  // from instance properties. Ignored by napi_define_properties.
  napi_static = 1 << 10,

  // Default for class methods.
  napi_default_method = napi_writable | napi_configurable,

  // Default for object properties, like in JS obj[prop].
  napi_default_jsproperty = napi_writable |
                          napi_enumerable |
                          napi_configurable,
} napi_property_attributes;
```

`napi_property_attributes`是用于控制属性行为的标志
在 JavaScript 对象上设置。以外`napi_static`它们对应于
中列出的属性[部分 6.1.7.1][Section 6.1.7.1]
的[ECMAScript 语言规范][ECMAScript Language Specification].
它们可以是以下一个或多个位标记：

*   `napi_default`：未在属性上设置显式属性。默认情况下，一个
    属性是只读的，不可枚举且不可配置。
*   `napi_writable`：属性是可写的。
*   `napi_enumerable`：属性是可枚举的。
*   `napi_configurable`：属性可按照中的定义进行配置
    [部分 6.1.7.1][Section 6.1.7.1]的[ECMAScript 语言规范][ECMAScript Language Specification].
*   `napi_static`：该属性将被定义为类上的静态属性，如下所示
    与实例属性相反，实例属性是默认值。这仅由
    [`napi_define_class`][napi_define_class].它被忽略`napi_define_properties`.
*   `napi_default_method`：与 JS 类中的方法一样，属性为
    可配置和可写，但不可枚举。
*   `napi_default_jsproperty`：就像在 JavaScript 中通过赋值设置的属性一样，
    该属性是可写的、可枚举的和可配置的。

#### `napi_property_descriptor`

```c
typedef struct {
  // One of utf8name or name should be NULL.
  const char* utf8name;
  napi_value name;

  napi_callback method;
  napi_callback getter;
  napi_callback setter;
  napi_value value;

  napi_property_attributes attributes;
  void* data;
} napi_property_descriptor;
```

*   `utf8name`：描述属性键的可选字符串，
    编码为 UTF8。其中之一`utf8name`或`name`必须为
    财产。
*   `name`： 可选`napi_value`指向 JavaScript 字符串或符号
    用作住宿的钥匙。其中之一`utf8name`或`name`必须
    为酒店提供。
*   `value`：由获取属性的访问权限检索到的值，如果
    属性是数据属性。如果传入，请设置`getter`,`setter`,
    `method`和`data`自`NULL`（因为这些成员不会被使用）。
*   `getter`：执行属性的获取访问权限时要调用的函数。
    如果传入，请设置`value`和`method`自`NULL`（因为这些成员
    不会被使用）。给定函数由运行时在以下情况下隐式调用：
    该属性是从 JavaScript 代码访问的（或者如果属性上的 get 是
    使用 Node-API 调用执行）。[`napi_callback`][napi_callback]提供了更多详细信息。
*   `setter`：执行属性的设置访问时要调用的函数。
    如果传入，请设置`value`和`method`自`NULL`（因为这些成员
    不会被使用）。给定函数由运行时在以下情况下隐式调用：
    该属性是从 JavaScript 代码设置的（或者如果该属性上的一个集合是
    使用 Node-API 调用执行）。[`napi_callback`][napi_callback]提供了更多详细信息。
*   `method`：设置此项可使属性描述符对象的`value`
    属性是表示的 JavaScript 函数`method`.如果这是
    传入，设置`value`,`getter`和`setter`自`NULL`（因为这些成员
    不会被使用）。[`napi_callback`][napi_callback]提供了更多详细信息。
*   `attributes`：与特定属性关联的属性。看
    [`napi_property_attributes`][napi_property_attributes].
*   `data`：传入的回调数据`method`,`getter`和`setter`如果这是
    调用函数。

### 功能

#### `napi_get_property_names`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_property_names(napi_env env,
                                    napi_value object,
                                    napi_value* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中检索属性的对象。
*   `[out] result`： A`napi_value`表示 JavaScript 值的数组
    表示对象的属性名称。该接口可用于
    遍历`result`用[`napi_get_array_length`][napi_get_array_length]
    和[`napi_get_element`][napi_get_element].

返回`napi_ok`如果 API 成功。

此 API 返回`object`作为数组
的字符串。属性`object`其键是符号将不是
包括。

#### `napi_get_all_property_names`

<!-- YAML
added:
 - v13.7.0
 - v12.17.0
 - v10.20.0
napiVersion: 6
-->

```c
napi_get_all_property_names(napi_env env,
                            napi_value object,
                            napi_key_collection_mode key_mode,
                            napi_key_filter key_filter,
                            napi_key_conversion key_conversion,
                            napi_value* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中检索属性的对象。
*   `[in] key_mode`：是否也检索原型属性。
*   `[in] key_filter`：要检索的属性
    （可枚举/可读/可写）。
*   `[in] key_conversion`：是否将编号的属性键转换为字符串。
*   `[out] result`： A`napi_value`表示 JavaScript 值的数组
    表示对象的属性名称。[`napi_get_array_length`][napi_get_array_length]
    和[`napi_get_element`][napi_get_element]可用于迭代`result`.

返回`napi_ok`如果 API 成功。

此 API 返回一个数组，其中包含可用属性的名称
此对象。

#### `napi_set_property`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_set_property(napi_env env,
                              napi_value object,
                              napi_value key,
                              napi_value value);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要在其上设置属性的对象。
*   `[in] key`：要设置的属性的名称。
*   `[in] value`：属性值。

返回`napi_ok`如果 API 成功。

此 API 在`Object`转会了。

#### `napi_get_property`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_property(napi_env env,
                              napi_value object,
                              napi_value key,
                              napi_value* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中检索属性的对象。
*   `[in] key`：要检索的属性的名称。
*   `[out] result`：属性的值。

返回`napi_ok`如果 API 成功。

此 API 从`Object`转会了。

#### `napi_has_property`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_has_property(napi_env env,
                              napi_value object,
                              napi_value key,
                              bool* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要查询的对象。
*   `[in] key`：要检查其是否存在的属性的名称。
*   `[out] result`：属性是否存在于对象上。

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入具有命名属性。

#### `napi_delete_property`

<!-- YAML
added: v8.2.0
napiVersion: 1
-->

```c
napi_status napi_delete_property(napi_env env,
                                 napi_value object,
                                 napi_value key,
                                 bool* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要查询的对象。
*   `[in] key`：要删除的属性的名称。
*   `[out] result`：属性删除是否成功。`result`能
    （可选）通过传递忽略`NULL`.

返回`napi_ok`如果 API 成功。

此 API 尝试删除`key`拥有的财产从`object`.

#### `napi_has_own_property`

<!-- YAML
added: v8.2.0
napiVersion: 1
-->

```c
napi_status napi_has_own_property(napi_env env,
                                  napi_value object,
                                  napi_value key,
                                  bool* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要查询的对象。
*   `[in] key`：要检查其是否存在的自有属性的名称。
*   `[out] result`：对象上是否存在自己的属性。

返回`napi_ok`如果 API 成功。

此 API 检查`Object`传入具有命名的自己的属性。`key`必须
成为`string`或`symbol`，否则将引发错误。Node-API 不会
执行数据类型之间的任何转换。

#### `napi_set_named_property`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_set_named_property(napi_env env,
                                    napi_value object,
                                    const char* utf8Name,
                                    napi_value value);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要在其上设置属性的对象。
*   `[in] utf8Name`：要设置的属性的名称。
*   `[in] value`：属性值。

返回`napi_ok`如果 API 成功。

此方法等效于调用[`napi_set_property`][napi_set_property]与`napi_value`
从传入的字符串创建为`utf8Name`.

#### `napi_get_named_property`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_named_property(napi_env env,
                                    napi_value object,
                                    const char* utf8Name,
                                    napi_value* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中检索属性的对象。
*   `[in] utf8Name`：要获取的属性的名称。
*   `[out] result`：属性的值。

返回`napi_ok`如果 API 成功。

此方法等效于调用[`napi_get_property`][napi_get_property]与`napi_value`
从传入的字符串创建为`utf8Name`.

#### `napi_has_named_property`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_has_named_property(napi_env env,
                                    napi_value object,
                                    const char* utf8Name,
                                    bool* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要查询的对象。
*   `[in] utf8Name`：要检查其是否存在的属性的名称。
*   `[out] result`：属性是否存在于对象上。

返回`napi_ok`如果 API 成功。

此方法等效于调用[`napi_has_property`][napi_has_property]与`napi_value`
从传入的字符串创建为`utf8Name`.

#### `napi_set_element`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_set_element(napi_env env,
                             napi_value object,
                             uint32_t index,
                             napi_value value);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中设置属性的对象。
*   `[in] index`：要设置的属性的索引。
*   `[in] value`：属性值。

返回`napi_ok`如果 API 成功。

此 API 在`Object`转会了。

#### `napi_get_element`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_element(napi_env env,
                             napi_value object,
                             uint32_t index,
                             napi_value* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中检索属性的对象。
*   `[in] index`：要获取的属性的索引。
*   `[out] result`：属性的值。

返回`napi_ok`如果 API 成功。

此 API 获取请求索引处的元素。

#### `napi_has_element`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_has_element(napi_env env,
                             napi_value object,
                             uint32_t index,
                             bool* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要查询的对象。
*   `[in] index`：要检查其是否存在的属性的索引。
*   `[out] result`：属性是否存在于对象上。

返回`napi_ok`如果 API 成功。

如果`Object`传入在
请求的索引。

#### `napi_delete_element`

<!-- YAML
added: v8.2.0
napiVersion: 1
-->

```c
napi_status napi_delete_element(napi_env env,
                                napi_value object,
                                uint32_t index,
                                bool* result);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要查询的对象。
*   `[in] index`：要删除的属性的索引。
*   `[out] result`：元素删除是否成功。`result`能
    （可选）通过传递忽略`NULL`.

返回`napi_ok`如果 API 成功。

此 API 尝试删除指定的`index`从`object`.

#### `napi_define_properties`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_define_properties(napi_env env,
                                   napi_value object,
                                   size_t property_count,
                                   const napi_property_descriptor* properties);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要从中检索属性的对象。
*   `[in] property_count`：元素的数量`properties`数组。
*   `[in] properties`：属性描述符的数组。

返回`napi_ok`如果 API 成功。

此方法允许在给定的多个属性上进行有效定义
对象。属性是使用属性描述符定义的（请参见
[`napi_property_descriptor`][napi_property_descriptor]).给定一个此类属性描述符数组，
此 API 将一次设置一个对象的属性，如
`DefineOwnProperty()`（描述于[第9.1.6节][Section 9.1.6]的 ECMA-262
规格）。

#### `napi_object_freeze`

<!-- YAML
added:
  - v14.14.0
  - v12.20.0
napiVersion: 8
-->

```c
napi_status napi_object_freeze(napi_env env,
                               napi_value object);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要冻结的对象。

返回`napi_ok`如果 API 成功。

此方法冻结给定的对象。这可以防止新属性
添加到其中，现有属性被删除，阻止
更改现有可枚举性、可配置性或可写性
属性，并防止更改现有属性的值。
它还可以防止对象的原型被更改。对此进行了描述
在[部分 19.1.2.6](https://tc39.es/ecma262/#sec-object.freeze)的
ECMA-262 规范。

#### `napi_object_seal`

<!-- YAML
added:
  - v14.14.0
  - v12.20.0
napiVersion: 8
-->

```c
napi_status napi_object_seal(napi_env env,
                             napi_value object);
```

*   `[in] env`：调用节点 API 调用的环境。
*   `[in] object`：要密封的物体。

返回`napi_ok`如果 API 成功。

此方法密封给定对象。这可以防止新属性成为
添加到其中，以及将所有现有属性标记为不可配置。
这在 中进行了描述[部分 19.1.2.20](https://tc39.es/ecma262/#sec-object.seal)
的 ECMA-262 规范。

## 使用 JavaScript 函数

Node-API 提供了一组 API，允许 JavaScript 代码
回调到本机代码。支持回调的节点 API
进入本机代码，在回调函数中表示
这`napi_callback`类型。当 JavaScript VM 回调到
本机代码，`napi_callback`调用提供的函数。接口
本节中记录的允许回调函数执行
以后：

*   获取有关调用回调的上下文的信息。
*   获取传递到回调中的参数。
*   返回`napi_value`从回调返回。

此外，Node-API提供了一组允许调用的函数。
来自本机代码的 JavaScript 函数。可以调用函数
像常规的JavaScript函数调用，或作为构造函数
功能。

任何非`NULL`通过`data`的领域
`napi_property_descriptor`项目可以关联`object`并释放
每当`object`通过传递两者来收集垃圾`object`和数据到
[`napi_add_finalizer`][napi_add_finalizer].

### `napi_call_function`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_call_function(napi_env env,
                                           napi_value recv,
                                           napi_value func,
                                           size_t argc,
                                           const napi_value* argv,
                                           napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] recv`：`this`传递给被调用函数的值。
*   `[in] func`:`napi_value`表示要调用的 JavaScript 函数。
*   `[in] argc`：元素的计数`argv`数组。
*   `[in] argv`：数组`napi_values`表示传入的 JavaScript 值
    作为函数的参数。
*   `[out] result`:`napi_value`表示返回的 JavaScript 对象。

返回`napi_ok`如果 API 成功。

此方法允许从本机调用 JavaScript 函数对象
附加组件。这是回拨的主要机制*从*附加组件的
本机代码*到*JavaScript.对于调用 JavaScript 的特殊情况
在异步操作之后，请参阅[`napi_make_callback`][napi_make_callback].

示例用例可能如下所示。考虑以下 JavaScript
片段：

```js
function AddTwo(num) {
  return num + 2;
}
global.AddTwo = AddTwo;
```

然后，可以使用
以下代码：

```c
// Get the function named "AddTwo" on the global object
napi_value global, add_two, arg;
napi_status status = napi_get_global(env, &global);
if (status != napi_ok) return;

status = napi_get_named_property(env, global, "AddTwo", &add_two);
if (status != napi_ok) return;

// const arg = 1337
status = napi_create_int32(env, 1337, &arg);
if (status != napi_ok) return;

napi_value* argv = &arg;
size_t argc = 1;

// AddTwo(arg);
napi_value return_val;
status = napi_call_function(env, global, add_two, argc, argv, &return_val);
if (status != napi_ok) return;

// Convert the result back to a native type
int32_t result;
status = napi_get_value_int32(env, return_val, &result);
if (status != napi_ok) return;
```

### `napi_create_function`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_create_function(napi_env env,
                                 const char* utf8name,
                                 size_t length,
                                 napi_callback cb,
                                 void* data,
                                 napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] utf8Name`：编码为 UTF8 的函数的可选名称。这是
    在 JavaScript 中可见，作为新函数对象的`name`财产。
*   `[in] length`：长度`utf8name`以字节为单位，或`NAPI_AUTO_LENGTH`如果
    它是以空值终止的。
*   `[in] cb`：此函数时应调用的本机函数
    对象被调用。[`napi_callback`][napi_callback]提供了更多详细信息。
*   `[in] data`：用户提供的数据上下文。这将被传回
    稍后调用时的函数。
*   `[out] result`:`napi_value`表示 JavaScript 函数对象
    新创建的函数。

返回`napi_ok`如果 API 成功。

此 API 允许加载项作者在本机代码中创建函数对象。
这是允许呼叫的主要机制*到*加载项的本机代码
*从*JavaScript.

在此之后，新创建的函数不会自动从脚本中可见
叫。相反，必须在任何可见对象上显式设置属性
到 JavaScript，以便可以从脚本访问函数。

为了公开函数作为
附加组件的模块导出，在导出时设置新创建的函数
对象。示例模块可能如下所示：

```c
napi_value SayHello(napi_env env, napi_callback_info info) {
  printf("Hello\n");
  return NULL;
}

napi_value Init(napi_env env, napi_value exports) {
  napi_status status;

  napi_value fn;
  status = napi_create_function(env, NULL, 0, SayHello, NULL, &fn);
  if (status != napi_ok) return NULL;

  status = napi_set_named_property(env, exports, "sayHello", fn);
  if (status != napi_ok) return NULL;

  return exports;
}

NAPI_MODULE(NODE_GYP_MODULE_NAME, Init)
```

给定上面的代码，可以从JavaScript使用附加组件，如下所示：

```js
const myaddon = require('./addon');
myaddon.sayHello();
```

传递给的字符串`require()`是 中目标的名称`binding.gyp`
负责创建`.node`文件。

任何非`NULL`通过`data`参数可以
与生成的 JavaScript 函数相关联（该函数在
`result`参数），并在函数被垃圾回收时释放
将 JavaScript 函数和数据传递给[`napi_add_finalizer`][napi_add_finalizer].

JavaScript`Function`s 描述于[第19.2节][Section 19.2]的 ECMAScript
语言规范。

### `napi_get_cb_info`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_cb_info(napi_env env,
                             napi_callback_info cbinfo,
                             size_t* argc,
                             napi_value* argv,
                             napi_value* thisArg,
                             void** data)
```

*   `[in] env`：调用 API 的环境。
*   `[in] cbinfo`：传递到回调函数中的回调信息。
*   `[in-out] argc`：指定所提供的长度`argv`数组和
    接收参数的实际计数。`argc`能
    （可选）通过传递忽略`NULL`.
*   `[out] argv`： C 数组`napi_value`参数将位于
    复制。如果参数数多于提供的计数，则只有
    复制请求的参数数。如果参数较少
    提供比声称的，其余的`argv`填充了`napi_value`值
    代表`undefined`.`argv`可以选择性地忽略
    通过`NULL`.
*   `[out] this`：接收 JavaScript`this`参数。`this`
    可以选择通过传递来忽略`NULL`.
*   `[out] data`：接收回调的数据指针。`data`能
    （可选）通过传递忽略`NULL`.

返回`napi_ok`如果 API 成功。

此方法在回调函数中用于检索有关
调用像参数和`this`来自给定回调信息的指针。

### `napi_get_new_target`

<!-- YAML
added: v8.6.0
napiVersion: 1
-->

```c
napi_status napi_get_new_target(napi_env env,
                                napi_callback_info cbinfo,
                                napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] cbinfo`：传递到回调函数中的回调信息。
*   `[out] result`：`new.target`的构造函数调用。

返回`napi_ok`如果 API 成功。

此 API 返回`new.target`的构造函数调用。如果当前
回调不是构造函数调用，结果是`NULL`.

### `napi_new_instance`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_new_instance(napi_env env,
                              napi_value cons,
                              size_t argc,
                              napi_value* argv,
                              napi_value* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] cons`:`napi_value`表示要调用的 JavaScript 函数
    作为构造函数。
*   `[in] argc`：元素的计数`argv`数组。
*   `[in] argv`：JavaScript 值数组 as`napi_value`代表
    构造函数的参数。如果`argc`为零 此参数可以是
    通过传入省略`NULL`.
*   `[out] result`:`napi_value`表示返回的 JavaScript 对象，
    在本例中为构造对象。

此方法用于使用给定的参数实例化新的 JavaScript 值
`napi_value`表示对象的构造函数。例如
请考虑以下代码段：

```js
function MyObject(param) {
  this.param = param;
}

const arg = 'hello';
const value = new MyObject(arg);
```

可以使用以下代码段在 Node-API 中近似显示以下内容：

```c
// Get the constructor function MyObject
napi_value global, constructor, arg, value;
napi_status status = napi_get_global(env, &global);
if (status != napi_ok) return;

status = napi_get_named_property(env, global, "MyObject", &constructor);
if (status != napi_ok) return;

// const arg = "hello"
status = napi_create_string_utf8(env, "hello", NAPI_AUTO_LENGTH, &arg);
if (status != napi_ok) return;

napi_value* argv = &arg;
size_t argc = 1;

// const value = new MyObject(arg)
status = napi_new_instance(env, constructor, argc, argv, &value);
```

返回`napi_ok`如果 API 成功。

## 对象环绕

Node-API 提供了一种“包装”C++类和实例的方法，以便类
构造函数和方法可以从 JavaScript 调用。

1.  这[`napi_define_class`][napi_define_class]API定义了一个带有构造函数的JavaScript类，
    静态属性和方法，以及实例属性和方法
    对应于C++类。
2.  当 JavaScript 代码调用构造函数时，构造函数回调
    使用[`napi_wrap`][napi_wrap]将新的C++实例包装在 JavaScript 对象中，
    然后返回包装器对象。
3.  当 JavaScript 代码调用类上的方法或属性访问器时，
    相应的`napi_callback`调用C++函数。对于实例
    回调[`napi_unwrap`][napi_unwrap]获取作为目标的 C++ 实例
    呼叫。

对于包装对象，可能很难区分函数
在类原型上调用，在类的实例上调用函数。
用于解决此问题的常见模式是保存持久性
以后对类构造函数的引用`instanceof`检查。

```c
napi_value MyClass_constructor = NULL;
status = napi_get_reference_value(env, MyClass::es_constructor, &MyClass_constructor);
assert(napi_ok == status);
bool is_instance = false;
status = napi_instanceof(env, es_this, MyClass_constructor, &is_instance);
assert(napi_ok == status);
if (is_instance) {
  // napi_unwrap() ...
} else {
  // otherwise...
}
```

一旦不再需要该引用，就必须释放它。

在某些情况下，`napi_instanceof()`不足以确保
JavaScript 对象是某个本机类型的包装器。就是这种情况
特别是当包装的JavaScript对象通过
静态方法，而不是作为`this`原型方法的价值。在这样的
在某些情况下，它们可能会被错误地打开包装。

```js
const myAddon = require('./build/Release/my_addon.node');

// `openDatabase()` returns a JavaScript object that wraps a native database
// handle.
const dbHandle = myAddon.openDatabase();

// `query()` returns a JavaScript object that wraps a native query handle.
const queryHandle = myAddon.query(dbHandle, 'Gimme ALL the things!');

// There is an accidental error in the line below. The first parameter to
// `myAddon.queryHasRecords()` should be the database handle (`dbHandle`), not
// the query handle (`query`), so the correct condition for the while-loop
// should be
//
// myAddon.queryHasRecords(dbHandle, queryHandle)
//
while (myAddon.queryHasRecords(queryHandle, dbHandle)) {
  // retrieve records
}
```

在上面的示例中`myAddon.queryHasRecords()`是一种接受两个的方法
参数。第一个是数据库句柄，第二个是查询句柄。
在内部，它解开第一个参数的包装，并将生成的指针转换为
本机数据库句柄。然后，它解开第二个参数的包装，并转换
生成的指向查询句柄的指针。如果参数传递错误
秩序，演员阵容会起作用，但是，很有可能底层
数据库操作将失败，甚至会导致无效的内存访问。

确保从第一个参数中检索到的指针确实是指针
到数据库句柄，同样，指针从第二个句柄中检索
参数确实是指向查询句柄的指针，实现
`queryHasRecords()`必须执行类型验证。保留 JavaScript
从中实例化数据库句柄的类构造函数，以及
从中实例化查询句柄的构造函数`napi_ref`s 罐
帮助，因为`napi_instanceof()`然后，可以使用这些实例来确保实例
传递到`queryHashRecords()`确实是正确的类型。

不幸`napi_instanceof()`不能防止原型
操纵。例如，数据库句柄实例的原型可以是
设置为查询句柄实例的构造函数的原型。在此
在案例中，数据库句柄实例可以显示为查询句柄实例，并且
将通过`napi_instanceof()`测试查询句柄实例，同时仍然
包含指向数据库句柄的指针。

为此，Node-API 提供了类型标记功能。

类型标记是插件独有的 128 位整数。Node-API 提供了
`napi_type_tag`用于存储类型标记的结构。传递此类值时
以及存储在`napi_value`自
`napi_type_tag_object()`，JavaScript 对象将被“标记”为
类型标记。“标记”在JavaScript方面是不可见的。当一个 JavaScript
对象到达本机绑定，`napi_check_object_type_tag()`可以使用
以及原始类型标记，以确定 JavaScript 对象是否
以前用类型标记“标记”。这将创建类型检查功能
的保真度高于`napi_instanceof()`可以提供，因为这种类型-
标记在原型操作和插件卸载/重新加载中幸存下来。

继续上面的示例，下面的骨架插件实现
说明使用`napi_type_tag_object()`和
`napi_check_object_type_tag()`.

```c
// This value is the type tag for a database handle. The command
//
//   uuidgen | sed -r -e 's/-//g' -e 's/(.{16})(.*)/0x\1, 0x\2/'
//
// can be used to obtain the two values with which to initialize the structure.
static const napi_type_tag DatabaseHandleTypeTag = {
  0x1edf75a38336451d, 0xa5ed9ce2e4c00c38
};

// This value is the type tag for a query handle.
static const napi_type_tag QueryHandleTypeTag = {
  0x9c73317f9fad44a3, 0x93c3920bf3b0ad6a
};

static napi_value
openDatabase(napi_env env, napi_callback_info info) {
  napi_status status;
  napi_value result;

  // Perform the underlying action which results in a database handle.
  DatabaseHandle* dbHandle = open_database();

  // Create a new, empty JS object.
  status = napi_create_object(env, &result);
  if (status != napi_ok) return NULL;

  // Tag the object to indicate that it holds a pointer to a `DatabaseHandle`.
  status = napi_type_tag_object(env, result, &DatabaseHandleTypeTag);
  if (status != napi_ok) return NULL;

  // Store the pointer to the `DatabaseHandle` structure inside the JS object.
  status = napi_wrap(env, result, dbHandle, NULL, NULL, NULL);
  if (status != napi_ok) return NULL;

  return result;
}

// Later when we receive a JavaScript object purporting to be a database handle
// we can use `napi_check_object_type_tag()` to ensure that it is indeed such a
// handle.

static napi_value
query(napi_env env, napi_callback_info info) {
  napi_status status;
  size_t argc = 2;
  napi_value argv[2];
  bool is_db_handle;

  status = napi_get_cb_info(env, info, &argc, argv, NULL, NULL);
  if (status != napi_ok) return NULL;

  // Check that the object passed as the first parameter has the previously
  // applied tag.
  status = napi_check_object_type_tag(env,
                                      argv[0],
                                      &DatabaseHandleTypeTag,
                                      &is_db_handle);
  if (status != napi_ok) return NULL;

  // Throw a `TypeError` if it doesn't.
  if (!is_db_handle) {
    // Throw a TypeError.
    return NULL;
  }
}
```

### `napi_define_class`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_define_class(napi_env env,
                              const char* utf8name,
                              size_t length,
                              napi_callback constructor,
                              void* data,
                              size_t property_count,
                              const napi_property_descriptor* properties,
                              napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] utf8name`：JavaScript 构造函数的名称;包装时
    C++类，为清楚起见，我们建议此名称与
    C++类。
*   `[in] length`：长度`utf8name`以字节为单位，或`NAPI_AUTO_LENGTH`
    如果以空值终止。
*   `[in] constructor`：处理构造实例的回调函数
    的类。包装C++类时，此方法必须是静态成员
    与[`napi_callback`][napi_callback]签名。C++类构造函数不能是
    使用。[`napi_callback`][napi_callback]提供了更多详细信息。
*   `[in] data`：要传递给构造函数回调的可选数据
    这`data`属性。
*   `[in] property_count`：中的项目数`properties`数组参数。
*   `[in] properties`：描述静态和的属性描述符数组
    类上的实例数据属性、访问器和方法
    看`napi_property_descriptor`.
*   `[out] result`： A`napi_value`表示 的构造函数
    类。

返回`napi_ok`如果 API 成功。

定义一个 JavaScript 类，包括：

*   具有类名的 JavaScript 构造函数。包装时
    对应C++类，回调传递方式为`constructor`可用于
    实例化一个新的C++类实例，然后可以将其放置在
    使用正在构造的 JavaScript 对象实例[`napi_wrap`][napi_wrap].
*   其实现可以调用的构造函数的属性
    相应*静态的*C++的数据属性、访问器和方法
    类（由属性描述符定义，其中`napi_static`属性）。
*   构造函数的属性`prototype`对象。包装时
    C++类，*非静态*C++的数据属性、访问器和方法
    class 可以从属性中给定的静态函数调用
    不带 的描述符`napi_static`检索C++类后的属性
    使用
    [`napi_unwrap`][napi_unwrap].

包装C++类时，C++构造函数回调通过`constructor`
应该是调用实际类构造函数的类上的静态方法，
然后将新的C++实例包装在 JavaScript 对象中，并返回包装器
对象。看[`napi_wrap`][napi_wrap]了解详情。

从以下位置返回的 JavaScript 构造函数[`napi_define_class`][napi_define_class]是
通常保存并在以后用于从本机构造类的新实例
代码，和/或检查提供的值是否是类的实例。在
在这种情况下，为了防止函数值被垃圾回收，一个
可以使用以下命令创建对它的强持久引用
[`napi_create_reference`][napi_create_reference]，确保引用计数保持在 >= 1。

任何非`NULL`通过`data`参数或通过
这`data`的领域`napi_property_descriptor`数组项可以关联
使用生成的 JavaScript 构造函数（在`result`
参数），并在通过传递两者来垃圾回收类时释放
JavaScript 函数和数据[`napi_add_finalizer`][napi_add_finalizer].

### `napi_wrap`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_wrap(napi_env env,
                      napi_value js_object,
                      void* native_object,
                      napi_finalize finalize_cb,
                      void* finalize_hint,
                      napi_ref* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] js_object`：将成为
    本机对象。
*   `[in] native_object`：将包装在
    JavaScript 对象。
*   `[in] finalize_cb`：可选的本机回调，可用于释放
    JavaScript 对象准备好进行垃圾回收时的本机实例。
    [`napi_finalize`][napi_finalize]提供了更多详细信息。
*   `[in] finalize_hint`：传递给
    完成回调。
*   `[out] result`：对包装对象的可选引用。

返回`napi_ok`如果 API 成功。

将本机实例包装在 JavaScript 对象中。本机实例可以是
稍后使用检索`napi_unwrap()`.

当 JavaScript 代码调用使用
`napi_define_class()`这`napi_callback`，因为调用构造函数。
构造本机类的实例后，回调必须调用
`napi_wrap()`将新构造的实例包装在已创建的实例中
JavaScript 对象，即`this`构造函数回调的参数。
（那`this`对象是从构造函数的`prototype`,
所以它已经有了所有实例属性和方法的定义。

通常，在包装类实例时，finalize 回调应该是
前提是，只需删除作为`data`
参数以完成回调。

可选返回的引用最初是弱引用，这意味着它
引用计数为 0。通常，此引用计数将递增
在要求实例保持有效的异步操作期间临时执行。

*谨慎*：应通过以下方式删除可选的返回引用（如果已获得）
[`napi_delete_reference`][napi_delete_reference]仅用于响应最终确定的回调
调用。如果在此之前被删除，则 finalize 回调可能永远不会
被调用。因此，在获取引用时，finalize 回调也是
需要才能正确处置参考文献。

终结器回调可能会被推迟，留下一个窗口，其中对象具有
已被垃圾回收（弱引用无效），但终结器
尚未被调用。使用时`napi_get_reference_value()`弱
返回的引用`napi_wrap()`中，您仍应处理空结果。

叫`napi_wrap()`在对象上第二次将返回错误。自
将另一个本机实例与对象关联，使用`napi_remove_wrap()`
第一。

### `napi_unwrap`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_unwrap(napi_env env,
                        napi_value js_object,
                        void** result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] js_object`：与本机实例关联的对象。
*   `[out] result`：指向已包装的本机实例的指针。

返回`napi_ok`如果 API 成功。

检索以前包装在 JavaScript 中的本机实例
对象使用`napi_wrap()`.

当 JavaScript 代码调用类上的方法或属性访问器时，
相应`napi_callback`被调用。如果回调是针对实例的
方法或访问器，然后`this`回调的参数是包装器
对象;可以获得作为调用目标的包装C++实例
然后通过呼叫`napi_unwrap()`在包装器对象上。

### `napi_remove_wrap`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
napi_status napi_remove_wrap(napi_env env,
                             napi_value js_object,
                             void** result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] js_object`：与本机实例关联的对象。
*   `[out] result`：指向已包装的本机实例的指针。

返回`napi_ok`如果 API 成功。

检索以前包装在 JavaScript 中的本机实例
对象`js_object`用`napi_wrap()`并移除包装。如果最终确定
回调与换行相关联，当
JavaScript 对象被垃圾回收。

### `napi_type_tag_object`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
napiVersion: 8
-->

```c
napi_status napi_type_tag_object(napi_env env,
                                 napi_value js_object,
                                 const napi_type_tag* type_tag);
```

*   `[in] env`：调用 API 的环境。
*   `[in] js_object`：要标记的 JavaScript 对象。
*   `[in] type_tag`：用于标记对象的标记。

返回`napi_ok`如果 API 成功。

关联价值`type_tag`带有 JavaScript 对象的指针。
`napi_check_object_type_tag()`然后可用于比较
附加到对象上，并带有一个由插件拥有的对象，以确保该对象
具有正确的类型。

如果对象已具有关联的类型标记，则此 API 将返回
`napi_invalid_arg`.

### `napi_check_object_type_tag`

<!-- YAML
added:
  - v14.8.0
  - v12.19.0
napiVersion: 8
-->

```c
napi_status napi_check_object_type_tag(napi_env env,
                                       napi_value js_object,
                                       const napi_type_tag* type_tag,
                                       bool* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] js_object`：要检查其类型标记的 JavaScript 对象。
*   `[in] type_tag`：用于比较在对象上找到的任何标签的标签。
*   `[out] result`：给定的类型标签是否与
    对象。`false`如果在对象上未找到类型标记，也会返回。

返回`napi_ok`如果 API 成功。

将给定的指针比较为`type_tag`与任何可以在上找到的
`js_object`.如果在 上找不到标记`js_object`或者，如果找到标记但它确实存在
不匹配`type_tag`然后`result`设置为`false`.如果找到标记并且它
比赛`type_tag`然后`result`设置为`true`.

### `napi_add_finalizer`

<!-- YAML
added: v8.0.0
napiVersion: 5
-->

```c
napi_status napi_add_finalizer(napi_env env,
                               napi_value js_object,
                               void* native_object,
                               napi_finalize finalize_cb,
                               void* finalize_hint,
                               napi_ref* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] js_object`：本机数据将指向的 JavaScript 对象
    附加。
*   `[in] native_object`：将附加到 JavaScript 的本机数据
    对象。
*   `[in] finalize_cb`：将用于释放
    JavaScript 对象准备好进行垃圾回收时的本机数据。
    [`napi_finalize`][napi_finalize]提供了更多详细信息。
*   `[in] finalize_hint`：传递给
    完成回调。
*   `[out] result`：对 JavaScript 对象的可选引用。

返回`napi_ok`如果 API 成功。

添加`napi_finalize`当 JavaScript 对象调用时将调用的回调
在`js_object`已准备好进行垃圾回收。此 API 类似于
`napi_wrap()`除了：

*   以后无法检索本机数据`napi_unwrap()`,
*   以后也不能使用`napi_remove_wrap()`和
*   API可以使用不同的数据项多次调用，以便
    将它们中的每一个附加到 JavaScript 对象，以及
*   由 API 操作的对象可用于`napi_wrap()`.

*谨慎*：应通过以下方式删除可选的返回引用（如果已获得）
[`napi_delete_reference`][napi_delete_reference]仅用于响应最终确定的回调
调用。如果在此之前被删除，则 finalize 回调可能永远不会
被调用。因此，在获取引用时，finalize 回调也是
需要才能正确处置参考文献。

## 简单的异步操作

插件模块通常需要利用 libuv 中的异步帮助程序作为其
实现。这允许他们安排异步执行工作
以便他们的方法可以在工作完成之前返回。这
允许他们避免阻止 Node.js应用程序的整体执行。

Node-API为这些提供了一个ABI稳定的接口
支持函数，涵盖最常见的异步用例。

Node-API 定义了`napi_async_work`用于管理的结构
异步工作线程。创建/删除实例时
[`napi_create_async_work`][napi_create_async_work]和[`napi_delete_async_work`][napi_delete_async_work].

这`execute`和`complete`回调是将
当执行器准备好执行并且当它完成其
分别是任务。

这`execute`函数应避免进行任何节点 API 调用
这可能导致 JavaScript 的执行或与
JavaScript 对象。大多数情况下，任何需要制作Node-API的代码
应在`complete`改为回调。
避免使用`napi_env`参数在执行回调中为
它可能会执行JavaScript。

这些函数实现以下接口：

```c
typedef void (*napi_async_execute_callback)(napi_env env,
                                            void* data);
typedef void (*napi_async_complete_callback)(napi_env env,
                                             napi_status status,
                                             void* data);
```

调用这些方法时，`data`传递的参数将是
插件提供`void*`已传递到
`napi_create_async_work`叫。

创建异步工作线程后，可以排队
用于执行，请使用[`napi_queue_async_work`][napi_queue_async_work]功能：

```c
napi_status napi_queue_async_work(napi_env env,
                                  napi_async_work work);
```

[`napi_cancel_async_work`][napi_cancel_async_work]如果工作需要，可以使用
在工作开始执行之前取消。

呼叫后[`napi_cancel_async_work`][napi_cancel_async_work]这`complete`回调
将调用状态值为`napi_cancelled`.
该作品不应在`complete`
回调调用，即使它已被取消。

### `napi_create_async_work`

<!-- YAML
added: v8.0.0
napiVersion: 1
changes:
  - version: v8.6.0
    pr-url: https://github.com/nodejs/node/pull/14697
    description: Added `async_resource` and `async_resource_name` parameters.
-->

```c
napi_status napi_create_async_work(napi_env env,
                                   napi_value async_resource,
                                   napi_value async_resource_name,
                                   napi_async_execute_callback execute,
                                   napi_async_complete_callback complete,
                                   void* data,
                                   napi_async_work* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] async_resource`：与异步工作关联的可选对象
    这将传递到可能`async_hooks` [`init`钩][init hooks].
*   `[in] async_resource_name`：正在使用的资源类型的标识符
    提供了 由`async_hooks`应用程序接口。
*   `[in] execute`：应调用以执行
    异步逻辑。给定的函数是从工作线程池线程调用的
    并且可以与主事件循环线程并行执行。
*   `[in] complete`：当
    异步逻辑已完成或被取消。给定的函数称为
    从主事件循环线程。[`napi_async_complete_callback`][napi_async_complete_callback]提供
    更多详情。
*   `[in] data`：用户提供的数据上下文。这将被传回
    执行并完成功能。
*   `[out] result`:`napi_async_work*`这是新创建的句柄
    异步工作。

返回`napi_ok`如果 API 成功。

此 API 分配一个用于异步执行逻辑的工作对象。
它应该使用[`napi_delete_async_work`][napi_delete_async_work]一旦工作不再
必填。

`async_resource_name`应为空终止的 UTF-8 编码字符串。

这`async_resource_name`标识符由用户提供，应为
代表正在执行的异步工作的类型。也推荐
将命名空间应用于标识符，例如，通过包含模块名称。看
这[`async_hooks`文档][async_hooks `type`]了解更多信息。

### `napi_delete_async_work`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_delete_async_work(napi_env env,
                                   napi_async_work work);
```

*   `[in] env`：调用 API 的环境。
*   `[in] work`：调用返回的句柄`napi_create_async_work`.

返回`napi_ok`如果 API 成功。

此 API 释放以前分配的工作对象。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

### `napi_queue_async_work`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_queue_async_work(napi_env env,
                                  napi_async_work work);
```

*   `[in] env`：调用 API 的环境。
*   `[in] work`：调用返回的句柄`napi_create_async_work`.

返回`napi_ok`如果 API 成功。

此 API 请求计划以前分配的工作
用于执行。一旦成功返回，则不得再次调用此 API
与相同`napi_async_work`项或结果将未定义。

### `napi_cancel_async_work`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_cancel_async_work(napi_env env,
                                   napi_async_work work);
```

*   `[in] env`：调用 API 的环境。
*   `[in] work`：调用返回的句柄`napi_create_async_work`.

返回`napi_ok`如果 API 成功。

此 API 会取消排队的工作（如果尚未排队）
已启动。如果它已经开始执行，则不能
已取消和`napi_generic_failure`将被退回。如果成功，
这`complete`调用回调时的状态值为
`napi_cancelled`.该作品不应在`complete`
回调调用，即使它已成功取消。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

## 自定义异步操作

上面的简单异步工作 API 可能不适合每个
场景。使用任何其他异步机制时，以下 API
是确保异步操作由 正确跟踪所必需的
运行时。

### `napi_async_init`

<!-- YAML
added: v8.6.0
napiVersion: 1
-->

```c
napi_status napi_async_init(napi_env env,
                            napi_value async_resource,
                            napi_value async_resource_name,
                            napi_async_context* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] async_resource`：与异步工作关联的对象
    这将传递到可能`async_hooks` [`init`钩][init hooks]并且可以是
    访问者[`async_hooks.executionAsyncResource()`][async_hooks.executionAsyncResource()].
*   `[in] async_resource_name`：正在使用的资源类型的标识符
    提供了 由`async_hooks`应用程序接口。
*   `[out] result`：初始化的异步上下文。

返回`napi_ok`如果 API 成功。

这`async_resource`对象需要保持活动状态，直到
[`napi_async_destroy`][napi_async_destroy]保留`async_hooks`相关的 API 运行正常。在
为了保持ABI与以前版本的兼容性，`napi_async_context`s
没有保持对`async_resource`对象
避免引入导致内存泄漏。但是，如果`async_resource`是
在 JavaScript 引擎之前收集的垃圾`napi_async_context`是
销毁者`napi_async_destroy`叫`napi_async_context`相关接口
喜欢[`napi_open_callback_scope`][napi_open_callback_scope]和[`napi_make_callback`][napi_make_callback]可导致
使用 时异步上下文丢失等问题`AsyncLocalStorage`应用程序接口。

为了保持 ABI 与以前版本的兼容性，通过`NULL`
为`async_resource`不会导致错误。但是，这不是
建议使用，因为这会导致较差的结果 `async_hooks`
[`init`钩][init hooks]和`async_hooks.executionAsyncResource()`因为资源是
现在由底层用户要求`async_hooks`实施以提供
异步回调之间的链接。

### `napi_async_destroy`

<!-- YAML
added: v8.6.0
napiVersion: 1
-->

```c
napi_status napi_async_destroy(napi_env env,
                               napi_async_context async_context);
```

*   `[in] env`：调用 API 的环境。
*   `[in] async_context`：要销毁的异步上下文。

返回`napi_ok`如果 API 成功。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

### `napi_make_callback`

<!-- YAML
added: v8.0.0
napiVersion: 1
changes:
  - version: v8.6.0
    pr-url: https://github.com/nodejs/node/pull/15189
    description: Added `async_context` parameter.
-->

```c
NAPI_EXTERN napi_status napi_make_callback(napi_env env,
                                           napi_async_context async_context,
                                           napi_value recv,
                                           napi_value func,
                                           size_t argc,
                                           const napi_value* argv,
                                           napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] async_context`：异步操作的上下文，即
    调用回调。这通常应该是以前的值
    从以下位置获得[`napi_async_init`][napi_async_init].
    为了保持 ABI 与以前版本的兼容性，通过`NULL`
    为`async_context`不会导致错误。但是，这会导致
    在异步挂接的错误操作中。潜在问题包括
    使用`AsyncLocalStorage`应用程序接口。
*   `[in] recv`：`this`传递给被调用函数的值。
*   `[in] func`:`napi_value`表示要调用的 JavaScript 函数。
*   `[in] argc`：元素的计数`argv`数组。
*   `[in] argv`：JavaScript 值数组 as`napi_value`代表
    函数的参数。如果`argc`为零 此参数可以是
    通过传入省略`NULL`.
*   `[out] result`:`napi_value`表示返回的 JavaScript 对象。

返回`napi_ok`如果 API 成功。

此方法允许从本机调用 JavaScript 函数对象
附加组件。此 API 类似于`napi_call_function`.但是，它用于调用
*从*本机代码返回*到*JavaScript*后*从异步返回
操作（当堆栈上没有其他脚本时）。这是一个相当简单的
包装`node::MakeCallback`.

注意它是*不*必要使用`napi_make_callback`从
`napi_async_complete_callback`;在这种情况下，回调的异步
上下文已经设置完毕，因此直接调用`napi_call_function`
是充分和适当的。使用`napi_make_callback`功能
在实现不使用的自定义异步行为时可能需要
`napi_create_async_work`.

任何`process.nextTick`s 或 承诺在微任务队列上调度
在回调期间运行 JavaScript，然后再返回到 C/C++。

### `napi_open_callback_scope`

<!-- YAML
added: v9.6.0
napiVersion: 3
-->

```c
NAPI_EXTERN napi_status napi_open_callback_scope(napi_env env,
                                                 napi_value resource_object,
                                                 napi_async_context context,
                                                 napi_callback_scope* result)
```

*   `[in] env`：调用 API 的环境。
*   `[in] resource_object`：与异步工作关联的对象
    这将传递到可能`async_hooks` [`init`钩][init hooks].这
    参数已被弃用，并在运行时被忽略。使用
    `async_resource`参数[`napi_async_init`][napi_async_init]相反。
*   `[in] context`：调用回调的异步操作的上下文。
    这应该是以前从[`napi_async_init`][napi_async_init].
*   `[out] result`：新创建的作用域。

在某些情况下（例如，解决承诺）是
必须具有与回调关联的作用域的等效项
在进行某些 Node-API 调用时就位。如果 上没有其他脚本
堆栈[`napi_open_callback_scope`][napi_open_callback_scope]和
[`napi_close_callback_scope`][napi_close_callback_scope]函数可用于打开/关闭
所需的范围。

### `napi_close_callback_scope`

<!-- YAML
added: v9.6.0
napiVersion: 3
-->

```c
NAPI_EXTERN napi_status napi_close_callback_scope(napi_env env,
                                                  napi_callback_scope scope)
```

*   `[in] env`：调用 API 的环境。
*   `[in] scope`：要关闭的范围。

即使存在挂起的 JavaScript 异常，也可以调用此 API。

## 版本管理

### `napi_get_node_version`

<!-- YAML
added: v8.4.0
napiVersion: 1
-->

```c
typedef struct {
  uint32_t major;
  uint32_t minor;
  uint32_t patch;
  const char* release;
} napi_node_version;

napi_status napi_get_node_version(napi_env env,
                                  const napi_node_version** version);
```

*   `[in] env`：调用 API 的环境。
*   `[out] version`：指向 Node 本身的版本信息.js指针。

返回`napi_ok`如果 API 成功。

此函数填充`version`具有主要、次要和补丁的结构
当前正在运行的 Node.js 的版本，以及`release`字段与
的值[`process.release.name`][`process.release`].

返回的缓冲区是静态分配的，不需要释放。

### `napi_get_version`

<!-- YAML
added: v8.0.0
napiVersion: 1
-->

```c
napi_status napi_get_version(napi_env env,
                             uint32_t* result);
```

*   `[in] env`：调用 API 的环境。
*   `[out] result`：支持的最高版本的 Node-API。

返回`napi_ok`如果 API 成功。

此 API 返回
节点.js运行时。Node-API计划是累加的，使得
Node.js的较新版本可能支持其他 API 函数。
为了允许插件在运行时使用较新的函数
支持它的 Node.js 版本，同时提供
与节点一起运行时的回退行为.js不与节点无关的版本
支持它：

*   叫`napi_get_version()`以确定 API 是否可用。
*   如果可用，则使用`uv_dlsym()`.
*   使用动态加载的指针调用函数。
*   如果该函数不可用，请提供备用实现
    不使用该函数。

## 内存管理

### `napi_adjust_external_memory`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_adjust_external_memory(napi_env env,
                                                    int64_t change_in_bytes,
                                                    int64_t* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] change_in_bytes`：保留的外部分配内存中的更改
    由 JavaScript 对象激活。
*   `[out] result`：调整后的值

返回`napi_ok`如果 API 成功。

此函数为 V8 提供外部分配量的指示
由 JavaScript 对象（即 JavaScript 对象）保持活动的内存
指向由本机插件分配的自身内存）。注册
外部分配的内存将触发全局垃圾回收 更多
通常比否则。

## 承诺

Node-API 提供了用于创建的工具`Promise`对象，如 中所述
[第25.4节][Section 25.4]的 ECMA 规范。它将承诺实现为一对
对象。当承诺由以下人员创建时`napi_create_promise()`，“延迟”
对象与`Promise`.延迟的对象是
绑定到创建的`Promise`并且是解决或拒绝
`Promise`用`napi_resolve_deferred()`或`napi_reject_deferred()`.这
由 创建的延迟对象`napi_create_promise()`由 释放
`napi_resolve_deferred()`或`napi_reject_deferred()`.这`Promise`对象可能
返回到 JavaScript，在那里它可以以通常的方式使用。

例如，要创建一个 promise 并将其传递给异步工作线程，

```c
napi_deferred deferred;
napi_value promise;
napi_status status;

// Create the promise.
status = napi_create_promise(env, &deferred, &promise);
if (status != napi_ok) return NULL;

// Pass the deferred to a function that performs an asynchronous action.
do_something_asynchronous(deferred);

// Return the promise to JS
return promise;
```

以上功能`do_something_asynchronous()`将执行其异步
行动，然后它将解决或拒绝推迟，从而得出结论
承诺并释放推迟的：

```c
napi_deferred deferred;
napi_value undefined;
napi_status status;

// Create a value with which to conclude the deferred.
status = napi_get_undefined(env, &undefined);
if (status != napi_ok) return NULL;

// Resolve or reject the promise associated with the deferred depending on
// whether the asynchronous action succeeded.
if (asynchronous_action_succeeded) {
  status = napi_resolve_deferred(env, deferred, undefined);
} else {
  status = napi_reject_deferred(env, deferred, undefined);
}
if (status != napi_ok) return NULL;

// At this point the deferred has been freed, so we should assign NULL to it.
deferred = NULL;
```

### `napi_create_promise`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
napi_status napi_create_promise(napi_env env,
                                napi_deferred* deferred,
                                napi_value* promise);
```

*   `[in] env`：调用 API 的环境。
*   `[out] deferred`：新创建的延迟对象，以后可以传递给
    `napi_resolve_deferred()`或`napi_reject_deferred()`以解决拒绝问题
    相关的承诺。
*   `[out] promise`：与延迟对象关联的 JavaScript 承诺。

返回`napi_ok`如果 API 成功。

此 API 创建一个延迟对象和一个 JavaScript 承诺。

### `napi_resolve_deferred`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
napi_status napi_resolve_deferred(napi_env env,
                                  napi_deferred deferred,
                                  napi_value resolution);
```

*   `[in] env`：调用 API 的环境。
*   `[in] deferred`：要解析其关联承诺的延迟对象。
*   `[in] resolution`：用于解析承诺的值。

此 API 通过延迟对象的方式解析 JavaScript 承诺
与之相关的内容。因此，它只能用于解析JavaScript。
相应的延迟对象可用的承诺。这
有效意味着承诺必须已使用
`napi_create_promise()`并且从该调用返回的延迟对象必须
已保留以传递给此 API。

成功完成后，将释放延迟的对象。

### `napi_reject_deferred`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
napi_status napi_reject_deferred(napi_env env,
                                 napi_deferred deferred,
                                 napi_value rejection);
```

*   `[in] env`：调用 API 的环境。
*   `[in] deferred`：要解析其关联承诺的延迟对象。
*   `[in] rejection`：拒绝承诺的价值。

此 API 通过延迟对象拒绝 JavaScript 承诺
与之相关的内容。因此，它只能用于拒绝JavaScript。
相应的延迟对象可用的承诺。这
有效意味着承诺必须已使用
`napi_create_promise()`并且从该调用返回的延迟对象必须
已保留以传递给此 API。

成功完成后，将释放延迟的对象。

### `napi_is_promise`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
napi_status napi_is_promise(napi_env env,
                            napi_value value,
                            bool* is_promise);
```

*   `[in] env`：调用 API 的环境。
*   `[in] value`：要检查的值
*   `[out] is_promise`：指示是否`promise`是原生承诺
    对象（即，由基础引擎创建的 promise 对象）。

## 脚本执行

Node-API 提供了一个 API，用于使用
底层 JavaScript 引擎。

### `napi_run_script`

<!-- YAML
added: v8.5.0
napiVersion: 1
-->

```c
NAPI_EXTERN napi_status napi_run_script(napi_env env,
                                        napi_value script,
                                        napi_value* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] script`：包含要执行的脚本的 JavaScript 字符串。
*   `[out] result`：执行脚本后生成的值。

此函数执行一串 JavaScript 代码，并返回其结果
以下注意事项：

*   与`eval`，此函数不允许脚本访问当前
    词法范围，因此也不允许访问
    [模块范围][module scope]，表示伪全局，例如`require`不会
    可用。
*   该脚本可以访问[全局范围][global scope].功能和`var`声明
    在脚本中将添加到[`global`][global]对象。变量声明
    使用`let`和`const`将全局可见，但不会添加
    到[`global`][global]对象。
*   的价值`this`是[`global`][global]在脚本中。

## libuv 事件循环

Node-API 提供了一个函数，用于获取与
一个特定的`napi_env`.

### `napi_get_uv_event_loop`

<!-- YAML
added:
  - v9.3.0
  - v8.10.0
napiVersion: 2
-->

```c
NAPI_EXTERN napi_status napi_get_uv_event_loop(napi_env env,
                                               struct uv_loop_s** loop);
```

*   `[in] env`：调用 API 的环境。
*   `[out] loop`：当前 libuv 循环实例。

## 异步线程安全函数调用

JavaScript 函数通常只能从本机插件的主函数调用
线。如果插件创建其他线程，则 Node-API 会将
需要一个`napi_env`,`napi_value`或`napi_ref`不得从那些
线程。

当插件有额外的线程并且需要调用JavaScript函数时
根据这些线程完成的处理，这些线程必须
与插件的主线程通信，以便主线程可以调用
JavaScript 代表他们运行函数。线程安全函数 API 提供了一个
简单的方法来做到这一点。

这些 API 提供的类型`napi_threadsafe_function`以及
创建、销毁和调用此类型的对象。
`napi_create_threadsafe_function()`创建对
`napi_value`它包含一个JavaScript函数，可以从多个调用
线程。调用以异步方式发生。这意味着，具有
JavaScript回调将被调用将被放置在队列中，并且，对于每个
值，最终将调用 JavaScript 函数。

在创建`napi_threadsafe_function`一个`napi_finalize`回调可以是
提供。当线程安全时，将在主线程上调用此回调
函数即将被破坏。它接收上下文并完成数据
在施工期间提供，并提供在
线程，例如通过调用`uv_thread_join()`.**除了主循环线程，
在完成回调后，任何线程都不应使用线程安全函数
完成。**

这`context`在呼叫期间给出`napi_create_threadsafe_function()`能
从任何调用
`napi_get_threadsafe_function_context()`.

### 调用线程安全函数

`napi_call_threadsafe_function()`可用于发起调用
JavaScript.`napi_call_threadsafe_function()`接受控制
API 的行为是否为阻塞。如果设置为`napi_tsfn_nonblocking`，则 API
行为非阻塞，返回`napi_queue_full`如果队列已满，
阻止数据成功添加到队列中。如果设置为
`napi_tsfn_blocking`，则 API 将阻塞，直到队列中出现可用空间。
`napi_call_threadsafe_function()`如果线程安全函数是
创建的最大队列大小为 0。

`napi_call_threadsafe_function()`不应调用`napi_tsfn_blocking`
从 JavaScript 线程，因为，如果队列已满，可能会导致
JavaScript 线程到死锁。

对 JavaScript 的实际调用由通过
`call_js_cb`参数。`call_js_cb`在主线程上为每个调用一次
通过成功调用 到队列中的值
`napi_call_threadsafe_function()`.如果未给出此类回调，则默认
将使用回调，并且生成的 JavaScript 调用将没有参数。
这`call_js_cb`回调接收要调用的 JavaScript 函数作为
`napi_value`在其参数中，以及`void*`上下文指针在以下情况下使用
创建`napi_threadsafe_function`，以及下一个数据指针
由其中一个辅助线程创建。然后，回调可以使用这样的 API
如`napi_call_function()`以调用 JavaScript。

回调也可以通过以下方式调用：`env`和`call_js_cb`两者都设置为`NULL`
以指示不再可能调用 JavaScript，而项目
保留在可能需要释放的队列中。这通常发生在
节点.js进程在线程安全函数仍处于活动状态时退出。

没有必要通过以下方式调用 JavaScript`napi_make_callback()`因为
节点 API 运行`call_js_cb`在适合于回调的上下文中。

### 线程安全函数的引用计数

可以将线程添加到和从中删除`napi_threadsafe_function`对象
在其存在期间。因此，除了指定初始数量
创建时的线程，`napi_acquire_threadsafe_function`可以调用到
指示新线程将开始使用线程安全函数。
同样地`napi_release_threadsafe_function`可以调用以指示
现有线程将停止使用线程安全函数。

`napi_threadsafe_function`对象在使用
对象已调用`napi_release_threadsafe_function()`或已收到
返回状态`napi_closing`响应电话
`napi_call_threadsafe_function`.队列在
`napi_threadsafe_function`被摧毁。`napi_release_threadsafe_function()`
应该是与给定的 API 一起进行的最后一次 API 调用
`napi_threadsafe_function`，因为呼叫完成后，没有
保证`napi_threadsafe_function`仍已分配。对于相同
原因，不使用线程安全函数
收到返回值后`napi_closing`响应电话
`napi_call_threadsafe_function`.与
`napi_threadsafe_function`可以在其`napi_finalize`回调哪个
已传递到`napi_create_threadsafe_function()`.参数
`initial_thread_count`之`napi_create_threadsafe_function`标记初始
线程安全函数的获取次数，而不是调用
`napi_acquire_threadsafe_function`创建时多次。

一旦线程数使用`napi_threadsafe_function`达到
零，没有进一步的线程可以通过调用来开始使用它
`napi_acquire_threadsafe_function()`.事实上，所有后续的 API 调用
与之相关，除了`napi_release_threadsafe_function()`，将返回一个
错误值`napi_closing`.

线程安全函数可以通过提供值来“中止”`napi_tsfn_abort`
自`napi_release_threadsafe_function()`.这将导致所有后续 API
与线程安全函数相关联，除了
`napi_release_threadsafe_function()`返回`napi_closing`甚至在它之前
引用计数达到零。特别`napi_call_threadsafe_function()`
会再来`napi_closing`，从而通知线程它不再是
可以对线程安全函数进行异步调用。这可以是
用作终止线程的条件。**收到返回值时
之`napi_closing`从`napi_call_threadsafe_function()`线程不得使用
线程安全函数不再存在，因为它不再保证
被分配。**

### 决定是否保持流程运行

与 libuv 句柄类似，线程安全函数可以被“引用”和
“未引用”。“引用的”线程安全函数将导致事件循环打开
在其上创建它以保持活动状态的线程，直到线程安全函数
被摧毁。相反，“未引用”的线程安全函数不会
阻止事件循环退出。接口`napi_ref_threadsafe_function`和
`napi_unref_threadsafe_function`为此目的而存在。

也没有`napi_unref_threadsafe_function`将线程安全函数标记为
能够被摧毁，也不能被摧毁`napi_ref_threadsafe_function`防止它从
被摧毁。

### `napi_create_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
changes:
  - version:
     - v12.6.0
     - v10.17.0
    pr-url: https://github.com/nodejs/node/pull/27791
    description: Made `func` parameter optional with custom `call_js_cb`.
-->

```c
NAPI_EXTERN napi_status
napi_create_threadsafe_function(napi_env env,
                                napi_value func,
                                napi_value async_resource,
                                napi_value async_resource_name,
                                size_t max_queue_size,
                                size_t initial_thread_count,
                                void* thread_finalize_data,
                                napi_finalize thread_finalize_cb,
                                void* context,
                                napi_threadsafe_function_call_js call_js_cb,
                                napi_threadsafe_function* result);
```

*   `[in] env`：调用 API 的环境。
*   `[in] func`：从另一个线程调用的可选 JavaScript 函数。它
    在以下情况下必须提供`NULL`传递给`call_js_cb`.
*   `[in] async_resource`：与异步工作关联的可选对象
    将传递到可能`async_hooks` [`init`钩][init hooks].
*   `[in] async_resource_name`：为其提供标识符的 JavaScript 字符串
    为公开的诊断信息提供的资源类型
    由`async_hooks`应用程序接口。
*   `[in] max_queue_size`：队列的最大大小。`0`没有限制。
*   `[in] initial_thread_count`：初始收购次数，即
    初始线程数，包括将要使用的主线程
    的此功能。
*   `[in] thread_finalize_data`：要传递到的可选数据`thread_finalize_cb`.
*   `[in] thread_finalize_cb`：当
    `napi_threadsafe_function`正在被摧毁。
*   `[in] context`：要附加到结果的可选数据
    `napi_threadsafe_function`.
*   `[in] call_js_cb`：可选的回调，用于调用 JavaScript 函数
    对不同线程上的调用的响应。此回调将在
    主线程。如果未给出，则将调用 JavaScript 函数，而不带
    参数和`undefined`作为其`this`价值。
    [`napi_threadsafe_function_call_js`][napi_threadsafe_function_call_js]提供了更多详细信息。
*   `[out] result`：异步线程安全的 JavaScript 函数。

### `napi_get_threadsafe_function_context`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

```c
NAPI_EXTERN napi_status
napi_get_threadsafe_function_context(napi_threadsafe_function func,
                                     void** result);
```

*   `[in] func`：用于检索其上下文的线程安全函数。
*   `[out] result`：存储上下文的位置。

此 API 可以从任何使用`func`.

### `napi_call_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
changes:
  - version: v14.5.0
    pr-url: https://github.com/nodejs/node/pull/33453
    description: Support for `napi_would_deadlock` has been reverted.
  - version: v14.1.0
    pr-url: https://github.com/nodejs/node/pull/32689
    description: Return `napi_would_deadlock` when called with
                 `napi_tsfn_blocking` from the main thread or a worker thread
                 and the queue is full.
-->

```c
NAPI_EXTERN napi_status
napi_call_threadsafe_function(napi_threadsafe_function func,
                              void* data,
                              napi_threadsafe_function_call_mode is_blocking);
```

*   `[in] func`：要调用的异步线程安全 JavaScript 函数。
*   `[in] data`：通过回调发送到 JavaScript 中的数据`call_js_cb`
    在创建线程安全的 JavaScript 函数期间提供。
*   `[in] is_blocking`：其值可以是`napi_tsfn_blocking`自
    指示如果队列已满，则呼叫应阻塞，或者
    `napi_tsfn_nonblocking`以指示呼叫应立即返回
    状态为`napi_queue_full`每当队列已满时。

不应使用调用此 API`napi_tsfn_blocking`从 JavaScript
线程，因为，如果队列已满，可能会导致 JavaScript 线程
僵局。

此 API 将返回`napi_closing`如果`napi_release_threadsafe_function()`是
调用`abort`设置为`napi_tsfn_abort`从任何线程。该值仅
如果 API 返回，则添加到队列中`napi_ok`.

此 API 可以从任何使用`func`.

### `napi_acquire_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

```c
NAPI_EXTERN napi_status
napi_acquire_threadsafe_function(napi_threadsafe_function func);
```

*   `[in] func`：异步线程安全的JavaScript函数开始制作
    使用。

线程应在通过之前调用此 API`func`到任何其他线程安全
函数 API，以指示它将使用`func`.这可以防止
`func`当所有其他线程都停止使用
它。

此 API 可以从任何将开始使用`func`.

### `napi_release_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

```c
NAPI_EXTERN napi_status
napi_release_threadsafe_function(napi_threadsafe_function func,
                                 napi_threadsafe_function_release_mode mode);
```

*   `[in] func`：其引用的异步线程安全 JavaScript 函数
    计数递减。
*   `[in] mode`：其值可以是`napi_tsfn_release`以指示
    当前线程将不再对线程安全进行进一步的调用
    函数，或`napi_tsfn_abort`以指示除当前
    线程，其他任何线程都不应对线程安全进行任何进一步的调用
    功能。如果设置为`napi_tsfn_abort`，进一步调用
    `napi_call_threadsafe_function()`会再来`napi_closing`，并且没有进一步
    值将被放置在队列中。

线程应在停止使用时调用此 API`func`.通过`func`
调用此 API 后的任何线程安全 API 都有未定义的结果，如
`func`可能已被销毁。

可以从任何将停止使用`func`.

### `napi_ref_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

```c
NAPI_EXTERN napi_status
napi_ref_threadsafe_function(napi_env env, napi_threadsafe_function func);
```

*   `[in] env`：调用 API 的环境。
*   `[in] func`：要引用的线程安全函数。

此 API 用于指示在主线程上运行的事件循环
不应退出，直到`func`已销毁。似[`uv_ref`][uv_ref]是的
也是幂等的。

也没有`napi_unref_threadsafe_function`将线程安全函数标记为
能够被摧毁，也不能被摧毁`napi_ref_threadsafe_function`防止它从
被摧毁。`napi_acquire_threadsafe_function`和
`napi_release_threadsafe_function`可用于此目的。

此 API 只能从主线程调用。

### `napi_unref_threadsafe_function`

<!-- YAML
added: v10.6.0
napiVersion: 4
-->

```c
NAPI_EXTERN napi_status
napi_unref_threadsafe_function(napi_env env, napi_threadsafe_function func);
```

*   `[in] env`：调用 API 的环境。
*   `[in] func`：用于取消引用的线程安全函数。

此 API 用于指示在主线程上运行的事件循环
可能在之前退出`func`被摧毁。似[`uv_unref`][uv_unref]它也是
幂等。

此 API 只能从主线程调用。

## 其他实用程序

### `node_api_get_module_file_name`

<!-- YAML
added:
  - v15.9.0
  - v14.18.0
  - v12.22.0
-->

> 稳定性： 1 - 实验

```c
NAPI_EXTERN napi_status
node_api_get_module_file_name(napi_env env, const char** result);

```

*   `[in] env`：调用 API 的环境。
*   `[out] result`：包含的绝对路径的网址
    加载加载项的位置。对于本地文件
    它将从文件系统开始`file://`.该字符串以空值终止，并且
    拥有者`env`因此不得修改或释放。

`result`如果加载项加载过程无法建立，则可能是空字符串
加载期间加载项的文件名。

[ABI Stability]: https://nodejs.org/en/docs/guides/abi-stability/

[AppVeyor]: https://www.appveyor.com

[C++ Addons]: addons.md

[CMake]: https://cmake.org

[CMake.js]: https://github.com/cmake-js/cmake-js

[ECMAScript Language Specification]: https://tc39.github.io/ecma262/

[Error handling]: #error-handling

[GCC]: https://gcc.gnu.org

[GYP]: https://gyp.gsrc.io

[GitHub releases]: https://help.github.com/en/github/administering-a-repository/about-releases

[LLVM]: https://llvm.org

[Native Abstractions for Node.js]: https://github.com/nodejs/nan

[Object lifetime management]: #object-lifetime-management

[Object wrap]: #object-wrap

[Section 12.10.4]: https://tc39.github.io/ecma262/#sec-instanceofoperator

[Section 12.5.5]: https://tc39.github.io/ecma262/#sec-typeof-operator

[Section 19.2]: https://tc39.github.io/ecma262/#sec-function-objects

[Section 19.4]: https://tc39.github.io/ecma262/#sec-symbol-objects

[Section 20.3]: https://tc39.github.io/ecma262/#sec-date-objects

[Section 22.1]: https://tc39.github.io/ecma262/#sec-array-objects

[Section 22.1.4.1]: https://tc39.github.io/ecma262/#sec-properties-of-array-instances-length

[Section 22.2]: https://tc39.github.io/ecma262/#sec-typedarray-objects

[Section 24.1]: https://tc39.github.io/ecma262/#sec-arraybuffer-objects

[Section 24.1.1.2]: https://tc39.es/ecma262/#sec-isdetachedbuffer

[Section 24.1.1.3]: https://tc39.es/ecma262/#sec-detacharraybuffer

[Section 24.3]: https://tc39.github.io/ecma262/#sec-dataview-objects

[Section 25.4]: https://tc39.github.io/ecma262/#sec-promise-objects

[Section 6]: https://tc39.github.io/ecma262/#sec-ecmascript-data-types-and-values

[Section 6.1]: https://tc39.github.io/ecma262/#sec-ecmascript-language-types

[Section 6.1.4]: https://tc39.github.io/ecma262/#sec-ecmascript-language-types-string-type

[Section 6.1.6]: https://tc39.github.io/ecma262/#sec-ecmascript-language-types-number-type

[Section 6.1.7]: https://tc39.github.io/ecma262/#sec-object-type

[Section 6.1.7.1]: https://tc39.github.io/ecma262/#table-2

[Section 7]: https://tc39.github.io/ecma262/#sec-abstract-operations

[Section 7.1.13]: https://tc39.github.io/ecma262/#sec-toobject

[Section 7.1.2]: https://tc39.github.io/ecma262/#sec-toboolean

[Section 7.1.3]: https://tc39.github.io/ecma262/#sec-tonumber

[Section 7.2.14]: https://tc39.github.io/ecma262/#sec-strict-equality-comparison

[Section 7.2.2]: https://tc39.github.io/ecma262/#sec-isarray

[Section 8.7]: https://tc39.es/ecma262/#sec-agents

[Section 9.1.6]: https://tc39.github.io/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots-defineownproperty-p-desc

[Travis CI]: https://travis-ci.org

[Visual Studio]: https://visualstudio.microsoft.com

[Working with JavaScript properties]: #working-with-javascript-properties

[Xcode]: https://developer.apple.com/xcode/

[`Number.MAX_SAFE_INTEGER`]: https://tc39.github.io/ecma262/#sec-number.max_safe_integer

[`Number.MIN_SAFE_INTEGER`]: https://tc39.github.io/ecma262/#sec-number.min_safe_integer

[`Worker`]: worker_threads.md#class-worker

[`async_hooks.executionAsyncResource()`]: async_hooks.md#async_hooksexecutionasyncresource

[`global`]: globals.md#global

[`init` hooks]: async_hooks.md#initasyncid-type-triggerasyncid-resource

[`napi_add_async_cleanup_hook`]: #napi_add_async_cleanup_hook

[`napi_add_env_cleanup_hook`]: #napi_add_env_cleanup_hook

[`napi_add_finalizer`]: #napi_add_finalizer

[`napi_async_cleanup_hook`]: #napi_async_cleanup_hook

[`napi_async_complete_callback`]: #napi_async_complete_callback

[`napi_async_destroy`]: #napi_async_destroy

[`napi_async_init`]: #napi_async_init

[`napi_callback`]: #napi_callback

[`napi_cancel_async_work`]: #napi_cancel_async_work

[`napi_close_callback_scope`]: #napi_close_callback_scope

[`napi_close_escapable_handle_scope`]: #napi_close_escapable_handle_scope

[`napi_close_handle_scope`]: #napi_close_handle_scope

[`napi_create_async_work`]: #napi_create_async_work

[`napi_create_error`]: #napi_create_error

[`napi_create_external_arraybuffer`]: #napi_create_external_arraybuffer

[`napi_create_range_error`]: #napi_create_range_error

[`napi_create_reference`]: #napi_create_reference

[`napi_create_type_error`]: #napi_create_type_error

[`napi_define_class`]: #napi_define_class

[`napi_delete_async_work`]: #napi_delete_async_work

[`napi_delete_reference`]: #napi_delete_reference

[`napi_escape_handle`]: #napi_escape_handle

[`napi_finalize`]: #napi_finalize

[`napi_get_and_clear_last_exception`]: #napi_get_and_clear_last_exception

[`napi_get_array_length`]: #napi_get_array_length

[`napi_get_element`]: #napi_get_element

[`napi_get_last_error_info`]: #napi_get_last_error_info

[`napi_get_property`]: #napi_get_property

[`napi_get_reference_value`]: #napi_get_reference_value

[`napi_get_value_external`]: #napi_get_value_external

[`napi_has_property`]: #napi_has_property

[`napi_instanceof`]: #napi_instanceof

[`napi_is_error`]: #napi_is_error

[`napi_is_exception_pending`]: #napi_is_exception_pending

[`napi_make_callback`]: #napi_make_callback

[`napi_open_callback_scope`]: #napi_open_callback_scope

[`napi_open_escapable_handle_scope`]: #napi_open_escapable_handle_scope

[`napi_open_handle_scope`]: #napi_open_handle_scope

[`napi_property_attributes`]: #napi_property_attributes

[`napi_property_descriptor`]: #napi_property_descriptor

[`napi_queue_async_work`]: #napi_queue_async_work

[`napi_reference_ref`]: #napi_reference_ref

[`napi_reference_unref`]: #napi_reference_unref

[`napi_remove_async_cleanup_hook`]: #napi_remove_async_cleanup_hook

[`napi_remove_env_cleanup_hook`]: #napi_remove_env_cleanup_hook

[`napi_set_instance_data`]: #napi_set_instance_data

[`napi_set_property`]: #napi_set_property

[`napi_threadsafe_function_call_js`]: #napi_threadsafe_function_call_js

[`napi_throw_error`]: #napi_throw_error

[`napi_throw_range_error`]: #napi_throw_range_error

[`napi_throw_type_error`]: #napi_throw_type_error

[`napi_throw`]: #napi_throw

[`napi_unwrap`]: #napi_unwrap

[`napi_wrap`]: #napi_wrap

[`node-addon-api`]: https://github.com/nodejs/node-addon-api

[`node_api.h`]: https://github.com/nodejs/node/blob/HEAD/src/node_api.h

[`node_api_create_syntax_error`]: #node_api_create_syntax_error

[`node_api_throw_syntax_error`]: #node_api_throw_syntax_error

[`process.release`]: process.md#processrelease

[`uv_ref`]: https://docs.libuv.org/en/v1.x/handle.html#c.uv_ref

[`uv_unref`]: https://docs.libuv.org/en/v1.x/handle.html#c.uv_unref

[async_hooks `type`]: async_hooks.md#type

[context-aware addons]: addons.md#context-aware-addons

[docs]: https://github.com/nodejs/node-addon-api#api-documentation

[global scope]: globals.md

[gyp-next]: https://github.com/nodejs/gyp-next

[module scope]: modules.md#the-module-scope

[node-gyp]: https://github.com/nodejs/node-gyp

[node-pre-gyp]: https://github.com/mapbox/node-pre-gyp

[prebuild]: https://github.com/prebuild/prebuild

[prebuildify]: https://github.com/prebuild/prebuildify

[worker threads]: https://nodejs.org/api/worker_threads.html
