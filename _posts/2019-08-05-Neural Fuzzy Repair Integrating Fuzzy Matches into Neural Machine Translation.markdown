---
layout: post
title:  "Neural Fuzzy Repair: Integrating Fuzzy Matches into Neural Machine Translation"
categories: paper
tags: paper nmt data-augmentation mt tm
---
# Reference
None
# 摘要
本文提出了一个数据增广方法，利用模糊翻译记忆（Translation Memory）匹配（fuzzy TM matches）提升NMT的表现。
# 背景
- 翻译记忆(TM)，猜测是一个数据库，包含了一批（源语言句子，最佳翻译）
- TM与MT的集成
  - 利用基于例子的MT系统（Simard and Langlais, 2001）
  - 编辑TM匹配（editing high-scoring TM matches, 也叫fuzzy repair）（Hewavitharana et al., 2005; Ortega et al., 2016）
  - PBSMT中
    - 强制让输出必须包含搜索到的TM匹配的一部分（Koehn and Senellart, 2010）
    - 扩充短语表（Bicici and Dymetman, 2008; Simard and Isabelle, 2009）
    - 适配PBSMT系统本身（Wang et al., 2013）。
  - NMT中
    - 增加一个词汇记忆（Feng et al., 2017）
    - 搜索算法中的词汇限制（Hokamp and Liu, 2017）
    - 指导NMT输出的附属于搜索到的和匹配的translation pieces的奖励（Zhang et al., 2018）
    - 解码中，显式提供NMT系统一个搜索到的TM匹配列表（Gu et al., 2018）
    - 为搜索到的TM匹配增加一个额外的encoder（Cao and Xiong, 2018）

# Neural Fuzzy Repair
TM系统包含源句和目标句对(S,T)的集合，该集合直接用作MT系统的训练数据。此外，每个源句$s_i$与其它所有源句$s_j$比较相似度（采用词级别的编辑距离），高于一个阈值$\lambda$的集合$S_i' \subset S$，与它们的目标句集合$T_i' \subset T$一起存于集合$F_{s_i}$。另外排除相似度为100%的句子。  
作者用3种方式加速计算：
1. python的SetSimilaritySearch库先选中候选相似句集合，再在该集合内算编辑距离。集合根据  
   $$containment_{max}(v_i,v_j)=\frac{\parallel v_i \cap v_j\parallel}{max(\parallel v_i\parallel,\parallel v_j \parallel)}$$  
   选取，其中$v_i$和$v_j$是句子i和j的one-hot向量。  
2. 只计算集合中的n-best候选的编辑距离  
3. 多线程  

然后可以连接$s_i@@@t_1'[@@@t_2'[@@@t_3']]$增广源句子，目标句子都使用$t_i$。$t_j'$是第j个最相似源句的目标句。@@@是break分隔符。也可采用$s_i@@@t_j', j\leq 3$的形式。  
测试集中的句子，采用$s_i@@@t_1'$作为输入。