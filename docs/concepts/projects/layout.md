# 项目结构与文件

## `pyproject.toml`

Python 项目的元数据定义在 [`pyproject.toml`](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) 文件中。uv 需要此文件来识别项目的根目录。

!!! tip

    可以使用 `uv init` 创建新项目。详情请参阅 [创建项目](./init.md)。

一个最小的项目定义包括名称和版本：

```toml title="pyproject.toml"
[project]
name = "example"
version = "0.1.0"
```

其他项目元数据和配置包括：

- [Python 版本要求](./config.md#python-version-requirement)
- [依赖项](./dependencies.md)
- [构建系统](./config.md#build-systems)
- [入口点（命令）](./config.md#entry-points)

## 项目环境

在使用 uv 处理项目时，uv 会根据需要创建虚拟环境。虽然某些 uv 命令会创建临时环境（例如 `uv run --isolated`），但 uv 还会在 `pyproject.toml` 旁边的 `.venv` 目录中管理一个持久化环境，其中包含项目及其依赖项。该环境存储在项目内部，以便编辑器可以轻松找到它——编辑器需要环境来提供代码补全和类型提示。不建议将 `.venv` 目录包含在版本控制中；它通过内部的 `.gitignore` 文件自动从 `git` 中排除。

要在项目环境中运行命令，请使用 `uv run`。或者，可以像普通虚拟环境一样激活项目环境。

当调用 `uv run` 时，如果项目环境不存在，它将创建项目环境；如果存在，它将确保环境是最新的。也可以使用 `uv sync` 显式创建项目环境。

不建议手动修改项目环境，例如使用 `uv pip install`。对于项目依赖项，请使用 `uv add` 将包添加到环境中。对于一次性需求，请使用 [`uvx`](../../guides/tools.md) 或 [`uv run --with`](./run.md#requesting-additional-dependencies)。

!!! tip

    如果不希望 uv 管理项目环境，可以设置 [`managed = false`](../../reference/settings.md#managed) 以禁用项目的自动锁定和同步。例如：

    ```toml title="pyproject.toml"
    [tool.uv]
    managed = false
    ```

## 锁文件

uv 会在 `pyproject.toml` 旁边创建一个 `uv.lock` 文件。

`uv.lock` 是一个 _通用_ 或 _跨平台_ 的锁文件，它捕获了在所有可能的 Python 标记（如操作系统、架构和 Python 版本）下将安装的包。

与用于指定项目广泛需求的 `pyproject.toml` 不同，锁文件包含在项目环境中安装的确切解析版本。此文件应检入版本控制，以便在不同机器上实现一致且可重复的安装。

锁文件确保项目开发人员使用一致的包版本集。此外，在将项目部署为应用程序时，它确保使用的确切包版本集是已知的。

锁文件在使用项目环境的 uv 调用期间创建和更新，即 `uv sync` 和 `uv run`。锁文件也可以使用 `uv lock` 显式更新。

`uv.lock` 是一个人类可读的 TOML 文件，但由 uv 管理，不应手动编辑。目前没有 Python 的锁文件标准，因此此文件的格式是 uv 特定的，不能被其他工具使用。