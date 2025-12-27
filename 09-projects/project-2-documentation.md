# 项目二：文档自动化

构建一个自动化文档生成和维护系统。

---

## 项目目标

- 自动生成代码文档
- 维护 README 文件
- 生成 API 文档
- 更新变更日志

---

## 项目结构

```
doc-automation/
├── .claude/
│   ├── settings.json
│   └── commands/
│       ├── gen-docs.md        # 生成文档
│       ├── update-readme.md   # 更新 README
│       ├── api-docs.md        # API 文档
│       └── changelog.md       # 变更日志
├── scripts/
│   └── doc-watcher.sh
└── templates/
    ├── readme.md
    ├── api.md
    └── component.md
```

---

## 步骤 1：创建文档生成命令

### 代码文档生成

```markdown
<!-- .claude/commands/gen-docs.md -->
为以下代码生成文档：$ARGUMENTS

## 执行步骤

1. **分析代码结构**
   - 读取目标文件/目录
   - 识别导出的函数、类、类型
   - 提取现有注释

2. **生成文档内容**
   对每个导出：
   - 函数签名
   - 参数说明（类型、描述、默认值）
   - 返回值说明
   - 使用示例
   - 注意事项

3. **输出格式**
   使用 JSDoc/TSDoc 格式添加文档注释。

## 示例输出

```typescript
/**
 * 计算两个数的和
 *
 * @param a - 第一个加数
 * @param b - 第二个加数
 * @returns 两数之和
 *
 * @example
 * ```ts
 * const result = add(1, 2);
 * console.log(result); // 3
 * ```
 */
export function add(a: number, b: number): number {
  return a + b;
}
```

请为目标代码生成完整的文档注释。
```

### README 更新命令

```markdown
<!-- .claude/commands/update-readme.md -->
更新项目 README.md：$ARGUMENTS

## 分析内容

1. **项目信息**
   - 读取 package.json
   - 分析项目结构
   - 识别主要功能

2. **README 结构**

# 项目名称

简短描述

## 特性
- 主要特性列表

## 安装
```bash
npm install xxx
```

## 快速开始
```typescript
// 使用示例
```

## API 文档
链接到详细文档

## 开发
```bash
npm run dev
npm test
```

## 贡献
贡献指南

## 许可证
MIT

3. **保持现有内容**
   - 不要删除用户自定义的部分
   - 只更新自动生成的部分

请根据当前项目状态更新 README。
```

### API 文档生成

```markdown
<!-- .claude/commands/api-docs.md -->
生成 API 文档：$ARGUMENTS

## 分析 API 端点

1. 扫描路由文件
2. 识别所有 API 端点
3. 提取请求/响应类型

## 文档格式

# API 文档

## 认证
[认证方式说明]

## 端点

### GET /api/users

获取用户列表

**请求参数**
| 参数 | 类型 | 必需 | 描述 |
|-----|------|-----|------|
| page | number | 否 | 页码 |
| limit | number | 否 | 每页数量 |

**响应**
```json
{
  "users": [...],
  "total": 100
}
```

**示例**
```bash
curl -X GET "https://api.example.com/api/users?page=1"
```

为项目生成完整的 API 文档。
```

### 变更日志命令

```markdown
<!-- .claude/commands/changelog.md -->
更新 CHANGELOG.md：$ARGUMENTS

## 分析变更

1. 读取最近的 git commits
2. 分类变更类型：
   - feat: 新功能
   - fix: 修复
   - docs: 文档
   - refactor: 重构
   - test: 测试
   - chore: 其他

## 格式

# Changelog

## [Unreleased]

### Added
- 新增功能

### Changed
- 变更

### Fixed
- 修复

### Removed
- 移除

## [1.0.0] - 2025-01-01

### Added
- 初始版本

根据 git 历史生成或更新 CHANGELOG。
```

---

## 步骤 2：配置 Hooks

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "src/.*\\.(ts|js)$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Consider running /gen-docs for $FILE_PATH' >&2"
          }
        ]
      }
    ]
  }
}
```

---

## 步骤 3：文档模板

### 组件文档模板

```markdown
<!-- templates/component.md -->
# {{componentName}}

{{description}}

## 属性

| 属性 | 类型 | 必需 | 默认值 | 描述 |
|-----|------|-----|-------|------|
{{#props}}
| {{name}} | `{{type}}` | {{required}} | {{default}} | {{description}} |
{{/props}}

## 使用示例

```tsx
import { {{componentName}} } from './{{componentName}}';

function App() {
  return (
    <{{componentName}}
      {{#exampleProps}}
      {{name}}={{{value}}}
      {{/exampleProps}}
    />
  );
}
```

## 注意事项

{{#notes}}
- {{.}}
{{/notes}}
```

---

## 步骤 4：使用示例

```bash
# 生成单个文件的文档
claude
> /gen-docs src/utils/helpers.ts

# 生成整个目录的文档
> /gen-docs src/components/

# 更新 README
> /update-readme

# 生成 API 文档
> /api-docs src/routes/

# 更新变更日志
> /changelog
```

---

## 步骤 5：自动化脚本

```bash
#!/bin/bash
# scripts/doc-watcher.sh

# 监控文件变化，自动更新文档

inotifywait -m -r -e modify,create ./src |
while read dir action file; do
    if [[ "$file" =~ \.(ts|js)$ ]]; then
        echo "File changed: $dir$file"
        claude -p "/gen-docs $dir$file" 2>/dev/null || true
    fi
done
```

---

## 扩展：TypeDoc 集成

```bash
# 安装 TypeDoc
npm install -D typedoc

# 配置 typedoc.json
cat > typedoc.json << 'EOF'
{
  "entryPoints": ["src/index.ts"],
  "out": "docs",
  "plugin": ["typedoc-plugin-markdown"]
}
EOF

# 创建集成命令
cat > .claude/commands/full-docs.md << 'EOF'
生成完整文档：

1. 运行 `npx typedoc` 生成 TypeDoc 文档
2. 更新 README
3. 生成 API 概览
4. 检查文档完整性

$ARGUMENTS
EOF
```

---

## 总结

本项目涵盖了：

- 多种文档生成命令
- 文档模板系统
- Hooks 提醒
- 文件监控自动化

下一步：集成文档部署流程，如 GitHub Pages。
