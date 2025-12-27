# 3.2 自定义命令开发

自定义命令让你能够创建可复用的提示模板，大大提高工作效率。

---

## 命令文件位置

自定义命令存放在 `.claude/commands/` 目录：

```
项目目录/
└── .claude/
    └── commands/
        ├── review.md       → /review
        ├── test.md         → /test
        ├── docs.md         → /docs
        └── subdir/
            └── special.md  → /subdir:special
```

### 目录层级

- **项目级别**: `项目目录/.claude/commands/`（仅当前项目可用）
- **用户级别**: `~/.claude/commands/`（所有项目可用）

---

## 命令文件格式

命令文件是普通的 Markdown 文件，内容就是发送给 Claude 的提示。

### 基本示例

创建 `.claude/commands/review.md`：

```markdown
请审查当前工作目录中最近修改的代码文件。

审查要点：
1. 代码逻辑是否正确
2. 是否有潜在的 bug
3. 代码风格是否一致
4. 是否有性能问题
5. 是否有安全隐患

请给出具体的改进建议。
```

使用：
```
> /review
```

### 带参数的命令

使用 `$ARGUMENTS` 占位符接收参数：

创建 `.claude/commands/explain.md`：

```markdown
请详细解释以下代码或概念：

$ARGUMENTS

解释应包括：
1. 基本原理
2. 工作机制
3. 使用场景
4. 示例代码
5. 注意事项
```

使用：
```
> /explain React 的 useCallback hook
# $ARGUMENTS 会被替换为 "React 的 useCallback hook"
```

---

## 实用命令示例

### 1. 代码审查命令

`.claude/commands/code-review.md`：

```markdown
请对以下文件进行代码审查：

$ARGUMENTS

审查清单：

## 代码质量
- [ ] 代码是否简洁易读
- [ ] 命名是否清晰
- [ ] 是否有重复代码
- [ ] 函数/方法是否过长

## 潜在问题
- [ ] 是否有明显的 bug
- [ ] 边界条件是否处理
- [ ] 错误处理是否完善
- [ ] 是否有内存泄漏风险

## 安全性
- [ ] 是否有注入风险
- [ ] 敏感数据是否处理得当
- [ ] 权限检查是否完善

## 性能
- [ ] 是否有 N+1 查询
- [ ] 是否有不必要的计算
- [ ] 缓存使用是否合理

请给出评分（1-10）和具体改进建议。
```

### 2. 测试生成命令

`.claude/commands/gen-test.md`：

```markdown
为以下代码生成单元测试：

$ARGUMENTS

要求：
1. 使用 Jest 作为测试框架
2. 覆盖所有公开方法
3. 包含正常情况和边界情况
4. 包含错误处理测试
5. 使用 describe/it 组织测试
6. 添加清晰的测试描述

请生成完整的测试文件。
```

### 3. 文档生成命令

`.claude/commands/gen-docs.md`：

```markdown
为以下代码生成文档：

$ARGUMENTS

文档应包括：
1. 模块/函数概述
2. 参数说明（类型、描述、是否必需）
3. 返回值说明
4. 使用示例
5. 注意事项

使用 JSDoc/TSDoc 格式。
```

### 4. 重构建议命令

`.claude/commands/refactor.md`：

```markdown
分析以下代码并提供重构建议：

$ARGUMENTS

重点关注：
1. 代码组织结构
2. 设计模式应用
3. 解耦和模块化
4. 可测试性提升
5. 性能优化

请提供：
- 当前问题分析
- 重构方案
- 重构后的代码示例
- 重构步骤
```

### 5. Bug 分析命令

`.claude/commands/debug.md`：

```markdown
帮我分析和修复以下问题：

$ARGUMENTS

请按以下步骤进行：
1. 分析问题描述，理解预期行为
2. 定位可能的原因
3. 查看相关代码
4. 提出修复方案
5. 给出修复代码
6. 建议如何避免类似问题
```

### 6. Git 提交命令

`.claude/commands/commit.md`：

```markdown
查看当前的 git 变更并生成提交信息。

要求：
1. 运行 git status 和 git diff
2. 分析变更内容
3. 生成符合 Conventional Commits 规范的提交信息
4. 格式：<type>(<scope>): <description>

类型包括：
- feat: 新功能
- fix: bug 修复
- docs: 文档变更
- style: 代码格式
- refactor: 重构
- test: 测试相关
- chore: 构建/工具

$ARGUMENTS
```

