---
title: Using alternative package indexes
description:
  A guide to using alternative package indexes with uv, including Azure Artifacts, Google Artifact
  Registry, AWS CodeArtifact, and more.
---

# 使用替代包索引
虽然 uv 默认使用官方的 Python 包索引 (PyPI)，但它也支持替代包索引。大多数替代索引需要各种形式的身份验证，这需要一些初始设置。

!!! important
请阅读 uv 中[使用多个索引](../../pip/compatibility.md#packages-that-exist-on-multiple-indexes)的文档 — 默认行为与 pip 不同，以防止依赖混淆攻击，但这意味着 uv 可能不会像你预期的那样找到包的版本。

## Azure Artifacts
uv 可以从 [Azure DevOps Artifacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops&tabs=nuget%2Cnugetserver) 安装包。使用[个人访问令牌](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows) (PAT) 或使用 [`keyring`](https://github.com/jaraco/keyring) 包进行交互式身份验证。

### 使用 PAT
如果有可用的 PAT（例如 [Azure 管道中的 `$(System.AccessToken)`](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml#systemaccesstoken)），可以通过 "Basic" HTTP 身份验证方案提供凭据。将 PAT 包含在 URL 的密码字段中。用户名也必须包含，但可以是任何字符串。

例如，如果令牌存储在 `$ADO_PAT` 环境变量中，可以使用以下命令设置索引 URL：
```console
$ export UV_EXTRA_INDEX_URL=https://dummy:$ADO_PAT@pkgs.dev.azure.com/{organisation}/{project}/_packaging/{feedName}/pypi/simple/
```

### 使用 `keyring`
如果没有可用的 PAT，可以使用 [`keyring`](https://github.com/jaraco/keyring) 包和 [artifacts-keyring 插件](https://github.com/Microsoft/artifacts-keyring) 进行身份验证。由于这两个包是用于 Azure Artifacts 身份验证的，因此必须从 Artifacts 以外的来源预先安装。

`artifacts-keyring` 插件封装了 [Azure Artifacts 凭据提供工具](https://github.com/microsoft/artifacts-credprovider)。该凭据提供工具支持几种不同的身份验证模式，包括交互式登录 — 有关配置的详细信息，请参阅[该工具的文档](https://github.com/microsoft/artifacts-credprovider)。

uv 仅支持在[子进程模式](https://github.com/astral-sh/uv/blob/main/PIP_COMPATIBILITY.md#registry-authentication)下使用 `keyring` 包。`keyring` 可执行文件必须在 `PATH` 中，即全局安装或在活动环境中安装。`keyring` CLI 要求 URL 中包含用户名，因此索引 URL 必须包含默认用户名 `VssSessionToken`。
```console
$ # 从公共 PyPI 预先安装 keyring 和 Artifacts 插件
$ uv tool install keyring --with artifacts-keyring
$ # 启用 keyring 身份验证
$ export UV_KEYRING_PROVIDER=subprocess
$ # 配置包含用户名的索引 URL
$ export UV_EXTRA_INDEX_URL=https://VssSessionToken@pkgs.dev.azure.com/{organisation}/{project}/_packaging/{feedName}/pypi/simple/
```

## Google Artifact Registry
uv 可以从 [Google Artifact Registry](https://cloud.google.com/artifact-registry/docs) 安装包。使用密码身份验证或 [`keyring`](https://github.com/jaraco/keyring) 包进行身份验证。

!!! note
本指南假设 `gcloud` CLI 已预先安装并设置。

### 密码身份验证
可以通过 "Basic" HTTP 身份验证方案提供凭据。将访问令牌包含在 URL 的密码字段中。用户名必须为 `oauth2accesstoken`，否则身份验证将失败。

例如，如果令牌存储在 `$ARTIFACT_REGISTRY_TOKEN` 环境变量中，可以使用以下命令设置索引 URL：
```bash
export ARTIFACT_REGISTRY_TOKEN=$(gcloud auth application-default print-access-token)
export UV_EXTRA_INDEX_URL=https://oauth2accesstoken:$ARTIFACT_REGISTRY_TOKEN@{region}-python.pkg.dev/{projectId}/{repositoryName}/simple
```

### 使用 `keyring`
你也可以使用 [`keyring`](https://github.com/jaraco/keyring) 包和 [`keyrings.google-artifactregistry-auth` 插件](https://github.com/GoogleCloudPlatform/artifact-registry-python-tools) 进行身份验证。由于这两个包是用于 Artifact Registry 身份验证的，因此必须从 Artifact Registry 以外的来源预先安装。

`artifacts-keyring` 插件封装了 [gcloud CLI](https://cloud.google.com/sdk/gcloud) 以生成短期访问令牌，安全地将其存储在系统密钥环中，并在过期时刷新它们。

uv 仅支持在[子进程模式](https://github.com/astral-sh/uv/blob/main/PIP_COMPATIBILITY.md#registry-authentication)下使用 `keyring` 包。`keyring` 可执行文件必须在 `PATH` 中，即全局安装或在活动环境中安装。`keyring` CLI 要求 URL 中包含用户名，且必须为 `oauth2accesstoken`。
```bash
# 从公共 PyPI 预先安装 keyring 和 Artifact Registry 插件
uv tool install keyring --with keyrings.google-artifactregistry-auth
# 启用 keyring 身份验证
export UV_KEYRING_PROVIDER=subprocess
# 配置包含用户名的索引 URL
export UV_EXTRA_INDEX_URL=https://oauth2accesstoken@{region}-python.pkg.dev/{projectId}/{repositoryName}/simple
```

## AWS CodeArtifact
uv 可以从 [AWS CodeArtifact](https://docs.aws.amazon.com/codeartifact/latest/ug/using-python.html) 安装包。授权令牌可以使用 `awscli` 工具获取。

!!! note
本指南假设 AWS CLI 已预先进行身份验证。

首先，为你的 CodeArtifact 仓库声明一些常量：
```bash
export AWS_DOMAIN="<your-domain>"
export AWS_ACCOUNT_ID="<your-account-id>"
export AWS_REGION="<your-region>"
export AWS_CODEARTIFACT_REPOSITORY="<your-repository>"
```

然后，从 `awscli` 获取令牌：
```bash
export AWS_CODEARTIFACT_TOKEN="$(
aws codeartifact get-authorization-token \
--domain $AWS_DOMAIN \
--domain-owner $AWS_ACCOUNT_ID \
--query authorizationToken \
--output text
)"
```

并配置索引 URL：
```bash
export UV_EXTRA_INDEX_URL="https://aws:${AWS_CODEARTIFACT_TOKEN}@${AWS_DOMAIN}-${AWS_ACCOUNT_ID}.d.codeartifact.${AWS_REGION}.amazonaws.com/pypi/${AWS_CODEARTIFACT_REPOSITORY}/simple/"
```

### 发布包
如果你还想将你自己的包发布到 AWS CodeArtifact，可以按照[发布指南](../package.md)中的描述使用 `uv publish`。你需要单独设置 `UV_PUBLISH_URL` 和凭据：
```bash
# 配置 uv 使用 AWS CodeArtifact
export UV_PUBLISH_URL="https://${AWS_DOMAIN}-${AWS_ACCOUNT_ID}.d.codeartifact.${AWS_REGION}.amazonaws.com/pypi/${AWS_CODEARTIFACT_REPOSITORY}/"
export UV_PUBLISH_USERNAME=aws
export UV_PUBLISH_PASSWORD="$AWS_CODEARTIFACT_TOKEN"
# 发布包
uv publish
```

## 其他索引
uv 也可以与 JFrog 的 Artifactory 一起使用。