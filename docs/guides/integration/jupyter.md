---
title: Using uv with Jupyter
description:
  A complete guide to using uv with Jupyter notebooks for interactive computing, data analysis, and
  visualization, including kernel management and virtual environment integration.
---

# 使用 uv 与 Jupyter
[Jupyter](https://jupyter.org/) 笔记本是一个用于交互式计算、数据分析和可视化的流行工具。你可以通过几种方式将 Jupyter 与 uv 结合使用，无论是与项目交互，还是作为独立工具。

## 在项目中使用 Jupyter
如果你正在一个 [项目](../../concepts/projects/index.md) 中工作，你可以通过以下命令启动一个 Jupyter 服务器，并访问项目的虚拟环境：
```console
$ uv run --with jupyter jupyter lab
```
默认情况下，`jupyter lab` 会在 [http://localhost:8888/lab](http://localhost:8888/lab) 启动服务器。

在笔记本中，你可以像在项目中的其他文件中一样导入项目的模块。例如，如果你的项目依赖 `requests`，那么 `import requests` 将从项目的虚拟环境中导入 `requests`。

如果你只需要对项目的虚拟环境进行只读访问，那么不需要额外操作。然而，如果你需要从笔记本中安装额外的包，则有一些额外的细节需要考虑。

### 创建内核
如果你需要从笔记本中安装包，我们建议为你的项目创建一个专用的内核。内核使 Jupyter 服务器能够在一个环境中运行，而各个笔记本则在它们自己的独立环境中运行。

在 uv 的上下文中，我们可以在安装 Jupyter 的同时为项目创建一个内核，例如 `uv run --with jupyter jupyter lab`。为项目创建内核可以确保笔记本连接到正确的环境，并且从笔记本中安装的任何包都会安装到项目的虚拟环境中。

要创建内核，你需要将 `ipykernel` 作为开发依赖安装：
```console
$ uv add --dev ipykernel
```
然后，你可以为 `project` 创建内核：
```console
$ uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name=project
```
接下来，启动服务器：
```console
$ uv run --with jupyter jupyter lab
```
在创建笔记本时，从下拉菜单中选择 `project` 内核。然后使用 `!uv add pydantic` 将 `pydantic` 添加到项目的依赖中，或者使用 `!uv pip install pydantic` 将 `pydantic` 安装到项目的虚拟环境中，而不会将更改持久化到项目的 `pyproject.toml` 或 `uv.lock` 文件中。这两个命令都会使 `import pydantic` 在笔记本中正常工作。

### 在没有内核的情况下安装包
如果你不想创建内核，你仍然可以从笔记本中安装包。然而，有一些注意事项需要考虑。

尽管 `uv run --with jupyter` 在隔离的环境中运行，但在笔记本内部，`!uv add` 和相关命令将修改 _项目的_ 环境，即使没有内核。例如，在笔记本中运行 `!uv add pydantic` 会将 `pydantic` 添加到项目的依赖和虚拟环境中，这样 `import pydantic` 将立即生效，无需进一步配置或重启服务器。

然而，由于 Jupyter 服务器是“活动”环境，`!uv pip install` 会将包安装到 _Jupyter_ 的环境中，而不是项目环境。这些依赖项将在 Jupyter 服务器的生命周期内持续存在，但在后续的 `jupyter` 调用中可能会消失。

如果你正在使用依赖 pip 的笔记本（例如通过 `%pip` 魔术命令），你可以在启动 Jupyter 服务器之前通过运行 `uv venv --seed` 将 pip 包含在项目的虚拟环境中。例如：
```console
$ uv venv --seed
$ uv run --with jupyter jupyter lab
```
随后在笔记本中调用 `%pip install` 会将包安装到项目的虚拟环境中。然而，这些修改不会反映在项目的 `pyproject.toml` 或 `uv.lock` 文件中。

## 将 Jupyter 作为独立工具使用
如果你需要临时访问笔记本（例如交互式运行 Python 代码片段），你可以随时使用 `uv tool run jupyter lab` 启动 Jupyter 服务器。这将在隔离的环境中运行 Jupyter 服务器。

## 在非项目环境中使用 Jupyter
如果你需要在与 [项目](../../concepts/projects/index.md) 无关的虚拟环境中运行 Jupyter（例如没有 `pyproject.toml` 或 `uv.lock`），你可以通过直接将 Jupyter 添加到环境中来实现。例如：
=== "macOS 和 Linux"
```console
$ uv venv --seed
$ uv pip install pydantic
$ uv pip install jupyterlab
$ .venv/bin/jupyter lab
```
=== "Windows"
```powershell
uv venv --seed
uv pip install pydantic
uv pip install jupyterlab
.venv\Scripts\jupyter lab
```
在这里，`import pydantic` 将在笔记本中生效，你可以通过 `!uv pip install` 甚至 `!pip install` 安装额外的包。

## 在 VS Code 中使用 Jupyter
你也可以在 VS Code 等编辑器中与 Jupyter 笔记本交互。要将 uv 管理的项目连接到 VS Code 中的 Jupyter 笔记本，我们建议为项目创建一个内核，如下所示：
```console
# 创建一个项目。
$ uv init project
# 进入项目目录。
$ cd project
# 将 ipykernel 作为开发依赖添加。
$ uv add --dev ipykernel
# 在 VS Code 中打开项目。
$ code .
```
一旦项目目录在 VS Code 中打开，你可以通过从命令面板中选择“创建：新建 Jupyter 笔记本”来创建一个新的 Jupyter 笔记本。当提示选择内核时，选择“Python 环境”并选择你之前创建的虚拟环境（例如，macOS 和 Linux 上的 `.venv/bin/python`，或 Windows 上的 `.venv\Scripts\python`）。

!!! note
VS Code 要求 `ipykernel` 存在于项目环境中。如果你不希望将 `ipykernel` 作为开发依赖添加，你可以直接使用 `uv pip install ipykernel` 将其安装到项目环境中。

如果你需要从笔记本中操作项目的环境，你可能需要将 `uv` 作为显式开发依赖添加：
```console
$ uv add --dev uv
```
然后，你可以使用 `!uv add pydantic` 将 `pydantic` 添加到项目的依赖中，或者使用 `!uv pip install pydantic` 将 `pydantic` 安装到项目的虚拟环境中，而不会更新项目的 `pyproject.toml` 或 `uv.lock` 文件。