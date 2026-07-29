---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-KV-Cache工程管理概览|KV-Cache工程管理概览]]"
previous: "[[08-注意力结构怎样改变缓存形态|注意力结构怎样改变缓存形态]]"
next: "[[00-API与流式传输概览|API与流式传输概览]]"
tags: [llm, runtime, kv-cache, multi-request, worked-example]
---

# 多请求 KV Cache 管理完整案例

> [!summary]
> 多请求缓存管理可以压缩成一个循环：请求获得Block、Prefill写入、Decode追加、相同前缀尝试复用、完成后解除占用，空间不足时再由Runtime选择等待或回收策略。

> [!phase] 运行阶段：概念串联案例
> 所有Block数量和Token数量均为教学虚构，不代表真实模型配置。

## 场景

假设：

```text
缓存池共有8个Block
每个Block能容纳4个Token位置
请求A和B先到达
请求C稍后到达
```

为了突出管理逻辑，省略不同层、KV Head、数据类型和物理字节数。

## 第一步：A和B进入

```text
A的输入需要2个Block
B的输入需要1个Block
```

Runtime 分配：

```text
A → Block 1、Block 4
B → Block 2
空闲 → Block 3、5、6、7、8
```

Prefill 执行后，这些 Block 中保存各自输入产生的缓存状态。A 和 B 共享缓存池，却没有共享请求内容。

## 第二步：A和B继续Decode

两者在 [[00-批处理与调度概览|调度器]]安排下参与计算：

```text
A生成新Token → 在自己的最后一个Block追加
B生成新Token → 在自己的最后一个Block追加
```

如果 A 的 Block 4 填满，Runtime 可以再分配 Block 7：

```text
A → [1, 4, 7]
```

虽然物理编号不连续，逻辑顺序仍由映射保持。

## 第三步：C带着相同前缀到达

假设 C 的开头与 A 的第一个完整前缀 Block 完全匹配，并且允许安全复用：

```text
C命中A的前缀Block 1
→ 复用已有前缀状态
→ 为C不同的后续内容分配Block 5
```

这减少 C 的部分重复 Prefill，但 C 的新问题和输出仍要继续计算。C 也不会因此读取 A 后面不同的私有内容。

## 第四步：B完成

B 生成 EOS：

```text
B结束
→ 解除B对Block 2的活动占用
```

Block 2 接下来可以回到可用资源管理；是否保留为可复用前缀，取决于具体 Runtime 和缓存策略。

## 第五步：缓存池接近满载

如果 A、C 继续增长，同时 D 到达，Runtime 可能发现剩余空间不足。

它不能覆盖 A 或 C 仍在使用的 Block，只能在策略允许范围内选择：

- 让 D 等待；
- 驱逐无人活动使用的旧 Prefix Cache；
- 暂停某些请求并在以后恢复；
- 使用支持的 Offload 或低精度缓存；
- 达到服务限制时拒绝请求。

这部分与调度策略一起决定延迟、公平性和吞吐量。

## 整条主线

```text
请求到达
→ 查找Prefix Cache
→ 检查容量并分配Block
→ Prefill写入
→ 调度多轮Decode
→ 按需追加Block
→ 请求完成或取消
→ 解除占用
→ 保留复用、驱逐或重新分配
```

## 谁负责什么

| 对象 | 在案例中的职责 |
|---|---|
| 模型Attention | 使用当前Query与正确历史状态计算 |
| 调度器 | 决定A、B、C哪一轮参与计算 |
| KV Cache Manager | 管理Block归属、增长、复用和回收 |
| Agent或应用 | 决定发送什么上下文并使用模型结果 |

## 常见误解

1. **“C复用A的前缀，就能看到A完整对话。”** 只允许复用匹配且授权的前缀状态。
2. **“B结束后模型也被卸载。”** 通常只处理B的请求状态，模型继续服务。
3. **“Block编号不连续会改变Token顺序。”** Runtime映射维护逻辑顺序。
4. **“缓存管理会让模型学到新知识。”** 它复用运行状态，不更新模型参数。

## 最终理解检查

1. 为什么 C 可以复用部分前缀，却仍需计算自己的后缀？
2. A 的物理 Block 不连续时，什么维持正确历史顺序？
3. B 完成后，“解除活动占用”和“立即清除所有复用信息”为什么不是同一件事？
4. 缓存池不足时，Runtime 为什么必须在多种代价之间选择？

下一专题：[[00-API与流式传输概览|API 与流式传输]]。
