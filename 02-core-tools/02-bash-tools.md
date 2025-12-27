# 2.2 代码执行工具

Bash 工具是 Claude Code 执行系统命令的核心能力，让 AI 能够运行构建、测试、Git 操作等各种命令。

---

## Bash 工具基础

### 工具能力

- 执行任意 shell 命令
- 持久化的 shell 会话
- 后台任务执行
- 超时控制
- 输出捕获

### 基本用法

```
> 运行 npm install

# Claude 使用 Bash 工具：
# Bash: npm install
# description: Install package dependencies
```

### 权限控制

Claude Code 默认会在执行敏感命令前请求确认：

```
> 删除 build 目录

# Claude 会请求确认：
# "Allow Bash: rm -rf build?" [y/n]
```

可以在 settings.json 中预配置权限：

```json
{
  "permissions": {
    "allow": [
      "Bash(npm:*)",
      "Bash(git:*)",
      "Bash(ls:*)"
    ],
    "deny": [
      "Bash(rm -rf /*:*)"
    ]
  }
}
```

---

## 命令执行技巧

### 链式命令

```bash
# 使用 && 连接依赖命令（前一个成功才执行后一个）
> 安装依赖并运行测试

# Bash: npm install && npm test

# 使用 ; 连接独立命令（无论成功与否都执行）
> 清理并重建项目

# Bash: rm -rf dist; npm run build
```

### 路径引用

```bash
# 使用绝对路径（推荐）
# Bash: npm test --prefix /Users/user/project

# 避免频繁 cd
# ❌ Bash: cd /path/to/project && npm test
# ✅ Bash: npm test --prefix /path/to/project
```

### 处理空格路径

```bash
# 使用双引号包裹
# Bash: cd "/Users/user/My Projects/app"

# 或使用转义
# Bash: cd /Users/user/My\ Projects/app
```

---

## 后台任务

### 启动后台任务

对于长时间运行的命令，可以在后台执行：

```
> 在后台启动开发服务器

# Bash: npm run dev
# run_in_background: true
# 返回: shell_id: "abc123"
```

### 检查后台任务输出

```
> 检查开发服务器的输出

# BashOutput: shell_id: "abc123"
# 返回最新输出
```

### 终止后台任务

```
> 停止开发服务器

# KillShell: shell_id: "abc123"
```

### 实际场景

```
# 场景1：运行测试同时做其他事情
> 在后台运行完整测试套件，然后继续帮我写代码

# Claude:
# 1. Bash(background): npm test
# 2. 继续对话和编码
# 3. 稍后检查测试结果

# 场景2：启动多个服务
> 同时启动前端和后端服务

# Claude:
# 1. Bash(background): npm run dev:frontend
# 2. Bash(background): npm run dev:backend
```

---

## 超时控制

### 默认超时

- 默认超时：120 秒（2 分钟）
- 最大超时：600 秒（10 分钟）

### 设置超时

```
> 运行可能很慢的构建命令

# Bash: npm run build:production
# timeout: 300000  # 5 分钟（毫秒）
```

### 处理超时

```
# 如果命令超时，Claude 会收到超时错误
# 然后可以：
# 1. 增加超时重试
# 2. 改用后台执行
# 3. 优化命令
```

---

## 常用命令场景

### 包管理

```bash
# npm
npm install
npm install package-name
npm uninstall package-name
npm update
npm run script-name

# yarn
yarn install
yarn add package-name
yarn remove package-name

# pnpm
pnpm install
pnpm add package-name
```

### Git 操作

```bash
# 状态查看
git status
git log --oneline -10
git diff

# 分支操作
git branch
git checkout -b feature/new-feature
git merge main

# 提交（Claude 会自动格式化提交信息）
git add .
git commit -m "message"

# 远程操作
git push origin branch-name
git pull origin main
```

### 构建与测试

```bash
# 构建
npm run build
npm run build:prod

# 测试
npm test
npm run test:coverage
npm run test:watch

# Lint
npm run lint
npm run lint:fix
```

### 系统命令

```bash
# 文件操作
mkdir -p directory/path
cp source dest
mv old new

# 进程管理
ps aux | grep node
kill -9 PID

# 网络
curl -X GET http://api.example.com
ping -c 3 google.com
```

