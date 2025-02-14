# 通过 anyio
starlette==0.37.2
    # 通过 fastapi
typing-extensions==4.10.0
    # 通过
    #   pydantic
    #   pydantic-core
```

或者，如果解析器优先包含最新版本的 `fastapi`，则需要使用满足上限的旧版本 `starlette`。实际上，这需要回退到 `starlette==0.36.3`：

```python title="requirements.txt"
# 该文件由 uv 通过以下命令自动生成：
#    uv pip compile requirements.in
annotated-types==0.6.0
    # 通过 pydantic
anyio==4.3.0
    # 通过 starlette
fastapi==0.110.0
idna==3.6
    # 通过 anyio
pydantic==2.6.3
    # 通过 fastapi
pydantic-core==2.16.3
    # 通过 pydantic
sniffio==1.3.1
    # 通过 anyio
starlette==0.36.3
    # 通过 fastapi
typing-extensions==4.10.0
    # 通过
    #   fastapi
    #   pydantic
    #   pydantic-core
```

当 uv 的解析结果与 `pip` 不一致且不符合预期时，通常表明规范过于宽松，用户应考虑收紧它们。例如，在 `starlette` 和 `fastapi` 的情况下，用户可以要求 `fastapi>=0.110.0`。

## `pip check`

目前，`uv pip check` 将显示以下诊断信息：

- 包没有 `METADATA` 文件，或者 `METADATA` 文件无法解析。
- 包的 `Requires-Python` 与当前解释器的 Python 版本不匹配。
- 包依赖于未安装的包。
- 包依赖于已安装的包，但版本不兼容。
- 虚拟环境中安装了多个版本的包。

在某些情况下，`uv pip check` 会显示 `pip check` 不会显示的诊断信息，反之亦然。例如，与 `uv pip check` 不同，`pip check` 不会在当前环境中安装了多个版本的包时发出警告。

## `--user` 和 `user` 安装方案

uv 不支持 `--user` 标志，该标志基于 `user` 安装方案安装包。相反，我们建议使用虚拟环境来隔离包安装。

此外，如果 pip 检测到用户没有目标目录的写权限（例如在某些系统上安装到系统 Python 时），pip 会回退到 `user` 安装方案。uv 没有实现任何此类回退。

更多信息，请参见 [#2077](https://github.com/astral-sh/uv/issues/2077)。

## `--only-binary` 强制执行

`--only-binary` 参数用于限制安装仅使用预构建的二进制分发版。当提供 `--only-binary :all:` 时，pip 和 uv 都会拒绝从 PyPI 和其他注册表构建源代码分发版。

然而，当依赖项以直接 URL 形式提供时（例如，`uv pip install https://...`），pip 不会强制执行 `--only-binary`，并且会为所有此类包构建源代码分发版。

与此同时，uv 会对直接 URL 依赖项强制执行 `--only-binary`，但有一个例外：给定 `uv pip install https://... --only-binary flask`，如果 uv 无法提前推断出包名，它将构建给定 URL 的源代码分发版，因为在这种情况下，uv 无法确定该包是否“允许”而不构建其元数据。

当提供 `--only-binary` 时，pip 和 uv 都允许构建和安装可编辑需求。例如，`uv pip install -e . --only-binary :all:` 是允许的。

## `--no-binary` 强制执行

`--no-binary` 参数用于限制安装仅使用源代码分发版。当提供 `--no-binary` 时，uv 将拒绝安装预构建的二进制分发版，但会重用本地缓存中已有的任何二进制分发版。

此外，与 pip 不同，当提供 `--no-binary` 时，uv 的解析器仍会从预构建的二进制分发版中读取元数据。

## `manylinux_compatible` 强制执行

