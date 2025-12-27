# 8.4 安全与权限

本节介绍如何配置 Claude Code 的安全权限，保护敏感操作和数据。

---

## 权限系统概述

Claude Code 有多层安全机制：

1. **默认确认**：敏感操作需要用户确认
2. **权限配置**：预设允许/拒绝的操作
3. **Hooks 拦截**：自定义安全检查
4. **沙箱模式**：隔离执行环境

---

## 权限配置

### 配置位置

```json
// ~/.claude/settings.json 或 .claude/settings.json
{
  "permissions": {
    "allow": [...],
    "deny": [...]
  }
}
```

### 允许规则（allow）

```json
{
  "permissions": {
    "allow": [
      "Read",                    // 允许所有读取
      "Bash(npm:*)",            // 允许 npm 命令
      "Bash(git:*)",            // 允许 git 命令
      "Bash(ls:*)",             // 允许 ls 命令
      "Write(*.ts)",            // 允许写入 .ts 文件
      "Edit"                    // 允许编辑
    ]
  }
}
```

### 拒绝规则（deny）

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",         // 禁止递归删除
      "Bash(sudo:*)",           // 禁止 sudo
      "Write(.env*)",           // 禁止写入 .env 文件
      "Read(/etc/*)"            // 禁止读取系统配置
    ]
  }
}
```

### 完整示例

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Write(src/**/*)",
      "Write(tests/**/*)",
      "Edit",
      "Bash(npm:*)",
      "Bash(yarn:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(cat:*)",
      "Bash(ls:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(chmod:*)",
      "Bash(chown:*)",
      "Write(.env*)",
      "Write(*.pem)",
      "Write(*.key)",
      "Read(/etc/*)",
      "Read(~/.ssh/*)"
    ]
  }
}
```

---

## 敏感文件保护

### 使用 Hooks 保护

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "(\\.env|\\.pem|\\.key|credentials|secrets)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'BLOCKED: Cannot write to sensitive files' && exit 1"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Read",
          "inputMatches": "(\\.env|\\.pem|\\.key|id_rsa)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'BLOCKED: Cannot read sensitive files' && exit 1"
          }
        ]
      }
    ]
  }
}
```

### 创建 .claudeignore

类似 .gitignore，指定 Claude 不应访问的文件：

```
# .claudeignore

# 敏感文件
.env
.env.*
*.pem
*.key
credentials.json
secrets.yaml

# 私密目录
private/
secrets/
.ssh/

# 大型文件
*.zip
*.tar.gz
node_modules/
```

---

## 命令审计

### 记录所有 Bash 命令

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
            "command": "echo \"[$(date '+%Y-%m-%d %H:%M:%S')] $TOOL_INPUT\" >> ~/.claude/audit.log"
          }
        ]
      }
    ]
  }
}
```

### 查看审计日志

```bash
# 查看所有命令
cat ~/.claude/audit.log

# 搜索特定命令
grep "rm" ~/.claude/audit.log
grep "git push" ~/.claude/audit.log
```

---

## 危险操作防护

### 阻止危险 Bash 命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputMatches": "(rm -rf /|rm -rf ~|mkfs|dd if=|> /dev/sd|:(){ :|:& };:)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'CRITICAL: Dangerous command blocked!' && exit 1"
          }
        ]
      }
    ]
  }
}
```

### 警告但不阻止

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputContains": "sudo"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'WARNING: sudo command detected' >&2"
          }
        ]
      }
    ]
  }
}
```

---

## 网络安全

### 限制网络访问

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Bash",
          "inputMatches": "(curl|wget|nc|ncat)"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Network commands require review' >&2"
          }
        ]
      }
    ]
  }
}
```

### 限制 WebFetch 域名

使用 WebFetch 的 allowed_domains 或 blocked_domains：

```
> 从 example.com 获取数据

# Claude 可以配置只允许特定域名
```

---

## 沙箱模式

### 禁用沙箱（需谨慎）

```bash
# 默认情况下某些命令在沙箱中运行
# 可以禁用但要非常小心

claude --dangerously-skip-permissions
```

### 容器化隔离

```bash
# 使用 Docker 运行 Claude Code
docker run -it \
  -v $(pwd):/workspace \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  claude-code:latest \
  claude
```

---

## 实验练习

### 实验 8.4.1：配置基本权限

```bash
# 创建安全配置
cat > .claude/settings.json << 'EOF'
{
  "permissions": {
    "allow": [
      "Read",
      "Bash(ls:*)",
      "Bash(cat:*)"
    ],
    "deny": [
      "Write",
      "Bash(rm:*)"
    ]
  }
}
EOF

# 测试
claude
> 列出当前目录文件  # 应该成功
> 创建一个新文件 test.txt  # 应该被阻止
> 删除某个文件  # 应该被阻止
```

### 实验 8.4.2：敏感文件保护

```bash
# 创建保护规则
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": {
          "toolName": "Read",
          "inputContains": ".env"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Cannot read .env files' && exit 1"
          }
        ]
      }
    ]
  }
}
EOF

# 创建测试文件
echo "SECRET=password123" > .env

# 测试
claude
> 读取 .env 文件  # 应该被阻止
```

### 实验 8.4.3：命令审计

```bash
# 启用审计
cat > .claude/settings.json << 'EOF'
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
            "command": "echo \"[$(date)] $TOOL_INPUT\" >> /tmp/claude-audit.log"
          }
        ]
      }
    ]
  }
}
EOF

# 运行一些命令
claude
> 运行 ls
> 运行 git status
> 查看 npm 版本

# 查看审计日志
cat /tmp/claude-audit.log
```

---

## 安全检查清单

| 项目 | 检查 |
|------|------|
| API Key | 不要硬编码，使用环境变量 |
| .env 文件 | 配置为禁止读写 |
| SSH 密钥 | 禁止访问 ~/.ssh |
| 系统文件 | 禁止访问 /etc |
| 危险命令 | 阻止 rm -rf、sudo 等 |
| 审计日志 | 记录所有 Bash 命令 |

---

## 第八章总结

| 主题 | 要点 |
|------|------|
| Extended Thinking | 复杂问题分析，更多 Token |
| Vision | 图片分析，UI/截图/图表 |
| 性能优化 | Token 控制、模型选择、并行 |
| 安全权限 | allow/deny、Hooks、审计 |

---

## 下一章

[第九章：实战项目](../09-projects/README.md) - 综合应用所学知识完成实战项目
