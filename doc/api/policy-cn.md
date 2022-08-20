# 政策

<!--introduced_in=v11.8.0-->

<!-- type=misc -->

> 稳定性： 1 - 实验

<!-- name=policy -->

Node.js包含对在加载代码时创建策略的实验性支持。

策略是一项安全功能，旨在允许保证
关于Node.js能够加载的代码。策略的使用假定
策略文件的安全做法，例如确保策略
文件不能被 Node.js 应用程序覆盖，使用
文件权限。

最佳做法是确保策略清单对于
正在运行的 Node.js应用程序，并且无法更改该文件
由正在运行的 Node 以任何方式.js应用程序。典型的设置是
将策略文件创建为与正在运行的 Node 不同的用户 ID.js
并向运行 Node.js 的用户 ID 授予读取权限。

## 使

<!-- type=misc -->

这`--experimental-policy`标志可用于启用策略的功能
加载模块时。

设置此项后，所有模块都必须符合策略清单文件
传递给旗帜：

```bash
node --experimental-policy=policy.json app.js
```

策略清单将用于对 加载的代码强制实施约束
节点.js。

要减少对磁盘上策略文件的篡改，请
策略文件本身可以通过以下方式提供`--policy-integrity`.
这允许运行`node`并断言策略文件内容
即使磁盘上的文件已更改。

```bash
node --experimental-policy=policy.json --policy-integrity="sha384-SggXRQHwCG8g+DktYYzxkXRIkTiEYWBHqev0xnpCxYlqMBufKZHAHQM3/boDaI/0" app.js
```

## 特征

### 错误行为

当策略检查失败时，默认情况下，Node.js 将引发错误。
可以将错误行为更改为几种可能性之一
通过在策略清单中定义“onerror”字段。以下值为
可用于更改行为：

*   `"exit"`：将立即退出进程。
    不允许运行任何清理代码。
*   `"log"`：将在故障发生时记录错误。
*   `"throw"`：将在故障现场抛出 JS 错误。这是
    违约。

```json
{
  "onerror": "log",
  "resources": {
    "./app/checked.js": {
      "integrity": "sha384-SggXRQHwCG8g+DktYYzxkXRIkTiEYWBHqev0xnpCxYlqMBufKZHAHQM3/boDaI/0"
    }
  }
}
```

### 完整性检查

