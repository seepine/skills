---
name: git-commit-generate
description: generate git commit message
metadata: 
  version: 0.1.1
---

给暂存的代码生成中英文双版本 git 提交信息

## 第一步：查看暂存文件

### 1.1 先查看暂存的文件

```sh
git --no-pager diff --cached --stat
```

### 1.2 再查看每个文件的变动

```sh
git --no-pager diff --cached <filePath>
```

## 第二步：生成逻辑

结合最近三次的提交信息，再根据是否存在提交规范进行判断生成

```sh
git --no-pager log --oneline -3
```

### 2.1 存在提交规范

若项目中存在提交规范，例如 `commitlint.config.js`, `.versionrc` 等文件，则按照其规范

### 2.2 不存在提交规范

优先参考 Conventional Commits 规范

```
[type]<scope>: [subject]
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
- scope（可选）: 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
- subject（必须）: commit 的简短描述，不超过50个字符。


## 注意

- 提交信息不要包含 / ` 等特殊符号
- 除非项目提交规范要求，否则提交信息不要包含 emoji 图标

$ARGUMENTS
