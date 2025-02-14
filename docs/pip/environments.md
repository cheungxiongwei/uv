```md docs/pip/environments.md (1-152)
# Python 环境

每个 Python 安装都有一个环境，在使用 Python 时该环境处于激活状态。可以将包安装到环境中，以便在 Python 脚本中使用它们的模块。通常，最佳实践是不修改 Python 安装的环境。这对于操作系统自带的 Python 安装尤为重要，因为它们通常自己管理包。虚拟环境是一种轻量级的方式，用于将包与 Python 安装的环境隔离开来。与 `pip` 不同，uv 默认要求使用虚拟环境。

## 创建虚拟环境

uv 支持创建虚拟环境，例如，在 `.venv` 处创建虚拟环境：

```console
$ uv venv
```

可以指定特定的名称或路径，例如，在 `my-name` 处创建虚拟环境：

```console
$ uv venv my-name
```

可以请求特定的 Python 版本，例如，创建使用 Python 3.11 的虚拟环境：

```console
$ uv venv --python 3.11
```

请注意，这要求系统上已安装请求的 Python 版本。然而，如果不可用，uv 将为您下载 Python。有关更多详细信息，请参阅 [Python 版本](../concepts/python-versions.md) 文档。

## 使用虚拟环境

当使用默认的虚拟环境名称时，uv 将在后续调用中自动找到并使用该虚拟环境。

```console
$ uv venv

$ # 在新虚拟环境中安装包
$ uv pip install ruff
```

可以“激活”虚拟环境以使其包可用：

=== "macOS 和 Linux"

    ```console
    $ source .venv/bin/activate
    ```

=== "Windows"

    ```console
    $ .venv\Scripts\activate
    ```

!!! note

    Unix 上的默认激活脚本适用于 POSIX 兼容的 shell，如 `sh`、`bash` 或 `zsh`。
    还有其他适用于常见替代 shell 的激活脚本。

    === "fish"

        ```console
        $ source .venv/bin/activate.fish
        ```

    === "csh / tcsh"

        ```console
        $ source .venv/bin/activate.csh
        ```

    === "Nushell"

        ```console
        $ use .venv\Scripts\activate.nu
        ```

## 停用环境

要退出虚拟环境，请使用 `deactivate` 命令：

```console
$ deactivate
```

## 使用任意 Python 环境

由于 uv 不依赖于 Python，它可以安装到其自身之外的虚拟环境中。例如，设置 `VIRTUAL_ENV=/path/to/venv` 将导致 uv 安装到 `/path/to/venv`，无论 uv 安装在何处。请注意，如果 `VIRTUAL_ENV` 设置为一个 **不是** [PEP 405 兼容](https://peps.python.org/pep-0405/#specification) 的虚拟环境的目录，它将被忽略。

uv 还可以使用 `uv pip sync` 或 `uv pip install` 的 `--python` 参数安装到任意的、甚至非虚拟的环境中。例如，`uv pip install --python /path/to/python` 将安装到与 `/path/to/python` 解释器链接的环境中。

为了方便，`uv pip install --system` 将安装到系统 Python 环境中。使用 `--system` 大致相当于 `uv pip install --python $(which python)`，但请注意，链接到虚拟环境的可执行文件将被跳过。虽然我们通常建议使用虚拟环境进行依赖管理，但 `--system` 在持续集成和容器化环境中是合适的。

`--system` 标志还用于选择是否允许修改系统环境。例如，`--python` 参数可用于请求 Python 版本（例如，`--python 3.12`），uv 将搜索满足请求的解释器。如果 uv 找到系统解释器（例如，`/usr/lib/python3.12`），则需要 `--system` 标志以允许修改此非虚拟 Python 环境。如果没有 `--system` 标志，uv 将忽略不在虚拟环境中的任何解释器。相反，当提供 `--system` 标志时，uv 将忽略任何在虚拟环境中的解释器。

跨平台和发行版安装到系统 Python 是众所周知的难题。uv 支持常见情况，但并非在所有情况下都有效。例如，由于 [发行版对 `distutils`（但不是 `sysconfig`）的补丁](https://ffy00.github.io/blog/02-python-debian-and-the-install-locations/)，在 Python 3.10 之前的 Debian 上安装到系统 Python 是不支持的。虽然我们始终建议使用虚拟环境，但在这些非标准环境中，uv 认为它们是必需的。

如果 uv 安装在 Python 环境中，例如通过 `pip` 安装，它仍然可以用于修改其他环境。然而，当通过 `python -m uv` 调用时，uv 将默认使用父解释器的环境。通过 Python 调用 uv 会增加启动开销，不建议在一般情况下使用。

uv 本身不依赖于 Python，但它需要定位一个 Python 环境以 (1) 将依赖项安装到环境中和 (2) 构建源代码分发。

## Python 环境的发现

当运行修改环境的命令（如 `uv pip sync` 或 `uv pip install`）时，uv 将按以下顺序搜索虚拟环境：

- 基于 `VIRTUAL_ENV` 环境变量激活的虚拟环境。
- 基于 `CONDA_PREFIX` 环境变量激活的 Conda 环境。
- 当前目录或最近父目录中的 `.venv` 虚拟环境。

如果未找到虚拟环境，uv 将提示用户在当前目录中通过 `uv venv` 创建一个。

如果包含 `--system` 标志，uv 将跳过虚拟环境搜索已安装的 Python 版本。同样，当运行不修改环境的命令（如 `uv pip compile`）时，uv 不需要虚拟环境——但仍需要 Python 解释器。有关已安装 Python 版本的发现详细信息，请参阅 [Python 发现](../concepts/python-versions.md#discovery-of-python-versions) 文档。
```