# 环境变量

uv 定义并遵循以下环境变量：

### `UV_BREAK_SYSTEM_PACKAGES`

等同于 `--break-system-packages` 命令行参数。如果设置为 `true`，uv 将允许安装与系统安装的包冲突的包。

警告：`UV_BREAK_SYSTEM_PACKAGES=true` 旨在用于持续集成（CI）或容器化环境，应谨慎使用，因为修改系统 Python 可能导致意外行为。

### `UV_BUILD_CONSTRAINT`

等同于 `--build-constraint` 命令行参数。如果设置，uv 将使用此文件作为任何源码分发的构建约束。使用空格分隔的文件列表。

### `UV_CACHE_DIR`

等同于 `--cache-dir` 命令行参数。如果设置，uv 将使用此目录进行缓存，而不是默认的缓存目录。

### `UV_COMPILE_BYTECODE`

等同于 `--compile-bytecode` 命令行参数。如果设置，uv 将在安装后将 Python 源文件编译为字节码。

### `UV_CONCURRENT_BUILDS`

设置 uv 在任何给定时间并发构建源码分发的最大数量。

### `UV_CONCURRENT_DOWNLOADS`

设置 uv 在任何给定时间并发下载的最大数量。

### `UV_CONCURRENT_INSTALLS`

控制安装和解压包时使用的线程数。

### `UV_CONFIG_FILE`

等同于 `--config-file` 命令行参数。期望使用本地 `uv.toml` 文件作为配置文件。

### `UV_CONSTRAINT`

等同于 `--constraint` 命令行参数。如果设置，uv 将使用此文件作为约束文件。使用空格分隔的文件列表。

### `UV_CUSTOM_COMPILE_COMMAND`

等同于 `--custom-compile-command` 命令行参数。

用于覆盖 `uv pip compile` 生成的 `requirements.txt` 文件输出头中的 uv。旨在用于从包装脚本中调用 `uv pip compile` 的场景，以便在输出文件中包含包装脚本的名称。

### `UV_DEFAULT_INDEX`

等同于 `--default-index` 命令行参数。如果设置，uv 将使用此 URL 作为搜索包的默认索引。

### `UV_ENV_FILE`

执行 `uv run` 命令时从中加载环境变量的 `.env` 文件。

### `UV_EXCLUDE_NEWER`

等同于 `--exclude-newer` 命令行参数。如果设置，uv 将排除在指定日期之后发布的分发版。

### `UV_EXTRA_INDEX_URL`

等同于 `--extra-index-url` 命令行参数。如果设置，uv 将使用此空格分隔的 URL 列表作为搜索包的附加索引。（已弃用：请使用 `UV_INDEX`。）

### `UV_FIND_LINKS`

等同于 `--find-links` 命令行参数。如果设置，uv 将使用此逗号分隔的附加位置列表来搜索包。

### `UV_FORK_STRATEGY`

等同于 `--fork-strategy` 参数。控制在通用解析期间的版本选择。

### `UV_FROZEN`

等同于 `--frozen` 命令行参数。如果设置，uv 将在不更新 `uv.lock` 文件的情况下运行。

### `UV_GITHUB_TOKEN`

等同于 `--token` 参数，用于自我更新。用于身份验证的 GitHub token。

### `UV_GIT_LFS`

启用从 Git 仓库安装包时获取存储在 Git LFS 中的文件。

### `UV_HTTP_TIMEOUT`

HTTP 请求的超时时间（以秒为单位）。（默认：30 秒）

### `UV_INDEX`

等同于 `--index` 命令行参数。如果设置，uv 将使用此空格分隔的 URL 列表作为搜索包的附加索引。

### `UV_INDEX_STRATEGY`

等同于 `--index-strategy` 命令行参数。

例如，如果设置为 `unsafe-any-match`，uv 将考虑所有索引 URL 中给定包的可用版本，而不是将其搜索限制在包含该包的第一个索引 URL。

### `UV_INDEX_URL`

等同于 `--index-url` 命令行参数。如果设置，uv 将使用此 URL 作为搜索包的默认索引。（已弃用：请使用 `UV_DEFAULT_INDEX`。）

### `UV_INDEX_{name}_PASSWORD`

为命名索引提供 HTTP Basic 身份验证密码。

`name` 参数是索引的名称。例如，给定一个名为 `foo` 的索引，环境变量键将为 `UV_INDEX_FOO_PASSWORD`。

### `UV_INDEX_{name}_USERNAME`

为命名索引提供 HTTP Basic 身份验证用户名。

