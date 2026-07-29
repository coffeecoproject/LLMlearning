---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-KV-Cache工程管理概览|KV-Cache工程管理概览]]"
previous: "[[05-KV-Cache怎样分配追加释放与驱逐|KV-Cache怎样分配追加释放与驱逐]]"
next: "[[07-内存不足时有哪些处理方式|内存不足时有哪些处理方式]]"
tags: [llm, runtime, kv-cache, prefix-cache, prefill]
---

# Prefix Cache 怎样复用相同前缀

> [!summary]
> 当新请求开头的 Token 序列与已计算内容完全匹配，并且模型与相关运行条件兼容时，Runtime 可以复用对应 KV Cache，跳过这部分重复 Prefill。

> [!phase] 运行阶段：跨请求计算复用

## 为什么会有大量重复前缀

在线服务中，很多请求可能拥有相同开头：

```text
相同System Prompt
+ 相同工具说明
+ 相同长文档
+ 不同的最后问题
```

如果每次都从第一个 Token 重新完成 Prefill，会重复计算相同前缀。

## 一条复用路径

请求 A：

```text
[系统规则][共同文档][问题A]
```

请求 B：

```text
[系统规则][共同文档][问题B]
```

如果前两段形成的 Token 前缀和运行条件能够安全匹配：

```text
找到已计算前缀缓存
→ 复用这部分KV状态
→ 从问题B开始处理不同后缀
```

## “相同”为什么比“意思相近”严格

KV Cache 是具体 Token 在具体模型层中计算出的状态。两段文字即使语义相似，只要 Token 序列、模型版本、位置或相关适配条件不同，就不能想当然地复用同一缓存。

因此 Prefix Cache 不是语义搜索：

```text
语义相近
≠ KV Cache可直接互换
```

## 它主要节省哪一段

Prefix Cache 主要减少重复前缀的 Prefill 计算。新问题之后的 Token 以及新生成的回答仍需正常计算和 Decode。

```text
命中前缀
→ 首Token等待可能减少

继续生成新回答
→ 仍然逐步Decode
```

所以它不等于让整个回答从缓存中直接取出。

## 与上下文和长期记忆的区别

| 对象 | 保存或处理什么 |
|---|---|
| 原始上下文 | Messages、文本、工具结果等逻辑输入 |
| Prefix Cache | 某段已匹配Token前缀的计算状态 |
| Session | 产品保存的对话与任务历史 |
| 长期记忆 | 跨请求筛选和保留的信息 |

Prefix Cache 可以过期或被驱逐，不能作为业务事实的唯一保存位置。

## 跨用户复用的安全边界

如果共享服务允许跨请求复用，必须防止错误匹配、租户越权和通过延迟差异推断他人是否使用过某段内容。具体系统可以采用隔离范围、缓存盐或其他安全设计。

vLLM 当前公开文档说明，其 Prefix Cache 使用包含前缀和当前 Block Token 等因素的哈希识别可复用块，并提供可选 `cache_salt` 来帮助隔离复用范围。

来源：[vLLM Automatic Prefix Caching](https://docs.vllm.ai/en/stable/design/prefix_caching/)，核对日期：2026-07-28。

## 常见误解

1. **“意思相同就能命中Prefix Cache。”** 通常要求具体前缀和相关条件匹配。
2. **“命中后整个回答都不用计算。”** 主要复用已有前缀，新后缀和新输出仍要计算。
3. **“Prefix Cache就是聊天记录。”** 它是临时计算状态，不是可靠业务存储。
4. **“跨用户共享一定安全。”** 必须考虑租户隔离和信息侧信道风险。

## 理解检查

1. Prefix Cache 为什么不是语义相似度搜索？
2. 它主要减少 Prefill 还是 Decode？
3. 为什么不能把 Prefix Cache 当作 Session 的唯一存储？

下一篇：[[07-内存不足时有哪些处理方式|内存不足时有哪些处理方式]]。
