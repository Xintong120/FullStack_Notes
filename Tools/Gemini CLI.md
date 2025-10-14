#  Gemini CLI 使用

> https://github.com/google-gemini/gemini-cli

## 1.安装

1. 前提条件： 确保您已安装 Node.js 版本 20 或更高版本。

2. 推荐使用以下方式安装：

   > npm install -g @google/gemini-cligemini
   >
   > npx https://github.com/google-gemini/gemini-cli

## 2.command

## 命令类型概览

Gemini CLI 支持三种前缀的命令：

- **斜杠命令 (`/`)** - 管理 CLI 本身的行为和会话
- **At 命令 (`@`)** - 在提示中包含文件或目录内容
- **感叹号命令 (`!`)** - 与系统 shell 交互

------

## 🔧 斜杠命令 (`/`)

### 会话管理

- **`/chat`** - 保存和恢复对话历史
  - `save <tag>` - 保存当前对话状态
  - `resume <tag>` - 恢复先前对话
  - `list` - 列出可用标签
  - `delete <tag>` - 删除保存的对话
  - `share [file]` - 导出对话到 Markdown/JSON 文件
- **`/clear`** - 清除终端屏幕（快捷键：**Ctrl+L**）
- **`/compress`** - 用摘要替换整个聊天上下文以节省 token

### 内容操作

- **`/copy`** - 复制最后一个输出到剪贴板
  - *依赖系统剪贴板工具：Linux (`xclip`/`xsel`), macOS (`pbcopy`), Windows (`clip`)*
- **`/directory`** (`/dir`) - 管理工作区目录
  - `add <path1>,<path2>` - 添加目录到工作区
  - `show` - 显示所有已添加目录

### 开发工具

- **`/editor`** - 打开编辑器选择对话框
- **`/vim`** - 切换 vim 编辑模式
  - 支持 NORMAL/INSERT 模式、vim 风格导航和编辑命令
  - 偏好设置保存在 `~/.gemini/settings.json`
- **`/restore`** - 将项目文件恢复到工具执行前的状态
  - *需要启用 `--checkpointing` 选项*

### 配置和信息

- **`/help`** (`/?`) - 显示帮助信息
- **`/settings`** - 打开设置编辑器
- **`/theme`** - 更改视觉主题
- **`/auth`** - 更改认证方法
- **`/about`** - 显示版本信息
- **`/privacy`** - 显示隐私声明和数据收集选项

### AI 和工具管理

- **`/mcp`** - 列出配置的 MCP 服务器和工具
  - `desc`/`descriptions` - 显示详细描述
  - `nodesc`/`nodescriptions` - 隐藏描述
  - `schema` - 显示工具参数的 JSON 模式
  - *快捷键：**Ctrl+T** 切换工具描述显示*
- **`/tools`** - 显示当前可用工具
  - `desc`/`descriptions` - 显示工具详细描述
  - `nodesc`/`nodescriptions` - 仅显示工具名称
- **`/memory`** - 管理 AI 的指令上下文（来自 `GEMINI.md` 文件）
  - `add <text>` - 添加文本到内存
  - `show` - 显示当前分层内存内容
  - `refresh` - 重新加载所有 `GEMINI.md` 文件
  - `list` - 列出使用的 `GEMINI.md` 文件路径

### 项目相关

- **`/init`** - 分析当前目录并生成定制的 `GEMINI.md` 文件
- **`/extensions`** - 列出所有活动的扩展
- **`/bug`** - 提交问题报告（可配置默认行为）

### 系统信息

- **`/stats`** - 显示会话统计信息（token 使用量、会话时长等）

### 退出命令

- **`/quit`** (`/exit`) - 退出 Gemini CLI

## At 命令 (`@`)

### 文件内容注入

- **`@<文件或目录路径>`** - 将指定文件或目录内容注入到当前提示中

  - *示例：*

    bash

    ```
    @src/main.py Explain this code
    @docs/ Summarize the documentation in this directory
    What is this about? @README.md
    ```

    

### 特性

- **Git 感知过滤** - 默认排除 git 忽略的文件
- **路径处理** - 支持绝对路径、相对路径和主目录引用
- **空格转义** - 使用反斜杠转义路径中的空格：`@My\ Documents/file.txt`
- **错误处理** - 路径无效时显示错误消息

### 特殊用法

- **`@`**（单独的 @ 符号）- 将 @ 符号作为普通字符传递给模型

------

## Shell 模式和透传命令 (`!`)

### 直接执行命令

- **`!<shell_command>`** - 直接执行系统命令

  - *示例：*

    bash

    ```
    !ls -la
    !git status
    !python --version
    ```

    

### Shell 模式切换

- **`!`**（单独使用）- 切换 shell 模式
  - **进入**：切换到系统 shell 环境，使用不同配色和指示器
  - **退出**：返回正常 Gemini CLI 行为

### 环境变量

- **`GEMINI_CLI=1`** - 在通过 `!` 执行的子进程中设置，用于检测是否在 Gemini CLI 内运行

### 安全注意

- 执行的命令具有与直接终端运行相同的权限
- 谨慎执行可能影响系统的命令

##  

## 输入提示快捷方式

- **撤销**：**Ctrl+Z** - 撤销输入提示中的上一个操作
- **重做**：**Ctrl+Shift+Z** - 重做上次撤销的操作

## 管理自定义提示词/记忆

### 1.添加规则

输入 **`update memory`** 指令添加

### 2.管理自定义提示词/记忆

使用 **`memory show` **命令查看所有规则

###  3.批量添加规则

如果觉得一条条加太麻烦，可以在当前的工作目录下，创建一个`.gemini`文件夹，然后，新建一个 `Gemini.md` 文件，把规则添加到文件中；再使用 **`memory refresh`** 命令刷新记忆文件；使用 **`memory show`** 指令查看当前的 memory 规则，检查配置是否生效。

