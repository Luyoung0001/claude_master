# 4.3 高级 Hook 技巧

本节介绍更复杂的 Hook 应用场景和技巧。

---

## 条件执行

### 基于环境的条件执行

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.ts$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "if [ \"$NODE_ENV\" = 'development' ]; then npx prettier --write \"$FILE_PATH\"; fi"
          }
        ]
      }
    ]
  }
}
```

### 基于文件内容的条件执行

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputContains": "git push"
        },
        "hooks": [
          {
            "type": "command",
            "command": "if grep -q 'TODO\\|FIXME\\|console.log' src/**/*.ts 2>/dev/null; then echo 'Warning: Found TODOs or console.logs' >&2; fi"
          }
        ]
      }
    ]
  }
}
```

---

## 外部脚本集成

### 使用外部脚本

将复杂逻辑放入脚本文件：

```bash
# .claude/hooks/pre-write-check.sh
#!/bin/bash

FILE_PATH="$1"
CONTENT="$2"

# 检查是否包含敏感信息
if echo "$CONTENT" | grep -qE "(password|secret|api_key|token)\s*[:=]"; then
    echo "ERROR: Potential sensitive data detected in file content"
    exit 1
fi

# 检查文件路径
if [[ "$FILE_PATH" == *".env"* ]] || [[ "$FILE_PATH" == *"credentials"* ]]; then
    echo "ERROR: Cannot write to sensitive file paths"
    exit 1
fi

exit 0
```

配置：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Write"
        },
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/pre-write-check.sh \"$FILE_PATH\" \"$TOOL_INPUT\""
          }
        ]
      }
    ]
  }
}
```

### Python 脚本集成

```python
#!/usr/bin/env python3
# .claude/hooks/analyze-code.py

import sys
import json
import subprocess

def main():
    file_path = sys.argv[1]

    # 运行代码分析
    result = subprocess.run(
        ['npx', 'eslint', file_path, '--format', 'json'],
        capture_output=True,
        text=True
    )

    if result.returncode != 0:
        try:
            issues = json.loads(result.stdout)
            error_count = sum(len(f.get('messages', [])) for f in issues)
            print(f"Found {error_count} issues in {file_path}")
        except:
            pass

    # 总是返回 0，不阻止执行
    sys.exit(0)

if __name__ == '__main__':
    main()
```

配置：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.(js|ts)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/hooks/analyze-code.py \"$FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

---

## 多 Hook 链式执行

### 有序执行多个操作

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.tsx?$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo '1. Formatting...' && npx prettier --write \"$FILE_PATH\""
          },
          {
            "type": "command",
            "command": "echo '2. Linting...' && npx eslint \"$FILE_PATH\" --fix"
          },
          {
            "type": "command",
            "command": "echo '3. Type checking...' && npx tsc --noEmit 2>&1 | head -5"
          },
          {
            "type": "command",
            "command": "echo '4. Done!'"
          }
        ]
      }
    ]
  }
}
```

### 失败时继续

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\" 2>/dev/null || true"
          },
          {
            "type": "command",
            "command": "npx eslint \"$FILE_PATH\" --fix 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 状态管理

### 使用临时文件保存状态

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
            "command": "echo \"$TOOL_INPUT\" >> /tmp/claude-session-commands.txt"
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
            "command": "if [ -f /tmp/claude-session-commands.txt ]; then echo '=== Session Commands ===' && cat /tmp/claude-session-commands.txt && rm /tmp/claude-session-commands.txt; fi"
          }
        ]
      }
    ]
  }
}
```

### 计数器

```bash
# .claude/hooks/counter.sh
#!/bin/bash

COUNTER_FILE="/tmp/claude-edit-counter"

case "$1" in
    "increment")
        count=$(cat "$COUNTER_FILE" 2>/dev/null || echo 0)
        echo $((count + 1)) > "$COUNTER_FILE"
        ;;
    "get")
        cat "$COUNTER_FILE" 2>/dev/null || echo 0
        ;;
    "reset")
        echo 0 > "$COUNTER_FILE"
        ;;
esac
```

配置：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Edit"
        },
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/counter.sh increment && echo \"Edits in this session: $(bash .claude/hooks/counter.sh get)\""
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
            "command": "echo \"Total edits: $(bash .claude/hooks/counter.sh get)\" && bash .claude/hooks/counter.sh reset"
          }
        ]
      }
    ]
  }
}
```

---

## 与其他工具集成

### 与 Docker 集成

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputContains": "Dockerfile"
        },
        "hooks": [
          {
            "type": "command",
            "command": "docker build -t test-build . --quiet && echo 'Docker build successful' || echo 'Docker build failed'"
          }
        ]
      }
    ]
  }
}
```

### 与 Git Hooks 集成

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputContains": "git commit"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test 2>&1 | tail -5 || echo 'Tests must pass before commit'"
          }
        ]
      }
    ]
  }
}
```

### 与 CI/CD 集成

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "if git diff --cached --quiet; then echo 'No staged changes'; else echo 'Changes ready for commit'; fi"
          }
        ]
      }
    ]
  }
}
```

---

## 性能优化

### 异步执行

对于耗时操作，使用后台执行：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write"
        },
        "hooks": [
          {
            "type": "command",
            "command": "(npx prettier --write \"$FILE_PATH\" &) 2>/dev/null"
          }
        ]
      }
    ]
  }
}
```

### 缓存检查结果

