# 处理工单流程

> 注意：遵循一次只处理一个工单的原则执行

基于工单即任务的理念，通过派生（Fork）和 Pull Request 机制完成工单任务开发。

## 流程概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    阶段一：事前准备                               │
├─────────────────────────────────────────────────────────────────┤
│ 1. 获取指派给自己且含有 `status/pending` 标签的工单                 │
│ 2. 获取仓库信息与工单索引                                          │
│ 3. 检查仓库是否 Fork，没有则 Fork                                  │
│ 4. 克隆自己 Fork 的仓库                                           │
│ 5. 添加上游仓库远程引用                                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ ⚠️ 阶段二：工单流转 【强制执行区 - 不可跳过】                        │
├─────────────────────────────────────────────────────────────────┤
│ 1. 给工单添加一条评论："I am working."                             │
│ 2. 移除工单标签 `status/pending`                                  │
│ 3. 添加工单标签 `status/working`                                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    阶段三：工单处理                                │
├─────────────────────────────────────────────────────────────────┤
│ 1. 查看工单详细信息和所有评论                                       │
│ 2. 详细分析工单需求                                                │
│ 3. 根据分析结果进行编码实现                                         │
│ 4. 使用 Git 提交代码                                              │
│ 5. 推送到远程仓库                                                 │
│ 6. 创建 Pull Request                                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ ⚠️ 阶段四：提交 PR 并更新状态 【强制执行区 - 不可跳过】               │
├─────────────────────────────────────────────────────────────────┤
│ ⚠️ 1. 移除工单标签 `status/working`   【强制执行】                  │
│ ⚠️ 2. 添加工单标签 `status/reviewing` 【强制执行】                  │
└─────────────────────────────────────────────────────────────────┘
```

## 阶段一：事前准备

### 1.1 获取指派给自己的工单

使用 `giteacli issue search` 搜索指派给自己的工单第一个：

```bash
giteacli issue search --state open --labels status/pending --assigned true --limit 20 | jq '.matchItems.[0]'
```

注意，工单必须：
- 必须是指派给自己
- 必须是开启的
- 必须包含标签 `status/pending`

### 1.2 获取仓库信息与工单索引

从第一个工单中提取 `owner`、`repo` 和工单 `index` 编号。

### 1.3 检查并 Fork 仓库（如未 Fork）

通过 `giteacli repo list` 检查是否已 Fork 仓库，例如上游仓库为 `owner/repo`，则应该存在仓库 `<my-username>/repo`，如未 Fork 则使用 `giteacli repo fork` 工具。

```bash
# 检查是否已 Fork
giteacli repo list

# Fork 仓库
giteacli repo fork --repo <owner/repo>
```

### 1.4 克隆自己 Fork 的仓库

进入工作空间，例如 openclaw/hermes 等一般会有自己专属的工作空间目录，若没有则一般使用用户目录，创建文件夹 `workspace_gitea`

并克隆 Fork 的仓库，记住不要直接克隆，而是克隆的名称附上工单及编号，避免多个工单并行开发时出错

```bash
cd <your_workspace_dir>/workspace_gitea
git clone https://gitea.example.com/${your_username}/${repo}.git ${repo}_issue_${index}
```

最终目录应该类似如下

```
- your_workspace_dir
  - workspace_gitea
    - repo_issue_1
    - repo_issue_2
    - repo_issue_3
```

### 1.5 添加上游仓库远程引用

验证远程配置：

```bash
git remote -v
```

若 `upstream` 已存在，先确认地址是否为上游仓库，不要重复添加。

```bash
git remote add upstream https://gitea.example.com/${上游仓库owner}/${repo}.git
```

### 1.6 设置提交者信息

通过命令获取 username 和 email

```bash
giteacli whoami
```

并设置

```bash
git config user.name <username>
git config user.email <email>
```

## 阶段二：工单流转

> ⚠️ **【强制执行区】** 此阶段包含 **3 个命令**，不可中断、不可跳过、不可调换顺序！

执行前先读取工单详情，确认工单仍为 `status/pending` 且没有其他人已经接手。确认后执行以下命令添加评论，并更新标签。

```bash
giteacli issue comment add --repo <owner/repo> --index <index> --body "I am working."
giteacli issue del-labels --repo <owner/repo> --index <index> --labels status/pending
giteacli issue add-labels --repo <owner/repo> --index <index> --labels status/working
```

## 阶段三：工单处理

### 3.1 查看工单详细信息和所有评论

使用 `giteacli issue get` 获取工单详情：

```bash
giteacli issue get --repo <owner/repo> --index <index>
```

使用 `giteacli issue comment list` 获取所有评论：

```bash
giteacli issue comment list --repo <owner/repo> --index <index>
```

### 3.2 详细分析工单需求

仔细阅读工单描述和所有评论，理解：
- 工单要解决什么问题
- 期望的实现方案

### 3.3 根据分析结果进行编码实现

根据分析结果进行代码编写。编码前同步上游最新代码，并创建工单分支：

```bash
git checkout main
git pull upstream main
git checkout -b dev/issue_${工单索引}
```

### 3.4 使用 Git 提交代码

编码完成之后提交代码。提交前必须检查工作区，确认没有混入用户或其他任务的变更；暂存时优先指定文件路径。

```bash
git status
git diff
git add <files>
git commit -m '提交信息'
```

提交信息格式参考项目历史，一般遵循：

```
<type>: <subject>

<body(optional)>
```

常用 type：
- `feat` - 新功能
- `fix` - 修复 bug
- `docs` - 文档变更
- `refactor` - 重构
- `test` - 测试相关
- `chore` - 构建/工具相关

### 3.5 推送到远程仓库

```bash
git push origin dev/issue_${工单索引}
```

### 3.6 创建 Pull Request

使用 `giteacli pr add` 创建 PR，将派生的 `dev/issue_${工单索引}` 分支合并到主仓库的默认分支。

- 标题尽量简短但能描述清楚，参考如下
  ```
  <type>: <subject>
  ```
- 内容可填写详细修改点，且内容中包含 `Fixed #<工单索引>`，将此 PR 与修复的工单进行关联，参考如下
  ```md
  ### 总结
  - <变更点1>
  - <变更点2>

  Fixed #<工单索引>
  ```

```bash
giteacli pr add --repo <owner/repo> --title "<title>" --head <head_branch> --base main --body '### 总结\n\n- <变更点1>\n- <变更点2>\n\nFixed #<工单索引>'
```

当 PR 创建完成之后，若有说明，则为 PR 添加评审人

```bash
giteacli pr reviewer add --repo <owner/repo> --index <prIndex> --username <username>
```

## 阶段四：提交 PR 并更新状态

> ⚠️ **【强制执行区】** 此阶段包含 **2 个命令**，不可中断、不可跳过、不可调换顺序！执行前再次确认 PR 已创建成功。

执行以下命令更新工单标签

```bash
giteacli issue del-labels --repo <owner/repo> --index <index> --labels status/working
giteacli issue add-labels --repo <owner/repo> --index <index> --labels status/reviewing
```

## 注意事项

1. **保持分支同步**：在开始新任务前，定期从 upstream/main 拉取最新代码
2. **频繁提交**：实现一个完整的小功能后即可提交，不要等到全部完成
3. **PR 描述清晰**：清晰说明解决的问题和变更内容
4. **标签状态互斥**：同一工单同时只能有一个状态标签
