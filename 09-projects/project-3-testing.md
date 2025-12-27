# 项目三：测试自动化

构建一个智能测试生成和管理系统。

---

## 项目目标

- 自动生成单元测试
- 分析测试覆盖率
- 运行回归测试
- 生成测试报告

---

## 项目结构

```
test-automation/
├── .claude/
│   ├── settings.json
│   └── commands/
│       ├── gen-test.md        # 生成测试
│       ├── coverage.md        # 覆盖率分析
│       └── test-report.md     # 测试报告
├── scripts/
│   ├── run-tests.sh
│   └── coverage-check.sh
└── jest.config.js
```

---

## 步骤 1：测试生成命令

```markdown
<!-- .claude/commands/gen-test.md -->
为以下代码生成单元测试：$ARGUMENTS

## 测试生成规则

1. **分析源代码**
   - 读取目标文件
   - 识别所有导出的函数/类
   - 分析函数签名和逻辑

2. **测试用例设计**
   为每个函数生成：
   - 正常情况测试
   - 边界条件测试
   - 错误处理测试
   - 类型检查测试

3. **测试框架**
   使用 Jest + TypeScript

4. **输出格式**

```typescript
import { functionName } from '../source-file';

describe('functionName', () => {
  describe('正常情况', () => {
    it('should return expected result for valid input', () => {
      expect(functionName(validInput)).toBe(expectedOutput);
    });
  });

  describe('边界条件', () => {
    it('should handle empty input', () => {
      expect(functionName('')).toBe(defaultValue);
    });

    it('should handle null input', () => {
      expect(() => functionName(null)).toThrow();
    });
  });

  describe('错误处理', () => {
    it('should throw error for invalid input', () => {
      expect(() => functionName(invalidInput)).toThrow(ExpectedError);
    });
  });
});
```

请生成完整的测试文件。
```

---

## 步骤 2：配置自动测试

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": "src/.*\\.ts$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --findRelatedTests \"$FILE_PATH\" --passWithNoTests 2>&1 | tail -10 || true"
          }
        ]
      },
      {
        "matcher": {
          "toolName": "Write",
          "inputMatches": ".*\\.test\\.ts$"
        },
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- \"$FILE_PATH\" 2>&1 | tail -20"
          }
        ]
      }
    ]
  }
}
```

---

## 步骤 3：覆盖率分析命令

```markdown
<!-- .claude/commands/coverage.md -->
分析测试覆盖率：$ARGUMENTS

## 执行步骤

1. 运行 `npm test -- --coverage`
2. 读取覆盖率报告
3. 分析未覆盖的代码
4. 提供改进建议

## 输出格式

# 测试覆盖率报告

## 总体覆盖率
| 指标 | 覆盖率 | 状态 |
|-----|-------|------|
| 语句 | 85% | ✓ |
| 分支 | 72% | ⚠ |
| 函数 | 90% | ✓ |
| 行数 | 84% | ✓ |

## 未覆盖的代码

### src/utils/parser.ts
- 第 45-52 行：错误处理分支未覆盖
- 第 78 行：边界条件未测试

### 建议的测试用例
```typescript
it('should handle parse error', () => {
  // 添加错误处理测试
});
```

请分析当前项目的测试覆盖率。
```

---

## 步骤 4：测试脚本

```bash
#!/bin/bash
# scripts/run-tests.sh

echo "=== Running Tests ==="

# 运行测试
npm test -- --coverage --json --outputFile=test-results.json 2>&1

# 检查结果
if [ $? -eq 0 ]; then
    echo "✓ All tests passed"

    # 检查覆盖率阈值
    coverage=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
    if (( $(echo "$coverage < 80" | bc -l) )); then
        echo "⚠ Coverage below 80%: $coverage%"
        exit 1
    fi
else
    echo "✗ Tests failed"
    exit 1
fi
```

```bash
#!/bin/bash
# scripts/coverage-check.sh

# 生成覆盖率报告
npm test -- --coverage --silent

# 使用 Claude 分析
claude -p "分析 coverage/lcov-report/index.html 中的覆盖率数据，找出需要添加测试的地方"
```

---

## 步骤 5：Jest 配置

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts'
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  coverageReporters: ['text', 'lcov', 'json-summary']
};
```

---

## 步骤 6：使用示例

```bash
# 为单个文件生成测试
claude
> /gen-test src/utils/calculator.ts

# 为整个目录生成测试
> /gen-test src/services/

# 分析覆盖率
> /coverage

# 生成测试报告
> 运行所有测试并生成报告
```

---

## 扩展：测试驱动开发 (TDD) 工作流

```markdown
<!-- .claude/commands/tdd.md -->
使用 TDD 方式实现功能：$ARGUMENTS

## TDD 流程

1. **Red**：先写失败的测试
   - 理解需求
   - 编写测试用例
   - 确认测试失败

2. **Green**：写最小代码通过测试
   - 实现功能
   - 运行测试确认通过

3. **Refactor**：重构代码
   - 优化实现
   - 确保测试仍然通过

## 执行

请按照 TDD 流程实现以下功能：
$ARGUMENTS

步骤：
1. 先生成测试文件
2. 运行测试（应该失败）
3. 实现功能代码
4. 运行测试（应该通过）
5. 重构（如需要）
```

---

## 总结

本项目涵盖了：

- 自动测试生成
- 覆盖率分析
- Hook 自动运行测试
- TDD 工作流支持

下一步：集成 E2E 测试和性能测试。
