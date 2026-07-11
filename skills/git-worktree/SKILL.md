---
name: git-worktree
description: Git Worktree 全流程管理。包含创建、合并、清理三个流程。适用于需要并行开发多个特性或在独立工作区处理任务的场景。
---

# Git Worktree 工作流

本技能管理 git worktree 的完整生命周期：**创建 → 合并 → 清理**。

## 参数规范

所有路径与分支命名遵循以下规范:

- `$projectName`: 当前项目名,例如 `myproject`
- `$date`: 当前日期,格式 `yyyyMMdd`
- `$taskName`: 本次任务简称,1-3 个单词描述清楚,单词用横杠隔开

**目录规范**: `../$projectName-$date-$taskName`
**分支规范**: `$date/$taskName`

**示例**:

- 目录: `../myproject-20260101-fix-password-schema`
- 分支: `20260101/fix-password-schema`

---

## 第一步:创建

### 1. 检查未提交代码

执行 `git status` 检查当前 worktree 是否存在未提交或未推送的改动:

```bash
git status --short
git log origin/main..HEAD --oneline 2>/dev/null
```

### 2. 向用户确认

**若存在未提交代码,必须停下向用户确认**,告知用户:

- 检测到未提交的修改或未推送的 commit
- 询问如何处理(丢弃/stash/提交后再创建/取消创建)
- **不要直接创建 worktree**,避免改动丢失或混乱

若工作区干净,可直接进入创建步骤。

### 3. 创建 Worktree

**必须在主 worktree(项目根目录)中执行**,且主 worktree 应位于 `main` 分支:

```bash
# 确保在主 worktree 且在 main 分支
cd $projectName
git checkout main
git pull origin main

# 创建 worktree
git worktree add -b <$date/$taskName> <../$projectName-$date-$taskName>
```

**完整示例**:

```bash
git worktree add -b 20260101/fix-password-schema ../myproject-20260101-fix-password-schema
```

创建完成后,提示用户切换到新 worktree 开始工作:

```bash
cd ../$projectName-$date-$taskName
```

---

## 第二步:合并

### 1. 检查远程分支

在主 worktree 中检查当前 worktree 分支是否已推送到远程:

```bash
cd $projectName
git branch -vv
git ls-remote origin <$date/$taskName>
```

### 2. 判断是否为 fork 仓库

检查是否存在 `upstream`/`up` 等特殊远程仓库:

```bash
git remote -v
```

**关键判断**:

- 若存在 `upstream` 等上游仓库(说明本仓库是 fork),PR 应优先创建到 `upstream`,而非 `origin`
- PR 的目标分支(base)通常为 `main`

### 3. 通过 PR 合并(推荐)

**若存在远程分支**:

1. **若存在 upstream 等上游仓库**,询问用户是否将 PR 创建到上游仓库
2. **否则**直接通过 origin 创建 PR

使用 `gh` CLI 创建 PR:

```bash
# 推送到远程(若尚未推送)
git push -u origin <$date/$taskName>

# 创建 PR(到 origin)
gh pr create --base main --head <$date/$taskName>

# 创建 PR(到 upstream,适用于 fork 场景)
gh pr create --repo <upstream-url> --base main --head <$myusername$:$date/$taskName>
```

完成后**停下,等待用户在 GitHub 上完成 PR 合并**,再进入清理流程。

> 若非 Github 仓库，而是自建 Gitea 平台，则可以尝试使用 giteacli

### 4. 直接合并到 main(无 PR)

**若不存在远程分支,或用户选择不通过 PR 合并**,在主 worktree 中执行:

```bash
# 必须切换回 main worktree
cd $projectName

# 切换到 main,避免主 worktree 不在 main 分支
git checkout main

# 拉取远程最新代码,避免分支已过期
git pull origin main

# 合并 worktree 分支(必须 --no-ff 保留分支轨迹)
git merge --no-ff <$date/$taskName>

# 推送
git push origin main
```

**完整示例**:

```bash
cd myproject
git checkout main
git pull origin main
git merge --no-ff 20260101/fix-password-schema
git push origin main
```

---

## 第三步:清理

合并完成后,清理 worktree 与本地分支。

### 1. 删除 worktree 目录

**必须在主 worktree 中执行**(不能在要删除的 worktree 中删除自身):

```bash
cd $projectName

# 删除指定目录
git worktree remove <../$projectName-$date-$taskName>
```

若有未提交修改，向用户确认是否强制删除，若用户确认删除，可使用 `--force` 强制删除

```bash

git worktree remove --force <../$projectName-$date-$taskName>
```

**完整示例**:

```bash
git worktree remove ../myproject-20260101-fix-password-schema
```

### 2. 清理失效引用

```bash
# 执行清理
git worktree prune
```

### 3. 删除本地分支(可选)

若分支已合并到 main,可安全删除:

```bash
# 小写 -d,仅删除已合并的分支(安全)
git branch -d <$date/$taskName>

# 大写 -D,强制删除,需向用户确认
git branch -D <$date/$taskName>
```

### 4. 验证清理结果

```bash
git worktree list
git branch -vv
```

确认目标 worktree 与分支已从列表中移除。

---

## 完整工作流示例

[example.md](./references/example.md)

## 注意事项

1. **同一分支不能同时在两个 worktree 检出**(避免冲突)
2. **不能在 worktree 中删除其所在分支**(要先离开)
3. 删除目录后记得执行 `git worktree prune` 清理引用
4. Worktree 之间共享 `.git` 目录,但工作区独立,**stash/reflog 是共享的**
5. **创建 worktree 前必须确认工作区干净**,避免改动丢失
6. **合并时必须 --no-ff**,保留分支合并轨迹,便于追溯
7. **fork 仓库的 PR 应创建到 upstream**,而非 origin
