# 2.1 文件操作工具

文件操作是 Claude Code 最核心的能力之一。本节将详细介绍五个文件相关工具。

---

## Read 工具 - 读取文件

### 基本用法

Read 工具用于读取文件内容，支持多种文件格式。

```
# 在对话中请求读取文件
> 请读取 package.json 文件

# Claude 会使用 Read 工具：
# Read: /path/to/package.json
# 返回文件内容（带行号）
```

### 支持的文件类型

| 类型 | 说明 |
|------|------|
| 文本文件 | .txt, .md, .json, .yaml, .xml 等 |
| 代码文件 | .js, .ts, .py, .go, .java, .rs 等 |
| 图片文件 | .png, .jpg, .gif, .webp（视觉分析） |
| PDF 文件 | .pdf（提取文本和视觉内容） |
| Jupyter | .ipynb（代码单元格和输出） |

### 部分读取

对于大文件，可以指定读取范围：

```
> 读取 large-file.log 的前 100 行

# Claude 使用 offset 和 limit 参数：
# Read: /path/to/large-file.log (offset: 0, limit: 100)
```

```
> 读取 app.js 的第 500-600 行

# Read: /path/to/app.js (offset: 500, limit: 100)
```

### 读取图片

```
> 分析这张截图 /tmp/screenshot.png

# Claude 会视觉分析图片内容
# 可以识别：UI 元素、文字、图表、代码截图等
```

### 读取 PDF

```
> 读取并总结 report.pdf 的内容

# Claude 会逐页提取文本和视觉内容
# 支持表格、图表识别
```

---

## Write 工具 - 写入文件

### 基本用法

Write 工具用于创建新文件或完全覆盖现有文件。

```
> 创建一个新的 Python 脚本 hello.py，内容是打印 Hello World

# Claude 使用 Write 工具：
# Write: /path/to/hello.py
# Content: print("Hello World")
```

### 注意事项

1. **覆盖警告**：Write 会完全覆盖现有文件
2. **必须先读取**：修改现有文件前，Claude 必须先 Read
3. **优先使用 Edit**：对于局部修改，应使用 Edit 工具

### 正确用法示例

```
> 创建一个新的配置文件 config.json

# ✅ 正确：创建新文件
# Write: config.json
# Content: { "debug": true }
```

```
> 修改 config.json，添加一个新字段

# ✅ 正确流程：
# 1. Read config.json
# 2. Edit config.json (添加新字段)

# ❌ 错误：直接 Write 会丢失其他内容
```

---

## Edit 工具 - 精确编辑

### 基本用法

Edit 工具用于对文件进行精确的局部修改，基于字符串替换。

```
> 将 app.js 中的 console.log 改为 logger.info

# Claude 使用 Edit 工具：
# Edit: /path/to/app.js
# old_string: console.log("message")
# new_string: logger.info("message")
```

### 关键参数

| 参数 | 必需 | 说明 |
|------|------|------|
| `file_path` | 是 | 文件绝对路径 |
| `old_string` | 是 | 要替换的原始文本 |
| `new_string` | 是 | 替换后的新文本 |
| `replace_all` | 否 | 替换所有匹配项（默认 false） |

### 唯一性要求

`old_string` 必须在文件中唯一，否则编辑会失败：

```
# ❌ 失败：old_string 不唯一
# Edit: app.js
# old_string: return
# new_string: return null
# Error: Found 15 matches, old_string must be unique

# ✅ 成功：提供更多上下文使其唯一
# old_string: function getData() {\n  return
# new_string: function getData() {\n  return null
```

### replace_all 用法

批量替换所有匹配项：

```
> 将所有 var 声明改为 const

# Edit: app.js
# old_string: var
# new_string: const
# replace_all: true
```

### 保持缩进

Edit 会保持原始缩进，无需手动处理：

```
# 原文件：
#     if (condition) {
#         doSomething();
#     }

# 编辑：
# old_string:         doSomething();
# new_string:         doSomethingElse();

# 结果正确保持缩进
```

---

## Glob 工具 - 文件模式匹配

### 基本用法

Glob 用于查找匹配特定模式的文件。

```
> 查找所有 TypeScript 文件

# Glob: **/*.ts
# 返回匹配的文件列表
```

### 常用模式

| 模式 | 说明 | 示例 |
|------|------|------|
| `*` | 匹配任意字符（不含 /） | `*.js` |
| `**` | 匹配任意目录层级 | `**/*.js` |
| `?` | 匹配单个字符 | `file?.txt` |
| `[abc]` | 匹配字符集 | `file[123].txt` |
| `{a,b}` | 匹配选项 | `*.{js,ts}` |

### 实用示例

