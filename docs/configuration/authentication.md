# 认证

## Git 认证

uv 允许从 Git 安装包，并支持以下方案用于与私有仓库进行认证。

使用 SSH：

- `git+ssh://git@<hostname>/...` (例如 `git+ssh://git@github.com/astral-sh/uv`)
- `git+ssh://git@<host>/...` (例如 `git+ssh://git@github.com-key-2/astral-sh/uv`)

有关如何配置 SSH 的更多详细信息，请参阅 [GitHub SSH 文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)。

使用密码或令牌：

- `git+https://<user>:<token>@<hostname>/...` (例如 `git+https://git:github_pat_asdf@github.com/astral-sh/uv`)
- `git+https://<token>@<hostname>/...` (例如 `git+https://github_pat_asdf@github.com/astral-sh/uv`)
- `git+https://<user>@<hostname>/...` (例如 `git+https://git@github.com/astral-sh/uv`)

使用 GitHub 个人访问令牌时，用户名是任意的。GitHub 不支持直接使用密码登录，尽管其他主机可能支持。如果提供了用户名但没有凭据，系统将提示您输入凭据。

如果 URL 中没有凭据且需要认证，将查询 [Git 凭据助手](https://git-scm.com/doc/credential-helpers)。

## HTTP 认证

uv 支持在查询包注册表时通过 HTTP 进行认证。

认证可以来自以下来源，按优先级顺序：

- URL，例如 `https://<user>:<password>@<hostname>/...`
- [`.netrc`](https://everything.curl.dev/usingcurl/netrc) 配置文件
- [keyring](https://github.com/jaraco/keyring) 提供程序（需要选择启用）

如果为单个网络位置（协议、主机和端口）找到认证，它将在命令的持续时间内缓存，并用于对该网络位置的其他查询。认证不会在 uv 的不同调用之间缓存。

默认情况下启用 `.netrc` 认证，并且如果定义了 `NETRC` 环境变量，将尊重该变量，否则回退到 `~/.netrc`。

要启用基于 keyring 的认证，请将 `--keyring-provider subprocess` 命令行参数传递给 uv，或设置 `UV_KEYRING_PROVIDER=subprocess`。

认证可用于在以下上下文中指定的主机：

- `index-url`
- `extra-index-url`
- `find-links`
- `package @ https://...`

有关与 `pip` 的差异的详细信息，请参阅 [`pip` 兼容性指南](../pip/compatibility.md#registry-authentication)。

## 自定义 CA 证书

默认情况下，uv 从捆绑的 `webpki-roots` crate 加载证书。`webpki-roots` 是来自 Mozilla 的一组可靠的信任根，将它们包含在 uv 中提高了可移植性和性能（尤其是在 macOS 上，读取系统信任库会导致显著的延迟）。

然而，在某些情况下，您可能希望使用平台的原生证书库，特别是如果您依赖包含在系统证书库中的企业信任根（例如，用于强制代理）。要指示 uv 使用系统的信任库，请使用 `--native-tls` 命令行标志运行 uv，或将 `UV_NATIVE_TLS` 环境变量设置为 `true`。

如果需要直接指定证书路径（例如在 CI 中），请将 `SSL_CERT_FILE` 环境变量设置为证书包的路径，以指示 uv 使用该文件而不是系统的信任库。

如果需要客户端证书认证（mTLS），请将 `SSL_CLIENT_CERT` 环境变量设置为包含证书和私钥的 PEM 格式文件的路径。

最后，如果您使用的设置中希望信任自签名证书或以其他方式禁用证书验证，您可以通过 `allow-insecure-host` 配置选项指示 uv 允许与指定主机的非安全连接。例如，将以下内容添加到 `pyproject.toml` 将允许与 `example.com` 的非安全连接：

```toml
[tool.uv]
allow-insecure-host = ["example.com"]
```

`allow-insecure-host` 期望接收主机名（例如 `localhost`）或主机名-端口对（例如 `localhost:8080`），并且仅适用于 HTTPS 连接，因为 HTTP 连接本质上是非安全的。

请谨慎使用 `allow-insecure-host`，并仅在受信任的环境中使用，因为它可能由于缺乏证书验证而使您面临安全风险。

## 与替代包索引的认证

有关与流行的替代 Python 包索引的认证的详细信息，请参阅 [替代索引集成指南](../guides/integration/alternative-indexes.md)。