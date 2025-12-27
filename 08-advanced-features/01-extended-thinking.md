# 8.1 Extended Thinking

Extended Thinking（扩展思考）让 Claude 能够进行更深入、更复杂的推理。

---

## 什么是 Extended Thinking？

Extended Thinking 是 Claude 的一种高级推理模式：

- **更长的思考时间**：Claude 会花更多时间分析问题
- **更深入的推理**：逐步分解复杂问题
- **更高的准确性**：减少错误和遗漏
- **透明的思考过程**：可以看到推理步骤

---

## 使用场景

### 适合使用 Extended Thinking

1. **复杂架构设计**
   - 系统设计决策
   - 技术选型分析
   - 大规模重构规划

2. **困难 Bug 调试**
   - 多因素交织的问题
   - 难以复现的错误
   - 性能瓶颈分析

3. **代码审查**
   - 安全漏洞检测
   - 边界条件分析
   - 并发问题识别

4. **算法设计**
   - 复杂算法实现
   - 性能优化
   - 正确性证明

### 不适合使用

- 简单的代码生成
- 快速问答
- 格式转换等机械任务

---

## 在 Claude Code 中使用

### 命令行参数

```bash
# 启用扩展思考模式
claude --thinking

# 或者使用完整参数
claude --extended-thinking
```

### 在对话中请求

```
> 请使用深度思考模式分析这个并发问题
```

### API 使用

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 16000,
  thinking: {
    type: "enabled",
    budget_tokens: 10000,  // 分配给思考的 token 数
  },
  messages: [
    {
      role: "user",
      content: "分析这个分布式系统的一致性问题...",
    },
  ],
});

// 思考过程在 thinking 字段
for (const block of response.content) {
  if (block.type === "thinking") {
    console.log("Thinking:", block.thinking);
  } else if (block.type === "text") {
    console.log("Response:", block.text);
  }
}
```

---

## 思考过程示例

### 输入

```
分析以下代码的线程安全问题：

class Counter {
  private int count = 0;

  public void increment() {
    count++;
  }

  public int getCount() {
    return count;
  }
}
```

### Extended Thinking 输出

```
<thinking>
让我仔细分析这段代码的线程安全性...

1. 首先，识别共享状态：
   - count 是实例变量，会被多个线程共享

2. 分析 increment() 方法：
   - count++ 不是原子操作
   - 它包含三个步骤：读取 → 增加 → 写入
   - 多线程同时执行时可能出现 race condition

3. 具体场景分析：
   - 线程 A 读取 count = 0
   - 线程 B 读取 count = 0
   - 线程 A 写入 count = 1
   - 线程 B 写入 count = 1
   - 结果：丢失了一次增加

4. getCount() 的问题：
   - 虽然读取 int 是原子的
   - 但可能读到过时的值（可见性问题）

5. 解决方案评估：
   - synchronized：简单但性能较低
   - AtomicInteger：更高效
   - volatile + CAS：最佳性能
</thinking>

这段代码存在两个线程安全问题：

1. **竞态条件（Race Condition）**
   `count++` 不是原子操作，多线程并发时会丢失更新。

2. **可见性问题（Visibility）**
   一个线程的修改可能对其他线程不可见。

**推荐修复方案：**

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
  private AtomicInteger count = new AtomicInteger(0);

  public void increment() {
    count.incrementAndGet();
  }

  public int getCount() {
    return count.get();
  }
}
```
```

---

## Token 成本考虑

Extended Thinking 会消耗更多 Token：

| 模式 | 输入 Token | 思考 Token | 输出 Token | 总成本 |
|------|-----------|-----------|-----------|--------|
| 普通 | 1000 | 0 | 500 | 较低 |
| Extended | 1000 | 5000 | 500 | 较高 |

### 成本控制

```typescript
// 限制思考 Token 预算
thinking: {
  type: "enabled",
  budget_tokens: 5000,  // 最多 5000 token 用于思考
}
```

---

## 实验练习

### 实验 8.1.1：复杂问题分析

```bash
# 创建一个有问题的代码文件
cat > /tmp/buggy-code.ts << 'EOF'
class Cache<T> {
  private data: Map<string, { value: T; expiry: number }> = new Map();

  set(key: string, value: T, ttl: number) {
    this.data.set(key, {
      value,
      expiry: Date.now() + ttl
    });
  }

  get(key: string): T | undefined {
    const item = this.data.get(key);
    if (item && item.expiry > Date.now()) {
      return item.value;
    }
    return undefined;
  }

  // 清理过期项
  cleanup() {
    for (const [key, item] of this.data) {
      if (item.expiry <= Date.now()) {
        this.data.delete(key);
      }
    }
  }
}
EOF

# 使用 Extended Thinking 分析
claude --thinking
> 深入分析 /tmp/buggy-code.ts 的潜在问题，包括内存泄漏、并发安全、性能问题
```

### 实验 8.1.2：架构设计

```
# 在 Claude Code 中
> 使用深度思考模式，设计一个支持百万用户的实时聊天系统架构

# 观察 Claude 的思考过程
# 包括技术选型、扩展性考虑、故障处理等
```

---

## 最佳实践

### 1. 明确问题复杂度

```
# ✅ 适合 Extended Thinking
> 分析这个分布式事务的一致性保证，考虑网络分区场景

# ❌ 不需要 Extended Thinking
> 写一个 Hello World 程序
```

### 2. 提供充分上下文

```
# 提供完整的问题背景
> 我们的系统使用 PostgreSQL + Redis，QPS 约 10000
> 最近发现缓存和数据库数据不一致
> 请深入分析可能的原因和解决方案
```

### 3. 要求展示推理

```
> 请展示你的推理过程，逐步分析这个问题
```

---

## 下一节

[8.2 Vision 能力](./02-vision.md) - 学习使用 Claude 的图像分析能力
