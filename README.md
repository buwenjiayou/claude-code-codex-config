# Claude Code & Codex 配置经验笔记

这是一个面向个人开发环境的配置经验仓库，主要整理 Claude Code 和 Codex 在终端、IDE、本地电脑、远程服务器中的常见配置入口与排错思路。

推荐仓库名：`claude-code-codex-config-notes`

> 本文档只记录个人实践经验。不同版本的 Claude Code、Codex、IDE 插件或中转服务可能存在差异，请以你实际安装版本和官方说明为准。

## 目录

- [适用场景](#适用场景)
- [核心原则](#核心原则)
- [Claude Code 配置](#claude-code-配置)
- [Codex 配置](#codex-配置)
- [本地与服务器注意事项](#本地与服务器注意事项)
- [排错清单](#排错清单)

## 适用场景

这份笔记适合以下情况：

- 在 Windows 本地终端使用 Claude Code 或 Codex。
- 在 VS Code、Cursor 等 IDE 中使用 Claude Code 或 Codex 插件。
- 在远程服务器上配置 AI 编程工具。
- 使用 `ccswitch` 或类似工具切换模型服务商、API 地址和密钥。
- 希望把配置经验整理成可复用、可分享、不会泄露密钥的模板。

## 核心原则

配置这类工具时，最容易出错的不是字段本身，而是“当前入口到底读取哪个配置文件”。

建议先记住这几条：

1. 终端入口和 IDE 插件入口通常读取不同配置。
2. 用户级配置和项目级配置要分开看。
3. Windows、本地 Linux/macOS、远程服务器的配置路径不同。
4. `auth.json`、API key、token、真实中转地址不要提交到 GitHub。
5. 如果你使用 `ccswitch`，要确认它实际修改的是哪个文件。

## Claude Code 配置

### 终端使用

在终端直接使用 Claude Code 时，Windows 下通常关注这个文件：

```text
C:\Users\用户名\.claude\settings.json
```

在 Linux、macOS 或远程服务器上，通常对应：

```text
~/.claude/settings.json
```

如果你使用 `ccswitch`，需要确认它修改的也是这个文件。终端启动时一般以这个配置文件，或当前 shell 中的环境变量为准。

模板文件见：

```text
examples/claude-terminal-settings.json
```

示例：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "<your-base-url>",
    "ANTHROPIC_AUTH_TOKEN": "<your-anthropic-auth-token>",
    "ANTHROPIC_MODEL": "<your-default-model>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "<your-sonnet-model>",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "<your-opus-model>",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "<your-haiku-model>"
  }
}
```

也可以直接在系统环境变量或 shell 环境变量中配置：

- `ANTHROPIC_BASE_URL`
- `ANTHROPIC_AUTH_TOKEN`
- `ANTHROPIC_MODEL`
- `ANTHROPIC_DEFAULT_SONNET_MODEL`
- `ANTHROPIC_DEFAULT_OPUS_MODEL`
- `ANTHROPIC_DEFAULT_HAIKU_MODEL`

如果同时存在文件配置和环境变量，建议不要混用。排错时先固定一种方式，确认生效后再扩展。

### IDE 插件使用

在 IDE 中使用 Claude Code 插件时，通常以 IDE 自己的 `settings.json` 为准。也就是说，终端里的 `~/.claude/settings.json` 生效，不代表 IDE 插件一定会读取同一份配置。

模板文件见：

```text
examples/claude-ide-settings.jsonc
```

精简配置一般只需要 API 地址和 token：

```jsonc
{
  // 精简配置：先保证 IDE 插件能连上服务。
  "claudeCode.environmentVariables": [
    {
      "name": "ANTHROPIC_BASE_URL",
      "value": "<your-base-url>"
    },
    {
      "name": "ANTHROPIC_AUTH_TOKEN",
      "value": "<your-anthropic-auth-token>"
    }
  ]
}
```

如果插件支持更多选项，可以再加界面位置和模型选择：

```jsonc
{
  // 可选：把 Claude Code 固定在 IDE 面板中。
  "claudeCode.preferredLocation": "panel",

  // 精简配置：先保证 IDE 插件能连上服务。
  "claudeCode.environmentVariables": [
    {
      "name": "ANTHROPIC_BASE_URL",
      "value": "<your-base-url>"
    },
    {
      "name": "ANTHROPIC_AUTH_TOKEN",
      "value": "<your-anthropic-auth-token>"
    }
  ],

  // 可选：填写插件和服务商都支持的模型名。
  "claudeCode.selectedModel": "<your-selected-model>"
}
```

## Codex 配置

### CLI 使用

使用 Codex CLI 时，重点关注指定 `.codex` 目录中的两个文件：

```text
.codex/config.toml
.codex/auth.json
```

如果是用户级配置，常见位置是：

```text
~/.codex/config.toml
~/.codex/auth.json
```

如果使用 `ccswitch`，也要确认它修改的是当前 Codex 实际读取的 `.codex` 目录。

`config.toml` 模板见：

```text
examples/codex-config.toml
```

示例：

```toml
# 这里的名称需要和下方 model_providers 中的名称一致。
model_provider = "<provider-name>"

# 使用服务商支持的模型名。
model = "gpt-5.5"

# xhigh 更适合复杂任务，但响应会更慢。
model_reasoning_effort = "xhigh"

# 支持时关闭响应存储，减少敏感内容留存。
disable_response_storage = true

# 允许编辑当前工作区，限制更大范围的文件系统写入。
sandbox_mode = "workspace-write"

[model_providers."<provider-name>"]
name = "<provider-display-name>"
base_url = "<your-base-url>"
wire_api = "responses"
requires_openai_auth = true
```

`auth.json` 模板见：

```text
examples/codex-auth.example.json
```

示例：

```json
{
  "OPENAI_API_KEY": "<your-api-key>"
}
```

注意：真实的 `.codex/auth.json` 不应该上传到 GitHub。本仓库只提供 `codex-auth.example.json` 作为模板。

### IDE 插件使用

在 IDE 中使用 Codex 插件时，建议在项目目录下放置 `.codex` 配置目录：

```text
your-project/
  .codex/
    config.toml
    auth.json
```

IDE 内打开插件时，项目目录下的 `.codex` 配置通常更容易被定位，也更适合按项目隔离不同服务商、模型和权限设置。

如果一个项目多人协作，只建议提交 `.codex/config.example.toml` 或 README 说明，不要提交真实 `auth.json`。

## 本地与服务器注意事项

### 路径差异

Windows 路径示例：

```text
C:\Users\用户名\.claude\settings.json
C:\Users\用户名\.codex\config.toml
C:\Users\用户名\.codex\auth.json
```

Linux/macOS/服务器路径示例：

```text
~/.claude/settings.json
~/.codex/config.toml
~/.codex/auth.json
```

项目级 Codex 配置示例：

```text
your-project/.codex/config.toml
your-project/.codex/auth.json
```

### 环境变量刷新

如果通过系统环境变量配置 API 地址和密钥，修改后通常需要重启终端或 IDE。

在服务器上，建议先在当前 shell 中临时验证：

```bash
export ANTHROPIC_BASE_URL="<your-base-url>"
export ANTHROPIC_AUTH_TOKEN="<your-anthropic-auth-token>"
```

确认可用后，再写入 shell 配置文件或服务启动脚本。

### 权限与密钥

远程服务器上需要特别注意文件权限。建议只让当前用户读取鉴权文件：

```bash
chmod 600 ~/.codex/auth.json
chmod 600 ~/.claude/settings.json
```

如果多人共用服务器，不建议把 token 放在公共目录、共享项目目录或命令历史中。

### 中转地址与模型名

使用中转服务时，要同时确认三件事：

- `base_url` 或 `ANTHROPIC_BASE_URL` 是否是服务商要求的完整地址。
- `model` 或 `selectedModel` 是否是服务商支持的模型名。
- 鉴权方式是否和工具期望一致，例如 OpenAI key、Anthropic token 或服务商自定义 token。

## 排错清单

遇到插件不生效、模型无法调用、鉴权失败时，建议按这个顺序排查：

- 配置文件是不是写在了当前入口实际读取的位置。
- JSON、JSONC、TOML 格式是否有效。
- token、base URL、模型名是否有多余空格。
- 终端或 IDE 是否已经重启。
- 系统环境变量是否覆盖了文件配置。
- `ccswitch` 修改的文件是否就是当前工具读取的文件。
- 本地和服务器是否使用了不同用户，例如 `root` 和普通用户。
- 项目目录下是否存在另一份 `.codex` 配置。
- GitHub 仓库中是否误提交了真实 `auth.json`、`.env` 或 token。

## 安全建议

提交到 GitHub 前，至少执行一次：

```bash
git status --short
```

确认没有这些内容：

- `.codex/auth.json`
- `.env`
- 真实 API key
- 真实 token
- 私有中转地址

本仓库的 `.gitignore` 已经忽略常见密钥文件，但不要完全依赖忽略规则。提交前人工看一眼 diff，仍然是最稳妥的做法。

## License

本文档采用 [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/) 授权。
