# 1.3 会话管理

Claude Code 的会话管理系统允许你保存、恢复和管理多个对话上下文。本节将深入介绍会话管理的高级功能。

---

## 会话基础概念

### 什么是会话？

会话（Session）是 Claude Code 中的一个完整对话上下文，包括：
- 对话历史（所有消息）
- 工具调用记录
- 文件读取缓存
- 当前工作目录状态

### 会话存储位置

```bash
# 会话存储在用户目录下
~/.claude/sessions/

# 每个会话是一个独立的 JSON 文件
~/.claude/sessions/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json
```

---

## 会话列表与查看

### 查看所有会话

```bash
claude --list-sessions

# 输出示例：
# ╭──────────────────────────────────────────────────────────────────╮
# │ Sessions                                                         │
# ├──────────────────────────────────────────────────────────────────┤
# │ ID                                    Created      Last Active   │
# │ a1b2c3d4-e5f6-7890-abcd-ef1234567890 Dec 26 10:30 Dec 26 14:22  │
# │ b2c3d4e5-f6a7-8901-bcde-f12345678901 Dec 25 09:15 Dec 25 18:45  │
# │ c3d4e5f6-a7b8-9012-cdef-123456789012 Dec 24 11:00 Dec 24 16:30  │
# ╰──────────────────────────────────────────────────────────────────╯
```

### 按项目过滤会话

```bash
# 只显示当前目录相关的会话
claude --list-sessions --cwd .

# 显示特定项目的会话
claude --list-sessions --cwd /path/to/project
```

---

## 会话恢复策略

### 继续最近会话

```bash
# 最简单的方式：继续上次会话
claude -c
# 或
claude --continue
```

### 恢复特定会话

```bash
# 使用会话 ID 恢复
claude --resume a1b2c3d4-e5f6-7890-abcd-ef1234567890

# 支持部分 ID 匹配（前缀匹配）
claude --resume a1b2c3d4
```

### 带上下文恢复

```bash
# 恢复会话并添加新的上下文
claude --resume a1b2c3d4 -p "继续之前的工作，现在请..."
```

---

## 上下文自动摘要机制

Claude Code 使用智能摘要机制来管理长对话的上下文。

### 工作原理

1. **Token 计数**：持续监控上下文使用量
2. **阈值触发**：当接近上下文窗口限制时自动触发
3. **智能摘要**：保留关键信息，压缩冗余内容
4. **无缝继续**：用户无感知地继续对话

### 手动触发摘要

```
# 在交互模式中
> /compact

# 输出：
# Compacting context...
# Before: 128,000 tokens
# After:  32,000 tokens
# Preserved: Key decisions, file changes, current task context
```

### 查看上下文使用情况

```
> /cost

# 输出包含上下文信息：
# ╭─────────────────────────────────╮
# │ Context Usage                   │
# ├─────────────────────────────────┤
# │ Current tokens:  45,234         │
# │ Max tokens:      200,000        │
# │ Usage:           22.6%          │
# ╰─────────────────────────────────╯
```

---

## 多会话并行管理

### 场景：同时处理多个项目

```bash
# 终端 1：处理前端项目
cd ~/projects/frontend
claude
> 帮我优化 React 组件性能

# 终端 2：处理后端项目
cd ~/projects/backend
claude
> 帮我设计 API 接口

# 两个会话完全独立，互不影响
```

### 会话命名建议

虽然 Claude Code 使用 UUID 作为会话 ID，但你可以在对话开始时声明会话目的：

```
> 这是一个关于"用户认证系统重构"的会话，请记住这个上下文

# 之后恢复时：
claude -c
> 继续我们的用户认证系统重构讨论
```

---

## 会话清理与管理

### 清除当前上下文

```
# 在交互模式中清除上下文（保留会话）
> /clear

# 确认提示
# Are you sure you want to clear the context? (y/n)
```

### 删除会话文件

```bash
# 查看会话文件
ls ~/.claude/sessions/

# 删除特定会话
rm ~/.claude/sessions/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json

# 删除所有会话（谨慎！）
rm -rf ~/.claude/sessions/*
```

