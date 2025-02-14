# 缓存

## 依赖缓存

uv 使用积极的缓存策略，以避免重新下载（和重新构建）在之前运行中已经访问过的依赖项。

uv 的缓存语义根据依赖项的性质有所不同：

- **对于 registry 依赖项**（例如从 PyPI 下载的依赖项），uv 会遵循 HTTP 缓存头。
- **对于直接 URL 依赖项**，uv 会遵循 HTTP 缓存头，并且还会基于 URL 本身进行缓存。
- **对于 Git 依赖项**，uv 会基于完全解析的 Git commit hash 进行缓存。因此，`uv pip compile` 在写入解析后的依赖集时会将 Git 依赖项固定到特定的 commit hash。
- **对于本地依赖项**，uv 会基于源归档文件（即本地的 `.whl` 或 `.tar.gz` 文件）的最后修改时间进行缓存。对于目录，uv 会基于 `pyproject.toml`、`setup.py` 或 `setup.cfg` 文件的最后修改时间进行缓存。

如果您遇到缓存问题，uv 提供了一些应急措施：

- 要强制 uv 重新验证所有依赖项的缓存数据，可以向任何命令传递 `--refresh` 参数（例如，`uv sync --refresh` 或 `uv pip install --refresh ...`）。
- 要强制 uv 重新验证特定依赖项的缓存数据，可以向任何命令传递 `--refresh-package` 参数（例如，`uv sync --refresh-package flask` 或 `uv pip install --refresh-package flask ...`）。
- 要强制 uv 忽略已安装的版本，可以向任何安装命令传递 `--reinstall` 参数（例如，`uv sync --reinstall` 或 `uv pip install --reinstall ...`）。

## 动态元数据

默认情况下，uv 仅会在目录根目录中的 `pyproject.toml`、`setup.py` 或 `setup.cfg` 文件发生变化时重新构建和重新安装本地目录依赖项（例如，可编辑安装）。这是一种启发式方法，在某些情况下，可能会导致比预期更少的重新安装。

要将其他信息纳入特定包的缓存键中，您可以在 `tool.uv.cache-keys` 下添加缓存键条目，这些条目可以包括文件路径和 Git commit hash。

