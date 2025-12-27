# 6.3 其他编辑器

Claude Code 可以通过终端和插件在多种编辑器中使用。

---

## Vim / Neovim

### 终端集成

最简单的方式是使用 Vim 的终端功能：

```vim
" 在 Vim 中打开终端运行 Claude
:terminal claude

" 或者使用垂直分割
:vertical terminal claude
```

### Neovim 配置

使用 `vim.fn.system` 或终端缓冲区：

```lua
-- init.lua

-- 快捷键打开 Claude
vim.keymap.set('n', '<leader>cc', function()
  vim.cmd('vsplit | terminal claude')
end, { desc = 'Open Claude Code' })

-- 发送选中文本到 Claude
vim.keymap.set('v', '<leader>cs', function()
  -- 获取选中文本
  local lines = vim.fn.getregion(vim.fn.getpos('v'), vim.fn.getpos('.'))
  local text = table.concat(lines, '\n')

  -- 发送到 Claude
  local result = vim.fn.system('claude -p "解释这段代码:\n' .. text .. '"')
  print(result)
end, { desc = 'Send to Claude' })
```

### 使用 vim-floaterm

```vim
" 安装 vim-floaterm
Plug 'voldikss/vim-floaterm'

" 配置快捷键
nnoremap <leader>cc :FloatermNew claude<CR>

" 自定义浮动终端
let g:floaterm_width = 0.8
let g:floaterm_height = 0.8
```

### 使用 toggleterm.nvim（Neovim）

```lua
-- 使用 lazy.nvim 安装
{
  "akinsho/toggleterm.nvim",
  config = function()
    require("toggleterm").setup({
      size = 20,
      open_mapping = [[<c-\>]],
    })

    -- Claude 专用终端
    local Terminal = require('toggleterm.terminal').Terminal
    local claude = Terminal:new({
      cmd = "claude",
      direction = "float",
      float_opts = {
        border = "curved",
      },
    })

    vim.keymap.set("n", "<leader>cc", function()
      claude:toggle()
    end, { desc = "Toggle Claude" })
  end
}
```

---

## Emacs

### Shell 模式

```elisp
;; 在 shell 中运行 Claude
(defun run-claude ()
  "Run Claude Code in a shell buffer."
  (interactive)
  (let ((buffer (get-buffer-create "*claude*")))
    (with-current-buffer buffer
      (shell-mode)
      (comint-exec buffer "claude" "claude" nil nil))
    (switch-to-buffer-other-window buffer)))

(global-set-key (kbd "C-c c c") 'run-claude)
```

### 发送代码到 Claude

```elisp
(defun send-region-to-claude (start end)
  "Send the selected region to Claude for explanation."
  (interactive "r")
  (let ((code (buffer-substring-no-properties start end)))
    (shell-command
     (format "claude -p '解释这段代码:\n%s'" code))))

(global-set-key (kbd "C-c c s") 'send-region-to-claude)
```

### 使用 vterm

```elisp
;; 使用 vterm 获得更好的终端体验
(use-package vterm
  :ensure t
  :config
  (defun run-claude-vterm ()
    "Run Claude in vterm."
    (interactive)
    (let ((vterm-shell "claude"))
      (vterm "*claude-vterm*")))

  (global-set-key (kbd "C-c c v") 'run-claude-vterm))
```

---

## Zed

Zed 编辑器有内置的 AI 支持。

### 配置 Claude

编辑 `~/.config/zed/settings.json`：

```json
{
  "assistant": {
    "default_model": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514"
    }
  },
  "language_models": {
    "anthropic": {
      "api_key": "sk-ant-xxxx"
    }
  }
}
```

### 使用方法

```
1. 按 Cmd/Ctrl + Enter 打开 AI 面板
2. 选中代码后按 Cmd/Ctrl + Enter 询问
3. 使用 /edit 命令直接编辑代码
```

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd + Enter` | 打开/发送到 AI 面板 |
| `Cmd + Shift + Enter` | 内联 AI 编辑 |

---

## Sublime Text

### 使用终端插件

安装 Terminus 插件：

1. 打开命令面板（`Cmd/Ctrl + Shift + P`）
2. 输入 "Install Package"
3. 搜索 "Terminus"

配置快捷键：

```json
// Preferences → Key Bindings
[
  {
    "keys": ["ctrl+alt+c"],
    "command": "terminus_open",
    "args": {
      "cmd": ["claude"],
      "title": "Claude Code"
    }
  }
]
```

### 创建构建系统

```json
// Tools → Build System → New Build System
{
  "cmd": ["claude", "-p", "$selection"],
  "selector": "source",
  "shell": true
}
```

---

## 通用终端工作流

无论使用哪个编辑器，都可以使用这些通用技巧：

### 分屏工作流

```bash
# 使用 tmux 分屏
tmux new-session -s dev

# 分割窗口
Ctrl+b %  # 垂直分割
Ctrl+b "  # 水平分割

# 一边是编辑器，一边是 Claude
# 左边窗格运行 vim
# 右边窗格运行 claude
```

### 使用 tmux 脚本

```bash
#!/bin/bash
# dev-session.sh

tmux new-session -d -s dev

# 主窗格：编辑器
tmux send-keys -t dev 'nvim .' C-m

# 右侧窗格：Claude
tmux split-window -h -t dev
tmux send-keys -t dev 'claude' C-m

# 下方窗格：终端
tmux split-window -v -t dev

tmux attach -t dev
```

### 管道工作流

```bash
# 从文件发送到 Claude
cat myfile.py | claude -p "审查这段代码"

# 从 clipboard 发送
pbpaste | claude -p "解释这段代码"  # macOS
xclip -selection clipboard -o | claude -p "解释这段代码"  # Linux

# 结果输出到文件
claude -p "生成一个 Python 函数计算斐波那契" > fib.py
```

---

## 实验练习

### 实验 6.3.1：Vim 终端集成

```vim
" 在 .vimrc 中添加
nnoremap <leader>cc :vertical terminal claude<CR>

" 测试
" 1. 打开 Vim
" 2. 按 <leader>cc
" 3. 在右侧终端中使用 Claude
```

### 实验 6.3.2：tmux 工作流

```bash
# 创建开发会话脚本
cat > ~/dev-session.sh << 'EOF'
#!/bin/bash
tmux new-session -d -s dev
tmux send-keys -t dev 'vim .' C-m
tmux split-window -h -t dev
tmux send-keys -t dev 'claude' C-m
tmux attach -t dev
EOF

chmod +x ~/dev-session.sh

# 运行
~/dev-session.sh
```

### 实验 6.3.3：管道使用

```bash
# 创建测试文件
cat > test.py << 'EOF'
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
    return arr
EOF

# 发送到 Claude
cat test.py | claude -p "优化这个排序算法"
```

---

## 第六章总结

| 编辑器 | 集成方式 | 推荐度 |
|--------|---------|--------|
| VS Code | 官方扩展 | ⭐⭐⭐⭐⭐ |
| JetBrains | 官方插件 | ⭐⭐⭐⭐ |
| Vim/Neovim | 终端 + 配置 | ⭐⭐⭐⭐ |
| Emacs | Shell + 配置 | ⭐⭐⭐ |
| Zed | 内置支持 | ⭐⭐⭐⭐⭐ |
| Sublime Text | Terminus 插件 | ⭐⭐⭐ |

---

## 下一章

[第七章：Agent SDK 开发](../07-agent-sdk/README.md) - 学习使用 Claude Agent SDK 构建自定义 Agent
