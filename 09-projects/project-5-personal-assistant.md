# 项目五：个人编程助手

构建一个高度定制化的个人编程助手，整合所有 Claude Code 功能。

---

## 项目目标

- 定制化配置
- 项目特定命令
- 工作流自动化
- 知识库集成

---

## 项目结构

```
personal-assistant/
├── .claude/
│   ├── settings.json          # 完整配置
│   ├── CLAUDE.md              # 项目知识库
│   └── commands/
│       ├── daily.md           # 日常任务
│       ├── review.md          # 代码审查
│       ├── refactor.md        # 重构助手
│       ├── debug.md           # 调试助手
│       ├── learn.md           # 学习助手
│       └── project/
│           ├── init.md        # 项目初始化
│           ├── status.md      # 项目状态
│           └── summary.md     # 项目总结
├── scripts/
│   ├── morning-routine.sh     # 早晨例行
│   ├── end-of-day.sh          # 一天结束
│   └── weekly-review.sh       # 周回顾
└── templates/
    ├── component.md
    ├── service.md
    └── test.md
```

---

## 步骤 1：完整配置文件

```json
// .claude/settings.json
{
  "model": "claude-sonnet-4-20250514",
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}",
    "EDITOR": "code"
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${HOME}/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  },
  "permissions": {
    "allow": [
      "Read",
      "Write(src/**/*)",
      "Write(tests/**/*)",
      "Write(docs/**/*)",
      "Edit",
      "Bash(npm:*)",
      "Bash(yarn:*)",
      "Bash(pnpm:*)",
      "Bash(git:*)",
      "Bash(docker:*)",
      "Bash(code:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Write(.env*)",
      "Write(*.pem)",
      "Write(*.key)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%H:%M:%S')] $TOOL_INPUT\" >> ~/.claude/command-history.log"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.(ts|tsx|js|jsx)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\" 2>/dev/null || true"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.test\\.(ts|js)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- \"$FILE_PATH\" --passWithNoTests 2>&1 | tail -5 || true"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"任务完成\" with title \"Claude Code\"' 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 步骤 2：项目知识库

```markdown
<!-- .claude/CLAUDE.md -->
# 个人编程助手知识库

## 我的偏好

### 代码风格
- 语言：TypeScript 优先
- 框架：React + Next.js
- 样式：Tailwind CSS
- 测试：Jest + React Testing Library
- 包管理：pnpm

### 命名约定
- 组件：PascalCase (UserProfile.tsx)
- 函数：camelCase (getUserData)
- 常量：UPPER_SNAKE_CASE (MAX_RETRY_COUNT)
- 文件：kebab-case (user-profile.tsx) 或 PascalCase

### 项目结构偏好
```
src/
├── components/     # React 组件
│   ├── ui/        # 基础 UI 组件
│   └── features/  # 功能组件
├── hooks/         # 自定义 Hooks
├── lib/           # 工具库
├── services/      # API 服务
├── types/         # TypeScript 类型
└── utils/         # 工具函数
```

## 常用命令

### 开发
- `pnpm dev` - 启动开发服务器
- `pnpm build` - 构建生产版本
- `pnpm test` - 运行测试
- `pnpm lint` - 运行 lint

### Git 工作流
- 分支命名：feature/xxx, fix/xxx, chore/xxx
- 提交格式：conventional commits

## 当前项目

### 项目名称
[项目描述]

### 技术栈
- 前端：React, TypeScript, Tailwind
- 后端：Node.js, Express
- 数据库：PostgreSQL
- 部署：Vercel

### 重要文件
- `src/app/page.tsx` - 首页
- `src/lib/api.ts` - API 客户端
- `prisma/schema.prisma` - 数据库模型

## 注意事项

- 所有 API 调用需要错误处理
- 使用 React Query 管理服务端状态
- 表单使用 react-hook-form + zod
- 不要在客户端暴露敏感信息
```

---

## 步骤 3：日常任务命令

```markdown
<!-- .claude/commands/daily.md -->
执行日常开发任务：$ARGUMENTS

## 可用任务

### 1. 开始工作 (start)
- 拉取最新代码
- 检查待处理的 PR
- 查看今天的任务

### 2. 代码审查 (review)
- 列出待审查的 PR
- 执行代码审查

### 3. 提交工作 (commit)
- 查看变更
- 生成提交信息
- 提交代码

### 4. 结束工作 (end)
- 总结今日完成的工作
- 推送未推送的提交
- 创建明日待办

## 执行

根据参数执行对应任务：$ARGUMENTS

如果没有指定任务，显示任务列表并询问要执行哪个。
```

### 调试助手

```markdown
<!-- .claude/commands/debug.md -->
调试问题：$ARGUMENTS

## 调试流程

1. **理解问题**
   - 错误信息是什么？
   - 期望行为是什么？
   - 实际行为是什么？

2. **收集信息**
   - 读取相关代码
   - 查看错误日志
   - 检查最近的变更

3. **分析原因**
   - 使用深度思考模式
   - 逐步追踪问题
   - 识别根本原因

4. **提供解决方案**
   - 修复代码
   - 解释修复原因
   - 添加测试防止回归

## 问题描述

$ARGUMENTS
```

### 学习助手

```markdown
<!-- .claude/commands/learn.md -->
学习主题：$ARGUMENTS

## 学习模式

1. **概念解释**
   - 用简单语言解释
   - 提供类比
   - 说明应用场景

2. **代码示例**
   - 提供可运行的示例
   - 注释关键部分
   - 展示最佳实践

