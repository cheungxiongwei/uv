---
title: Using uv with pre-commit
description:
  A guide to using uv with pre-commit to automatically update lock files, export requirements, and
  compile requirements files.
---

# 在 pre-commit 中使用 uv

官方提供了一个 pre-commit 钩子，位于 [`astral-sh/uv-pre-commit`](https://github.com/astral-sh/uv-pre-commit)。

为了确保即使通过 pre-commit 更改了 `pyproject.toml` 文件，`uv.lock` 文件也能保持最新，请将以下内容添加到 `.pre-commit-config.yaml` 中：

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv 版本。
    rev: 0.5.30
    hooks:
      - id: uv-lock
```

为了使用 pre-commit 保持 `requirements.txt` 文件更新：

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv 版本。
    rev: 0.5.30
    hooks:
      - id: uv-export
```

为了通过 pre-commit 编译 requirements，请将以下内容添加到 `.pre-commit-config.yaml` 中：

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv 版本。
    rev: 0.5.30
    hooks:
      # 编译 requirements
      - id: pip-compile
        args: [requirements.in, -o, requirements.txt]
```

要编译其他文件，请修改 `args` 和 `files`：

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv 版本。
    rev: 0.5.30
    hooks:
      # 编译 requirements
      - id: pip-compile
        args: [requirements-dev.in, -o, requirements-dev.txt]
        files: ^requirements-dev\.(in|txt)$
```

要同时运行多个文件的钩子：

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv 版本。
    rev: 0.5.30
    hooks:
      # 编译 requirements
      - id: pip-compile
        name: pip-compile requirements.in
        args: [requirements.in, -o, requirements.txt]
      - id: pip-compile
        name: pip-compile requirements-dev.in
        args: [requirements-dev.in, -o, requirements-dev.txt]
        files: ^requirements-dev\.(in|txt)$
```