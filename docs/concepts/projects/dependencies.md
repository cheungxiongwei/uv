## 添加依赖

要添加依赖项，请使用 `uv add` 命令：

```console
$ uv add httpx
```

这将把 `httpx` 添加到 `pyproject.toml` 的 `project.dependencies` 表中，并使用与当前 Python 版本兼容的最新版本。可以提供替代约束：

```console
$ uv add "httpx>=0.20"
```

当从非包注册表的源添加依赖项时，uv 将在源表中添加一个条目。例如，从 GitHub 添加 `httpx`：

```console
$ uv add "httpx @ git+https://github.com/encode/httpx"
```

`pyproject.toml` 将包含一个 [Git 源条目](#git)：

```toml title="pyproject.toml" hl_lines="8-9"
[project]
name = "example"
version = "0.1.0"
dependencies = [
    "httpx",
]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx" }
```

如果依赖项无法使用，uv 将显示错误：

```console
$ uv add "httpx>9999"
  × No solution found when resolving dependencies:
  ╰─▶ Because only httpx<=1.0.0b0 is available and your project depends on httpx>9999,
      we can conclude that your project's requirements are unsatisfiable.
```

### 导入依赖

可以使用 `-r` 选项将 `requirements.txt` 文件中声明的依赖项添加到项目中：

```
uv add -r requirements.txt
```

## 移除依赖

要移除依赖项：

```console
$ uv remove httpx
```

可以使用 `--dev`、`--group` 或 `--optional` 标志从特定表中移除依赖项。

如果为移除的依赖项定义了[源](#dependency-sources)，并且没有其他引用该依赖项，则源也将被移除。

## 更改依赖

要更改现有依赖项，例如为 `httpx` 使用不同的约束：

```console
$ uv add "httpx>0.1.0"
```

!!! note

    在此示例中，我们正在更改 `pyproject.toml` 中依赖项的约束。只有在满足新约束的情况下，依赖项的锁定版本才会更改。要强制将包版本升级到约束范围内的最新版本，请使用 `--upgrade-package <name>`，例如：

    ```console
    $ uv add "httpx>0.1.0" --upgrade-package httpx
    ```

    有关升级包的更多详细信息，请参阅 [lockfile](./sync.md#upgrading-locked-package-versions) 文档。

请求不同的依赖源将更新 `tool.uv.sources` 表，例如在开发期间使用本地路径的 `httpx`：

```console
$ uv add "httpx @ ../httpx"
```

## 平台特定依赖

要确保依赖项仅在特定平台或特定 Python 版本上安装，请使用 [环境标记](https://peps.python.org/pep-0508/#environment-markers)。

例如，在 Linux 上安装 `jax`，但在 Windows 或 macOS 上不安装：

```console
$ uv add "jax; sys_platform == 'linux'"
```

生成的 `pyproject.toml` 将在依赖项定义中包含环境标记：

```toml title="pyproject.toml" hl_lines="6"
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = ["jax; sys_platform == 'linux'"]
```

同样，要在 Python 3.11 及更高版本中包含 `numpy`：

```console
$ uv add "numpy; python_version >= '3.11'"
```

有关可用标记和运算符的完整枚举，请参阅 Python 的 [环境标记](https://peps.python.org/pep-0508/#environment-markers) 文档。

!!! tip

    依赖源也可以[按平台更改](#platform-specific-sources)。

## 项目依赖

`project.dependencies` 表表示在上传到 PyPI 或构建 wheel 时使用的依赖项。使用 [依赖项说明符](https://packaging.python.org/en/latest/specifications/dependency-specifiers/) 语法指定各个依赖项，该表遵循 [PEP 621](https://packaging.python.org/en/latest/specifications/pyproject-toml/) 标准。

`project.dependencies` 定义了项目所需的包列表，以及安装时应使用的版本约束。每个条目包括依赖项名称和版本。条目可能包括 extras 或用于平台特定包的环境标记。例如：

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
dependencies = [
  # 此范围内的任何版本
  "tqdm >=4.66.2,<5",
  # 确切此版本的 torch
  "torch ==2.2.2",
  # 使用 torch extra 安装 transformers
  "transformers[torch] >=4.39.3,<5",
  # 仅在较旧的 Python 版本上安装此包
  # 有关更多信息，请参阅“环境标记”
  "importlib_metadata >=7.1.0,<8; python_version < '3.10'",
  "mollymawk ==0.1.0"
]
```

## 依赖源

`tool.uv.sources` 表扩展了标准依赖表，提供了在开发期间使用的替代依赖源。

依赖源增加了对 `project.dependencies` 标准不支持的常见模式的支持，例如可编辑安装和相对路径。例如，要从相对于项目根目录的目录安装 `foo`：

```toml title="pyproject.toml" hl_lines="7"
[project]
name = "example"
version = "0.1.0"
dependencies = ["foo"]

[tool.uv.sources]
foo = { path = "./packages/foo" }
```

uv 支持以下依赖源：

- [Index](#index)：从特定包索引解析的包。
- [Git](#git)：Git 仓库。
- [URL](#url)：远程 wheel 或源代码分发。
- [Path](#path)：本地 wheel、源代码分发或项目目录。
- [Workspace](#workspace-member)：当前工作区的成员。

!!! important

    源仅由 uv 尊重。如果使用其他工具，则仅使用标准项目表中的定义。如果使用其他工具进行开发，则需要在其他工具的格式中重新指定源表中提供的任何元数据。

### Index

要从特定索引添加 Python 包，请使用 `--index` 选项：

```console
$ uv add torch --index pytorch=https://download.pytorch.org/whl/cpu
```

uv 将索引存储在 `[[tool.uv.index]]` 中，并添加 `[tool.uv.sources]` 条目：

```toml title="pyproject.toml"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
```

!!! tip

    由于 PyTorch 索引的具体情况，上述示例仅适用于 x86-64 Linux。有关设置 PyTorch 的更多信息，请参阅 [PyTorch 指南](../../guides/integration/pytorch.md)。

使用 `index` 源将包_固定_到给定索引——它不会从其他索引下载。

定义索引时，可以包含 `explicit` 标志，以指示索引应_仅_用于在 `tool.uv.sources` 中明确指定它的包。如果未设置 `explicit`，则如果未在其他地方找到，其他包可能会从该索引解析。

```toml title="pyproject.toml" hl_lines="3"
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

```md docs/concepts/projects/dependencies.md (209-729)
### Git

要添加 Git 依赖源，请在 Git 兼容的 URL（即与 `git clone` 命令使用的 URL 相同）前加上 `git+`。

例如：

```console
$ uv add git+https://github.com/encode/httpx
```

```toml title="pyproject.toml" hl_lines="5"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx" }
```

可以请求特定的 Git 引用，例如标签：

```console
$ uv add git+https://github.com/encode/httpx --tag 0.27.0
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.0" }
```

或者，指定分支：

```console
$ uv add git+https://github.com/encode/httpx --branch main
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", branch = "main" }
```

或者，指定修订版本（commit）：

```console
$ uv add git+https://github.com/encode/httpx --rev 326b9431c761e1ef1e00b9f760d1f654c8db48c6
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", rev = "326b9431c761e1ef1e00b9f760d1f654c8db48c6" }
```

如果包不在仓库根目录，可以指定 `subdirectory`：

```console
$ uv add git+https://github.com/langchain-ai/langchain#subdirectory=libs/langchain
```

```toml title="pyproject.toml"
[project]
dependencies = ["langchain"]

[tool.uv.sources]
langchain = { git = "https://github.com/langchain-ai/langchain", subdirectory = "libs/langchain" }
```

### URL

要添加 URL 源，请提供指向 wheel 文件（以 `.whl` 结尾）或源码分发文件（通常以 `.tar.gz` 或 `.zip` 结尾；参见[此处](../../concepts/resolution.md#source-distribution)了解所有支持的格式）的 `https://` URL。

例如：

```console
$ uv add "https://files.pythonhosted.org/packages/5c/2d/3da5bdf4408b8b2800061c339f240c1802f2e82d55e50bd39c5a881f47f0/httpx-0.27.0.tar.gz"
```

将在 `pyproject.toml` 中生成以下内容：

```toml title="pyproject.toml" hl_lines="5"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { url = "https://files.pythonhosted.org/packages/5c/2d/3da5bdf4408b8b2800061c339f240c1802f2e82d55e50bd39c5a881f47f0/httpx-0.27.0.tar.gz" }
```

URL 依赖也可以手动添加或使用 `{ url = <url> }` 语法在 `pyproject.toml` 中编辑。如果源码分发文件不在存档根目录，可以指定 `subdirectory`。

### Path

要添加路径源，请提供 wheel 文件（以 `.whl` 结尾）、源码分发文件（通常以 `.tar.gz` 或 `.zip` 结尾；参见[此处](../../concepts/resolution.md#source-distribution)了解所有支持的格式）或包含 `pyproject.toml` 的目录的路径。

例如：

```console
$ uv add /example/foo-0.1.0-py3-none-any.whl
```

将在 `pyproject.toml` 中生成以下内容：

```toml title="pyproject.toml"
[project]
dependencies = ["foo"]

[tool.uv.sources]
foo = { path = "/example/foo-0.1.0-py3-none-any.whl" }
```

路径也可以是相对路径：

```console
$ uv add ./foo-0.1.0-py3-none-any.whl
```

或者，指向项目目录的路径：

```console
$ uv add ~/projects/bar/
```

!!! important

    默认情况下，路径依赖不会使用[可编辑安装](#editable-dependencies)。可以为项目目录请求可编辑安装：

    ```console
    $ uv add --editable ~/projects/bar/
    ```

    对于同一仓库中的多个包，[_workspaces_](./workspaces.md) 可能是更好的选择。

### Workspace member

要声明对 workspace 成员的依赖，请使用 `{ workspace = true }` 添加成员名称。所有 workspace 成员必须明确声明。Workspace 成员始终是[可编辑的](#editable-dependencies)。有关 workspaces 的更多详细信息，请参阅 [workspace](./workspaces.md) 文档。

```toml title="pyproject.toml"
[project]
dependencies = ["foo==0.1.0"]

[tool.uv.sources]
foo = { workspace = true }

[tool.uv.workspace]
members = [
  "packages/foo"
]
```

### 平台特定源

您可以通过为源提供[依赖说明符](https://packaging.python.org/en/latest/specifications/dependency-specifiers/)兼容的环境标记来限制源在特定平台或 Python 版本上的使用。

例如，要在 macOS 上从 GitHub 拉取 `httpx`，请使用以下配置：

```toml title="pyproject.toml" hl_lines="8"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.2", marker = "sys_platform == 'darwin'" }
```

通过在源上指定标记，uv 仍会在所有平台上包含 `httpx`，但仅在 macOS 上从 GitHub 下载源，而在其他平台上回退到 PyPI。

### 多源

您可以通过提供源列表为单个依赖指定多个源，使用 [PEP 508](https://peps.python.org/pep-0508/#environment-markers) 兼容的环境标记进行区分。

例如，在 macOS 和 Linux 上拉取不同的 `httpx` 标签：

```toml title="pyproject.toml" hl_lines="8-9 13-14"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = [
  { git = "https://github.com/encode/httpx", tag = "0.27.2", marker = "sys_platform == 'darwin'" },
  { git = "https://github.com/encode/httpx", tag = "0.24.1", marker = "sys_platform == 'linux'" },
]
```

此策略可扩展到基于环境标记使用不同的索引。例如，根据平台从不同的 PyTorch 索引安装 `torch`：

```toml title="pyproject.toml" hl_lines="6-7"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", marker = "platform_system == 'Darwin'"},
  { index = "torch-gpu", marker = "platform_system == 'Linux'"},
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu124"
```

### 禁用源

要指示 uv 忽略 `tool.uv.sources` 表（例如，模拟使用包的发布元数据进行解析），请使用 `--no-sources` 标志：

```console
$ uv lock --no-sources
```

使用 `--no-sources` 还会阻止 uv 发现任何可能满足给定依赖的[workspace 成员](#workspace-member)。

## 可选依赖

作为库发布的项目通常会使其某些功能成为可选的，以减少默认的依赖树。例如，Pandas 有一个[`excel` 额外依赖](https://pandas.pydata.org/docs/getting_started/install.html#excel-files)和一个[`plot` 额外依赖](https://pandas.pydata.org/docs/getting_started/install.html#visualization)，以避免在未明确要求时安装 Excel 解析器和 `matplotlib`。额外依赖使用 `package[<extra>]` 语法请求，例如 `pandas[plot, excel]`。

可选依赖在 `[project.optional-dependencies]` 中指定，这是一个 TOML 表，将额外依赖名称映射到其依赖项，遵循[依赖说明符](#dependency-specifiers-pep-508)语法。

可选依赖可以像普通依赖一样在 `tool.uv.sources` 中有条目。

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "1.0.0"

[project.optional-dependencies]
plot = [
  "matplotlib>=3.6.3"
]
excel = [
  "odfpy>=1.4.1",
  "openpyxl>=3.1.0",
  "python-calamine>=0.1.7",
  "pyxlsb>=1.0.10",
  "xlrd>=2.0.1",
  "xlsxwriter>=3.0.5"
]
```

要添加可选依赖，请使用 `--optional <extra>` 选项：

```console
$ uv add httpx --optional network
```

!!! note

    如果您的可选依赖相互冲突，除非您明确[将它们声明为冲突](./config.md#conflicting-dependencies)，否则解析将失败。

源也可以声明为仅适用于特定的可选依赖。例如，根据可选的 `cpu` 或 `gpu` 额外依赖从不同的 PyTorch 索引拉取 `torch`：

```toml title="pyproject.toml"
[project]
dependencies = []

[project.optional-dependencies]
cpu = [
  "torch",
]
gpu = [
  "torch",
]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", extra = "cpu" },
  { index = "torch-gpu", extra = "gpu" },
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu124"
```

## 开发依赖

与可选依赖不同，开发依赖是本地专用的，在发布到 PyPI 或其他索引时不会包含在项目需求中。因此，开发依赖不包含在 `[project]` 表中。

开发依赖可以像普通依赖一样在 `tool.uv.sources` 中有条目。

要添加开发依赖，请使用 `--dev` 标志：

```console
$ uv add --dev pytest
```

uv 使用 `[dependency-groups]` 表（如 [PEP 735](https://peps.python.org/pep-0735/) 中所定义）来声明开发依赖。上述命令将创建一个 `dev` 组：

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  "pytest >=8.1.1,<9"
]
```

`dev` 组是特殊处理的；有 `--dev`、`--only-dev` 和 `--no-dev` 标志来切换其依赖的包含或排除。请参阅 `--no-default-groups` 以禁用所有默认组。此外，`dev` 组[默认同步](#default-groups)。

### 依赖组

开发依赖可以划分为多个组，使用 `--group` 标志。

例如，要在 `lint` 组中添加开发依赖：

```console
$ uv add --group lint ruff
```

这将生成以下 `[dependency-groups]` 定义：

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  "pytest"
]
lint = [
  "ruff"
]
```

一旦定义了组，可以使用 `--all-groups`、`--no-default-groups`、`--group`、`--only-group` 和 `--no-group` 选项来包含或排除它们的依赖。

!!! tip

    `--dev`、`--only-dev` 和 `--no-dev` 标志分别等同于 `--group dev`、`--only-group dev` 和 `--no-group dev`。

uv 要求所有依赖组彼此兼容，并在创建锁文件时一起解析所有组。

如果一个组中声明的依赖与另一个组中的依赖不兼容，uv 将无法解析项目需求并报错。

!!! note

    如果您的依赖组相互冲突，除非您明确[将它们声明为冲突](./config.md#conflicting-dependencies)，否则解析将失败。

### 默认组

默认情况下，uv 在环境中包含 `dev` 依赖组（例如，在 `uv run` 或 `uv sync` 期间）。可以使用 `tool.uv.default-groups` 设置更改包含的默认组。

```toml title="pyproject.toml"
[tool.uv]
default-groups = ["dev", "foo"]
```

!!! tip

    要在 `uv run` 或 `uv sync` 期间禁用此行为，请使用 `--no-default-groups`。
    要排除特定的默认组，请使用 `--no-group <name>`。

### 旧版 `dev-dependencies`

在 `[dependency-groups]` 标准化之前，uv 使用 `tool.uv.dev-dependencies` 字段来指定开发依赖，例如：

```toml title="pyproject.toml"
[tool.uv]
dev-dependencies = [
  "pytest"
]
```

在此部分声明的依赖将与 `dependency-groups.dev` 中的内容合并。最终，`dev-dependencies` 字段将被弃用并移除。

!!! note

    如果存在 `tool.uv.dev-dependencies` 字段，`uv add --dev` 将使用现有部分，而不是添加新的 `dependency-groups.dev` 部分。

## 构建依赖

如果项目结构为 [Python 包](./config.md#build-systems)，它可能会声明构建项目所需的依赖，但运行项目时不需要这些依赖。这些依赖在 `[build-system]` 表中的 `build-system.requires` 下指定，遵循 [PEP 518](https://peps.python.org/pep-0518/)。

例如，如果项目使用 `setuptools` 作为其构建后端，则应声明 `setuptools` 为构建依赖：

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"
```

默认情况下，uv 在解析构建依赖时会尊重 `tool.uv.sources`。例如，要使用本地版本的 `setuptools` 进行构建，请将源添加到 `tool.uv.sources`：

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"

[tool.uv.sources]
setuptools = { path = "./packages/setuptools" }
```

在发布包时，我们建议运行 `uv build --no-sources` 以确保在禁用 `tool.uv.sources` 时包能

## 构建依赖

如果项目结构为 [Python 包](./config.md#build-systems)，它可以声明构建项目所需的依赖项，但这些依赖项在运行时并不需要。这些依赖项在 `[build-system]` 表中的 `build-system.requires` 下指定，遵循 [PEP 518](https://peps.python.org/pep-0518/)。

例如，如果项目使用 `setuptools` 作为其构建后端，则应声明 `setuptools` 为构建依赖项：

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"
```

默认情况下，uv 在解析构建依赖项时会遵循 `tool.uv.sources`。例如，要使用本地版本的 `setuptools` 进行构建，请将源添加到 `tool.uv.sources`：

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"

[tool.uv.sources]
setuptools = { path = "./packages/setuptools" }
```

在发布包时，建议运行 `uv build --no-sources`，以确保在禁用 `tool.uv.sources` 的情况下包能够正确构建，因为在使用其他构建工具（如 [`pypa/build`](https://github.com/pypa/build)）时通常会禁用该配置。

## 可编辑依赖

Python 包的常规安装会首先构建一个 wheel 文件，然后将该 wheel 文件安装到虚拟环境中，复制所有源文件。当包源文件被编辑时，虚拟环境中的文件将变为过时版本。

可编辑安装通过向虚拟环境中添加一个指向项目的链接（一个 `.pth` 文件）来解决此问题，该链接指示解释器直接包含源文件。

可编辑安装有一些限制（主要是：构建后端需要支持它们，并且在导入之前不会重新编译原生模块），但它们对开发非常有用，因为虚拟环境将始终使用包的最新更改。

uv 默认对工作区包使用可编辑安装。

要添加可编辑依赖项，请使用 `--editable` 标志：

```console
$ uv add --editable ./path/foo
```

或者，要在工作区中退出使用可编辑依赖项：

```console
$ uv add --no-editable ./path/foo
```

## 依赖项说明符 (PEP 508)

uv 使用 [依赖项说明符](https://packaging.python.org/en/latest/specifications/dependency-specifiers/)，以前称为 [PEP 508](https://peps.python.org/pep-0508/)。依赖项说明符按顺序由以下部分组成：

- 依赖项名称
- 所需的 extras（可选）
- 版本说明符
- 环境标记（可选）

版本说明符以逗号分隔并组合在一起，例如，`foo >=1.2.3,<2,!=1.4.0` 被解释为“`foo` 的版本至少为 1.2.3，但小于 2，且不为 1.4.0”。

如果需要，说明符会用尾随零填充，因此 `foo ==2` 也匹配 foo 2.0.0。

可以使用星号表示最后一个数字，例如 `foo ==2.1.*` 将接受 2.1 系列的任何版本。类似地，`~=` 匹配最后一个数字相等或更高的版本，例如，`foo ~=1.2` 等同于 `foo >=1.2,<2`，而 `foo ~=1.2.3` 等同于 `foo >=1.2.3,<1.3`。

extras 在名称和版本之间的方括号中以逗号分隔，例如 `pandas[excel,plot] ==2.2`。extras 名称之间的空格会被忽略。

某些依赖项仅在特定环境中需要，例如特定的 Python 版本或操作系统。例如，要为 `importlib.metadata` 模块安装 `importlib-metadata` 回退，请使用 `importlib-metadata >=7.1.0,<8; python_version < '3.10'`。要在 Windows 上安装 `colorama`（但在其他平台上省略），请使用 `colorama >=0.4.6,<5; platform_system == "Windows"`。

标记通过 `and`、`or` 和括号组合，例如 `aiohttp >=3.7.4,<4; (sys_platform != 'win32' or implementation_name != 'pypy') and python_version >= '3.10'`。请注意，标记中的版本必须用引号括起来，而标记外的版本则不能使用引号。
