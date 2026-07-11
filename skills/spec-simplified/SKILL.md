---
name: spec-simplified
description: spec 规范流程简化版。判断任务是否分三步：编写 spec.md 明确需求、编写 plan.md 拆解计划、编写 task 文档执行落地。每步完成后停下向用户确认。适用于大功能点或重构。
---

# Simple Spec

复杂任务进入 spec 流程,分三步推进, 除非用户说不要确认直接做，否则 **每步完成后停下向用户确认,通过后再进入下一步**。

所有文档保存在 `<rootDir>/docs/specs/$date-$name/` 目录下:

- `$date`: 当前日期,格式 `yyyyMMdd`
- `$name`: 本次任务简称,1-3 个单词描述清楚,单词用横杠隔开

例如「修复密码格式校验」→ `docs/specs/20260710-password-schema/`

## 第一步: 编写 spec.md

路径:`docs/specs/$date-$name/spec.md`

遵循 `./references/grill-me.md` 原则,明确需求与假设。

如果存在阻塞性歧义,先停下向用户提问;否则直接编写 spec.md 草案。spec.md 至少包含:

- 背景/问题
- 目标
- 影响范围
- 验收标准

完成后**停下,向用户确认**,通过后进入第二步。

## 第二步: 编写 plan.md

路径:`docs/specs/$date-$name/plan.md`

根据 spec.md 拆解功能点,编写 plan.md 计划文档:

- 按优先级用「字母 + 数字」编号(A1、A2、B1...)
- 每项带复选框,用于标记完成状态
- 只列大概思路,**不写具体修复代码**

模板:

```markdown
## A xxx

- [ ] A1 xxxx
- [ ] A2 xxxx

## B xxx

- [ ] B1 xxxx
```

完成后**停下,向用户确认**,通过后进入第三步。

## 第三步:编写 task 文档

路径:`docs/specs/$date-$name/task-$index.md`

根据 plan.md,为每个字母分组编写一个任务文档,如 `task-A.md`、`task-B.md`。

- 对该组内每项计划(A1、A2...)编写具体、可执行、可复现的任务内容
- 每项任务需包含:目标、修改范围、执行步骤、验证方式
- 必要时包含伪代码、接口草案或关键路径说明
- 仅在对应实现任务完成并验证通过后,才回到 plan.md 将对应项标记为已完成 `[x]`

## 开始执行

完成所有文档后**停下,向用户确认两件事**:

1. 是否使用子 agent 模式修复
2. 是每完成一个任务就停下等待确认,还是一次性完成所有任务
