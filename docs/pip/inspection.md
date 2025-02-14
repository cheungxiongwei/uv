```md docs/pip/inspection.md (1-43)
# 检查环境

## 列出已安装的包

要列出环境中的所有包：

```console
$ uv pip list
```

要以 JSON 格式列出包：

```console
$ uv pip list --format json
```

要以 `requirements.txt` 格式列出环境中的所有包：

```console
$ uv pip freeze
```

## 检查包

要显示已安装包的信息，例如 `numpy`：

```console
$ uv pip show numpy
```

可以同时检查多个包。

## 验证环境

如果在多个步骤中安装包，可能会将具有冲突需求的包安装到环境中。

要检查环境中的冲突或缺失的依赖项：

```console
$ uv pip check
```

```