---
type: framework-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-目标与任务契约概览|目标与任务契约概览]]"
tags: [agent, goal, objective, success-criteria, scope, non-goals]
---

# Goal 的四个核心部分

> [!summary]
> 一个高可靠 Goal Contract 应分别说明“要实现什么、怎样算成功、必须覆盖什么、明确不做什么”；简单 Agent 或 Thread Goal 可能只显式保存 Objective。

## 这四项属于哪一层

```text
简单 Agent
→ 用户请求可能同时隐含目标和边界

Codex Thread Goal
→ 可以把完整要求写进一个 Objective，但系统不把四项拆成独立字段

高可靠 Goal Contract
→ 将四项分别保存、确认、修订并用于后续验收
```

所以“四个核心部分”是本专题推荐的任务契约结构，不是所有 Agent 框架的统一数据格式。

## 从一句话变成任务契约

用户说：

```text
修复订单重复扣款。
```

可以先整理成：

```text
Objective
修复支付回调重复触发造成的重复扣款。

Success Criteria
1. 同一支付事件重复到达时只能扣款一次。
2. 正常首次支付仍然成功。
3. 并发重复回调测试通过。

Scope
支付回调、扣款幂等处理和相关订单状态更新。

Non-goals
1. 本次不自动退款历史重复扣款。
2. 本次不更换第三方支付平台。
```

这是说明性例子，真实系统仍需要结合项目和用户确认。

## Objective：要改变什么

Objective 描述目标结果，而不是一串操作。

```text
较弱：检查支付代码并修改几个文件
较好：阻止重复支付事件造成重复扣款
```

前者描述动作，后者描述希望产生的结果。实现方式可能随着 Discovery 改变，但目标结果应保持稳定。

## Success Criteria：怎样才算成功

成功条件必须能够在任务结束时被观察或检查。

```text
较弱：代码质量更好
较好：相同支付事件重复提交时，账本只产生一条扣款记录
```

成功条件不必全部是自动测试，也可以由人工检查、外部系统事实或业务确认完成。但必须明确由什么证据判断。

## Scope：必须覆盖什么

Scope 防止系统只修复最容易看到的一小块，也限制它无限扩大任务。

它可以包括：

- 业务场景；
- 用户类型；
- 数据范围；
- 允许影响的系统；
- 必须检查的模块；
- 风险等级和时间边界。

Scope 不应过早缩成“只改某一个文件”，因为 Discovery 可能发现真实影响跨越多个模块。

## Non-goals：明确不做什么

Non-goals 不是无关备注，而是防止范围蔓延和错误期待。

例如：

```text
修复未来重复扣款
≠ 自动处理过去已经发生的退款
```

如果不明确，Agent 可能把历史数据修复当成必要步骤，也可能用户误以为它已经处理。

## 四者怎样共同工作

```text
Objective
→ 指明方向

Success Criteria
→ 定义完成

Scope
→ 定义覆盖与允许影响的边界

Non-goals
→ 定义明确排除项
```

在高可靠任务契约中，缺少任何一项都可能带来不同问题：

| 缺少内容 | 常见后果 |
|---|---|
| Objective | 系统不知道最终要改变什么 |
| Success Criteria | 模型可以随意声称完成 |
| Scope | 可能遗漏业务路径或无限扩展 |
| Non-goals | 用户和 Agent 对交付范围理解不同 |

## 框架停止点

如果你能把一个模糊需求分别写成 Objective、Success Criteria、Scope 和 Non-goals，并说明四者的作用，就可以进入下一专题。想理解形成过程，继续阅读 [[00-从模糊需求到正式Goal概览|从模糊需求到正式 Goal 概览]]。
