---
layout: post
title:  "Attention is all you need"
categories: "transformer"
tags: transformer
---
# 摘要
此前最好的序列转换模型都基于encoder-decoder模型。本文提出Transformer，完全基于Attention机制。模型在两个MT任务上做了实验，效果很好而且比其他模型并行性更好、训练时间更短。
# 动机
RNN必须顺序计算，不能并行，效率比较差。而Attention机制可以并行，可以无视距离建立依赖（RNN只能建立短距离依赖，LSTM才能建立长距离的。但一般的Attention由于做了平均，弱化了有效分辨率，故后面用Multi-Head Attention抵消这个影响），但过去都是和RNN结合使用。故本文研究了一个完全基于Attention的模型。
# 模型
Transformer也采用encoder-decoder架构。两者都是堆叠自注意力(self-attention)层和全连接层形成。
## Encoder
Encoder有$N=6$层。每层是一个多头自注意力子层(multi-head self-attention)+全连接子层。子层与子层之间用残差连接和一个标准化，即若x为当前子层的输入，则下一子层的输入是： 
$$LayerNorm(x+Sublayer(x))$$  
Sublayer(x)是当前子层的输出。残差连接就是这里额外加上的x。  
Encoder的输入是词语序列，每个词用Word Embedding与Positional Encoding的和表示，输出是同样维度，同样长度的向量序列。  
## Decoder
在Encoder的基础上，每层中间插入了一个多头注意力子层，用encoder的输出作为Key和Value。而原先的注意力子层通过加Mask，避免attend到了还未被计算的位置。  
Decoder的输入是已预测的词语序列，同样用Word Embedding+Positional Encoding表示每个词语。输出是下一个词语的概率。因此若待预测句子长度为L，Decoder也需要计算L次。
## 预测
一个全连接+一个softmax
## Attention层
本文使用的注意力层基于Multi-Head Scaled Dot-Product Attention。
### Scaled Dot-Product Attention
$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V$$  
用点积注意力的原因：点积注意力和加性注意力(additive attention，即用一个单层前馈神经网络计算权重)在理论复杂度上是相似的，点积注意力时间和空间消耗更低。  
用$\sqrt{d_k}$缩放的原因：加性的表现通常比点积更好，作者认为这是因为点积后的数值较大，导致softmax函数的梯度很小，模型不易收敛，故通过缩放抵消这个影响。  
### Multi-Head Attention
多头注意力将Q,K,V经过不同的线性映射做多个Attention，最后将输出连接，再线性映射至原来的维度  
$$MultiHead(Q,K,V)=Concat(head_1,\cdots,head_h)W^O$$  
$$head_i=Attention(QW_i^Q,KW_i^K,VW_i^V)$$  
其中$W_i^Q, W_i^Q \in R^{d_{model}\times d_k}, W_i^V \in R^{d_{model} \times d_v}, W^O \in R^{hd_v \times d_{model}}$  
好处：作者认为这可以从不同的子空间中学习不同的信息。  
本文中h取8，$d_k=d_v=d_{model}/h=64$  
因为每个head的维度减小了，计算量基本与Single Head相同。
### Transformer中的Attention层
`decoder新增的attention层`，Q来自decoder的上一子层，K,V来自encoder的输出  
`encoder的self-attention层`，Q,K,V都来自上一层的encoder输出  
`decoder的self-attention层`，Q,K,V来自decoder的输入，且右边的位置被mask（置为$-\infty$）来模拟RNN版本的decoder。
## Position-wise Feed-Forward Networks
模型中使用的全连接层： 
$$FFN(x)=max(0,xW_1+b_1)W_2+b_2$$  
独立地在每个位置计算（Attention层是所有位置一起计算）。而参数在同一层不同位置相同，在不同层不同。
## Positional Encoding
$$ PE(pos,2i)=\sin(pos/10000^{2i/d_{model}})$$  
$$ PE(pos,2i+1)=\cos(pos/10000^{2i/d_{model}})$$  
这里pos是位置，i是维度，$d_{model}$是PE的维度，这里与原来的词向量维度相同
### 目的
Transformer不是RNN和CNN结构，需要用这种方式利用上序列的位置信息
### 怎么加
在encoder和decoder栈的底部（应该是最先被计算的那一层？？），与对应位置的输入向量相加
# Why Self-Attention
1. 计算复杂度nice  
2. 并行性nice  
3. 长距离序列依赖的计算路径长度是常数
# 训练
## 优化
使用Adam optimizer，此外更新学习率：
$$ lrate=d^{-0.5}_{model}\cdot min(step\_num^{-0.5},step\_num\cdot warmup\_steps^{-1.5})$$  
前warmup_steps（本文=4000）次迭代，学习率逐渐增大，然后逐渐减小。
## Regularization
### Residual Dropout
每个子层的输出都Dropout（在加上残差之前），另外embedding和PE的和也Dropout。（$P_{drop}=0.1$）
### Label Smoothing
是什么？？
# 备注
byte-pair encoding  
Label Smoothing 
