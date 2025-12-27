# 5.4 自定义 MCP Server 开发

本节介绍如何开发自己的 MCP Server，以扩展 Claude Code 的能力。

---

## 开发环境准备

### TypeScript/Node.js

```bash
# 创建项目
mkdir my-mcp-server
cd my-mcp-server
npm init -y

# 安装依赖
npm install @modelcontextprotocol/sdk
npm install -D typescript @types/node

# 初始化 TypeScript
npx tsc --init
```

### Python

```bash
# 创建项目
mkdir my-mcp-server
cd my-mcp-server

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/macOS
# or: venv\Scripts\activate  # Windows

# 安装 SDK
pip install mcp
```

---

## 基础 Server 结构

### TypeScript 版本

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

// 创建 Server 实例
const server = new Server(
  {
    name: "my-custom-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// 定义工具列表
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "hello",
        description: "Say hello to someone",
        inputSchema: {
          type: "object",
          properties: {
            name: {
              type: "string",
              description: "Name to greet",
            },
          },
          required: ["name"],
        },
      },
      {
        name: "add",
        description: "Add two numbers",
        inputSchema: {
          type: "object",
          properties: {
            a: { type: "number", description: "First number" },
            b: { type: "number", description: "Second number" },
          },
          required: ["a", "b"],
        },
      },
    ],
  };
});

// 处理工具调用
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "hello":
      return {
        content: [
          {
            type: "text",
            text: `Hello, ${args.name}! Nice to meet you.`,
          },
        ],
      };

    case "add":
      const result = (args.a as number) + (args.b as number);
      return {
        content: [
          {
            type: "text",
            text: `The sum of ${args.a} and ${args.b} is ${result}`,
          },
        ],
      };

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// 启动 Server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("My MCP Server running on stdio");
}

main().catch(console.error);
```

### Python 版本

```python
# server.py
import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

# 创建 Server
server = Server("my-custom-server")

# 定义工具
@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="hello",
            description="Say hello to someone",
            inputSchema={
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string",
                        "description": "Name to greet"
                    }
                },
                "required": ["name"]
            }
        ),
        Tool(
            name="add",
            description="Add two numbers",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number", "description": "First number"},
                    "b": {"type": "number", "description": "Second number"}
                },
                "required": ["a", "b"]
            }
        )
    ]

# 处理工具调用
@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "hello":
        return [TextContent(
            type="text",
            text=f"Hello, {arguments['name']}! Nice to meet you."
        )]

    elif name == "add":
        result = arguments["a"] + arguments["b"]
        return [TextContent(
            type="text",
            text=f"The sum of {arguments['a']} and {arguments['b']} is {result}"
        )]

    else:
        raise ValueError(f"Unknown tool: {name}")

# 主函数
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 构建和运行

### TypeScript 构建

```bash
# package.json
{
  "name": "my-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "bin": {
    "my-mcp-server": "dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  }
}

# tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*"]
}
```

```bash
# 构建
npm run build

# 测试运行
npm start
```

