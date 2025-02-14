## 项目元数据
### [`conflicts`](#conflicts) {: #conflicts }

声明相互冲突的extras或依赖组集合（即互斥的）。

当两个或多个extras具有互不兼容的依赖时，声明冲突非常有用。例如，extra `foo` 可能依赖于 `numpy==2.0.0`，而 extra `bar` 依赖于 `numpy==2.1.0`。虽然这些依赖存在冲突，但用户可能不会同时激活 `foo` 和 `bar`，因此仍然可以为项目生成一个通用的解析方案。

通过明确声明这些冲突，uv可以为项目生成一个通用的解析方案，同时考虑到某些extras和组的组合是互斥的。作为交换，如果用户尝试同时激活两个冲突的extras，安装将会失败。

**默认值**: `[]`

**类型**: `list[list[dict]]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
# Require that `package[extra1]` and `package[extra2]` are resolved
# in different forks so that they cannot conflict with one another.
conflicts = [
    [
        { extra = "extra1" },
        { extra = "extra2" },
    ]
]

# Require that the dependency groups `group1` and `group2`
# are resolved in different forks so that they cannot conflict
# with one another.
conflicts = [
    [
        { group = "group1" },
        { group = "group2" },
    ]
]
```

---

### [`constraint-dependencies`](#constraint-dependencies) {: #constraint-dependencies }

在解析项目依赖时应用的约束。

约束用于限制在解析过程中选择的依赖版本。

将某个包作为约束包含进来，并不会单独触发该包的安装；相反，该包必须在项目的一级或传递依赖中被明确请求。

!!! 注意 在 uv lock、uv sync 和 uv run 中，uv 只会从工作区根目录的 pyproject.toml 中读取 constraint-dependencies，并会忽略其他工作区成员或 uv.toml 文件中的声明。

默认值: []

类型: list[str]

示例用法:

```toml title="pyproject.toml"
[tool.uv]
# Ensure that the grpcio version is always less than 1.65, if it's requested by a
# direct or transitive dependency.
constraint-dependencies = ["grpcio<1.65"]
```

---

### [`default-groups`](#default-groups) {: #default-groups }

默认安装的 `dependency-groups` 列表。

**默认值**: `["dev"]`

**类型**: `list[str]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
default-groups = ["docs"]
```

---

### [`dev-dependencies`](#dev-dependencies) {: #dev-dependencies }

项目的开发依赖项。

开发依赖项默认会在 `uv run` 和 `uv sync` 中安装，但不会出现在项目发布的元数据中。

不再推荐使用此字段。建议使用 `dependency-groups.dev` 字段，这是声明开发依赖项的标准方式。`tool.uv.dev-dependencies` 和 `dependency-groups.dev` 的内容会被合并，以确定 `dev` 依赖组的最终需求。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
dev-dependencies = ["ruff==0.5.0"]
```

---

### [`environments`](#environments) {: #environments }

用于解析依赖项的受支持环境列表。

默认情况下，uv 会在执行 `uv lock` 操作时为所有可能的环境解析依赖项。
然而，您可以限制受支持的环境集以提高性能，并避免解决方案空间中出现无法满足的分支。

当使用 `--universal` 标志调用 `uv pip compile` 时，这些环境也将被遵循。

**默认值**: `[]`

**类型**: `str | list[str]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
# Resolve for macOS, but not for Linux or Windows.
environments = ["sys_platform == 'darwin'"]
```

---

### [`index`](#index) {: #index }

用于解析依赖项的索引。

接受符合 [PEP 503](https://peps.python.org/pep-0503/) 的仓库（简单仓库 API），或具有相同格式的本地目录。

索引按照定义的顺序进行考虑，因此第一个定义的索引具有最高优先级。此外，此设置提供的索引优先级高于通过 [`index_url`](#index-url) 或 [`extra_index_url`](#extra-index-url) 指定的任何索引。uv 只会考虑包含特定软件包的第一个索引，除非指定了其他 [索引策略](#index-strategy)。

如果索引标记为 `explicit = true`，它将仅用于通过 `[tool.uv.sources]` 明确选择它的依赖项，例如：

```toml
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cu121"
explicit = true

[tool.uv.sources]
torch = { index = "pytorch" }
```

如果索引标记为 `default = true`，它将被移动到优先级列表的末尾，从而在解析软件包时给予最低优先级。此外，将索引标记为默认值将禁用 PyPI 默认索引。

**默认值**: `[]`

**类型**: `dict`

**示例用法**:

```toml title="pyproject.toml"

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cu121"
```

---

### [`managed`](#managed) {: #managed }

项目是否由 uv 管理。如果为 `false`，当调用 `uv run` 时，uv 将忽略该项目。

**默认值**: `true`

**类型**: `bool`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
managed = false
```

---

### [`override-dependencies`](#override-dependencies) {: #override-dependencies }

在解析项目依赖时应用的覆盖。

覆盖用于强制选择特定版本的包，无论其他包请求的版本是什么，也无论选择该版本是否通常会导致无效的解析。

虽然约束是_累加的_，即它们与组成包的要求相结合，但覆盖是_绝对的_，即它们完全替换任何组成包的要求。

将包包含为覆盖_不会_单独触发该包的安装；相反，该包必须在项目的一级或传递依赖中其他地方被请求。

!!! note
    在 `uv lock`、`uv sync` 和 `uv run` 中，uv 只会从工作区根目录的 `pyproject.toml` 中读取 `override-dependencies`，并会忽略其他工作区成员或 `uv.toml` 文件中的任何声明。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
# 无论传递依赖是否请求其他版本，始终安装 Werkzeug 2.3.0。
override-dependencies = ["werkzeug==2.3.0"]
```

---

### [`package`](#package) {: #package }

项目是否应被视为 Python 包，或非包（“虚拟”）项目。

包以可编辑模式构建并安装到虚拟环境中，因此需要构建后端，而虚拟项目_不_构建或安装；相反，只有它们的依赖项包含在虚拟环境中。

创建包需要在 `pyproject.toml` 中存在 `build-system`，并且项目结构符合构建后端的期望（例如，`src` 布局）。

**默认值**: `true`

**类型**: `bool`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv]
package = false
```

---

### [`sources`](#sources) {: #sources }

解析依赖时使用的源。

`tool.uv.sources` 通过额外的源丰富了依赖元数据，这些源在开发过程中被纳入。依赖源可以是 Git 仓库、URL、本地路径或替代注册表。

有关更多信息，请参阅 [Dependencies](../concepts/projects/dependencies.md)。

**默认值**: `{}`

**类型**: `dict`

**示例用法**:

```toml title="pyproject.toml"

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.0" }
pytest = { url = "https://files.pythonhosted.org/packages/6b/77/7440a06a8ead44c7757a64362dd22df5760f9b12dc5f11b6188cd2fc27a0/pytest-8.3.3-py3-none-any.whl" }
pydantic = { path = "/path/to/pydantic", editable = true }
```

---

### `workspace`

#### [`exclude`](#workspace_exclude) {: #workspace_exclude }
<span id="exclude"></span>

要从工作区成员中排除的包。如果一个包同时匹配 `members` 和 `exclude`，它将被排除。

支持 glob 模式和显式路径。

