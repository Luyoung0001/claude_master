# 第三章：Slash Commands 自定义命令

Slash Commands（斜杠命令）是 Claude Code 的强大特性之一，让你可以创建可复用的自定义命令，提高工作效率。

## 本章内容

- [3.1 内置命令](./01-builtin-commands.md)
- [3.2 自定义命令开发](./02-custom-commands.md)
- [3.3 命令最佳实践](./03-best-practices.md)

## 学习目标

完成本章后，你将能够：

1. 熟练使用所有内置斜杠命令
2. 创建自定义命令处理常见任务
3. 使用参数使命令更灵活
4. 构建团队共享的命令库

## 命令概览

### 内置命令

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助信息 |
| `/exit` | 退出当前会话 |
| `/clear` | 清除会话上下文 |
| `/compact` | 压缩上下文以节省 token |
| `/cost` | 显示 token 使用量和成本 |
| `/doctor` | 运行诊断检查 |
| `/config` | 显示当前配置 |
| `/model` | 查看或切换模型 |
| `/init` | 初始化 CLAUDE.md 文件 |

### 自定义命令

自定义命令存放在 `.claude/commands/` 目录，以 Markdown 文件形式定义。

## 预计学习时间

约 2-3 小时
