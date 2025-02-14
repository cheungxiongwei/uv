# 在项目中运行命令

在项目中工作时，项目会被安装到 `.venv` 虚拟环境中。默认情况下，该环境与当前 shell 是隔离的，因此需要项目的命令调用（例如 `python -c "import example"`）会失败。相反，应使用 `uv run` 在项目环境中运行命令：

```console
$ uv run python -c "import example"
```

使用 `run` 时，uv 会确保在运行给定命令之前项目环境是最新的。

给定的命令可以由项目环境提供，也可以存在于项目环境之外，例如：

```console
$ # 假设项目提供了 `example-cli`
$ uv run example-cli foo

$ # 运行一个需要项目可用的 `bash` 脚本
$ uv run bash scripts/foo.sh
```

## 请求额外的依赖项

可以为每次调用请求额外的依赖项或不同版本的依赖项。

`--with` 选项用于在调用中包含一个依赖项，例如请求不同版本的 `httpx`：

```console
$ uv run --with httpx==0.26.0 python -c "import httpx; print(httpx.__version__)"
0.26.0
$ uv run --with httpx==0.25.0 python -c "import httpx; print(httpx.__version__)"
0.25.0
```

无论项目的要求如何，请求的版本都会被尊重。例如，即使项目要求 `httpx==0.24.0`，上面的输出结果也会相同。

## 运行脚本

声明了内联元数据的脚本会自动在与项目隔离的环境中执行。有关更多详细信息，请参阅 [脚本指南](../../guides/scripts.md#declaring-script-dependencies)。

例如，给定一个脚本：

```python title="example.py"
# /// script
# dependencies = [
#   "httpx",
# ]
# ///

import httpx

resp = httpx.get("https://peps.python.org/api/peps.json")
data = resp.json()
print([(k, v["title"]) for k, v in data.items()][:10])
```

调用 `uv run example.py` 将在与项目隔离的环境中运行，并且仅使用列出的依赖项。

## 信号处理

uv 不会将进程的控制权交给生成的命令，以便在失败时提供更好的错误信息。因此，uv 负责将一些信号转发给所请求命令运行的子进程。

在 Unix 系统上，uv 会将 SIGINT 和 SIGTERM 转发给子进程。由于 shell 在按下 Ctrl-C 时会将 SIGINT 发送到前台进程组，因此只有在多次看到 SIGINT 或子进程组与 uv 的进程组不同时，uv 才会将 SIGINT 转发给子进程。

在 Windows 上，这些概念不适用，uv 会忽略 Ctrl-C 事件，将处理权交给子进程，以便其可以正常退出。