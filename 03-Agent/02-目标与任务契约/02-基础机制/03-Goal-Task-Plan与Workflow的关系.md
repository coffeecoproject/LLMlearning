---
type: mechanism-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-从模糊需求到正式Goal概览|从模糊需求到正式 Goal 概览]]"
tags: [goal, task, plan, workflow]
---

# Goal、Task、Plan 与 Workflow 的关系

> [!summary]
> Goal 定义最终结果，Task 表示可管理的工作单元，Plan 描述准备怎样完成这些工作，Workflow 则用系统规则控制阶段、转换和允许动作。

## 四个对象放在一起

```text
Goal
修复重复扣款，并满足三条成功条件
        ↓
Tasks
调查回调入口、确认幂等策略、实现修改、验证回归
        ↓
Plan
先定位所有写账入口，再选择实现方案，然后修改并测试
        ↓
Workflow
DISCOVERY → PLAN → IMPLEMENT → VERIFY → CLOSEOUT
```

## Goal：为什么做

Goal 应尽量在实现方法变化时保持稳定。例如从数据库唯一约束改为幂等键，只要最终业务结果不变，就不一定需要改 Goal。

## Task：管理哪一块工作

Task 是可分配、可阻塞、可完成的工作单元。它可以有：

- 输入；
- 依赖；
- 负责人或 Worker；
- 预期产物；
- 局部完成条件。

Task 完成不等于整个 Goal 完成。

## Plan：当前准备怎样做

Plan 是基于当前事实形成的执行假设。Discovery 发现新模块后，Plan 可以修订。

```text
Goal 通常较稳定
Plan 可以随 Observation 调整
```

模型可以提出 Plan，但外部系统需要保存版本、检查 Scope，并决定哪些任务可执行。

## Workflow：哪些阶段和转换合法

Workflow 更接近控制结构：

- 当前处于哪个阶段；
- 哪些动作允许；
- 进入下一阶段需要什么；
- 失败后回到哪里；
- 哪些状态是完成、阻塞或取消。

它不一定由 LLM 生成，很多情况下适合由程序和策略预先定义。

## 最常见的混淆

```text
模型输出一份计划
≠ Workflow 已经改变

完成一个 Task
≠ Goal 已满足

Workflow 走到最后一步
≠ Acceptance 一定通过

Goal 改变
≠ 只改 Prompt 就足够
```

## 一个判断方法

| 问题 | 对应对象 |
|---|---|
| 最终要得到什么 | Goal |
| 需要管理哪些工作单元 | Task |
| 当前准备按什么顺序和方法完成 | Plan |
| 系统允许怎样推进、失败和恢复 | Workflow |
