# 评审修订流程

> 注意：遵循一次只处理一个 PullRequest 的原则执行

## 流程概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    阶段一：发现需要修订的 PR                       │
├─────────────────────────────────────────────────────────────────┤
│ 1. 获取当前用户创建的 Pull Request                                 │
│ 2. 筛选出存在评审意见为 `REQUEST_CHANGES` 状态的 PR                  │
│ 3. 确认对应的 Issue 编号和仓库信息                                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ ⚠️ 阶段二：更新工单状态 【强制执行区 - 不可跳过】                    │
├─────────────────────────────────────────────────────────────────┤
│ 1. 给工单添加评论："I am revising."                                │
│ 2. 移除工单标签 `status/reviewing`                                │
│ 3. 添加工单标签 `status/revising`                                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    阶段三：本地修订开发                            │
├─────────────────────────────────────────────────────────────────┤
│ 1. 检查本地是否存在对应工作目录 ${repo}_issue_${index}              │
│ 2. 若不存在，参照【处理工单流程，阶段一：事前准备】创建工作目录          │
│ 3. 同步上游最新代码                                               │
│ 4. 根据评审意见进行代码修订                                        │
│ 5. 提交代码并推送                                                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ ⚠️ 阶段四：通知评审人并更新状态 【强制执行区 - 不可跳过】               │
├─────────────────────────────────────────────────────────────────┤
│ ⚠️ 1. 给工单添加评论说明修订内容        【强制执行】                   │
│ ⚠️ 2. 请求评审人重新审核               【强制执行】                   │
│ ⚠️ 3. 移除工单标签 `status/revising`   【强制执行】                 │
│ ⚠️ 4. 添加工单标签 `status/reviewing`  【强制执行】                 │
└─────────────────────────────────────────────────────────────────┘
```

## 阶段一：发现需要修订的 PR

### 1.1 获取需要修订的 PR

使用命令获取需要我修订的 PR，若命令没有获取到，则不需要往下执行

```bash
giteacli pr tbd
```

### 1.2 确认 Issue 信息

若找到需要修订的 PR，则从 PR 中提取关联的 Issue 编号，PR body 中通常包含 `Fixed #<issue_index>` 或类似引用。

```bash
giteacli pr get --repo <owner/repo> --index <prIndex>
```

## 阶段二：更新工单状态

> ⚠️ **【强制执行区】** 此阶段包含 **4 个严格按顺序执行的步骤**，不可中断、不可跳过、不可调换顺序！

执行以下命令，记得将其中 `<owner/repo>` 等变量替换成实际的
- 给工单添加评论、更新标签
- 修改 PR 的标题，添加 `WIP: ` 前缀

```bash
giteacli issue comment add --repo <owner/repo> --index <index> --body "I am revising."
giteacli issue del-labels --repo <owner/repo> --index <index> --labels status/reviewing
giteacli issue add-labels --repo <owner/repo> --index <index> --labels status/revising

giteacli pr edit --repo <owner/repo> --index <prIndex> --title "WIP: <originalTitle>"
```

## 阶段三：本地修订开发

### 3.1 检查本地工作目录

检查本地是否存在对应工作目录：

```bash
ls -d workspace_gitea/${repo}_issue_${index}
```

### 3.2 若不存在，参照【处理工单流程，阶段一：事前准备】创建工作目录

参照 [issue-workflow.md](issue-workflow.md) 中的阶段一步骤：

1. 检查并 Fork 仓库（如未 Fork）
2. 克隆 Fork 的仓库到 `${repo}_issue_${index}` 目录
3. 添加上游仓库远程引用

```bash
cd <your_workspace_dir>/workspace_gitea
git clone https://gitea.example.com/${your_username}/${repo}.git ${repo}_issue_${index}
cd ${repo}_issue_${index}
git remote add upstream https://gitea.example.com/${上游仓库owner}/${repo}.git
# 创建对应工单开发分支
git checkout -b dev/issue_${index}
```

### 3.3 同步上游最新代码

同步上游最新代码到工单开发分支，参考命令如下，请注意合并上游最新代码时可能存在冲突，需要先解决冲突

```bash
# 确保处于对应工单分支
git checkout dev/issue_${index}
# 从远程获取 main 分支最新提交
git fetch upstream main
# 合并上游 main 分支代码到开发分支
git merge upstream/main
```

### 3.4 根据评审意见进行代码修订

使用 `giteacli pr reviews` 获取评审的具体意见，根据评审意见逐条修改代码

```bash
giteacli pr reviews --repo <owner/repo> --index <prIndex>
```

有必要的话还可以使用评论相关命令查看历史评论记录

```bash
giteacli pr comments --repo <owner/repo> --index <prIndex>
giteacli issue comment list --repo <owner/repo> --index <issueIndex>
```

### 3.5 设置提交者信息

通过命令获取 username 和 email

```bash
giteacli whoami
```

并设置

```bash
git config user.name <username>
git config user.email <email>
```

### 3.6 提交并推送代码

```bash
git status
git diff
git add <files>
git commit -m '<type>: <subject>

- 根据评审意见修订'
git push origin <head_branch>
```

## 阶段四：通知评审人并更新状态

> ⚠️ **【强制执行区】** 此阶段包含 **4 个严格按顺序执行的步骤**，不可中断、不可跳过、不可调换顺序！
>
> **执行顺序不可调换：必须先给工单添加评论，再请求评审人重新审核，最后更新标签状态**。执行前确认代码已推送成功。

执行前先确认代码已推送成功，且工单仍为 `status/revising`。确认后执行以下命令，记得将其中 `<owner/repo>` 等变量替换成实际的。
- 给工单添加评论
- 修改 PR 的标题去掉 `WIP: ` 前缀，并重新请求审查人审批
- 更新工单标签

```bash
giteacli issue comment add --repo <owner/repo> --index <index> --body "### Revisions\n\n@<creatorUsername> 已完成评审意见的修订，请重新评审。\n\n修订内容：\n- <修订点1>\n- <修订点2>\n\n..."
giteacli pr edit --repo <owner/repo> --index <prIndex> --title "<originalTitle>"
giteacli pr reviewer review --repo <owner/repo> --index <prIndex> --username <username>

giteacli issue del-labels --repo <owner/repo> --index <index> --labels status/revising
giteacli issue add-labels --repo <owner/repo> --index <index> --labels status/reviewing
```

## 注意事项

1. **保持分支同步**：在开始修订前，先从 upstream 拉取最新代码
2. **逐条处理评审意见**：根据评审意见逐条修改(除非说的是同一个问题)，确保每条意见都得到响应
3. **标签状态互斥**：同一工单同时只能有一个状态标签
