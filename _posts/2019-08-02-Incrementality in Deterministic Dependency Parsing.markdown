---
layout: post
title:  "Incrementality in Deterministic Dependency Parsing"
categories: paper
tags: paper dependency-parsing
---
# Reference
[Incrementality in Deterministic Dependency Parsing](https://www.aclweb.org/anthology/W/W04/W04-0308.pdf)
# 摘要
本文分析确定性依赖解析对增量处理的潜力。本文得到的结论是严格增量无法在这个框架下达成，但是，通过选择最优的解析算法，可以最小化要求非增量处理的结构数量。
# 背景
- 支持增量解析的两个原因：(1)实时应用的要求；(2)能将解析与认知建模联系起来。
- 大部分最先进的解析方法都不是增量的。例如**完全解析**首先使用动规生成解析树森林，属于非确定性解析，无法增量解析。
>完全解析、完全句法分析(full parsing)，指完全对输入消岐的解析。
- 部分解析（只部分消岐）通常是确定性的，只遍历一轮输入构建最终的分析。但输出的结果一般是未连接的短语或块序列，不满足增量的限制。
# Dependensy Parsing
- 每个词语最多依赖于一个其它词语，称为head或regent。
- projective dependency graph: 带标签的有向图$D=(W,A)$。
  - $W$是节点集合，即输入字符串中的词语；A是弧集合$(w_i,w_j) (w_i,w_j \in W)$。
  - $w_i<w_j$表示句子中$w_i$在$w_j$之前。$w_i \rightarrow w_j$表示有向弧，$\rightarrow^*$表示弧关系的自反和传递闭包。$\leftrightarrow$和$\leftrightarrow^*$表示相应的无向弧。
  - 满足以下条件时，称其是well-formed
  - 1. Unique Label, $(w_i \rightarrow^r w_j \bigwedge w_i \rightarrow^{r'} w_j) \Rightarrow r=r'$
  - 2. Single head, $(w_i\rightarrow w_j \bigwedge w_k \rightarrow w_j) \Rightarrow w_i=w_k$
  - 3. Acyclic, $\lnot(w_i\rightarrow w_j \bigwedge w_j\rightarrow^* w_i)$
  - 4. Connected, $w_i \leftrightarrow^* w_j$
  - 5. Projective, $(w_i \leftrightarrow w_k \bigwedge w_i<w_j<w_k)\Rightarrow (w_i \rightarrow^*w_j \bigwedge w_k \rightarrow^*w_j)$

# Incrementality in Dependency Parsing
- Incrementality最严格地定义为：在解析过程中，只存在一个连接图表示到目前输入为止的解析。
- incrementality不等于determinism，incrementality允许回溯但determinism不允许。
- 本文考虑left-to-right bottom-up parsing，一种最直接的解析策略，在这情况下等价于shift-reduce parsing with a context-free grammar in Chomsky normal form（见[dependency parsing笔记]内的Transition-Based Dependency Parsing）。
  - 为了使这种parsing是deterministic的，parser必须在最多2n(句长)步内终止（无论使用任何机制），且必须生成acyclic和projective的依赖图。
  - 在这个框架下，要实现incremental parsing，栈内元素不能超过2个。这种解析方式无法得到所有可能的解析图，故determinisitic dependency parsing不能实现incremental parsing。
  - 问题在于，incremental parsing要求最左边的元素直接连接，以及一个元素的dependents全部连接后，才能将该元素连接至它的head。

# Arc-Eager Dependency
- head和dependent可用时，就将该arc加入依赖图。
- 定义四种transition
  - Left-Arc: 加入$w_j \rightarrow^r w_i$，$w_j$是下一个输入token，$w_i$是栈顶元素，然后将$w_i$出栈。
  - Right-Arc: 加入$w_i \rightarrow^r w_j$，$w_i$是栈顶元素，$w_j$是下一个输入token，然后将$w_j$入栈。
  - Reduce: 栈顶元素出栈
  - Shift(SH): 下一个token入栈
  - Left-Arc和Right-Arc在转移时必须满足Single Head约束。
  - Reduce转移只能在栈顶元素已经有head的时候使用。
- 这种情况下，incrementality要求图$(S,A_S)$总是连接的，而$A_S=\{(w_i,w_j)\in A|w_i,w_j \in S\}$。
- Arc-Eager Dependency仍然无法解决一部分结构的incremental parsing问题。(即$a\rightarrow c,c\rightarrow b$和$c \rightarrow  a, c \rightarrow b$)
- top-down parsing指head先与dependent连接，dependent再和它的dependent连接
- bottom-up parsing指dependent先和head连接，head再和head的head连接。
# 结论
- 确定性依赖解析无法完成严格增量解析。