### 导出会话

```bash
# 导出会话为可读格式
claude --export-session a1b2c3d4 > session_backup.json

# 或者直接复制会话文件
cp ~/.claude/sessions/a1b2c3d4*.json ./backup/
```

---

## 实验练习

### 实验 1.3.1：会话生命周期

```bash
# 1. 创建新会话
claude
> 我们来讨论 Docker 容器化最佳实践
> Docker 和 Kubernetes 有什么区别？
> /cost
> /exit

# 2. 查看会话列表
claude --list-sessions

# 3. 继续会话
claude -c
> 继续讨论，如何在生产环境中使用 Docker？
> /exit

# 4. 开始新会话（不继续）
claude
> 这是一个新话题：GraphQL vs REST
> /exit

# 5. 恢复第一个会话
claude --list-sessions
# 找到第一个会话的 ID
claude --resume <first-session-id>
> 回到 Docker 话题
```

### 实验 1.3.2：上下文压缩

```bash
# 1. 创建一个长对话
claude
> 请详细解释 JavaScript 的事件循环机制
> 什么是微任务和宏任务？
> Promise 和 async/await 如何工作？
> 请给出一些复杂的示例代码
> 解释 Node.js 的事件循环与浏览器的区别

# 2. 查看上下文使用
> /cost

# 3. 压缩上下文
> /compact

# 4. 再次查看
> /cost

# 5. 验证关键信息保留
> 总结我们之前讨论的要点
```

### 实验 1.3.3：多项目会话管理

```bash
# 创建两个测试项目
mkdir -p ~/test-projects/{project-a,project-b}

# 在 project-a 中启动会话
cd ~/test-projects/project-a
claude
> 初始化：这是项目 A 的开发会话
> /exit

# 在 project-b 中启动会话
cd ~/test-projects/project-b
claude
> 初始化：这是项目 B 的开发会话
> /exit

# 查看两个项目的会话
claude --list-sessions --cwd ~/test-projects/project-a
claude --list-sessions --cwd ~/test-projects/project-b
```

---

## 会话管理最佳实践

### 1. 任务驱动的会话

```
# 好的做法：明确会话目标
> 会话目标：重构用户认证模块，从 JWT 迁移到 OAuth2

# 避免：一个会话处理多个不相关任务
```

### 2. 定期检查上下文

```
# 养成习惯：在复杂任务中定期检查
> /cost
# 当使用率超过 70% 时考虑压缩
> /compact
```

### 3. 会话归档

```bash
# 为重要会话创建备份
mkdir -p ~/claude-session-archives

# 复制重要会话
cp ~/.claude/sessions/important-session-id.json \
   ~/claude-session-archives/auth-refactor-$(date +%Y%m%d).json
```

### 4. 清理过期会话

```bash
# 删除 30 天前的会话
find ~/.claude/sessions -type f -mtime +30 -delete
```

---

## 故障排除

### Q: 会话恢复后上下文丢失？

可能原因：
1. 上下文已被自动压缩
2. 会话文件损坏

解决方案：
```bash
# 检查会话文件完整性
cat ~/.claude/sessions/<session-id>.json | jq .

# 如果文件损坏，需要开始新会话
```

### Q: 会话列表为空？

```bash
# 检查会话目录
ls -la ~/.claude/sessions/

# 检查权限
chmod 755 ~/.claude/sessions/
```

### Q: 如何在不同机器间同步会话？

```bash
# 方法1：使用云同步（Dropbox/iCloud）
ln -s ~/Dropbox/.claude ~/.claude

# 方法2：手动复制
scp -r ~/.claude/sessions/ user@remote:~/.claude/sessions/
```

---

## 总结

| 操作 | 命令 |
|------|------|
| 继续最近会话 | `claude -c` |
| 恢复特定会话 | `claude --resume <id>` |
| 列出会话 | `claude --list-sessions` |
| 压缩上下文 | `/compact` |
| 查看使用量 | `/cost` |
| 清除上下文 | `/clear` |

---

## 下一章

[第二章：核心工具使用](../02-core-tools/README.md) - 掌握 Claude Code 的强大工具集
