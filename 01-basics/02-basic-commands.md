# 1.2 基本命令

Claude Code 提供了丰富的命令行接口，本节将详细介绍所有基本命令及其用法。

---

## 启动交互式会话

### 基本启动

```bash
# 启动新的交互式会话
claude

# 在特定目录启动
claude /path/to/project
```

启动后进入交互模式，可以持续对话。按 `Ctrl+C` 或输入 `/exit` 退出。

### 交互模式界面

```
╭─────────────────────────────────────────────────────────────╮
│ Claude Code                                                  │
│ Model: claude-sonnet-4-20250514                              │
│ Context: 12,345 tokens                                       │
╰─────────────────────────────────────────────────────────────╯

> 你好，请帮我分析这个项目的结构
```

---

## 单次提问模式

### 使用 -p/--print 参数

```bash
# 单次提问，输出后退出
claude -p "解释什么是 TypeScript 的泛型"

# 等价于
claude --print "解释什么是 TypeScript 的泛型"
```

### 结合管道使用

```bash
# 分析文件内容
cat package.json | claude -p "分析这个项目的依赖"

# 分析命令输出
git diff | claude -p "总结这些代码变更"

# 分析错误日志
npm run build 2>&1 | claude -p "分析这些错误并提供修复建议"
```

### 指定输出格式

```bash
# 输出为 JSON
claude -p "列出 5 种排序算法" --output-format json

# 输出为 Markdown
claude -p "写一个 README 模板" --output-format markdown

# 流式输出（默认）
claude -p "写一个长故事" --stream

# 非流式输出
claude -p "写一个长故事" --no-stream
```

---

## 会话继续与恢复

### 继续上次会话

```bash
# 继续最近的会话
claude -c

# 等价于
claude --continue
```

这会恢复上次会话的完整上下文，包括之前的对话历史。

### 恢复特定会话

```bash
# 查看所有会话
claude --list-sessions

# 输出示例：
# Session ID                              Created              Last Active
# a1b2c3d4-e5f6-7890-abcd-ef1234567890   2025-12-26 10:30    2025-12-26 14:22
# b2c3d4e5-f6a7-8901-bcde-f12345678901   2025-12-25 09:15    2025-12-25 18:45

# 恢复特定会话
claude --resume a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

---

## 模型与配置参数

### 指定模型

```bash
# 使用完整模型名
claude --model claude-opus-4-5-20250514

# 使用简写
claude --model opus
claude --model sonnet
claude --model haiku
```

### 其他配置参数

```bash
# 设置最大 token 数
claude --max-tokens 4096

# 设置系统提示
claude --system-prompt "你是一个 Python 专家"

# 允许所有权限（谨慎使用）
claude --dangerously-skip-permissions

# 指定工作目录
claude --cwd /path/to/project
```

---

## 帮助与信息命令

### 查看帮助

```bash
# 查看完整帮助
claude --help

# 输出：
# Usage: claude [options] [prompt]
#
# Options:
#   -p, --print <prompt>    Print response and exit
#   -c, --continue          Continue last session
#   --resume <session-id>   Resume specific session
#   --model <model>         Specify model to use
#   ...
```

### 查看版本

```bash
claude --version
# 输出: claude-code v1.x.x
```

### 运行诊断

```bash
# 诊断常见问题
claude /doctor

# 输出示例：
# ✓ Node.js version: 20.10.0
# ✓ API key configured
# ✓ Network connectivity
# ✓ Permissions configured
```

---

## 内置斜杠命令

在交互模式中，可以使用斜杠命令：

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助信息 |
| `/exit` | 退出会话 |
| `/clear` | 清除当前上下文 |
| `/compact` | 压缩上下文以节省 token |
| `/cost` | 显示当前会话的 token 使用量和成本 |
| `/doctor` | 运行诊断检查 |
| `/config` | 显示当前配置 |
| `/model` | 切换模型 |
| `/init` | 初始化项目的 CLAUDE.md 文件 |

### 使用示例

```
> /cost
╭─────────────────────────────────╮
│ Session Cost Summary            │
├─────────────────────────────────┤
│ Input tokens:  15,234           │
│ Output tokens: 8,456            │
│ Total cost:    $0.12            │
╰─────────────────────────────────╯

> /compact
Context compacted: 45,000 → 12,000 tokens

> /model opus
Model switched to claude-opus-4-5-20250514
```

---

## 实验练习

### 实验 1.2.1：交互模式基础

```bash
# 1. 启动会话
claude

# 2. 进行对话
> 你好，介绍一下你自己

# 3. 查看成本
> /cost

# 4. 清除上下文
> /clear

# 5. 退出
> /exit
```

### 实验 1.2.2：管道与单次提问

```bash
# 创建测试文件
echo 'function add(a, b) { return a + b; }' > test.js

# 分析代码
cat test.js | claude -p "为这个函数添加 TypeScript 类型注解"

# 分析 git 历史
git log --oneline -10 | claude -p "总结最近的开发活动"

# 生成代码
claude -p "写一个 Python 函数，检查字符串是否是回文" > palindrome.py
```

### 实验 1.2.3：会话管理

```bash
# 1. 开始一个会话并讨论某个主题
claude
> 让我们讨论 React Hooks 的最佳实践
> ...
> /exit

# 2. 继续该会话
claude -c
> 继续我们之前关于 React Hooks 的讨论

# 3. 查看所有会话
claude --list-sessions
```

### 实验 1.2.4：不同模型对比

创建一个测试脚本来比较不同模型：

```bash
#!/bin/bash
# compare_models.sh

PROMPT="用 Python 实现一个 LRU 缓存，包含 get 和 put 方法"

echo "=== Haiku 响应 ==="
time claude --model haiku -p "$PROMPT"

echo -e "\n=== Sonnet 响应 ==="
time claude --model sonnet -p "$PROMPT"

echo -e "\n=== Opus 响应 ==="
time claude --model opus -p "$PROMPT"
```

运行并观察：
- 响应速度差异
- 代码质量差异
- 解释详细程度差异

---

## 命令速查表

| 命令 | 说明 | 示例 |
|------|------|------|
| `claude` | 启动交互会话 | `claude` |
| `claude -p` | 单次提问 | `claude -p "问题"` |
| `claude -c` | 继续上次会话 | `claude -c` |
| `claude --resume` | 恢复特定会话 | `claude --resume <id>` |
| `claude --model` | 指定模型 | `claude --model opus` |
| `claude --help` | 查看帮助 | `claude --help` |
| `claude --version` | 查看版本 | `claude --version` |

---

## 下一节

[1.3 会话管理](./03-session-management.md) - 深入了解会话的高级管理功能
