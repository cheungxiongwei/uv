# 配置文件

uv 支持在项目级别和用户级别使用持久化配置文件。

具体来说，uv 会在当前目录或最近的父目录中搜索 `pyproject.toml` 或 `uv.toml` 文件。

!!! note

    对于 `tool` 命令，由于它们在用户级别操作，本地配置文件将被忽略。相反，uv 将仅从用户级别配置（例如 `~/.config/uv/uv.toml`）和系统级别配置（例如 `/etc/uv/uv.toml`）中读取。

在工作区中，uv 将从工作区根目录开始搜索，忽略工作区成员中定义的任何配置。由于工作区作为一个整体被锁定，配置在所有成员之间共享。

如果找到 `pyproject.toml` 文件，uv 将从 `[tool.uv]` 表中读取配置。例如，要设置持久化的索引 URL，可以在 `pyproject.toml` 中添加以下内容：

```toml title="pyproject.toml"
[[tool.uv.index]]
url = "https://test.pypi.org/simple"
default = true
```

（如果没有这样的表，`pyproject.toml` 文件将被忽略，uv 将继续在目录层次结构中搜索。）

uv 还会搜索 `uv.toml` 文件，其结构相同，但省略了 `[tool.uv]` 前缀。例如：

```toml title="uv.toml"
[[index]]
url = "https://test.pypi.org/simple"
default = true
```

!!! note

    `uv.toml` 文件的优先级高于 `pyproject.toml` 文件，因此如果目录中同时存在 `uv.toml` 和 `pyproject.toml` 文件，配置将从 `uv.toml` 中读取，而 `pyproject.toml` 中的 `[tool.uv]` 部分将被忽略。

uv 还会在 macOS 和 Linux 上发现用户级别配置位于 `~/.config/uv/uv.toml`（或 `$XDG_CONFIG_HOME/uv/uv.toml`），在 Windows 上位于 `%APPDATA%\uv\uv.toml`；在 macOS 和 Linux 上发现系统级别配置位于 `/etc/uv/uv.toml`（或 `$XDG_CONFIG_DIRS/uv/uv.toml`），在 Windows 上位于 `%SYSTEMDRIVE%\ProgramData\uv\uv.toml`。

用户和系统级别配置必须使用 `uv.toml` 格式，而不是 `pyproject.toml` 格式，因为 `pyproject.toml` 旨在定义 Python _项目_。

如果找到项目、用户和系统级别的配置文件，设置将被合并，项目级别配置优先于用户级别配置，用户级别配置优先于系统级别配置。（如果找到多个系统级别配置文件，例如在 `/etc/uv/uv.toml` 和 `$XDG_CONFIG_DIRS/uv/uv.toml`，仅使用第一个发现的文件，XDG 优先。）

例如，如果字符串、数字或布尔值同时存在于项目和用户级别配置表中，将使用项目级别的值，用户级别的值将被忽略。如果数组同时存在于两个表中，数组将被连接，项目级别的设置将出现在合并数组的前面。

通过环境变量提供的设置优先于持久化配置，通过命令行提供的设置优先于两者。

uv 接受 `--no-config` 命令行参数，当提供时，禁用任何持久化配置的发现。

uv 还接受 `--config-file` 命令行参数，该参数接受 `uv.toml` 文件的路径作为配置文件。当提供时，此文件将代替 _任何_ 发现的配置文件使用（例如，用户级别配置将被忽略）。

## 设置

有关可用设置的枚举，请参阅 [设置参考](../reference/settings.md)。

## `.env`

`uv run` 可以从 dotenv 文件（例如 `.env`、`.env.local`、`.env.development`）加载环境变量，由 [`dotenvy`](https://github.com/allan2/dotenvy) crate 提供支持。

要从专用位置加载 `.env` 文件，请设置 `UV_ENV_FILE` 环境变量，或向 `uv run` 传递 `--env-file` 标志。

例如，要从当前工作目录中的 `.env` 文件加载环境变量：

```console
$ echo "MY_VAR='Hello, world!'" > .env
$ uv run --env-file .env -- python -c 'import os; print(os.getenv("MY_VAR"))'
Hello, world!
```

`--env-file` 标志可以多次提供，后续文件将覆盖先前文件中定义的值。要通过 `UV_ENV_FILE` 环境变量提供多个文件，请用空格分隔路径（例如 `UV_ENV_FILE="/path/to/file1 /path/to/file2"`）。

要禁用 dotenv 加载（例如，覆盖 `UV_ENV_FILE` 或 `--env-file` 命令行参数），请将 `UV_NO_ENV_FILE` 环境变量设置为 `1`，或向 `uv run` 传递 `--no-env-file` 标志。

如果同一变量在环境和 `.env` 文件中都有定义，环境中的值将优先。

## 配置 pip 接口

提供了一个专用的 [`[tool.uv.pip]`](../reference/settings.md#pip) 部分，用于配置 _仅_ `uv pip` 命令行界面。此部分中的设置不会应用于 `uv pip` 命名空间之外的 `uv` 命令。然而，此部分中的许多设置在顶级命名空间中有对应的设置，除非被 `uv.pip` 部分中的值覆盖，否则这些设置将应用于 `uv pip` 接口。

`uv.pip` 设置旨在紧密遵循 pip 的接口，并单独声明以保持兼容性，同时允许全局设置使用替代设计（例如 `--no-build`）。

例如，在 `[tool.uv.pip]` 下设置 `index-url`，如下面的 `pyproject.toml` 所示，将仅影响 `uv pip` 子命令（例如 `uv pip install`，但不影响 `uv sync`、`uv lock` 或 `uv run`）：

```toml title="pyproject.toml"
[tool.uv.pip]
index-url = "https://test.pypi.org/simple"
```