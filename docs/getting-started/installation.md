# 安装 uv

## 安装方法

使用我们的独立安装程序或您选择的包管理器来安装 uv。

### 独立安装程序

uv 提供了一个独立安装程序来下载和安装 uv：

=== "macOS 和 Linux"

    使用 `curl` 下载脚本并使用 `sh` 执行：

    ```console
    $ curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

    如果您的系统没有 `curl`，可以使用 `wget`：

    ```console
    $ wget -qO- https://astral.sh/uv/install.sh | sh
    ```

    通过在 URL 中包含特定版本来请求该版本：

    ```console
    $ curl -LsSf https://astral.sh/uv/0.5.30/install.sh | sh
    ```

=== "Windows"

    使用 `irm` 下载脚本并使用 `iex` 执行：

    ```console
    $ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    ```

    更改 [执行策略](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4#powershell-execution-policies) 允许从互联网运行脚本。

    通过在 URL 中包含特定版本来请求该版本：

    ```console
    $ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/0.5.30/install.ps1 | iex"
    ```

!!! tip

    安装脚本可以在使用前进行检查：

    === "macOS 和 Linux"

        ```console
        $ curl -LsSf https://astral.sh/uv/install.sh | less
        ```

    === "Windows"

        ```console
        $ powershell -c "irm https://astral.sh/uv/install.ps1 | more"
        ```

    或者，可以直接从 [GitHub](#github-releases) 下载安装程序或二进制文件。

有关自定义 uv 安装的详细信息，请参阅 [安装程序配置](../configuration/installer.md) 文档。

### PyPI

为了方便起见，uv 已发布到 [PyPI](https://pypi.org/project/uv/)。

如果从 PyPI 安装，我们建议将 uv 安装到隔离的环境中，例如使用 `pipx`：

```console
$ pipx install uv
```

不过，也可以使用 `pip`：

```console
$ pip install uv
```

!!! note

    uv 为许多平台提供了预构建的发行版（wheels）；如果某个平台没有可用的 wheel，uv 将从源代码构建，这需要 Rust 工具链。有关从源代码构建 uv 的详细信息，请参阅 [贡献设置指南](https://github.com/astral-sh/uv/blob/main/CONTRIBUTING.md#setup)。

### Cargo

uv 可以通过 Cargo 安装，但由于其依赖未发布的 crate，必须从 Git 构建而不是从 [crates.io](https://crates.io) 安装。

```console
$ cargo install --git https://github.com/astral-sh/uv uv
```

### Homebrew

uv 已包含在 Homebrew 的核心包中。

```console
$ brew install uv
```

### WinGet

uv 可通过 [WinGet](https://winstall.app/apps/astral-sh.uv) 安装。

```console
$ winget install --id=astral-sh.uv  -e
```

### Scoop

uv 可通过 [Scoop](https://scoop.sh/#/apps?q=uv) 安装。

```console
$ scoop install main/uv
```

### Docker

uv 提供了一个 Docker 镜像，位于 [`ghcr.io/astral-sh/uv`](https://github.com/astral-sh/uv/pkgs/container/uv)。

有关在 Docker 中使用 uv 的更多详细信息，请参阅我们的 [Docker 集成指南](../guides/integration/docker.md)。

### GitHub Releases

uv 的发布工件可以直接从 [GitHub Releases](https://github.com/astral-sh/uv/releases) 下载。

每个发布页面都包含所有支持平台的二进制文件，以及通过 `github.com` 而不是 `astral.sh` 使用独立安装程序的说明。

## 升级 uv

当 uv 通过独立安装程序安装时，它可以按需更新自身：

```console
$ uv self update
```

!!! tip

    更新 uv 将重新运行安装程序，并可能修改您的 shell 配置文件。要禁用此行为，请设置 `INSTALLER_NO_MODIFY_PATH=1`。

当使用其他安装方法时，自更新功能被禁用。请使用包管理器的升级方法。例如，使用 `pip`：

```console
$ pip install --upgrade uv
```

## Shell 自动补全

!!! tip

    您可以运行 `echo $SHELL` 来帮助确定您的 shell。

要为 uv 命令启用 shell 自动补全，请运行以下命令之一：

=== "Bash"

    ```bash
    echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
    ```

=== "Zsh"

    ```bash
    echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc
    ```

=== "fish"

    ```bash
    echo 'uv generate-shell-completion fish | source' >> ~/.config/fish/config.fish
    ```

=== "Elvish"

    ```bash
    echo 'eval (uv generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
    ```

=== "PowerShell / pwsh"

    ```powershell
    if (!(Test-Path -Path $PROFILE)) {
      New-Item -ItemType File -Path $PROFILE -Force
    }
    Add-Content -Path $PROFILE -Value '(& uv generate-shell-completion powershell) | Out-String | Invoke-Expression'
    ```

要为 uvx 启用 shell 自动补全，请运行以下命令之一：

=== "Bash"

    ```bash
    echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
    ```

=== "Zsh"

    ```bash
    echo 'eval "$(uvx --generate-shell-completion zsh)"' >> ~/.zshrc
    ```

=== "fish"

    ```bash
    echo 'uvx --generate-shell-completion fish | source' >> ~/.config/fish/config.fish
    ```

=== "Elvish"

    ```bash
    echo 'eval (uvx --generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
    ```

=== "PowerShell / pwsh"

    ```powershell
    if (!(Test-Path -Path $PROFILE)) {
      New-Item -ItemType File -Path $PROFILE -Force
    }
    Add-Content -Path $PROFILE -Value '(& uvx --generate-shell-completion powershell) | Out-String | Invoke-Expression'
    ```

然后重新启动 shell 或 source shell 配置文件。

## 卸载

如果您需要从系统中移除 uv，请按照以下步骤操作：

1.  清理存储的数据（可选）：

    ```console
    $ uv cache clean
    $ rm -r "$(uv python dir)"
    $ rm -r "$(uv tool dir)"
    ```

    !!! tip

        在移除二进制文件之前，您可能希望删除 uv 存储的任何数据。

2.  移除 uv 和 uvx 二进制文件：

    === "macOS 和 Linux"

        ```console
        $ rm ~/.local/bin/uv ~/.local/bin/uvx
        ```

    === "Windows"

        ```powershell
        $ rm $HOME\.local\bin\uv.exe
        $ rm $HOME\.local\bin\uvx.exe
        ```

    !!! note

        在 0.5.0 之前，uv 被安装到 `~/.cargo/bin`。可以从那里移除二进制文件以进行卸载。从旧版本升级不会自动移除 `~/.cargo/bin` 中的二进制文件。

## 下一步

请参阅 [第一步](./first-steps.md) 或直接跳转到 [指南](../guides/index.md) 开始使用 uv。