# 使用工作区

受 [Cargo](https://doc.rust-lang.org/cargo/reference/workspaces.html) 同名概念的启发，工作区是“一个或多个包的集合，称为_工作区成员_，它们被一起管理。”

工作区通过将大型代码库拆分为具有共同依赖的多个包来组织代码。例如：一个基于 FastAPI 的 Web 应用程序，以及一系列作为独立 Python 包版本化和维护的库，所有这些都在同一个 Git 仓库中。

在工作区中，每个包都定义了自己的 `pyproject.toml`，但工作区共享一个锁文件，确保工作区使用一致的依赖集。

因此，`uv lock` 会一次性对整个工作区进行操作，而 `uv run` 和 `uv sync` 默认在工作区根目录下运行，尽管两者都接受 `--package` 参数，允许您从任何工作区目录在特定工作区成员中运行命令。

## 入门指南

要创建工作区，请在 `pyproject.toml` 中添加 `tool.uv.workspace` 表，这将隐式地在该包下创建一个工作区。

!!! tip

    默认情况下，在现有包中运行 `uv init` 会将新创建的成员添加到工作区中，如果工作区根目录中尚不存在 `tool.uv.workspace` 表，则会创建它。

在定义工作区时，您必须指定 `members`（必需）和 `exclude`（可选）键，它们分别指示工作区包含或排除特定目录作为成员，并接受 glob 列表：

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]
exclude = ["packages/seeds"]
```

由 `members` glob 包含（且未被 `exclude` glob 排除）的每个目录都必须包含一个 `pyproject.toml` 文件。然而，工作区成员可以是[应用程序](./init.md#applications)或[库](./init.md#libraries)；两者都在工作区上下文中受支持。

每个工作区都需要一个根目录，它_也_是一个工作区成员。在上面的示例中，`albatross` 是工作区根目录，工作区成员包括 `packages` 目录下的所有项目，但 `seeds` 除外。

默认情况下，`uv run` 和 `uv sync` 在工作区根目录下运行。例如，在上面的示例中，`uv run` 和 `uv run --package albatross` 是等价的，而 `uv run --package bird-feeder` 将在 `bird-feeder` 包中运行命令。

## 工作区源

在工作区内，通过 [`tool.uv.sources`](./dependencies.md) 实现工作区成员之间的依赖关系，例如：

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

在此示例中，`albatross` 项目依赖于 `bird-feeder` 项目，后者是工作区的成员。`tool.uv.sources` 表中的 `workspace = true` 键值对表示 `bird-feeder` 依赖应由工作区提供，而不是从 PyPI 或其他注册表获取。

!!! note

    工作区成员之间的依赖是可编辑的。

工作区根目录中的任何 `tool.uv.sources` 定义都适用于所有成员，除非在特定成员的 `tool.uv.sources` 中被覆盖。例如，给定以下 `pyproject.toml`：

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }
tqdm = { git = "https://github.com/tqdm/tqdm" }

[tool.uv.workspace]
members = ["packages/*"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

默认情况下，每个工作区成员将从 GitHub 安装 `tqdm`，除非特定成员在其自己的 `tool.uv.sources` 表中覆盖 `tqdm` 条目。

## 工作区布局

最常见的工作区布局可以被视为一个根项目及其伴随的一系列库。

例如，继续上面的示例，此工作区在 `albatross` 处有一个显式根目录，`packages` 目录中有两个库（`bird-feeder` 和 `seeds`）：

```text
albatross
├── packages
│   ├── bird-feeder
│   │   ├── pyproject.toml
│   │   └── src
│   │       └── bird_feeder
│   │           ├── __init__.py
│   │           └── foo.py
│   └── seeds
│       ├── pyproject.toml
│       └── src
│           └── seeds
│               ├── __init__.py
│               └── bar.py
├── pyproject.toml
├── README.md
├── uv.lock
└── src
    └── albatross
        └── main.py
```

由于 `seeds` 在 `pyproject.toml` 中被排除，工作区共有两个成员：`albatross`（根目录）和 `bird-feeder`。

## 何时（不）使用工作区

工作区旨在促进单个仓库中多个相互关联的包的开发。随着代码库复杂性的增加，将其拆分为更小、可组合的包，每个包都有自己的依赖和版本约束，可能会有所帮助。

工作区有助于强制隔离和关注点分离。例如，在 uv 中，我们有核心库和命令行接口的独立包，使我们能够独立于 CLI 测试核心库，反之亦然。

工作区的其他常见用例包括：

- 一个库，其性能关键的子例程在扩展模块（Rust、C++ 等）中实现。
- 一个具有插件系统的库，其中每个插件都是一个独立的工作区包，依赖于根目录。

工作区_不_适用于成员有冲突需求或希望为每个成员使用单独虚拟环境的情况。在这种情况下，路径依赖通常是更可取的。例如，与其将 `albatross` 及其成员分组到一个工作区中，您始终可以将每个包定义为其独立的项目，并在 `tool.uv.sources` 中定义包间依赖为路径依赖：

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { path = "packages/bird-feeder" }

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

这种方法传达了许多相同的优点，但允许更细粒度地控制依赖解析和虚拟环境管理（缺点是 `uv run --package` 不再可用；相反，必须从相关包目录运行命令）。

最后，uv 的工作区强制整个工作区使用单一的 `requires-python`，取所有成员的 `requires-python` 值的交集。如果您需要支持在不受工作区其他成员支持的 Python 版本上测试某个成员，则可能需要使用 `uv pip` 将该成员安装到单独的虚拟环境中。

!!! note

    由于 Python 不提供依赖隔离，uv 无法确保包使用其声明的依赖而不使用其他内容。对于工作区，uv 无法确保包不会导入另一个工作区成员声明的依赖。