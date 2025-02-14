# 获取帮助

## 帮助菜单

可以使用 `--help` 标志查看命令的帮助菜单，例如，对于 `uv`：

```console
$ uv --help
```

要查看特定命令的帮助菜单，例如，对于 `uv init`：

```console
$ uv init --help
```

使用 `--help` 标志时，uv 会显示简短的帮助菜单。要查看命令的详细帮助菜单，请使用 `uv help`：

```console
$ uv help
```

要查看特定命令的详细帮助菜单，例如，对于 `uv init`：

```console
$ uv help init
```

使用详细帮助菜单时，uv 会尝试使用 `less` 或 `more` 来“分页”显示输出，以免一次性显示所有内容。要退出分页器，请按 `q`。

## 查看版本

在寻求帮助时，确定您使用的 uv 版本非常重要——有时问题在较新的版本中已经解决。

要检查安装的版本：

```console
$ uv version
```

以下命令同样有效：

```console
$ uv --version      # 与 `uv version` 输出相同
$ uv -V             # 不包含构建提交和日期
$ uv pip --version  # 可以与子命令一起使用
```

## 问题排查

参考文档中包含针对常见问题的[问题排查指南](../reference/troubleshooting/index.md)。

## 在 GitHub 上提交问题

GitHub 上的[问题跟踪器](https://github.com/astral-sh/uv/issues)是报告错误和请求功能的好地方。请确保先搜索类似问题，因为其他人可能也遇到了相同的问题。

## 在 Discord 上交流

Astral 有一个 [Discord 服务器](https://discord.com/invite/astral-sh)，这是提问、了解更多关于 uv 的信息以及与其他社区成员互动的好地方。

