# 5.3 常用 MCP Server

本节介绍一些最常用和实用的 MCP Server。

---

## 官方 Server

### 1. Filesystem Server

提供增强的文件系统访问能力。

**安装配置**：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allow"]
    }
  }
}
```

**可用工具**：

| 工具 | 说明 |
|------|------|
| `read_file` | 读取文件内容 |
| `write_file` | 写入文件 |
| `list_directory` | 列出目录内容 |
| `create_directory` | 创建目录 |
| `move_file` | 移动/重命名文件 |
| `search_files` | 搜索文件 |
| `get_file_info` | 获取文件元信息 |

**使用示例**：

```
> 使用 filesystem 工具搜索所有包含 "TODO" 的 .ts 文件

> 用 filesystem 工具读取 /project/src/main.ts 的内容
```

---

### 2. PostgreSQL Server

连接和操作 PostgreSQL 数据库。

**配置**：

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:password@localhost:5432/database"
      }
    }
  }
}
```

**可用工具**：

| 工具 | 说明 |
|------|------|
| `query` | 执行 SQL 查询 |
| `list_tables` | 列出所有表 |
| `describe_table` | 描述表结构 |

**使用示例**：

```
> 使用 postgres 工具列出所有数据库表

> 查询 users 表的前 10 条记录

> 描述 orders 表的结构
```

---

### 3. GitHub Server

与 GitHub API 交互。

**配置**：

```json
{
  "mcpServers": {
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

**获取 Token**：

1. 访问 https://github.com/settings/tokens
2. 创建 Personal Access Token
3. 选择需要的权限（repo, issues 等）
4. 设置环境变量：`export GITHUB_TOKEN=ghp_xxxx`

**可用工具**：

| 工具 | 说明 |
|------|------|
| `search_repositories` | 搜索仓库 |
| `get_repository` | 获取仓库信息 |
| `list_issues` | 列出 issues |
| `create_issue` | 创建 issue |
| `create_pull_request` | 创建 PR |
| `get_file_contents` | 获取文件内容 |

**使用示例**：

```
> 使用 GitHub 工具搜索 "react hooks" 相关的热门仓库

> 获取 facebook/react 仓库的信息

> 列出我的仓库中的 open issues
```

---

### 4. Slack Server

与 Slack 工作区交互。

**配置**：

```json
{
  "mcpServers": {
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

**可用工具**：

| 工具 | 说明 |
|------|------|
| `send_message` | 发送消息 |
| `list_channels` | 列出频道 |
| `get_channel_history` | 获取频道历史 |

---

### 5. Memory Server

提供会话级别的键值存储。

**配置**：

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**可用工具**：

| 工具 | 说明 |
|------|------|
| `store` | 存储数据 |
| `retrieve` | 获取数据 |
| `list_keys` | 列出所有键 |
| `delete` | 删除数据 |

**使用示例**：

```
> 在 memory 中存储当前项目的分析结果

> 从 memory 中获取之前存储的分析结果
```

---

## 第三方 Server

### 6. Puppeteer Server

浏览器自动化和网页抓取。

**配置**：

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-puppeteer"]
    }
  }
}
```

**可用工具**：

| 工具 | 说明 |
|------|------|
| `navigate` | 导航到 URL |
| `screenshot` | 截取屏幕 |
| `click` | 点击元素 |
| `type` | 输入文本 |
| `evaluate` | 执行 JavaScript |

**使用示例**：

```
> 使用 puppeteer 打开 https://example.com 并截图

> 在页面上找到登录表单并填写信息
```

---

### 7. SQLite Server

轻量级 SQLite 数据库操作。

**配置**：

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-sqlite", "/path/to/database.db"]
    }
  }
}
```

**可用工具**：

| 工具 | 说明 |
|------|------|
| `query` | 执行 SQL |
| `list_tables` | 列出表 |
| `describe_table` | 描述表结构 |

---

### 8. Brave Search Server

使用 Brave 搜索引擎。

**配置**：

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

---

## 完整配置示例

### 开发环境配置

```json
{
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
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://localhost/devdb"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

---

## 实验练习

### 实验 5.3.1：使用 Filesystem Server

```bash
# 配置
cat > ~/.claude/settings.json << 'EOF'
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp/mcp-fs-test"]
    }
  }
}
EOF

# 创建测试目录和文件
mkdir -p /tmp/mcp-fs-test
echo "Hello MCP" > /tmp/mcp-fs-test/hello.txt
mkdir /tmp/mcp-fs-test/subdir
echo "Secret" > /tmp/mcp-fs-test/subdir/secret.txt

# 测试
claude
> 使用 MCP filesystem 列出 /tmp/mcp-fs-test 的内容
> 搜索包含 "Hello" 的文件
> 读取 hello.txt 的内容
```

### 实验 5.3.2：使用 Memory Server

```bash
# 添加 memory server 配置
cat > ~/.claude/settings.json << 'EOF'
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
EOF

# 测试
claude
> 在 memory 中存储一个笔记，key 是 "project-idea"，内容是 "Build an AI assistant"
> 列出 memory 中所有的键
> 获取 "project-idea" 的内容
```

### 实验 5.3.3：使用 GitHub Server（需要 Token）

```bash
# 设置 GitHub Token
export GITHUB_TOKEN="your-token-here"

# 配置
cat > ~/.claude/settings.json << 'EOF'
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
EOF

# 测试
claude
> 使用 GitHub 工具搜索 "mcp server" 相关的仓库
> 获取某个仓库的 README 文件
```

---

## Server 选择指南

| 需求 | 推荐 Server |
|------|-------------|
| 增强文件操作 | filesystem |
| 数据库查询 | postgres / sqlite |
| GitHub 集成 | github |
| 团队协作 | slack |
| 网页抓取 | puppeteer |
| 临时存储 | memory |
| 网络搜索 | brave-search |

---

## 下一节

[5.4 自定义 MCP Server 开发](./04-custom-server.md) - 学习如何开发自己的 MCP Server
