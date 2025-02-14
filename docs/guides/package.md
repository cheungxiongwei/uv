---
title: Building and publishing a package
description: A guide to using uv to build and publish Python packages to a package index, like PyPI.
---

# 构建和发布包

uv 支持通过 `uv build` 将 Python 包构建为源码和二进制分发版，并通过 `uv publish` 将它们上传到注册表。

## 准备项目打包

在尝试发布项目之前，您需要确保项目已准备好进行分发打包。

如果您的项目在 `pyproject.toml` 中没有包含 `[build-system]` 定义，uv 默认不会构建它。这意味着您的项目可能尚未准备好进行分发。有关声明构建系统的影响的更多信息，请参阅[项目概念](../concepts/projects/config.md#build-systems)文档。

!!! note

    如果您有不想发布的内部分包，可以将它们标记为私有：

    ```toml
    [project]
    classifiers = ["Private :: Do Not Upload"]
    ```

    此设置会使 PyPI 拒绝您上传的包进行发布。它不会影响其他注册表的安全或隐私设置。

    我们还建议仅生成每个项目的令牌：如果没有与项目匹配的 PyPI 令牌，它就不会被意外发布。

## 构建您的包

使用 `uv build` 构建您的包：

```console
$ uv build
```

默认情况下，`uv build` 会构建当前目录中的项目，并将构建的工件放置在 `dist/` 子目录中。

或者，`uv build <SRC>` 会构建指定目录中的包，而 `uv build --package <PACKAGE>` 会构建当前工作区中的指定包。

!!! info

    默认情况下，`uv build` 会尊重 `tool.uv.sources`，从 `pyproject.toml` 的 `build-system.requires` 部分解析构建依赖项。在发布包时，我们建议运行 `uv build --no-sources`，以确保在禁用 `tool.uv.sources` 时包能正确构建，就像使用其他构建工具（如 [`pypa/build`](https://github.com/pypa/build)）时的情况一样。

## 发布您的包

使用 `uv publish` 发布您的包：

```console
$ uv publish
```

使用 `--token` 或 `UV_PUBLISH_TOKEN` 设置 PyPI 令牌，或使用 `--username` 或 `UV_PUBLISH_USERNAME` 设置用户名，并使用 `--password` 或 `UV_PUBLISH_PASSWORD` 设置密码。对于从 GitHub Actions 发布到 PyPI，您不需要设置任何凭据。相反，[将受信任的发布者添加到 PyPI 项目](https://docs.pypi.org/trusted-publishers/adding-a-publisher/)。

!!! note

    PyPI 不再支持使用用户名和密码发布，您需要生成一个令牌。使用令牌等同于设置 `--username __token__` 并将令牌作为密码。

如果您通过 `[[tool.uv.index]]` 使用自定义索引，请添加 `publish-url` 并使用 `uv publish --index <name>`。例如：

```toml
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
```

!!! note

    使用 `uv publish --index <name>` 时，必须存在 `pyproject.toml`，即在发布 CI 作业中需要有一个检出步骤。

尽管 `uv publish` 会重试失败的上传，但有时发布可能会在中间失败，部分文件已上传，而部分文件仍缺失。对于 PyPI，您可以重试完全相同的命令，现有的相同文件将被忽略。对于其他注册表，请使用 `--check-url <index url>` 并指定包所属的索引 URL（而不是发布 URL）。使用 `--index` 时，索引 URL 将用作检查 URL。uv 将跳过上传与注册表中相同的文件，并且还会处理并行的竞争上传。请注意，现有文件需要与之前上传到注册表的文件完全匹配，这可以避免意外发布相同版本但内容不同的源码分发版和轮子文件。

## 安装您的包

使用 `uv run` 测试包是否可以安装和导入：

```console
$ uv run --with <PACKAGE> --no-project -- python -c "import <PACKAGE>"
```

`--no-project` 标志用于避免从本地项目目录安装包。

!!! tip

    如果您最近安装了该包，可能需要包含 `--refresh-package <PACKAGE>` 选项，以避免使用包的缓存版本。

## 下一步

要了解更多关于发布包的信息，请查看 [PyPA 指南](https://packaging.python.org/en/latest/guides/section-build-and-publish/)中关于构建和发布的部分。

或者，继续阅读 [指南](./integration/index.md)，了解如何将 uv 与其他软件集成。