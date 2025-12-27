# 1.1 安装与配置

## 安装 Claude Code CLI

### 方式一：使用 npm（推荐）

```bash
# 全局安装
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version
```

### 方式二：使用 Homebrew（macOS）

```bash
# 添加 tap
brew tap anthropics/claude-code

# 安装
brew install claude-code

# 验证安装
claude --version
```

### 方式三：直接下载二进制

从 [GitHub Releases](https://github.com/anthropics/claude-code/releases) 下载对应平台的二进制文件。

---

## 环境变量配置

### 基本配置

Claude Code 需要 API 密钥才能工作。有两种配置方式：

#### 方式一：环境变量

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export ANTHROPIC_API_KEY="sk-ant-xxxxx"

# 使配置生效
source ~/.zshrc
```

#### 方式二：首次运行时登录

```bash
# 首次运行会提示登录
claude

# 按提示完成 OAuth 登录流程
```

### 自定义端点配置

如果使用代理或自定义端点：

```bash
# 设置自定义 API 端点
export ANTHROPIC_BASE_URL="https://your-proxy.example.com"

# 如果需要自定义认证
export ANTHROPIC_AUTH_TOKEN="your-token"
```

---

## settings.json 配置文件详解

Claude Code 使用 `settings.json` 进行配置，配置文件位置：

- **用户级别**: `~/.claude/settings.json`
- **项目级别**: `.claude/settings.json`（当前项目目录）

项目级别配置会覆盖用户级别配置。

### 完整配置示例

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-xxxxx",
    "ANTHROPIC_BASE_URL": "https://api.anthropic.com"
  },
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(git:*)",
      "Read",
      "Write",
      "Edit"
    ],
    "deny": [
      "Bash(rm -rf:*)"
    ]
  },
  "hooks": {},
  "mcpServers": {}
}
```

### 配置项说明

| 配置项 | 类型 | 说明 |
|--------|------|------|
| `env` | object | 环境变量，会注入到 Claude Code 运行环境 |
| `model` | string | 默认使用的模型 |
| `permissions` | object | 权限控制，allow/deny 列表 |
| `hooks` | object | 钩子配置（第4章详解） |
| `mcpServers` | object | MCP 服务器配置（第5章详解） |

---

## 模型选择

Claude Code 支持多种模型，各有特点：

### 可用模型

| 模型 | 模型 ID | 特点 | 适用场景 |
|------|---------|------|----------|
| **Claude Opus 4** | `claude-opus-4-5-20250514` | 最强能力，深度思考 | 复杂架构设计、困难 bug |
| **Claude Sonnet 4** | `claude-sonnet-4-20250514` | 平衡性能与成本 | 日常开发、代码审查 |
| **Claude Haiku** | `claude-haiku-3-5-20241022` | 快速响应，成本低 | 简单任务、快速查询 |

### 切换模型

```bash
# 命令行参数
claude --model claude-opus-4-5-20250514

# 或使用简写
claude --model opus
claude --model sonnet
claude --model haiku
```

### 在 settings.json 中设置默认模型

```json
{
  "model": "claude-sonnet-4-20250514"
}
```

---

## 实验练习

### 实验 1.1.1：验证安装

```bash
# 1. 检查版本
claude --version

# 2. 查看帮助
claude --help

# 3. 运行诊断
claude /doctor
```

### 实验 1.1.2：配置文件创建

创建你的第一个项目配置文件：

```bash
# 创建项目级别配置目录
mkdir -p .claude

# 创建配置文件
cat > .claude/settings.json << 'EOF'
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "allow": [
      "Read",
      "Bash(ls:*)",
      "Bash(git status:*)"
    ]
  }
}
EOF
```

### 实验 1.1.3：测试不同模型

```bash
# 使用 Haiku 快速回答
claude --model haiku -p "什么是 REST API？用一句话解释"

# 使用 Sonnet 进行代码任务
claude --model sonnet -p "写一个计算斐波那契数列的 Python 函数"

# 使用 Opus 进行复杂分析
claude --model opus -p "分析微服务架构与单体架构的优缺点"
```

---

## 常见问题

### Q: 安装后提示 command not found？

确保 npm 全局安装目录在 PATH 中：

```bash
# 查看 npm 全局目录
npm config get prefix

# 添加到 PATH（以 zsh 为例）
echo 'export PATH="$(npm config get prefix)/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Q: API 密钥无效？

```bash
# 验证密钥格式
echo $ANTHROPIC_API_KEY

# 密钥应以 sk-ant- 开头
# 如果使用自定义端点，检查 ANTHROPIC_BASE_URL
```

### Q: 如何查看当前配置？

```bash
# 查看生效的配置
claude /config
```

---

## 下一节

[1.2 基本命令](./02-basic-commands.md) - 学习 Claude Code 的核心命令
