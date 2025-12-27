# 7.2 工具定义与使用

工具（Tools）是 Agent 的核心能力，让 Claude 能够与外部世界交互。

---

## 工具定义

### 基本结构

```typescript
interface Tool {
  name: string; // 工具名称
  description: string; // 工具描述
  input_schema: JSONSchema; // 输入参数 Schema
}
```

### 示例：天气工具

```typescript
const weatherTool = {
  name: "get_weather",
  description:
    "Get the current weather for a specific location. Returns temperature and conditions.",
  input_schema: {
    type: "object",
    properties: {
      location: {
        type: "string",
        description: "The city and country, e.g., 'Tokyo, Japan'",
      },
      unit: {
        type: "string",
        enum: ["celsius", "fahrenheit"],
        description: "Temperature unit",
      },
    },
    required: ["location"],
  },
};
```

### 示例：数据库查询工具

```typescript
const dbQueryTool = {
  name: "query_database",
  description: "Execute a read-only SQL query on the database",
  input_schema: {
    type: "object",
    properties: {
      query: {
        type: "string",
        description: "The SQL SELECT query to execute",
      },
      limit: {
        type: "number",
        description: "Maximum number of rows to return",
        default: 100,
      },
    },
    required: ["query"],
  },
};
```

### 示例：文件操作工具

```typescript
const fileTools = [
  {
    name: "read_file",
    description: "Read the contents of a file",
    input_schema: {
      type: "object",
      properties: {
        path: {
          type: "string",
          description: "The file path to read",
        },
      },
      required: ["path"],
    },
  },
  {
    name: "write_file",
    description: "Write content to a file",
    input_schema: {
      type: "object",
      properties: {
        path: {
          type: "string",
          description: "The file path to write",
        },
        content: {
          type: "string",
          description: "The content to write",
        },
      },
      required: ["path", "content"],
    },
  },
];
```

---

## 工具执行

### TypeScript 实现

```typescript
import Anthropic from "@anthropic-ai/sdk";
import * as fs from "fs";

const client = new Anthropic();

// 工具定义
const tools: Anthropic.Tool[] = [
  {
    name: "read_file",
    description: "Read file contents",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string" },
      },
      required: ["path"],
    },
  },
  {
    name: "write_file",
    description: "Write content to file",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string" },
        content: { type: "string" },
      },
      required: ["path", "content"],
    },
  },
  {
    name: "list_directory",
    description: "List files in a directory",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string" },
      },
      required: ["path"],
    },
  },
];

// 工具执行函数
function executeTool(name: string, input: Record<string, unknown>): string {
  switch (name) {
    case "read_file":
      try {
        return fs.readFileSync(input.path as string, "utf-8");
      } catch (e) {
        return `Error: ${e.message}`;
      }

    case "write_file":
      try {
        fs.writeFileSync(input.path as string, input.content as string);
        return `File written successfully: ${input.path}`;
      } catch (e) {
        return `Error: ${e.message}`;
      }

    case "list_directory":
      try {
        const files = fs.readdirSync(input.path as string);
        return files.join("\n");
      } catch (e) {
        return `Error: ${e.message}`;
      }

    default:
      return `Unknown tool: ${name}`;
  }
}

// Agent 循环
async function runAgent(userMessage: string) {
  const messages: Anthropic.MessageParam[] = [
    { role: "user", content: userMessage },
  ];

  while (true) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4096,
      tools: tools,
      messages: messages,
    });

    // 打印助手响应
    for (const block of response.content) {
      if (block.type === "text") {
        console.log("Claude:", block.text);
      }
    }

    if (response.stop_reason === "tool_use") {
      // 收集所有工具调用
      const toolResults: Anthropic.ToolResultBlockParam[] = [];

      for (const block of response.content) {
        if (block.type === "tool_use") {
          console.log(`[Tool: ${block.name}]`);
          const result = executeTool(
            block.name,
            block.input as Record<string, unknown>
          );
          console.log(`[Result: ${result.substring(0, 100)}...]`);

          toolResults.push({
            type: "tool_result",
            tool_use_id: block.id,
            content: result,
          });
        }
      }

      // 添加消息
      messages.push({ role: "assistant", content: response.content });
      messages.push({ role: "user", content: toolResults });
    } else {
      break;
    }
  }
}

// 运行
runAgent("列出当前目录的文件，然后读取 package.json 的内容");
```

### Python 实现

