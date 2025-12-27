# 2.4 任务管理工具

Claude Code 提供了强大的任务管理工具，包括启动子 Agent 的 Task 工具和任务列表管理的 TodoWrite 工具。

---

## Task 工具 - 子 Agent

### 什么是子 Agent？

Task 工具可以启动专门的子 Agent 来处理复杂任务。每个子 Agent：
- 独立运行
- 有特定的能力和工具
- 可以并行执行
- 返回结果给主对话

### 可用的 Agent 类型

| Agent 类型 | 用途 | 可用工具 |
|------------|------|----------|
| `general-purpose` | 通用任务、搜索代码、多步骤任务 | 所有工具 |
| `Explore` | 快速探索代码库 | Glob, Grep, Read |
| `Plan` | 设计实现方案 | 所有工具 |
| `claude-code-guide` | Claude Code 使用指南 | Glob, Grep, Read, WebFetch, WebSearch |

### 使用场景

#### 1. Explore Agent - 代码探索

```
> 帮我了解这个项目的目录结构和主要组件

# Claude 启动 Explore Agent:
# Task:
#   subagent_type: "Explore"
#   prompt: "探索代码库结构，找出主要组件和入口文件"
#   description: "Explore codebase structure"
```

探索不同深度：

```
# 快速探索
> 快速查看 src 目录下有哪些文件

# Task: prompt 包含 "quick" 或 "快速"

# 中等深度
> 了解 API 端点的组织方式

# Task: prompt 包含 "medium" 或 "适度"

# 深度探索
> 彻底分析认证系统的实现方式

# Task: prompt 包含 "thorough" 或 "彻底"
```

#### 2. Plan Agent - 方案设计

```
> 规划如何实现用户权限系统

# Claude 启动 Plan Agent:
# Task:
#   subagent_type: "Plan"
#   prompt: "设计用户权限系统的实现方案，考虑 RBAC 模型"
#   description: "Plan permission system"
```

#### 3. general-purpose Agent - 通用任务

```
> 在整个代码库中搜索所有使用 deprecated API 的地方

# Task:
#   subagent_type: "general-purpose"
#   prompt: "搜索所有标记为 @deprecated 的 API 调用"
#   description: "Search deprecated APIs"
```

#### 4. claude-code-guide Agent - 使用指南

```
> Claude Code 如何配置 MCP Server？

# Task:
#   subagent_type: "claude-code-guide"
#   prompt: "解释如何在 Claude Code 中配置 MCP Server"
#   description: "MCP configuration guide"
```

### 并行执行多个 Agent

```
> 同时分析前端和后端代码的测试覆盖率

# Claude 可以并行启动多个 Agent:
# Task 1: 分析前端测试
# Task 2: 分析后端测试
# 两个 Agent 同时运行
```

### 后台执行

```
> 在后台分析整个代码库的依赖关系，然后继续我们的对话

# Task:
#   run_in_background: true
#   subagent_type: "general-purpose"
#   prompt: "分析 package.json 和所有 import 语句，生成依赖图"

# Claude 继续主对话，稍后使用 AgentOutputTool 获取结果
```

---

## TodoWrite 工具 - 任务列表

### 功能介绍

TodoWrite 用于创建和管理任务列表：
- 跟踪多步骤任务进度
- 让用户了解当前状态
- 帮助 Claude 不遗漏任务

### 任务状态

| 状态 | 说明 |
|------|------|
| `pending` | 等待处理 |
| `in_progress` | 正在处理 |
| `completed` | 已完成 |

### 使用示例

```
> 帮我重构用户认证模块

# Claude 创建任务列表:
# TodoWrite:
# [
#   { "content": "分析现有认证代码", "status": "in_progress", "activeForm": "分析认证代码中..." },
#   { "content": "设计新的认证架构", "status": "pending", "activeForm": "设计认证架构中..." },
#   { "content": "实现 JWT 验证", "status": "pending", "activeForm": "实现 JWT 验证中..." },
#   { "content": "添加单元测试", "status": "pending", "activeForm": "添加测试中..." },
#   { "content": "更新文档", "status": "pending", "activeForm": "更新文档中..." }
# ]
```