策略文件必须将完整性检查与子资源完整性字符串结合使用
与浏览器兼容
[完整性属性](https://www.w3.org/TR/SRI/#the-integrity-attribute)
与绝对网址相关联。

使用时`require()`或`import`检查加载中涉及的所有资源
如果已指定策略清单，则为完整性。如果资源没有
与清单中列出的完整性匹配，将引发错误。

允许加载文件的示例策略文件`checked.js`:

```json
{
  "resources": {
    "./app/checked.js": {
      "integrity": "sha384-SggXRQHwCG8g+DktYYzxkXRIkTiEYWBHqev0xnpCxYlqMBufKZHAHQM3/boDaI/0"
    }
  }
}
```

策略清单中列出的每个资源可以是以下资源之一
格式以确定其位置：

1.  一个[相对网址字符串][relative-URL string]从清单到资源，例如`./resource.js`,`../resource.js`或`/resource.js`.
2.  资源的完整 URL 字符串，例如`file:///resource.js`.

加载资源时，整个 URL 必须匹配，包括搜索参数
和哈希片段。`./a.js?b`尝试加载时不会使用
`./a.js`反之亦然。

要生成完整性字符串，请使用脚本，例如
`node -e 'process.stdout.write("sha256-");process.stdin.pipe(crypto.createHash("sha256").setEncoding("base64")).pipe(process.stdout)' < FILE`
可以使用。

完整性可以指定为布尔值`true`接受任何
对当地发展有用的资源主体。它不是
建议在生产中使用，因为它允许意外更改
要视为有效的资源。

### 依赖关系重定向

应用程序可能需要提供模块的修补版本或防止
模块，允许所有模块访问所有其他模块。重定向
可以通过拦截尝试来加载希望加载的模块来使用
取代。

```json
{
  "resources": {
    "./app/checked.js": {
      "dependencies": {
        "fs": true,
        "os": "./app/node_modules/alt-os",
        "http": { "import": true }
      }
    }
  }
}
```

依赖项由请求的说明符字符串键入，并具有值
的任一`true`,`null`，指向要解析的模块的字符串，
或条件对象。

说明符字符串不执行任何搜索，并且必须与内容完全匹配
提供给`require()`或`import`规范化步骤除外。
因此，如果策略使用多个说明符，则可能需要多个说明符
指向同一模块的不同字符串（例如排除扩展名）。

说明符字符串在用于
匹配，以便与导入地图有一定的兼容性，例如，如果
资源`file:///C:/app/server.js`从
保单位于`file:///C:/app/policy.json`:

```json
{
  "resources": {
    "file:///C:/app/utils.js": {
      "dependencies": {
        "./utils.js": "./utils-v2.js"
      }
    }
  }
}
```

用于加载的任何说明符`file:///C:/app/utils.js`然后会被拦截
并重定向至`file:///C:/app/utils-v2.js`而是不考虑使用
绝对或相对说明符。但是，如果说明符不是绝对值
或者使用相对URL字符串，它不会被拦截。因此，如果导入
如`import('#utils')`被使用，它不会被拦截。

如果重定向的值为`true`，顶部的“依赖关系”字段
将使用策略文件。如果策略文件顶部的该字段是
`true`默认节点搜索算法用于查找模块。

如果重定向的值是字符串，则相对于
清单，然后立即使用而无需搜索。

尝试解决且未在 中列出的任何说明符字符串
依赖项根据策略导致错误。

重定向不会阻止通过直接访问等方式访问 API
自`require.cache`或通过`module.constructor`允许访问
加载模块。策略重定向仅影响说明符`require()`和
`import`.其他方法，例如通过以下方式防止对 API 的意外访问
变量是锁定加载模块路径所必需的。

布尔值`true`的依赖关系映射可以指定为允许
模块来加载任何说明符而不进行重定向。这对于本地很有用
开发，并且在生产中可能有一些有效的用法，但应该使用
只有在审核模块以确保其行为有效后才能小心。

似`"exports"`在`package.json`，也可以将依赖项指定为
是包含条件的对象，这些条件将依赖项的加载方式分支。在
前面的例子，`"http"`当`"import"`条件是
加载它的一部分。

值`null`，则解析的值会导致解析失败。这
可用于确保显式阻止某些类型的动态访问。

已解析的模块位置的未知值会导致故障，但
不保证向前兼容。

#### 示例：修补的依赖项

重定向的依赖项可以提供适合的衰减或修改功能
应用程序。例如，通过以下方式记录有关函数持续时间计时的数据
包装原件：

```js
const original = require('fn');
module.exports = function fn(...args) {
  console.time();
  try {
    return new.target ?
      Reflect.construct(original, args) :
      Reflect.apply(original, this, args);
  } finally {
    console.timeEnd();
  }
};
```

### 范围

使用`"scopes"`用于为许多资源设置配置的清单字段
立即。这`"scopes"`字段的工作原理是按其段匹配资源。
如果范围或资源包括`"cascade": true`，未知说明符将
在其包含的范围内进行搜索。级联的包含范围
通过递归地减少资源 URL 来查找，方法是删除
[特殊方案][special schemes]，保持尾随`"/"`后缀，并删除查询和
哈希片段。这会导致 URL 最终减少到其源。
如果 URL 不是特殊的，则作用域将按 URL 的源进行定位。如果不是
范围是为原点找到的，或者在不透明原点的情况下，协议
字符串可用作作用域。如果未找到 URL 协议的作用域，则
最后一个空字符串`""`将使用范围。

注意`blob:`URL采用它们所包含的路径的起源，因此范围
之`"blob:https://nodejs.org"`将不起作用，因为没有URL可以具有
起源`blob:https://nodejs.org`;以 开头的网址
`blob:https://nodejs.org/`将使用`https://nodejs.org`因为它的起源和
因此`https:`作为其协议范围。对于不透明原点`blob:`他们将的网址
有`blob:`因为它们的协议范围，因为它们不采用起源。

#### 例

```json
{
  "scopes": {
    "file:///C:/app/": {},
    "file:": {},
    "": {}
  }
}
```

给定一个位于`file:///C:/app/bin/main.js`，则以下作用域将
按顺序检查：

1.  `"file:///C:/app/bin/"`

这决定了以下所有基于文件的资源的策略
`"file:///C:/app/bin/"`.这不在`"scopes"`政策和领域
将被跳过。将此作用域添加到策略将导致它被使用
在`"file:///C:/app/"`范围。

2.  `"file:///C:/app/"`

这决定了以下所有基于文件的资源的策略
`"file:///C:/app/"`.这是在`"scopes"`政策领域，它将
确定资源的策略`file:///C:/app/bin/main.js`.如果
范围有`"cascade": true`，则有关资源的任何未满足查询都将
委托到下一个相关范围`file:///C:/app/bin/main.js`,`"file:"`.

3.  `"file:///C:/"`

这决定了以下所有基于文件的资源的策略`"file:///C:/"`.
这不在`"scopes"`字段，并将被跳过。它会
不用于`file:///C:/app/bin/main.js`除非`"file:///"`设置为
级联或不在`"scopes"`的策略。

4.  `"file:///"`

这将确定上所有基于文件的资源的策略`localhost`.这
不在`"scopes"`字段，并将被跳过。它不会
用于`file:///C:/app/bin/main.js`除非`"file:///"`设置为级联
或不在`"scopes"`的策略。

5.  `"file:"`

这将确定所有基于文件的资源的策略。它不会被使用
为`file:///C:/app/bin/main.js`除非`"file:///"`设置为级联或不级联
在`"scopes"`的策略。

6.  `""`

这将确定所有资源的策略。它不会用于
`file:///C:/app/bin/main.js`除非`"file:"`设置为级联。

#### 使用作用域的完整性

将完整性设置为`true`在作用域上将为任何
在清单中找不到资源`true`.

将完整性设置为`null`在作用域上将为任何
在清单中找不到资源以失败匹配。

不包括完整性与将完整性设置为`null`.

`"cascade"`完整性检查将被忽略，如果`"integrity"`是显式的
设置。

以下示例允许加载任何文件：

```json
{
  "scopes": {
    "file:": {
      "integrity": true
    }
  }
}
```

#### 使用作用域的依赖关系重定向

以下示例将允许访问`fs`对于内的所有资源
`./app/`:

```json
{
  "resources": {
    "./app/checked.js": {
      "cascade": true,
      "integrity": true
    }
  },
  "scopes": {
    "./app/": {
      "dependencies": {
        "fs": true
      }
    }
  }
}
```

以下示例将允许访问`fs`面向所有人`data:`资源：

```json
{
  "resources": {
    "data:text/javascript,import('node:fs');": {
      "cascade": true,
      "integrity": true
    }
  },
  "scopes": {
    "data:": {
      "dependencies": {
        "fs": true
      }
    }
  }
}
```

#### 例：[导入地图][import maps]仿真

给定导入映射：

```json
{
  "imports": {
    "react": "./app/node_modules/react/index.js"
  },
  "scopes": {
    "./ssr/": {
      "react": "./app/node_modules/server-side-react/index.js"
    }
  }
}
```

```json
{
  "dependencies": true,
  "scopes": {
    "": {
      "cascade": true,
      "dependencies": {
        "react": "./app/node_modules/react/index.js"
      }
    },
    "./ssr/": {
      "cascade": true,
      "dependencies": {
        "react": "./app/node_modules/server-side-react/index.js"
      }
    }
  }
}
```

导入映射假定默认情况下可以获取任何资源。这意味着
`"dependencies"`在策略的顶层应设置为`true`.
策略要求此选项处于选择加入状态，因为它启用了
应用程序交叉链接，这在很多情况下都没有意义。他们还
假设任何给定的作用域都可以访问其允许的依赖项之上的任何作用域;
必须设置模拟导入映射的所有作用域`"cascade": true`.

导入地图的“导入”只有一个顶级范围。所以对于
仿真`"imports"`使用`""`范围。用于模拟`"scopes"`使用
`"scopes"`以类似的方式`"scopes"`适用于导入地图。

注意事项：策略不使用字符串匹配来查找各种范围。他们
做 URL 遍历。这意味着这样的事情`blob:`和`data:`网址可能不是
两个系统之间完全可互操作。例如，导入地图可以
部分匹配`data:`或`blob:`通过将 URL 分区到`/`
性格，政策故意不能。为`blob:`URL 导入映射范围
不采用原产地`blob:`网址。

此外，导入地图仅适用于`import`因此，可能需要添加一个
`"import"`条件到所有依赖项映射。

[import maps]: https://url.spec.whatwg.org/#relative-url-with-fragment-string

[relative-url string]: https://url.spec.whatwg.org/#relative-url-with-fragment-string

[special schemes]: https://url.spec.whatwg.org/#special-scheme
