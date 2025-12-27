# 3.3 命令最佳实践

本节介绍设计和管理自定义命令的最佳实践，包括命令设计原则、团队共享和模板库建设。

---

## 命令设计原则

### 1. 单一职责

每个命令应该只做一件事：

```markdown
<!-- ❌ 不好：一个命令做太多事 -->
审查代码，生成测试，更新文档，并提交 git。

<!-- ✅ 好：职责单一 -->
审查代码并给出改进建议。
```

### 2. 清晰的指令

提供明确、具体的指令：

```markdown
<!-- ❌ 模糊 -->
改进这段代码。

<!-- ✅ 清晰 -->
从以下方面改进代码：
1. 可读性：变量命名、代码结构
2. 性能：算法复杂度、内存使用
3. 安全性：输入验证、错误处理

提供修改后的代码和解释。
```

### 3. 结构化输出

要求结构化的输出格式：

```markdown
分析结果请使用以下格式：

## 概述
[一句话总结]

## 发现的问题
| 严重程度 | 问题描述 | 位置 | 建议 |
|---------|---------|------|------|
| ...     | ...     | ...  | ...  |

## 改进建议
1. [建议1]
2. [建议2]

## 示例代码
```
[修复后的代码]
```
```

### 4. 上下文感知

利用项目上下文：

```markdown
审查代码时，请考虑：

1. 参考 CLAUDE.md 中的项目约定
2. 遵循项目现有的代码风格
3. 使用项目已有的工具函数
4. 保持与现有架构一致
```

### 5. 可配置性

通过参数提供灵活性：

```markdown
生成代码，配置如下：

$ARGUMENTS

如果未指定，使用默认配置：
- 语言：TypeScript
- 风格：函数式
- 测试：Jest
```

---

## 命令模板库

### 项目初始化模板

`.claude/commands/project/init.md`：

```markdown
初始化新项目，类型：$ARGUMENTS

请执行以下步骤：

1. **项目结构创建**
   - 创建标准目录结构
   - 生成配置文件
   - 设置 Git 忽略规则

2. **依赖安装**
   - 创建 package.json
   - 安装必要依赖
   - 配置开发工具

3. **代码模板**
   - 创建入口文件
   - 添加示例代码
   - 配置构建脚本

4. **文档**
   - 生成 README.md
   - 创建 CLAUDE.md
   - 添加贡献指南

请逐步执行并报告进度。
```

### 代码质量模板

`.claude/commands/quality/check.md`：

```markdown
对以下目标执行质量检查：

$ARGUMENTS

检查项目：

## 静态分析
- 运行 ESLint/TSLint
- 检查类型错误
- 分析代码复杂度

## 安全扫描
- 检查依赖漏洞
- 分析代码安全问题
- 检查敏感信息泄露

## 测试状态
- 运行测试套件
- 分析覆盖率
- 检查失败的测试

## 报告
生成综合报告，包括：
- 发现的问题数量
- 严重程度分布
- 改进建议
- 优先级排序
```

### Git 工作流模板

`.claude/commands/git/pr.md`：

```markdown
为当前分支创建 Pull Request：

$ARGUMENTS

步骤：

1. **变更分析**
   - 查看所有提交
   - 分析修改的文件
   - 理解变更目的

2. **PR 标题**
   - 简洁描述变更
   - 遵循规范格式

3. **PR 描述**
   - 变更概述
   - 关键修改说明
   - 测试计划
   - 截图（如适用）

4. **检查清单**
   - [ ] 代码已自审
   - [ ] 测试已通过
   - [ ] 文档已更新
   - [ ] 无破坏性变更

5. **创建 PR**
   - 使用 gh 命令创建
   - 添加适当标签
   - 分配审查者
```

---

## 团队共享

### 共享命令仓库

```bash
# 创建团队共享的命令仓库
mkdir team-claude-commands
cd team-claude-commands
git init

# 组织结构
mkdir -p commands/{review,gen,deploy,docs}

# 添加命令文件
# ...

# 推送到共享仓库
git remote add origin git@github.com:team/claude-commands.git
git push -u origin main
```

### 在项目中使用共享命令

```bash
# 方法1：Git submodule
git submodule add git@github.com:team/claude-commands.git .claude/team-commands

# 创建符号链接
ln -s .claude/team-commands/commands/* .claude/commands/

# 方法2：直接克隆到用户目录（全局可用）
git clone git@github.com:team/claude-commands.git ~/.claude/team-commands
ln -s ~/.claude/team-commands/commands/* ~/.claude/commands/
```

### 版本管理