### 任务更新

任务完成后，Claude 会及时更新状态：

```
# 完成第一个任务，开始第二个
# TodoWrite:
# [
#   { "content": "分析现有认证代码", "status": "completed", ... },
#   { "content": "设计新的认证架构", "status": "in_progress", ... },
#   { "content": "实现 JWT 验证", "status": "pending", ... },
#   ...
# ]
```

### 何时使用 TodoWrite

**适合使用的场景：**
- 多步骤任务（3+ 步骤）
- 复杂的代码重构
- 用户提供了多个任务
- 需要跟踪进度的工作

**不适合使用的场景：**
- 简单的单一任务
- 快速问答
- 信息查询

---

## 实验练习

### 实验 2.4.1：使用 Explore Agent

```
# 在一个 Git 项目中：
claude

> 使用探索 Agent 分析这个项目的结构

> 找出这个项目的主要入口文件和配置文件

> 深入分析 src 目录的模块组织方式
```

### 实验 2.4.2：使用 Plan Agent

```
> 我想给这个项目添加一个缓存层，请帮我规划实现方案

> 规划如何将这个项目从 JavaScript 迁移到 TypeScript
```

### 实验 2.4.3：并行 Agent

```
> 同时做以下两件事：
> 1. 分析 src/api 目录的 API 实现
> 2. 分析 src/utils 目录的工具函数

# Claude 应该并行启动两个 Agent
```

### 实验 2.4.4：任务列表管理

```
> 帮我完成以下任务：
> 1. 创建一个新的 React 组件 UserCard
> 2. 为组件添加样式
> 3. 编写组件的单元测试
> 4. 在 App.tsx 中使用这个组件

# 观察 Claude 如何：
# 1. 创建任务列表
# 2. 逐个完成任务
# 3. 更新任务状态
```

---

## Task vs 直接执行

### 何时使用 Task

| 使用 Task | 直接执行 |
|-----------|----------|
| 开放式探索 | 已知文件路径 |
| 多轮搜索可能需要 | 简单的单次搜索 |
| 复杂分析任务 | 简单查询 |
| 需要并行处理 | 顺序操作 |

### 示例对比

```
# ✅ 使用 Task - 开放式探索
> 找出这个项目中所有处理用户输入的地方

# Task: Explore agent 会系统性地搜索

# ✅ 直接执行 - 已知目标
> 读取 src/components/Button.tsx

# Read: 直接读取文件
```

---

## 高级用法

### 指定模型

```
> 用 Opus 模型深度分析代码架构

# Task:
#   model: "opus"
#   subagent_type: "Plan"
#   prompt: "..."
```

### 恢复 Agent

```
# 如果之前的 Agent 没完成，可以恢复
# Task:
#   resume: "previous-agent-id"
```

### 获取后台 Agent 结果

```
> 检查之前启动的代码分析任务完成了吗

# AgentOutputTool:
#   agentId: "abc123"
#   block: false  # 不阻塞，立即返回状态
```

---

## 最佳实践

### 1. 复杂任务才用 Task

```
# ❌ 简单任务不需要 Task
> 读取 package.json

# ✅ 复杂探索使用 Task
> 分析整个项目的依赖关系和模块结构
```

### 2. 善用并行

```
# 如果任务之间无依赖，并行执行
> 同时检查前端和后端的代码规范
```

### 3. 明确的任务描述

```
# ✅ 好的任务描述
> 探索 src/auth 目录，找出认证流程的入口点和主要函数

# ❌ 模糊的描述
> 看看 auth 相关的东西
```

### 4. 任务分解

```
# 大任务分解为小任务
> 重构用户模块：
> 1. 先分析现有代码
> 2. 列出需要修改的文件
> 3. 逐个文件重构
> 4. 运行测试验证
```

---

## 下一节

[2.5 交互工具](./05-interaction-tools.md) - 学习使用 AskUserQuestion 等交互工具
