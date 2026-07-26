---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
tags: [llm, attention, query, key, value]
---

# Q、K、V 投影系统

> [!summary]
> 同一组输入 Hidden States 会通过不同的线性投影产生 Query、Key 和 Value：Q 参与表达当前位置怎样发起匹配，K 参与表达各位置怎样被匹配，V 则提供匹配后要汇总的内容。

## 子结构

1. [[为什么需要QKV三种角色|为什么需要 Q、K、V 三种角色]] ✓
2. [[Query是什么|Query 是什么]] ✓
3. [[Key是什么|Key 是什么]] ✓
4. [[Value是什么|Value 是什么]] ✓
5. [[QKV怎样由线性投影产生|Q、K、V 怎样由线性投影产生]] ✓

## 一条计算主线

```text
同一组输入 Hidden States
├→ Q 投影 → Query
├→ K 投影 → Key
└→ V 投影 → Value

Query × Key
→ 决定从哪些位置取多少信息

Attention Weight × Value
→ 汇总各位置实际提供的内容
```

## 类比的使用边界

入门时常把 Q、K、V 类比成“检索请求、索引标签和实际内容”。这个类比有助于分清角色，但真实 Q、K、V 都是由模型参数计算出的向量，不是自然语言问题、数据库字段或原文复制。

## 完成标准

学完后应能：

1. 解释为什么匹配依据和被取出的内容需要分开；
2. 说明每个 Token 位置都会产生自己的 Q、K、V；
3. 区分 Q/K 的匹配职责与 V 的内容职责；
4. 说明 Q、K、V 是投影结果，不是模型权重本身；
5. 在单头教学示例中读懂其基本形状。