[PEP 600](https://peps.python.org/pep-0600/#package-installers) 描述了一种机制，Python 分发者可以通过在 `_manylinux` 标准库模块上定义 `manylinux_compatible` 函数来选择退出 `manylinux` 兼容性。

uv 尊重 `manylinux_compatible`，但仅针对当前的 glibc 版本进行测试，并全局应用 `manylinux_compatible` 的返回值。

换句话说，如果 `manylinux_compatible` 返回 `True`，uv 将系统视为 `manylinux` 兼容；如果返回 `False`，uv 将系统视为 `manylinux` 不兼容，而不会为每个 glibc 版本调用 `manylinux_compatible`。

这种方法并不是规范的完整实现，但与常见的全局 `manylinux_compatible` 实现（如 [`no-manylinux`](https://pypi.org/project/no-manylinux/)）兼容：

```python
from __future__ import annotations
manylinux1_compatible = False
manylinux2010_compatible = False
manylinux2014_compatible = False


def manylinux_compatible(*_, **__):  # PEP 600
    return False
```

## 字节码编译

与 `pip` 不同，uv 默认情况下不会在安装期间将 `.py` 文件编译为 `.pyc` 文件（即，uv 不会创建或填充 `__pycache__` 目录）。要在安装期间启用字节码编译，请将 `--compile-bytecode` 标志传递给 `uv pip install` 或 `uv pip sync`，或将 `UV_COMPILE_BYTECODE` 环境变量设置为 `1`。

跳过字节码编译在某些工作流程中可能是不理想的；例如，我们建议在 [Docker 构建](../guides/integration/docker.md) 中启用字节码编译，以提高启动时间（以增加构建时间为代价）。

由于字节码编译会抑制 Python 解释器发出的各种警告，在极少数情况下，您可能会在运行使用 uv 安装的 Python 代码时看到 `SyntaxWarning` 或 `DeprecationWarning` 消息，而这些消息在使用 `pip` 时不会出现。这些是有效的警告，但通常被字节码编译过程隐藏，可以忽略、在上游修复，或通过启用 uv 中的字节码编译来类似地抑制。

## 严格性和规范执行

uv 通常比 `pip` 更严格，并且经常会拒绝 `pip` 会安装的包。例如，uv 会拒绝带有无效 URL 片段的 HTML 索引（参见：[PEP 503](https://peps.python.org/pep-0503/)），而 `pip` 会忽略这些片段。

在某些情况下，uv 会为已知存在特定规范合规性问题的流行包实现宽松行为。

如果 uv 由于规范违规而拒绝安装 `pip` 会安装的包，最佳做法是首先尝试安装该包的较新版本；如果失败，则向包维护者报告问题。

## `pip` 命令行选项和子命令

uv 不支持 `pip` 的完整命令行选项和子命令集，尽管它支持其中的大部分。

缺失的选项和子命令根据用户需求和实现的复杂性进行优先排序，并通常在单独的问题中进行跟踪。例如：

- [`--trusted-host`](https://github.com/astral-sh/uv/issues/1339)
- [`--user`](https://github.com/astral-sh/uv/issues/2077)

如果您遇到缺失的选项或子命令，请搜索问题跟踪器以查看是否已报告，如果没有，请考虑打开一个新问题。欢迎对现有问题进行投票以表达您的兴趣。

## 注册表认证

uv 不支持 `pip` 的 `auto` 或 `import` 选项用于 `--keyring-provider`。目前，仅支持 `subprocess` 选项。

与 `pip` 不同，uv 默认情况下不启用 keyring 认证。

与 `pip` 不同，uv 不会等到请求返回 HTTP 401 后才搜索认证。uv 会为所有具有可用凭据的主机请求附加认证。

## `egg` 支持

uv 不支持 `pip` 中视为遗留或已弃用的功能。例如，uv 不支持 `.egg` 风格的分发版。

然而，uv 确实部分支持 (1) `.egg-info` 风格的分发版（偶尔在 Docker 镜像和 Conda 环境中找到）和 (2) 遗留的可编辑 `.egg-link` 风格的分发版。

具体来说，uv 不支持安装新的 `.egg-info` 或 `.egg-link` 风格的分发版，但会在解析期间尊重任何此类现有分发版，使用 `uv pip list` 和 `uv pip freeze` 列出它们，并使用 `uv pip uninstall` 卸载它们。

## 构建约束

当通过 `--constraint`（或 `UV_CONSTRAINT`）提供约束时，uv 不会在解析构建依赖项（即构建源代码分发版）时应用这些约束。相反，构建约束应通过专用的 `--build-constraint`（或 `UV_BUILD_CONSTRAINT`）设置提供。

与此同时，pip 在通过 `PIP_CONSTRAINT` 指定时会应用构建依赖项的约束，但在命令行上通过 `--constraint` 提供时则不会。

例如，要确保使用 `setuptools 60.0.0` 来构建任何具有 `setuptools` 构建依赖项的包，请使用 `--build-constraint`，而不是 `--constraint`。

## `pip compile` 默认值

`pip compile` 和 `pip-tools` 的默认行为有一些小但值得注意的差异。

默认情况下，uv 不会将编译后的需求写入输出文件。相反，uv 要求用户使用 `-o` 或 `--output-file` 选项明确指定输出文件。

默认情况下，uv 在输出编译后的需求时会去除 extras。换句话说，uv 默认使用 `--strip-extras`，而 `pip-compile` 默认使用 `--no-strip-extras`。`pip-compile` 计划在下一个主要版本（v8.0.0）中更改此默认值，届时两个工具都将默认使用 `--strip-extras`。要在 uv 中保留 extras，请将 `--no-strip-extras` 标志传递给 `uv pip compile`。

默认情况下，uv 不会将任何索引 URL 写入输出文件，而 `pip-compile` 会输出任何不匹配默认值（PyPI）的 `--index-url` 或 `--extra-index-url`。要在输出文件中包含索引 URL，请将 `--emit-index-url` 标志传递给 `uv pip compile`。与 `pip-compile` 不同，uv 在传递 `--emit-index-url` 时会包含所有索引 URL，包括默认索引 URL。

## `requires-python` 强制执行

在评估依赖项的 `requires-python` 范围时，uv 仅考虑下限并完全忽略上限。例如，`>=3.8, <4` 被视为 `>=3.8`。尊重 `requires-python` 的上限通常会导致形式上正确但实际上不正确的解析，因为，例如，解析器会回溯到第一个发布时省略上限的版本（参见：[`Requires-Python` 上限](https://discuss.python.org/t/requires-python-upper-limits/12663)）。

在评估 Python 版本与 `requires-python` 规范时，uv 会将候选版本截断为主要、次要和补丁组件，忽略（例如）预发布和后发布标识符。

例如，声明 `requires-python: >=3.13` 的项目将接受 Python 3.13.0b1。虽然 3.13.0b1 并不严格大于 3.13，但在忽略预发布标识符时，它大于 3.13。

虽然这并不严格符合 [PEP 440](https://peps.python.org/pep-0440/)，但它与 [pip](https://github.com/pypa/pip/blob/24.1.1/src/pip/_internal/resolution/resolvelib/candidates.py#L540) 一致。

## 包优先级

给定一组需求，通常有许多可能的解决方案，解析器必须在它们之间进行选择。uv 的解析器和 pip 的解析器具有不同的包优先级。虽然两个解析器都使用用户提供的顺序作为其优先级之一，但 pip 有额外的 [优先级](https://pip.pypa.io/en/stable/topics/more-dependency-resolution/#the-resolver-algorithm)，而 uv 没有。因此，uv 比 pip 更容易受到用户顺序变化的影响。

例如，`uv pip install foo bar` 优先考虑 `foo` 的较新版本而不是 `bar`，可能会导致与 `uv pip install bar foo` 不同的解析结果。同样，这种行为适用于 `uv pip compile` 输入文件中需求的顺序。