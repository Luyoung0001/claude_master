# 7.3 构建实用 Agent

本节将构建几个完整的实用 Agent 示例。

---

## 示例 1：代码审查 Agent

### 完整实现

```typescript
// code-review-agent.ts
import Anthropic from "@anthropic-ai/sdk";
import * as fs from "fs";
import * as path from "path";
import { execSync } from "child_process";

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "read_file",
    description: "Read the contents of a file",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string", description: "File path to read" },
      },
      required: ["path"],
    },
  },
  {
    name: "list_files",
    description: "List files in a directory with optional extension filter",
    input_schema: {
      type: "object",
      properties: {
        directory: { type: "string", description: "Directory path" },
        extension: { type: "string", description: "File extension filter" },
      },
      required: ["directory"],
    },
  },
  {
    name: "run_linter",
    description: "Run ESLint on a file or directory",
    input_schema: {
      type: "object",
      properties: {
        target: { type: "string", description: "File or directory to lint" },
      },
      required: ["target"],
    },
  },
  {
    name: "run_tests",
    description: "Run tests for a specific file",
    input_schema: {
      type: "object",
      properties: {
        testFile: { type: "string", description: "Test file to run" },
      },
      required: ["testFile"],
    },
  },
];

function executeTool(name: string, input: Record<string, unknown>): string {
  switch (name) {
    case "read_file":
      try {
        return fs.readFileSync(input.path as string, "utf-8");
      } catch (e) {
        return `Error reading file: ${e.message}`;
      }

    case "list_files":
      try {
        const dir = input.directory as string;
        const ext = input.extension as string;
        let files = fs.readdirSync(dir, { recursive: true }) as string[];

        if (ext) {
          files = files.filter((f) => f.endsWith(ext));
        }

        return files.slice(0, 50).join("\n");
      } catch (e) {
        return `Error listing files: ${e.message}`;
      }

    case "run_linter":
      try {
        const output = execSync(`npx eslint ${input.target} --format compact`, {
          encoding: "utf-8",
          timeout: 30000,
        });
        return output || "No linting errors found";
      } catch (e) {
        return e.stdout || e.message;
      }

    case "run_tests":
      try {
        const output = execSync(
          `npx jest ${input.testFile} --passWithNoTests`,
          {
            encoding: "utf-8",
            timeout: 60000,
          }
        );
        return output;
      } catch (e) {
        return e.stdout || e.message;
      }

    default:
      return `Unknown tool: ${name}`;
  }
}

async function reviewCode(targetPath: string) {
  const systemPrompt = `You are a senior code reviewer. Your job is to:
1. Read and analyze code files
2. Run linting checks
3. Identify potential bugs, security issues, and improvements
4. Provide actionable feedback

Be thorough but concise. Focus on important issues.`;

  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content: `Please review the code in: ${targetPath}

Perform a comprehensive code review:
1. List and read the relevant source files
2. Run linting checks
3. Analyze the code quality
4. Provide a summary of findings and recommendations`,
    },
  ];

  console.log("Starting code review...\n");

  while (true) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4096,
      system: systemPrompt,
      tools: tools,
      messages: messages,
    });

    // 处理响应
    for (const block of response.content) {
      if (block.type === "text") {
        console.log(block.text);
      }
    }

    if (response.stop_reason === "tool_use") {
      const toolResults: Anthropic.ToolResultBlockParam[] = [];

      for (const block of response.content) {
        if (block.type === "tool_use") {
          console.log(`\n[Executing: ${block.name}]`);
          const result = executeTool(
            block.name,
            block.input as Record<string, unknown>
          );
          console.log(`[Done]\n`);

          toolResults.push({
            type: "tool_result",
            tool_use_id: block.id,
            content: result,
          });
        }
      }

      messages.push({ role: "assistant", content: response.content });
      messages.push({ role: "user", content: toolResults });
    } else {
      break;
    }
  }
}

// 使用
reviewCode("./src");
```

---

## 示例 2：文档生成 Agent

```typescript
// doc-generator-agent.ts
import Anthropic from "@anthropic-ai/sdk";
import * as fs from "fs";

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "read_file",
    description: "Read source code file",
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
    description: "Write documentation file",
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
    name: "list_exports",
    description: "List exported functions and classes from a file",
    input_schema: {
      type: "object",
      properties: {
        path: { type: "string" },
      },
      required: ["path"],
    },
  },
];

function executeTool(name: string, input: Record<string, unknown>): string {
  switch (name) {
    case "read_file":
      return fs.readFileSync(input.path as string, "utf-8");

    case "write_file":
      fs.writeFileSync(input.path as string, input.content as string);
      return `Written to ${input.path}`;

    case "list_exports":
      const content = fs.readFileSync(input.path as string, "utf-8");
      const exports = content.match(/export\s+(function|class|const|let)\s+\w+/g);
      return exports ? exports.join("\n") : "No exports found";

    default:
      return `Unknown tool: ${name}`;
  }
}

async function generateDocs(sourceFile: string, outputFile: string) {
  const messages: Anthropic.MessageParam[] = [
    {
      role: "user",
      content: `Generate comprehensive documentation for ${sourceFile}.

1. Read the source file
2. Analyze all exported functions and classes
3. Generate Markdown documentation including:
   - Overview
   - Function/Class descriptions
   - Parameters and return types
   - Usage examples
4. Write the documentation to ${outputFile}`,
    },
  ];

  // ... Agent 循环（与前面类似）
}

generateDocs("./src/utils.ts", "./docs/utils.md");
```

---

## 示例 3：数据分析 Agent

