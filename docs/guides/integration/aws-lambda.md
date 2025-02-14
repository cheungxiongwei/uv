---
title: Using uv with AWS Lambda
description:
  A complete guide to using uv with AWS Lambda to manage Python dependencies and deploy serverless
  functions via Docker containers or zip archives.
---

# 使用 uv 与 AWS Lambda

[AWS Lambda](https://aws.amazon.com/lambda/) 是一种无服务器计算服务，允许您运行代码而无需预置或管理服务器。

您可以使用 uv 与 AWS Lambda 来管理 Python 依赖项、构建部署包并部署 Lambda 函数。

!!! tip

    查看 [`uv-aws-lambda-example`](https://github.com/astral-sh/uv-aws-lambda-example) 项目，了解使用 uv 将应用程序部署到 AWS Lambda 的最佳实践示例。

## 入门指南

首先，假设我们有一个最小的 FastAPI 应用程序，结构如下：

```plaintext
project
├── pyproject.toml
└── app
    ├── __init__.py
    └── main.py
```

其中 `pyproject.toml` 包含：

```toml title="pyproject.toml"
[project]
name = "uv-aws-lambda-example"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    # FastAPI 是一个用于构建 API 的现代 Python Web 框架。
    "fastapi",
    # Mangum 是一个将 ASGI 应用程序适配到 AWS Lambda 和 API Gateway 的库。
    "mangum",
]

[dependency-groups]
dev = [
    # 在开发模式下，包含 FastAPI 开发服务器。
    "fastapi[standard]>=0.115",
]
```

而 `main.py` 文件包含：

```python title="app/main.py"
import logging

from fastapi import FastAPI
from mangum import Mangum

logger = logging.getLogger()
logger.setLevel(logging.INFO)

app = FastAPI()
handler = Mangum(app)


@app.get("/")
async def root() -> str:
    return "Hello, world!"
```

我们可以通过以下命令在本地运行此应用程序：

```console
$ uv run fastapi dev
```

然后，在浏览器中打开 http://127.0.0.1:8000/ 将显示 "Hello, world!"

## 部署 Docker 镜像

要部署到 AWS Lambda，我们需要构建一个包含应用程序代码和依赖项的容器镜像。

我们将遵循 [Docker 指南](./docker.md) 中概述的原则（特别是多阶段构建），以确保最终镜像尽可能小且缓存友好。

在第一阶段，我们将所有应用程序代码和依赖项打包到一个目录中。在第二阶段，我们将此目录复制到最终镜像中，省略构建工具和其他不必要的文件。

```dockerfile title="Dockerfile"
FROM ghcr.io/astral-sh/uv:0.5.30 AS uv

# 首先，将依赖项打包到任务根目录。
FROM public.ecr.aws/lambda/python:3.13 AS builder

# 启用字节码编译，以提高冷启动性能。
ENV UV_COMPILE_BYTECODE=1

# 禁用安装程序元数据，以创建确定性层。
ENV UV_NO_INSTALLER_METADATA=1

# 启用复制模式以支持绑定挂载缓存。
ENV UV_LINK_MODE=copy

# 通过 `uv pip install --target` 将依赖项打包到 Lambda 任务根目录。
#
# 省略任何本地包 (`--no-emit-workspace`) 和开发依赖项 (`--no-dev`)。
# 这确保 Docker 层缓存仅在 `pyproject.toml` 或 `uv.lock` 文件更改时失效，但在应用程序代码更改时保持稳健。
RUN --mount=from=uv,source=/uv,target=/bin/uv \
    --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv export --frozen --no-emit-workspace --no-dev --no-editable -o requirements.txt && \
    uv pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

FROM public.ecr.aws/lambda/python:3.13

# 从构建阶段复制运行时依赖项。
COPY --from=builder ${LAMBDA_TASK_ROOT} ${LAMBDA_TASK_ROOT}

# 复制应用程序代码。
COPY ./app ${LAMBDA_TASK_ROOT}/app

# 设置 AWS Lambda 处理程序。
CMD ["app.main.handler"]
```

!!! tip

    要部署到基于 ARM 的 AWS Lambda 运行时，请将 `public.ecr.aws/lambda/python:3.13` 替换为 `public.ecr.aws/lambda/python:3.13-arm64`。

我们可以通过以下命令构建镜像：

```console
$ uv lock
$ docker build -t fastapi-app .
```

此 Dockerfile 结构的核心优势如下：

1. **镜像体积最小化。** 通过使用多阶段构建，我们可以确保最终镜像仅包含应用程序代码和依赖项。例如，uv 二进制文件本身不包含在最终镜像中。
2. **最大化缓存重用。** 通过将应用程序依赖项与应用程序代码分开安装，我们可以确保 Docker 层缓存仅在依赖项更改时失效。

具体来说，在修改应用程序源代码后重建镜像可以重用缓存层，从而实现毫秒级构建：

```console
 => [internal] load build definition from Dockerfile                                                                 0.0s
 => => transferring dockerfile: 1.31kB                                                                               0.0s
 => [internal] load metadata for public.ecr.aws/lambda/python:3.13                                                   0.3s
 => [internal] load metadata for ghcr.io/astral-sh/uv:latest                                                         0.3s
 => [internal] load .dockerignore                                                                                    0.0s
 => => transferring context: 106B                                                                                    0.0s
 => [uv 1/1] FROM ghcr.io/astral-sh/uv:latest@sha256:ea61e006cfec0e8d81fae901ad703e09d2c6cf1aa58abcb6507d124b50286f  0.0s
 => [builder 1/2] FROM public.ecr.aws/lambda/python:3.13@sha256:f5b51b377b80bd303fe8055084e2763336ea8920d12955b23ef  0.0s
 => [internal] load build context                                                                                    0.0s
 => => transferring context: 185B                                                                                    0.0s
 => CACHED [builder 2/2] RUN --mount=from=uv,source=/uv,target=/bin/uv     --mount=type=cache,target=/root/.cache/u  0.0s
 => CACHED [stage-2 2/3] COPY --from=builder /var/task /var/task                                                     0.0s
 => CACHED [stage-2 3/3] COPY ./app /var/task                                                                        0.0s
 => exporting to image                                                                                               0.0s
 => => exporting layers                                                                                              0.0s
 => => writing image sha256:6f8f9ef715a7cda466b677a9df4046ebbb90c8e88595242ade3b4771f547652d                         0.0
```

构建完成后，我们可以将镜像推送到 [Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)，例如：

```console
$ aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
$ docker tag fastapi-app:latest aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest
$ docker push aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest
```

最后，我们可以使用 AWS 管理控制台或 AWS CLI 将镜像部署到 AWS Lambda，例如：

```console
$ aws lambda create-function \
   --function-name myFunction \
   --package-type Image \
   --code ImageUri=aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest \
   --role arn:aws:iam::111122223333:role/my-lambda-role
```

其中 [执行角色](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-api) 通过以下命令创建：

```console
$ aws iam create-role \
   --role-name my-lambda-role \
   --assume-role-policy-document '{"Version": "2012-10-17", "Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```

或者，更新现有函数：

```console
$ aws lambda update-function-code \
   --function-name myFunction \
   --image-uri aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest \
   --publish
```

要测试 Lambda，我们可以通过 AWS 管理控制台或 AWS CLI 调用它，例如：

```console
$ aws lambda invoke \
   --function-name myFunction \
   --payload file://event.json \
   --cli-binary-format raw-in-base64-out \
   response.json
{
  "StatusCode": 200,
  "ExecutedVersion": "$LATEST"
}
```

其中 `event.json` 包含传递给 Lambda 函数的事件负载：

```json title="event.json"
{
  "httpMethod": "GET",
  "path": "/",
  "requestContext": {},
  "version": "1.0"
}
```

而 `response.json` 包含 Lambda 函数的响应：

```json title="response.json"
{
  "statusCode": 200,
  "headers": {
    "content-length": "14",
    "content-type": "application/json"
  },
  "multiValueHeaders": {},
  "body": "\"Hello, world!\"",
  "isBase64Encoded": false
}
```

有关详细信息，请参阅 [AWS Lambda 文档](https://docs.aws.amazon.com/lambda/latest/dg/python-image.html)。

### 工作区支持

如果项目包含本地依赖项（例如通过[工作区](../../concepts/projects/workspaces.md)），这些依赖项也必须包含在部署包中。

我们将通过扩展上述示例来包含对本地开发的库`library`的依赖。

首先，我们将创建库本身：

```console
$ uv init --lib library
$ uv add ./library
```

在`project`目录中运行`uv init`将自动将`project`转换为工作区，并将`library`添加为工作区成员：

```toml title="pyproject.toml"
[project]
name = "uv-aws-lambda-example"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    # FastAPI 是一个用于使用 Python 构建 API 的现代 Web 框架。
    "fastapi",
    # 一个本地库。
    "library",
    # Mangum 是一个将 ASGI 应用程序适配到 AWS Lambda 和 API Gateway 的库。
    "mangum",
]

[dependency-groups]
dev = [
    # 在开发模式下，包含 FastAPI 开发服务器。
    "fastapi[standard]",
]

[tool.uv.workspace]
members = ["library"]

[tool.uv.sources]
lib = { workspace = true }
```

默认情况下，`uv init --lib`将创建一个导出`hello`函数的包。我们将修改应用程序源代码以调用该函数：

```python title="app/main.py"
import logging

from fastapi import FastAPI
from mangum import Mangum

from library import hello

logger = logging.getLogger()
logger.setLevel(logging.INFO)

app = FastAPI()
handler = Mangum(app)


@app.get("/")
async def root() -> str:
    return hello()
```

我们可以通过以下命令在本地运行修改后的应用程序：

```console
$ uv run fastapi dev
```

并确认在浏览器中打开 http://127.0.0.1:8000/ 会显示 "Hello from library!"（而不是 "Hello, World!"）

最后，我们将更新 Dockerfile 以将本地库包含在部署包中：

```dockerfile title="Dockerfile"
FROM ghcr.io/astral-sh/uv:0.5.30 AS uv

# 首先，将依赖项打包到任务根目录中。
FROM public.ecr.aws/lambda/python:3.13 AS builder

# 启用字节码编译，以提高冷启动性能。
ENV UV_COMPILE_BYTECODE=1

# 禁用安装程序元数据，以创建确定性层。
ENV UV_NO_INSTALLER_METADATA=1

# 启用复制模式以支持绑定挂载缓存。
ENV UV_LINK_MODE=copy

# 通过 `uv pip install --target` 将依赖项打包到 Lambda 任务根目录中。
#
# 省略任何本地包（`--no-emit-workspace`）和开发依赖项（`--no-dev`）。
# 这确保 Docker 层缓存仅在 `pyproject.toml` 或 `uv.lock` 文件更改时失效，
# 但对应用程序代码的更改保持稳健。
RUN --mount=from=uv,source=/uv,target=/bin/uv \
    --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv export --frozen --no-emit-workspace --no-dev --no-editable -o requirements.txt && \
    uv pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# 如果你有工作区，请将其复制并安装。
#
# 通过省略 `--no-emit-workspace`，`library` 将被复制到任务根目录中。使用单独的
# `RUN` 命令确保所有第三方依赖项单独缓存，并对工作区的更改保持稳健。
RUN --mount=from=uv,source=/uv,target=/bin/uv \
    --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    --mount=type=bind,source=library,target=library \
    uv export --frozen --no-dev --no-editable -o requirements.txt && \
    uv pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

FROM public.ecr.aws/lambda/python:3.13

# 从构建阶段复制运行时依赖项。
COPY --from=builder ${LAMBDA_TASK_ROOT} ${LAMBDA_TASK_ROOT}

# 复制应用程序代码。
COPY ./app ${LAMBDA_TASK_ROOT}/app

# 设置 AWS Lambda 处理程序。
CMD ["app.main.handler"]
```

!!! tip

    要部署到基于 ARM 的 AWS Lambda 运行时，请将 `public.ecr.aws/lambda/python:3.13` 替换为 `public.ecr.aws/lambda/python:3.13-arm64`。

从那里，我们可以像以前一样构建和部署更新后的镜像。

## 部署 zip 存档

AWS Lambda 也支持通过 zip 存档进行部署。对于简单的应用程序，zip 存档可能比 Docker 镜像更直接和高效；然而，zip 存档的大小限制为 [250 MB](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html#python-package-create-update)。

回到 FastAPI 示例，我们可以通过以下方式将应用程序依赖项打包到本地目录中以便 AWS Lambda 使用：

```console
$ uv export --frozen --no-dev --no-editable -o requirements.txt
$ uv pip install \
   --no-installer-metadata \
   --no-compile-bytecode \
   --python-platform x86_64-manylinux2014 \
   --python 3.13 \
   --target packages \
   -r requirements.txt
```

!!! tip

    要部署到基于 ARM 的 AWS Lambda 运行时，请将 `x86_64-manylinux2014` 替换为 `aarch64-manylinux2014`。

按照 [AWS Lambda 文档](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html)，我们可以将这些依赖项打包成 zip 文件，如下所示：

```console
$ cd packages
$ zip -r ../package.zip .
$ cd ..
```

最后，我们可以将应用程序代码添加到 zip 存档中：

```console
$ zip -r package.zip app
```

然后，我们可以通过 AWS 管理控制台或 AWS CLI 将 zip 存档部署到 AWS Lambda，例如：

```console
$ aws lambda create-function \
   --function-name myFunction \
   --runtime python3.13 \
   --zip-file fileb://package.zip \
   --handler app.main.handler \
   --role arn:aws:iam::111122223333:role/service-role/my-lambda-role
```

其中[执行角色](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-api)通过以下命令创建：

```console
$ aws iam create-role \
   --role-name my-lambda-role \
   --assume-role-policy-document '{"Version": "2012-10-17", "Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```

或者，更新现有函数：

```console
$ aws lambda update-function-code \
   --function-name myFunction \
   --zip-file fileb://package.zip
```

!!! note

    默认情况下，AWS 管理控制台假定 Lambda 入口点为 `lambda_function.lambda_handler`。
    如果你的应用程序使用不同的入口点，你需要在 AWS 管理控制台中修改它。
    例如，上述 FastAPI 应用程序使用 `app.main.handler`。

要测试 Lambda，我们可以通过 AWS 管理控制台或 AWS CLI 调用它，例如：

```console
$ aws lambda invoke \
   --function-name myFunction \
   --payload file://event.json \
   --cli-binary-format raw-in-base64-out \
   response.json
{
  "StatusCode": 200,
  "ExecutedVersion": "$LATEST"
}
```

其中 `event.json` 包含传递给 Lambda 函数的事件负载：

```json title="event.json"
{
  "httpMethod": "GET",
  "path": "/",
  "requestContext": {},
  "version": "1.0"
}
```

而 `response.json` 包含 Lambda 函数的响应：

```json title="response.json"
{
  "statusCode": 200,
  "headers": {
    "content-length": "14",
    "content-type": "application/json"
  },
  "multiValueHeaders": {},
  "body": "\"Hello, world!\"",
  "isBase64Encoded": false
}
```

### 使用 Lambda 层

AWS Lambda 还支持在使用 zip 存档时部署多个组合的 [Lambda 层](https://docs.aws.amazon.com/lambda/latest/dg/python-layers.html)。这些层在概念上类似于 Docker 镜像中的层，允许你将应用程序代码与依赖项分离。

特别是，我们可以为应用程序依赖项创建一个 Lambda 层，并将其附加到 Lambda 函数，与应用程序代码本身分开。这种设置可以提高应用程序更新的冷启动性能，因为依赖项层可以在多个部署中重复使用。

要创建 Lambda 层，我们将遵循类似的步骤，但创建两个单独的 zip 存档：一个用于应用程序代码，另一个用于应用程序依赖项。

首先，我们将创建依赖项层。Lambda 层需要遵循稍微不同的结构，因此我们将使用 `--prefix` 而不是 `--target`：

```console
$ uv export --frozen --no-dev --no-editable -o requirements.txt
$ uv pip install \
   --no-installer-metadata \
   --no-compile-bytecode \
   --python-platform x86_64-manylinux2014 \
   --python 3.13 \
   --prefix packages \
   -r requirements.txt
```

然后，我们将按照 Lambda 层的预期布局压缩依赖项：

```console
$ mkdir python
$ cp -r packages/lib python/
$ zip -r layer_content.zip python
```

!!! tip

    要生成确定性的 zip 存档，请考虑将 `-X` 标志传递给 `zip` 以排除扩展属性和文件系统元数据。

并发布 Lambda 层：

```console
$ aws lambda publish-layer-version --layer-name dependencies-layer \
   --zip-file fileb://layer_content.zip \
   --compatible-runtimes python3.13 \
   --compatible-architectures "x86_64"
```

然后，我们可以像前面的示例一样创建 Lambda 函数，省略依赖项：

```console
$ # 压缩应用程序代码。
$ zip -r app.zip app

$ # 创建 Lambda 函数。
$ aws lambda create-function \
   --function-name myFunction \
   --runtime python3.13 \
   --zip-file fileb://app.zip \
   --handler app.main.handler \
   --role arn:aws:iam::111122223333:role/service-role/my-lambda-role
```

最后，我们可以将依赖项层附加到 Lambda 函数，使用 `publish-layer-version` 步骤返回的 ARN：

```console
$ aws lambda update-function-configuration --function-name myFunction \
    --cli-binary-format raw-in-base64-out \
    --layers "arn:aws:lambda:region:111122223333:layer:dependencies-layer:1"
```

当应用程序依赖项更改时，可以通过重新发布层并更新 Lambda 函数配置来独立更新层：

```console
$ # 更新层中的依赖项。
$ aws lambda publish-layer-version --layer-name dependencies-layer \
   --zip-file fileb://layer_content.zip \
   --compatible-runtimes python3.13 \
   --compatible-architectures "x86_64"

$ # 更新 Lambda 函数配置。
$ aws lambda update-function-configuration --function-name myFunction \
    --cli-binary-format raw-in-base64-out \
    --layers "arn:aws:lambda:region:111122223333:layer:dependencies-layer:2"
```