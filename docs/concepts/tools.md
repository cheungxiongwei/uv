# 工具

工具是提供命令行接口的 Python 包。

!!! note

    有关如何使用工具接口的介绍，请参阅[工具指南](../guides/tools.md) —— 本文档讨论工具管理的细节。

## `uv tool` 接口

uv 提供了一个专用的接口来与工具进行交互。可以使用 `uv tool run` 在不安装的情况下运行工具，此时它们的依赖项将被安装在一个临时的虚拟环境中，与当前项目隔离。

由于在不安装的情况下运行工具非常常见，因此提供了 `uvx` 作为 `uv tool run` 的别名 —— 这两个命令完全等效。为简洁起见，文档将主要使用 `uvx` 而不是 `uv tool run`。

工具也可以使用 `uv tool install` 安装，此时它们的可执行文件将[在 `PATH` 上可用](#the-path) —— 仍然使用隔离的虚拟环境，但命令完成后不会删除该环境。

## 执行与安装

在大多数情况下，使用 `uvx` 执行工具比安装工具更合适。安装工具适用于需要将工具提供给系统上的其他程序使用的情况，例如，如果某些不受控制的脚本需要该工具，或者您处于 Docker 镜像中并希望将工具提供给用户使用。

## 工具环境

使用 `uvx` 运行工具时，虚拟环境存储在 uv 缓存目录中，并被视为可丢弃的，即如果您运行 `uv cache clean`，该环境将被删除。缓存环境只是为了减少重复调用的开销。如果环境被删除，将自动创建一个新的环境。

使用 `uv tool install` 安装工具时，会在 uv 工具目录中创建一个虚拟环境。除非卸载工具，否则不会删除该环境。如果手动删除该环境，工具将无法运行。

## 工具版本

除非请求特定版本，否则 `uv tool install` 将安装请求工具的最新可用版本。`uvx` 将在第一次调用时使用请求工具的最新可用版本。之后，除非请求了不同的版本、缓存被清理或缓存被刷新，否则 `uvx` 将使用缓存的工具版本。

例如，运行特定版本的 Ruff：

```console
$ uvx ruff@0.6.0 --version
ruff 0.6.0
```

后续调用 `uvx` 将使用最新版本，而不是缓存的版本。

```console
$ uvx ruff --version
ruff 0.6.2
```

但是，如果发布了新版本的 Ruff，除非刷新缓存，否则不会使用它。

要请求 Ruff 的最新版本并刷新缓存，请使用 `@latest` 后缀：

```console
$ uvx ruff@latest --version
0.6.2
```

一旦使用 `uv tool install` 安装了工具，`uvx` 将默认使用已安装的版本。

例如，安装旧版本的 Ruff 后：

```console
$ uv tool install ruff==0.5.0
```

`ruff` 和 `uvx ruff` 的版本相同：

```console
$ ruff --version
ruff 0.5.0
$ uvx ruff --version
ruff 0.5.0
```

但是，您可以通过显式请求最新版本来忽略已安装的版本，例如：

```console
$ uvx ruff@latest --version
0.6.2
```

或者，使用 `--isolated` 标志，这将避免刷新缓存但忽略已安装的版本：

```console
$ uvx --isolated ruff --version
0.6.2
```

`uv tool install` 也会尊重 `{package}@{version}` 和 `{package}@latest` 的指定，例如：

```console
$ uv tool install ruff@latest
$ uv tool install ruff@0.6.0
```

### 工具目录

默认情况下，uv 工具目录名为 `tools`，位于 uv 应用程序状态目录中，例如 `~/.local/share/uv/tools`。可以使用 `UV_TOOL_DIR` 环境变量自定义该位置。

要显示工具安装目录的路径：

```console
$ uv tool dir
```

工具环境放置在以工具包名称命名的目录中，例如 `.../tools/<name>`。

### 修改工具环境

工具环境**不**应直接修改。强烈建议不要使用 `pip` 操作手动修改工具环境。

工具环境可以通过 `uv tool upgrade` 进行升级，或通过后续的 `uv tool install` 操作完全重新创建。

要升级工具环境中的所有包：

```console
$ uv tool upgrade black
```

要升级工具环境中的单个包：

```console
$ uv tool upgrade black --upgrade-package click
```

要重新安装工具环境中的所有包：

```console
$ uv tool upgrade black --reinstall
```

要重新安装工具环境中的单个包：

```console
$ uv tool upgrade black --reinstall-package click
```

工具升级将尊重安装工具时提供的版本约束。例如，`uv tool install black >=23,<24` 后跟 `uv tool upgrade black` 将把 Black 升级到 `>=23,<24` 范围内的最新版本。

要替换版本约束，请使用 `uv tool install` 重新安装工具：

```console
$ uv tool install black>=24
```

同样，工具升级将保留安装工具时提供的设置。例如，`uv tool install black --prerelease allow` 后跟 `uv tool upgrade black` 将保留 `--prerelease allow` 设置。

工具升级将重新安装工具可执行文件，即使它们没有更改。

### 包含额外依赖项

可以在工具执行期间包含额外的包：

```console
$ uvx --with <extra-package> <tool>
```

以及，在工具安装期间：

```console
$ uv tool install --with <extra-package> <tool-package>
```

`--with` 选项可以多次提供以包含额外的包。

`--with` 选项支持包规范，因此可以请求特定版本：

```console
$ uvx --with <extra-package>==<version> <tool-package>
```

如果请求的版本与工具包的要求冲突，包解析将失败，命令将出错。

## 工具可执行文件

工具可执行文件包括 Python 包提供的所有控制台入口点、脚本入口点和二进制脚本。工具可执行文件在 Unix 上符号链接到 `bin` 目录，在 Windows 上复制到 `bin` 目录。

### `bin` 目录

可执行文件安装到遵循 XDG 标准的用户 `bin` 目录中，例如 `~/.local/bin`。与 uv 中的其他目录方案不同，XDG 标准在**所有平台**上使用，包括 Windows 和 macOS —— 在这些平台上没有明确的替代位置来放置可执行文件。安装目录由第一个可用的环境变量确定：

- `$UV_TOOL_BIN_DIR`
- `$XDG_BIN_HOME`
- `$XDG_DATA_HOME/../bin`
- `$HOME/.local/bin`

工具包的依赖项提供的可执行文件不会被安装。

### `PATH`

`bin` 目录必须在 `PATH` 变量中，以便工具可执行文件可以从 shell 中使用。如果不在 `PATH` 中，将显示警告。可以使用 `uv tool update-shell` 命令将 `bin` 目录添加到常见 shell 配置文件的 `PATH` 中。

### 覆盖可执行文件

工具安装不会覆盖 `bin` 目录中先前未由 uv 安装的可执行文件。例如，如果使用 `pipx` 安装了工具，`uv tool install` 将失败。可以使用 `--force` 标志覆盖此行为。

## 与 `uv run` 的关系

调用 `uv tool run <name>`（或 `uvx <name>`）几乎等同于：

```console
$ uv run --no-project --with <name> -- <name>
```

但是，使用 uv 的工具接口时有一些显著的区别：

- 不需要 `--with` 选项 —— 所需包从命令名称推断。
- 临时环境缓存在专用位置。
- 不需要 `--no-project` 标志 —— 工具始终与项目隔离运行。
- 如果工具已安装，`uv tool run` 将使用已安装的版本，但 `uv run` 不会。

如果工具不应与项目隔离，例如运行 `pytest` 或 `mypy`，则应使用 `uv run` 而不是 `uv tool run`。