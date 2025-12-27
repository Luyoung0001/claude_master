# 5.2 配置 MCP Server

本节详细介绍如何在 Claude Code 中配置 MCP Server。

---

## 配置位置

MCP Server 配置在 settings.json 的 `mcpServers` 字段：

```json
{
  "mcpServers": {
    "server-name": {
      "command": "command-to-run",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### 配置层级

1. **用户级别**：`~/.claude/settings.json`
2. **项目级别**：`.claude/settings.json`

项目级别会与用户级别合并（不是覆盖）。

---

## 配置结构

### 基本字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `command` | 是 | 启动 Server 的命令 |
| `args` | 否 | 命令行参数数组 |
| `env` | 否 | 环境变量对象 |
| `cwd` | 否 | 工作目录 |

### 完整示例

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allow"],
      "env": {}
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:pass@localhost/db"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

## 常见配置模式

### 使用 npx

最简单的方式，自动下载和运行：

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@package/server-name", ...其他参数]
    }
  }
}
```

### 使用本地安装

```bash
# 全局安装
npm install -g @modelcontextprotocol/server-filesystem
```

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": ["/allowed/path"]
    }
  }
}
```

### 使用 Python Server

```json
{
  "mcpServers": {
    "python-server": {
      "command": "python",
      "args": ["-m", "my_mcp_server"],
      "env": {
        "PYTHONPATH": "/path/to/modules"
      }
    }
  }
}
```

### 使用 Docker

```json
{
  "mcpServers": {
    "docker-server": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-v", "/host/path:/container/path",
        "my-mcp-server:latest"
      ]
    }
  }
}
```

---

## 环境变量

### 直接设置

```json
{
  "mcpServers": {
    "server": {
      "command": "...",
      "env": {
        "API_KEY": "sk-xxxx",
        "DEBUG": "true"
      }
    }
  }
}
```

### 引用系统环境变量

```json
{
  "mcpServers": {
    "server": {
      "command": "...",
      "env": {
        "API_KEY": "${MY_API_KEY}",
        "HOME": "${HOME}"
      }
    }
  }
}
```

### 使用 .env 文件

配合 dotenv：

```json
{
  "mcpServers": {
    "server": {
      "command": "npx",
      "args": ["-y", "dotenv", "-e", ".env", "--", "mcp-server"],
      "env": {}
    }
  }
}
```

---

## 多 Server 配置

### 同时使用多个 Server

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

### Server 之间的隔离

每个 Server：
- 独立的进程
- 独立的权限
- 独立的环境变量
- 独立的工具命名空间

---

## 验证配置

### 检查 Server 状态

```
# 在 Claude Code 中
> /mcp

# 输出：
# MCP Servers:
#   ✓ filesystem (3 tools)
#   ✓ github (5 tools)
#   ✗ postgres (connection failed)
```

### 列出可用工具

```
> 列出所有 MCP 工具

# Claude 会调用各 Server 获取工具列表
```

### 调试启动问题

```bash
# 手动运行 Server 查看输出
npx -y @modelcontextprotocol/server-filesystem /path

# 检查错误信息
```

---

## 实验练习

### 实验 5.2.1：配置文件系统 Server

```bash
# 创建测试目录
mkdir -p /tmp/mcp-test

# 配置 MCP Server
mkdir -p ~/.claude
cat > ~/.claude/settings.json << 'EOF'
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp/mcp-test"]
    }
  }
}
EOF

# 测试
claude
> 使用 MCP filesystem 工具列出 /tmp/mcp-test 目录的文件
> 使用 MCP filesystem 创建一个文件 test.txt
```

### 实验 5.2.2：配置多个 Server

```bash
cat > ~/.claude/settings.json << 'EOF'
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp/mcp-test"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
EOF

# 测试
claude
> 有哪些 MCP Server 可用？
> 在 memory server 中存储一些信息
```

### 实验 5.2.3：使用环境变量

```bash
# 设置环境变量
export TEST_API_KEY="my-secret-key"

# 配置
cat > ~/.claude/settings.json << 'EOF'
{
  "mcpServers": {
    "custom": {
      "command": "node",
      "args": ["-e", "console.error('API Key:', process.env.API_KEY)"],
      "env": {
        "API_KEY": "${TEST_API_KEY}"
      }
    }
  }
}
EOF

# 验证环境变量被正确传递
```

---

## 故障排除

### Server 启动失败

```bash
# 检查命令是否存在
which npx

# 手动运行 Server
npx -y @modelcontextprotocol/server-filesystem /path

# 检查权限
ls -la /path/to/server
```

### 工具调用失败

```bash
# 检查 Server 日志（如果有）
tail -f ~/.claude/logs/mcp-*.log

# 验证参数
# 确保传递的参数格式正确
```

### 环境变量问题

```bash
# 确认环境变量已设置
echo $MY_ENV_VAR

# 检查 shell 配置
grep MY_ENV_VAR ~/.bashrc ~/.zshrc

# 重启 Claude Code 使变量生效
```

---

## 最佳实践

### 1. 使用环境变量存储凭证

```json
{
  "mcpServers": {
    "api": {
      "command": "...",
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

### 2. 限制文件系统访问

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/specific/project/path"]
    }
  }
}
```

### 3. 项目特定配置

```
项目/
└── .claude/
    └── settings.json  # 只包含该项目需要的 Server
```

### 4. 版本锁定

```json
{
  "mcpServers": {
    "server": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem@1.0.0", "/path"]
    }
  }
}
```

---

## 下一节

[5.3 常用 MCP Server](./03-common-servers.md) - 学习如何使用各种常用的 MCP Server
