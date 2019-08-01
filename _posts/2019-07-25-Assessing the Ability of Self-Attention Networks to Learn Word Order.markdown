---
layout: post
title:  "Assessing the Ability of Self-Attention Networks to Learn Word Order"
categories: paper
tags: paper nmt seq2seq word-order mt
---
# 摘要
SAN(Self-attention networks)被认为不善于学习位置信息。但这种观点没有被证实，而且也解释不了为何在缺少位置信息的情况下仍然可以有好的翻译表现。本文提出了一个新的词语重排检测(word reordering detection, WRD)任务，量化SAN和RAN对词序信息的学习能力。该任务随机移动一个词语的位置，用模型检测它插入的位置和原来的位置。实验结果显示：1)即便有positional embedding，在该任务上训练的SAN也很难学习到位置信息；2)在MT任务上训练的SAN能比RNN更好地学习到位置信息。
# WRD任务
在句子$X={x_1,\cdots,x_i,\cdots,x_N}$中，随机抽出一个词语$x_i$并插入到另一个位置j。任务目标时预测位置i和j，分别标注为O和I。
# 模型
## Position Detector
设$H={h_1,\cdots,h_N}$为encoder学习到的序列表示。  
I的概率预测为：  
$$P_I=SoftMax(U_I^Ttanh(W_IH)) \in R^N$$  
O的概率预测为：  
$$E=P_I(W_QH) \in R^d$$  
$$P_O=ATT(E,W_KH) \in R^N$$  
其中$W_Q,W_K \in R^{d\times d}$  
（$P_O$的计算方式有什么道理？ 需要参考Xu et al. 2017)  
## Encoder
分别使用RNN-GRU、SAN、DiSAN
## 目标函数
$$ L=Q_I^T \log P_I + Q_O^T \log P_O$$  
其中$Q_I,Q_O \in R^N$是groundtruth的ohe-hot向量
# 训练
1. 直接在WRD任务上训练
2. 先在翻译任务上训练NMT模型，再固定encoder部分的参数，decoder换成position detector，然后在WRD任务上训练。
3. 句子用BPE表示
# 结论
- 学习目标很重要   
- DiSAN学习词序信息的通用性更好  
- SAN的positional Embedding对NMT和WRD任务都很重要
# 其他要点
- 不同NLP任务对语言学信息的需求不同  
- 对MT，局部模式（如短语）更重要
- 抛弃源语言句子的无关特征对下游NLP任务有好处
# 备注
pointer detection task(Vinyals et al. 2015)  
information bottleneck principle (Tishby and Zaslavsky 2015; Alemi et al. 2016)
