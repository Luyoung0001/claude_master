# 8.3 性能优化

本节介绍如何优化 Claude Code 的性能，包括 Token 使用、响应速度和成本控制。

---

## Token 优化

### 理解 Token 消耗

| 操作 | Token 消耗 |
|------|-----------|
| 对话历史 | 持续累积 |
| 文件读取 | 约 1 token/4 字符 |
| 工具调用 | 工具定义 + 输入 + 输出 |
| 系统提示 | 每次请求都计入 |

### 优化策略

#### 1. 及时压缩上下文

```
# 当上下文接近限制时
> /compact

# 或定期压缩
> /cost
# 如果使用率 > 50%，考虑压缩
```

#### 2. 精确的文件读取

```
# ❌ 不好：读取整个大文件
> 读取 large-log.txt 并找出错误

# ✅ 好：只读取需要的部分
> 读取 large-log.txt 的最后 100 行
```

#### 3. 使用具体的搜索

```
# ❌ 不好：模糊搜索
> 在项目中搜索 error

# ✅ 好：精确搜索
> 在 src/api/*.ts 中搜索 "catch.*Error"
```

#### 4. 选择合适的模型

```bash
# 简单任务用 Haiku
claude --model haiku -p "格式化这个 JSON"

# 复杂任务用 Sonnet/Opus
claude --model opus -p "设计分布式缓存架构"
```

---

## 响应速度优化

### 1. 使用流式响应

```typescript
// 流式输出，用户可以更早看到结果
const stream = await client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: messages,
});

for await (const event of stream) {
  // 实时处理
}
```

### 2. 并行任务

```
> 同时分析 src/api 和 src/utils 目录的代码

# Claude 会并行启动多个子 Agent
```

### 3. 后台任务

```
> 在后台运行完整的测试套件，然后继续帮我写代码
```

### 4. 缓存策略

文件读取会被缓存，避免重复读取：

```
# 第一次读取：从磁盘
> 读取 package.json

# 后续读取：从缓存（更快）
> 再看一下 package.json 的 dependencies
```

---

## 成本控制

### 监控使用量

```
# 查看当前会话成本
> /cost

# 输出：
# Input tokens:  45,234
# Output tokens: 12,456
# Estimated cost: $0.42
```

### 设置预算

```typescript
// 在 API 中设置最大 token
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1000,  // 限制输出
  messages: messages,
});
```

### 模型选择策略

| 任务类型 | 推荐模型 | 成本 |
|---------|---------|------|
| 快速问答 | Haiku | $ |
| 代码生成 | Sonnet | $$ |
| 复杂分析 | Sonnet | $$ |
| 架构设计 | Opus | $$$ |

---

## 批量操作优化

### 1. 批量文件处理

```
# ❌ 不好：逐个处理
> 读取 file1.ts
> 读取 file2.ts
> 读取 file3.ts

# ✅ 好：批量处理
> 读取 src/*.ts 下的所有文件并分析
```

### 2. 合并相关请求

```
# ❌ 不好：分开请求
> 生成一个 React 组件
> 为组件添加样式
> 写组件测试

# ✅ 好：一次请求
> 创建一个 React 组件，包含样式和测试
```

---

## 上下文管理

### 清理不需要的上下文

```
# 开始新任务时
> /clear
> 现在帮我处理另一个完全不同的问题
```

### 保持焦点

```
# 复杂任务分步骤
> 第一步：分析现有代码
> /compact
> 第二步：设计新方案
> /compact
> 第三步：实现代码
```

---

## 实验练习

### 实验 8.3.1：比较模型速度

```bash
#!/bin/bash
# 比较不同模型的响应时间

PROMPT="用 Python 写一个快速排序算法"

echo "=== Haiku ==="
time claude --model haiku -p "$PROMPT" > /dev/null

echo "=== Sonnet ==="
time claude --model sonnet -p "$PROMPT" > /dev/null

echo "=== Opus ==="
time claude --model opus -p "$PROMPT" > /dev/null
```

### 实验 8.3.2：Token 优化

```
# 在 Claude Code 中

# 1. 进行一些对话
> 解释 JavaScript 的原型链
> 详细说明 ES6 的新特性
> 比较 let、const 和 var

# 2. 查看 token 使用
> /cost

# 3. 压缩上下文
> /compact

# 4. 再次查看
> /cost

# 比较压缩前后的差异
```

---

## 性能监控

### 使用 Hooks 监控

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%H:%M:%S')] $TOOL_NAME completed\" >> /tmp/claude-perf.log"
          }
        ]
      }
    ]
  }
}
```

### 分析日志

```bash
# 查看工具调用频率
cat /tmp/claude-perf.log | cut -d' ' -f2 | sort | uniq -c | sort -rn
```

---

## 最佳实践总结

| 策略 | 效果 |
|------|------|
| 使用 Haiku 处理简单任务 | 降低成本 50%+ |
| 及时压缩上下文 | 避免 token 限制 |
| 精确的文件读取 | 减少不必要的 token |
| 并行处理任务 | 提高速度 |
| 流式响应 | 改善用户体验 |

---

## 下一节

[8.4 安全与权限](./04-security.md) - 学习配置安全权限保护敏感操作
