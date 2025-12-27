# 4.2 实用 Hook 示例

本节提供一系列实用的 Hook 配置示例，覆盖常见的自动化场景。

---

## 代码质量自动化

### 1. 自动代码格式化

写入 JS/TS 文件后自动运行 Prettier：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.(js|ts|jsx|tsx|json|css|scss|md)$"
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
          "toolName": "Edit",
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
```

### 2. 自动 ESLint 检查

编辑代码后自动检查 lint 错误：

```json
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
            "command": "npx eslint \"$FILE_PATH\" --fix 2>/dev/null || echo 'ESLint check completed'"
          }
        ]
      }
    ]
  }
}
```

### 3. TypeScript 类型检查

写入 TS 文件后运行类型检查：

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
            "command": "npx tsc --noEmit \"$FILE_PATH\" 2>&1 | head -20 || true"
          }
        ]
      }
    ]
  }
}
```

---

## 测试自动化

### 4. 修改代码后运行相关测试

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Edit",
          "inputMatches": "src/.*\\.(js|ts)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --findRelatedTests \"$FILE_PATH\" --passWithNoTests 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 5. 写入测试文件后立即运行

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.(test|spec)\\.(js|ts|jsx|tsx)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- \"$FILE_PATH\" 2>&1 | tail -30"
          }
        ]
      }
    ]
  }
}
```

---

## Git 自动化

### 6. 记录文件修改

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
            "command": "echo \"$(date '+%Y-%m-%d %H:%M:%S') - Modified: $FILE_PATH\" >> .claude/modification-log.txt"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Edit"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date '+%Y-%m-%d %H:%M:%S') - Edited: $FILE_PATH\" >> .claude/modification-log.txt"
          }
        ]
      }
    ]
  }
}
```

### 7. 自动添加到 Git 暂存区

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
            "command": "git add \"$FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 安全防护

### 8. 阻止修改敏感文件

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "(\\.env|\\.env\\..*|credentials|secrets|password)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'ERROR: Cannot write to sensitive files (.env, credentials, etc.)' && exit 1"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Edit",
          "inputMatches": "(\\.env|\\.env\\..*|credentials|secrets)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'ERROR: Cannot edit sensitive files' && exit 1"
          }
        ]
      }
    ]
  }
}
```

### 9. 阻止危险 Bash 命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputMatches": "(rm -rf /|rm -rf ~|:(){ :|:& };:|> /dev/sd)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'ERROR: Dangerous command blocked!' && exit 1"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Bash",
          "inputContains": "sudo"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'WARNING: sudo command detected. Proceed with caution.' >&2"
          }
        ]
      }
    ]
  }
}
```

### 10. 审计所有 Bash 命令

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
            "command": "echo \"[$(date '+%Y-%m-%d %H:%M:%S')] CMD: $TOOL_INPUT\" >> ~/.claude/bash-audit.log"
          }
        ]
      }
    ]
  }
}
```

---

## 通知与报告

### 11. 桌面通知（macOS）

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code 任务完成\" with title \"Claude Code\" sound name \"Glass\"'"
          }
        ]
      }
    ]
  }
}
```

### 12. 桌面通知（Linux）

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' '任务已完成' --icon=terminal"
          }
        ]
      }
    ]
  }
}
```

### 13. Slack 通知

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "curl -X POST -H 'Content-type: application/json' --data '{\"text\":\"Claude Code 任务完成\"}' $SLACK_WEBHOOK_URL 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 开发工作流

### 14. 写入后自动重启开发服务器

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "src/.*\\.(js|ts|jsx|tsx)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "touch .trigger-reload 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 15. 包安装后更新 lock 文件

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputMatches": "npm install|yarn add|pnpm add"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Dependencies updated. Lock file regenerated.'"
          }
        ]
      }
    ]
  }
}
```

---

## 组合配置示例

### 完整的开发环境 Hooks

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.env"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Blocked: Cannot modify .env files' && exit 1"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Bash"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date)] $TOOL_INPUT\" >> ~/.claude/cmd-history.log"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.(js|ts|jsx|tsx)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$FILE_PATH\" 2>/dev/null; npx eslint \"$FILE_PATH\" --fix 2>/dev/null || true"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "\\.test\\.(js|ts)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- \"$FILE_PATH\" --passWithNoTests 2>&1 | tail -10"
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
            "command": "osascript -e 'display notification \"Task completed\" with title \"Claude Code\"' 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

---

## 实验练习

### 实验 4.2.1：构建代码质量 Hook

```bash
# 安装依赖
npm install -D prettier eslint

# 创建配置
cat > .claude/settings.json << 'EOF'
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
            "command": "echo '>>> Running Prettier...' && npx prettier --write \"$FILE_PATH\""
          }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 创建一个名为 ugly.js 的文件，包含一个格式很乱的函数

# 检查文件是否被自动格式化
cat ugly.js
```

### 实验 4.2.2：安全防护 Hook

```bash
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputContains": ".env"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'BLOCKED: Sensitive file protection' && exit 1"
          }
        ]
      }
    ]
  }
}
EOF

# 测试
claude
> 创建一个 .env 文件，内容是 API_KEY=secret123

# 应该被阻止
```

---

## 下一节

[4.3 高级 Hook 技巧](./03-advanced-hooks.md) - 学习更复杂的 Hook 应用技巧
