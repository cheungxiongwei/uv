# 包索引

默认情况下，uv 使用 [Python Package Index (PyPI)](https://pypi.org) 进行依赖解析和包安装。然而，uv 可以通过 `[[tool.uv.index]]` 配置选项（以及类似的命令行选项 `--index`）配置为使用其他包索引，包括私有索引。

## 定义索引

要在解析依赖时包含额外的索引，请在 `pyproject.toml` 中添加 `[[tool.uv.index]]` 条目：

```toml
[[tool.uv.index]]
# 索引的可选名称。
name = "pytorch"
# 索引的必需 URL。
url = "https://download.pytorch.org/whl/cpu"
```

索引按定义的顺序优先，因此配置文件中列出的第一个索引是解析依赖时首先查询的索引，命令行提供的索引优先于配置文件中的索引。

默认情况下，uv 将 Python Package Index (PyPI) 包含为“默认”索引，即当包在其他索引上找不到时使用的索引。要从索引列表中排除 PyPI，请在另一个索引条目上设置 `default = true`（或使用 `--default-index` 命令行选项）：

```toml
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
default = true
```

默认索引始终被视为最低优先级，无论其在索引列表中的位置如何。

索引名称只能包含字母数字字符、短划线、下划线和句点，并且必须是有效的 ASCII 字符。

在命令行上提供索引时（使用 `--index` 或 `--default-index`）或通过环境变量（`UV_INDEX` 或 `UV_DEFAULT_INDEX`），名称是可选的，但可以使用 `<name>=<url>` 语法包含，例如：

```shell
# 在命令行上。
$ uv lock --index pytorch=https://download.pytorch.org/whl/cpu
# 通过环境变量。
$ UV_INDEX=pytorch=https://download.pytorch.org/whl/cpu uv lock
```

## 将包固定到特定索引

可以通过在包的 `tool.uv.sources` 条目中指定索引来将包固定到特定索引。例如，要确保 `torch` 始终从 `pytorch` 索引安装，请在 `pyproject.toml` 中添加以下内容：

```toml
[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
```

同样，要根据平台从不同的索引拉取包，可以提供由环境标记区分的源列表：

```toml title="pyproject.toml"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = [
  { index = "pytorch-cu118", marker = "sys_platform == 'darwin'"},
  { index = "pytorch-cu124", marker = "sys_platform != 'darwin'"},
]

[[tool.uv.index]]
name = "pytorch-cu118"
url = "https://download.pytorch.org/whl/cu118"

[[tool.uv.index]]
name = "pytorch-cu124"
url = "https://download.pytorch.org/whl/cu124"
```

可以将索引标记为 `explicit = true`，以防止包从该索引安装，除非显式固定到该索引。例如，要确保 `torch` 从 `pytorch` 索引安装，但所有其他包从 PyPI 安装，请在 `pyproject.toml` 中添加以下内容：

```toml
[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

通过 `tool.uv.sources` 引用的命名索引必须在项目的 `pyproject.toml` 文件中定义；通过命令行、环境变量或用户级配置提供的索引将不会被识别。

如果索引同时标记为 `default = true` 和 `explicit = true`，它将被视为显式索引（即只能通过 `tool.uv.sources` 使用），同时也会移除 PyPI 作为默认索引。

## 跨多个索引搜索

默认情况下，uv 将在找到给定包的第一个索引处停止，并将解析限制为该索引上的版本（`first-index`）。

例如，如果通过 `[[tool.uv.index]]` 指定了内部索引，uv 的行为是，如果包存在于该内部索引上，它将始终从该内部索引安装，而不会从 PyPI 安装。其目的是防止“依赖混淆”攻击，即攻击者在 PyPI 上发布与内部包同名的恶意包，从而导致安装恶意包而不是内部包。例如，参见 2022 年 12 月的 [`torchtriton` 攻击](https://pytorch.org/blog/compromised-nightly-dependency/)。

用户可以通过 `--index-strategy` 命令行选项或 `UV_INDEX_STRATEGY` 环境变量选择其他索引行为，支持以下值：

- `first-index`（默认）：在所有索引中搜索每个包，将候选版本限制为包含该包的第一个索引中的版本。
- `unsafe-first-match`：在所有索引中搜索每个包，但优先选择第一个具有兼容版本的索引，即使其他索引上有更新的版本。
- `unsafe-best-match`：在所有索引中搜索每个包，并从候选版本的组合集中选择最佳版本。

虽然 `unsafe-best-match` 最接近 pip 的行为，但它使用户面临“依赖混淆”攻击的风险。

## 提供凭据

大多数私有注册表需要身份验证才能访问包，通常通过用户名和密码（或访问令牌）进行。

要通过提供索引进行身份验证，可以通过环境变量提供凭据，或将凭据嵌入 URL 中。

例如，给定一个名为 `internal-proxy` 的索引，需要用户名（`public`）和密码（`koala`），请在 `pyproject.toml` 中定义索引（不带凭据）：

```toml
[[tool.uv.index]]
name = "internal-proxy"
url = "https://example.com/simple"
```

然后，您可以设置 `UV_INDEX_INTERNAL_PROXY_USERNAME` 和 `UV_INDEX_INTERNAL_PROXY_PASSWORD` 环境变量，其中 `INTERNAL_PROXY` 是索引名称的大写版本，非字母数字字符替换为下划线：

```sh
export UV_INDEX_INTERNAL_PROXY_USERNAME=public
export UV_INDEX_INTERNAL_PROXY_PASSWORD=koala
```

通过环境变量提供凭据，可以避免在明文 `pyproject.toml` 文件中存储敏感信息。

或者，凭据可以直接嵌入索引定义中：

```toml
[[tool.uv.index]]
name = "internal"
url = "https://public:koala@pypi-proxy.corp.dev/simple"
```

出于安全目的，凭据永远不会存储在 `uv.lock` 文件中；因此，uv 在安装时必须能够访问经过身份验证的 URL。

## `--index-url` 和 `--extra-index-url`

除了 `[[tool.uv.index]]` 配置选项外，uv 还支持 pip 风格的 `--index-url` 和 `--extra-index-url` 命令行选项以实现兼容性，其中 `--index-url` 定义默认索引，`--extra-index-url` 定义附加索引。

这些选项可以与 `[[tool.uv.index]]` 配置选项结合使用，并遵循相同的优先级规则：

- 默认索引始终被视为最低优先级，无论是通过传统的 `--index-url` 参数、推荐的 `--default-index` 参数还是带有 `default = true` 的 `[[tool.uv.index]]` 条目定义的。
- 索引按定义的顺序查询，无论是通过传统的 `--extra-index-url` 参数、推荐的 `--index` 参数还是 `[[tool.uv.index]]` 条目定义的。

实际上，`--index-url` 和 `--extra-index-url` 可以被视为未命名的 `[[tool.uv.index]]` 条目，前者启用了 `default = true`。在这种情况下，`--index-url` 映射到 `--default-index`，`--extra-index-url` 映射到 `--index`。