有关 glob 语法的更多信息，请参阅 [`glob` 文档](https://docs.rs/glob/latest/glob/struct.Pattern.html)。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv.workspace]
exclude = ["member1", "path/to/member2", "libs/*"]
```

---

#### [`members`](#workspace_members) {: #workspace_members }
<span id="members"></span>

要包含为工作区成员的包。

支持 glob 模式和显式路径。

有关 glob 语法的更多信息，请参阅 [`glob` 文档](https://docs.rs/glob/latest/glob/struct.Pattern.html)。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

```toml title="pyproject.toml"
[tool.uv.workspace]
members = ["member1", "path/to/member2", "libs/*"]
```

---

## 配置
### [`allow-insecure-host`](#allow-insecure-host) {: #allow-insecure-host }

允许与主机建立不安全的连接。

期望接收主机名（例如 `localhost`）、主机-端口对（例如 `localhost:8080`）或 URL（例如 `https://localhost`）。

警告：包含在此列表中的主机将不会与系统的证书存储进行验证。仅在安全的网络环境中使用已验证的来源时使用 `--allow-insecure-host`，因为它会绕过 SSL 验证，可能会使您面临中间人攻击的风险。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    allow-insecure-host = ["localhost:8080"]
    ```
=== "uv.toml"

    ```toml
    allow-insecure-host = ["localhost:8080"]
    ```

---

### [`cache-dir`](#cache-dir) {: #cache-dir }

缓存目录的路径。

在 macOS 上默认为 `$HOME/Library/Caches/uv`，在 Linux 上默认为 `$XDG_CACHE_HOME/uv` 或 `$HOME/.cache/uv`，在 Windows 上默认为 `%LOCALAPPDATA%\uv\cache`。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    cache-dir = "./.uv_cache"
    ```
=== "uv.toml"

    ```toml
    cache-dir = "./.uv_cache"
    ```

---

### [`cache-keys`](#cache-keys) {: #cache-keys }

在缓存项目构建时考虑的键。

缓存键允许您指定在修改时应触发重建的文件或目录。默认情况下，uv 会在项目目录中的 `pyproject.toml`、`setup.py` 或 `setup.cfg` 文件被修改时重建项目，即：

```toml
cache-keys = [{ file = "pyproject.toml" }, { file = "setup.py" }, { file = "setup.cfg" }]
```

例如：如果项目使用动态元数据从 `requirements.txt` 文件中读取其依赖项，您可以指定 `cache-keys = [{ file = "requirements.txt" }, { file = "pyproject.toml" }]`，以确保在 `requirements.txt` 文件被修改时重建项目（同时监视 `pyproject.toml`）。

支持 glob 语法，遵循 [`glob`](https://docs.rs/glob/0.3.1/glob/struct.Pattern.html) crate 的语法。例如，要在项目目录或其子目录中的任何 `.toml` 文件被修改时使缓存失效，您可以指定 `cache-keys = [{ file = "**/*.toml" }]`。请注意，使用 glob 可能会比较耗时，因为 uv 可能需要遍历文件系统以确定是否有文件发生变化。

缓存键还可以包含版本控制信息。例如，如果项目使用 `setuptools_scm` 从 Git 提交中读取其版本，您可以指定 `cache-keys = [{ git = { commit = true }, { file = "pyproject.toml" }]`，以将当前 Git 提交哈希包含在缓存键中（同时包含 `pyproject.toml`）。Git 标签也支持，例如 `cache-keys = [{ git = { commit = true, tags = true } }]`。

缓存键还可以包含环境变量。例如，如果项目依赖 `MACOSX_DEPLOYMENT_TARGET` 或其他环境变量来确定其行为，您可以指定 `cache-keys = [{ env = "MACOSX_DEPLOYMENT_TARGET" }]`，以在环境变量发生变化时使缓存失效。

缓存键仅影响定义它们的 `pyproject.toml` 中的项目（例如，不会影响工作区中的所有成员），并且所有路径和 glob 都解释为相对于项目目录。

**默认值**: `[{ file = "pyproject.toml" }, { file = "setup.py" }, { file = "setup.cfg" }]`

**类型**: `list[dict]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    cache-keys = [{ file = "pyproject.toml" }, { file = "requirements.txt" }, { git = { commit = true }]
    ```
=== "uv.toml"

    ```toml
    cache-keys = [{ file = "pyproject.toml" }, { file = "requirements.txt" }, { git = { commit = true }]
    ```

---

### [`check-url`](#check-url) {: #check-url }

检查索引URL以跳过重复上传的文件。

此选项允许重试在仅上传了部分文件后失败的发布，并处理由于并行上传相同文件导致的错误。

在上传之前，会检查索引。如果索引中已存在完全相同的文件，则不会上传该文件。如果在上传过程中发生错误，将再次检查索引，以处理相同文件被并行上传两次的情况。

具体行为将根据索引的不同而有所变化。当上传到PyPI时，即使没有`--check-url`，上传相同文件也会成功，而大多数其他索引则会报错。

索引必须提供支持的哈希值之一（SHA-256、SHA-384或SHA-512）。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    check-url = "https://test.pypi.org/simple"
    ```
=== "uv.toml"

    ```toml
    check-url = "https://test.pypi.org/simple"
    ```

---

### [`compile-bytecode`](#compile-bytecode) {: #compile-bytecode }

在安装后将Python文件编译为字节码。

默认情况下，uv不会将Python（`.py`）文件编译为字节码（`__pycache__/*.pyc`）；而是在首次导入模块时惰性执行编译。对于启动时间至关重要的用例（如CLI应用程序和Docker容器），可以启用此选项，以较长的安装时间换取更快的启动时间。

启用后，uv将处理整个site-packages目录（包括未被当前操作修改的包）以确保一致性。与pip一样，它也会忽略错误。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    compile-bytecode = true
    ```
=== "uv.toml"

    ```toml
    compile-bytecode = true
    ```

---

### [`concurrent-builds`](#concurrent-builds) {: #concurrent-builds }

uv在任何给定时间并发构建的源分发的最大数量。

默认为可用CPU核心数。

**默认值**: `None`

**类型**: `int`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    concurrent-builds = 4
    ```
=== "uv.toml"

    ```toml
    concurrent-builds = 4
    ```

---

### [`concurrent-downloads`](#concurrent-downloads) {: #concurrent-downloads }

uv在任何给定时间执行的并发下载的最大数量。

**默认值**: `50`

**类型**: `int`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    concurrent-downloads = 4
    ```
=== "uv.toml"

    ```toml
    concurrent-downloads = 4
    ```

---

### [`concurrent-installs`](#concurrent-installs) {: #concurrent-installs }

安装和解压包时使用的线程数。

默认为可用CPU核心数。

**默认值**: `None`

**类型**: `int`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    concurrent-installs = 4
    ```
=== "uv.toml"

    ```toml
    concurrent-installs = 4
    ```

---

### [`config-settings`](#config-settings) {: #config-settings }

传递给[PEP 517](https://peps.python.org/pep-0517/)构建后端的设置，以`KEY=VALUE`对的形式指定。

**默认值**: `{}`

**类型**: `dict`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    config-settings = { editable_mode = "compat" }
    ```
=== "uv.toml"

    ```toml
    config-settings = { editable_mode = "compat" }
    ```

---

### [`dependency-metadata`](#dependency-metadata) {: #dependency-metadata }

项目的依赖项（直接或传递）的预定义静态元数据。提供后，解析器将使用指定的元数据，而不是查询注册表或从源代码构建相关包。

元数据应遵循[Metadata 2.3](https://packaging.python.org/en/latest/specifications/core-metadata/)标准提供，但仅以下字段会被使用：

- `name`: 包的名称。
- （可选）`version`: 包的版本。如果省略，元数据将应用于该包的所有版本。
- （可选）`requires-dist`: 包的依赖项（例如，`werkzeug>=0.14`）。
- （可选）`requires-python`: 包所需的Python版本（例如，`>=3.10`）。
- （可选）`provides-extras`: 包提供的额外功能。

**默认值**: `[]`

**类型**: `list[dict]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    dependency-metadata = [
        { name = "flask", version = "1.0.0", requires-dist = ["werkzeug"], requires-python = ">=3.6" },
    ]
    ```
=== "uv.toml"

    ```toml
    dependency-metadata = [
        { name = "flask", version = "1.0.0", requires-dist = ["werkzeug"], requires-python = ">=3.6" },
    ]
    ```

---

### [`exclude-newer`](#exclude-newer) {: #exclude-newer }

限制候选包为在指定日期之前上传的包。

接受 [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339.html) 时间戳（例如，`2006-12-02T02:07:43Z`）和本地日期（例如，`2006-12-02`），使用系统配置的时区。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    exclude-newer = "2006-12-02"
    ```
=== "uv.toml"

    ```toml
    exclude-newer = "2006-12-02"
    ```

---

### [`extra-index-url`](#extra-index-url) {: #extra-index-url }

除 `--index-url` 外，额外使用的包索引 URL。

接受符合 [PEP 503](https://peps.python.org/pep-0503/) 的仓库（简单仓库 API），或具有相同格式的本地目录。

通过此标志提供的所有索引优先于由 [`index_url`](#index-url) 或 [`index`](#index) 指定的索引（`default = true`）。当提供多个索引时，较早的值具有更高的优先级。

要控制 uv 在存在多个索引时的解析策略，请参阅 [`index_strategy`](#index-strategy)。

（已弃用：请使用 `index` 代替。）

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    extra-index-url = ["https://download.pytorch.org/whl/cpu"]
    ```
=== "uv.toml"

    ```toml
    extra-index-url = ["https://download.pytorch.org/whl/cpu"]
    ```

---

### [`find-links`](#find-links) {: #find-links }

除注册表索引外，搜索候选分发的其他位置。

如果是路径，目标必须是一个目录，其中包含顶层的 wheel 文件（`.whl`）或源码分发文件（例如，`.tar.gz` 或 `.zip`）。

如果是 URL，页面必须包含指向上述格式的包文件的扁平链接列表。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    find-links = ["https://download.pytorch.org/whl/torch_stable.html"]
    ```
=== "uv.toml"

    ```toml
    find-links = ["https://download.pytorch.org/whl/torch_stable.html"]
    ```

---

### [`fork-strategy`](#fork-strategy) {: #fork-strategy }

在跨 Python 版本和平台选择给定包的多个版本时使用的策略。

默认情况下，uv 会优化为每个支持的 Python 版本（`requires-python`）选择每个包的最新版本，同时最小化跨平台选择的版本数量。

在 `fewest` 策略下，uv 将最小化每个包选择的版本数量，优先选择与更广泛的 Python 版本或平台兼容的旧版本。

**默认值**: `"requires-python"`

**可选值**:

- `"fewest"`: 优化为每个包选择最少的版本数量。如果旧版本与更广泛的 Python 版本或平台兼容，则优先选择旧版本
- `"requires-python"`: 优化为每个支持的 Python 版本选择每个包的最新支持版本

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    fork-strategy = "fewest"
    ```
=== "uv.toml"

    ```toml
    fork-strategy = "fewest"
    ```

---

### [`index`](#index) {: #index }

解析依赖时使用的包索引。

接受符合 [PEP 503](https://peps.python.org/pep-0503/) 的仓库（简单仓库 API），或具有相同格式的本地目录。

索引按定义的顺序进行考虑，因此第一个定义的索引具有最高优先级。此外，此设置提供的索引优先于通过 [`index_url`](#index-url) 或 [`extra_index_url`](#extra-index-url) 指定的任何索引。除非指定了替代的 [索引策略](#index-strategy)，否则 uv 只会考虑包含给定包的第一个索引。

如果索引标记为 `explicit = true`，它将专门用于通过 `[tool.uv.sources]` 明确选择它的依赖项，例如：

```toml
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cu121"
explicit = true

[tool.uv.sources]
torch = { index = "pytorch" }
```

如果索引标记为 `default = true`，它将被移到优先级列表的末尾，因此在解析包时赋予最低优先级。此外，将索引标记为默认值将禁用 PyPI 默认索引。

**默认值**: `"[]"`

**类型**: `dict`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [[tool.uv.index]]
    name = "pytorch"
    url = "https://download.pytorch.org/whl/cu121"
    ```
=== "uv.toml"

    ```toml
    [[tool.uv.index]]
    name = "pytorch"
    url = "https://download.pytorch.org/whl/cu121"
    ```

---

### [`index-strategy`](#index-strategy) {: #index-strategy }

解析多个索引URL时使用的策略。

默认情况下，uv会在找到给定包的第一个索引处停止，并将解析限制在该第一个索引上可用的包（`first-index`）。这可以防止“依赖混淆”攻击，即攻击者可以在备用索引上上传同名恶意包。

**默认值**: `"first-index"`

**可能的值**:

- `"first-index"`: 仅使用第一个返回匹配包的索引的结果
- `"unsafe-first-match"`: 在所有索引中搜索每个包名，在第一个索引的版本用尽后再继续下一个索引
- `"unsafe-best-match"`: 在所有索引中搜索每个包名，优先选择找到的“最佳”版本。如果一个包版本在多个索引中，则仅查看第一个索引的条目

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    index-strategy = "unsafe-best-match"
    ```
=== "uv.toml"

    ```toml
    index-strategy = "unsafe-best-match"
    ```

---

### [`index-url`](#index-url) {: #index-url }

Python包索引的URL（默认：<https://pypi.org/simple>）。

接受符合[PEP 503](https://peps.python.org/pep-0503/)（简单仓库API）的仓库，或相同格式的本地目录。

此设置提供的索引优先级低于通过[`extra_index_url`](#extra-index-url)或[`index`](#index)指定的任何索引。

（已弃用：请使用`index`代替。）

**默认值**: `"https://pypi.org/simple"`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    index-url = "https://test.pypi.org/simple"
    ```
=== "uv.toml"

    ```toml
    index-url = "https://test.pypi.org/simple"
    ```

---

### [`keyring-provider`](#keyring-provider) {: #keyring-provider }

尝试使用`keyring`进行索引URL的身份验证。

目前仅支持`--keyring-provider subprocess`，它配置uv使用`keyring` CLI处理身份验证。

**默认值**: `"disabled"`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    keyring-provider = "subprocess"
    ```
=== "uv.toml"

    ```toml
    keyring-provider = "subprocess"
    ```

---

### [`link-mode`](#link-mode) {: #link-mode }

从全局缓存安装包时使用的方法。

在macOS上默认为`clone`（也称为写时复制），在Linux和Windows上默认为`hardlink`。

**默认值**: `"clone" (macOS) 或 "hardlink" (Linux, Windows)`

**可能的值**:

- `"clone"`: 从wheel中克隆（即写时复制）包到`site-packages`目录
- `"copy"`: 从wheel中复制包到`site-packages`目录
- `"hardlink"`: 从wheel中硬链接包到`site-packages`目录
- `"symlink"`: 从wheel中符号链接包到`site-packages`目录

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    link-mode = "copy"
    ```
=== "uv.toml"

    ```toml
    link-mode = "copy"
    ```

---

### [`native-tls`](#native-tls) {: #native-tls }

是否从平台的本地证书存储中加载TLS证书。

默认情况下，uv从捆绑的`webpki-roots` crate中加载证书。`webpki-roots`是来自Mozilla的一组可靠信任根，将其包含在uv中可以提高可移植性和性能（尤其是在macOS上）。

然而，在某些情况下，您可能希望使用平台的本地证书存储，特别是如果您依赖包含在系统证书存储中的公司信任根（例如，用于强制代理）。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    native-tls = true
    ```
=== "uv.toml"

    ```toml
    native-tls = true
    ```

---

### [`no-binary`](#no-binary) {: #no-binary }

不安装预构建的wheel。

给定的包将从源代码构建和安装。解析器仍将使用预构建的wheel来提取包元数据（如果可用）。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-binary = true
    ```
=== "uv.toml"

    ```toml
    no-binary = true
    ```

---

### [`no-binary-package`](#no-binary-package) {: #no-binary-package }

不安装特定包的预构建wheel。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-binary-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    no-binary-package = ["ruff"]
    ```

---

### [`no-build`](#no-build) {: #no-build }

不构建源码分发包。

启用时，解析过程将不会运行任意 Python 代码。已构建的源码分发包的缓存 wheel 将被重用，但需要构建分发包的操作将退出并报错。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-build = true
    ```
=== "uv.toml"

    ```toml
    no-build = true
    ```

---

### [`no-build-isolation`](#no-build-isolation) {: #no-build-isolation }

构建源码分发包时禁用隔离。

假设 [PEP 518](https://peps.python.org/pep-0518/) 指定的构建依赖已安装。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-build-isolation = true
    ```
=== "uv.toml"

    ```toml
    no-build-isolation = true
    ```

---

### [`no-build-isolation-package`](#no-build-isolation-package) {: #no-build-isolation-package }

为特定包构建源码分发包时禁用隔离。

假设该包的构建依赖已按照 [PEP 518](https://peps.python.org/pep-0518/) 指定安装。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-build-isolation-package = ["package1", "package2"]
    ```
=== "uv.toml"

    ```toml
    no-build-isolation-package = ["package1", "package2"]
    ```

---

### [`no-build-package`](#no-build-package) {: #no-build-package }

不为特定包构建源码分发包。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-build-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    no-build-package = ["ruff"]
    ```

---

### [`no-cache`](#no-cache) {: #no-cache }

避免读取或写入缓存，而是在操作期间使用临时目录。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-cache = true
    ```
=== "uv.toml"

    ```toml
    no-cache = true
    ```

---

### [`no-index`](#no-index) {: #no-index }

忽略所有注册表索引（例如 PyPI），仅依赖直接 URL 依赖项和通过 `--find-links` 提供的依赖项。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-index = true
    ```
=== "uv.toml"

    ```toml
    no-index = true
    ```

---

### [`no-sources`](#no-sources) {: #no-sources }

忽略 `tool.uv.sources` 表在解析依赖项时。用于锁定符合标准的、可发布的包元数据，而不是使用任何本地或 Git 源。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    no-sources = true
    ```
=== "uv.toml"

    ```toml
    no-sources = true
    ```

---

### [`offline`](#offline) {: #offline }

禁用网络访问，仅依赖本地缓存的数据和本地可用的文件。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    offline = true
    ```
=== "uv.toml"

    ```toml
    offline = true
    ```

---

### [`prerelease`](#prerelease) {: #prerelease }

考虑预发布版本时使用的策略。

默认情况下，uv 会接受仅发布预发布版本的包的预发布版本，以及声明中包含显式预发布标记的第一方依赖项（`if-necessary-or-explicit`）。

**默认值**: `"if-necessary-or-explicit"`

**可选值**:

- `"disallow"`: 禁止所有预发布版本
- `"allow"`: 允许所有预发布版本
- `"if-necessary"`: 如果包的所有版本都是预发布版本，则允许预发布版本
- `"explicit"`: 对于版本要求中包含显式预发布标记的第一方包，允许预发布版本
- `"if-necessary-or-explicit"`: 如果包的所有版本都是预发布版本，或者包的版本要求中包含显式预发布标记，则允许预发布版本

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    prerelease = "allow"
    ```
=== "uv.toml"

    ```toml
    prerelease = "allow"
    ```

---

### [`preview`](#preview) {: #preview }

是否启用实验性的预览功能。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    preview = true
    ```
=== "uv.toml"

    ```toml
    preview = true
    ```

---

### [`publish-url`](#publish-url) {: #publish-url }

用于发布包到Python包索引的URL（默认为：
<https://upload.pypi.org/legacy/>）。

**默认值**: `"https://upload.pypi.org/legacy/"`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    publish-url = "https://test.pypi.org/legacy/"
    ```
=== "uv.toml"

    ```toml
    publish-url = "https://test.pypi.org/legacy/"
    ```

---

### [`pypy-install-mirror`](#pypy-install-mirror) {: #pypy-install-mirror }

用于下载托管PyPy安装的镜像URL。

默认情况下，托管的PyPy安装从[downloads.python.org](https://downloads.python.org/)下载。
此变量可以设置为镜像URL，以使用不同的PyPy安装源。
提供的URL将替换`https://downloads.python.org/pypy`，例如在`https://downloads.python.org/pypy/pypy3.8-v7.3.7-osx64.tar.bz2`中。

可以通过使用`file://` URL方案从本地目录读取分发文件。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    pypy-install-mirror = "https://downloads.python.org/pypy"
    ```
=== "uv.toml"

    ```toml
    pypy-install-mirror = "https://downloads.python.org/pypy"
    ```

---

### [`python-downloads`](#python-downloads) {: #python-downloads }

是否允许下载Python。

**默认值**: `"automatic"`

**可选值**:

- `"automatic"`: 在需要时自动下载托管的Python安装
- `"manual"`: 不自动下载托管的Python安装；需要显式安装
- `"never"`: 永远不允许下载Python

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    python-downloads = "manual"
    ```
=== "uv.toml"

    ```toml
    python-downloads = "manual"
    ```

---

### [`python-install-mirror`](#python-install-mirror) {: #python-install-mirror }

用于下载托管Python安装的镜像URL。

默认情况下，托管的Python安装从[`python-build-standalone`](https://github.com/astral-sh/python-build-standalone)下载。
此变量可以设置为镜像URL，以使用不同的Python安装源。
提供的URL将替换`https://github.com/astral-sh/python-build-standalone/releases/download`，例如在`https://github.com/astral-sh/python-build-standalone/releases/download/20240713/cpython-3.12.4%2B20240713-aarch64-apple-darwin-install_only.tar.gz`中。

可以通过使用`file://` URL方案从本地目录读取分发文件。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    python-install-mirror = "https://github.com/astral-sh/python-build-standalone/releases/download"
    ```
=== "uv.toml"

    ```toml
    python-install-mirror = "https://github.com/astral-sh/python-build-standalone/releases/download"
    ```

---

### [`python-preference`](#python-preference) {: #python-preference }

是优先使用系统上已有的Python安装，还是优先使用uv下载和安装的Python。

**默认值**: `"managed"`

**可选值**:

- `"only-managed"`: 仅使用托管的Python安装；从不使用系统Python安装
- `"managed"`: 优先使用托管的Python安装，而不是系统Python安装
- `"system"`: 优先使用系统Python安装，而不是托管的Python安装
- `"only-system"`: 仅使用系统Python安装；从不使用托管的Python安装

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    python-preference = "managed"
    ```
=== "uv.toml"

    ```toml
    python-preference = "managed"
    ```

---

### [`reinstall`](#reinstall) {: #reinstall }

重新安装所有包，无论是否已安装。隐含`refresh`。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    reinstall = true
    ```
=== "uv.toml"

    ```toml
    reinstall = true
    ```

---

### [`reinstall-package`](#reinstall-package) {: #reinstall-package }

重新安装特定包，无论是否已安装。隐含`refresh-package`。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    reinstall-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    reinstall-package = ["ruff"]
    ```

---

### [`required-version`](#required-version) {: #required-version }

强制执行uv版本要求。

如果运行时的uv版本不符合要求，uv将退出并报错。

接受[PEP 440](https://peps.python.org/pep-0440/)规范，如`==0.5.0`或`>=0.5.0`。

**默认值**: `null`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    required-version = ">=0.5.0"
    ```
=== "uv.toml"

    ```toml
    required-version = ">=0.5.0"
    ```

---

### [`resolution`](#resolution) {: #resolution }

在选择给定包要求的不同兼容版本时使用的策略。

默认情况下，uv将使用每个包的最新兼容版本（`highest`）。

**默认值**: `"highest"`

**可选值**:

- `"highest"`: 解析每个包的最高兼容版本
- `"lowest"`: 解析每个包的最低兼容版本
- `"lowest-direct"`: 解析任何直接依赖的最低兼容版本，以及任何传递依赖的最高兼容版本

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    resolution = "lowest-direct"
    ```
=== "uv.toml"

    ```toml
    resolution = "lowest-direct"
    ```

---

### [`trusted-publishing`](#trusted-publishing) {: #trusted-publishing }

通过GitHub Actions配置可信发布。

默认情况下，uv在GitHub Actions中运行时检查可信发布，但如果未配置或工作流没有足够的权限（例如，来自fork的pull request），则忽略它。

**默认值**: `automatic`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    trusted-publishing = "always"
    ```
=== "uv.toml"

    ```toml
    trusted-publishing = "always"
    ```

---

### [`upgrade`](#upgrade) {: #upgrade }

允许包升级，忽略任何现有输出文件中的固定版本。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    upgrade = true
    ```
=== "uv.toml"

    ```toml
    upgrade = true
    ```

---

### [`upgrade-package`](#upgrade-package) {: #upgrade-package }

允许特定包的升级，忽略任何现有输出文件中的固定版本。

接受独立的包名（`ruff`）和版本规范（`ruff<0.5.0`）。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv]
    upgrade-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    upgrade-package = ["ruff"]
    ```

---

### `pip`

特定于`uv pip`命令行的设置。

这些值在运行`uv pip`命名空间之外的命令时将被忽略（例如，`uv lock`，`uvx`）。

#### [`all-extras`](#pip_all-extras) {: #pip_all-extras }
<span id="all-extras"></span>

包含所有可选依赖项。

仅适用于`pyproject.toml`、`setup.py`和`setup.cfg`源。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    all-extras = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    all-extras = true
    ```

---

#### [`allow-empty-requirements`](#pip_allow-empty-requirements) {: #pip_allow-empty-requirements }
<span id="allow-empty-requirements"></span>

允许使用空的 requirements 执行 `uv pip sync`，这将清除环境中的所有包。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    allow-empty-requirements = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    allow-empty-requirements = true
    ```

---

#### [`annotation-style`](#pip_annotation-style) {: #pip_annotation-style }
<span id="annotation-style"></span>

输出文件中包含的注释样式，用于指示每个包的来源。

**默认值**: `"split"`

**可选值**:

- `"line"`: 将注释渲染为单行，用逗号分隔
- `"split"`: 将每个注释渲染为单独一行

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    annotation-style = "line"
    ```
=== "uv.toml"

    ```toml
    [pip]
    annotation-style = "line"
    ```

---

#### [`break-system-packages`](#pip_break-system-packages) {: #pip_break-system-packages }
<span id="break-system-packages"></span>

允许 uv 修改 `EXTERNALLY-MANAGED` 的 Python 安装。

警告：`--break-system-packages` 旨在用于持续集成（CI）环境中，当安装到由外部包管理器（如 `apt`）管理的 Python 安装时。应谨慎使用，因为此类 Python 安装明确建议不要由其他包管理器（如 uv 或 pip）进行修改。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    break-system-packages = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    break-system-packages = true
    ```

---

#### [`compile-bytecode`](#pip_compile-bytecode) {: #pip_compile-bytecode }
<span id="compile-bytecode"></span>

在安装后将 Python 文件编译为字节码。

默认情况下，uv 不会将 Python (`.py`) 文件编译为字节码 (`__pycache__/*.pyc`)；而是在首次导入模块时进行延迟编译。对于启动时间至关重要的用例，如 CLI 应用程序和 Docker 容器，可以启用此选项，以较长的安装时间换取更快的启动时间。

启用后，uv 将处理整个 site-packages 目录（包括当前操作未修改的包）以确保一致性。与 pip 一样，它也会忽略错误。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    compile-bytecode = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    compile-bytecode = true
    ```

---

#### [`config-settings`](#pip_config-settings) {: #pip_config-settings }
<span id="config-settings"></span>

传递给 [PEP 517](https://peps.python.org/pep-0517/) 构建后端的设置，以 `KEY=VALUE` 对的形式指定。

**默认值**: `{}`

**类型**: `dict`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    config-settings = { editable_mode = "compat" }
    ```
=== "uv.toml"

    ```toml
    [pip]
    config-settings = { editable_mode = "compat" }
    ```

---

#### [`custom-compile-command`](#pip_custom-compile-command) {: #pip_custom-compile-command }
<span id="custom-compile-command"></span>

在 `uv pip compile` 生成的输出文件顶部包含的自定义编译命令的注释。

用于反映封装 `uv pip compile` 的自定义构建脚本和命令。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    custom-compile-command = "./custom-uv-compile.sh"
    ```
=== "uv.toml"

    ```toml
    [pip]
    custom-compile-command = "./custom-uv-compile.sh"
    ```

---

#### [`dependency-metadata`](#pip_dependency-metadata) {: #pip_dependency-metadata }
<span id="dependency-metadata"></span>

项目的依赖项（直接或传递）的预定义静态元数据。当提供时，解析器将使用指定的元数据，而不是查询注册表或从源代码构建相关包。

元数据应遵循 [Metadata 2.3](https://packaging.python.org/en/latest/specifications/core-metadata/) 标准，但仅以下字段会被使用：

- `name`: 包的名称。
- (可选) `version`: 包的版本。如果省略，元数据将应用于该包的所有版本。
- (可选) `requires-dist`: 包的依赖项（例如，`werkzeug>=0.14`）。
- (可选) `requires-python`: 包所需的 Python 版本（例如，`>=3.10`）。
- (可选) `provides-extras`: 包提供的 extras。

**默认值**: `[]`

**类型**: `list[dict]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    dependency-metadata = [
        { name = "flask", version = "1.0.0", requires-dist = ["werkzeug"], requires-python = ">=3.6" },
    ]
    ```
=== "uv.toml"

    ```toml
    [pip]
    dependency-metadata = [
        { name = "flask", version = "1.0.0", requires-dist = ["werkzeug"], requires-python = ">=3.6" },
    ]
    ```

---

#### [`emit-build-options`](#pip_emit-build-options) {: #pip_emit-build-options }
<span id="emit-build-options"></span>

在 `uv pip compile` 生成的输出文件中包含 `--no-binary` 和 `--only-binary` 条目。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    emit-build-options = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    emit-build-options = true
    ```

---

#### [`emit-find-links`](#pip_emit-find-links) {: #pip_emit-find-links }
<span id="emit-find-links"></span>

在 `uv pip compile` 生成的输出文件中包含 `--find-links` 条目。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    emit-find-links = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    emit-find-links = true
    ```

---

#### [`emit-index-annotation`](#pip_emit-index-annotation) {: #pip_emit-index-annotation }
<span id="emit-index-annotation"></span>

包含注释，指示用于解析每个包的索引（例如，`# from https://pypi.org/simple`）。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    emit-index-annotation = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    emit-index-annotation = true
    ```

---

#### [`emit-index-url`](#pip_emit-index-url) {: #pip_emit-index-url }
<span id="emit-index-url"></span>

在 `uv pip compile` 生成的输出文件中包含 `--index-url` 和 `--extra-index-url` 条目。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    emit-index-url = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    emit-index-url = true
    ```

---

#### [`emit-marker-expression`](#pip_emit-marker-expression) {: #pip_emit-marker-expression }
<span id="emit-marker-expression"></span>

是否发出标记字符串，指示锁定依赖项集有效的条件。

即使标记表达式为 false，锁定的依赖项也可能有效，但当表达式为 true 时，可以确定需求是正确的。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    emit-marker-expression = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    emit-marker-expression = true
    ```

---

#### [`exclude-newer`](#pip_exclude-newer) {: #pip_exclude-newer }
<span id="exclude-newer"></span>

将候选包限制为在给定时间点之前上传的包。

接受 [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339.html) 的超集（例如，`2006-12-02T02:07:43Z`）。需要完整的时间戳以确保解析器在不同时区中表现一致。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    exclude-newer = "2006-12-02T02:07:43Z"
    ```
=== "uv.toml"

    ```toml
    [pip]
    exclude-newer = "2006-12-02T02:07:43Z"
    ```

---

#### [`extra`](#pip_extra) {: #pip_extra }
<span id="extra"></span>

包含指定 extra 的可选依赖项；可以多次提供。

仅适用于 `pyproject.toml`、`setup.py` 和 `setup.cfg` 源。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    extra = ["dev", "docs"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    extra = ["dev", "docs"]
    ```

---

#### [`extra-index-url`](#pip_extra-index-url) {: #pip_extra-index-url }
<span id="extra-index-url"></span>

除了 `--index-url` 之外，使用的额外包索引的 URL。

接受符合 [PEP 503](https://peps.python.org/pep-0503/)（简单仓库 API）的仓库，或按相同格式布局的本地目录。

通过此标志提供的所有索引优先于 [`index_url`](#index-url) 指定的索引。当提供多个索引时，较早的值优先。

要控制 uv 在存在多个索引时的解析策略，请参见 [`index_strategy`](#index-strategy)。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    extra-index-url = ["https://download.pytorch.org/whl/cpu"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    extra-index-url = ["https://download.pytorch.org/whl/cpu"]
    ```

---

#### [`find-links`](#pip_find-links) {: #pip_find-links }
<span id="find-links"></span>

除了在注册表索引中找到的候选分发位置外，搜索候选分发的其他位置。

如果是一个路径，目标必须是一个目录，该目录包含位于顶层的wheel文件（`.whl`）或
源代码分发文件（例如，`.tar.gz`或`.zip`）。

如果是一个URL，页面必须包含一个指向符合上述格式的包文件的链接的扁平列表。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    find-links = ["https://download.pytorch.org/whl/torch_stable.html"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    find-links = ["https://download.pytorch.org/whl/torch_stable.html"]
    ```

---

#### [`fork-strategy`](#pip_fork-strategy) {: #pip_fork-strategy }
<span id="fork-strategy"></span>

在跨Python版本和平台选择给定包的多个版本时使用的策略。

默认情况下，uv会优化为每个支持的Python版本（`requires-python`）选择每个包的最新版本，同时最小化跨平台选择的版本数量。

在`fewest`策略下，uv将最小化每个包选择的版本数量，优先选择与更广泛的Python版本或平台兼容的旧版本。

**默认值**: `"requires-python"`

**可能的值**:

- `"fewest"`: 优化为每个包选择最少的版本数量。如果旧版本与更广泛的Python版本或平台兼容，则优先选择旧版本
- `"requires-python"`: 优化为每个支持的Python版本选择每个包的最新支持版本

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    fork-strategy = "fewest"
    ```
=== "uv.toml"

    ```toml
    [pip]
    fork-strategy = "fewest"
    ```

---

#### [`generate-hashes`](#pip_generate-hashes) {: #pip_generate-hashes }
<span id="generate-hashes"></span>

在输出文件中包含分发哈希值。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    generate-hashes = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    generate-hashes = true
    ```

---

#### [`index-strategy`](#pip_index-strategy) {: #pip_index-strategy }
<span id="index-strategy"></span>

在解析多个索引URL时使用的策略。

默认情况下，uv会在找到给定包的第一个索引处停止，并将解析限制在该第一个索引上可用的内容（`first-index`）。这可以防止“依赖混淆”攻击，即攻击者可以在备用索引上上传同名的恶意包。

**默认值**: `"first-index"`

**可能的值**:

- `"first-index"`: 仅使用第一个返回给定包名匹配结果的索引
- `"unsafe-first-match"`: 在所有索引中搜索每个包名，在第一个索引的版本用尽后再继续搜索下一个索引
- `"unsafe-best-match"`: 在所有索引中搜索每个包名，优先选择找到的“最佳”版本。如果一个包版本在多个索引中，则仅查看第一个索引的条目

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    index-strategy = "unsafe-best-match"
    ```
=== "uv.toml"

    ```toml
    [pip]
    index-strategy = "unsafe-best-match"
    ```

---

#### [`index-url`](#pip_index-url) {: #pip_index-url }
<span id="index-url"></span>

Python包索引的URL（默认：<https://pypi.org/simple>）。

接受符合[PEP 503](https://peps.python.org/pep-0503/)（简单仓库API）的仓库，或具有相同格式的本地目录。

通过此设置提供的索引优先级低于通过[`extra_index_url`](#extra-index-url)指定的任何索引。

**默认值**: `"https://pypi.org/simple"`

**类型**: `str`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    index-url = "https://test.pypi.org/simple"
    ```
=== "uv.toml"

    ```toml
    [pip]
    index-url = "https://test.pypi.org/simple"
    ```

---

#### [`keyring-provider`](#pip_keyring-provider) {: #pip_keyring-provider }
<span id="keyring-provider"></span>

尝试使用`keyring`进行索引URL的身份验证。

目前仅支持`--keyring-provider subprocess`，该选项配置uv使用`keyring` CLI处理身份验证。

**默认值**: `disabled`

**类型**: `str`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    keyring-provider = "subprocess"
    ```
=== "uv.toml"

    ```toml
    [pip]
    keyring-provider = "subprocess"
    ```

---

#### [`link-mode`](#pip_link-mode) {: #pip_link-mode }
<span id="link-mode"></span>

从全局缓存安装包时使用的方法。

在macOS上默认为`clone`（也称为写时复制），在Linux和Windows上默认为`hardlink`。

**默认值**: `"clone" (macOS) 或 "hardlink" (Linux, Windows)`

**可能的值**:

- `"clone"`: 从wheel克隆（即写时复制）包到`site-packages`目录
- `"copy"`: 从wheel复制包到`site-packages`目录
- `"hardlink"`: 从wheel硬链接包到`site-packages`目录
- `"symlink"`: 从wheel符号链接包到`site-packages`目录

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    link-mode = "copy"
    ```
=== "uv.toml"

    ```toml
    [pip]
    link-mode = "copy"
    ```

---

#### [`no-annotate`](#pip_no-annotate) {: #pip_no-annotate }
<span id="no-annotate"></span>

在`uv pip compile`生成的输出文件中排除指示每个包来源的注释。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-annotate = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-annotate = true
    ```

---

#### [`no-binary`](#pip_no-binary) {: #pip_no-binary }
<span id="no-binary"></span>

不安装预构建的wheel。

给定的包将从源代码构建和安装。解析器仍将使用预构建的wheel来提取包元数据（如果可用）。

可以提供多个包。使用`:all:`禁用所有包的二进制文件。使用`:none:`清除之前指定的包。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-binary = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-binary = ["ruff"]
    ```

---

#### [`no-build`](#pip_no-build) {: #pip_no-build }
<span id="no-build"></span>

不构建源代码分发。

启用后，解析将不会运行任意Python代码。将重用已构建的源代码分发的缓存wheel，但需要构建分发的操作将退出并报错。

`--only-binary :all:`的别名。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-build = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-build = true
    ```

---

#### [`no-build-isolation`](#pip_no-build-isolation) {: #pip_no-build-isolation }
<span id="no-build-isolation"></span>

在构建源代码分发时禁用隔离。

假设[PEP 518](https://peps.python.org/pep-0518/)指定的构建依赖项已安装。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-build-isolation = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-build-isolation = true
    ```

---

#### [`no-build-isolation-package`](#pip_no-build-isolation-package) {: #pip_no-build-isolation-package }
<span id="no-build-isolation-package"></span>

在构建特定包的源代码分发时禁用隔离。

假设该包的构建依赖项已由[PEP 518](https://peps.python.org/pep-0518/)指定并已安装。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-build-isolation-package = ["package1", "package2"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-build-isolation-package = ["package1", "package2"]
    ```

---

#### [`no-deps`](#pip_no-deps) {: #pip_no-deps }
<span id="no-deps"></span>

忽略包依赖关系，仅将命令行中明确列出的包添加到生成的requirements文件中。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-deps = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-deps = true
    ```

---

#### [`no-emit-package`](#pip_no-emit-package) {: #pip_no-emit-package }
<span id="no-emit-package"></span>

指定要从输出解析中省略的包。其依赖项仍将包含在解析结果中。相当于 pip-compile 的 `--unsafe-package` 选项。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-emit-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-emit-package = ["ruff"]
    ```

---

#### [`no-extra`](#pip_no-extra) {: #pip_no-extra }
<span id="no-extra"></span>

如果指定了 `all-extras`，则排除指定的可选依赖项。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    all-extras = true
    no-extra = ["dev", "docs"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    all-extras = true
    no-extra = ["dev", "docs"]
    ```

---

#### [`no-header`](#pip_no-header) {: #pip_no-header }
<span id="no-header"></span>

排除 `uv pip compile` 生成的输出文件顶部的注释头。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-header = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-header = true
    ```

---

#### [`no-index`](#pip_no-index) {: #pip_no-index }
<span id="no-index"></span>

忽略所有注册表索引（例如 PyPI），转而依赖直接 URL 依赖项和通过 `--find-links` 提供的依赖项。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-index = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-index = true
    ```

---

#### [`no-sources`](#pip_no-sources) {: #pip_no-sources }
<span id="no-sources"></span>

在解析依赖项时忽略 `tool.uv.sources` 表。用于锁定符合标准的、可发布的包元数据，而不是使用任何本地或 Git 源。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-sources = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-sources = true
    ```

---

#### [`no-strip-extras`](#pip_no-strip-extras) {: #pip_no-strip-extras }
<span id="no-strip-extras"></span>

在输出文件中包含 extras。

默认情况下，uv 会去除 extras，因为 extras 引入的任何包已经作为依赖项直接包含在输出文件中。此外，使用 `--no-strip-extras` 生成的输出文件不能用作 `install` 和 `sync` 调用中的约束文件。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-strip-extras = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-strip-extras = true
    ```

---

#### [`no-strip-markers`](#pip_no-strip-markers) {: #pip_no-strip-markers }
<span id="no-strip-markers"></span>

在 `uv pip compile` 生成的输出文件中包含环境标记。

默认情况下，uv 会去除环境标记，因为 `compile` 生成的解析结果仅保证对目标环境正确。

**默认值**: `false`

**类型**: `bool`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    no-strip-markers = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    no-strip-markers = true
    ```

---

#### [`only-binary`](#pip_only-binary) {: #pip_only-binary }
<span id="only-binary"></span>

仅使用预构建的 wheels；不构建源码分发包。

启用后，解析将不会运行给定包的代码。已构建的源码分发的缓存 wheels 将被重用，但需要构建分发的操作将退出并报错。

可以提供多个包。使用 `:all:` 禁用所有包的二进制文件。使用 `:none:` 清除之前指定的包。

**默认值**: `[]`

**类型**: `list[str]`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    only-binary = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    only-binary = ["ruff"]
    ```

---

#### [`output-file`](#pip_output-file) {: #pip_output-file }
<span id="output-file"></span>

将 `uv pip compile` 生成的需求写入指定的 `requirements.txt` 文件。

如果文件已存在，则在解析依赖项时将优先使用现有版本，除非同时指定了 `--upgrade`。

**默认值**: `None`

**类型**: `str`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    output-file = "requirements.txt"
    ```
=== "uv.toml"

    ```toml
    [pip]
    output-file = "requirements.txt"
    ```

---

#### [`prefix`](#pip_prefix) {: #pip_prefix }
<span id="prefix"></span>

将包安装到指定目录下的 `lib`、`bin` 和其他顶级文件夹中，就像该位置存在一个虚拟环境一样。

通常，建议使用 `--python` 安装到备用环境中，因为通过 `--prefix` 安装的脚本和其他工件将引用安装的解释器，而不是添加到 `--prefix` 目录中的任何解释器，从而使它们不可移植。

**默认值**: `None`

**类型**: `str`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    prefix = "./prefix"
    ```
=== "uv.toml"

    ```toml
    [pip]
    prefix = "./prefix"
    ```

---

#### [`prerelease`](#pip_prerelease) {: #pip_prerelease }
<span id="prerelease"></span>

考虑预发布版本时使用的策略。

默认情况下，uv 将接受仅发布预发布版本的包的预发布版本，以及在其声明的版本说明符中包含显式预发布标记的第一方需求（`if-necessary-or-explicit`）。

**默认值**: `"if-necessary-or-explicit"`

**可能的值**:

- `"disallow"`: 禁止所有预发布版本
- `"allow"`: 允许所有预发布版本
- `"if-necessary"`: 如果包的所有版本都是预发布版本，则允许预发布版本
- `"explicit"`: 对于在其版本需求中具有显式预发布标记的第一方包，允许预发布版本
- `"if-necessary-or-explicit"`: 如果包的所有版本都是预发布版本，或者包在其版本需求中具有显式预发布标记，则允许预发布版本

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    prerelease = "allow"
    ```
=== "uv.toml"

    ```toml
    [pip]
    prerelease = "allow"
    ```

---

#### [`python`](#pip_python) {: #pip_python }
<span id="python"></span>

用于安装包的 Python 解释器。

默认情况下，uv 会安装到当前工作目录或任何父目录中的虚拟环境中。`--python` 选项允许您指定不同的解释器，适用于持续集成（CI）环境或其他自动化工作流。

支持的格式：
- `3.10` 在 Windows 上查找注册表中安装的 Python 3.10（参见 `py --list-paths`），或在 Linux 和 macOS 上查找 `python3.10`。
- `python3.10` 或 `python.exe` 在 `PATH` 中查找具有给定名称的二进制文件。
- `/home/ferris/.local/bin/python3.10` 使用给定路径的确切 Python。

**默认值**: `None`

**类型**: `str`

**示例用法**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    python = "3.10"
    ```
=== "uv.toml"

    ```toml
    [pip]
    python = "3.10"
    ```

---

#### [`python-platform`](#pip_python-platform) {: #pip_python-platform }
<span id="python-platform"></span>

用于解析需求的平台。

表示为“目标三元组”，一个描述目标平台的字符串，包括其 CPU、供应商和操作系统名称，例如 `x86_64-unknown-linux-gnu` 或 `aarch64-apple-darwin`。

**默认值**: `None`

**类型**: `str`

**示例用法**:
=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    python-platform = "x86_64-unknown-linux-gnu"
    ```
=== "uv.toml"

    ```toml
    [pip]
    python-platform = "x86_64-unknown-linux-gnu"
    ```

---

#### [`python-version`](#pip_python-version) {: #pip_python-version }
<span id="python-version"></span>

应支持的最低 Python 版本（例如，`3.8` 或 `3.8.17`）。

如果省略了补丁版本，则假定为最低补丁版本。例如，`3.8` 映射为 `3.8.0`。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    python-version = "3.8"
    ```
=== "uv.toml"

    ```toml
    [pip]
    python-version = "3.8"
    ```

---

#### [`reinstall`](#pip_reinstall) {: #pip_reinstall }
<span id="reinstall"></span>

重新安装所有包，无论它们是否已安装。隐含 `refresh`。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    reinstall = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    reinstall = true
    ```

---

#### [`reinstall-package`](#pip_reinstall-package) {: #pip_reinstall-package }
<span id="reinstall-package"></span>

重新安装特定包，无论其是否已安装。隐含 `refresh-package`。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    reinstall-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    reinstall-package = ["ruff"]
    ```

---

#### [`require-hashes`](#pip_require-hashes) {: #pip_require-hashes }
<span id="require-hashes"></span>

要求每个需求都有匹配的哈希值。

哈希检查模式是全有或全无的。如果启用，_所有_ 需求都必须提供相应的哈希值或哈希值集合。此外，如果启用，_所有_ 需求都必须固定到确切的版本（例如，`==1.0.0`），或者通过直接 URL 指定。

哈希检查模式引入了许多额外的约束：

- 不支持 Git 依赖。
- 不支持可编辑安装。
- 不支持本地依赖，除非它们指向特定的 wheel（`.whl`）或源存档（`.zip`、`.tar.gz`），而不是目录。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    require-hashes = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    require-hashes = true
    ```

---

#### [`resolution`](#pip_resolution) {: #pip_resolution }
<span id="resolution"></span>

选择不同兼容版本时的策略。

默认情况下，uv 将使用每个包的最新兼容版本（`highest`）。

**默认值**: `"highest"`

**可能的值**:

- `"highest"`: 解析每个包的最高兼容版本
- `"lowest"`: 解析每个包的最低兼容版本
- `"lowest-direct"`: 解析任何直接依赖的最低兼容版本，以及任何传递依赖的最高兼容版本

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    resolution = "lowest-direct"
    ```
=== "uv.toml"

    ```toml
    [pip]
    resolution = "lowest-direct"
    ```

---

#### [`strict`](#pip_strict) {: #pip_strict }
<span id="strict"></span>

验证 Python 环境，以检测缺少依赖项和其他问题。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    strict = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    strict = true
    ```

---

#### [`system`](#pip_system) {: #pip_system }
<span id="system"></span>

将包安装到系统 Python 环境中。

默认情况下，uv 会安装到当前工作目录或任何父目录中的虚拟环境中。`--system` 选项指示 uv 改为使用系统 `PATH` 中找到的第一个 Python。

警告：`--system` 旨在用于持续集成（CI）环境，应谨慎使用，因为它可能会修改系统 Python 安装。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    system = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    system = true
    ```

---

#### [`target`](#pip_target) {: #pip_target }
<span id="target"></span>

将包安装到指定目录，而不是虚拟或系统 Python 环境中。包将安装在目录的顶层。

**默认值**: `None`

**类型**: `str`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    target = "./target"
    ```
=== "uv.toml"

    ```toml
    [pip]
    target = "./target"
    ```

---

#### [`universal`](#pip_universal) {: #pip_universal }
<span id="universal"></span>

执行通用解析，尝试生成一个兼容所有操作系统、架构和 Python 实现的单一 `requirements.txt` 输出文件。

在通用模式下，当前 Python 版本（或用户提供的 `--python-version`）将被视为下限。例如，`--universal --python-version 3.7` 将为 Python 3.7 及更高版本生成通用解析。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    universal = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    universal = true
    ```

---

#### [`upgrade`](#pip_upgrade) {: #pip_upgrade }
<span id="upgrade"></span>

允许包升级，忽略任何现有输出文件中的固定版本。

**默认值**: `false`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    upgrade = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    upgrade = true
    ```

---

#### [`upgrade-package`](#pip_upgrade-package) {: #pip_upgrade-package }
<span id="upgrade-package"></span>

允许特定包升级，忽略任何现有输出文件中的固定版本。

接受独立的包名（`ruff`）和版本说明符（`ruff<0.5.0`）。

**默认值**: `[]`

**类型**: `list[str]`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    upgrade-package = ["ruff"]
    ```
=== "uv.toml"

    ```toml
    [pip]
    upgrade-package = ["ruff"]
    ```

---

#### [`verify-hashes`](#pip_verify-hashes) {: #pip_verify-hashes }
<span id="verify-hashes"></span>

验证需求文件中提供的任何哈希值。

与 `--require-hashes` 不同，`--verify-hashes` 不要求所有需求都有哈希值；相反，它只会验证那些包含哈希值的需求。

**默认值**: `true`

**类型**: `bool`

**使用示例**:

=== "pyproject.toml"

    ```toml
    [tool.uv.pip]
    verify-hashes = true
    ```
=== "uv.toml"

    ```toml
    [pip]
    verify-hashes = true
    ```

---

