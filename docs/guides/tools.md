---
title: Using tools
description:
  A guide to using uv to run tools published as Python packages, including one-off invocations with
  uvx, requesting specific tool versions, installing tools, upgrading tools, and more.
---

```md docs/guides/tools.md (8-245)
# 使用工具

许多 Python 包提供了可以作为工具使用的应用程序。uv 提供了专门的支持，可以轻松调用和安装这些工具。

## 运行工具

`uvx` 命令可以在不安装工具的情况下调用它。

例如，运行 `ruff`：

```console
$ uvx ruff
```

!!! note

    这完全等同于：

    ```console
    $ uv tool run ruff
    ```

    `uvx` 是一个为了方便而提供的别名。

可以在工具名称后提供参数：

```console
$ uvx pycowsay hello from uv

  -------------
< hello from uv >
  -------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||

```

使用 `uvx` 时，工具会被安装到临时的、隔离的环境中。

!!! note

    如果你在 [_项目_](../concepts/projects/index.md) 中运行工具，并且该工具要求安装你的项目，例如使用 `pytest` 或 `mypy` 时，你应该使用 [`uv run`](./projects.md#running-commands) 而不是 `uvx`。否则，工具将在与你的项目隔离的虚拟环境中运行。

    如果你的项目是扁平结构的，例如没有使用 `src` 目录来存放模块，那么项目本身不需要安装，使用 `uvx` 是没问题的。在这种情况下，使用 `uv run` 的唯一好处是如果你想在项目的依赖中固定工具的版本。

## 包名与命令名不同的情况

当调用 `uvx ruff` 时，uv 会安装提供 `ruff` 命令的 `ruff` 包。然而，有时包名和命令名并不相同。

可以使用 `--from` 选项来调用特定包中的命令，例如 `httpie` 包提供的 `http` 命令：

```console
$ uvx --from httpie http
```

## 请求特定版本

要运行特定版本的工具，使用 `command@<version>`：

```console
$ uvx ruff@0.3.0 check
```

要运行最新版本的工具，使用 `command@latest`：

```console
$ uvx ruff@latest check
```

`--from` 选项也可以用于指定包的版本，如上所述：

```console
$ uvx --from 'ruff==0.3.0' ruff check
```

或者，限制版本范围：

```console
$ uvx --from 'ruff>0.2.0,<0.3.0' ruff check
```

注意，`@` 语法只能用于指定确切的版本。

## 请求额外功能

`--from` 选项可以用于运行带有额外功能的工具：

```console
$ uvx --from 'mypy[faster-cache,reports]' mypy --xml-report mypy_report
```

这也可以与版本选择结合使用：

```console
$ uvx --from 'mypy[faster-cache,reports]==1.13.0' mypy --xml-report mypy_report
```

## 请求不同的来源

`--from` 选项还可以用于从其他来源安装工具。

例如，从 git 拉取：

```console
$ uvx --from git+https://github.com/httpie/cli httpie
```

你也可以从特定的分支拉取最新提交：

```console
$ uvx --from git+https://github.com/httpie/cli@master httpie
```

或者拉取特定的标签：

```console
$ uvx --from git+https://github.com/httpie/cli@3.2.4 httpie
```

甚至是特定的提交：

```console
$ uvx --from git+https://github.com/httpie/cli@2843b87 httpie
```

## 带有插件的命令

可以包含额外的依赖项，例如在运行 `mkdocs` 时包含 `mkdocs-material`：

```console
$ uvx --with mkdocs-material mkdocs --help
```

## 安装工具

如果一个工具经常使用，将其安装到持久化环境并添加到 `PATH` 中，而不是反复调用 `uvx`，会很有用。

!!! tip

    `uvx` 是 `uv tool run` 的便捷别名。所有其他与工具交互的命令都需要完整的 `uv tool` 前缀。

安装 `ruff`：

```console
$ uv tool install ruff
```

当工具安装后，其可执行文件会被放置在 `PATH` 中的 `bin` 目录下，这样工具就可以在不使用 uv 的情况下运行。如果它不在 `PATH` 中，会显示警告，并且可以使用 `uv tool update-shell` 将其添加到 `PATH` 中。

安装 `ruff` 后，它应该可用：

```console
$ ruff --version
```

与 `uv pip install` 不同，安装工具不会使其模块在当前环境中可用。例如，以下命令将失败：

```console
$ python -c "import ruff"
```

这种隔离对于减少工具、脚本和项目之间的依赖冲突非常重要。

与 `uvx` 不同，`uv tool install` 操作的是一个 _包_，并且会安装该工具提供的所有可执行文件。

例如，以下命令将安装 `http`、`https` 和 `httpie` 可执行文件：

```console
$ uv tool install httpie
```

此外，可以在不使用 `--from` 的情况下包含包版本：

```console
$ uv tool install 'httpie>0.1.0'
```

同样，对于包的来源：

```console
$ uv tool install git+https://github.com/httpie/cli
```

与 `uvx` 一样，安装时可以包含额外的包：

```console
$ uv tool install mkdocs --with mkdocs-material
```

## 升级工具

要升级工具，使用 `uv tool upgrade`：

```console
$ uv tool upgrade ruff
```

工具升级将遵循安装工具时提供的版本约束。例如，`uv tool install ruff >=0.3,<0.4` 后跟 `uv tool upgrade ruff` 会将 Ruff 升级到 `>=0.3,<0.4` 范围内的最新版本。

要替换版本约束，请使用 `uv tool install` 重新安装工具：

```console
$ uv tool install ruff>=0.4
```

要升级所有工具：

```console
$ uv tool upgrade --all
```

## 下一步

要了解更多关于使用 uv 管理工具的信息，请参阅 [工具概念](../concepts/tools.md) 页面和 [命令参考](../reference/cli.md#uv-tool)。

或者，继续阅读以了解如何 [处理项目](./projects.md)。
```
