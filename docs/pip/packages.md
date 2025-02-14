```md docs/pip/packages.md (1-123)
# 管理包

## 安装包

要在虚拟环境中安装一个包，例如 Flask：

```console
$ uv pip install flask
```

要安装一个包并启用可选依赖项，例如 Flask 的 "dotenv" 额外依赖：

```console
$ uv pip install "flask[dotenv]"
```

要安装多个包，例如 Flask 和 Ruff：

```console
$ uv pip install flask ruff
```

要安装一个带有约束的包，例如 Ruff v0.2.0 或更新版本：

```console
$ uv pip install 'ruff>=0.2.0'
```

要安装一个特定版本的包，例如 Ruff v0.3.0：

```console
$ uv pip install 'ruff==0.3.0'
```

要从磁盘安装一个包：

```console
$ uv pip install "ruff @ ./projects/ruff"
```

要从 GitHub 安装一个包：

```console
$ uv pip install "git+https://github.com/astral-sh/ruff"
```

要从 GitHub 安装一个特定引用的包：

```console
$ # 安装一个标签
$ uv pip install "git+https://github.com/astral-sh/ruff@v0.2.0"

$ # 安装一个提交
$ uv pip install "git+https://github.com/astral-sh/ruff@1fadefa67b26508cc59cf38e6130bde2243c929d"

$ # 安装一个分支
$ uv pip install "git+https://github.com/astral-sh/ruff@main"
```

有关从私有仓库安装的信息，请参阅 [Git 认证](../configuration/authentication.md#git-authentication) 文档。

## 可编辑包

可编辑包在源代码更改后无需重新安装即可生效。

要将当前项目安装为可编辑包：

```console
$ uv pip install -e .
```

要将另一个目录中的项目安装为可编辑包：

```console
$ uv pip install -e "ruff @ ./project/ruff"
```

## 从文件安装包

可以从标准文件格式一次性安装多个包。

从 `requirements.txt` 文件安装：

```console
$ uv pip install -r requirements.txt
```

有关 `requirements.txt` 文件的更多信息，请参阅 [`uv pip compile`](./compile.md) 文档。

从 `pyproject.toml` 文件安装：

```console
$ uv pip install -r pyproject.toml
```

从 `pyproject.toml` 文件安装并启用可选依赖项，例如 "foo" 额外依赖：

```console
$ uv pip install -r pyproject.toml --extra foo
```

从 `pyproject.toml` 文件安装并启用所有可选依赖项：

```console
$ uv pip install -r pyproject.toml --all-extras
```

## 卸载包

要卸载一个包，例如 Flask：

```console
$ uv pip uninstall flask
```

要卸载多个包，例如 Flask 和 Ruff：

```console
$ uv pip uninstall flask ruff
```

```