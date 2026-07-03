---
name: gitea-workflow
description: 当需要处理 Gitea 工单任务时使用，包括认领工单、创建分支开发、提交 Pull Request、或根据评审意见进行修订等场景
metadata: 
  version: 0.2.7
---

# Gitea Workflow

## 概述

基于**工单即任务**的理念，借助 `giteacli` 与 Gitea 实例交互，通过标签管理工单状态，配合 Fork 和 Pull Request 机制完成工单开发。

本 skill 涵盖两个核心场景：
- **场景A**：认领工单 → 开发 → 创建 PR
- **场景B**：评审被拒 → 修订 → 重新评审

> 状态标签使用 `status/` 前缀，完整定义见下方章节。

### giteacli

若 `giteacli` 未安装，可执行以下命令安装：

```bash
npm install -g @seepine/giteacli
# 或
bun add -g @seepine/giteacli
```

并进行认证，确保能获取到用户信息：

```bash
# 登录到 Gitea 实例
giteacli login --host <host> --token <token>
# 查看当前登录用户
giteacli whoami
```

> 若需要完整命令参考，优先使用 `gitea-cli` skill 或 `giteacli --help` 查看；本 skill 只保留工单工作流相关命令。

## 工单管理

基于工单即任务的理念，我们通过工单状态和标签对工单进行流转管理。

### 标签定义

> 每个工单在同一时刻应只具有一个状态标签。
> 若标签不存在，优先使用 `giteacli org label add` 命令添加

- `status/draft` - 草稿：工单创建中，内容尚未完善，不进入处理队列
- `status/pending` - 等待中：工单已就绪，等待分配或认领
- `status/working` - 进行中：已有人员正在处理该工单
- `status/reviewing` - 评审中：相关代码/成果已提交，等待评审
- `status/revising` - 修订中：根据评审意见进行修订
- `status/finished` - 已完成：工单任务已处理完成
- `status/aborted` - 已终止：工单不再需要处理（取消、重复、无效等）

### 标签使用约定

1. **状态标签互斥**：一个工单同一时刻只能有一个状态标签
2. **标签前缀**：所有状态标签使用 `status/` 前缀，便于分类和管理
3. **标签流转明确**：认领、提交评审和修订等关键节点必须显式更新工单标签状态


### 严格遵守

1. **最终态不可处理**：当工单标签处于最终状态时，不再进行任何处理，最终状态包括：
    - `status/finished` - 已完成
    - `status/aborted` - 已终止
2. **工单状态**：工单的开启关闭由管理员操作，你不应该操作。
3. **执行前确认**：更新标签前必须先读取工单详情，确认当前状态标签与预期一致，且没有其他人已经接手。

## 最佳实践流程

### 场景A: 处理工单流程

获取指派给自己的工单，并通过派生、克隆、创建工单分支等系列流程进行开发，最后推送代码并创建 Pull Request 请求合并到主仓库，完成工单任务。

具体流程和细节请查看 [references/issue-workflow.md](./references/issue-workflow.md)。

### 场景B: 评审修订流程

创建 Pull Request 后会由专人评审，当评审失败时，需要根据评审意见进行修订。

具体流程和细节请查看 [references/fix-review-workflow.md](./references/fix-review-workflow.md)。
