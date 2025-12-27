# Claude Code CLI 完全指南 - 实验计划大纲

> 通过一系列实验彻底掌握 Claude Code CLI 的所有玩法

## 目录

1. [基础入门](#1-基础入门)
2. [核心工具使用](#2-核心工具使用)
3. [Slash Commands 自定义命令](#3-slash-commands-自定义命令)
4. [Hooks 钩子系统](#4-hooks-钩子系统)
5. [MCP Server 集成](#5-mcp-server-集成)
6. [IDE 集成](#6-ide-集成)
7. [Agent SDK 开发](#7-agent-sdk-开发)
8. [高级功能与技巧](#8-高级功能与技巧)
9. [实战项目](#9-实战项目)

---

## 1. 基础入门

### 1.1 安装与配置
- [ ] Claude Code CLI 安装方式（npm/homebrew）
- [ ] 环境变量配置（ANTHROPIC_API_KEY / 自定义端点）
- [ ] settings.json 配置文件详解
- [ ] 模型选择：opus / sonnet / haiku 的差异与适用场景

### 1.2 基本命令
- [ ] `claude` - 启动交互式会话
- [ ] `claude -p "prompt"` - 单次提问模式
- [ ] `claude -c` / `claude --continue` - 继续上次会话
- [ ] `claude --resume` - 恢复特定会话
- [ ] `claude --help` - 帮助信息
- [ ] `claude --version` - 版本信息

### 1.3 会话管理
- [ ] 会话历史查看与恢复
- [ ] 上下文自动摘要机制
- [ ] 多会话并行管理

---

## 2. 核心工具使用

### 2.1 文件操作工具
- [ ] Read - 读取文件（支持图片、PDF、Jupyter Notebook）
- [ ] Write - 写入新文件
- [ ] Edit - 精确编辑现有文件
- [ ] Glob - 文件模式匹配搜索
- [ ] Grep - 内容搜索（基于 ripgrep）

### 2.2 代码执行工具
- [ ] Bash - 执行 shell 命令
- [ ] 后台任务运行与监控
- [ ] 超时设置与控制

### 2.3 Web 工具
- [ ] WebSearch - 网络搜索
- [ ] WebFetch - 获取网页内容

### 2.4 任务管理工具
- [ ] TodoWrite - 任务列表管理
- [ ] Task - 启动子 Agent 处理复杂任务
- [ ] 子 Agent 类型：general-purpose / Explore / Plan / claude-code-guide

### 2.5 交互工具
- [ ] AskUserQuestion - 向用户提问
- [ ] EnterPlanMode / ExitPlanMode - 规划模式

---

## 3. Slash Commands 自定义命令

### 3.1 内置命令
- [ ] `/help` - 帮助
- [ ] `/clear` - 清除上下文
- [ ] `/compact` - 压缩上下文
- [ ] `/cost` - 查看 token 使用量
- [ ] `/doctor` - 诊断问题
- [ ] `/init` - 初始化 CLAUDE.md

### 3.2 自定义命令开发
- [ ] 命令文件位置：`.claude/commands/`
- [ ] 命令格式：Markdown 文件
- [ ] 参数传递：`$ARGUMENTS`
- [ ] 实验：创建自定义代码审查命令
- [ ] 实验：创建自动生成文档命令
- [ ] 实验：创建项目脚手架命令

### 3.3 命令最佳实践
- [ ] 命令模板设计
- [ ] 复用与组合命令
- [ ] 团队共享命令库

---

## 4. Hooks 钩子系统

### 4.1 Hook 类型
- [ ] PreToolUse - 工具调用前
- [ ] PostToolUse - 工具调用后
- [ ] Notification - 通知钩子
- [ ] Stop - 停止条件钩子

### 4.2 Hook 配置
- [ ] settings.json 中的 hooks 配置
- [ ] 匹配器（matchers）配置
- [ ] 命令执行与环境变量

### 4.3 实验项目
- [ ] 实验：代码格式化自动化（保存后自动 prettier）
- [ ] 实验：测试自动运行（代码修改后自动测试）
- [ ] 实验：Git 操作审计（记录所有 git 命令）
- [ ] 实验：敏感文件保护（阻止修改特定文件）
- [ ] 实验：自定义审批流程（危险操作需确认）

---

## 5. MCP Server 集成

### 5.1 MCP 协议基础
- [ ] Model Context Protocol 概念
- [ ] 资源（Resources）与工具（Tools）
- [ ] 传输层：stdio / HTTP

### 5.2 MCP Server 配置
- [ ] settings.json 中的 mcpServers 配置
- [ ] 环境变量与参数传递
- [ ] 多 Server 并行使用

### 5.3 常用 MCP Server
- [ ] 文件系统 MCP Server
- [ ] 数据库 MCP Server（PostgreSQL/MySQL）
- [ ] GitHub MCP Server
- [ ] Puppeteer/Playwright MCP Server
- [ ] Slack/Discord MCP Server

### 5.4 实验项目
- [ ] 实验：连接本地数据库进行查询
- [ ] 实验：集成 GitHub 自动化 PR 操作
- [ ] 实验：网页自动化测试
- [ ] 实验：自定义 MCP Server 开发

---

## 6. IDE 集成

### 6.1 VS Code 集成
- [ ] 扩展安装与配置
- [ ] 快捷键设置
- [ ] 内联代码建议
- [ ] 侧边栏会话管理
- [ ] 终端集成

### 6.2 JetBrains IDE 集成
- [ ] 插件安装
- [ ] 功能配置
- [ ] 代码补全集成

### 6.3 其他编辑器
- [ ] Vim/Neovim 集成
- [ ] Emacs 集成
- [ ] Zed 集成

---

## 7. Agent SDK 开发

### 7.1 SDK 基础
- [ ] TypeScript SDK 安装与配置
- [ ] Python SDK 安装与配置
- [ ] 基本 Agent 创建

### 7.2 工具定义
- [ ] 自定义工具 schema
- [ ] 工具执行函数
- [ ] 工具组合与编排

### 7.3 会话管理
- [ ] 会话持久化
- [ ] 上下文管理
- [ ] 多轮对话处理

### 7.4 实验项目
- [ ] 实验：构建代码审查 Agent
- [ ] 实验：构建文档生成 Agent
- [ ] 实验：构建测试生成 Agent
- [ ] 实验：多 Agent 协作系统

---

## 8. 高级功能与技巧

### 8.1 Extended Thinking（扩展思考）
- [ ] 启用方式与配置
- [ ] 适用场景
- [ ] Token 成本优化

### 8.2 Vision 能力
- [ ] 图片分析
- [ ] 截图理解
- [ ] 图表解读
- [ ] UI 设计分析

### 8.3 性能优化
- [ ] Token 使用优化
- [ ] 上下文压缩策略
- [ ] 并行任务处理
- [ ] 缓存策略

### 8.4 安全与权限
- [ ] 沙箱模式
- [ ] 文件访问控制
- [ ] 敏感信息处理
- [ ] 审计日志

### 8.5 调试技巧
- [ ] `/doctor` 诊断
- [ ] 日志查看
- [ ] 常见问题排查

---

## 9. 实战项目

### 9.1 项目一：智能代码审查系统
- 集成 GitHub PR
- 自动检测代码问题
- 生成审查报告
- 自动修复建议

### 9.2 项目二：文档自动化
- 代码注释生成
- README 自动更新
- API 文档生成
- 变更日志维护

### 9.3 项目三：测试自动化
- 单元测试生成
- 集成测试编写
- 测试覆盖率分析
- 回归测试自动化

### 9.4 项目四：DevOps 助手
- CI/CD 配置生成
- 部署脚本编写
- 监控告警分析
- 性能问题诊断

### 9.5 项目五：个人编程助手
- 定制化配置
- 项目特定命令
- 工作流自动化
- 知识库集成

---

## 附录

### A. 配置文件模板
- settings.json 完整模板
- .claude/commands/ 命令模板
- hooks 配置模板
- MCP Server 配置模板

### B. 常用命令速查表

### C. 故障排查指南

### D. 资源链接
- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [MCP 协议规范](https://modelcontextprotocol.io)
- [Agent SDK 文档](https://docs.anthropic.com/claude-agent-sdk)
- [GitHub 仓库](https://github.com/anthropics/claude-code)

---

## 学习路径建议

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

---

*最后更新：2025-12-27*
