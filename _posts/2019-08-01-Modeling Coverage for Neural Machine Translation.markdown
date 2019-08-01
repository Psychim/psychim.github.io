---
layout: post
title:  "Modeling Coverage for Neural Machine Translation"
categories: paper
tags: paper nmt
---
# 摘要
注意力机制容易忽视过去的对齐信息，导致过额翻译或欠额翻译。本文提出coverage mechanism，通过维护一个coverage向量，跟踪注意力历史，提高NMT系统对未翻译的词语的考虑。
# 动机
- attention没有考虑翻译的历史信息。
# 模型
## General Model
$$C_{i,j}=g_{update}(C_{i-1,j},\alpha_{i,j},\Phi(h_j),\Psi)$$  
- $g_{update}(\cdot)$是解码过程中序列第i步的注意力$\alpha_{i,j}$产生后，coverage向量的更新函数。
- $C_{i,j}$是d维的coverage向量，总结了第i步为止，对$h_j$的注意力历史
- $\Psi$是辅助输入

## Linguistic Coverage Model
$$C_{i,j}=C_{i-1,j}+\frac{1}{\Phi_j}\alpha_{i,j}=\frac{1}{\Phi_j}\sum^i_{k=1}\alpha_{k,j}$$  
- $\Phi_j$是预定义的权重，表示词语$x_j$期望被翻译的次数。可定义为：$\Phi_j=N\cdot \sigma(U_fh_j)$。其中N为最大次数，$\sigma(\cdot)$是sigmoid函数，$U_f$是权重矩阵。

## Neural Network Based Coverage Model
可用一个RNN来迭代$C_{i,j}$  
$$C_{i,j}=f(C_{i-1,j},\alpha_{i,j},h_j,t_{i-1})$$

## Integrating Coverage into NMT
修改attention的计算：  
$$
\begin{aligned}
e_{i,j}&=a(t_{i-1},h_j,C_{i-1,j})\\
&=v_a^Ttanh(W_at_{i-1}+U_ah_j+V_aC_{i-1,j})
\end{aligned}
$$  
## Objective
加入一个辅助目标函数显式学习$\Phi$：
$$-\lambda\left\{\sum^J_{j=1}(\Phi_j-\sum^I_{i=1}\alpha_{i,j})^2; \eta\right\}$$  
这可以改善对齐质量但会降低翻译质量。
# 评估
## SAER(S??? Alignment Error Rate)
$$SAER=1-\frac{|M_A\times M_S|+|M_A \times M_P|}{|M_A|+|M_S|}$$
- A是候选对齐，S和P是确定和可能的连接集合（人为标注，且$S \subseteq P$）,M表示对齐矩阵。

# 其它要点
- Moses(Koehn et al., 2007)，一个基于短语的SMT系统
- GroundHog（Bahdanau et al., 2015），将attention引入Seq2Seq的那个模型
