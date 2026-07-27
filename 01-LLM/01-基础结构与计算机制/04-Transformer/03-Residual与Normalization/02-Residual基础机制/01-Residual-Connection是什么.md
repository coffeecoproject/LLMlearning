---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Residual基础机制概览|Residual基础机制概览]]"
previous: "[[00-Causal-Self-Attention概览|Causal-Self-Attention概览]]"
next: "[[02-为什么不能只保留子层新结果|为什么不能只保留子层新结果]]"
tags: [llm, transformer, residual-connection, hidden-state]
---

# Residual Connection 是什么

> [!summary]
> Residual Connection（残差连接）让输入 Hidden State 绕过一个子层形成直接通路，再与该子层产生的变化逐元素相加。

## 先定位：它在 Transformer Block 里面

一个常见 Transformer Block 可以先画成：

```text
Transformer Block
├── Attention 子层
│   └── 外围有一条 Residual Connection
└── FFN 子层
    └── 外围有一条 Residual Connection
```

更准确地看数据流：

```text
Block 输入
→ Attention 分支与第一条 Residual 汇合
→ FFN 分支与第二条 Residual 汇合
→ Block 输出
→ 进入下一个 Transformer Block
```

所以 Residual：

- 不是 Transformer Block 外面的单独阶段；
- 不是 Attention 内部计算 QKV 的一步；
- 是 Block 内部围绕子层建立的直接通路和相加点；
- 会在模型堆叠的每个 Block 中重复出现。

常见模型有很多个 Block，因此整套模型中通常不止两条 Residual；准确说是**每个常见串行 Block 内通常有两条**。

## 先不要把它想复杂

假设你有一份已经写到一半的文档：

```text
当前文档 = x
审阅后提出的修改 = F(x)
应用修改后的文档 = x + F(x)
```

Residual 的核心直觉不是“重新写一份文档”，而是：

```text
保留当前版本
+
加入这一步计算得到的修改
```

如果修改意见是“这一处不用改”，对应的变化可以接近 0；如果需要加强或减弱某些内部特征，变化中可以出现正值或负值。

> [!warning] 类比边界
> 真实模型不会把向量解释成自然语言修改指令，也不是先写出一张修改清单再执行。子层直接产生一个数值张量，并与输入张量逐元素相加。

## 从 Attention 的输出接着看

前面已经学过，Attention 会让每个位置汇集允许读取的上下文信息。假设某个位置进入 Attention 前的表示是：

```text
x = [1.0, 2.0, 1.0]
```

Attention 子层根据上下文计算出：

```text
Attention 输出 = [0.2, 0.5, -0.1]
```

Transformer 通常不会简单丢弃 `x`，只留下 Attention 输出，而会执行：

```text
[1.0, 2.0, 1.0]
+
[0.2, 0.5, -0.1]
=
[1.2, 2.5, 0.9]
```

这条“输入直接绕过子层，再与子层结果相加”的路径，就是 Residual Connection，也常被称为 Skip Connection（跳跃连接）。

> [!note] 教学数字
> 上面的向量是人为设计的小例子，不是从真实模型中读取的 Hidden State。

## 为什么叫“Residual”

`Residual` 可以理解为“残差”或“增量”。子层不必从头重建一个全新表示，而可以学习：

```text
在当前表示基础上，需要增加、减弱或改变什么
```

用简化记号表示：

```text
y = x + F(x)
```

其中：

| 记号 | 含义 |
|---|---|
| `x` | 进入子层前的 Hidden State |
| `F(x)` | Attention 或 FFN 等子层根据输入算出的变化 |
| `y` | 相加后传给后续结构的新 Hidden State |

`F(x)` 不是某个叫作“F”的固定模块，只是对“子层计算结果”的简写。

## “绕过”不是不经过子层

Residual 形成两条同时存在的数据路径：

```text
路径 A：x ───────────────────────┐
                                ├→ 相加 → y
路径 B：x → 子层计算得到 F(x) ───┘
```

