---
type: optional-concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-执行优化并行与部署概览|执行优化并行与部署概览]]"
previous: "[[07-优化怎样权衡速度容量质量成本与复杂度|优化怎样权衡速度容量质量成本与复杂度]]"
next: "[[09-多GPU并行方式分别在拆什么|多GPU并行方式分别在拆什么]]"
tags: [llm, runtime, kernel, compilation, speculative-decoding, optional]
---

# Kernel、融合、编译与推测解码在优化什么

> [!optional] 工程选读
> 只需知道这些名称分别减少哪类浪费，不要求学习 GPU 编程或算法细节。

## Kernel、融合与编译

Kernel 可以直白理解为在加速器上执行某一类具体计算的程序。一次模型前向包含很多操作；如果每个小操作都单独启动、反复读写数据，会产生额外开销。

- **算子融合**：把适合连续完成的操作组合起来，减少启动和中间数据搬运；
- **编译优化**：根据计算图、形状和硬件选择或生成更合适的执行方案；
- **专用 Kernel**：针对注意力、量化等常见计算写更高效实现。

它们通常不改变“模型由哪些层组成”，而是改变同一数学工作怎样在硬件上完成。

## Speculative Decoding

推测解码先让较便宜的草稿方法提出多个候选 Token，再由目标模型验证这些候选：

```text
草稿：提出 A B C
目标模型：一次验证能接受到哪里
→ 接受A、B，拒绝C并从目标分布继续
```

它没有让目标模型放弃校验，也没有让未来 Token 变成相互独立。加速效果取决于候选命中率、验证成本、硬件和请求负载，并非总会更快。

来源：[vLLM Speculative Decoding](https://docs.vllm.ai/en/latest/features/speculative_decoding/)，核对日期：2026-07-28。

## 理解检查

1. 算子融合主要减少哪类额外开销？
2. 推测解码为什么不等于草稿模型直接替代目标模型？

下一篇仍为选读：[[09-多GPU并行方式分别在拆什么|多 GPU 并行方式分别在拆什么]]。
