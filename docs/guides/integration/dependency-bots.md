---
title: Using uv with dependency bots
description: A guide to using uv with dependency bots like Renovate and Dependabot.
---

# 依赖管理机器人
定期更新依赖项被认为是最佳实践，以避免暴露于漏洞、限制依赖项之间的不兼容性，并在从过旧版本升级时避免复杂的升级过程。有多种工具可以通过创建自动化的拉取请求来帮助保持依赖项的最新状态。其中一些工具已经支持 `uv`，或者正在努力支持它。

## Renovate
`uv` 由 [Renovate](https://github.com/renovatebot/renovate) 支持。

!!! note
更新 `uv pip compile` 输出（如 `requirements.txt`）目前还不支持。进展可以在 [renovatebot/renovate#30909](https://github.com/renovatebot/renovate/issues/30909) 跟踪。

### `uv.lock` 输出
Renovate 通过检测 `uv.lock` 文件的存在来确定是否使用 `uv` 来管理依赖项，并将建议升级 [项目依赖项](../../concepts/projects/dependencies.md#project-dependencies)、[可选依赖项](../../concepts/projects/dependencies.md#optional-dependencies) 和 [开发依赖项](../../concepts/projects/dependencies.md#development-dependencies)。Renovate 将同时更新 `pyproject.toml` 和 `uv.lock` 文件。

还可以通过启用 [`lockFileMaintenance`](https://docs.renovatebot.com/configuration-options/#lockfilemaintenance) 选项定期刷新锁文件（例如以更新传递依赖项）：

```jsx title="renovate.json5"
{
$schema: "https://docs.renovatebot.com/renovate-schema.json",
lockFileMaintenance: {
enabled: true,
},
}
```

### 内联脚本元数据
Renovate 支持更新使用 [脚本内联元数据](../scripts.md/#declaring-script-dependencies) 定义的依赖项。由于它无法自动检测哪些 Python 文件使用了脚本内联元数据，因此需要使用 [`fileMatch`](https://docs.renovatebot.com/configuration-options/#filematch) 明确指定它们的位置，如下所示：

```jsx title="renovate.json5"
{
$schema: "https://docs.renovatebot.com/renovate-schema.json",
pep723: {
fileMatch: [
"scripts/generate_docs\\.py",
"scripts/run_server\\.py",
],
},
}
```

## Dependabot
目前尚不支持 `uv`。进展可以在 [dependabot/dependabot-core#10478](https://github.com/dependabot/dependabot-core/issues/10478) 跟踪。