---
layout: post
title:  "BERT: Pre-training of Deep Bidirectional Transformers for Langua Understanding"
categories: paper
tags: paper bert
---
# 摘要
BERT被设计用于预训练深度双向表示，基于无标签文本并以在全层依赖于左右上下文的方式学习。预训练的BERT模型只需一层额外的输出层就可以微调至特定任务，产生SOTA模型，不需要修改模型架构。
# 背景
- 有两种将预训练语言表示应用于下游任务的策略：基于特征(feature-based)和微调(fine-tuning)。
- 单向语言模型限制了模型架构的选择。
# 模型
## 输入/输出
- 输入可以是一个句子或一对句子。句子以[CLS]开头。当输入为一对句子时，句子之间加入[SEP]作为分隔符，合并为一个句子，并为每个token增加一个表示它所属句子的Embedding（学习得到）。
- 句子使用WordPiece embedding  
- [CLS]位置的输出可以用作分类任务的句子表示。
## 预训练
### Masked LM(MLM)
- 直觉上，从左到右的单向语言模型，或一个从左到右和一个从右到左的语言模型的简单连接效果不如深度双向模型。但深度双向模型会使每个词间接地“看到自己”，使得预测变得平凡。
- MLM则随机遮掩一定比例的输入token，预测这些被遮掩的token（文中使用15%）。
- 加入[MASK]标签会导致预训练与下游任务不匹配（因为下游任务没有[MASK]），故遮掩时，将80%替换为[MASK]，10%替换为一个随机token，10%保持不变。
### Next Sentence Prediction(NSP)
选择句子A和B构造数据集，B有50%的比例是A的下一句，有50%的比例是语料库中随机的一句。训练模型预测B是否是A的下一句。
# 其它要点
- OpenAI GPT Radford：et al., 2018提出的一种预训练模型
- WordPiece： Wu et al., 2016
# 备注
-|-
-|-
from scratch|从零开始