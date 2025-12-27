# 2.5 交互工具

Claude Code 提供了与用户交互的工具，让 AI 能够在需要时获取澄清或用户决策。

---

## AskUserQuestion 工具

### 功能介绍

AskUserQuestion 让 Claude 能够：
- 获取用户偏好
- 澄清模糊指令
- 获取实现决策
- 提供选项让用户选择

### 基本用法

当 Claude 需要用户输入时，会弹出选项：

```
> 帮我创建一个新的 API 端点

# Claude 可能会问：
# AskUserQuestion:
# questions: [
#   {
#     "question": "你想使用哪种 HTTP 框架？",
#     "header": "Framework",
#     "options": [
#       { "label": "Express", "description": "最流行的 Node.js 框架" },
#       { "label": "Fastify", "description": "高性能框架" },
#       { "label": "Koa", "description": "轻量级框架" }
#     ],
#     "multiSelect": false
#   }
# ]
```

### 问题类型

#### 单选问题

```
# 选择一个选项
{
  "question": "使用哪种数据库？",
  "header": "Database",
  "options": [
    { "label": "PostgreSQL", "description": "关系型数据库" },
    { "label": "MongoDB", "description": "文档数据库" },
    { "label": "SQLite", "description": "嵌入式数据库" }
  ],
  "multiSelect": false
}
```

#### 多选问题

```
# 可以选择多个
{
  "question": "需要添加哪些功能？",
  "header": "Features",
  "options": [
    { "label": "认证", "description": "用户登录/注册" },
    { "label": "缓存", "description": "Redis 缓存层" },
    { "label": "日志", "description": "结构化日志" },
    { "label": "监控", "description": "性能监控" }
  ],
  "multiSelect": true
}
```

#### 多个问题

```
# 一次问多个问题（最多4个）
questions: [
  { "question": "选择框架?", "header": "Framework", ... },
  { "question": "选择数据库?", "header": "Database", ... },
  { "question": "需要 TypeScript?", "header": "TypeScript", ... }
]
```

### 自定义输入

用户始终可以选择 "Other" 提供自定义答案：

```
# 当选项都不满足时
# 用户可以选择 Other 并输入自己的答案
> 我想使用 Hono 框架
```

---

## EnterPlanMode 工具

### 功能介绍

EnterPlanMode 让 Claude 进入规划模式，用于：
- 设计复杂实现方案
- 探索多种可能性
- 在执行前获得用户确认

### 使用场景

#### 何时使用

- 多种有效方案可选择
- 需要重大架构决策
- 大规模代码变更
- 需求不明确需要澄清

#### 何时不使用

- 简单、直接的任务
- 解决方案明确
- 小 bug 修复
- 纯研究任务

### 规划流程

```
> 帮我重构整个认证系统

# Claude 使用 EnterPlanMode:
# 1. 进入规划模式
# 2. 探索代码库
# 3. 分析现有实现
# 4. 设计多个方案
# 5. 写入 plan.md
# 6. 使用 ExitPlanMode 等待用户确认

# 用户可以：
# - 批准方案
# - 要求修改
# - 选择其他方案
```

### 规划输出

规划模式会生成详细的实现计划：

```markdown
# 认证系统重构计划

## 方案概述
将现有的 session-based 认证迁移到 JWT + Refresh Token 模式

## 实现步骤

### 阶段 1：准备工作
1. 安装必要依赖（jsonwebtoken, bcrypt）
2. 创建 JWT 工具函数
3. 设计 token 存储策略

### 阶段 2：核心实现
1. 实现 token 生成/验证
2. 创建认证中间件
3. 更新登录/注册接口

### 阶段 3：迁移与测试
1. 添加兼容层
2. 编写测试用例
3. 渐进式迁移

## 关键文件
- src/auth/jwt.ts
- src/middleware/auth.ts
- src/routes/auth.ts

## 风险与考虑
- 现有 session 的过渡期
- Token 失效处理
- 安全性考虑
```

---

## 实验练习

### 实验 2.5.1：选项问答

```
# 在 Claude Code 中：
claude

> 帮我创建一个新的 Node.js 项目

# 观察 Claude 如何询问：
# - 使用什么框架？
# - 是否使用 TypeScript？
# - 需要哪些功能？

# 尝试选择不同选项，观察结果差异
```

### 实验 2.5.2：多选问题

```
> 帮我配置 ESLint，选择合适的规则

# Claude 可能会问需要哪些规则类别：
# - 最佳实践
# - 可能的错误
# - 代码风格
# - ES6+ 特性

# 可以多选
```

### 实验 2.5.3：规划模式

```
> 我想把这个 Express 应用重构为 NestJS，请先规划一下

# Claude 应该：
# 1. 进入规划模式
# 2. 分析现有代码
# 3. 设计迁移方案
# 4. 等待你确认

# 确认后再开始执行
```

### 实验 2.5.4：自定义输入

```
> 帮我选择一个 CSS 框架

# 当 Claude 提供选项时：
# 如果你想用的框架不在列表中
# 选择 "Other" 并输入你的选择
```

---

## 何时 Claude 会主动提问

### 1. 关键决策

```
> 实现用户认证

# Claude 会问认证方式：
# - JWT？
# - Session？
# - OAuth？
```

### 2. 模糊指令

```
> 优化这段代码

# Claude 可能会问优化目标：
# - 性能？
# - 可读性？
# - 内存使用？
```

### 3. 破坏性操作

```
> 删除所有测试文件

# Claude 会确认：
# "确定要删除所有 *.test.js 文件吗？"
```

### 4. 多种方案可选

```
> 添加状态管理

# Claude 会问使用哪个库：
# - Redux
# - Zustand
# - Jotai
# - MobX
```

---

## 与 Hooks 的交互

用户可以配置 hooks 来自动响应某些提问：

```json
// settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "AskUserQuestion"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Question asked: ${question}'"
          }
        ]
      }
    ]
  }
}
```

---

## 最佳实践

### 对于 Claude

1. **不确定时询问**：比假设用户意图更好
2. **提供清晰选项**：每个选项有描述
3. **限制问题数量**：一次最多 4 个问题
4. **合理使用规划模式**：复杂任务才需要

### 对于用户

1. **提供明确指令**：减少需要询问的机会
2. **在提示中说明偏好**：例如 "使用 TypeScript 和 Express"
3. **利用 Other 选项**：当选项不满足时
4. **审查规划**：在执行前检查方案

---

## 第二章总结

| 工具 | 用途 | 关键特性 |
|------|------|----------|
| Read | 读取文件 | 支持图片、PDF、Notebook |
| Write | 创建文件 | 完全覆盖 |
| Edit | 编辑文件 | 精确替换 |
| Glob | 文件搜索 | 模式匹配 |
| Grep | 内容搜索 | 正则表达式 |
| Bash | 执行命令 | 后台任务 |
| WebSearch | 网络搜索 | 实时信息 |
| WebFetch | 获取网页 | 内容提取 |
| Task | 子 Agent | 并行执行 |
| TodoWrite | 任务管理 | 进度跟踪 |
| AskUserQuestion | 用户交互 | 选项/多选 |
| EnterPlanMode | 规划模式 | 方案设计 |

---

## 下一章

[第三章：Slash Commands 自定义命令](../03-slash-commands/README.md) - 学习创建和使用自定义斜杠命令
