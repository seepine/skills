# Skills Collection

一个小而美的 skills 合集。

- 优先 cli，避免手写代码脚本
- 破坏性小，避免创建额外文件影响项目

## 安装 skill

可直接将仓库地址发给 `ClaudeCode`、`openclaw` 等 Agent，或者执行以下命令安装

### 1. 快速安装

安装全部技能

```
npx -y skills add https://github.com/seepine/skills
```

### 2. 指定安装

只安装某个技能

```
npx -y skills add https://github.com/seepine/skills --skill git-guide
```

### 3. 指定 Agent

安装全部技能到指定的 Agent

```
npx -y skills add https://github.com/seepine/skills -a claude-code -a codex -a opencode -g
```

## Skills

### 编码实践

| Skill                 | 描述                                                               |
| --------------------- | ------------------------------------------------------------------ |
| `karpathy-guidelines` | 降低 LLM 编码时的常见问题，强调简单、克制、明确假设和可验证结果。  |
| `code-simplifier`     | 在代码修改后进行简化和清理，提升可读性与一致性，同时保持功能不变。 |

### 协作流程

| Skill            | 描述                                                                              |
| ---------------- | --------------------------------------------------------------------------------- |
| `git-guide`      | 基于 git 开发指南，阐述如何使用 Fork 和 PR 通过 git 协同开发。                    |
| `gh-cli`         | GitHub CLI 使用参考，用于通过命令行管理 GitHub 仓库、Issue、PR 和 Actions 等。    |
| `gitea-cli`      | Gitea CLI 使用参考，用于通过命令行管理 Gitea 仓库、Issue、PR、评审和 Actions 等。 |
| `gitea-workflow` | Gitea 工单处理工作流，用于认领工单、创建分支、提交 PR 和根据评审意见修订。        |

### 文案推广

| Skill                 | 描述                                                                |
| --------------------- | ------------------------------------------------------------------- |
| `copywriting-formula` | 使用“现象 + 危害 + 原因 + 解决办法”公式撰写和改写更有转化感的文案。 |

### 多媒体工具

| Skill            | 描述                                                                               |
| ---------------- | ---------------------------------------------------------------------------------- |
| `text-to-speech` | 使用 Microsoft Edge 在线 TTS 服务通过 `node-edge-tts` 将文本转换为语音并生成音频。 |

## 目录结构

```text
.
├── README.md
└── skills/
    ├── <skill-name>/
    │   └── SKILL.md
```

每个 skill 都是一个独立目录，核心说明文件为 `SKILL.md`。

## 许可证

除非单个 skill 另有说明，本仓库内容默认按 MIT License 使用。

## 致谢

部分 skill 参考或引用了以下项目：

- [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
