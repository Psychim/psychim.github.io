---
layout: post
title:  "Bridging the Gap between Training and Inference for Neural Machine Translation"
categories: paper
tags: paper nmt mt
---
# Reference
[Wen Zhang et al., 2019](https://www.aclweb.org/anthology/P19-1426)
# 摘要
NMT在训练时基于ground truth，而预测时又从头开始生成序列，这种不一致会导致误差积累。另外，词级别的训练要求生成序列和ground truth严格匹配，会导致过度矫正。本文通过从ground truth和模型生成的预测序列采样上下文词语解决这些问题。预测序列是根据句级别的最优解选择。
# 背景
- 训练时和预测时的差异导致了生成序列产生自不同的分布（训练-数据分布，预测-模型分布），称作exposure bias(Ranzato et al., 2015)。
- 过度矫正问题例子，比如ground truth是"comply with"时，"comply"处生成了"abide"，词级别的目标函数为了产生较高的似然函数值，下一个词仍然会倾向于生成"with"，从而生成了"abide with"这一错误的搭配，而此处如果生成"abide by"本来也是正确的。再比如，训练时模型可能会记住"the rule"总是在"with"之后的模式，而"abide by"之后就可能会错误地生成"the law"。
- Overcorrection Recovery：即便模型生成了短语"abide by"，预测下一个词语时，仍然输入"with"而不是"by"。

# 方法
1. 选择oracle word
2. 以概率p在ground truth和oracle word之间采样
3. 用采样词作为下一个预测的输入。

## Oracle Word Selection
- oracle word是与ground truth相似的词或是其同义词
- Word-level Oracle(WO)：用词级别的贪心搜索得到的oracle word
  - 即直接选预测概率最高的词
  - 也可以采用Gumbel-Max技巧进行采样，得到一个更鲁棒的WO
    - $\eta=-\log(-\log u)$
    - $\tilde{o}_{j-1}=(o_{j-1}+\eta)/\tau$
    - $\tilde{P}_{j-1}=softmax(\tilde{o}_{j-1})$
    - 其中u服从均匀分布，$\tau$是温度，越小$\tilde{P}_{j-1}$越接近原始分布，越大越接近均匀分布
    - 
- Sentence-level Oracle(SO)：用beam search生成序列，再用BLEU等句级别的指标对序列排名，得到的BLEU最高的序列称为oracle sentence，翻译中的词语称为Sentence-level Oracle
  - force decoding：强制让序列的长度与ground truth相同，序列长度短于ground truth时不生成EOS，相等时强制生成EOS。
  - 也可以加入Gumbel-Max

## Sampling with Decay
- 训练前期，采样ground truth的概率要大，后期要小
- $p=\frac{\mu}{\mu+\exp(e/\mu)}$