```bash
# 更新共享命令
cd .claude/team-commands
git pull origin main

# 或使用 submodule
git submodule update --remote
```

---

## 命令维护

### 命令版本控制

在命令文件中添加版本注释：

```markdown
<!--
Command: code-review
Version: 2.1.0
Author: Team
Updated: 2025-12-27
-->

请审查以下代码：

$ARGUMENTS
...
```

### 命令文档

创建命令说明文档：

```markdown
# 团队自定义命令说明

## 代码审查

### /review:code
审查代码质量

**参数**: 文件路径或目录

**示例**:
```
/review:code src/utils/parser.ts
/review:code src/
```

### /review:security
安全审查

**参数**: 文件路径

**示例**:
```
/review:security src/auth/login.ts
```

...
```

### 命令测试

创建命令测试用例：

```bash
#!/bin/bash
# test-commands.sh

echo "Testing /review command..."
claude -p "/review example.js" > /dev/null
if [ $? -eq 0 ]; then
    echo "✓ /review works"
else
    echo "✗ /review failed"
fi

echo "Testing /gen:test command..."
claude -p "/gen:test example.js" > /dev/null
if [ $? -eq 0 ]; then
    echo "✓ /gen:test works"
else
    echo "✗ /gen:test failed"
fi
```

---

## 高级模式

### 命令链

创建调用其他命令的元命令：

```markdown
<!-- full-review.md -->
执行完整的代码审查流程：

$ARGUMENTS

步骤：
1. 首先进行代码质量审查（类似 /review:code）
2. 然后进行安全审查（类似 /review:security）
3. 最后进行性能审查（类似 /review:performance）

为每个阶段生成独立报告，最后给出综合评分。
```

### 动态命令

根据文件类型采取不同行动：

```markdown
智能分析以下目标：

$ARGUMENTS

根据目标类型：
- 如果是 .ts/.js 文件：进行 JavaScript/TypeScript 代码审查
- 如果是 .py 文件：进行 Python 代码审查
- 如果是 .go 文件：进行 Go 代码审查
- 如果是目录：分析目录结构和主要文件
- 如果是 package.json：分析依赖和脚本
- 如果是配置文件：检查配置正确性

提供针对性的分析报告。
```

### 交互式命令

引导 Claude 提问：

```markdown
帮我创建一个新的 API 端点。

首先，请询问以下信息：
1. 端点路径（例如 /api/users）
2. HTTP 方法（GET/POST/PUT/DELETE）
3. 需要的参数
4. 认证要求

根据回答生成：
1. 路由定义
2. 控制器代码
3. 验证逻辑
4. 测试用例

初始信息：$ARGUMENTS
```

---

## 实验练习

### 实验 3.3.1：创建团队命令库

```bash
# 创建命令库结构
mkdir -p team-commands/{review,gen,docs}

# 创建审查命令
cat > team-commands/review/pr.md << 'EOF'
审查 Pull Request：$ARGUMENTS

检查项：
1. 代码变更是否符合需求
2. 是否有测试覆盖
3. 文档是否更新
4. 是否有破坏性变更

使用以下格式输出：
## PR 审查报告
### 概述
### 发现的问题
### 建议
### 结论（批准/需要修改）
EOF

# 链接到项目
ln -s $(pwd)/team-commands .claude/commands/team
```

### 实验 3.3.2：创建智能命令

```bash
cat > .claude/commands/smart-analyze.md << 'EOF'
智能分析目标：$ARGUMENTS

执行流程：
1. 确定目标类型（文件/目录/URL/代码片段）
2. 根据类型选择分析策略
3. 执行分析
4. 生成报告

报告格式：
## 分析报告
**目标**: [目标]
**类型**: [类型]

### 分析结果
[详细结果]

### 建议
[改进建议]
EOF

# 测试
claude
> /smart-analyze src/utils
> /smart-analyze https://github.com/user/repo
> /smart-analyze "function add(a, b) { return a + b; }"
```

---

## 第三章总结

| 主题 | 要点 |
|------|------|
| 内置命令 | /help, /doctor, /cost, /compact, /init |
| 自定义命令 | `.claude/commands/` 目录，Markdown 格式 |
| 参数传递 | 使用 `$ARGUMENTS` 占位符 |
| 命令组织 | 子目录创建命名空间 |
| 团队共享 | Git 仓库 + 符号链接 |
| 最佳实践 | 单一职责、清晰指令、结构化输出 |

---

## 下一章

[第四章：Hooks 钩子系统](../04-hooks/README.md) - 学习使用钩子自动化工作流