路径 A 直接保留输入，路径 B 产生新的变化。二者最终相加，而不是二选一。因此不能把 Residual 理解成“为了省计算，跳过 Attention”。Attention 或 FFN 仍然执行。

可以把它压缩成四个动作：

```text
1. 留住 x
2. 用 x 计算 F(x)
3. 把 x 和 F(x) 对齐
4. 逐项相加得到 y
```

其中第 1、2 步是并行分出的两条路径，第 3、4 步再让它们汇合。

## 相加具体怎样发生

Residual 通常是同一位置、同一维度的数值逐项相加：

```text
x：    [x₁, x₂, x₃]
F(x)： [f₁, f₂, f₃]
y：    [x₁+f₁, x₂+f₂, x₃+f₃]
```

如果整条序列的 Hidden States 形状为：

```text
[sequence_length, hidden_size] = [3,4]
```

那么 Residual 分支和子层输出通常也需要保持 `[3,4]`，才能按位置、按维度相加：

```text
[3,4] + [3,4] → [3,4]
```

这也是 Attention 多个 Head 合并后还需要 Output Projection 的原因之一：它把合并结果投影回 Residual 通路所需的 `hidden_size`。

## Residual 是拼接吗

不是。

```text
相加：[3,4] + [3,4] → [3,4]
拼接：[3,4] 与 [3,4] → 可能变成 [3,8]
```

常见 Residual Connection 保持 Hidden State 的整体形状不变。形状不变不代表内容没变，因为各维数值已经加入了子层产生的变化。

## 三个动作不要混淆

| 动作 | 直观结果 | 是否属于常见 Residual 核心操作 |
|---|---|---|
| 替换 | 只留下 `F(x)` | 否 |
| 拼接 | 把 `x` 与 `F(x)` 接成长向量 | 否 |
| 相加 | `x` 与 `F(x)` 同位置、同维度相加 | 是 |

看到 Residual 时，先问两个问题就够了：

```text
旧主干是谁？
哪个子层结果要加回来？
```

## Residual 自己有可学习参数吗

最基础的 Residual 相加本身通常没有像 Q、K、V 投影那样的权重矩阵：

```text
Residual 的核心操作：直接相加
```

但是 Residual 分支周围的 Attention、FFN、Normalization 或某些架构中的缩放机制可能拥有参数。不能因为整个结构附近存在参数，就把它们都归为 Residual 参数。

## 阶段边界

> [!info] 两阶段共同
> 训练和普通运行都会执行 Residual 前向相加。训练阶段可以更新周围子层的参数；普通运行阶段读取固定参数，但 Hidden State 仍会逐层变化。

Residual 不是 KV Cache，也不是 Agent 的对话记忆：

```text
Residual
→ 同一次模型前向计算中，连接相邻子层的表示通路

KV Cache
→ 模型运行时缓存过去 Token 的 K、V

Agent Memory
→ Agent 系统保存的对话、任务或外部信息
```

## 常见误解

- **“Residual 就是把前一层完整复制到最后。”** 它把输入加入当前子层结果，之后仍会继续经过后续子层和更多 Block。
- **“有 Residual 就可以不执行 Attention。”** 两条路径同时存在，Attention 仍需计算 `F(x)`。
- **“Residual 是把两个向量拼起来。”** 常见实现是逐元素相加，形状通常保持不变。
- **“Residual 自己是一张大参数矩阵。”** 基础相加通常没有独立的大权重矩阵。
- **“Residual 是模型的长期记忆。”** 它只是当前前向计算中的数据通路。

## 理解检查

1. 在 `y=x+F(x)` 中，`x`、`F(x)` 和 `y` 分别表示什么？
2. Residual 的“绕过子层”为什么不等于“不执行子层”？
3. `[3,4]` 的输入和 `[3,4]` 的子层输出相加后，为什么通常仍是 `[3,4]`？
4. Residual 与 KV Cache 的时间范围和职责有什么不同？

下一篇：[[02-为什么不能只保留子层新结果|为什么不能只保留子层新结果]]。
