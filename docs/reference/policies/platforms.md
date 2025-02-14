# 平台支持

uv 对以下平台提供 Tier 1 支持：

- macOS (Apple Silicon)
- macOS (x86_64)
- Linux (x86_64)
- Windows (x86_64)

uv 在其 Tier 1 平台上持续构建、测试和开发。受 Rust 项目的启发，Tier 1 可以被认为是
["保证可用"](https://doc.rust-lang.org/beta/rustc/platform-support.html)。

uv 对以下平台提供 Tier 2 支持
（["保证可构建"](https://doc.rust-lang.org/beta/rustc/platform-support.html)）：

- Linux (PPC64)
- Linux (PPC64LE)
- Linux (aarch64)
- Linux (armv7)
- Linux (i686)
- Linux (s390x)

uv 为其 Tier 1 和 Tier 2 平台向 [PyPI](https://pypi.org/project/uv/) 提供预构建的 wheel 包。然而，虽然 Tier 2 平台持续构建，但它们并未持续测试或开发，因此在实践中稳定性可能有所不同。

除了 Tier 1 和 Tier 2 平台之外，uv 已知可以在 i686 Windows 上构建，但已知无法在 aarch64 Windows 上构建，但 uv 目前并未将这两个平台视为受支持的平台。最低支持的 Windows 版本是 Windows 10 和 Windows Server 2016，遵循
[Rust 自身的 Tier 1 支持](https://blog.rust-lang.org/2024/02/26/Windows-7.html)。

uv 支持并测试了 Python 3.8、3.9、3.10、3.11、3.12 和 3.13。