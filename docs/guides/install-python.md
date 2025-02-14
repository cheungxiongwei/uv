---
title: Installing and managing Python
description:
  A guide to using uv to install Python, including requesting specific versions, automatic
  installation, viewing installed versions, and more.
---

# 安装 Python

如果系统上已经安装了 Python，uv 将会[检测并使用](#using-existing-python-versions)它，无需额外配置。然而，uv 也可以安装和管理 Python 版本。uv 会在需要时[自动安装](#automatic-python-downloads)缺失的 Python 版本 —— 你不需要预先安装 Python 即可开始使用。

## 开始使用

要安装最新的 Python 版本：

```console
$ uv python install
```

!!! note

    Python 官方并未发布可分发二进制文件。因此，uv 使用了 Astral [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) 项目的发行版。更多详情请参阅 [Python 发行版](../concepts/python-versions.md#managed-python-distributions)文档。

安装完成后，uv 命令将自动使用该 Python 版本。

!!! important

    当 Python 由 uv 安装时，它不会全局可用（即无法通过 `python` 命令调用）。此功能目前处于 _预览_ 阶段。详情请参阅 [安装 Python 可执行文件](../concepts/python-versions.md#installing-python-executables)。

    你仍然可以使用
    [`uv run`](../guides/scripts.md#using-different-python-versions) 或
    [创建并激活虚拟环境](../pip/environments.md) 来直接使用 `python`。

## 安装特定版本

要安装特定的 Python 版本：

```console
$ uv python install 3.12
```

要安装多个 Python 版本：

```console
$ uv python install 3.11 3.12
```

要安装替代的 Python 实现，例如 PyPy：

```console
$ uv python install pypy@3.10
```

更多详情请参阅 [`python install`](../concepts/python-versions.md#installing-a-python-version) 文档。

## 重新安装 Python

要重新安装由 uv 管理的 Python 版本，使用 `--reinstall` 选项，例如：

```console
$ uv python install --reinstall
```

这将重新安装所有之前安装的 Python 版本。Python 发行版不断改进，因此重新安装可能会修复某些问题，即使 Python 版本没有变化。

## 查看 Python 安装

要查看可用和已安装的 Python 版本：

```console
$ uv python list
```

更多详情请参阅 [`python list`](../concepts/python-versions.md#viewing-available-python-versions) 文档。

## 自动下载 Python

使用 uv 时，无需显式安装 Python。默认情况下，uv 会在需要时自动下载 Python 版本。例如，以下命令会在 Python 3.12 未安装时自动下载它：

```console
$ uvx python@3.12 -c "print('hello world')"
```

即使没有指定特定的 Python 版本，uv 也会在需要时下载最新版本。例如，如果系统上没有安装任何 Python 版本，以下命令会在创建新虚拟环境之前安装 Python：

```console
$ uv venv
```

!!! tip

    如果你希望更好地控制 Python 的下载时机，可以[轻松禁用](../concepts/python-versions.md#disabling-automatic-python-downloads)自动下载功能。

<!-- TODO(zanieb): Restore when Python shim management is added
注意，当自动安装 Python 时，`python` 命令不会添加到 shell 中。使用 `uv python install-shim` 来确保 `python` shim 已安装。
-->

## 使用现有的 Python 版本

如果系统上已经安装了 Python，uv 将会使用它。此行为无需配置：如果系统 Python 满足命令调用的要求，uv 将使用它。详情请参阅 [Python 发现](../concepts/python-versions.md#discovery-of-python-versions) 文档。

要强制 uv 使用系统 Python，请提供 `--python-preference only-system` 选项。详情请参阅 [Python 版本偏好](../concepts/python-versions.md#adjusting-python-version-preferences) 文档。

## 下一步

要了解更多关于 `uv python` 的信息，请参阅 [Python 版本概念](../concepts/python-versions.md) 页面和 [命令参考](../reference/cli.md#uv-python)。

或者，继续阅读以了解如何 [运行脚本](./scripts.md) 并使用 uv 调用 Python。
