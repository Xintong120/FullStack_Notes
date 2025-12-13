## 通用Git工作流程

**初次上传项目到GitHub的标准步骤**：

1. 初始化仓库：`git init -b main`
2. 创建.gitignore：根据项目类型编辑
3. 添加文件：`git add .`
4. 提交更改：`git commit -m "Initial commit: 项目描述"`
5. 添加远程仓库：`git remote add origin <仓库URL>`
6. 推送代码：`git push -u origin main`

## 常用Git命令总结

- `git init`：初始化仓库
- `git add .`：添加所有文件到暂存区
- `git commit -m "消息"`：提交更改
- `git status`：查看仓库状态
- `git remote add origin <URL>`：添加远程仓库
- `git push -u origin main`：推送代码
- `git pull origin main --allow-unrelated-histories`：拉取并合并

## 最佳实践建议

1. 始终检查仓库状态：`git status`
2. 写有意义的提交信息
3. 定期提交，不要等所有工作完成
4. 配置.gitignore排除不需要的文件
5. 如果遇到问题，先查看错误信息和建议的解决方案

## Git网络错误解决方案总结

当浏览器可以正常访问GitHub，但Git命令失败时的解决方案：

#### 立即解决方案（按优先级）

##### 1. **调整Git缓冲区大小**
```bash
git config --global http.postBuffer 524288000
```
- **原因**: 默认缓冲区太小，可能导致大文件上传失败
- **效果**: 增加HTTP缓冲区到500MB

##### 2. **检查并重置Git配置**
```bash
git config --global http.sslVerify false  # 临时禁用SSL验证
git config --global core.preloadindex true  # 启用预加载
git config --global core.fscache true      # 启用文件系统缓存
```

##### 3. **DNS和网络问题**
```bash
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
```
- 使用GitHub镜像加速访问

##### 4. **代理设置**
```bash
git config --global http.proxy http://proxy.company.com:8080
git config --global https.proxy https://proxy.company.com:8080
```

#### 完整诊断流程

##### 步骤1: 检查基础配置
```bash
git config --list | grep -E "(url|proxy|ssl)"
```

##### 步骤2: 测试网络连接
```bash
curl -I https://github.com
telnet github.com 443  # 检查端口连通性
```

##### 步骤3: 重置网络相关设置
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global --unset http.sslcainfo
```

#### 常见错误及解决方案

| 错误信息                                   | 可能原因      | 解决方案              |
| ------------------------------------------ | ------------- | --------------------- |
| `Connection was reset`                     | 网络不稳定    | 增加缓冲区、切换网络  |
| `Could not resolve host`                   | DNS问题       | 检查网络、使用镜像    |
| `Failed to connect to github.com port 443` | SSL/HTTPS问题 | 禁用SSL验证或更新证书 |
| `SSL certificate problem`                  | 证书问题      | 更新证书或禁用验证    |

#### 最佳实践

##### 1. **网络环境检查**
- 确保网络连接稳定
- 检查防火墙设置
- 确认代理配置正确

##### 2. **Git配置优化**
```bash
# 推荐的基础配置
git config --global core.autocrlf true
git config --global core.fscache true
git config --global credential.helper manager
```

##### 3. **备用方案**
- 使用SSH替代HTTPS
- 配置多个远程仓库
- 离线提交，待网络恢复后推送

#### 快速修复脚本

创建 `git-fix-network.sh`：
```bash
#!/bin/bash
echo "修复Git网络配置..."

# 1. 增加缓冲区
git config --global http.postBuffer 524288000

# 2. 启用缓存
git config --global core.preloadindex true
git config --global core.fscache true

# 3. 禁用SSL验证（仅限测试环境）
git config --global http.sslVerify false

echo "Git网络配置修复完成！"
```

----------------------------------------------------------------------------------------------------

## 常见问题

### 1. 分支名称问题：`master` vs `main`

**问题描述**：尝试推送到 `origin master`，但GitHub使用 `main` 作为默认分支名称。

**错误信息**：`src refspec main does not match any`

**解决方案**：
```bash
# 将本地分支从 master 重命名为 main
git branch -m master main

