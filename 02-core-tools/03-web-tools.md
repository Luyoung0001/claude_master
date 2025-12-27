# 2.3 Web 工具

Claude Code 提供了两个强大的 Web 工具，让你能够搜索互联网和获取网页内容，获取最新信息。

---

## WebSearch 工具

### 功能介绍

WebSearch 工具让 Claude 能够搜索互联网，获取实时信息：

- 最新的技术文档
- 当前事件和新闻
- 库和框架的更新
- 错误信息的解决方案

### 基本用法

```
> 搜索 React 19 的新特性

# WebSearch: "React 19 new features 2025"
# 返回搜索结果摘要和链接
```

### 搜索技巧

#### 精确搜索

```
> 搜索 "useEffect cleanup" 的最佳实践

# WebSearch: "useEffect cleanup best practices React 2025"
```

#### 错误排查

```
> 搜索这个错误：TypeError: Cannot read property 'map' of undefined

# WebSearch: "TypeError Cannot read property map of undefined React"
```

#### 库文档

```
> 搜索 Prisma ORM 的官方文档

# WebSearch: "Prisma ORM official documentation"
```

### 域名过滤

可以限制或排除特定网站：

```
# 只搜索特定网站
> 在 MDN 上搜索 JavaScript Promise

# WebSearch: "JavaScript Promise"
# allowed_domains: ["developer.mozilla.org"]

# 排除某些网站
> 搜索 Python 教程，但不要 W3Schools

# WebSearch: "Python tutorial"
# blocked_domains: ["w3schools.com"]
```

### 引用来源

搜索后 Claude 会提供来源链接：

```
Sources:
- [React 19 Release Notes](https://react.dev/blog/2024/...)
- [What's New in React 19](https://example.com/...)
```

---

## WebFetch 工具

### 功能介绍

WebFetch 工具用于获取特定网页的内容：

- 读取在线文档
- 获取 API 响应
- 分析网页内容
- 提取特定信息

### 基本用法

```
> 获取 https://react.dev/learn 页面的内容

# WebFetch: https://react.dev/learn
# prompt: "提取主要内容和教程列表"
```

### prompt 参数

WebFetch 使用 prompt 来指导内容提取：

```
# 提取特定信息
> 从 GitHub Release 页面获取最新版本号

# WebFetch: https://github.com/nodejs/node/releases
# prompt: "提取最新版本号和发布日期"
```

```
# 总结长文档
> 总结这篇技术博客的要点

# WebFetch: https://blog.example.com/article
# prompt: "总结文章的主要观点，列出关键要点"
```

### 处理重定向

WebFetch 会通知重定向并提供新 URL：

```
# 如果发生跨域重定向
# WebFetch 返回: "Redirected to: https://new-domain.com/page"
# Claude 会自动发起新请求
```

### 缓存机制

WebFetch 有 15 分钟的缓存：

- 重复请求同一 URL 会使用缓存
- 加快响应速度
- 减少网络请求

---

## 实用场景

### 场景 1：查找库的使用方法

```
> 我想使用 Zod 进行表单验证，帮我搜索相关教程

# Claude:
# 1. WebSearch: "Zod form validation React 2025"
# 2. WebFetch: 获取相关教程页面
# 3. 总结使用方法并提供示例代码
```

### 场景 2：调试错误

```
> 我遇到这个错误，帮我找解决方案：
> Error: ENOENT: no such file or directory, open '/app/.env'

# Claude:
# 1. WebSearch: "ENOENT no such file or directory .env Node.js"
# 2. 分析搜索结果
# 3. 提供多种解决方案
```

### 场景 3：了解最新技术动态

```
> 帮我了解 2025 年 Node.js 的最新版本和特性

# Claude:
# 1. WebSearch: "Node.js latest version 2025 features"
# 2. WebFetch: Node.js 官方博客或发布页面
# 3. 总结新特性
```

### 场景 4：API 文档查询

```
> 获取 OpenAI API 的定价信息

# Claude:
# 1. WebFetch: https://openai.com/pricing
# 2. 提取定价表格
# 3. 格式化展示
```

---

## 实验练习

### 实验 2.3.1：基本搜索

```
# 在 Claude Code 中：
claude

> 搜索 TypeScript 5.0 的新特性

> 搜索如何在 Next.js 14 中使用 Server Actions

> 搜索 "Tailwind CSS responsive design" 的最佳实践
```

### 实验 2.3.2：错误排查

```
> 搜索这个错误的解决方案：
> Module not found: Can't resolve '@/components/Button'

> 搜索 "CORS error fetch API" 的解决方法
```

### 实验 2.3.3：获取网页内容

```
> 获取 https://nodejs.org/en 的内容，告诉我当前的 LTS 版本

> 获取 https://github.com/vercel/next.js 的内容，告诉我最新版本和 star 数量
```

### 实验 2.3.4：综合应用

```
# 研究一个新库
> 我想使用 tRPC，请帮我：
> 1. 搜索 tRPC 的基本概念
> 2. 获取官方文档的快速开始指南
> 3. 总结核心概念和使用方法
```

---

## 与 MCP 的关系

如果配置了 MCP Server 提供的 Web 工具，可能有更强大的功能：

```
# 如果有 mcp__browser__fetch 等工具
# Claude 会优先使用 MCP 工具，因为可能限制更少

# 例如 Puppeteer MCP Server 可以：
# - 渲染 JavaScript 页面
# - 执行页面交互
# - 截取屏幕截图
```

---

## 限制与注意事项

### WebSearch 限制

- 目前仅在美国地区可用
- 搜索结果数量有限
- 可能无法访问付费内容

### WebFetch 限制

- 无法执行 JavaScript（纯 HTML 提取）
- 超大页面会被截断
- 某些网站可能阻止访问
- HTTPS 要求（HTTP 会自动升级）

### 最佳实践

1. **使用具体的搜索词**：更精确的结果
2. **添加年份**：获取最新信息
3. **结合使用**：先搜索找到好的链接，再 fetch 详细内容
4. **检查来源**：验证信息的可靠性

---

## 故障排除

### Q: 搜索结果不相关

```
# 尝试更具体的搜索词
> 搜索 "React useState async update batching"

# 而不是
> 搜索 "React state"
```

### Q: WebFetch 返回空内容

```
# 可能原因：
# 1. 页面需要 JavaScript 渲染
# 2. 网站阻止了爬虫
# 3. 需要登录

# 解决方案：
# 1. 使用 MCP Puppeteer Server
# 2. 尝试其他来源
# 3. 直接搜索内容
```

### Q: 无法访问某些网站

```
# 检查 URL 格式
# 确保使用 HTTPS
# 某些网站可能地域限制

# 可以使用 allowed_domains 限制可信源
```

---

## 下一节

[2.4 任务管理工具](./04-task-tools.md) - 学习使用 Task 和 TodoWrite 管理复杂任务