### 7. API 设计命令

`.claude/commands/api-design.md`：

```markdown
设计 RESTful API 接口：

$ARGUMENTS

请提供：
1. 端点 URL 设计
2. HTTP 方法
3. 请求参数/Body 结构
4. 响应格式
5. 错误处理
6. 认证要求
7. 示例请求和响应

遵循 RESTful 最佳实践和项目现有 API 风格。
```

---

## 高级技巧

### 组合使用多个命令

```
# 先审查，再生成测试
> /review src/utils/parser.ts
> /gen-test src/utils/parser.ts
```

### 命令目录组织

```
.claude/commands/
├── review/
│   ├── code.md        → /review:code
│   ├── security.md    → /review:security
│   └── performance.md → /review:performance
├── gen/
│   ├── test.md        → /gen:test
│   ├── docs.md        → /gen:docs
│   └── types.md       → /gen:types
└── project/
    ├── init.md        → /project:init
    └── setup.md       → /project:setup
```

### 条件逻辑

虽然命令文件是静态的，但可以让 Claude 根据情况处理：

```markdown
分析参数并执行相应操作：

$ARGUMENTS

规则：
- 如果参数是文件路径：读取并分析该文件
- 如果参数是目录路径：分析目录结构
- 如果参数是 URL：获取并分析网页内容
- 如果参数是代码片段：直接分析代码
- 如果没有参数：分析当前目录
```

---

## 实验练习

### 实验 3.2.1：创建第一个命令

```bash
# 创建命令目录
mkdir -p .claude/commands

# 创建简单命令
cat > .claude/commands/hello.md << 'EOF'
用友好的方式向用户打招呼，并介绍一下你的能力。
EOF

# 测试命令
claude
> /hello
```

### 实验 3.2.2：带参数的命令

```bash
# 创建带参数的命令
cat > .claude/commands/translate.md << 'EOF'
将以下内容翻译成中文（如果是中文则翻译成英文）：

$ARGUMENTS

要求：
1. 保持原意
2. 用自然流畅的语言
3. 适当解释专业术语
EOF

# 测试
claude
> /translate Hello, World!
> /translate 你好，世界！
```

### 实验 3.2.3：创建审查命令

```bash
# 创建代码审查命令
cat > .claude/commands/review.md << 'EOF'
请审查以下文件：

$ARGUMENTS

检查项：
1. 代码质量和可读性
2. 潜在的 bug
3. 安全问题
4. 性能问题
5. 测试覆盖

请给出具体建议。
EOF

# 创建测试文件
cat > example.js << 'EOF'
function fetchData(url) {
  return fetch(url)
    .then(res => res.json())
    .catch(err => console.log(err));
}
EOF

# 测试
claude
> /review example.js
```

### 实验 3.2.4：创建多层命令

```bash
# 创建命令子目录
mkdir -p .claude/commands/gen

# 创建多个生成命令
cat > .claude/commands/gen/component.md << 'EOF'
创建一个 React 组件：

组件名称：$ARGUMENTS

要求：
1. 使用 TypeScript
2. 使用函数组件和 Hooks
3. 包含 Props 类型定义
4. 包含基本样式
5. 添加 JSDoc 注释
EOF

cat > .claude/commands/gen/hook.md << 'EOF'
创建一个自定义 React Hook：

Hook 名称：$ARGUMENTS

要求：
1. 使用 TypeScript
2. 返回类型明确
3. 包含使用示例
4. 添加 JSDoc 注释
EOF

# 测试
claude
> /gen:component UserProfile
> /gen:hook useDebounce
```

---

## 命令调试

### 查看可用命令

```
> /help
# 列出所有命令，包括自定义命令
```

### 命令不生效？

检查：
1. 文件位置是否正确（`.claude/commands/`）
2. 文件扩展名是否为 `.md`
3. 文件名是否包含特殊字符
4. 文件编码是否为 UTF-8

### 查看命令内容

```bash
# 直接查看命令文件
cat .claude/commands/review.md
```

---

## 下一节

[3.3 命令最佳实践](./03-best-practices.md) - 学习命令设计和团队共享的最佳实践