# 然后推送
git push -u origin main
```

**学习点**：
- GitHub在2020年后将默认分支从 `master` 改为 `main`
- 建议在初始化仓库时就使用 `main` 分支：`git init -b main`

### 2. Git敏感信息处理指南

**场景1：文件还未推送（仍在本地）**

**检查敏感文件是否被跟踪：**
```bash
git status
git ls-files | grep "敏感文件名"
```

**从Git跟踪中移除文件（保留本地文件）：**
```bash
git rm --cached 文件名          # 单个文件
git rm --cached -r 目录名       # 整个目录
```

**更新 .gitignore 并提交：**
```bash
echo "文件名" >> .gitignore
git add .gitignore
git commit -m "remove sensitive files from tracking"
```

**场景2：文件已被推送（需要从历史中移除）**

**方法1：撤销最后几个提交（推荐用于最近的提交）**
```bash
git log --oneline -5                    # 查看最近5个提交
git reset --soft HEAD~N                # 撤销最后N个提交，但保留更改
git rm --cached 敏感文件名               # 取消跟踪敏感文件
git commit -m "remove sensitive files"  # 重新提交干净的版本
git push origin 分支名 --force-with-lease  # 强制推送
```

**方法2：使用 git filter-branch 重写历史（用于深层历史）**
```bash
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 敏感文件名' --prune-empty --tag-name-filter cat -- --all
git push origin --force --all
git push origin --force --tags
```

**方法3：完全重写历史（最彻底）**
```bash
git checkout --orphan 新分支名
git add -A
git commit -m "Initial commit"
git branch -D main                    # 删除旧分支
git branch -m main                    # 重命名新分支为main
git push -f origin main              # 强制推送新历史
```

**场景3：GitHub推送保护触发**

**临时允许推送（不推荐）：**
- 访问 GitHub 提供的链接，允许推送该次敏感信息

**正确处理：**
- 撤销提交，移除敏感文件
- 更新 .gitignore
- 重新提交并推送

#### 预防措施

**1. 配置 .gitignore**
```bash
# 添加常见敏感文件类型
.env
.env.local
*.key
*.pem
secrets.json
config/local.json
```

**2. 使用环境变量**
- 不要在代码中硬编码敏感信息
- 使用环境变量或配置文件引用

**3. 定期检查**
```bash
git ls-files | grep -E "\.(env|key|pem|secret)"  # 检查是否有敏感文件被跟踪
```

**4. 使用 Git 钩子**
```bash
# .git/hooks/pre-commit
#!/bin/sh
# 检查是否包含敏感文件
if git diff --cached --name-only | grep -q "敏感文件"; then
    echo "警告：检测到敏感文件！"
    exit 1
fi
```

#### 关键原则

1. **宁可重写历史，也不要让敏感信息留在仓库中**
2. **使用 --force-with-lease 而不是 --force**（更安全）
3. **备份重要数据**（重写历史前）
4. **通知协作者**（如果有团队协作）
5. **定期审计仓库**（检查是否有意外提交的敏感信息）

### 3. 远程仓库冲突问题

**问题描述**：推送被拒绝，远程仓库包含本地没有的提交。

**错误信息**：`[rejected] main -> main (fetch first)`

**解决方案**：
```bash
# 拉取远程仓库内容并允许不相关历史合并
git pull origin main --allow-unrelated-histories

# 然后推送
git push origin main
```

**学习点**：
- 当本地和远程仓库都有独立的历史时，需要使用 `--allow-unrelated-histories`
- `git pull` 实际上是 `git fetch` + `git merge` 的组合

### 3. 行结束符警告

**问题描述**：警告 `LF will be replaced by CRLF the next time Git touches it`

**解决方案**：这种警告通常不需要特殊处理，Git会自动处理。但如果需要配置：
```bash
# Windows推荐
git config core.autocrlf true

# Linux/Mac推荐
git config core.autocrlf input
```

**学习点**：
- Windows使用CRLF (`\r\n`)，Linux/Mac使用LF (`\n`)
- Git可以自动转换，但可能产生警告
### 4. Git子模块问题

**问题描述**：`frontend` 文件夹显示为子模块，无法正常提交和在GitHub上浏览。

**问题现象**：
- GitHub上显示文件夹有右箭头图标
- `git commit` 失败，提示子模块有未跟踪内容
- `git status` 显示 "modified: frontend (modified content, untracked content)"

**发现问题的命令**：
```bash
git ls-files --stage
```

**输出示例**：
```
160000 5bff57d4e10a86ad9c6677da1c135846b58350ba 0	frontend
100644 38b70bdd109e4d52614b44ea08b6d86155274125 0	.env.example
```

**文件模式含义**：
- `100644`：普通可读写文件
- `100755`：可执行文件
- `120000`：符号链接
- `160000`：**Git子模块**（特殊模式，表示指向另一个仓库的链接）
- `040000`：子目录

**解决方案**：

1. **取消子模块初始化**：
   ```bash
   git submodule deinit -f frontend
   ```

2. **移除子模块映射从索引**：
   ```bash
   git rm --cached frontend
   ```

3. **删除隐藏的.git目录**（如果存在）：
   ```bash
   rmdir /s /q frontend\.git  # Windows
   # 或 rm -rf frontend/.git  # Linux/Mac
   ```

4. **重新添加为普通文件夹**：
   ```bash
   git add frontend/
   ```

5. **提交更改**：
   ```bash
   git commit -m "Convert submodule to regular directory"
   ```

6. **推送到GitHub**：
   ```bash
   git push origin main
   ```

**提交输出示例**：
```
delete mode 160000 frontend
create mode 100644 frontend/... (119 files)
```

**学习点**：
- Git子模块使用特殊的 `160000` 模式存储指向外部仓库的引用
- GitHub识别此模式并显示子模块图标
- 转换为普通文件后，GitHub会正常显示文件夹内容
- `git ls-files --stage` 是诊断Git索引问题的强大工具

**预防措施**：
- 避免意外添加包含 `.git` 的目录
- 使用 `git add .` 而不是 `git add *` 以避免意外包含隐藏目录
- 定期检查 `git ls-files --stage` 输出以发现异常模式