`name` 参数是索引的名称。例如，给定一个名为 `foo` 的索引，环境变量键将为 `UV_INDEX_FOO_USERNAME`。

### `UV_INSECURE_HOST`

等同于 `--allow-insecure-host` 参数。

### `UV_INSTALLER_GHE_BASE_URL`

使用独立安装程序和 `self update` 功能下载 uv 的 URL，代替默认的 GitHub Enterprise URL。

### `UV_INSTALLER_GITHUB_BASE_URL`

使用独立安装程序和 `self update` 功能下载 uv 的 URL，代替默认的 GitHub URL。

### `UV_INSTALL_DIR`

使用独立安装程序和 `self update` 功能安装 uv 的目录。默认为 `~/.local/bin`。

### `UV_KEYRING_PROVIDER`

等同于 `--keyring-provider` 命令行参数。如果设置，uv 将使用此值作为密钥环提供程序。

### `UV_LINK_MODE`

等同于 `--link-mode` 命令行参数。如果设置，uv 将使用此作为链接模式。

### `UV_LOCKED`

等同于 `--locked` 命令行参数。如果设置，uv 将断言 `uv.lock` 保持不变。

### `UV_NATIVE_TLS`

等同于 `--native-tls` 命令行参数。如果设置为 `true`，uv 将使用系统的信任存储而不是捆绑的 `webpki-roots` crate。

### `UV_NO_BINARY`

等同于 `--no-binary` 命令行参数。如果设置，uv 将从源码安装所有包。解析器仍将使用预构建的 wheel 来提取包元数据（如果可用）。

### `UV_NO_BINARY_PACKAGE`

等同于 `--no-binary-package` 命令行参数。如果设置，uv 将不会使用给定空格分隔的包列表的预构建 wheel。

### `UV_NO_BUILD_ISOLATION`

等同于 `--no-build-isolation` 命令行参数。如果设置，uv 将在构建源码分发时跳过隔离。

### `UV_NO_CACHE`

等同于 `--no-cache` 命令行参数。如果设置，uv 将不会为任何操作使用缓存。

### `UV_NO_CONFIG`

等同于 `--no-config` 命令行参数。如果设置，uv 将不会从当前目录、父目录或用户配置目录读取任何配置文件。

### `UV_NO_ENV_FILE`

执行 `uv run` 命令时忽略 `.env` 文件。

### `UV_NO_INSTALLER_METADATA`

跳过将 `uv` 安装程序元数据文件（例如，`INSTALLER`、`REQUESTED` 和 `direct_url.json`）写入 site-packages `.dist-info` 目录。

### `UV_NO_PROGRESS`

等同于 `--no-progress` 命令行参数。禁用所有进度输出。例如，旋转器和进度条。

### `UV_NO_SYNC`

等同于 `--no-sync` 命令行参数。如果设置，uv 将跳过更新环境。

### `UV_NO_VERIFY_HASHES`

等同于 `--no-verify-hashes` 参数。禁用对 `requirements.txt` 文件的哈希验证。

### `UV_NO_WRAP`

用于禁用诊断信息的换行。

### `UV_OFFLINE`

等同于 `--offline` 命令行参数。如果设置，uv 将禁用网络访问。

### `UV_OVERRIDE`

等同于 `--override` 命令行参数。如果设置，uv 将使用此文件作为覆盖文件。使用空格分隔的文件列表。

### `UV_PRERELEASE`

等同于 `--prerelease` 命令行参数。例如，如果设置为 `allow`，uv 将允许所有依赖项的预发布版本。

### `UV_PREVIEW`

等同于 `--preview` 参数。启用预览模式。

### `UV_PROJECT_ENVIRONMENT`

指定用于项目虚拟环境的目录路径。

