---
type: mechanism
module: 3
learning_layer: persistent-high-reliability
status: complete
audience: non-specialist
parent: "[[00-上下文构建与工作记忆概览|上下文构建与工作记忆概览]]"
previous: "[[02-系统指令Goal历史与工具结果怎样排序|系统指令、Goal、历史与工具结果怎样排序]]"
next: "[[04-Retrieval摘要与Compaction|Retrieval、摘要与 Compaction]]"
tags: [agent, context-package, context-manifest, provenance, observability]
---

# Context Package 与 Context Manifest

> [!summary]
> Context Package 是为一次 Model Call 准备的结构化输入；Context Manifest 是对这次构建过程的记录，用来解释选了什么、用了哪个版本、删了什么以及模型当时可能看到了什么。

> [!note] 整篇层级：高可靠理想结构（选读）
> 基础 Agent 可以直接由程序构造消息与 Tool Schema。Context Package 是帮助复杂系统保留结构的逻辑模式，Context Manifest 是调试、审计和评估增强；两者都不是 Agent 或 Context Builder 成立的必要对象。

## 为什么不应把 Context 理解成一条长字符串

最终进入 LLM 的确是一段 Token 序列，但在进入 Chat Template 和 Tokenizer 之前，Agent Runtime 需要保留更丰富的结构：

~~~text
哪些是指令
哪些是当前 Goal
哪些是权威状态
哪些是工具结果
哪些是假设
哪些是外部资料
哪些工具本次可用
哪些内容被裁剪
~~~

如果过早把所有内容拼成一条字符串，后续就很难：

- 检查来源和版本；
- 调整不同区块的顺序；
- 单独删除敏感字段；
- 判断某条命令来自可信规则还是外部数据；
- 记录本次到底选择了什么；
- 为不同模型接口生成不同请求格式。

因此可以先建立 Context Package，再由 Context Compiler 转换成具体 Model Request。

## Context Package 是什么

Context Package 是本教程使用的逻辑名称，不是行业统一 API。它可以理解为：

> 一次 Model Call 所需信息的、带角色和来源的结构化集合。

一个说明性结构如下：

~~~yaml
identity:
  thread_id: thread-8
  turn_id: turn-12
  goal_id: goal-3
  run_id: run-4
  model_call_id: call-27

purpose:
  decide_next_action

instructions:
  - ref: runtime-policy-v5
  - ref: project-rules-v2

goal:
  objective: 修复重复扣款
  success_criteria:
    - 并发回调不产生第二次扣款
    - 正常支付流程保持通过
  scope:
    - 支付回调与内部数据库约束

state_view:
  run_status: running
  current_step: 检查事务边界
  latest_observation_ref: obs-41

current_turn:
  user_input_ref: item-88

working_memory:
  - memory-9
  - memory-14

resources:
  - artifact-27

tools:
  - read_file
  - search_code
  - run_test

budget:
  input_tokens: 18000
  reserved_output_tokens: 4000
~~~

这是说明性小例子，字段和数值不是某个真实产品的固定配置。

## 为什么使用引用而不是复制全部内容

Package 中既可以内联短文本，也可以保存对外部记录的引用。

~~~text
短 Goal 摘要
→ 可以直接内联

大型测试报告
→ 保存 artifact 引用和相关片段

完整代码库
→ 保存当前查询结果或文件范围，不直接复制整个仓库

审批记录
→ 引用权威审批对象，而不是复制一句“已批准”
~~~

引用可以减少重复和失真，但在真正调用模型前，Compiler 仍要解析需要的内容。若引用指向的对象会变化，还要记录版本、哈希或读取时间。

## Context Manifest 是什么

Context Manifest 是对一次构建结果的清单。它主要服务于：

- 调试：模型为什么没有使用某条信息；
- 审计：模型当时是否接触过敏感资料；
- 评估：某种裁剪或检索策略是否稳定；
- 恢复：中断前最后一次调用使用了哪些状态版本；
- 成本分析：哪些区块消耗了最多 Token。

一个说明性 Manifest 可以记录：

~~~yaml
model_call_id: call-27
builder_version: context-builder-3
compiler_version: request-compiler-2
created_at: 2026-08-04T10:30:00+08:00

included:
  - source: goal-3
    version: 4
    role: goal
  - source: obs-41
    version: 1
    role: observation
  - source: artifact-27
    fragment: lines-120-180
    role: evidence_candidate

excluded:
  - source: summary-6
    reason: stale
  - source: log-frontend-9
    reason: irrelevant

transforms:
  - source: artifact-27
    operation: extract_fragment
  - source: history-turns-1-8
    operation: use_verified_summary

