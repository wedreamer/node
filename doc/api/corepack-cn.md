# Corepack

<!-- introduced_in=v14.19.0 -->

<!-- type=misc -->

<!-- YAML
added:
  - v16.9.0
  - v14.19.0
-->

> Stability: 1 - Experimental

_[Corepack][]_ 是一个实验性工具，可帮助管理包管理器的版本。它为每个 [supported package manager][] 公开二进制代理，当被调用时，它将识别为当前项目配置的任何包管理器，如果需要，透明地安装它，最后运行它而不需要明确的用户交互.

此功能简化了两个核心工作流程:

* 它简化了新贡献者的入职，因为他们不必再遵循特定于系统的安装过程，只需拥有您希望他们使用的包管理器.

* 它允许您确保团队中的每个人都将使用您希望他们使用的包管理器版本，而无需他们在每次需要进行更新时手动同步它.

## Workflows

### Enabling the feature

由于其实验状态，Corepack 目前需要显式启用才能生效。为此，请运行 [`corepack enable`][]，它将在您的环境中的 `node` 二进制文件旁边设置符号链接（并在必要时覆盖现有的符号链接）.

从现在开始，任何对 [supported binaries][] 的调用都无需进一步设置即可工作。如果您遇到问题，请运行 [`corepack disable`][] 从系统中删除代理（并考虑在 [Corepack 存储库][] 上打开问题让我们知道）.

### Configuring a package

Corepack 代理将在当前目录层次结构中找到最近的 [`package.json`][] 文件，以提取其 [`"packageManager"`][] 属性.

如果该值对应于 [supported package manager][]，Corepack 将确保对相关二进制文件的所有调用都针对请求的版本运行，如果需要则按需下载，如果无法成功检索则中止.

### Upgrading the global versions

在现有项目之外运行时（例如运行 `yarn init`），Corepack 默认使用与每个工具的最新稳定版本大致对应的预定义版本。这些版本可以通过运行 [`corepack prepare`][] 命令以及您希望设置的包管理器版本来覆盖:

```bash
corepack prepare yarn@x.y.z --activate
```

### Offline workflow

许多生产环境没有网络访问权限。由于 Corepack 通常直接从他们的注册表下载包管理器版本，
它可能与此类环境发生冲突。为避免这种情况发生，请在您仍有网络访问权限时调用 [`corepack prepare`][] 命令（通常在准备部署映像的同时）。这将确保即使没有网络访问，所需的包管理器也可用.

`prepare` 命令有[各种标志][]。查阅详细的 [Corepack 文档][] 了解更多信息.

## Supported package managers

以下二进制文件通过 Corepack 提供:

| Package manager | Binary names      |
| --------------- | ----------------- |
| [Yarn][]        | `yarn`, `yarnpkg` |
| [pnpm][]        | `pnpm`, `pnpx`    |

## Common questions

### How does Corepack interact with npm?

虽然 Corepack 可以像任何其他包管理器一样支持 npm，但默认情况下不启用它的 shim。这有一些后果:

* 总是可以在配置为与另一个包管理器一起使用的项目中运行“npm”命令，因为 Corepack 无法拦截它.

* 虽然 `npm` 是 [`"packageManager"`][] 属性中的有效选项，但缺少 shim 将导致使用全局 npm.

### Running `npm install -g yarn` doesn't work

npm 防止在进行全局安装时意外覆盖 Corepack 二进制文件。为避免此问题，请考虑以下选项之一:

* 不要运行此命令； Corepack 将提供包管理器二进制文件，并确保请求的版本始终可用，因此不需要显式安装包管理器.

* 将 `--force` 标志添加到 `npm install`；这将告诉 npm 可以覆盖二进制文件，但您将在此过程中删除 Corepack 文件。 （运行 [`corepack enable`][] 将它们添加回来。）

[Corepack]: https://github.com/nodejs/corepack
[Corepack documentation]: https://github.com/nodejs/corepack#readme
[Corepack repository]: https://github.com/nodejs/corepack
[Yarn]: https://yarnpkg.com
[`"packageManager"`]: packages.md#packagemanager
[`corepack disable`]: https://github.com/nodejs/corepack#corepack-disable--name
[`corepack enable`]: https://github.com/nodejs/corepack#corepack-enable--name
[`corepack prepare`]: https://github.com/nodejs/corepack#corepack-prepare--nameversion
[`package.json`]: packages.md#nodejs-packagejson-field-definitions
[pnpm]: https://pnpm.js.org
[supported binaries]: #supported-package-managers
[supported package manager]: #supported-package-managers
[various flags]: https://github.com/nodejs/corepack#utility-commands
