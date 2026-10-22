# CatCode

> 一个运行在终端里的 AI 编程助手 / A terminal-based AI coding assistant, built with Python and Textual.

CatCode 是一个可交互的命令行 AI 编码 Agent：在终端或浏览器里和模型对话，让它读写文件、执行命令、管理会话，并支持子代理、多代理协作、MCP 工具、钩子和持久记忆。

CatCode is an interactive CLI coding agent: talk to an LLM in the terminal (or in a browser UI), let it read/write files and run commands, with sub-agents, multi-agent teams, MCP tools, hooks, and persistent memory.

---

## 功能特性 / Features

- **终端 TUI**：基于 [Textual](https://textual.textualize.io/) 的全屏交互界面，也支持一次性非交互调用。
- **多供应商**：支持 Anthropic、OpenAI 以及任意 OpenAI 兼容接口（DeepSeek / Kimi / GLM / Qwen / MiniMax 等）。
- **权限模式**：`default` / `acceptEdits` / `plan` / `bypassPermissions`，逐级放开写操作。
- **工具集**：读文件、写文件、编辑文件、grep、glob、执行命令等内置工具。
- **子代理与团队**：fork 子代理、任务管理、Lead/队员协作与协调器模式。
- **MCP**：通过 stdio 或 HTTP(S) 接入外部 MCP 服务器。
- **钩子 Hooks**：在指定事件上运行自定义脚本。
- **记忆系统**：自动记忆、召回与会话记忆。
- **工作区隔离 Worktree**：为任务创建隔离的改动工作区。
- **文件历史与回滚**：记录改动历史，支持 `/rewind` 回到历史版本。
- **远程模式**：`--remote` 启动 WebSocket 服务，在浏览器中使用。

---

## 环境要求 / Requirements

- Python **3.11+**
- 一个可用的模型 API 密钥（Anthropic / OpenAI / 兼容接口）
- （可选）[uv](https://docs.astral.sh/uv/)；没有 uv 也可以直接用 pip

---

## 安装 / Installation

### 方式一：pip（推荐，无需 uv）

```powershell
# Windows PowerShell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e .
```

```bash
# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

### 方式二：uv

```bash
uv sync
uv run catcode
```

安装完成后即可使用 `catcode` 命令。

---

## 配置 / Configuration

复制示例配置并填写供应商：

```bash
mkdir -p .catcode
cp config.example.yaml .catcode/config.yaml
```

密钥优先使用环境变量，不要把真实 key 提交进仓库：

```powershell
# PowerShell
$env:ANTHROPIC_API_KEY = "sk-..."      # anthropic 协议
$env:OPENAI_API_KEY = "sk-..."          # openai / openai-compat 协议
```

```bash
# bash
export ANTHROPIC_API_KEY="sk-..."
export OPENAI_API_KEY="sk-..."
```

也可以直接把 key 填进 `.catcode/config.yaml` 的 `api_key` 字段（该文件已被 `.gitignore` 忽略）。

配置文件查找顺序：项目目录 `.catcode/config.yaml` → 项目目录 `.catcode/config.local.yaml` → 用户主目录 `~/.catcode/config.yaml`。

### 最小示例（OpenAI 兼容接口）

```yaml
providers:
  - name: my-compat
    protocol: openai-compat
    base_url: https://your-endpoint/v1
    api_key: ""          # 留空则读取 OPENAI_API_KEY
    model: your-model-name

permission_mode: default
```

---

## 使用 / Usage

```bash
# 启动交互式 TUI
catcode

# 一次性非交互执行（结果打印到 stdout）
catcode -p "解释当前目录的项目结构"

# 指定权限模式
catcode --mode plan
catcode --mode acceptEdits

# 远程模式：浏览器访问 http://localhost:18888
catcode --remote

# 非交互流式 JSON 输出
catcode -p "重构这个函数" --output-format stream-json
```

### 内置斜杠命令 / Built-in slash commands

在 TUI 输入框中输入 `/` 查看，常用命令：

| 命令 | 说明 |
| --- | --- |
| `/help` (`/h`, `/?`) | 列出全部命令 |
| `/clear` | 清空当前对话 |
| `/compact` (`/c`) | 压缩上下文 |
| `/plan` (`/p`) | 进入规划模式 |
| `/review` | 让模型复审当前改动 |
| `/rewind` | 回到历史版本 |
| `/session` | 会话管理 |
| `/memory` | 查看 / 清除 / 编辑记忆 |
| `/mcp` | 查看已连接的 MCP 服务器 |
| `/tasks` | 查看子代理任务 |
| `/status` | 查看状态 |
| `/worktree` | 工作区管理 |
| `/skill` | 技能管理 |
| `/trace` | 查看执行轨迹 |
| `/sandbox` | 沙箱设置 |

---

## 测试 / Running tests

```bash
pip install pytest pytest-asyncio
pytest -q
```

---

## 项目结构 / Project layout

```
catcode/
├── __main__.py        # 入口：argparse、TUI / -p / --remote 分支
├── app.py             # Textual TUI 主应用
├── agent.py           # Agent 主循环
├── client.py          # Anthropic / OpenAI 客户端封装
├── config.py          # 配置加载与合并
├── conversation.py    # 会话管理
├── tools/             # 内置工具（文件、grep、glob、bash…）
├── agents/            # 子代理 / fork / 任务管理
├── teams/             # 多代理协作与协调器
├── mcp/               # MCP 客户端与工具封装
├── hooks/             # 钩子引擎
├── memory/            # 记忆 / 召回 / 合并
├── skills/            # 技能加载与执行
├── worktree/          # 隔离工作区
├── permissions/       # 权限模式与校验
├── commands/          # 斜杠命令注册与处理
└── remote.py          # --remote WebSocket + 浏览器 UI
tests/                 # pytest 测试
config.example.yaml    # 配置样例（无密钥）
```

---

## 已知问题 / Known issues

- **Windows 传统控制台（conhost）下的中文/日文输入法**：Textual 在 Windows 控制台对 IME 的支持尚未完善，候选词上屏可能闪烁或需要回车后才显示；可改用 Windows Terminal，或先在别处复制后 `Ctrl+V` 粘贴到输入框。
- 模型调用会产生第三方 API 费用，请自行承担并遵守对应供应商条款。

---

## 开源协议 / License

[MIT](LICENSE) © 2026 casey
