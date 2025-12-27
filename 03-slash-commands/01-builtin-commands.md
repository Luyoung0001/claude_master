# 3.1 内置命令

Claude Code 提供了一系列内置的斜杠命令，用于会话管理、诊断和配置。

---

## 帮助与信息命令

### /help

显示帮助信息和可用命令列表。

```
> /help

# 输出：
# Claude Code Help
# ================
#
# Available commands:
#   /help     - Show this help message
#   /exit     - Exit the session
#   /clear    - Clear the conversation context
#   /compact  - Compact the context to save tokens
#   /cost     - Show token usage and cost
#   /doctor   - Run diagnostic checks
#   /config   - Show current configuration
#   /model    - View or switch models
#   /init     - Initialize CLAUDE.md file
#
# Custom commands:
#   /your-command  - Your custom commands appear here
```

### /doctor

运行诊断检查，排查常见问题。

```
> /doctor

# 输出：
# ╭────────────────────────────────────────╮
# │ Claude Code Diagnostics                │
# ├────────────────────────────────────────┤
# │ ✓ Node.js version: 20.10.0             │
# │ ✓ Claude Code version: 1.0.0           │
# │ ✓ API key configured                   │
# │ ✓ Network connectivity                 │
# │ ✓ Permissions configured               │
# │ ✓ Working directory accessible         │
# │ ✓ Git repository detected              │
# ╰────────────────────────────────────────╯
```

常见诊断项：
- Node.js 版本兼容性
- API 密钥配置
- 网络连接状态
- 权限配置
- 工作目录访问权限
- Git 仓库状态

### /config

显示当前生效的配置。

```
> /config

# 输出：
# Current Configuration
# =====================
# Model: claude-sonnet-4-20250514
# Working Directory: /Users/user/project
#
# Settings:
#   permissions:
#     allow: ["Read", "Bash(npm:*)"]
#     deny: []
#
# MCP Servers:
#   - filesystem (running)
#
# Hooks:
#   - PreToolUse: 2 hooks configured
```

---

## 会话管理命令

### /exit

退出当前会话。

```
> /exit

# 会话结束，返回终端
# 会话会被保存，可以稍后使用 claude -c 继续
```

### /clear

清除当前会话的上下文。

```
> /clear

# 确认提示：
# Clear conversation context? This cannot be undone. (y/n)

> y

# Context cleared. Starting fresh.
```

**注意**：清除上下文后：
- 之前的对话历史将丢失
- 已读取的文件缓存将清除
- 相当于开始新对话（但保持在同一会话中）

### /compact

压缩上下文以节省 token 和避免超出限制。

```
> /compact

# 输出：
# Compacting context...
#
# Before: 128,000 tokens
# After:  32,000 tokens
# Saved:  96,000 tokens (75%)
#
# Preserved:
#   - Key decisions and conclusions
#   - File modification history
#   - Current task context
#   - Important code snippets
```

压缩保留的内容：
- 重要决策和结论
- 文件修改记录
- 当前任务上下文
- 关键代码片段

压缩移除的内容：
- 详细的中间讨论
- 重复的内容
- 已解决问题的详细分析

---

## 成本与模型命令

### /cost

显示当前会话的 token 使用量和估算成本。

```
> /cost

# 输出：
# ╭────────────────────────────────────────╮
# │ Session Cost Summary                   │
# ├────────────────────────────────────────┤
# │ Input tokens:   45,234                 │
# │ Output tokens:  12,456                 │
# │ Total tokens:   57,690                 │
# │                                        │
# │ Estimated cost: $0.42                  │
# │                                        │
# │ Context usage:  28.5% (57,690/200,000) │
# ╰────────────────────────────────────────╯
```

### /model

查看当前模型或切换模型。

```
# 查看当前模型
> /model

# Current model: claude-sonnet-4-20250514
#
# Available models:
#   - claude-opus-4-5-20250514 (opus)
#   - claude-sonnet-4-20250514 (sonnet) [current]
#   - claude-haiku-3-5-20241022 (haiku)
```

```
# 切换模型
> /model opus

# Model switched to claude-opus-4-5-20250514
```

---

## 项目初始化命令

### /init

初始化项目的 CLAUDE.md 文件，包含项目结构和约定。

```
> /init

# Analyzing project structure...
#
# Creating CLAUDE.md with:
#   - Project overview
#   - Directory structure
#   - Key files and their purposes
#   - Development conventions
#   - Common commands
#
# CLAUDE.md created successfully!
```

生成的 CLAUDE.md 示例：

```markdown
# Project: my-app

## Overview
A React application with TypeScript and Vite.

## Directory Structure
```
src/
├── components/    # React components
├── hooks/         # Custom React hooks
├── utils/         # Utility functions
├── api/           # API client and endpoints
└── types/         # TypeScript type definitions
```

## Key Files
- `src/main.tsx` - Application entry point
- `src/App.tsx` - Root component
- `vite.config.ts` - Vite configuration

## Development Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm test` - Run tests

## Conventions
- Use functional components with hooks
- Follow ESLint and Prettier rules
- Write tests for new features
```

---

## 实验练习

### 实验 3.1.1：信息命令

```
# 在 Claude Code 中：
claude

> /help
# 查看所有可用命令

> /doctor
# 检查环境是否正常

> /config
# 查看当前配置

> /model
# 查看当前使用的模型
```

### 实验 3.1.2：会话管理

```
> 讨论一下 React 的生命周期

> /cost
# 查看消耗了多少 token

> 详细解释每个生命周期方法

> /cost
# 再次查看成本变化

> /compact
# 压缩上下文

> /cost
# 比较压缩前后的 token 使用量
```

### 实验 3.1.3：模型切换

```
> /model
# 查看可用模型

> /model haiku
# 切换到 Haiku

> 什么是 TypeScript 的类型推断？
# 使用 Haiku 回答简单问题

> /model sonnet
# 切换回 Sonnet

> 帮我设计一个复杂的类型系统
# 使用 Sonnet 处理复杂任务
```

### 实验 3.1.4：项目初始化

```bash
# 创建一个新项目
mkdir -p /tmp/init-test
cd /tmp/init-test
npm init -y
mkdir -p src/{components,hooks,utils}
touch src/index.js src/App.js

# 在 Claude Code 中：
claude

> /init
# 观察生成的 CLAUDE.md 内容

> 读取 CLAUDE.md
# 查看完整内容
```

---

## 命令快捷方式

某些命令支持简写：

| 完整命令 | 简写 |
|----------|------|
| `/help` | `/h` |
| `/exit` | `/q` |
| `/clear` | `/c` |
| `/model opus` | `/m opus` |

---

## 内置命令总结

| 命令 | 用途 | 何时使用 |
|------|------|----------|
| `/help` | 获取帮助 | 不确定有什么命令时 |
| `/doctor` | 诊断问题 | 遇到异常时 |
| `/config` | 查看配置 | 需要确认配置时 |
| `/exit` | 退出会话 | 结束工作时 |
| `/clear` | 清除上下文 | 开始新话题时 |
| `/compact` | 压缩上下文 | 上下文接近限制时 |
| `/cost` | 查看成本 | 监控使用量时 |
| `/model` | 切换模型 | 需要不同能力时 |
| `/init` | 初始化项目 | 新项目开始时 |

---

## 下一节

[3.2 自定义命令开发](./02-custom-commands.md) - 学习创建自己的斜杠命令