例如，如果项目使用 [`setuptools-scm`](https://pypi.org/project/setuptools-scm/)，并且应在 commit hash 发生变化时重新构建，您可以在项目的 `pyproject.toml` 中添加以下内容：

```toml title="pyproject.toml"
[tool.uv]
cache-keys = [{ git = { commit = true } }]
```

如果您的动态元数据包含来自 Git 标签集的信息，您可以扩展缓存键以包含标签：

```toml title="pyproject.toml"
[tool.uv]
cache-keys = [{ git = { commit = true, tags = true } }]
```

同样，如果项目从 `requirements.txt` 中读取依赖项，您可以在项目的 `pyproject.toml` 中添加以下内容：

```toml title="pyproject.toml"
[tool.uv]
cache-keys = [{ file = "requirements.txt" }]
```

支持 glob 模式，遵循 [`glob`](https://docs.rs/glob/0.3.1/glob/struct.Pattern.html) crate 的语法。例如，要在项目目录或其任何子目录中的 `.toml` 文件被修改时使缓存失效，请使用以下内容：

```toml title="pyproject.toml"
[tool.uv]
cache-keys = [{ file = "**/*.toml" }]
```

!!! note

    使用 glob 模式可能会比较昂贵，因为 uv 可能需要遍历文件系统以确定是否有文件发生了变化。这可能需要遍历大型或深度嵌套的目录。

同样，如果项目依赖于环境变量，您可以在项目的 `pyproject.toml` 中添加以下内容，以在环境变量发生变化时使缓存失效：

```toml title="pyproject.toml"
[tool.uv]
cache-keys = [{ env = "MY_ENV_VAR" }]
```

作为一种应急措施，如果项目使用的 `dynamic` 元数据未被 `tool.uv.cache-keys` 覆盖，您可以通过将项目添加到 `tool.uv.reinstall-package` 列表中来指示 uv 始终重新构建和重新安装它：

```toml title="pyproject.toml"
[tool.uv]
reinstall-package = ["my-package"]
```

这将强制 uv 在每次运行时重新构建和重新安装 `my-package`，无论包的 `pyproject.toml`、`setup.py` 或 `setup.cfg` 文件是否发生了变化。

## 缓存安全性

可以安全地同时运行多个 uv 命令，即使针对相同的虚拟环境。uv 的缓存设计为线程安全且仅追加，因此对多个并发读取和写入具有鲁棒性。uv 在安装时对目标虚拟环境应用基于文件的锁，以避免跨进程的并发修改。

请注意，在其他 uv 命令运行时修改 uv 缓存（例如，`uv cache clean`）是不安全的，并且直接修改缓存（例如，删除文件或目录）是绝对不安全的。

## 清除缓存

uv 提供了几种不同的机制来从缓存中删除条目：

- `uv cache clean` 从缓存目录中删除所有缓存条目，完全清空缓存。
- `uv cache clean ruff` 删除 `ruff` 包的所有缓存条目，适用于使单个或有限包的缓存失效。
- `uv cache prune` 删除所有未使用的缓存条目。例如，缓存目录可能包含在之前的 uv 版本中创建的条目，这些条目不再需要，可以安全地删除。`uv cache prune` 可以定期运行，以保持缓存目录的清洁。

## 持续集成中的缓存

在持续集成环境（如 GitHub Actions 或 GitLab CI）中缓存包安装工件以加速后续运行是很常见的。

默认情况下，uv 会缓存从源代码构建的 wheel 和直接下载的预构建 wheel，以实现高性能的包安装。

然而，在持续集成环境中，持久化预构建 wheel 可能是不理想的。使用 uv 时，通常更快的是从缓存中省略预构建 wheel（而是在每次运行时从 registry 重新下载它们）。另一方面，缓存从源代码构建的 wheel 往往是值得的，因为 wheel 构建过程可能很昂贵，尤其是对于扩展模块。

为了支持这种缓存策略，uv 提供了 `uv cache prune --ci` 命令，该命令会删除缓存中的所有预构建 wheel 和解压的源分发文件，但保留从源代码构建的任何 wheel。我们建议在持续集成作业结束时运行 `uv cache prune --ci`，以确保最大的缓存效率。有关示例，请参见 [GitHub 集成指南](../guides/integration/github.md#caching)。

## 缓存目录

uv 按以下顺序确定缓存目录：

1. 如果请求了 `--no-cache`，则使用临时缓存目录。
2. 通过 `--cache-dir`、`UV_CACHE_DIR` 或 [`tool.uv.cache-dir`](../reference/settings.md#cache-dir) 指定的特定缓存目录。
3. 系统适当的缓存目录，例如，Unix 上的 `$XDG_CACHE_HOME/uv` 或 `$HOME/.cache/uv`，以及 Windows 上的 `%LOCALAPPDATA%\uv\cache`

!!! note

    uv 始终需要一个缓存目录。当请求 `--no-cache` 时，uv 仍将使用临时缓存来在该单次调用中共享数据。

    在大多数情况下，应使用 `--refresh` 而不是 `--no-cache`——因为它会更新后续操作的缓存，但不会从缓存中读取。

为了性能考虑，缓存目录应位于 uv 操作的 Python 环境的同一文件系统上。否则，uv 将无法从缓存中链接文件到环境中，并且将需要回退到较慢的复制操作。

## 缓存版本控制

uv 缓存由多个桶组成（例如，一个用于 wheel 的桶，一个用于源分发的桶，一个用于 Git 仓库的桶，等等）。每个桶都有版本控制，因此如果发布包含缓存格式的破坏性更改，uv 将不会尝试读取或写入不兼容的缓存桶。

例如，uv 0.4.13 包含对核心元数据桶的破坏性更改。因此，桶版本从 v12 增加到 v13。在缓存版本内，更改保证是向前和向后兼容的。

由于缓存格式的更改伴随着缓存版本的更改，因此多个版本的 uv 可以安全地读取和写入同一缓存目录。然而，如果缓存版本在一对 uv 发布之间发生了变化，那么这些发布可能无法共享相同的底层缓存条目。

例如，为 uv 0.4.12 和 uv 0.4.13 使用单个共享缓存是安全的，尽管由于缓存版本的变化，缓存本身可能在核心元数据桶中包含重复的条目。