3. **实践练习**
   - 设计小练习
   - 逐步引导完成
   - 提供反馈

4. **深入探索**
   - 相关主题推荐
   - 进阶内容
   - 实际项目应用

## 学习内容

$ARGUMENTS
```

---

## 步骤 4：项目管理命令

```markdown
<!-- .claude/commands/project/status.md -->
检查项目状态

## 收集信息

1. **Git 状态**
   - 当前分支
   - 未提交的变更
   - 落后/领先的提交

2. **依赖状态**
   - 过时的依赖
   - 安全漏洞

3. **测试状态**
   - 运行测试
   - 覆盖率

4. **代码质量**
   - Lint 错误
   - TypeScript 错误

## 输出报告

# 项目状态报告

## Git
- 分支：main
- 状态：[clean/dirty]
- 未推送提交：N

## 依赖
- 过时：N 个
- 漏洞：N 个

## 测试
- 通过：N
- 失败：N
- 覆盖率：N%

## 代码质量
- Lint 错误：N
- 类型错误：N

## 建议
- [改进建议]
```

```markdown
<!-- .claude/commands/project/summary.md -->
生成项目总结：$ARGUMENTS

## 分析内容

1. **项目概述**
   - 读取 README 和 package.json
   - 分析项目结构
   - 识别技术栈

2. **代码统计**
   - 文件数量
   - 代码行数
   - 语言分布

3. **架构分析**
   - 主要模块
   - 依赖关系
   - 设计模式

4. **最近活动**
   - 最近的提交
   - 活跃的分支
   - 待处理的 PR

## 输出

生成项目总结文档。

$ARGUMENTS
```

---

## 步骤 5：自动化脚本

```bash
#!/bin/bash
# scripts/morning-routine.sh

echo "🌅 Good morning! Starting daily routine..."

# 更新代码
echo "📥 Pulling latest changes..."
git pull origin main

# 检查依赖
echo "📦 Checking dependencies..."
npm outdated 2>/dev/null | head -10

# 检查待处理 PR
echo "🔍 Checking pending PRs..."
gh pr list --limit 5 2>/dev/null || echo "No GitHub CLI available"

# 启动 Claude
echo "🤖 Starting Claude Code..."
claude -p "/daily start"
```

```bash
#!/bin/bash
# scripts/end-of-day.sh

echo "🌙 End of day routine..."

# 检查未提交的变更
if ! git diff --quiet; then
    echo "⚠️  You have uncommitted changes:"
    git status --short
    read -p "Commit now? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        claude -p "查看变更并生成提交信息，然后提交"
    fi
fi

# 推送未推送的提交
if [ $(git log origin/main..HEAD --oneline | wc -l) -gt 0 ]; then
    echo "📤 Pushing commits..."
    git push origin main
fi

# 生成今日总结
echo "📝 Generating daily summary..."
claude -p "总结今天的 git 提交，生成工作日报"

echo "✅ Done! Have a good evening!"
```

```bash
#!/bin/bash
# scripts/weekly-review.sh

echo "📊 Weekly Review"

# 本周提交统计
echo "=== Commits This Week ==="
git log --since="1 week ago" --oneline | wc -l
git log --since="1 week ago" --oneline | head -20

# 使用 Claude 生成周报
claude -p "分析最近一周的 git 历史，生成周报，包括：
1. 完成的主要工作
2. 代码统计（新增/删除/修改）
3. 下周计划建议"
```

---

## 步骤 6：使用示例

```bash
# 开始一天的工作
./scripts/morning-routine.sh

# 或者直接使用命令
claude
> /daily start

# 调试问题
> /debug 用户登录后没有跳转到首页

# 学习新知识
> /learn React Server Components

# 检查项目状态
> /project:status

# 结束一天的工作
./scripts/end-of-day.sh
```

---

## 步骤 7：快捷别名

添加到 `~/.bashrc` 或 `~/.zshrc`：

```bash
# Claude Code 快捷命令
alias cc="claude"
alias ccd="/daily"
alias ccr="claude -p '/review'"
alias ccs="claude -p '/project:status'"

# 快速启动
alias morning="~/scripts/morning-routine.sh"
alias eod="~/scripts/end-of-day.sh"

# 带上下文启动
alias ccw="claude --continue"  # 继续上次会话
```

---

## 总结

本项目整合了 Claude Code 的所有核心功能：

| 功能 | 应用 |
|------|------|
| 配置文件 | 完整的 settings.json |
| CLAUDE.md | 项目知识库 |
| Slash Commands | 多种自定义命令 |
| Hooks | 自动格式化、测试、通知 |
| MCP Server | GitHub、文件系统、内存 |
| 自动化脚本 | 日常工作流 |

这是一个可以持续迭代的个人助手系统，根据你的实际需求不断优化和扩展。

---

## 第九章总结

| 项目 | 核心技术 | 收获 |
|------|---------|------|
| 代码审查系统 | GitHub MCP, Hooks | CI/CD 集成 |
| 文档自动化 | 自定义命令, 模板 | 文档工作流 |
| 测试自动化 | Hooks, Jest | TDD 工作流 |
| DevOps 助手 | MCP, Bash | 运维自动化 |
| 个人助手 | 全面配置 | 定制化开发环境 |

---

## 课程完结

恭喜你完成了 Claude Code CLI 完全指南的学习！

你现在已经掌握了：
- Claude Code 的所有核心功能
- 自定义命令和 Hooks 系统
- MCP Server 集成
- Agent SDK 开发
- 高级功能应用
- 实战项目构建

继续探索，构建属于你自己的 AI 编程助手！
