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

