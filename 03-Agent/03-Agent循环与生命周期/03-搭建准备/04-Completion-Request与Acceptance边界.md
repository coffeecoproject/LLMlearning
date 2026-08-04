---
type: implementation-note
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-Agent-Loop搭建概览|Agent Loop 搭建概览]]"
tags: [completion-request, acceptance, lifecycle]
---

# Completion Request 与 Acceptance 边界

> [!summary]
> 模型或 Worker 可以申请“我认为工作完成”，但只有验收系统根据当前 Goal 和 Evidence 才能把 Run 转为真正完成。

## 两种完全不同的事件

```text
Completion Request
→ 执行者提出：我认为可以开始验收

Acceptance Decision
→ 验收方判断：当前证据是否满足正式 Goal
```

前者是提案，后者是权威决定。

## Completion Request 可以包含什么

```text
attemptId
candidateRef
claimedScope
summary
proposedEvidenceRefs[]
knownLimitations[]
```

这些内容帮助系统开始验证，但不能自行成为证据。

## Runtime 收到申请后做什么

```text
检查当前 Phase 是否允许申请完成
→ 检查 Goal revision 和 Candidate 身份
→ 冻结或识别确切待验证产物
→ 建立 Verification Obligations
→ 运行检查并收集 Evidence
→ 交给 Acceptance Policy
```

## 可能结果

```text
ACCEPT
→ Run 可以进入 COMPLETED

REPAIR
→ 创建新的实现 Attempt 或 Candidate generation

BLOCK
→ 缺少事实、权限或验证能力

REJECT
→ 当前结果不满足 Goal
```

## 为什么要分开

如果执行者能够批准自己：

- 模型可能遗漏未测试场景；
- 工具输出可能被误读；
- 验证可能针对旧 Candidate；
- “已有测试通过”可能没有覆盖 Goal；
- 模型可能在没有真实执行时生成成功叙述。

详细 Evidence 和 Acceptance 机制留给 [[00-验证证据与验收概览|验证、证据与验收]]。