有关更多详细信息，请参阅[项目文档](../concepts/projects/config.md#project-environment-path)。

### `UV_PUBLISH_CHECK_URL`

如果文件已存在于索引上，则不上传文件。值为索引的 URL。

### `UV_PUBLISH_INDEX`

等同于 `uv publish` 中的 `--index` 命令行参数。如果设置，uv 将使用配置中此名称的索引进行发布。

### `UV_PUBLISH_PASSWORD`

等同于 `uv publish` 中的 `--password` 命令行参数。如果设置，uv 将使用此密码进行发布。

### `UV_PUBLISH_TOKEN`

等同于 `uv publish` 中的 `--token` 命令行参数。如果设置，uv 将使用此 token（用户名为 `__token__`）进行发布。

### `UV_PUBLISH_URL`

等同于 `--publish-url` 命令行参数。与 `uv publish` 一起使用的索引上传端点的 URL。

### `UV_PUBLISH_USERNAME`

等同于 `uv publish` 中的 `--username` 命令行参数。如果设置，uv 将使用此用户名进行发布。

### `UV_PYPY_INSTALL_MIRROR`

托管的 PyPy 安装从 [python.org](https://downloads.python.org/) 下载。

此变量可以设置为镜像 URL 以使用不同的 PyPy 安装源。提供的 URL 将替换 `https://downloads.python.org/pypy`，例如 `https://downloads.python.org/pypy/pypy3.8-v7.3.7-osx64.tar.bz2`。通过使用 `file://` URL 方案，可以从本地目录读取分发版。

### `UV_PYTHON`

等同于 `--python` 命令行参数。如果设置为路径，uv 将使用此 Python 解释器进行所有操作。

### `UV_PYTHON_BIN_DIR`

指定放置已安装的托管 Python 可执行文件链接的目录。

### `UV_PYTHON_DOWNLOADS`

等同于 [`python-downloads`](../reference/settings.md#python-downloads) 设置，并且在禁用时等同于 `--no-python-downloads` 选项。是否允许 uv 下载 Python。

### `UV_PYTHON_INSTALL_DIR`

指定存储托管 Python 安装的目录。

### `UV_PYTHON_INSTALL_MIRROR`

托管的 Python 安装从 Astral [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) 项目下载。

此变量可以设置为镜像 URL 以使用不同的 Python 安装源。提供的 URL 将替换 `https://github.com/astral-sh/python-build-standalone/releases/download`，例如 `https://github.com/astral-sh/python-build-standalone/releases/download/20240713/cpython-3.12.4%2B20240713-aarch64-apple-darwin-install_only.tar.gz`。通过使用 `file://` URL 方案，可以从本地目录读取分发版。

### `UV_PYTHON_PREFERENCE`

等同于 `--python-preference` 命令行参数。uv 是否应优先使用系统或托管的 Python 版本。

### `UV_REQUEST_TIMEOUT`

HTTP 请求的超时时间（以秒为单位）。等同于 `UV_HTTP_TIMEOUT`。

### `UV_REQUIRE_HASHES`

等同于 `--require-hashes` 命令行参数。如果设置为 `true`，uv 将要求所有依赖项在需求文件中指定哈希值。

### `UV_RESOLUTION`

等同于 `--resolution` 命令行参数。例如，如果设置为 `lowest-direct`，uv 将安装所有直接依赖项的最低兼容版本。

### `UV_SYSTEM_PYTHON`

等同于 `--system` 命令行参数。如果设置为 `true`，uv 将使用系统 `PATH` 中找到的第一个 Python 解释器。

警告：`UV_SYSTEM_PYTHON=true` 旨在用于持续集成（CI）或容器化环境，应谨慎使用，因为修改系统 Python 可能导致意外行为。

### `UV_TOOL_BIN_DIR`

指定安装工具可执行文件的 "bin" 目录。

### `UV_TOOL_DIR`

指定 uv 存储托管工具的目录。

### `UV_UNMANAGED_INSTALL`

用于 CI 等临时环境，将 uv 安装到特定路径，同时防止安装程序修改 shell 配置文件或环境变量。

### `UV_VENV_SEED`

将种子包（一个或多个：`pip`、`setuptools` 和 `wheel`）安装到 `uv venv` 创建的虚拟环境中。

请注意，`setuptools` 和 `wheel` 不包含在 Python 3.12+ 环境中。

## 外部定义的环境变量

uv 还读取以下外部定义的环境变量：

### `ACTIONS_ID_TOKEN_REQUEST_TOKEN`

用于通过 `uv publish` 进行可信发布。包含 oidc 请求 token。

### `ACTIONS_ID_TOKEN_REQUEST_URL`

用于通过 `uv publish` 进行可信发布。包含 oidc token URL。

### `ALL_PROXY`

所有网络请求的通用代理。

### `BASH_VERSION`

用于检测 Bash shell 的使用。

### `CLICOLOR_FORCE`

用于通过 `anstyle` 控制颜色。

### `CONDA_DEFAULT_ENV`

用于确定激活的 Conda 环境是否是基础环境。

### `CONDA_PREFIX`

用于检测激活的 Conda 环境。

### `FISH_VERSION`

用于检测 Fish shell 的使用。

### `FORCE_COLOR`

无论终端支持如何，强制彩色输出。

参见 [force-color.org](https://force-color.org)。

### `GITHUB_ACTIONS`

用于通过 `uv publish` 进行可信发布。

### `HOME`

标准的 `HOME` 环境变量。

### `HTTPS_PROXY`

HTTPS 请求的代理。

### `HTTP_PROXY`

HTTP 请求的代理。

### `HTTP_TIMEOUT`

HTTP 请求的超时时间（以秒为单位）。等同于 `UV_HTTP_TIMEOUT`。

### `INSTALLER_NO_MODIFY_PATH`

使用独立安装程序和 `self update` 功能安装 uv 时，避免修改 `PATH` 环境变量。

### `JPY_SESSION_NAME`

用于检测是否在 Jupyter notebook 中运行。

### `KSH_VERSION`

用于检测 Ksh shell 的使用。

### `LOCALAPPDATA`

用于查找 Microsoft Store Python 安装。

### `MACOSX_DEPLOYMENT_TARGET`

与 `--python-platform macos` 及相关变体一起使用，以设置部署目标（即支持的最低 macOS 版本）。

默认为 `12.0`，这是撰写本文时最新的非 EOL macOS 版本。

### `NETRC`

用于设置 .netrc 文件位置。

### `NO_COLOR`

禁用彩色输出（优先于 `FORCE_COLOR`）。

参见 [no-color.org](https://no-color.org)。

### `NU_VERSION`

用于检测 `NuShell` 的使用。

### `PAGER`

标准的 `PAGER` posix 环境变量。由 `uv` 用于配置适当的分页器。

### `PATH`

标准的 `PATH` 环境变量。

### `PROMPT`

用于检测 Windows 命令提示符（与 PowerShell 相对）的使用。

### `PWD`

标准的 `PWD` posix 环境变量。

### `PYC_INVALIDATION_MODE`

与 `--compile` 一起使用时使用的验证模式。

参见 [`PycInvalidationMode`](https://docs.python.org/3/library/py_compile.html#py_compile.PycInvalidationMode)。

### `PYTHONPATH`

将目录添加到 Python 模块搜索路径（例如，`PYTHONPATH=/path/to/modules`）。

### `RUST_LOG`

如果设置，uv 将使用此值作为其 `--verbose` 输出的日志级别。接受与 `tracing_subscriber` crate 兼容的任何过滤器。

例如：

* `RUST_LOG=uv=debug` 等同于在命令行中添加 `--verbose`
* `RUST_LOG=trace` 将启用跟踪级别的日志记录。

有关更多信息，请参阅 [tracing 文档](https://docs.rs/tracing-subscriber/latest/tracing_subscriber/filter/struct.EnvFilter.html#example-syntax)。

### `RUST_MIN_STACK`

用于设置 uv 使用的堆栈大小。

值为字节，默认值通常为 2MB（2097152）。与正常的 `RUST_MIN_STACK` 语义不同，这可能会影响主线程堆栈大小，因为我们实际上会生成自己的 main2 线程以解决 Windows 的真实主线程仅为 1MB 的问题。该线程的大小为 `max(RUST_MIN_STACK, 4MB)`。

### `SHELL`

标准的 `SHELL` posix 环境变量。

### `SSL_CERT_FILE`

SSL 连接的自定义证书包文件路径。

### `SSL_CLIENT_CERT`

如果设置，uv 将使用此文件进行 mTLS 身份验证。这应该是一个包含证书和私钥的 PEM 格式的单个文件。

### `SYSTEMDRIVE`

Windows 系统上系统级配置目录的路径。

### `TRACING_DURATIONS_FILE`

用于通过 `tracing-durations-export` 功能创建跟踪持续时间文件。

### `VIRTUAL_ENV`

用于检测激活的虚拟环境。

### `VIRTUAL_ENV_DISABLE_PROMPT`
如果在激活虚拟环境之前设置为 1，则虚拟环境名称将不会添加到终端提示符前。

### `XDG_BIN_HOME`
可执行文件安装目录的路径。

### `XDG_CACHE_HOME`
Unix 系统上缓存目录的路径。

### `XDG_CONFIG_DIRS`
Unix 系统上系统级配置目录的路径。

### `XDG_CONFIG_HOME`
Unix 系统上用户级配置目录的路径。

### `XDG_DATA_HOME`
用于存储托管 Python 安装和工具的目录路径。

### `ZDOTDIR`
用于在使用 Zsh 时确定使用哪个 .zshenv。

### `ZSH_VERSION`
用于检测 Zsh shell 的使用情况。
