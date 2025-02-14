# Python 版本

Python 版本由 Python 解释器（即 `python` 可执行文件）、标准库和其他支持文件组成。

## 托管和系统 Python 安装

由于系统中通常已经存在 Python 安装，uv 支持[发现](#discovery-of-python-versions) Python 版本。然而，uv 也支持[安装 Python 版本](#installing-a-python-version)本身。为了区分这两种类型的 Python 安装，uv 将其安装的 Python 版本称为 _托管_ Python 安装，而其他所有 Python 安装称为 _系统_ Python 安装。

!!! note

    uv 不会区分由操作系统安装的 Python 版本与由其他工具安装和管理的 Python 版本。例如，如果 Python 安装由 `pyenv` 管理，它仍将被视为 uv 中的 _系统_ Python 版本。

## 请求版本

在大多数 uv 命令中，可以使用 `--python` 标志请求特定的 Python 版本。例如，在创建虚拟环境时：

```console
$ uv venv --python 3.11.6
```

uv 将确保 Python 3.11.6 可用——如果需要则下载并安装——然后使用它创建虚拟环境。

支持以下 Python 版本请求格式：

- `<version>` 例如 `3`, `3.12`, `3.12.3`
- `<version-specifier>` 例如 `>=3.12,<3.13`
- `<implementation>` 例如 `cpython` 或 `cp`
- `<implementation>@<version>` 例如 `cpython@3.12`
- `<implementation><version>` 例如 `cpython3.12` 或 `cp312`
- `<implementation><version-specifier>` 例如 `cpython>=3.12,<3.13`
- `<implementation>-<version>-<os>-<arch>-<libc>` 例如 `cpython-3.12.3-macos-aarch64-none`

此外，可以使用以下方式请求特定的系统 Python 解释器：

- `<executable-path>` 例如 `/opt/homebrew/bin/python3`
- `<executable-name>` 例如 `mypython3`
- `<install-dir>` 例如 `/some/environment/`

默认情况下，如果系统中找不到 Python 版本，uv 会自动下载。可以通过[`python-downloads` 选项](#disabling-automatic-python-downloads)禁用此行为。

### Python 版本文件

`.python-version` 文件可用于创建默认的 Python 版本请求。uv 会在工作目录及其每个父目录中搜索 `.python-version` 文件。可以使用上述任何请求格式，但建议使用版本号以确保与其他工具的互操作性。

可以使用 [`uv python pin`](../reference/cli.md/#uv-python-pin) 命令在当前目录中创建 `.python-version` 文件。

可以通过 `--no-config` 禁用 `.python-version` 文件的发现。

uv 不会在项目或工作区边界之外搜索 `.python-version` 文件。

## 安装 Python 版本

uv 捆绑了 macOS、Linux 和 Windows 上可下载的 CPython 和 PyPy 发行版列表。

!!! tip

    默认情况下，Python 版本会在需要时自动下载，而无需使用 `uv python install`。

要安装特定版本的 Python：

```console
$ uv python install 3.12.3
```

要安装最新的补丁版本：

```console
$ uv python install 3.12
```

要安装满足约束的版本：

```console
$ uv python install '>=3.8,<3.10'
```

要安装多个版本：

```console
$ uv python install 3.9 3.10 3.11
```

要安装特定的实现：

```console
$ uv python install pypy
```

支持所有 [Python 版本请求](#requesting-a-version) 格式，除了用于请求本地解释器的格式，如文件路径。

默认情况下，`uv python install` 会验证托管 Python 版本是否已安装，或安装最新版本。如果存在 `.python-version` 文件，uv 将安装文件中列出的 Python 版本。需要多个 Python 版本的项目可以定义 `.python-versions` 文件。如果存在，uv 将安装文件中列出的所有 Python 版本。

!!! important

    可用的 Python 版本在每个 uv 版本中是固定的。要安装新的 Python 版本，可能需要升级 uv。

### 安装 Python 可执行文件

!!! important

    安装 Python 可执行文件的功能处于 _预览_ 阶段，这意味着该行为是实验性的，可能会发生变化。

要将 Python 可执行文件安装到 `PATH` 中，请提供 `--preview` 选项：

```console
$ uv python install 3.12 --preview
```

这将在 `~/.local/bin` 中安装请求版本的 Python 可执行文件，例如 `python3.12`。

!!! tip

    如果 `~/.local/bin` 不在 `PATH` 中，可以使用 `uv tool update-shell` 添加它。

要安装 `python` 和 `python3` 可执行文件，请包含 `--default` 选项：

```console
$ uv python install 3.12 --default --preview
```

在安装 Python 可执行文件时，uv 只会覆盖由 uv 管理的现有可执行文件——例如，如果 `~/.local/bin/python3.12` 已经存在，uv 不会在没有 `--force` 标志的情况下覆盖它。

uv 会更新它管理的可执行文件。然而，默认情况下，它会优先选择每个 Python 次要版本的最新补丁版本。例如：

```console
$ uv python install 3.12.7 --preview  # 将 `python3.12` 添加到 `~/.local/bin`
$ uv python install 3.12.6 --preview  # 不会更新 `python3.12`
$ uv python install 3.12.8 --preview  # 将 `python3.12` 更新为 3.12.8
```

## 项目 Python 版本

uv 会在项目命令调用期间尊重 `pyproject.toml` 文件中 `requires-python` 定义的 Python 要求。将使用第一个与要求兼容的 Python 版本，除非通过 `.python-version` 文件或 `--python` 标志请求了其他版本。

## 查看可用的 Python 版本

要列出已安装和可用的 Python 版本：

```console
$ uv python list
```

默认情况下，隐藏其他平台和旧补丁版本的下载。

要查看所有版本：

```console
$ uv python list --all-versions
```

要查看其他平台的 Python 版本：

```console
$ uv python list --all-platforms
```

要排除下载并仅显示已安装的 Python 版本：

```console
$ uv python list --only-installed
```

有关更多详细信息，请参阅 [`uv python list`](../reference/cli.md#uv-python-list) 参考。

## 查找 Python 可执行文件

要查找 Python 可执行文件，请使用 `uv python find` 命令：

```console
$ uv python find
```

默认情况下，这将显示第一个可用 Python 可执行文件的路径。有关如何发现可执行文件的详细信息，请参阅[发现规则](#discovery-of-python-versions)。

此接口还支持许多[请求格式](#requesting-a-version)，例如，查找版本为 3.11 或更新的 Python 可执行文件：

```console
$ uv python find >=3.11
```

默认情况下，`uv python find` 会包含来自虚拟环境的 Python 版本。如果在工作目录或任何父目录中找到 `.venv` 目录，或者设置了 `VIRTUAL_ENV` 环境变量，它将优先于 `PATH` 上的任何 Python 可执行文件。

要忽略虚拟环境，请使用 `--system` 标志：

```console
$ uv python find --system
```

## Python 版本的发现

在搜索 Python 版本时，会检查以下位置：

- `UV_PYTHON_INSTALL_DIR` 中的托管 Python 安装。
- `PATH` 上的 Python 解释器，如 macOS 和 Linux 上的 `python`、`python3` 或 `python3.x`，或 Windows 上的 `python.exe`。
- 在 Windows 上，Windows 注册表中的 Python 解释器和 Microsoft Store Python 解释器（参见 `py --list-paths`）与请求的版本匹配。

在某些情况下，uv 允许使用来自虚拟环境的 Python 版本。在这种情况下，虚拟环境的解释器将在搜索安装之前检查是否与请求兼容。有关详细信息，请参阅 [pip 兼容的虚拟环境发现](../pip/environments.md#discovery-of-python-environments) 文档。

在执行发现时，非可执行文件将被忽略。每个发现的可执行文件都会被查询元数据，以确保其满足[请求的 Python 版本](#requesting-a-version)。如果查询失败，将跳过该可执行文件。如果可执行文件满足请求，则使用它而不检查其他可执行文件。

在搜索托管 Python 版本时，uv 会优先选择较新的版本。在搜索系统 Python 版本时，uv 会使用第一个兼容的版本——而不是最新的版本。

如果系统中找不到 Python 版本，uv 将检查是否有兼容的托管 Python 版本下载。

### Python 预发布版本

默认情况下，不会选择 Python 预发布版本。如果没有其他可用的安装与请求匹配，将使用 Python 预发布版本。例如，如果只有预发布版本可用，则将使用它，否则将使用稳定发布版本。同样，如果提供了预发布 Python 可执行文件的路径，则没有其他 Python 版本与请求匹配，将使用预发布版本。

如果预发布 Python 版本可用并与请求匹配，uv 不会下载稳定 Python 版本。

## 禁用自动 Python 下载

默认情况下，uv 会在需要时自动下载 Python 版本。

可以使用 [`python-downloads`](../reference/settings.md#python-downloads) 选项禁用此行为。默认情况下，它设置为 `automatic`；设置为 `manual` 以仅在 `uv python install` 期间允许 Python 下载。

!!! tip

    `python-downloads` 设置可以在[持久配置文件](../configuration/files.md)中设置以更改默认行为，或者可以将 `--no-python-downloads` 标志传递给任何 uv 命令。

## 调整 Python 版本偏好

默认情况下，uv 会尝试使用系统中找到的 Python 版本，仅在必要时下载托管解释器。

可以使用 [`python-preference`](../reference/settings.md#python-preference) 选项调整此行为。默认情况下，它设置为 `managed`，即优先选择托管 Python 安装而不是系统 Python 安装。然而，系统 Python 安装仍然优先于下载托管 Python 版本。

以下替代选项可用：

- `only-managed`：仅使用托管 Python 安装；从不使用系统 Python 安装
- `system`：优先选择系统 Python 安装而不是托管 Python 安装
- `only-system`：仅使用系统 Python 安装；从不使用托管 Python 安装

这些选项允许完全禁用 uv 的托管 Python 版本，或者始终使用它们并忽略任何现有的系统安装。

!!! note

    可以在不更改偏好的情况下[禁用](#disabling-automatic-python-downloads)自动 Python 版本下载。

## Python 实现支持

uv 支持 CPython、PyPy 和 GraalPy Python 实现。如果 Python 实现不受支持，uv 将无法发现其解释器。

可以使用长名称或短名称请求实现：

- CPython: `cpython`, `cp`
- PyPy: `pypy`, `pp`
- GraalPy: `graalpy`, `gp`

实现名称请求不区分大小写。

有关支持的格式的更多详细信息，请参阅 [Python 版本请求](#requesting-a-version) 文档。

## 托管 Python 发行版

uv 支持下载和安装 CPython 和 PyPy 发行版。

### CPython 发行版

由于 Python 不发布官方的可分发 CPython 二进制文件，uv 转而使用来自 Astral [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) 项目的预构建发行版。`python-build-standalone` 也被许多其他 Python 项目使用，如 [Rye](https://github.com/astral-sh/rye)、[Mise](https://mise.jdx.dev/lang/python.html) 和 [bazelbuild/rules_python](https://github.com/bazelbuild/rules_python)。

uv 的 Python 发行版是自包含的、高度可移植的且性能优异。虽然 Python 可以从源代码构建，如在 `pyenv` 等工具中，但这样做需要预安装系统依赖项，并且创建优化的、性能优异的构建（例如，启用 PGO 和 LTO）非常慢。

这些发行版有一些行为怪癖，通常是可移植性的结果；目前，uv 不支持在基于 musl 的 Linux 发行版（如 Alpine Linux）上安装它们。有关详细信息，请参阅 [`python-build-standalone` 怪癖](https://gregoryszorc.com/docs/python-build-standalone/main/quirks.html) 文档。

### PyPy 发行版

PyPy 发行版由 PyPy 项目提供。