---
type: mechanism-note
module: 3
status: complete
audience: non-specialist
parent: "[[00-运行对象与状态机概览|运行对象与状态机概览]]"
tags: [observe, decide, act, loop]
---

# Observe–Decide–Act 循环

> [!summary]
> Agent 每一轮先读取当前状态与环境观察，再让模型提出决策，最后由 Runtime 执行获准动作；动作结果会成为下一轮的新观察。

## 三个环节

### Observe：观察

观察可能来自：

- 用户新消息；
- 文件读取结果；
- 数据库查询；
- 测试输出；
- 浏览器状态；
- 工具错误；
- Runtime 的预算和权限状态。

Observation 是外部系统记录的结果，不等于模型对结果的解释。

### Decide：决策

Context Builder 把相关 Goal、状态和 Observation 交给模型。模型可以提出：

- 返回消息；
- 请求工具；
- 询问用户；
- 修改计划；
- 申请完成；
- 声明无法继续。

这些都是候选决策，需要 Runtime 解析和校验。

### Act：行动

Runtime 根据当前 Phase、权限和参数决定：

```text
执行
拒绝
请求 Approval
转换为等待状态
记录协议错误
```

真实工具结果随后形成新的 Observation。

## 一个两轮例子

```text
Observe 1：测试 `payment_test` 失败
Decide 1：模型请求读取回调和账本代码
Act 1：Runtime 执行只读工具

Observe 2：发现两个入口都会写入账本
Decide 2：模型提出检查幂等键来源
Act 2：Runtime 执行下一项查询
```

## 为什么不一定每轮都有工具

某一轮可能只产生：

- 用户澄清问题；
- 计划提案；
- 最终说明；
- 完成申请。

所以更一般的循环是“观察—决策—处理结果”，Tool Use 只是常见动作之一。

## 关键边界

```text
模型看到的 Observation
→ 可能是经过裁剪的上下文表示

Runtime 保存的真实结果
→ 应保留身份、来源和执行状态
```

不能因为模型在摘要中遗漏了错误，就把真实工具失败改写成成功。
