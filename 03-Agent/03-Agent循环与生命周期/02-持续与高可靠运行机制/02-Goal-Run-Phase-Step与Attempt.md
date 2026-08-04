---
type: mechanism-note
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-运行对象与状态机概览|运行对象与状态机概览]]"
tags: [goal, run, phase, step, attempt]
---

# Goal、Run、Phase、Step 与 Attempt

> [!summary]
> 这些对象描述任务执行的不同粒度：Goal 说明为什么做，Run 标识哪一次执行，Phase 限制当前阶段，Attempt 区分第几次尝试，Step 记录其中一个具体处理步骤。

## 先与交互协议对象分开

本页讨论的是任务执行与控制对象：

```text
Goal / Run / Phase / Attempt / Step
```

上一篇讨论的是产品交互与协议对象：

```text
Thread / Turn / Item / Model Call
```

两组对象会互相引用，但不应强行塞进一棵通用父子树。不同 Agent 产品可能只实现其中一部分，也可能使用不同名称。

## 一层层看

```text
Goal：修复重复扣款
└── Run R1：本次受控执行
    ├── Phase：DISCOVERY
    │   └── Attempt D1
    │       ├── Step：搜索支付入口
    │       └── Step：读取回调和账本代码
    ├── Phase：IMPLEMENT
    │   ├── Attempt I1：第一版修改失败
    │   └── Attempt I2：根据失败证据修复
    └── Phase：VERIFY
        └── Attempt V1：执行验收检查
```

这是一种高可靠说明性结构，不是所有 Agent 的最低要求。

## 每个对象解决什么

| 对象 | 解决的问题 |
|---|---|
| Goal | 最终为什么执行、怎样算完成 |
| Run | 哪一次实际执行正在处理这个 Goal |
| Phase | 当前阶段的目的、权限和退出条件是什么 |
| Attempt | 这是对当前阶段或任务的第几次尝试 |
| Step | Runtime 正在执行或记录哪个具体处理步骤 |

## 它们怎样关联 Codex 的 Thread 与 Turn

```text
Run R1
├── 可以使用一个持续 Thread
│   ├── Turn 1：用户提出任务，内部发生多次 Model Call 和工具执行
│   └── Turn 2：用户补充要求，Agent 继续处理
└── 也可以在不同 Phase 启动新的 Thread 或 Worker
```

一个 Codex Turn 内可以包含多个 Tool Step；一个长期 Run 也可以跨越多个 Turn。因此：

```text
Turn
≠ Step
≠ Model Call
≠ Run
```

## Attempt 为什么重要

假设第一次实现失败：

```text
IMPLEMENT Attempt I1
→ 修改 Candidate C1
→ 测试失败

IMPLEMENT Attempt I2
→ 根据 C1 的失败证据生成 Candidate C2
→ 重新验证
```

如果不区分 Attempt，旧结果、错误和新结果容易混在一起，也无法准确说明哪份 Evidence 对应哪次产物。

## 不要过度建模

最小 Agent 可以只有：

```text
用户目标
+ 当前执行状态
+ Model Call
+ Tool Call / Result
+ 明确停止条件
```

当任务需要跨进程恢复、阶段权限、重试隔离和独立验收时，再逐步引入 Run、Phase、Attempt 和结构化 Step。概念上理解这些对象，不代表第一版必须全部实现。

下一篇：[[03-状态转换与守卫|状态转换与守卫]]。