```python
# data-analysis-agent.py
import anthropic
import pandas as pd
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "load_csv",
        "description": "Load a CSV file into memory",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string", "description": "Path to CSV file"}
            },
            "required": ["path"]
        }
    },
    {
        "name": "describe_data",
        "description": "Get statistical summary of the data",
        "input_schema": {
            "type": "object",
            "properties": {},
            "required": []
        }
    },
    {
        "name": "query_data",
        "description": "Query data using pandas syntax",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Pandas query string"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "plot_data",
        "description": "Create a plot and save to file",
        "input_schema": {
            "type": "object",
            "properties": {
                "x": {"type": "string", "description": "X axis column"},
                "y": {"type": "string", "description": "Y axis column"},
                "kind": {"type": "string", "description": "Plot type: line, bar, scatter"},
                "output": {"type": "string", "description": "Output file path"}
            },
            "required": ["x", "y", "kind", "output"]
        }
    }
]

# 全局数据存储
current_df = None

def execute_tool(name: str, input: dict) -> str:
    global current_df

    if name == "load_csv":
        current_df = pd.read_csv(input["path"])
        return f"Loaded {len(current_df)} rows, columns: {list(current_df.columns)}"

    elif name == "describe_data":
        if current_df is None:
            return "No data loaded"
        return current_df.describe().to_string()

    elif name == "query_data":
        if current_df is None:
            return "No data loaded"
        result = current_df.query(input["query"])
        return result.head(20).to_string()

    elif name == "plot_data":
        if current_df is None:
            return "No data loaded"
        import matplotlib.pyplot as plt
        current_df.plot(x=input["x"], y=input["y"], kind=input["kind"])
        plt.savefig(input["output"])
        plt.close()
        return f"Plot saved to {input['output']}"

    return f"Unknown tool: {name}"

def analyze_data(csv_path: str, question: str):
    messages = [{
        "role": "user",
        "content": f"""Analyze the data in {csv_path} to answer: {question}

1. Load the CSV file
2. Explore the data structure
3. Perform relevant analysis
4. Create visualizations if helpful
5. Provide insights and conclusions"""
    }]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )

        for block in response.content:
            if hasattr(block, 'text'):
                print(block.text)

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"\n[Executing: {block.name}]")
                    result = execute_tool(block.name, block.input)
                    print(f"[Result preview: {result[:200]}...]\n")
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })

            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
        else:
            break

# 使用
analyze_data("sales.csv", "What are the top selling products and trends?")
```

---

## Agent 架构模式

### 1. 单一职责 Agent

```typescript
// 专注于一个任务
const codeReviewAgent = createAgent({
  name: "CodeReviewer",
  tools: [readFile, runLint, analyzeComplexity],
  systemPrompt: "You are a code reviewer...",
});
```

### 2. 多 Agent 协作

```typescript
// 多个 Agent 协作
const agents = {
  planner: createAgent({...}),
  coder: createAgent({...}),
  reviewer: createAgent({...}),
  tester: createAgent({...})
};

async function developFeature(requirement: string) {
  // 1. 规划
  const plan = await agents.planner.run(requirement);

  // 2. 编码
  const code = await agents.coder.run(plan);

  // 3. 审查
  const review = await agents.reviewer.run(code);

  // 4. 测试
  const tests = await agents.tester.run(code);

  return { plan, code, review, tests };
}
```

### 3. 层级 Agent

```typescript
// 主 Agent 可以调用子 Agent
const tools = [
  {
    name: "delegate_to_specialist",
    description: "Delegate a subtask to a specialist agent",
    input_schema: {...}
  }
];

async function executeSpecialist(task: string, specialistType: string) {
  const specialist = getSpecialist(specialistType);
  return await specialist.run(task);
}
```

---

## 最佳实践

### 1. 工具设计

```typescript
// ✅ 好：明确的描述和参数
{
  name: "search_code",
  description: "Search for code patterns in the codebase using regex",
  input_schema: {
    properties: {
      pattern: {
        type: "string",
        description: "Regex pattern to search for"
      },
      fileType: {
        type: "string",
        description: "File extension to limit search, e.g., '.ts'"
      }
    }
  }
}

// ❌ 差：模糊的描述
{
  name: "search",
  description: "Search something",
  input_schema: {
    properties: {
      query: { type: "string" }
    }
  }
}
```

### 2. 错误处理

```typescript
function executeTool(name: string, input: unknown): string {
  try {
    // 执行工具
    return doExecute(name, input);
  } catch (error) {
    // 返回有用的错误信息，让 Claude 能够调整策略
    return `Error: ${error.message}. You may want to try a different approach.`;
  }
}
```

### 3. 资源限制

```typescript
const MAX_ITERATIONS = 20;
const MAX_TOKENS_PER_TURN = 4096;

async function runAgent(userMessage: string) {
  let iterations = 0;

  while (iterations < MAX_ITERATIONS) {
    iterations++;
    // ... agent 循环
  }

  if (iterations >= MAX_ITERATIONS) {
    console.log("Max iterations reached");
  }
}
```

---

## 第七章总结

| 主题 | 要点 |
|------|------|
| SDK 安装 | `@anthropic-ai/sdk` (TS) / `anthropic` (Python) |
| 消息格式 | role + content |
| 工具定义 | name + description + input_schema |
| Agent 循环 | 消息 → 响应 → 工具调用 → 结果 → 循环 |
| 最佳实践 | 清晰工具描述、错误处理、资源限制 |

---

## 下一章

[第八章：高级功能与技巧](../08-advanced-features/README.md) - 学习 Extended Thinking、Vision 等高级功能
