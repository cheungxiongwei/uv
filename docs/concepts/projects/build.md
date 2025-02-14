# 构建分发文件

要将你的项目分发给其他人（例如上传到 PyPI 等索引），你需要将其构建为可分发的格式。

Python 项目通常以源代码分发（sdists）和二进制分发（wheels）两种形式分发。前者通常是一个包含项目源代码以及一些额外元数据的 `.tar.gz` 或 `.zip` 文件，而后者是一个包含预构建工件的 `.whl` 文件，可以直接安装。

!!! important

    使用 `uv build` 时，uv 作为 [build frontend](https://peps.python.org/pep-0517/#terminology-and-goals)，仅确定要使用的 Python 版本并调用 build backend。构建的详细信息，例如包含的文件和分发文件名，由 build backend 决定，如 [`[build-system]`](./config.md#build-systems) 中所定义。有关构建配置的信息可以在相应工具的文档中找到。

## 使用 `uv build`

`uv build` 可用于为你的项目构建源代码分发和二进制分发。默认情况下，`uv build` 会在当前目录下构建项目，并将构建的工件放置在 `dist/` 子目录中：

```console
$ uv build
$ ls dist/
example-0.1.0-py3-none-any.whl
example-0.1.0.tar.gz
```

你可以通过提供路径来在不同的目录中构建项目，例如 `uv build path/to/project`。

`uv build` 会首先构建一个源代码分发，然后从该源代码分发构建一个二进制分发（wheel）。

你可以通过 `uv build --sdist` 限制 `uv build` 仅构建源代码分发，通过 `uv build --wheel` 仅构建二进制分发，或者通过 `uv build --sdist --wheel` 从源代码构建两种分发。

## 构建约束

`uv build` 接受 `--build-constraint`，可用于在构建过程中约束任何构建需求的版本。当与 `--require-hashes` 结合使用时，uv 将确保用于构建项目的需求与特定的已知哈希值匹配，以实现可重复性。

例如，给定以下 `constraints.txt`：

```text
setuptools==68.2.2 --hash=sha256:b454a35605876da60632df1a60f736524eb73cc47bbc9f3f1ef1b644de74fd2a
```

运行以下命令将使用指定版本的 `setuptools` 构建项目，并验证下载的 `setuptools` 分发是否与指定的哈希值匹配：

```console
$ uv build --build-constraint constraints.txt --require-hashes
```