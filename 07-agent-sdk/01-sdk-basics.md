# 7.1 SDK 基础

本节介绍 Claude Agent SDK 的基本概念和使用方法。

---

## 安装 SDK

### TypeScript/Node.js

```bash
npm install @anthropic-ai/sdk
```

### Python

```bash
pip install anthropic
```

---

## 基本用法

### TypeScript 示例

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

async function main() {
  const message = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "Hello, Claude!",
      },
    ],
  });

  console.log(message.content);
}

main();
```

### Python 示例

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello, Claude!"}
    ]
)

print(message.content)
```

---

## 核心概念

### 1. 消息（Messages）

消息是与 Claude 交互的基本单位：

```typescript
interface Message {
  role: "user" | "assistant";
  content: string | ContentBlock[];
}

// 简单文本消息
const message = {
  role: "user",
  content: "What is TypeScript?",
};

// 包含多种内容的消息
const complexMessage = {
  role: "user",
  content: [
    { type: "text", text: "Analyze this image:" },
    {
      type: "image",
      source: {
        type: "base64",
        media_type: "image/png",
        data: "base64_encoded_image",
      },
    },
  ],
};
```

### 2. 工具（Tools）

工具让 Claude 能够执行操作：

```typescript
const tools = [
  {
    name: "get_weather",
    description: "Get current weather for a location",
    input_schema: {
      type: "object",
      properties: {
        location: {
          type: "string",
          description: "City name",
        },
      },
      required: ["location"],
    },
  },
];
```

### 3. 系统提示（System Prompt）

定义 Claude 的角色和行为：

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  system: "You are a helpful coding assistant specialized in TypeScript.",
  messages: [...],
});
```

---

## Agent 循环

Agent 的核心是工具调用循环：

```typescript
async function runAgent(userMessage: string) {
  const messages: Message[] = [{ role: "user", content: userMessage }];

  while (true) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4096,
      tools: tools,
      messages: messages,
    });

    // 检查是否需要工具调用
    if (response.stop_reason === "tool_use") {
      // 提取工具调用
      const toolUse = response.content.find((c) => c.type === "tool_use");

      // 执行工具
      const toolResult = await executeTools(toolUse);

      // 添加助手响应和工具结果
      messages.push({ role: "assistant", content: response.content });
      messages.push({
        role: "user",
        content: [
          {
            type: "tool_result",
            tool_use_id: toolUse.id,
            content: toolResult,
          },
        ],
      });
    } else {
      // 没有更多工具调用，返回最终响应
      return response;
    }
  }
}
```

### Python 版本

```python
async def run_agent(user_message: str):
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "tool_use":
            # 处理工具调用
            tool_use = next(c for c in response.content if c.type == "tool_use")
            tool_result = execute_tool(tool_use)

            messages.append({"role": "assistant", "content": response.content})
            messages.append({
                "role": "user",
                "content": [{
                    "type": "tool_result",
                    "tool_use_id": tool_use.id,
                    "content": tool_result
                }]
            })
        else:
            return response
```

---

## 流式响应

### TypeScript 流式处理

```typescript
const stream = await client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Tell me a story" }],
});

for await (const event of stream) {
  if (
    event.type === "content_block_delta" &&
    event.delta.type === "text_delta"
  ) {
    process.stdout.write(event.delta.text);
  }
}
```

### Python 流式处理

```python
with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## 错误处理

```typescript
import Anthropic, { APIError, RateLimitError } from "@anthropic-ai/sdk";

try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof RateLimitError) {
    console.log("Rate limited, waiting...");
    await sleep(60000);
    // 重试
  } else if (error instanceof APIError) {
    console.error("API error:", error.message);
  } else {
    throw error;
  }
}
```

### Python 错误处理

```python
from anthropic import APIError, RateLimitError

try:
    response = client.messages.create(...)
except RateLimitError:
    print("Rate limited, waiting...")
    time.sleep(60)
    # 重试
except APIError as e:
    print(f"API error: {e}")
```

---

## 实验练习

### 实验 7.1.1：基本消息

```typescript
// basic-message.ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

async function main() {
  const response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 256,
    messages: [
      {
        role: "user",
        content: "用一句话解释什么是 TypeScript",
      },
    ],
  });

  console.log(response.content[0].text);
}

main();
```

### 实验 7.1.2：多轮对话

```typescript
// conversation.ts
import Anthropic from "@anthropic-ai/sdk";
import * as readline from "readline";

const client = new Anthropic();
const messages: Anthropic.MessageParam[] = [];

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

async function chat(userInput: string) {
  messages.push({ role: "user", content: userInput });

  const response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    system: "你是一个友好的助手",
    messages: messages,
  });

  const assistantMessage = response.content[0].text;
  messages.push({ role: "assistant", content: assistantMessage });

  return assistantMessage;
}

function prompt() {
  rl.question("You: ", async (input) => {
    if (input.toLowerCase() === "exit") {
      rl.close();
      return;
    }

    const response = await chat(input);
    console.log(`Claude: ${response}\n`);
    prompt();
  });
}

console.log("Chat with Claude (type 'exit' to quit)\n");
prompt();
```

### 实验 7.1.3：流式响应

```typescript
// streaming.ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

async function main() {
  console.log("Claude: ");

  const stream = await client.messages.stream({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: "写一个关于 AI 的短诗",
      },
    ],
  });

  for await (const event of stream) {
    if (
      event.type === "content_block_delta" &&
      event.delta.type === "text_delta"
    ) {
      process.stdout.write(event.delta.text);
    }
  }

  console.log("\n");
}

main();
```

---

## 下一节

[7.2 工具定义与使用](./02-tools.md) - 学习如何定义和实现自定义工具
