# 8.2 Vision 能力

Claude Code 具备强大的视觉理解能力，可以分析图片、截图、图表等视觉内容。

---

## 支持的图像格式

| 格式 | 扩展名 | 说明 |
|------|--------|------|
| JPEG | .jpg, .jpeg | 照片、截图 |
| PNG | .png | 截图、图表 |
| GIF | .gif | 静态图（不支持动图） |
| WebP | .webp | 现代图像格式 |

---

## 使用方式

### 1. 直接读取图片文件

```
# 在 Claude Code 中
> 分析 /path/to/screenshot.png 这张截图

# Claude 会使用 Read 工具读取图片并视觉分析
```

### 2. 分析 UI 截图

```
> 读取 /tmp/ui-screenshot.png 并分析这个界面的用户体验问题
```

### 3. 分析代码截图

```
> 这是一张代码截图 /tmp/code.png，帮我分析代码逻辑
```

---

## 应用场景

### 1. UI/UX 分析

```
> 分析这个登录页面截图，指出设计问题：
> /path/to/login-page.png

# Claude 会分析：
# - 布局合理性
# - 颜色对比度
# - 可访问性问题
# - 用户体验优化建议
```

### 2. 错误截图调试

```
> 这是我的错误截图 /tmp/error.png，帮我分析问题原因

# Claude 会：
# - 识别错误信息
# - 分析错误类型
# - 提供解决方案
```

### 3. 图表数据分析

```
> 分析这个性能监控图表 /path/to/metrics.png
> 识别异常模式和潜在问题

# Claude 会：
# - 识别图表类型
# - 分析数据趋势
# - 指出异常点
```

### 4. 架构图理解

```
> 读取 /path/to/architecture.png
> 解释这个系统架构，并指出潜在的单点故障

# Claude 会：
# - 识别组件和连接
# - 解释数据流
# - 分析架构问题
```

### 5. 手写内容识别

```
> 识别这张手写笔记的内容 /path/to/notes.jpg
> 整理成结构化的文本
```

---

## API 使用

### TypeScript 示例

```typescript
import Anthropic from "@anthropic-ai/sdk";
import * as fs from "fs";

const client = new Anthropic();

async function analyzeImage(imagePath: string, prompt: string) {
  // 读取图片并转换为 base64
  const imageData = fs.readFileSync(imagePath);
  const base64 = imageData.toString("base64");

  // 获取 MIME 类型
  const mimeType = imagePath.endsWith(".png")
    ? "image/png"
    : imagePath.endsWith(".jpg") || imagePath.endsWith(".jpeg")
    ? "image/jpeg"
    : "image/webp";

  const response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "image",
            source: {
              type: "base64",
              media_type: mimeType,
              data: base64,
            },
          },
          {
            type: "text",
            text: prompt,
          },
        ],
      },
    ],
  });

  return response.content[0].text;
}

// 使用
const analysis = await analyzeImage(
  "./screenshot.png",
  "分析这个 UI 界面的用户体验问题"
);
console.log(analysis);
```

### Python 示例

```python
import anthropic
import base64

client = anthropic.Anthropic()

def analyze_image(image_path: str, prompt: str) -> str:
    with open(image_path, "rb") as f:
        image_data = base64.standard_b64encode(f.read()).decode("utf-8")

    # 获取 MIME 类型
    if image_path.endswith(".png"):
        media_type = "image/png"
    elif image_path.endswith(".jpg") or image_path.endswith(".jpeg"):
        media_type = "image/jpeg"
    else:
        media_type = "image/webp"

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": media_type,
                        "data": image_data
                    }
                },
                {
                    "type": "text",
                    "text": prompt
                }
            ]
        }]
    )

    return response.content[0].text

# 使用
analysis = analyze_image("./screenshot.png", "分析这个界面")
print(analysis)
```

---

## 多图分析

### 比较多张图片

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "base64", media_type: "image/png", data: beforeImage },
        },
        {
          type: "image",
          source: { type: "base64", media_type: "image/png", data: afterImage },
        },
        {
          type: "text",
          text: "比较这两张截图，描述 UI 的变化",
        },
      ],
    },
  ],
});
```

---

## 实验练习

### 实验 8.2.1：分析 UI 截图

```bash
# 截取一个网页
# macOS: Cmd + Shift + 4
# 保存为 /tmp/webpage.png

# 在 Claude Code 中
claude
> 分析 /tmp/webpage.png 这个网页的设计，指出可以改进的地方
```

### 实验 8.2.2：识别代码截图

```bash
# 创建一个代码截图
# 或者使用现有的代码截图

claude
> 读取 /path/to/code-screenshot.png
> 将截图中的代码转换为文本，并指出可能的问题
```

### 实验 8.2.3：分析错误截图

```
> 这是我的控制台错误截图 /tmp/error.png
> 帮我分析错误原因并提供解决方案
```

---

## 最佳实践

### 1. 提供清晰的图片

- 使用高分辨率截图
- 确保文字清晰可读
- 避免过度压缩

### 2. 明确分析目标

```
# ✅ 好
> 分析这个表单的可访问性问题

# ❌ 差
> 看看这个图
```

### 3. 结合文本上下文

```
> 这是我们 React 应用的组件层级图
> 用户反馈页面加载慢，请分析可能的性能问题
```

---

## 限制

1. **图片大小**：单张图片最大约 20MB
2. **分辨率**：极高分辨率可能被降采样
3. **动态内容**：不支持 GIF 动画
4. **隐私信息**：注意不要发送敏感信息

---

## 下一节

[8.3 性能优化](./03-performance.md) - 学习优化 Token 使用和响应速度