tool_schemas:
  - read_file@2
  - run_test@3

token_usage:
  estimated_input: 14200
  actual_input: 14610
  reserved_output: 4000
~~~

Manifest 不要求把所有原始 Prompt 永久复制一份。它可以只保存来源标识、版本、哈希、转换方式和受控摘要。

## Package 与 Manifest 的区别

| 对象 | 主要用途 | 是否给模型看 |
|---|---|---|
| Context Package | 准备一次模型调用的结构化输入 | 经过 Compiler 后，相关内容会进入请求 |
| Model Request | 某个模型接口实际接收的消息、工具和参数 | 是 |
| 最终 Token Context | Chat Template 和 Tokenizer 后的有效输入 | 是，模型实际计算它 |
| Context Manifest | 记录本次构建选择、版本与裁剪 | 通常不需要 |
| Event Log | 记录运行过程中发生过什么 | 只有被再次选择的部分才进入 Context |

## 一次生命周期

~~~text
确定 Model Call 目的
→ 读取 Goal、State、Turn 和候选资料
→ 构建 Context Package
→ 生成 Context Manifest 初稿
→ Compiler 形成 Model Request
→ Tokenize 前检查预算
→ 必要时裁剪并更新 Manifest
→ 调用模型
→ 记录实际 Token 用量和调用结果
~~~

Manifest 应描述最终实际采用的选择，而不只是裁剪前的计划。

## Package 需要满足哪些基本条件

### 身份明确

必须知道这次调用属于哪个 Goal、Run 和 Turn，避免跨任务拿错上下文。

### 目的明确

“决定下一步工具”“解释失败”和“生成最终回复”所需内容不同。

### 来源可追踪

重要事实应能回到原始 Observation、用户决定或外部权威记录。

### 版本可判断

Goal revision、Run State、文件和摘要都可能变化。Package 不应把不同版本混成一份无时间信息的文本。

### 信任边界保留

不可信网页、日志和用户引用材料不能在编译时变成高权限指令。

### 预算完整

工具 Schema、附件、图片表示和预留输出都可能消耗窗口，不能只统计聊天正文。

## 可重现不等于必然完全复现

Manifest 可以帮助重建“当时使用了哪些输入”，但不一定保证以后得到完全相同的模型输出，因为：

- 模型版本可能变化；
- 采样可能具有随机性；
- 外部引用可能被修改；
- 服务端模板和协议可能升级；
- 某些隐私数据可能按策略不再保留。

因此 Manifest 的主要目标是可解释、可比较和可审计，而不是承诺所有运行都能逐 Token 重放。

如果任务要求更强重现性，就要额外固定模型版本、采样参数、工具版本、资源哈希和编译器版本。

## 隐私与 Secrets 边界

记录 Context Manifest 也可能产生风险：

- Manifest 可能暴露敏感资源名称；
- 原始 Prompt 可能包含个人信息；
- 工具 Schema 可能泄露内部能力；
- 长期日志可能让已删除内容继续存在。

因此应区分：

~~~text
为了调试需要知道“使用了某个凭证”
≠ 需要记录凭证的真实值

为了审计需要知道“读取了客户文件”
≠ 必须永久保存完整客户文件副本
~~~

可以使用脱敏、哈希、受限引用、保留期限和访问控制。

## 常见误解

### 误解一：Context Package 就是 Prompt 字符串

Package 保留角色、来源、版本和结构；Prompt 字符串只是某种接口下的编译结果。

### 误解二：有了 Manifest 就不需要 Event Log

Manifest 解释一次模型调用看到了什么；Event Log 记录整个 Run 发生了什么，时间范围和职责不同。

### 误解三：为了可审计必须保存全部敏感输入

可以保存来源、版本、哈希和转换记录。审计深度要与隐私风险平衡。

### 误解四：Package 字段形式合法就说明内容正确

Schema 只能检查结构。Goal 是否合理、Observation 是否过期、摘要是否失真仍需要语义和来源检查。

## 理解检查

1. 为什么应先构建结构化 Package，再编译成具体模型请求？
2. Context Manifest 与 Event Log 分别回答什么问题？
3. 为什么 Manifest 记录了全部来源仍不一定能完全复现一次回答？
4. 大型测试报告为什么适合使用带版本引用和相关片段，而不是每次完整复制？

## 本篇停止点

如果你能区分“给模型准备的 Package”和“记录构建过程的 Manifest”，并能说明来源、版本、裁剪与隐私为什么需要单独记录，本篇机制已经掌握。

下一篇：[[04-Retrieval摘要与Compaction|Retrieval、摘要与 Compaction]]。
