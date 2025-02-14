```md docs/pip/index.md (1-26)
# pip 接口

uv 提供了对常见 `pip`、`pip-tools` 和 `virtualenv` 命令的替代方案。这些命令直接与虚拟环境交互，与 uv 的主要接口不同，后者会自动管理虚拟环境。`uv pip` 接口将 uv 的速度和功能暴露给高级用户以及尚未准备好从 `pip` 和 `pip-tools` 过渡的项目。

以下部分讨论了使用 `uv pip` 的基础知识：

- [创建和使用环境](./environments.md)
- [安装和管理包](./packages.md)
- [检查环境和包](./inspection.md)
- [声明包依赖](./dependencies.md)
- [锁定和同步环境](./compile.md)

请注意，这些命令并不完全实现它们所基于工具的接口和行为。越偏离常见工作流程，越有可能遇到差异。详情请参阅 [pip 兼容性指南](./compatibility.md)。

!!! important

    uv 不依赖也不调用 pip。pip 接口如此命名是为了突出其提供与 pip 接口匹配的低级命令的专用目的，并将其与 uv 其他在更高抽象级别上操作的命令区分开来。
```