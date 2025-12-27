# 4.1 Hook 类型与配置

本节详细介绍 Claude Code 的四种 Hook 类型及其配置方法。

---

## Hook 类型

### 1. PreToolUse - 工具执行前

在 Claude 调用工具之前触发，可以：
- 验证工具参数
- 准备运行环境
- 拦截并阻止执行
- 修改工具输入

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": { "toolName": "Bash" },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'About to execute: $TOOL_INPUT'"
          }
        ]
      }
    ]
  }
}
```

### 2. PostToolUse - 工具执行后

在工具执行完成后触发，可以：
- 处理工具输出
- 运行后续任务
- 发送通知
- 记录日志

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "toolName": "Write" },
        "hooks": [
          {
            "type": "command",
            "command": "npm run format -- $TOOL_INPUT"
          }
        ]
      }
    ]
  }
}
```

### 3. Notification - 通知事件

当 Claude 发送通知时触发：

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' '$MESSAGE'"
          }
        ]
      }
    ]
  }
}
```

### 4. Stop - 任务停止

当 Claude 完成或停止任务时触发：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Task completed' >> ~/.claude/task-log.txt"
          }
        ]
      }
    ]
  }
}
```

---

## 配置结构

### settings.json 中的 Hooks 配置

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": { /* 匹配条件 */ },
        "hooks": [ /* Hook 动作列表 */ ]
      }
    ],
    "PostToolUse": [ /* ... */ ],
    "Notification": [ /* ... */ ],
    "Stop": [ /* ... */ ]
  }
}
```

### 完整配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo '[PRE] Bash command: $TOOL_INPUT' >> /tmp/claude-log.txt"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Write",
          "inputContains": ".env"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Blocked: Cannot write to .env files' && exit 1"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputContains": ".ts"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

---

## 匹配器（Matcher）

匹配器决定 Hook 何时触发。

### 可用的匹配条件

| 条件 | 说明 | 示例 |
|------|------|------|
| `toolName` | 工具名称 | `"Bash"`, `"Write"`, `"Edit"` |
| `inputContains` | 输入包含字符串 | `".ts"`, `"rm -rf"` |
| `inputMatches` | 输入匹配正则 | `"\\.test\\.ts$"` |

### 匹配器示例

#### 匹配特定工具

```json
{
  "matcher": {
    "toolName": "Bash"
  }
}
```

#### 匹配工具和输入

```json
{
  "matcher": {
    "toolName": "Write",
    "inputContains": ".tsx"
  }
}
```

#### 使用正则表达式

```json
{
  "matcher": {
    "toolName": "Bash",
    "inputMatches": "^(npm|yarn|pnpm) (install|add)"
  }
}
```

#### 匹配所有（空匹配器）

```json
{
  "matcher": {}
}
```

---

## Hook 动作

### command 类型

执行 shell 命令：

```json
{
  "type": "command",
  "command": "echo 'Hello from hook'"
}
```

### 可用的环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `$TOOL_NAME` | 工具名称 | `Bash`, `Write` |
| `$TOOL_INPUT` | 工具输入（JSON） | `{"command": "ls"}` |
| `$FILE_PATH` | 文件路径（文件工具） | `/path/to/file.ts` |
| `$MESSAGE` | 通知消息（Notification） | `"Task completed"` |

### 阻止工具执行

在 PreToolUse 中，如果命令返回非零退出码，工具执行将被阻止：

```json
{
  "type": "command",
  "command": "echo 'Blocked!' && exit 1"
}
```

---

## 配置位置

### 项目级别

在项目根目录的 `.claude/settings.json`：

```bash
# 创建配置
mkdir -p .claude
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "toolName": "Write" },
        "hooks": [
          { "type": "command", "command": "echo 'File written'" }
        ]
      }
    ]
  }
}
EOF
```

### 用户级别

在 `~/.claude/settings.json`：

```bash
# 编辑用户配置
vim ~/.claude/settings.json
```

### 优先级

1. 项目级别配置优先
2. 用户级别配置作为默认

---

## 实验练习

### 实验 4.1.1：基本日志 Hook

```bash
# 创建项目配置
mkdir -p .claude
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date)] Tool: $TOOL_NAME\" >> /tmp/claude-hooks.log"
          }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 列出当前目录的文件

# 查看日志
cat /tmp/claude-hooks.log
```

### 实验 4.1.2：阻止危险命令

```bash
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputContains": "rm -rf"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'ERROR: rm -rf is not allowed' && exit 1"
          }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 删除 /tmp 目录下的所有文件

# Claude 的 rm -rf 命令应该被阻止
```

### 实验 4.1.3：文件写入后格式化

```bash
# 确保有 prettier
npm install -g prettier

cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.(js|ts|jsx|tsx)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 创建一个新的 TypeScript 文件 test.ts，包含一个简单的函数

# 查看文件是否被格式化
cat test.ts
```

### 实验 4.1.4：通知 Hook

```bash
# macOS 通知
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code task completed\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 帮我创建一个简单的 README.md 文件
> /exit

# 应该看到系统通知
```

---

## 调试 Hooks

### 查看 Hook 是否触发

```bash
# 添加调试日志
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[DEBUG] Tool=$TOOL_NAME Input=$TOOL_INPUT\" >> /tmp/hook-debug.log"
          }
        ]
      }
    ]
  }
}

# 监控日志
tail -f /tmp/hook-debug.log
```

### 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| Hook 不触发 | 匹配器不正确 | 检查 toolName 拼写 |
| 命令失败 | 环境变量未展开 | 使用双引号包裹变量 |
| 意外阻止 | exit 1 触发 | 检查命令逻辑 |

---

## 下一节

[4.2 实用 Hook 示例](./02-hook-examples.md) - 学习更多实用的 Hook 配置示例
