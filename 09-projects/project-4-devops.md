# 项目四：DevOps 助手

构建一个 DevOps 自动化助手，处理部署、监控和运维任务。

---

## 项目目标

- CI/CD 配置生成
- 部署脚本编写
- 监控告警分析
- 性能问题诊断

---

## 项目结构

```
devops-assistant/
├── .claude/
│   ├── settings.json
│   └── commands/
│       ├── deploy.md          # 部署命令
│       ├── ci-config.md       # CI 配置
│       ├── analyze-logs.md    # 日志分析
│       └── diagnose.md        # 问题诊断
├── scripts/
│   ├── deploy.sh
│   ├── health-check.sh
│   └── rollback.sh
└── templates/
    ├── github-actions.yml
    ├── dockerfile.template
    └── docker-compose.template.yml
```

---

## 步骤 1：配置 MCP Server

```json
// .claude/settings.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/var/log", "/etc"]
    }
  },
  "permissions": {
    "allow": [
      "Read",
      "Bash(docker:*)",
      "Bash(kubectl:*)",
      "Bash(git:*)",
      "Bash(npm:*)",
      "Bash(curl:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo rm:*)"
    ]
  }
}
```

---

## 步骤 2：CI/CD 配置命令

```markdown
<!-- .claude/commands/ci-config.md -->
生成 CI/CD 配置：$ARGUMENTS

## 分析项目

1. 检查项目类型（Node.js/Python/Go 等）
2. 识别构建流程
3. 确定测试命令
4. 分析部署需求

## 生成配置

### GitHub Actions

```yaml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup
        # 根据项目类型设置
      - name: Install
        run: npm ci
      - name: Test
        run: npm test
      - name: Build
        run: npm run build

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        # 部署步骤
```

根据项目实际情况调整配置。

$ARGUMENTS
```

---

## 步骤 3：部署命令

```markdown
<!-- .claude/commands/deploy.md -->
执行部署任务：$ARGUMENTS

## 部署流程

1. **预检查**
   - 检查 git 状态
   - 运行测试
   - 构建检查

2. **部署步骤**
   ```bash
   # 构建镜像
   docker build -t app:latest .

   # 推送镜像
   docker push registry/app:latest

   # 更新服务
   kubectl set image deployment/app app=registry/app:latest
   ```

3. **验证**
   - 健康检查
   - 回滚准备

4. **监控**
   - 检查日志
   - 监控指标

$ARGUMENTS
```

---

## 步骤 4：日志分析命令

```markdown
<!-- .claude/commands/analyze-logs.md -->
分析日志文件：$ARGUMENTS

## 分析任务

1. **读取日志**
   - 读取指定日志文件
   - 或读取最近的 N 行

2. **识别模式**
   - ERROR 级别日志
   - 异常堆栈
   - 警告信息
   - 性能问题

3. **统计分析**
   - 错误频率
   - 错误类型分布
   - 时间分布

4. **输出报告**

# 日志分析报告

## 概述
- 分析时间范围：[时间]
- 总日志条数：N
- 错误数：N
- 警告数：N

## 主要问题

### 1. [问题描述]
- 出现次数：N
- 首次出现：[时间]
- 相关日志：
  ```
  [日志内容]
  ```
- 可能原因：
- 建议解决方案：

## 趋势分析
[错误趋势图/描述]

## 建议
1. [建议1]
2. [建议2]

$ARGUMENTS
```

---

## 步骤 5：问题诊断命令

```markdown
<!-- .claude/commands/diagnose.md -->
诊断系统问题：$ARGUMENTS

## 诊断步骤

1. **收集信息**
   ```bash
   # 系统资源
   top -bn1 | head -20
   free -m
   df -h

   # 网络状态
   netstat -tlnp
   curl -I http://localhost:PORT

   # 进程状态
   ps aux | grep [process]

   # Docker 状态
   docker ps
   docker stats --no-stream
   ```

2. **分析症状**
   - CPU 使用率
   - 内存使用
   - 磁盘空间
   - 网络连接
   - 进程状态

3. **识别问题**
   - 资源瓶颈
   - 连接问题
   - 配置错误
   - 代码 bug

4. **提供解决方案**

$ARGUMENTS
```

---

## 步骤 6：部署脚本

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

APP_NAME=${1:-"myapp"}
VERSION=${2:-"latest"}
ENVIRONMENT=${3:-"staging"}

echo "=== Deploying $APP_NAME:$VERSION to $ENVIRONMENT ==="

# 预检查
echo "Running pre-deploy checks..."
npm test || { echo "Tests failed!"; exit 1; }
npm run build || { echo "Build failed!"; exit 1; }

# 构建镜像
echo "Building Docker image..."
docker build -t $APP_NAME:$VERSION .

# 推送镜像（如果需要）
if [ "$ENVIRONMENT" = "production" ]; then
    docker tag $APP_NAME:$VERSION registry.example.com/$APP_NAME:$VERSION
    docker push registry.example.com/$APP_NAME:$VERSION
fi

# 部署
echo "Deploying..."
if [ "$ENVIRONMENT" = "production" ]; then
    kubectl set image deployment/$APP_NAME $APP_NAME=registry.example.com/$APP_NAME:$VERSION
else
    docker-compose up -d
fi

# 健康检查
echo "Running health check..."
./scripts/health-check.sh

echo "=== Deployment complete ==="
```

```bash
#!/bin/bash
# scripts/health-check.sh

MAX_RETRIES=30
RETRY_INTERVAL=2
URL=${1:-"http://localhost:3000/health"}

echo "Checking $URL..."

for i in $(seq 1 $MAX_RETRIES); do
    if curl -sf "$URL" > /dev/null; then
        echo "✓ Health check passed"
        exit 0
    fi
    echo "Attempt $i/$MAX_RETRIES failed, retrying..."
    sleep $RETRY_INTERVAL
done

echo "✗ Health check failed after $MAX_RETRIES attempts"
exit 1
```

```bash
#!/bin/bash
# scripts/rollback.sh

APP_NAME=${1:-"myapp"}
PREVIOUS_VERSION=${2:-""}

if [ -z "$PREVIOUS_VERSION" ]; then
    # 获取上一个版本
    PREVIOUS_VERSION=$(kubectl rollout history deployment/$APP_NAME | tail -2 | head -1 | awk '{print $1}')
fi

echo "Rolling back $APP_NAME to revision $PREVIOUS_VERSION..."
kubectl rollout undo deployment/$APP_NAME --to-revision=$PREVIOUS_VERSION

echo "Rollback complete"
kubectl rollout status deployment/$APP_NAME
```

---

## 步骤 7：Dockerfile 模板

```dockerfile
# templates/dockerfile.template
# Node.js 应用 Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

EXPOSE 3000

USER node

CMD ["node", "dist/index.js"]
```

---

## 使用示例

```bash
# 生成 CI 配置
claude
> /ci-config 为这个 Node.js 项目生成 GitHub Actions 配置

# 部署应用
> /deploy 部署到 staging 环境

# 分析日志
> /analyze-logs /var/log/app/error.log

# 诊断问题
> /diagnose 应用响应变慢，CPU 使用率高
```

---

## 总结

本项目涵盖了：

- CI/CD 配置自动生成
- 部署脚本自动化
- 日志分析和问题诊断
- 健康检查和回滚机制

下一步：集成监控平台（Prometheus/Grafana）和告警系统。
