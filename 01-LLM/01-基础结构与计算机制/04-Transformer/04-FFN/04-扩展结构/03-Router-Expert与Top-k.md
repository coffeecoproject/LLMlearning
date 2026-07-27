---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN扩展结构概览|FFN扩展结构概览]]"
previous: "[[02-MoE基础与Dense-FFN对比|MoE基础与Dense-FFN对比]]"
next: "[[04-MoE总参数与每Token激活参数|MoE总参数与每Token激活参数]]"
tags: [llm, moe, router, expert, top-k]
---

# Router、Expert 与 Top-k

> [!summary]
> Router 根据每个 Token 位置当前的 Hidden State 计算 Expert 分数，选出 Top-k Expert；选中 Expert 分别处理该位置，再按路由权重合并输出。

## 完整路线

```text
一个位置的 Hidden State
→ Router 产生所有 Expert 分数
→ 选择 Top-k
→ 选中 Expert 分别计算
→ 按路由权重合并
→ MoE 输出
```

## 小数字示例

假设 4 个 Expert 的教学分数是：

```text
Expert 1：0.10
Expert 2：0.55
Expert 3：0.25
Expert 4：0.10
```

`Top-k=2` 时选择 Expert 2 和 Expert 3。假设重新归一化后的权重为 `0.69` 和 `0.31`：

```text
MoE输出
= 0.69 × Expert2(x)
+ 0.31 × Expert3(x)
```

数字仅用于说明，不来自真实模型。

## 路由为什么是逐 Token 的

Router 输入是当前层 Hidden State。同一个 Token 字符串在不同上下文、不同层拥有不同表示，因此不保证始终选择相同 Expert。

## Top-k 不要混淆

```text
MoE Top-k
→ 为当前 Token 选择 k 个 Expert

生成 Top-k Sampling
→ 从词表 Logits 中保留 k 个候选 Token
```

二者选择对象和发生位置完全不同。

## Routed 与 Shared Expert

部分架构还包含 Shared Expert：

```text
Routed Expert → 由 Router 选择
Shared Expert → 所有 Token 共同启用
```

因此不是所有 MoE 都能简化成“只运行 Top-k，其他一切关闭”。

## 与 Agent Router 的区别

```text
MoE Router
→ 模型内部，每层、每 Token 的数值路径选择

Agent Router
→ 模型外部，选择工具、模型、子 Agent 或工作流
```

## 常见误解

- Router 通常根据 Hidden State，不是人工 Token 分类表。
- Top-k 选择的是 Expert，不是词表 Token。
- 多个选中 Expert 的结果通常会加权合并。
- 相同文本 Token 不保证各层路由相同。
- Shared Expert 不一定经过 Top-k。

下一篇：[[04-MoE总参数与每Token激活参数|MoE 总参数与每 Token 激活参数]]。
