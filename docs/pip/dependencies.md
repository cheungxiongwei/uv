# 声明依赖项

最佳实践是在静态文件中声明依赖项，而不是通过临时安装来修改环境。一旦定义了依赖项，就可以将它们[锁定](./compile.md)以创建一致且可重现的环境。

## 使用 `pyproject.toml`

`pyproject.toml` 文件是定义项目配置的 Python 标准。

要在 `pyproject.toml` 文件中定义项目依赖项：

```toml title="pyproject.toml"
[project]
dependencies = [
  "httpx",
  "ruff>=0.3.0"
]
```

要在 `pyproject.toml` 文件中定义可选依赖项：

```toml title="pyproject.toml"
[project.optional-dependencies]
cli = [
  "rich",
  "click",
]
```

每个键都定义了一个 "extra"，可以使用 `--extra` 和 `--all-extras` 标志或 `package[<extra>]` 语法进行安装。有关更多详细信息，请参阅[安装包](./packages.md#installing-packages-from-files)的文档。

有关开始使用 `pyproject.toml` 的更多详细信息，请参阅官方的[`pyproject.toml` 指南](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)。

## 使用 `requirements.in`

通常也使用轻量级的 `requirements.txt` 格式来声明项目的依赖项。每个需求都在单独的一行中定义。通常，此文件称为 `requirements.in`，以区别于用于锁定依赖项的 `requirements.txt`。

要在 `requirements.in` 文件中定义依赖项：

```python title="requirements.in"
httpx
ruff>=0.3.0
```

此格式不支持可选依赖项组。