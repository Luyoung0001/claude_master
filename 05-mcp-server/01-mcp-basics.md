# 5.1 MCP 协议基础

本节介绍 Model Context Protocol（MCP）的核心概念和工作原理。

---

## 什么是 MCP？

MCP（Model Context Protocol）是 Anthropic 推出的开放协议，用于：

- 让 AI 模型与外部工具和数据源交互
- 提供标准化的工具定义和调用方式
- 支持双向通信和资源共享

### 核心价值

1. **标准化**：统一的工具接口定义
2. **可扩展**：轻松添加新能力
3. **安全**：明确的权限边界
4. **复用**：社区共享的 Server 生态

---

## 核心概念

### 1. Resources（资源）

资源是 MCP Server 可以提供的数据：

```typescript
// 资源示例
{
  uri: "file:///path/to/document.md",
  name: "Project README",
  mimeType: "text/markdown",
  description: "Main project documentation"
}
```

资源类型：
- 文件内容
- 数据库记录
- API 响应
- 实时数据流

### 2. Tools（工具）

工具是 MCP Server 提供的可执行操作：

```typescript
// 工具定义示例
{
  name: "query_database",
  description: "Execute SQL query on the database",
  inputSchema: {
    type: "object",
    properties: {
      query: {
        type: "string",
        description: "SQL query to execute"
      }
    },
    required: ["query"]
  }
}
```

### 3. Prompts（提示模板）

预定义的提示模板，可以带参数：

```typescript
// 提示模板示例
{
  name: "analyze_data",
  description: "Analyze data from a specific table",
  arguments: [
    {
      name: "table_name",
      description: "Name of the table to analyze",
      required: true
    }
  ]
}
```

---

## 通信机制

### 传输层

MCP 支持两种传输方式：

#### 1. stdio（标准输入/输出）

最常用的方式，Server 作为子进程运行：

```
Claude Code → stdin → MCP Server
Claude Code ← stdout ← MCP Server
```

优点：
- 简单可靠
- 进程隔离
- 易于调试

#### 2. HTTP/SSE

Server 作为独立服务运行：

```
Claude Code ⟷ HTTP/SSE ⟷ MCP Server (port 8080)
```

优点：
- 支持远程 Server
- 可以服务多个客户端
- 适合长期运行的服务

### 消息格式

MCP 使用 JSON-RPC 2.0 格式：

```json
// 请求
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "query_database",
    "arguments": {
      "query": "SELECT * FROM users LIMIT 10"
    }
  }
}

// 响应
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "[{\"id\": 1, \"name\": \"Alice\"}, ...]"
      }
    ]
  }
}
```

---

## MCP 生命周期

### 1. 初始化

```
1. Claude Code 启动 MCP Server 进程
2. 发送 initialize 请求
3. Server 返回能力列表
4. 发送 initialized 通知
```

### 2. 运行时

```
1. Claude 决定使用 MCP 工具
2. Claude Code 发送 tools/call 请求
3. Server 执行操作
4. Server 返回结果
5. Claude 使用结果继续对话
```

### 3. 关闭

```
1. Claude Code 发送 shutdown 请求
2. Server 清理资源
3. 进程退出
```

---

## Server 能力声明

Server 在初始化时声明其能力：

```json
{
  "capabilities": {
    "resources": {
      "subscribe": true,
      "listChanged": true
    },
    "tools": {},
    "prompts": {}
  }
}
```

| 能力 | 说明 |
|------|------|
| `resources` | 提供资源访问 |
| `resources.subscribe` | 支持资源变更订阅 |
| `tools` | 提供可调用的工具 |
| `prompts` | 提供提示模板 |

---

## 与 Claude Code 的集成

### Claude Code 如何使用 MCP

1. **配置 Server**：在 settings.json 中定义
2. **自动发现**：Claude Code 启动时连接所有配置的 Server
3. **工具合并**：MCP 工具与内置工具一起可用
4. **透明调用**：Claude 自动选择合适的工具

### 工具命名

MCP 工具使用前缀命名避免冲突：

```
mcp__<server_name>__<tool_name>

例如：
mcp__filesystem__read_file
mcp__database__query
mcp__github__create_issue
```

---

## 安全考虑

### 权限控制

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-filesystem",
      "args": ["--root", "/safe/directory"]
    }
  }
}
```

### 最小权限原则

- 只配置需要的 Server
- 限制 Server 的访问范围
- 使用只读模式（如果可能）

### 敏感数据

- 使用环境变量传递凭证
- 不在配置中明文存储密码
- 注意 Server 的数据处理方式

---

## 实验练习

### 实验 5.1.1：理解 MCP 消息

```bash
# 创建一个简单的 MCP 消息记录器
cat > /tmp/mcp-logger.js << 'EOF'
const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stderr
});

// 记录所有接收到的消息
rl.on('line', (line) => {
  console.error('[RECEIVED]', line);

  try {
    const msg = JSON.parse(line);

    if (msg.method === 'initialize') {
      // 响应初始化
      const response = {
        jsonrpc: "2.0",
        id: msg.id,
        result: {
          protocolVersion: "2024-11-05",
          capabilities: {
            tools: {}
          },
          serverInfo: {
            name: "logger",
            version: "1.0.0"
          }
        }
      };
      console.log(JSON.stringify(response));
    }

    if (msg.method === 'tools/list') {
      const response = {
        jsonrpc: "2.0",
        id: msg.id,
        result: {
          tools: [{
            name: "echo",
            description: "Echo back a message",
            inputSchema: {
              type: "object",
              properties: {
                message: { type: "string" }
              },
              required: ["message"]
            }
          }]
        }
      };
      console.log(JSON.stringify(response));
    }

    if (msg.method === 'tools/call') {
      const response = {
        jsonrpc: "2.0",
        id: msg.id,
        result: {
          content: [{
            type: "text",
            text: `Echo: ${msg.params.arguments.message}`
          }]
        }
      };
      console.log(JSON.stringify(response));
    }
  } catch (e) {
    console.error('[ERROR]', e.message);
  }
});
EOF

# 测试
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' | node /tmp/mcp-logger.js
```

### 实验 5.1.2：查看现有 MCP 配置

```bash
# 查看当前 MCP 配置
cat ~/.claude/settings.json | jq '.mcpServers // empty'

# 或者在项目中
cat .claude/settings.json | jq '.mcpServers // empty' 2>/dev/null || echo "No project MCP config"
```

---

## 下一节

[5.2 配置 MCP Server](./02-mcp-configuration.md) - 学习如何配置和管理 MCP Server