```bash
# .claude/hooks/cached-lint.sh
#!/bin/bash

FILE_PATH="$1"
CACHE_DIR="/tmp/claude-lint-cache"
mkdir -p "$CACHE_DIR"

FILE_HASH=$(md5sum "$FILE_PATH" | cut -d' ' -f1)
CACHE_FILE="$CACHE_DIR/$FILE_HASH"

if [ -f "$CACHE_FILE" ]; then
    echo "Using cached lint result"
    cat "$CACHE_FILE"
else
    npx eslint "$FILE_PATH" --format compact 2>&1 | tee "$CACHE_FILE"
fi
```

### 限制执行频率

```bash
# .claude/hooks/throttle.sh
#!/bin/bash

LOCK_FILE="/tmp/claude-hook-throttle"
THROTTLE_SECONDS=5

if [ -f "$LOCK_FILE" ]; then
    LAST_RUN=$(cat "$LOCK_FILE")
    NOW=$(date +%s)
    DIFF=$((NOW - LAST_RUN))

    if [ $DIFF -lt $THROTTLE_SECONDS ]; then
        echo "Throttled (wait $((THROTTLE_SECONDS - DIFF))s)"
        exit 0
    fi
fi

date +%s > "$LOCK_FILE"
exec "$@"
```

---

## 调试与故障排除

### 详细日志

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%H:%M:%S')] PRE  $TOOL_NAME: $TOOL_INPUT\" >> /tmp/claude-hooks-debug.log"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%H:%M:%S')] POST $TOOL_NAME: completed\" >> /tmp/claude-hooks-debug.log"
          }
        ]
      }
    ]
  }
}
```

监控日志：

```bash
tail -f /tmp/claude-hooks-debug.log
```

### 测试 Hook

```bash
# 创建测试脚本
cat > test-hook.sh << 'EOF'
#!/bin/bash

export TOOL_NAME="Write"
export TOOL_INPUT='{"file_path": "/tmp/test.ts"}'
export FILE_PATH="/tmp/test.ts"

# 测试你的 hook 命令
echo "Testing hook with:"
echo "  TOOL_NAME: $TOOL_NAME"
echo "  FILE_PATH: $FILE_PATH"

# 运行 hook 命令
your_hook_command_here
EOF

chmod +x test-hook.sh
./test-hook.sh
```

---

## 实验练习

### 实验 4.3.1：创建多阶段 Hook

```bash
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.ts$"
        },
        "hooks": [
          { "type": "command", "command": "echo '=== Step 1: Format ===' && npx prettier --write \"$FILE_PATH\" 2>/dev/null || true" },
          { "type": "command", "command": "echo '=== Step 2: Lint ===' && npx eslint \"$FILE_PATH\" 2>&1 | head -10 || true" },
          { "type": "command", "command": "echo '=== Step 3: Complete ==='" }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 创建一个有一些格式问题的 TypeScript 文件

# 观察三个步骤的执行
```

### 实验 4.3.2：会话统计 Hook

```bash
# 创建统计脚本
mkdir -p .claude/hooks
cat > .claude/hooks/stats.sh << 'EOF'
#!/bin/bash

STATS_FILE="/tmp/claude-session-stats.json"

case "$1" in
    "init")
        echo '{"reads":0,"writes":0,"edits":0,"bashes":0}' > "$STATS_FILE"
        ;;
    "increment")
        TOOL="$2"
        if [ -f "$STATS_FILE" ]; then
            case "$TOOL" in
                "Read") jq '.reads += 1' "$STATS_FILE" > /tmp/tmp.json && mv /tmp/tmp.json "$STATS_FILE" ;;
                "Write") jq '.writes += 1' "$STATS_FILE" > /tmp/tmp.json && mv /tmp/tmp.json "$STATS_FILE" ;;
                "Edit") jq '.edits += 1' "$STATS_FILE" > /tmp/tmp.json && mv /tmp/tmp.json "$STATS_FILE" ;;
                "Bash") jq '.bashes += 1' "$STATS_FILE" > /tmp/tmp.json && mv /tmp/tmp.json "$STATS_FILE" ;;
            esac
        fi
        ;;
    "show")
        if [ -f "$STATS_FILE" ]; then
            echo "=== Session Statistics ==="
            cat "$STATS_FILE" | jq .
        fi
        ;;
esac
EOF

chmod +x .claude/hooks/stats.sh

# 配置
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {},
        "hooks": [
          { "type": "command", "command": "bash .claude/hooks/stats.sh increment $TOOL_NAME 2>/dev/null || true" }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          { "type": "command", "command": "bash .claude/hooks/stats.sh show" }
        ]
      }
    ]
  }
}
EOF

# 初始化并测试
bash .claude/hooks/stats.sh init

claude
> 读取 package.json
> 创建一个新文件 test.js
> 修改 test.js 添加一行代码
> 运行 ls
> /exit

# 查看统计
```

---

## 第四章总结

| 主题 | 要点 |
|------|------|
| Hook 类型 | PreToolUse, PostToolUse, Notification, Stop |
| 匹配器 | toolName, inputContains, inputMatches |
| 阻止执行 | exit 1 在 PreToolUse 中阻止 |
| 环境变量 | $TOOL_NAME, $TOOL_INPUT, $FILE_PATH |
| 最佳实践 | 外部脚本、错误处理、异步执行 |

---

## 下一章

[第五章：MCP Server 集成](../05-mcp-server/README.md) - 学习使用 Model Context Protocol 扩展能力
