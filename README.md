# Claude Code CLI 完全指南

> 通过系统性的实验，彻底掌握 Claude Code CLI 的所有功能

[![Claude Code](https://img.shields.io/badge/Claude%20Code-CLI-blueviolet)](https://claude.ai)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 简介

这是一份全面的 Claude Code CLI 学习指南，包含 9 个章节、40+ 篇详细教程，从基础入门到高级应用，帮助你彻底掌握 Claude Code 的所有功能。

## 目录

### 第一部分：基础篇

| 章节 | 内容 | 预计时间 |
|------|------|----------|
| [第1章：基础入门](./01-basics/README.md) | 安装配置、基本命令、会话管理 | 2-3 小时 |
| [第2章：核心工具使用](./02-core-tools/README.md) | 文件操作、Bash、Web工具、任务管理 | 4-5 小时 |

### 第二部分：进阶篇

| 章节 | 内容 | 预计时间 |
|------|------|----------|
| [第3章：Slash Commands](./03-slash-commands/README.md) | 内置命令、自定义命令开发、最佳实践 | 2-3 小时 |
| [第4章：Hooks 钩子系统](./04-hooks/README.md) | PreToolUse、PostToolUse、自动化工作流 | 3-4 小时 |
| [第5章：MCP Server 集成](./05-mcp-server/README.md) | 协议基础、配置、常用Server、自定义开发 | 4-5 小时 |

### 第三部分：高级篇

| 章节 | 内容 | 预计时间 |
|------|------|----------|
| [第6章：IDE 集成](./06-ide-integration/README.md) | VS Code、JetBrains、Vim/Neovim、Zed | 2-3 小时 |
| [第7章：Agent SDK 开发](./07-agent-sdk/README.md) | TypeScript/Python SDK、工具定义、构建Agent | 4-5 小时 |
| [第8章：高级功能](./08-advanced-features/README.md) | Extended Thinking、Vision、性能优化、安全 | 3-4 小时 |

### 第四部分：实战篇

| 章节 | 内容 | 预计时间 |
|------|------|----------|
| [第9章：实战项目](./09-projects/README.md) | 5个完整项目实战 | 10-20 小时 |

## 快速开始

### 安装 Claude Code

```bash
# 使用 npm 安装
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version
```

### 配置 API Key

```bash
# 方式一：环境变量
export ANTHROPIC_API_KEY="sk-ant-xxxxx"

# 方式二：首次运行时登录
claude
```

### 开始使用

```bash
# 启动交互式会话
claude

# 单次提问
claude -p "什么是 TypeScript？"

# 继续上次会话
claude -c
```

## 章节详情

### 第1章：基础入门
- [1.1 安装与配置](./01-basics/01-installation.md) - 多种安装方式、环境变量、settings.json
- [1.2 基本命令](./01-basics/02-basic-commands.md) - 交互模式、单次提问、管道使用
- [1.3 会话管理](./01-basics/03-session-management.md) - 会话恢复、上下文压缩、多会话

### 第2章：核心工具使用
- [2.1 文件操作工具](./02-core-tools/01-file-tools.md) - Read、Write、Edit、Glob、Grep
- [2.2 代码执行工具](./02-core-tools/02-bash-tools.md) - Bash、后台任务、超时控制
- [2.3 Web 工具](./02-core-tools/03-web-tools.md) - WebSearch、WebFetch
- [2.4 任务管理工具](./02-core-tools/04-task-tools.md) - Task、TodoWrite、子Agent
- [2.5 交互工具](./02-core-tools/05-interaction-tools.md) - AskUserQuestion、规划模式

### 第3章：Slash Commands
- [3.1 内置命令](./03-slash-commands/01-builtin-commands.md) - /help、/cost、/compact、/init
- [3.2 自定义命令开发](./03-slash-commands/02-custom-commands.md) - 命令格式、参数传递
- [3.3 命令最佳实践](./03-slash-commands/03-best-practices.md) - 设计原则、团队共享

### 第4章：Hooks 钩子系统
- [4.1 Hook 类型与配置](./04-hooks/01-hook-types.md) - PreToolUse、PostToolUse、匹配器
- [4.2 实用 Hook 示例](./04-hooks/02-hook-examples.md) - 格式化、测试、安全防护
- [4.3 高级 Hook 技巧](./04-hooks/03-advanced-hooks.md) - 条件执行、外部脚本、状态管理

### 第5章：MCP Server 集成
- [5.1 MCP 协议基础](./05-mcp-server/01-mcp-basics.md) - Resources、Tools、通信机制
- [5.2 配置 MCP Server](./05-mcp-server/02-mcp-configuration.md) - settings.json配置
- [5.3 常用 MCP Server](./05-mcp-server/03-common-servers.md) - filesystem、postgres、github
- [5.4 自定义 MCP Server](./05-mcp-server/04-custom-server.md) - TypeScript/Python 开发

### 第6章：IDE 集成
- [6.1 VS Code 集成](./06-ide-integration/01-vscode.md) - 扩展安装、快捷键、工作流
- [6.2 JetBrains 集成](./06-ide-integration/02-jetbrains.md) - 插件配置、使用场景
- [6.3 其他编辑器](./06-ide-integration/03-other-editors.md) - Vim、Emacs、Zed

### 第7章：Agent SDK 开发
- [7.1 SDK 基础](./07-agent-sdk/01-sdk-basics.md) - 安装、消息、流式响应
- [7.2 工具定义与使用](./07-agent-sdk/02-tools.md) - 工具 Schema、执行实现
- [7.3 构建实用 Agent](./07-agent-sdk/03-building-agents.md) - 代码审查、文档生成Agent

### 第8章：高级功能
- [8.1 Extended Thinking](./08-advanced-features/01-extended-thinking.md) - 深度推理模式
- [8.2 Vision 能力](./08-advanced-features/02-vision.md) - 图片分析、UI审查
- [8.3 性能优化](./08-advanced-features/03-performance.md) - Token优化、成本控制
- [8.4 安全与权限](./08-advanced-features/04-security.md) - 权限配置、敏感文件保护

### 第9章：实战项目
- [项目1：智能代码审查系统](./09-projects/project-1-code-review.md)
- [项目2：文档自动化](./09-projects/project-2-documentation.md)
- [项目3：测试自动化](./09-projects/project-3-testing.md)
- [项目4：DevOps 助手](./09-projects/project-4-devops.md)
- [项目5：个人编程助手](./09-projects/project-5-personal-assistant.md)

## 学习路径

```
Week 1: 基础入门 + 核心工具
    ↓
Week 2: Slash Commands + Hooks
    ↓
Week 3: MCP Server 集成
    ↓
Week 4: IDE 集成 + Agent SDK
    ↓
Week 5-6: 实战项目
```

## 功能速查

### 常用命令

| 命令 | 说明 |
|------|------|
| `claude` | 启动交互会话 |
| `claude -p "..."` | 单次提问 |
| `claude -c` | 继续上次会话 |
| `claude --model opus` | 使用 Opus 模型 |
| `/help` | 查看帮助 |
| `/cost` | 查看 Token 使用量 |
| `/compact` | 压缩上下文 |

### 核心工具

| 工具 | 功能 |
|------|------|
| Read | 读取文件（支持图片、PDF） |
| Write | 创建/覆盖文件 |
| Edit | 精确编辑文件 |
| Glob | 文件模式匹配 |
| Grep | 内容搜索 |
| Bash | 执行 Shell 命令 |
| WebSearch | 网络搜索 |
| Task | 启动子 Agent |

### 配置文件位置

| 文件 | 位置 |
|------|------|
| 用户配置 | `~/.claude/settings.json` |
| 项目配置 | `.claude/settings.json` |
| 自定义命令 | `.claude/commands/*.md` |
| 项目知识库 | `CLAUDE.md` |

## 资源链接

- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [MCP 协议规范](https://modelcontextprotocol.io)
- [Anthropic API 文档](https://docs.anthropic.com)
- [GitHub 仓库](https://github.com/anthropics/claude-code)

## 贡献

欢迎提交 Issue 和 Pull Request 来改进这份指南！

## 许可证

MIT License

---

**Happy Coding with Claude! 🤖**
