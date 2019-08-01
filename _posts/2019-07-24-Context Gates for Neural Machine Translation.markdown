---
layout: post
title:  "Context Gates for Neural Machine Translation"
categories: paper
tags: paper nmt mt seq2seq
---
# 摘要
直觉上，内容词的生成更依赖于源语言上下文，功能词的生成更依赖于目标语言上下文。传统NMT没有有效控制源语言和目标语言的影响，容易生成流畅但不够恰当的翻译。本文提出context gates，动态控制源语言和目标语言对目标词生成的影响比例。
# Context Gate
## 计算  
$$z_i=\sigma(W_z e(y_{i-1})+U_z t_{i-1}+C_z s_i)$$  
其中$y_{i-1}$是decoder上一个预测词，$e(·)$是词向量，$t_{i-1}$是RNN上一时刻状态，$s_i$是encoder提取出的向量，$W_z \in R^{n\times n_e}$，$U_z \in R^{n \times n_t}$，$C_z \in R^{n\times n_s}$。
## 怎么用
这里f是RNN单元  
1. source: $t_i=f(We(y_{i-1})+Ut_{i-1}+z_i \circ Cs_i)$
2. target: $t_i=f(z_i \circ (We(y_{i-1})+Ut_{i-1})+Cs_i)$
3. both: $t_i=f((1-z_i) \circ (We(y_{i-1})+Ut_{i-1})+z_i \circ Cs_i)$

# 结论
context gate和GRU、Coverage等机制能形成互补的作用。  
context gate和coverage机制结合使用时，可以改善alignment
# 疑问
用$1-z_i$和$z_i$加权合理吗？（source和target向量可能不是同一个向量空间的。）  
为什么source和target不分别计算两个gate？     
# 启发
比较性能指标时，加上统计显著度测试更有说服力