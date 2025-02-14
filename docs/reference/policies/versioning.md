# 版本控制

uv 采用自定义的版本控制方案，其中重大变更会递增次版本号，而错误修复、功能增强和其他非破坏性变更则会递增修订版本号。

uv 目前尚未拥有稳定的 API；一旦 uv 的 API 稳定（v1.0.0），版本控制方案将遵循 [Semantic Versioning](https://semver.org/)。

uv 的变更日志可以在 [GitHub](https://github.com/astral-sh/uv/blob/main/CHANGELOG.md) 上查看。

## 缓存版本控制

缓存版本被视为 uv 的内部实现，因此可能会在次版本或修订版本中更改。详情请参阅 [缓存版本控制](../../concepts/cache.md#cache-versioning)。

## Lockfile 版本控制

`uv.lock` 的 schema 版本被视为公共 API 的一部分，因此仅会在次版本中作为重大变更进行递增。详情请参阅 [Lockfile 版本控制](../../concepts/resolution.md#lockfile-versioning)。