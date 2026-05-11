---
tags:
  - git
  - 分支管理
  - 版本控制
created: 2026-05-06
---

# Git 分支管理

## 1. 分支查看与切换

```bash
# 查看所有分支（当前分支前有 * 标记）
git branch

# 查看当前分支名
git branch --show-current

# 切换到指定分支
git switch feat        # 切换到 feat 分支
git switch main        # 切回 main 分支

# 查看所有分支的提交历史（图形化）
git log --oneline --decorate --graph --all
```

## 2. 提交工作流

### 2.1 查看状态与差异

```bash
# 查看工作区状态（简短格式）
git status --short
# 或完整格式
git status

# 查看具体修改内容
git diff                # 工作区 vs 暂存区
git diff --stat         # 仅显示文件变更统计
```

### 2.2 暂存修改

```bash
# 添加所有修改（含新增、修改、删除）
git add -A

# 或使用 . （当前目录及子目录）
git add .

# 或只添加指定文件
git add 文件路径

# 查看暂存区状态
git status --short
```

### 2.3 提交

```bash
# 提交到当前分支
git commit -m "feat: 描述本次修改"

# 查看提交记录
git log --oneline --decorate --graph --all
```

## 3. git add 命令对比

| 命令 | 作用范围 | 新增文件 | 修改文件 | 删除文件 |
|------|----------|:--------:|:--------:|:--------:|
| `git add -A` | 整个仓库 | ✅ | ✅ | ✅ |
| `git add .` | 当前目录及子目录 | ✅ | ✅ | ✅ |
| `git add -u` | 已被 Git 跟踪的文件 | ❌ | ✅ | ✅ |
| `git add 文件1 文件2` | 指定文件 | ✅ | ✅ | ✅ |

> **记忆技巧**：`-A` = All（全部），`.` = 当前目录，`-u` = update（仅已跟踪文件）

## 4. 相关笔记

- [[Git原理]] — 深入理解 Git 的工作区、暂存区、版本库模型
