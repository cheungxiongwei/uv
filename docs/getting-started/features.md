# 功能

uv 提供了 Python 开发所需的核心功能 —— 从安装 Python 和编写简单脚本，到支持多 Python 版本和多平台的大型项目开发。

uv 的接口可以分为多个部分，这些部分可以独立使用，也可以组合使用。

## Python 版本

安装和管理 Python 本身。

- `uv python install`: 安装 Python 版本。
- `uv python list`: 查看可用的 Python 版本。
- `uv python find`: 查找已安装的 Python 版本。
- `uv python pin`: 将当前项目固定使用特定的 Python 版本。
- `uv python uninstall`: 卸载 Python 版本。

请参阅 [安装 Python 指南](../guides/install-python.md) 以开始使用。

## 脚本

执行独立的 Python 脚本，例如 `example.py`。

- `uv run`: 运行脚本。
- `uv add --script`: 为脚本添加依赖。
- `uv remove --script`: 从脚本中移除依赖。

请参阅 [运行脚本指南](../guides/scripts.md) 以开始使用。

## 项目

创建和开发 Python 项目，即包含 `pyproject.toml` 的项目。

- `uv init`: 创建新的 Python 项目。
- `uv add`: 为项目添加依赖。
- `uv remove`: 从项目中移除依赖。
- `uv sync`: 同步项目的依赖与环境。
- `uv lock`: 为项目的依赖创建锁文件。
- `uv run`: 在项目环境中运行命令。
- `uv tree`: 查看项目的依赖树。
- `uv build`: 将项目构建为分发存档。
- `uv publish`: 将项目发布到包索引。

请参阅 [项目指南](../guides/projects.md) 以开始使用。

## 工具

运行和安装发布到 Python 包索引的工具，例如 `ruff` 或 `black`。

- `uvx` / `uv tool run`: 在临时环境中运行工具。
- `uv tool install`: 全局安装工具。
- `uv tool uninstall`: 卸载工具。
- `uv tool list`: 列出已安装的工具。
- `uv tool update-shell`: 更新 shell 以包含工具可执行文件。

请参阅 [工具指南](../guides/tools.md) 以开始使用。

## pip 接口

手动管理环境和包 —— 适用于遗留工作流或高级命令无法提供足够控制的情况。

创建虚拟环境（替代 `venv` 和 `virtualenv`）：

- `uv venv`: 创建新的虚拟环境。

请参阅 [使用环境文档](../pip/environments.md) 以获取详细信息。

管理环境中的包（替代 [`pip`](https://github.com/pypa/pip) 和 [`pipdeptree`](https://github.com/tox-dev/pipdeptree)）：

- `uv pip install`: 在当前环境中安装包。
- `uv pip show`: 显示已安装包的详细信息。
- `uv pip freeze`: 列出已安装包及其版本。
- `uv pip check`: 检查当前环境中的包是否兼容。
- `uv pip list`: 列出已安装的包。
- `uv pip uninstall`: 卸载包。
- `uv pip tree`: 查看环境的依赖树。

请参阅 [管理包文档](../pip/packages.md) 以获取详细信息。

锁定环境中的包（替代 [`pip-tools`](https://github.com/jazzband/pip-tools)）：

- `uv pip compile`: 将需求编译为锁文件。
- `uv pip sync`: 使用锁文件同步环境。

请参阅 [锁定环境文档](../pip/compile.md) 以获取详细信息。

!!! important

    这些命令并不完全实现它们所基于工具的接口和行为。偏离常见工作流越远，越可能遇到差异。请参阅 [pip 兼容性指南](../pip/compatibility.md) 以获取详细信息。

## 实用工具

管理和检查 uv 的状态，例如缓存、存储目录或执行自我更新：

- `uv cache clean`: 清除缓存条目。
- `uv cache prune`: 清除过期的缓存条目。
- `uv cache dir`: 显示 uv 缓存目录路径。
- `uv tool dir`: 显示 uv 工具目录路径。
- `uv python dir`: 显示 uv 安装的 Python 版本路径。
- `uv self update`: 将 uv 更新到最新版本。

## 下一步

阅读 [指南](../guides/index.md) 以了解每个功能的介绍，查看 [概念](../concepts/index.md) 页面以深入了解 uv 的功能，或学习如何 [获取帮助](./help.md) 以解决遇到的问题。
