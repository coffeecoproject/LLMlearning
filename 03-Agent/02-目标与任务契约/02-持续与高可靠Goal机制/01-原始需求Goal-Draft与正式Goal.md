---
type: mechanism-note
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-从模糊需求到正式Goal概览|从模糊需求到正式 Goal 概览]]"
tags: [raw-request, goal-draft, goal, authority]
---

# 原始需求、Goal Draft 与正式 Goal

> [!summary]
> Raw Request 保存用户实际说了什么，Goal Draft 保存系统建议怎样理解，正式 Goal 保存用户和系统已经确认、可以约束执行的目标。

> [!note] 整篇层级：高可靠 Goal 形成机制（选读）
> 简单 Agent 可以只保存用户请求；只有正式区分需求草稿、用户确认与权威 Goal 时，才需要这三个对象。

## 三个对象

### Raw Request：原始需求

```text
用户说：把订单取消功能完善一下。
```

系统应保存原话、提交者、时间和项目身份。它是需求来源，但通常还不能直接约束复杂执行。

### Goal Draft：目标草稿

```text
候选 Objective：补齐未发货订单取消流程
候选 Criteria：取消后释放库存并阻止发货
Open Question：已支付订单是否自动退款？
```

草稿可以由 LLM 反复生成和修改。它允许存在候选解释、未知项和未确认字段，不拥有启动高风险执行的权威。

### Goal：正式目标

```text
用户已确认：
- 仅处理未发货订单
- 已支付订单取消后进入退款流程
- 已进入拣货的订单不允许取消
```

Goal Manager 把确认结果保存为带身份和 revision 的正式对象。后续 Plan、Run 和 Acceptance 都绑定它。

## 为什么不能覆盖原始需求

如果只保留模型整理后的版本，就无法判断：

- 哪些内容来自用户；
- 哪些是模型补充；
- 用户后来修改了什么；
- 是否发生了语义漂移。

因此合理关系是：

```text
Raw Request
→ 可追溯来源

Goal Draft
→ 可修改提案

Formal Goal
→ 经确认的执行权威
```

## 一个常见错误

```text
用户：修复重复扣款
模型草稿：修复重复扣款，并退款所有历史异常订单
系统：直接开始退款
```

这里模型把“可能相关的后续工作”误升格成正式范围。正确做法是把历史退款记录为待确认问题，而不是直接执行。

## 谁可以修改什么

| 对象 | LLM | 用户 | Goal Manager |
|---|---|---|---|
| Raw Request | 不能改写来源 | 可以补充新输入 | 保存来源记录 |
| Goal Draft | 可以提出和修改建议 | 可以编辑、接受或拒绝 | 校验草稿状态 |
| Formal Goal | 不能直接修改 | 可以申请确认修订 | 创建新 revision |

## 理解检查

1. 为什么 Goal Draft 即使写得很好也不能直接成为业务权威？
2. 为什么 Raw Request 仍需要保留？
3. 用户补充新要求时，应该覆盖旧 Goal，还是创建新 revision？