```python
import anthropic
import os
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "read_file",
        "description": "Read file contents",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"}
            },
            "required": ["path"]
        }
    },
    {
        "name": "list_directory",
        "description": "List files in directory",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"}
            },
            "required": ["path"]
        }
    }
]

def execute_tool(name: str, input: dict) -> str:
    if name == "read_file":
        try:
            with open(input["path"], "r") as f:
                return f.read()
        except Exception as e:
            return f"Error: {e}"

    elif name == "list_directory":
        try:
            return "\n".join(os.listdir(input["path"]))
        except Exception as e:
            return f"Error: {e}"

    return f"Unknown tool: {name}"

def run_agent(user_message: str):
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )

        # 打印文本响应
        for block in response.content:
            if hasattr(block, 'text'):
                print(f"Claude: {block.text}")

        if response.stop_reason == "tool_use":
            tool_results = []

            for block in response.content:
                if block.type == "tool_use":
                    print(f"[Tool: {block.name}]")
                    result = execute_tool(block.name, block.input)
                    print(f"[Result: {result[:100]}...]")

                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })

            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
        else:
            break

run_agent("List files in current directory and read package.json")
```

---

## 高级工具模式

### 1. 带验证的工具

```typescript
function executeTool(name: string, input: unknown): string {
  // 验证输入
  if (!validateInput(name, input)) {
    return "Error: Invalid input parameters";
  }

  // 权限检查
  if (!hasPermission(name)) {
    return "Error: Permission denied";
  }

  // 执行
  return doExecute(name, input);
}
```

### 2. 异步工具

```typescript
async function executeToolAsync(
  name: string,
  input: Record<string, unknown>
): Promise<string> {
  switch (name) {
    case "fetch_url":
      const response = await fetch(input.url as string);
      return await response.text();

    case "query_api":
      const data = await callExternalAPI(input);
      return JSON.stringify(data);

    default:
      return `Unknown tool: ${name}`;
  }
}
```

### 3. 带超时的工具

```typescript
async function executeWithTimeout(
  name: string,
  input: Record<string, unknown>,
  timeout: number = 30000
): Promise<string> {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const result = await executeToolAsync(name, input);
    return result;
  } catch (error) {
    if (error.name === "AbortError") {
      return "Error: Tool execution timed out";
    }
    throw error;
  } finally {
    clearTimeout(timeoutId);
  }
}
```

---

## 实验练习

### 实验 7.2.1：计算器工具

```typescript
// calculator-agent.ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "calculate",
    description: "Perform mathematical calculations",
    input_schema: {
      type: "object",
      properties: {
        expression: {
          type: "string",
          description: "Mathematical expression to evaluate, e.g., '2 + 2'",
        },
      },
      required: ["expression"],
    },
  },
];

function calculate(expression: string): string {
  try {
    // 安全的数学表达式评估
    const sanitized = expression.replace(/[^0-9+\-*/().%\s]/g, "");
    const result = Function(`"use strict"; return (${sanitized})`)();
    return String(result);
  } catch (e) {
    return `Error: ${e.message}`;
  }
}

async function main() {
  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content: "计算 (25 * 4) + (100 / 5) 的结果",
    },
  ];

  let response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    tools: tools,
    messages: messages,
  });

  while (response.stop_reason === "tool_use") {
    const toolResults: Anthropic.ToolResultBlockParam[] = [];

    for (const block of response.content) {
      if (block.type === "tool_use") {
        const result = calculate(block.input.expression as string);
        console.log(`计算: ${block.input.expression} = ${result}`);

        toolResults.push({
          type: "tool_result",
          tool_use_id: block.id,
          content: result,
        });
      }
    }

    messages.push({ role: "assistant", content: response.content });
    messages.push({ role: "user", content: toolResults });

    response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1024,
      tools: tools,
      messages: messages,
    });
  }

  // 打印最终响应
  for (const block of response.content) {
    if (block.type === "text") {
      console.log("Claude:", block.text);
    }
  }
}

main();
```

### 实验 7.2.2：Web 搜索工具

```typescript
// search-agent.ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "web_search",
    description: "Search the web for information",
    input_schema: {
      type: "object",
      properties: {
        query: {
          type: "string",
          description: "Search query",
        },
      },
      required: ["query"],
    },
  },
];

// 模拟搜索结果
function webSearch(query: string): string {
  // 实际项目中应调用真实搜索 API
  return JSON.stringify({
    query: query,
    results: [
      { title: "Result 1", snippet: "This is a mock search result..." },
      { title: "Result 2", snippet: "Another mock result..." },
    ],
  });
}

// ... Agent 循环实现
```

---

## 下一节

[7.3 构建实用 Agent](./03-building-agents.md) - 学习构建完整的 Agent 应用
