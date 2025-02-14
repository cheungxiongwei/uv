# 可复现示例

## 为什么可复现示例很重要

最小可复现示例（MRE）对于修复bug至关重要。如果没有一个可以用来复现问题的示例，维护者就无法调试或测试是否已修复。如果示例不是最小的，即包含大量与问题无关的内容，维护者可能需要更长时间才能识别问题的根本原因。

## 如何编写可复现示例

在编写可复现示例时，目标是提供所有必要的上下文，以便其他人能够复现你的示例。这包括：

- 你使用的平台（例如操作系统和架构）
- 任何相关的系统状态
- uv的版本
- 其他相关工具的版本
- 相关文件（如`uv.lock`、`pyproject.toml`等）
- 要运行的命令

为了确保你的复现是最小的，尽可能移除不必要的依赖、设置和文件。在分享之前，请务必测试你的复现。我们建议包含复现过程中的详细日志；它们可能在你的机器上以某种关键方式有所不同。使用[Gist](https://gist.github.com)可以非常有助于分享非常长的日志。

下面，我们将介绍几种具体的[策略](#strategies-for-reproducible-examples)来创建和分享可复现示例。

!!! tip

    在[Stack Overflow](https://stackoverflow.com/help/minimal-reproducible-example)上有一个关于创建MRE基础知识的优秀指南。

## 可复现示例的策略

### Docker镜像

编写Docker镜像通常是分享可复现示例的最佳方式，因为它是完全自包含的。这意味着复现者的系统状态不会影响问题。

!!! note

    使用Docker镜像仅在问题可以在Linux上复现时可行。在使用macOS时，明智的做法是确保你的镜像在Linux上不可复现，但有些bug确实与操作系统相关。虽然使用Docker运行Windows容器是可行的，但并不常见。这类bug通常以[脚本](#script)的形式报告。

在使用uv编写Docker MRE时，最好从[uv的Docker镜像](../../guides/integration/docker.md#available-images)之一开始。这样做时，请务必固定到uv的特定版本。

```Dockerfile
FROM ghcr.io/astral-sh/uv:0.5.24-debian-slim
```

虽然Docker镜像与系统隔离，但默认情况下构建将使用你系统的架构。在分享复现时，你可以显式设置平台以确保复现者获得预期的行为。uv发布了`linux/amd64`（例如Intel或AMD）和`linux/arm64`（例如Apple M系列或ARM）的镜像。

```Dockerfile
FROM --platform=linux/amd64 ghcr.io/astral-sh/uv:0.5.24-debian-slim
```

Docker镜像最适合用于可以通过命令构建的问题，例如：

```Dockerfile
FROM --platform=linux/amd64 ghcr.io/astral-sh/uv:0.5.24-debian-slim

RUN uv init /mre
WORKDIR /mre
RUN uv add pydantic
RUN uv sync
RUN uv run -v python -c "import pydantic"
```

然而，你也可以在镜像中内联写入文件：

```Dockerfile
FROM --platform=linux/amd64 ghcr.io/astral-sh/uv:0.5.24-debian-slim

COPY <<EOF /mre/pyproject.toml
[project]
name = "example"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["pydantic"]
EOF

WORKDIR /mre
RUN uv lock
```

如果你需要写入许多文件，最好创建并发布一个[Git仓库](#git-repository)。你可以结合这些方法，在仓库中包含一个`Dockerfile`。

在分享Docker复现时，包含构建日志会很有帮助。你可以通过禁用缓存和花哨的输出看到更多构建步骤的输出：

```console
docker build . --progress plain --no-cache
```

### 脚本

在报告无法在[容器](#docker-image)中复现的平台特定bug时，最佳实践是包含一个脚本，显示可用于复现bug的命令，例如：

```bash
uv init
uv add pydantic
uv sync
uv run -v python -c "import pydantic"
```

如果你的复现需要许多文件，请使用[Git仓库](#git-repository)来分享它们。

除了脚本外，请包含失败的_详细_日志（即使用`-v`标志）和完整的错误消息。

每当脚本依赖于外部状态时，请务必分享这些信息。例如，如果你在Windows上编写了脚本，并且它使用了通过`choco`安装的Python版本，并在PowerShell 6.2上运行，请在报告中包含这些信息。

### Git仓库

在分享Git仓库复现时，请包含一个复现问题的[脚本](#script)，或者更好的是，包含一个[Dockerfile](#docker-image)。脚本的第一步应该是克隆仓库并检出特定提交：

```console
$ git clone https://github.com/<user>/<project>.git
$ cd <project>
$ git checkout <commit>
$ <commands to produce error>
```

你可以通过[GitHub UI](https://github.com/new)或`gh` CLI快速创建一个新仓库：

```console
$ gh repo create uv-mre-1234 --clone
```

在使用Git仓库进行复现时，请记住通过排除不需要的文件或设置来_最小化_内容。