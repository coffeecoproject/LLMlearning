---
type: topic-overview
module: 3
status: complete
audience: non-specialist
parent: "[[00-目标与任务契约概览|目标与任务契约概览]]"
tags: [agent, goal-intake, clarification, mechanism]
---

# 从模糊需求到正式 Goal 概览

> [!summary]
> 当系统选择使用高可靠 Goal Intake 时，模糊需求不能经过一次模型生成就自动成为正式 Goal；它需要经历草稿、歧义识别、有限探索、用户确认和权威创建。

本页讲的是完整 Goal Contract 的形成机制。简单任务可以跳过这一流程，Codex 一类持续 Agent 也可以直接由用户设置 Thread Goal；是否启用 Intake 应取决于需求歧义、任务长度、影响范围和风险。

## 完整形成过程

```text
1. 接收 Raw Request
2. 保存用户原话和来源
3. LLM 或规则生成 Goal Draft
4. 标出缺失条件、歧义和候选解释
5. 必要时对项目做有限只读探索
6. 只向用户询问会改变目标的重要问题
7. 用户确认或修改草稿
8. Goal Manager 校验结构和权限
9. 创建正式 Goal revision 1
10. 后续 Agent Run 绑定这个 revision
```

## 为什么不能一次生成后直接执行

LLM 可以把语言整理得很完整，但完整的表达不代表内容来自用户。

例如模型可能自动补出：

```text
Non-goal：不处理历史订单
```

这看起来合理，却属于业务取舍。除非用户或已有权威规则确认，否则它只能留在草稿或未决问题中。

## 两类确认

```text
结构确认
→ 字段非空、格式合法、至少有一个成功条件

语义确认
→ 用户确实接受目标、范围、排除项和关键业务选择
```

程序比较容易自动完成结构确认；语义确认往往需要用户、产品规则或可信项目事实。

## 只读探索的作用

有些问题在看项目之前无法提出。例如用户说“增加取消订单”，只有读取现有状态机后才知道要问：

```text
已支付订单是否允许取消？
已进入拣货的订单怎样处理？
取消是否需要释放库存和发起退款？
```

这类探索帮助形成更好的问题，但仍不能替用户决定业务答案。

## 机制阅读顺序

1. [[01-原始需求Goal-Draft与正式Goal|原始需求、Goal Draft 与正式 Goal]]；
2. [[02-成功条件怎样变得可验证|成功条件怎样变得可验证]]；
3. [[03-Goal-Task-Plan与Workflow的关系|Goal、Task、Plan 与 Workflow 的关系]]；
4. [[04-Goal如何约束后续执行|Goal 如何约束后续执行]]。

读完后再进入 [[00-Goal-Intake搭建概览|Goal Intake 搭建概览]]。