---

## 安全注意事项

### Claude 自动规避的危险操作

1. **破坏性删除**：`rm -rf /` 等
2. **系统配置修改**：修改 `/etc` 下的文件
3. **凭证泄露**：输出包含密钥的命令
4. **无限循环**：可能导致系统挂起的命令

### 推荐的安全实践

```json
// settings.json
{
  "permissions": {
    "allow": [
      // 明确允许的安全命令
      "Bash(npm run:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(ls:*)",
      "Bash(cat:*)"
    ],
    "deny": [
      // 明确禁止的危险命令
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(chmod 777:*)"
    ]
  }
}
```

---

## 实验练习

### 实验 2.2.1：基本命令执行

```bash
# 创建测试项目
mkdir -p /tmp/bash-test
cd /tmp/bash-test
npm init -y
npm install lodash

# 在 Claude Code 中：
claude
> 查看当前目录结构
> 查看 package.json 的内容
> 安装 axios 依赖
> 列出所有已安装的 npm 包
```

### 实验 2.2.2：Git 操作

```bash
# 初始化 Git 仓库
cd /tmp/bash-test
git init
echo "# Test Project" > README.md
git add README.md
git commit -m "Initial commit"

# 在 Claude Code 中：
claude
> 查看 Git 状态
> 创建一个新分支 feature/test
> 切换到新分支
> 修改 README 添加一些内容
> 提交这些更改
```

### 实验 2.2.3：后台任务

```bash
# 创建一个简单的服务器脚本
cd /tmp/bash-test
cat > server.js << 'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
  console.log(`Request: ${req.url}`);
  res.writeHead(200);
  res.end('Hello World');
});
server.listen(3456, () => console.log('Server running on port 3456'));
EOF

# 在 Claude Code 中：
claude
> 在后台启动 server.js
> 检查服务器是否正常运行
> 使用 curl 测试服务器
> 检查服务器日志输出
> 停止服务器
```

### 实验 2.2.4：构建与测试流程

```bash
# 创建带测试的项目
cd /tmp/bash-test
npm install --save-dev jest

# 创建源代码
cat > sum.js << 'EOF'
function sum(a, b) {
  return a + b;
}
module.exports = sum;
EOF

# 创建测试
cat > sum.test.js << 'EOF'
const sum = require('./sum');
test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
EOF

# 添加测试脚本到 package.json
npm pkg set scripts.test="jest"

# 在 Claude Code 中：
claude
> 运行测试并查看结果
> 添加一个新的测试用例：测试 sum(0, 0) 等于 0
> 再次运行测试
```

---

## 与其他工具的配合

### Bash + Read/Edit 工作流

```
> 找出代码中的语法错误并修复

# Claude 可能会：
# 1. Bash: npm run lint  # 发现错误
# 2. Read: 读取有错误的文件
# 3. Edit: 修复错误
# 4. Bash: npm run lint  # 验证修复
```

### Bash + Grep 工作流

```
> 找出所有 TODO 并创建一个任务列表文件

# Claude 可能会：
# 1. Grep: TODO  # 找出所有 TODO
# 2. Write: 创建 tasks.md
# 3. Bash: cat tasks.md  # 验证结果
```

---

## 故障排除

### Q: 命令执行失败，提示 "command not found"

```bash
# 检查命令是否在 PATH 中
which node
echo $PATH

# 使用完整路径
/usr/local/bin/node script.js
```

### Q: 权限被拒绝

```bash
# 检查权限配置
# 在 settings.json 的 permissions.allow 中添加命令

# 或者使用 --dangerously-skip-permissions（不推荐）
```

### Q: 命令超时

```
# 增加超时时间
# 或使用后台执行
> 在后台运行这个命令，然后告诉我进度
```

---

## 最佳实践

1. **使用描述性的命令说明**：让 Claude 理解命令的目的
2. **链式执行相关命令**：减少来回确认
3. **对长时间运行的命令使用后台模式**
4. **在 settings.json 中预配置常用命令权限**
5. **避免在代码目录外执行命令**

---

## 下一节

[2.3 Web 工具](./03-web-tools.md) - 学习使用 WebSearch 和 WebFetch 获取网络信息
