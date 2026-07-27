---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN概览|FFN概览]]"
previous: "[[02-FFN参数量与规模|FFN参数量与规模]]"
next: "[[01-SwiGLU与GeGLU|SwiGLU与GeGLU]]"
tags: [llm, ffn, swiglu, moe, optional]
---

# FFN 扩展结构

> [!summary]
> 在基础 `升维 → 非线性 → 降维` 之上，现代模型常加入门控分支，或把一个 Dense FFN 扩展成由 Router 选择少数 Expert 的 MoE。

> [!tip] 阅读前提
> 这是深入扩展。若还不清楚 FFN 的基本作用，先返回 [[00-FFN框架速览概览|FFN 一页看懂]]；若需要理解内部过程，再读 [[00-FFN基础机制概览|FFN 基础机制]]。

## 扩展地图

```text
基础 Dense FFN
├── 门控结构
│   ├── SwiGLU
│   └── GeGLU
│
└── 参数路径扩展
    └── MoE
        ├── Router
        ├── Expert
        ├── Top-k
        └── 总参数与每 Token 激活参数
```

## 阅读顺序

1. [[01-SwiGLU与GeGLU|SwiGLU 与 GeGLU]]
2. [[02-MoE基础与Dense-FFN对比|MoE 基础与 Dense FFN 对比]]
3. [[03-Router-Expert与Top-k|Router、Expert 与 Top-k]]
4. [[04-MoE总参数与每Token激活参数|MoE 总参数与每 Token 激活参数]]

## 两种扩展改变的不是同一层级

```text
SwiGLU / GeGLU
→ 改变一套 FFN 内部怎样计算中间特征

MoE
→ 准备多套 Expert FFN，并选择其中少数路径
```

一个 MoE Expert 内部仍然可以采用 SwiGLU。因此不能把 SwiGLU 和 MoE 当成互斥选项。

下一篇：[[01-SwiGLU与GeGLU|SwiGLU 与 GeGLU]]。
