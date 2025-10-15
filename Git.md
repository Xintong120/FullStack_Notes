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

### 2. 远程仓库冲突问题

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

