---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-模型加载与请求生命周期概览|模型加载与请求生命周期概览]]"
previous: "[[01-服务启动与请求运行为什么是两个阶段|服务启动与请求运行为什么是两个阶段]]"
next: "[[03-模型怎样从磁盘进入内存与计算设备|模型怎样从磁盘进入内存与计算设备]]"
tags: [llm, runtime, config, weights, tokenizer, generation-config]
---

# Runtime 需要加载哪些模型资产

> [!summary]
> 一个可运行的 LLM 不只有权重文件：Runtime 还需要知道模型结构、Tokenizer 规则、特殊 Token、默认生成设置以及怎样执行这种架构。它们职责不同，必须彼此匹配。

## 先看“模型仓库”而不是一个模型文件

开放权重模型通常以一个目录或仓库出现，其中可能包含：

```text
模型配置
权重文件或权重分片
Tokenizer资产与配置
生成默认配置
模型说明和版本信息
有时还有自定义执行代码
```

这些文件共同描述“怎样还原并使用这个模型”，但不是每个文件都是模型参数。

## 1. 模型配置：说明结构长什么样

模型配置可能记录：

- 模型架构类型；
- 隐藏宽度和层数；
- Attention Head 与 KV Head 数；
- FFN 中间宽度；
- 位置机制和上下文相关设置；
- 数据类型提示与缓存接口设置。

Runtime 或模型库根据这些信息创建相应结构。

```text
只有配置、没有训练权重
→ 可以知道骨架
→ 但没有训练完成的参数内容
```

## 2. 权重：训练形成的参数值

权重文件保存 Embedding、Attention、FFN、Normalization 和 Output Layer 等处学到的数值。

大型模型的权重可能被拆成多个文件：

```text
权重分片1
权重分片2
权重分片3
……
```

这里的“文件分片”主要是存储和加载组织方式，不自动表示模型运行时也按相同边界分布在多张 GPU 上。

## 3. Tokenizer资产：规定文字和Token ID怎样互换

Tokenizer 需要词表、切分规则、字节规则、特殊 Token 和 Chat Template 等信息。

Runtime 加载 Tokenizer，不是在每次收到一句话后重新训练 Tokenizer，而是：

```text
服务启动时选择并建立配套Tokenizer对象
→ 请求到来后反复使用它进行Encode和Decode
```

这也解释了 `AutoTokenizer`：它是根据模型仓库中的配置选择合适实现的加载工具，不是把用户的一句话“分流到不同模型”。

## 4. 生成默认配置：提供可覆盖的起始设置

生成配置可能包含：

- EOS、BOS、PAD 等特殊 Token ID；
- 是否默认 Sampling；
- Temperature、Top-p、Top-k 等默认值。

它们通常是生成控制器使用的默认设置，不是 Transformer 权重。一次请求可以在服务允许的范围内覆盖部分默认值。

```text
模型配置 → 主要描述模型结构
生成配置 → 主要描述默认怎样选择和停止
```

## 5. 执行实现：知道怎样运行这种架构

配置写着 `model_type` 或架构名称，并不等于配置文件本身会执行矩阵计算。模型库或 Runtime 还需要有对应实现：

```text
配置说明“这是什么结构”
→ 软件找到对应模型类或执行后端
→ 创建层并装入权重
```

如果 Runtime 不支持某种新架构，即使权重文件完整，也可能无法直接运行。

## 开放模型观察：Qwen3-8B

Qwen 官方 `Qwen/Qwen3-8B` 仓库提供了不同职责的文件：

- `config.json` 记录 `Qwen3ForCausalLM`、层数、宽度、Attention 与位置相关配置；
- `tokenizer_config.json` 记录特殊 Token 和 Tokenizer 相关设置；
- `generation_config.json` 记录 EOS、Sampling、Temperature、Top-k、Top-p 等默认值；
- 模型卡分别演示 `AutoTokenizer.from_pretrained()` 和 `AutoModelForCausalLM.from_pretrained()`，说明 Tokenizer 与模型是配套但不同的对象。

这些公开文件证明的是 Qwen3-8B 仓库的组织方式，不代表所有模型仓库必须拥有完全相同的文件名和字段。

来源：[Qwen3-8B 模型仓库](https://huggingface.co/Qwen/Qwen3-8B)、[模型配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)、[Tokenizer 配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/tokenizer_config.json)、[生成配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/generation_config.json)，核对日期：2026-07-27。

## 配套错误会怎样

- 配置与权重结构不匹配：形状或参数名称可能无法装入；
- Tokenizer 不匹配：相同 Token ID 会被赋予错误含义；
- 特殊 Token 配置错误：模型可能无法按预期开始或停止；
- Runtime 不支持架构：模型可能无法建立或执行；
- 默认生成配置不同：同一模型可能表现出不同输出风格。

## 常见误解

1. **“下载了权重就等于拥有完整Runtime。”** 仍需要模型实现、配置、Tokenizer 和执行环境。
2. **“Tokenizer是模型权重的一层。”** 它是输入输出转换系统，不属于 Transformer 参数层。
3. **“generation_config训练了模型的性格。”** 它主要提供运行时生成默认值。
4. **“权重分成四个文件，就一定需要四张GPU。”** 文件分片与运行时设备并行不是同一个划分。

## 理解检查

1. 模型配置和模型权重分别回答什么问题？
2. 为什么 Runtime 通常要加载与模型配套的 Tokenizer？
3. `AutoTokenizer` 选择实现为什么不等于把每句话分配给不同 Tokenizer？

下一篇：[[03-模型怎样从磁盘进入内存与计算设备|模型怎样从磁盘进入内存与计算设备]]。
