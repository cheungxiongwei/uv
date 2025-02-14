# 配置项目

## Python 版本要求

项目可以在 `pyproject.toml` 的 `project.requires-python` 字段中声明项目支持的 Python 版本。

建议设置 `requires-python` 值：

```toml title="pyproject.toml" hl_lines="4"
[project]
name = "example"
version = "0.1.0"
requires-python = ">=3.12"
```

Python 版本要求决定了项目中允许的 Python 语法，并影响依赖版本的选择（它们必须支持相同的 Python 版本范围）。

## 入口点

[入口点](https://packaging.python.org/en/latest/specifications/entry-points/#entry-points)是已安装包用于公开接口的官方术语。这些包括：

- [命令行界面](#command-line-interfaces)
- [图形用户界面](#graphical-user-interfaces)
- [插件入口点](#plugin-entry-points)

!!! important

    使用入口点表需要定义[构建系统](#build-systems)。

### 命令行界面

项目可以在 `pyproject.toml` 的 `[project.scripts]` 表中定义命令行界面（CLIs）。

例如，声明一个名为 `hello` 的命令，调用 `example` 模块中的 `hello` 函数：

```toml title="pyproject.toml"
[project.scripts]
hello = "example:hello"
```

然后，可以从控制台运行该命令：

```console
$ uv run hello
```

### 图形用户界面

项目可以在 `pyproject.toml` 的 `[project.gui-scripts]` 表中定义图形用户界面（GUIs）。

!!! important

    这些仅在 Windows 上与[命令行界面](#command-line-interfaces)不同，它们被 GUI 可执行文件包装，因此可以在没有控制台的情况下启动。在其他平台上，它们的行为相同。

例如，声明一个名为 `hello` 的命令，调用 `example` 模块中的 `app` 函数：

```toml title="pyproject.toml"
[project.gui-scripts]
hello = "example:app"
```

### 插件入口点

项目可以在 `pyproject.toml` 的
[`[project.entry-points]`](https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/#using-package-metadata)
表中定义插件发现的入口点。

例如，将 `example-plugin-a` 包注册为 `example` 的插件：

```toml title="pyproject.toml"
[project.entry-points.'example.plugins']
a = "example_plugin_a"
```

然后，在 `example` 中，插件将通过以下方式加载：

```python title="example/__init__.py"
from importlib.metadata import entry_points

for plugin in entry_points(group='example.plugins'):
    plugin.load()
```

!!! note

    `group` 键可以是任意值，不需要包含包名或“plugins”。但是，建议使用包名作为命名空间以避免与其他包冲突。

## 构建系统

构建系统决定了项目应如何打包和安装。项目可以在 `pyproject.toml` 的 `[build-system]` 表中声明和配置构建系统。

uv 使用构建系统的存在来确定项目是否包含应安装在项目虚拟环境中的包。如果未定义构建系统，uv 将不会尝试构建或安装项目本身，只会安装其依赖项。如果定义了构建系统，uv 将构建并将项目安装到项目环境中。

`uv init` 可以接受 `--build-backend` 选项来创建具有适当布局的打包项目。`uv init` 可以接受 `--package` 选项来创建具有默认构建系统的打包项目。

!!! note

    虽然 uv 在没有构建系统定义的情况下不会构建和安装当前项目，但在其他包中并不需要 `[build-system]` 表。由于历史原因，如果未定义构建系统，则使用 `setuptools.build_meta:__legacy__` 来构建包。您依赖的包可能没有明确声明其构建系统，但仍然可以安装。同样，如果您添加对本地包的依赖或使用 `uv pip` 安装它，uv 将始终尝试构建并安装它。

### 构建系统选项

构建系统用于支持以下功能：

- 包含或排除分发文件
- 可编辑安装行为
- 动态项目元数据
- 本地代码编译
- 共享库的打包

要配置这些功能，请参阅所选构建系统的文档。

## 项目打包

如[构建系统](#build-systems)中所述，Python 项目必须构建后才能安装。此过程通常称为“打包”。

如果您想执行以下操作，可能需要一个包：

- 向项目添加命令
- 将项目分发给他人
- 使用 `src` 和 `test` 布局
- 编写库

如果您执行以下操作，则可能_不需要_包：

- 编写脚本
- 构建简单应用程序
- 使用扁平布局

虽然 uv 通常使用[构建系统](#build-systems)的声明来确定是否应打包项目，但 uv 也允许使用
[`tool.uv.package`](../../reference/settings.md#package) 设置覆盖此行为。

设置 `tool.uv.package = true` 将强制构建项目并将其安装到项目环境中。如果未定义构建系统，uv 将使用 setuptools 遗留后端。

设置 `tool.uv.package = false` 将强制不构建项目包并将其安装到项目环境中。uv 在与项目交互时将忽略已声明的构建系统。

## 项目环境路径

`UV_PROJECT_ENVIRONMENT` 环境变量可用于配置项目虚拟环境路径（默认为 `.venv`）。

如果提供相对路径，它将相对于工作区根目录解析。如果提供绝对路径，它将按原样使用，即不会为环境创建子目录。如果提供的路径上不存在环境，uv 将创建它。

此选项可用于写入系统 Python 环境，但不建议这样做。`uv sync` 默认会从环境中删除多余的包，因此可能会使系统处于损坏状态。

要定位系统环境，请将 `UV_PROJECT_ENVIRONMENT` 设置为 Python 安装的前缀。例如，在基于 Debian 的系统上，这通常是 `/usr/local`：

```console
$ python -c "import sysconfig; print(sysconfig.get_config_var('prefix'))"
/usr/local
```

要定位此环境，您可以导出 `UV_PROJECT_ENVIRONMENT=/usr/local`。

!!! important

    如果提供了绝对路径并且该设置用于多个项目，则环境将被每个项目中的调用覆盖。此设置仅建议在 CI 或 Docker 镜像中用于单个项目。

!!! note

    默认情况下，uv 在项目操作期间不会读取 `VIRTUAL_ENV` 环境变量。如果 `VIRTUAL_ENV` 设置为与项目环境不同的路径，将显示警告。可以使用 `--active` 标志选择尊重 `VIRTUAL_ENV`。可以使用 `--no-active` 标志来静默警告。

## 有限解析环境

如果您的项目支持更有限的平台或 Python 版本集，您可以通过 `environments` 设置来约束解析的平台集，该设置接受 PEP 508 环境标记列表。例如，将 lockfile 约束为 macOS 和 Linux，并排除 Windows：

```toml title="pyproject.toml"
[tool.uv]
environments = [
    "sys_platform == 'darwin'",
    "sys_platform == 'linux'",
]
```

或者，排除替代 Python 实现：

```toml title="pyproject.toml"
[tool.uv]
environments = [
    "implementation_name == 'cpython'"
]
```

`environments` 设置中的条目必须是不相交的（即它们不能重叠）。例如，`sys_platform == 'darwin'` 和 `sys_platform == 'linux'` 是不相交的，但 `sys_platform == 'darwin'` 和 `python_version >= '3.9'` 不是，因为它们可能同时为真。

## 构建隔离

默认情况下，uv 在隔离的虚拟环境中构建所有包，如 [PEP 517](https://peps.python.org/pep-0517/) 所述。某些包与构建隔离不兼容，无论是故意的（例如，由于使用了繁重的构建依赖项，最常见的是 PyTorch）还是无意的（例如，由于使用了遗留的打包设置）。

要为特定依赖项禁用构建隔离，请将其添加到 `pyproject.toml` 中的 `no-build-isolation-package` 列表：

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["cchardet"]

[tool.uv]
no-build-isolation-package = ["cchardet"]
```

在没有构建隔离的情况下安装包需要将包的构建依赖项在安装包本身之前安装在项目环境中。这可以通过将构建依赖项和需要它们的包分离到不同的 extras 中来实现。例如：

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = []

[project.optional-dependencies]
build = ["setuptools", "cython"]
compile = ["cchardet"]

[tool.uv]
no-build-isolation-package = ["cchardet"]
```

根据上述内容，用户将首先同步 `build` 依赖项：

```console
$ uv sync --extra build
 + cython==3.0.11
 + foo==0.1.0 (from file:///Users/crmarsh/workspace/uv/foo)
 + setuptools==73.0.1
```

然后是 `compile` 依赖项：

```console
$ uv sync --extra compile
 + cchardet==2.1.7
 - cython==3.0.11
 - setuptools==73.0.1
```

请注意，`uv sync --extra compile` 默认会卸载 `cython` 和 `setuptools` 包。要保留构建依赖项，请在第二次 `uv sync` 调用中包含两个 extras：

```console
$ uv sync --extra build
$ uv sync --extra build --extra compile
```

某些包，如上面的 `cchardet`，仅在 `uv sync` 的_安装_阶段需要构建依赖项。其他包，如 `flash-attn`，即使在_解析_阶段解析项目的 lockfile 时也需要其构建依赖项。

在这种情况下，必须在使用任何 `uv lock` 或 `uv sync` 命令之前使用较低级别的 `uv pip` API 安装构建依赖项。例如，给定：

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["flash-attn"]

[tool.uv]
no-build-isolation-package = ["flash-attn"]
```

您可以运行以下命令序列来同步 `flash-attn`：

```console
$ uv venv
$ uv pip install torch setuptools
$ uv sync
```

或者，您可以通过
[`dependency-metadata`](../../reference/settings.md#dependency-metadata) 设置提前提供 `flash-attn` 元数据，从而在依赖解析阶段无需构建包。例如，要提前提供 `flash-attn` 元数据，请在 `pyproject.toml` 中包含以下内容：

```toml title="pyproject.toml"
[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

!!! tip

    要确定像 `flash-attn` 这样的包的元数据，请导航到适当的 Git 仓库，或在 [PyPI](https://pypi.org/project/flash-attn) 上查找并下载包的源分发版。包需求通常可以在 `setup.py` 或 `setup.cfg` 文件中找到。

    （如果包包含构建分发版，您可以解压缩它以找到 `METADATA` 文件；但是，构建分发版的存在将消除提前提供元数据的需要，因为它已经对 uv 可用。）

一旦包含，您可以再次使用两步 `uv sync` 过程来安装构建依赖项。给定以下 `pyproject.toml`：

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = []

[project.optional-dependencies]
build = ["torch", "setuptools", "packaging"]
compile = ["flash-attn"]

[tool.uv]
no-build-isolation-package = ["flash-attn"]

[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

您可以运行以下命令序列来同步 `flash-attn`：

```console
$ uv sync --extra build
$ uv sync --extra build --extra compile
```

!!! note

    `tool.uv.dependency-metadata` 中的 `version` 字段对于基于注册表的依赖项是可选的（如果省略，uv 将假定元数据适用于包的所有版本），但对于直接 URL 依赖项（如 Git 依赖项）是_必需的_。

## 可编辑模式

默认情况下，项目将以可编辑模式安装，以便源代码的更改立即反映在环境中。`uv sync` 和 `uv run` 都接受 `--no-editable` 标志，该标志指示 uv 以非可编辑模式安装项目。`--no-editable` 适用于部署用例，例如构建 Docker 容器，其中项目应包含在部署环境中而不依赖于原始源代码。

## 冲突的依赖项

uv 要求项目声明的所有可选依赖项（“extras”）彼此兼容，并在创建 lockfile 时一起解析所有可选依赖项。

如果一个 extra 中声明的可选依赖项与另一个 extra 中的依赖项不兼容，uv 将无法解析项目的要求并返回错误。

为了解决这个问题，uv 支持声明冲突的 extras。例如，考虑两组相互冲突的可选依赖项：

```toml title="pyproject.toml"
[project.optional-dependencies]
extra1 = ["numpy==2.1.2"]
extra2 = ["numpy==2.0.0"]
```

如果您使用上述依赖项运行 `uv lock`，解析将失败：

```console
$ uv lock
  x No solution found when resolving dependencies:
  `-> Because myproject[extra2] depends on numpy==2.0.0 and myproject[extra1] depends on numpy==2.1.2, we can conclude that myproject[extra1] and
      myproject[extra2] are incompatible.
      And because your project requires myproject[extra1] and myproject[extra2], we can conclude that your projects's requirements are unsatisfiable.
```

但如果您指定 `extra1` 和 `extra2` 是冲突的，uv 将分别解析它们。在 `tool.uv` 部分中指定冲突：

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { extra = "extra1" },
      { extra = "extra2" },
    ],
]
```

现在，运行 `uv lock` 将成功。但请注意，您现在不能同时安装 `extra1` 和 `extra2`：

```console
$ uv sync --extra extra1 --extra extra2
Resolved 3 packages in 14ms
error: extra `extra1`, extra `extra2` are incompatible with the declared conflicts: {`myproject[extra1]`, `myproject[extra2]`}
```

此错误发生是因为同时安装 `extra1` 和 `extra2` 会导致在同一环境中安装两个不同版本的包。

上述处理冲突 extras 的策略也适用于依赖组：

```toml title="pyproject.toml"
[dependency-groups]
group1 = ["numpy==2.1.2"]
group2 = ["numpy==2.0.0"]

[tool.uv]
conflicts = [
    [
      { group = "group1" },
      { group = "group2" },
    ],
]
```

与冲突 extras 的唯一区别是您需要使用 `group` 而不是 `extra`。