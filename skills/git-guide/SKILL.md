---
name: git-guide
description: 通用 Git 协作指南，说明如何使用 Fork、分支和 PR 进行协同开发
metadata: 
  version: 0.1.0
---

# git-guide

## 定位

本 skill 只描述通用 Git 协作原则。若任务来自 Gitea 工单并需要认领、更新标签或修订评审，请优先使用 `gitea-workflow`。

## 基于 Fork + PR 的开发流程

### Fork

不直接克隆上游仓库进行开发，而是 Fork 一份自己的仓库。

克隆自己的仓库，并添加上游仓库到 remote，拉取上游仓库最新代码。

基于多分支或 worktree 的模型进行开发，编码完成后推送到自己的仓库。

### 开发

不要一开始就直接编码，先分析需求、约束和验证方式；复杂任务可先使用 `karpathy-guidelines`。

### 验证

若项目的 `AGENTS.md`、`CLAUDE.md` 或 README 有验证说明则优先遵循，否则尝试以下步骤进行验证。

- 尝试使用 code-simplifier 简化代码
- 查看项目是否有 lint、format、test 等相关命令或工具，有则执行
- 验证所有功能是否达成

### 提交

提交前先执行 `git status` 和 `git diff`，确认只包含本次任务相关变更；暂存时优先指定文件，不要无脑 `git add .`。

若仓库没有对 git 提交信息进行限制，则默认遵循 Conventional Commits 风格。

```
<type>(<scope>): <subject>
```

- type（必须） : commit 的类别，只允许使用下面几个标识：
  - feat : 新功能
  - fix : 修复bug
  - docs : 文档改变
  - style : 代码格式改变
  - refactor : 某个已有功能重构
  - perf : 性能优化
  - test : 增加测试
  - build : 改变了build工具 如 grunt换成了 npm
  - revert : 撤销上一次的 commit
  - chore : 构建过程或辅助工具的变动
- scope（可选） : 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
- subject（必须） : commit 的简短描述，尽量不超过 50 个字符。

### PR

通过 `gh`、`giteacli` 等 CLI 创建 PR，请求代码合并到上游仓库中。

若此次修复工单，在 PR 描述中关联工单：

```
Fixed #ISSUE_ID
```

## 修订 PR

- 当开始修订前，应该使用 CLI 修改 PR 的标题，添加 `WIP: ` 前缀
- 当修订完成，且推送最新代码上去之后，应该修改 PR 标题，移除 `WIP: ` 前缀

## 示例

例如你是：`job`，邮箱为 `job@example.com`

- 上游仓库：https://git.example.com/agenteam/repo
- Fork 的仓库应为 `https://git.example.com/job/repo`
- 创建的 PR 应该创建到上游仓库 `https://git.example.com/agenteam/repo` 中