### 配置 Claude Code

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/my-mcp-server/dist/index.js"]
    }
  }
}
```

---

## 实用 Server 示例

### 示例 1：天气查询 Server

```typescript
// weather-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "weather-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_weather",
      description: "Get current weather for a city",
      inputSchema: {
        type: "object",
        properties: {
          city: { type: "string", description: "City name" },
        },
        required: ["city"],
      },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "get_weather") {
    // 实际项目中应调用真实 API
    const mockWeather = {
      city: args.city,
      temperature: Math.floor(Math.random() * 30) + 5,
      condition: ["Sunny", "Cloudy", "Rainy"][Math.floor(Math.random() * 3)],
    };

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(mockWeather, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

const transport = new StdioServerTransport();
server.connect(transport);
```

### 示例 2：代码分析 Server

```typescript
// code-analyzer-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import * as fs from "fs";
import * as path from "path";

const server = new Server(
  { name: "code-analyzer", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "analyze_complexity",
      description: "Analyze code complexity of a file",
      inputSchema: {
        type: "object",
        properties: {
          filePath: { type: "string", description: "Path to the file" },
        },
        required: ["filePath"],
      },
    },
    {
      name: "count_lines",
      description: "Count lines of code in a directory",
      inputSchema: {
        type: "object",
        properties: {
          directory: { type: "string" },
          extension: { type: "string", description: "File extension filter" },
        },
        required: ["directory"],
      },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "analyze_complexity") {
    const content = fs.readFileSync(args.filePath as string, "utf-8");
    const lines = content.split("\n").length;
    const functions = (content.match(/function\s+\w+/g) || []).length;
    const classes = (content.match(/class\s+\w+/g) || []).length;

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify({ lines, functions, classes }, null, 2),
        },
      ],
    };
  }

  if (name === "count_lines") {
    const dir = args.directory as string;
    const ext = args.extension as string || ".ts";
    let totalLines = 0;
    let fileCount = 0;

    const walkDir = (d: string) => {
      for (const file of fs.readdirSync(d)) {
        const fullPath = path.join(d, file);
        if (fs.statSync(fullPath).isDirectory()) {
          walkDir(fullPath);
        } else if (file.endsWith(ext)) {
          totalLines += fs.readFileSync(fullPath, "utf-8").split("\n").length;
          fileCount++;
        }
      }
    };

    walkDir(dir);

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify({ totalLines, fileCount }, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

const transport = new StdioServerTransport();
server.connect(transport);
```

---

## 添加资源支持

```typescript
import {
  ListResourcesRequestSchema,
  ReadResourceRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "resource-server", version: "1.0.0" },
  {
    capabilities: {
      tools: {},
      resources: {},  // 启用资源支持
    },
  }
);

// 列出资源
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "config://app",
      name: "Application Config",
      mimeType: "application/json",
    },
    {
      uri: "doc://readme",
      name: "README",
      mimeType: "text/markdown",
    },
  ],
}));

// 读取资源
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;

  if (uri === "config://app") {
    return {
      contents: [
        {
          uri,
          mimeType: "application/json",
          text: JSON.stringify({ debug: true, version: "1.0.0" }),
        },
      ],
    };
  }

  if (uri === "doc://readme") {
    return {
      contents: [
        {
          uri,
          mimeType: "text/markdown",
          text: "# My Application\n\nThis is the README.",
        },
      ],
    };
  }

  throw new Error(`Unknown resource: ${uri}`);
});
```

---

## 实验练习

### 实验 5.4.1：创建简单的计算器 Server

```bash
mkdir calculator-server
cd calculator-server
npm init -y
npm install @modelcontextprotocol/sdk
npm install -D typescript @types/node
```

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "calculator", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "calculate",
      description: "Perform mathematical calculations",
      inputSchema: {
        type: "object",
        properties: {
          operation: {
            type: "string",
            enum: ["add", "subtract", "multiply", "divide"],
          },
          a: { type: "number" },
          b: { type: "number" },
        },
        required: ["operation", "a", "b"],
      },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { arguments: args } = request.params;
  const { operation, a, b } = args as { operation: string; a: number; b: number };

  let result: number;
  switch (operation) {
    case "add": result = a + b; break;
    case "subtract": result = a - b; break;
    case "multiply": result = a * b; break;
    case "divide": result = a / b; break;
    default: throw new Error("Unknown operation");
  }

  return {
    content: [{ type: "text", text: `Result: ${result}` }],
  };
});

const transport = new StdioServerTransport();
server.connect(transport);
```

```bash
# 构建并测试
npx tsc
node dist/index.js
```

---

## 第五章总结

| 主题 | 要点 |
|------|------|
| MCP 协议 | Resources, Tools, Prompts |
| 通信方式 | stdio（推荐）, HTTP/SSE |
| 配置 | mcpServers in settings.json |
| 官方 Server | filesystem, postgres, github, slack |
| 自定义开发 | TypeScript/Python SDK |

---

## 下一章

[第六章：IDE 集成](../06-ide-integration/README.md) - 学习在各种 IDE 中使用 Claude Code