```bash
# 所有 JavaScript/TypeScript 文件
**/*.{js,ts,jsx,tsx}

# src 目录下的所有文件
src/**/*

# 所有测试文件
**/*.test.{js,ts}
**/*.spec.{js,ts}
**/__tests__/**/*

# 所有配置文件
**/config*.{json,yaml,yml}

# 排除 node_modules
# （Glob 默认忽略 node_modules）
```

### 指定搜索目录

```
> 在 src/components 目录下查找所有 React 组件

# Glob: *.tsx
# path: /project/src/components
```

---

## Grep 工具 - 内容搜索

### 基本用法

Grep 基于 ripgrep，用于在文件内容中搜索。

```
> 搜索所有使用 useState 的文件

# Grep: useState
# 返回匹配的文件列表或内容
```

### 输出模式

| 模式 | 说明 |
|------|------|
| `files_with_matches` | 只返回文件路径（默认） |
| `content` | 返回匹配的行内容 |
| `count` | 返回每个文件的匹配数量 |

### 正则表达式支持

```
# 搜索函数定义
> 搜索所有 async function 定义

# Grep: async function \w+
# output_mode: content
```

```
# 搜索特定模式的 import
> 搜索从 @/utils 导入的语句

# Grep: import .* from ['"]@/utils
```

### 上下文显示

```
# 显示匹配行及前后各 3 行
# Grep: TODO
# output_mode: content
# -C: 3

# 只显示匹配行之前的 2 行
# -B: 2

# 只显示匹配行之后的 2 行
# -A: 2
```

### 文件类型过滤

```
# 只在 TypeScript 文件中搜索
# Grep: interface User
# type: ts

# 使用 glob 模式过滤
# Grep: class.*Component
# glob: "*.tsx"
```

### 大小写敏感

```
# 大小写不敏感搜索
# Grep: error
# -i: true
```

---

## 实验练习

### 实验 2.1.1：文件读取

```bash
# 创建测试文件
mkdir -p /tmp/claude-test
cd /tmp/claude-test

# 创建示例文件
cat > example.js << 'EOF'
function greet(name) {
  console.log(`Hello, ${name}!`);
}

function add(a, b) {
  return a + b;
}

module.exports = { greet, add };
EOF

# 在 Claude Code 中：
claude
> 读取 /tmp/claude-test/example.js 并解释每个函数的作用
```

### 实验 2.1.2：文件编辑

```
# 继续上面的会话

> 将 example.js 中的 console.log 改为 console.info

> 为 add 函数添加参数类型检查，如果参数不是数字则抛出错误

> 添加一个新函数 multiply(a, b) 到文件中
```

### 实验 2.1.3：批量搜索

```bash
# 创建多个测试文件
cd /tmp/claude-test
mkdir -p src/{components,utils,hooks}

cat > src/components/Button.tsx << 'EOF'
import React, { useState } from 'react';

export function Button({ label }) {
  const [clicked, setClicked] = useState(false);
  return <button onClick={() => setClicked(true)}>{label}</button>;
}
EOF

cat > src/hooks/useCounter.ts << 'EOF'
import { useState, useCallback } from 'react';

export function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = useCallback(() => setCount(c => c + 1), []);
  return { count, increment };
}
EOF

# 在 Claude Code 中：
claude
> 查找 /tmp/claude-test 中所有使用 useState 的文件
> 统计每个文件中 useState 的使用次数
> 查找所有 TypeScript 文件
```

### 实验 2.1.4：图片分析

```bash
# 如果有截图，可以测试图片分析
claude
> 分析 /path/to/screenshot.png，告诉我这个 UI 界面的组成部分
```

---

## 工具组合使用

### 场景：代码重构

```
> 1. 查找所有使用 moment.js 的文件
> 2. 读取这些文件的相关代码
> 3. 将 moment 替换为 dayjs

# Claude 会组合使用：
# 1. Grep: import.*moment / require.*moment
# 2. Read 相关文件
# 3. Edit 进行替换
```

### 场景：代码审查

```
> 审查 src 目录下的代码，找出所有 TODO 注释并整理成列表

# Claude 会：
# 1. Grep: TODO|FIXME|HACK (output_mode: content, -C: 1)
# 2. 整理并呈现结果
```

---

## 最佳实践

### 1. Read 之后再 Edit

```
# ✅ 正确流程
> 修改 config.json 的 debug 字段

# Claude 会先 Read，再 Edit
```

### 2. 使用具体的搜索模式

```
# ❌ 太宽泛
> 搜索 error

# ✅ 更具体
> 搜索 "catch.*Error" 在 .ts 文件中
```

### 3. 利用 Glob 缩小范围

```
# 先用 Glob 确定文件范围，再用 Grep 搜索内容
> 在 src/api 目录下的 .ts 文件中搜索 fetch 调用
```

---

## 下一节

[2.2 代码执行工具](./02-bash-tools.md) - 学习使用 Bash 工具执行系统命令
