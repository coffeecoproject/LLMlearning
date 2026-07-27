---
type: topic-index
module: 1
status: planned
audience: non-specialist
parent: "[[00-推理Runtime与模型服务大纲|推理Runtime与模型服务大纲]]"
previous: "[[00-批处理与调度概览|批处理与调度概览]]"
next: "[[00-接口流式传输与部署概览|接口流式传输与部署概览]]"
tags: [llm, runtime, kv-cache, paged-attention, prefix-cache]
---

# KV Cache 工程管理概览

> [!summary]
> 单请求机制说明为什么可以缓存历史 K、V；Runtime 工程则负责为许多长度不断变化的请求分配、定位、复用和释放缓存内存。

## 计划内容

1. 请求越长，KV Cache 为什么通常越大；
2. 连续内存分配为什么容易浪费和碎片化；
3. Paged KV Cache / PagedAttention 的“分页”解决什么；
4. Prefix Cache 怎样复用相同输入前缀的计算；
5. Cache Offload、量化和回收的基本权衡；
6. MHA、GQA、MQA、MLA 为什么会改变具体缓存形态。

机制入口：[[01-KV-Cache保存什么|KV Cache 保存什么]]。
