# 锁定与同步

### 创建锁文件

锁文件在使用项目环境的 uv 调用期间创建和更新，例如 `uv sync` 和 `uv run`。锁文件也可以使用 `uv lock` 显式创建或更新：

```console
$ uv lock
```

### 导出锁文件

如果需要将 uv 与其他工具或工作流集成，可以使用 `uv export --format requirements-txt` 将 `uv.lock` 导出为 `requirements.txt` 格式。生成的 `requirements.txt` 文件可以通过 `uv pip install` 或其他工具如 `pip` 进行安装。

通常，我们建议不要同时使用 `uv.lock` 和 `requirements.txt` 文件。如果你发现自己需要导出 `uv.lock` 文件，请考虑提交 issue 讨论你的使用场景。

### 检查锁文件是否最新

为了避免在 `uv sync` 和 `uv run` 调用期间更新锁文件，可以使用 `--frozen` 标志。

为了避免在 `uv run` 调用期间更新环境，可以使用 `--no-sync` 标志。

要断言锁文件与项目元数据匹配，请使用 `--locked` 标志。如果锁文件不是最新的，将引发错误而不是更新锁文件。

你也可以通过向 `uv lock` 传递 `--check` 标志来检查锁文件是否最新：

```console
$ uv lock --check
```

这与其他命令中的 `--locked` 标志等效。

### 升级锁定的包版本

默认情况下，uv 在运行 `uv sync` 和 `uv lock` 时会优先使用锁定的包版本（如果存在 `uv.lock` 文件）。只有当项目的依赖约束排除了之前锁定的版本时，包版本才会发生变化。

要升级所有包：

```console
$ uv lock --upgrade
```

要将单个包升级到最新版本，同时保留所有其他包的锁定版本：

```console
$ uv lock --upgrade-package <package>
```

要将单个包升级到特定版本：

```console
$ uv lock --upgrade-package <package>==<version>
```

在所有情况下，升级都受限于项目的依赖约束。例如，如果项目为某个包定义了上限，则升级不会超过该版本。

!!! note

    uv 对 Git 依赖项应用了类似的逻辑。例如，如果 Git 依赖项引用了 `main` 分支，uv 会优先使用现有 `uv.lock` 文件中的锁定提交 SHA，而不是 `main` 分支上的最新提交，除非使用了 `--upgrade` 或 `--upgrade-package` 标志。