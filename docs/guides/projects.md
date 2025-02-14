---
title: Working on projects
description:
  A guide to using uv to create and manage Python projects, including adding dependencies, running
  commands, and building publishable distributions.
---

# 项目工作

uv 支持管理 Python 项目，这些项目在 `pyproject.toml` 文件中定义了它们的依赖关系。

## 创建新项目

你可以使用 `uv init` 命令创建一个新的 Python 项目：

```console
$ uv init hello-world
$ cd hello-world
```

或者，你可以在当前工作目录中初始化一个项目：

```console
$ mkdir hello-world
$ cd hello-world
$ uv init
```

uv 将创建以下文件：

```text
.
├── .python-version
├── README.md
├── hello.py
└── pyproject.toml
```

`hello.py` 文件包含一个简单的 "Hello world" 程序。你可以使用 `uv run` 来运行它：

```console
$ uv run hello.py
Hello from hello-world!
```

## 项目结构

项目由几个重要的部分组成，它们共同工作，使 uv 能够管理你的项目。除了 `uv init` 创建的文件外，uv 还会在第一次运行项目命令（如 `uv run`、`uv sync` 或 `uv lock`）时，在项目根目录下创建一个虚拟环境和 `uv.lock` 文件。

完整的文件列表如下：

```text
.
├── .venv
│   ├── bin
│   ├── lib
│   └── pyvenv.cfg
├── .python-version
├── README.md
├── hello.py
├── pyproject.toml
└── uv.lock
```

### `pyproject.toml`

`pyproject.toml` 包含项目的元数据：

```toml title="pyproject.toml"
[project]
name = "hello-world"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
dependencies = []
```

你可以使用此文件来指定依赖项，以及项目的描述或许可证等详细信息。你可以手动编辑此文件，或使用 `uv add` 和 `uv remove` 等命令从终端管理你的项目。

!!! tip

    有关 `pyproject.toml` 格式的更多详细信息，请参阅官方的 [`pyproject.toml` 指南](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)。

你还可以在此文件中使用 [`[tool.uv]`](../reference/settings.md) 部分来指定 uv 的[配置选项](../configuration/files.md)。

### `.python-version`

`.python-version` 文件包含项目的默认 Python 版本。此文件告诉 uv 在创建项目的虚拟环境时使用哪个 Python 版本。

### `.venv`

`.venv` 文件夹包含项目的虚拟环境，这是一个与系统其他部分隔离的 Python 环境。uv 将在此处安装项目的依赖项。

有关更多详细信息，请参阅[项目环境](../concepts/projects/layout.md#the-project-environment)文档。

### `uv.lock`

`uv.lock` 是一个跨平台的锁文件，包含项目依赖项的确切信息。与用于指定项目广泛需求的 `pyproject.toml` 不同，锁文件包含安装在项目环境中的确切解析版本。此文件应提交到版本控制中，以便在不同机器上实现一致且可重复的安装。

`uv.lock` 是一个人类可读的 TOML 文件，但由 uv 管理，不应手动编辑。

有关更多详细信息，请参阅[锁文件](../concepts/projects/layout.md#the-lockfile)文档。

## 管理依赖项

你可以使用 `uv add` 命令将依赖项添加到 `pyproject.toml` 中。这还将更新锁文件和项目环境：

```console
$ uv add requests
```

你还可以指定版本约束或替代来源：

```console
$ # 指定版本约束
$ uv add 'requests==2.31.0'

$ # 添加 git 依赖项
$ uv add git+https://github.com/psf/requests
```

要删除一个包，你可以使用 `uv remove`：

```console
$ uv remove requests
```

要升级一个包，可以运行带有 `--upgrade-package` 标志的 `uv lock`：

```console
$ uv lock --upgrade-package requests
```

`--upgrade-package` 标志将尝试将指定的包更新到最新的兼容版本，同时保持锁文件的其余部分不变。

有关更多详细信息，请参阅[管理依赖项](../concepts/projects/dependencies.md)文档。

## 运行命令

`uv run` 可用于在项目环境中运行任意脚本或命令。

在每次 `uv run` 调用之前，uv 都会验证锁文件是否与 `pyproject.toml` 保持同步，以及环境是否与锁文件保持同步，从而无需手动干预即可保持项目同步。`uv run` 确保你的命令在一致、锁定的环境中运行。

例如，使用 `flask`：

```console
$ uv add flask
$ uv run -- flask run -p 3000
```

或者，运行一个脚本：

```python title="example.py"
# 需要一个项目依赖项
import flask

print("hello world")
```

```console
$ uv run example.py
```

或者，你可以使用 `uv sync` 手动更新环境，然后在执行命令之前激活它：

=== "macOS 和 Linux"

    ```console
    $ uv sync
    $ source .venv/bin/activate
    $ flask run -p 3000
    $ python example.py
    ```

=== "Windows"

    ```powershell
    uv sync
    source .venv\Scripts\activate
    flask run -p 3000
    python example.py
    ```

!!! note

    虚拟环境必须处于激活状态才能在不使用 `uv run` 的情况下在项目中运行脚本和命令。虚拟环境的激活方式因 shell 和平台而异。

有关在项目中[运行命令和脚本](../concepts/projects/run.md)的更多详细信息，请参阅相关文档。

## 构建分发包

`uv build` 可用于为项目构建源代码分发包和二进制分发包（wheel）。

默认情况下，`uv build` 将在当前目录中构建项目，并将构建的工件放置在 `dist/` 子目录中：

```console
$ uv build
$ ls dist/
hello-world-0.1.0-py3-none-any.whl
hello-world-0.1.0.tar.gz
```

有关[构建项目](../concepts/projects/build.md)的更多详细信息，请参阅相关文档。

## 下一步

要了解更多关于使用 uv 进行项目工作的信息，请参阅[项目概念](../concepts/projects/index.md)页面和[命令参考](../reference/cli.md#uv)。

或者，继续阅读以了解如何[构建并将你的项目发布到包索引](./package.md)。
