# 创建项目

uv 支持使用 `uv init` 创建项目。

在创建项目时，uv 支持两种基本模板：[**应用程序**](#applications) 和 [**库**](#libraries)。默认情况下，uv 会创建一个应用程序项目。可以使用 `--lib` 标志来创建一个库项目。

## 目标目录

uv 会在当前工作目录中创建项目，或者通过提供名称在目标目录中创建项目，例如 `uv init foo`。如果目标目录中已经存在项目（即存在 `pyproject.toml`），uv 将退出并报错。

## 应用程序

应用程序项目适用于 Web 服务器、脚本和命令行界面。

应用程序是 `uv init` 的默认目标，但也可以使用 `--app` 标志指定。

```console
$ uv init example-app
```

项目包括一个 `pyproject.toml`、一个示例文件（`hello.py`）、一个 README 文件和一个 Python 版本锁定文件（`.python-version`）。

```console
$ tree example-app
example-app
├── .python-version
├── README.md
├── hello.py
└── pyproject.toml
```

`pyproject.toml` 包含基本元数据。它不包含构建系统，不是一个[包](./config.md#project-packaging)，也不会安装到环境中：

```toml title="pyproject.toml"
[project]
name = "example-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []
```

示例文件定义了一个带有标准样板代码的 `main` 函数：

```python title="hello.py"
def main():
    print("Hello from example-app!")


if __name__ == "__main__":
    main()
```

可以使用 `uv run` 执行 Python 文件：

```console
$ uv run hello.py
Hello from example-project!
```

## 打包的应用程序

许多用例需要一个[包](./config.md#project-packaging)。例如，如果您正在创建一个将发布到 PyPI 的命令行界面，或者您希望在专用目录中定义测试。

可以使用 `--package` 标志创建一个打包的应用程序：

```console
$ uv init --package example-pkg
```

源代码被移动到 `src` 目录中，包含一个模块目录和一个 `__init__.py` 文件：

```console
$ tree example-pkg
example-pkg
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── example_packaged_app
        └── __init__.py
```

定义了[构建系统](./config.md#build-systems)，因此项目将被安装到环境中：

```toml title="pyproject.toml" hl_lines="12-14"
[project]
name = "example-pkg"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
example-pkg = "example_packaged_app:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

!!! tip

    可以使用 `--build-backend` 选项请求替代的构建系统。

包含一个[命令](./config.md#entry-points)定义：

```toml title="pyproject.toml" hl_lines="9 10"
[project]
name = "example-pkg"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
example-pkg = "example_packaged_app:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

可以使用 `uv run` 执行命令：

```console
$ uv run --directory example-pkg example-pkg
Hello from example-pkg!
```

## 库

库为其他项目提供函数和对象。库旨在被构建和分发，例如通过上传到 PyPI。

可以使用 `--lib` 标志创建库：

```console
$ uv init --lib example-lib
```

!!! note

    使用 `--lib` 意味着 `--package`。库始终需要一个打包的项目。

与[打包的应用程序](#packaged-applications)一样，使用 `src` 布局。包含一个 `py.typed` 标记，以向消费者表明可以从库中读取类型：

```console
$ tree example-lib
example-lib
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── example_lib
        ├── py.typed
        └── __init__.py
```

!!! note

    `src` 布局在开发库时特别有价值。它确保库与项目根目录中的任何 `python` 调用隔离，并且分发的库代码与项目源代码的其他部分很好地分离。

定义了[构建系统](./config.md#build-systems)，因此项目将被安装到环境中：

```toml title="pyproject.toml" hl_lines="12-14"
[project]
name = "example-lib"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

!!! tip

    您可以使用 `--build-backend` 选择不同的构建后端模板，如 `hatchling`、`flit-core`、`pdm-backend`、`setuptools`、`maturin` 或 `scikit-build-core`。如果您想创建[带有扩展模块的库](#projects-with-extension-modules)，则需要使用替代的后端。

创建的模块定义了一个简单的 API 函数：

```python title="__init__.py"
def hello() -> str:
    return "Hello from example-lib!"
```

可以使用 `uv run` 导入并执行它：

```console
$ uv run --directory example-lib python -c "import example_lib; print(example_lib.hello())"
Hello from example-lib!
```

## 带有扩展模块的项目

大多数 Python 项目是“纯 Python”的，这意味着它们没有定义其他语言（如 C、C++、FORTRAN 或 Rust）的模块。然而，带有扩展模块的项目通常用于性能敏感的代码。

创建带有扩展模块的项目需要选择替代的构建系统。uv 支持使用以下构建系统创建带有扩展模块的项目：

- [`maturin`](https://www.maturin.rs) 用于带有 Rust 的项目
- [`scikit-build-core`](https://github.com/scikit-build/scikit-build-core) 用于带有 C、C++、FORTRAN、Cython 的项目

使用 `--build-backend` 标志指定构建系统：

```console
$ uv init --build-backend maturin example-ext
```

!!! note

    使用 `--build-backend` 意味着 `--package`。

项目包含一个 `Cargo.toml` 和一个 `lib.rs` 文件，以及典型的 Python 项目文件：

```console
$ tree example-ext
example-ext
├── .python-version
├── Cargo.toml
├── README.md
├── pyproject.toml
└── src
    ├── lib.rs
    └── example_ext
        ├── __init__.py
        └── _core.pyi
```

!!! note

    如果使用 `scikit-build-core`，您将看到 CMake 配置和一个 `main.cpp` 文件。

Rust 库定义了一个简单的函数：

```rust title="src/lib.rs"
use pyo3::prelude::*;

#[pyfunction]
fn hello_from_bin() -> String {
    "Hello from example-ext!".to_string()
}

#[pymodule]
fn _core(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(hello_from_bin, m)?)?;
    Ok(())
}
```

Python 模块导入它：

```python title="src/example_ext/__init__.py"
from example_ext._core import hello_from_bin


def main() -> None:
    print(hello_from_bin())
```

可以使用 `uv run` 执行命令：

```console
$ uv run --directory example-ext example-ext
Hello from example-ext!
```

!!! important

    对 `lib.rs` 或 `main.cpp` 中的扩展代码的更改需要运行 `--reinstall` 来重新构建它们。

## 创建最小化项目

如果只想创建一个 `pyproject.toml`，请使用 `--bare` 选项：

```console
$ uv init example --bare
```

uv 将跳过创建 Python 版本锁定文件、README 文件以及任何源目录或文件。此外，uv 不会初始化版本控制系统（即 `git`）。

```console
$ tree example-bare
example-bare
└── pyproject.toml
```

uv 也不会向 `pyproject.toml` 添加额外的元数据，如 `description` 或 `authors`。

```toml
[project]
name = "example"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []
```

`--bare` 选项可以与其他选项（如 `--lib` 或 `--build-backend`）一起使用——在这些情况下，uv 仍会配置构建系统，但不会创建预期的文件结构。

当使用 `--bare` 时，仍然可以选择使用其他功能：

```console
$ uv init example --bare --description "Hello world" --author-from git --vcs git --python-pin
```