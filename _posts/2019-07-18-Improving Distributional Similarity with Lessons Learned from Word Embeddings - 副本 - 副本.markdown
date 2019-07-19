---
layout: post
title:  "Improving Distributional Similarity with Lessons Learned from Word Embeddings"
categories: paper
---
# 摘要
本文揭示了词向量的优秀表现源于特定的系统设计和超参数优化，而非算法本身。而且，这些方法可以被挪用至传统分布式模型，得到相似的优化。本文还观察到不同方法之间的表现差异大部分是局部的、不显著的。
# 前置
## word和context word
在统计共现对时，通常取句子中的一个词语$w_i$，与它周围一定长度的窗口内的另一个词语$w_j$记作一个共现对$(w_i,w_j)$，这里$w_i$就算作word，$w_j$算作context word。两者的集合可是不同的词表，可有不同的词向量。  
另外，这种context的定义叫作bag-of-word contexts
## PMI(Pointwise Mutual Information)矩阵

$$PMI(x;y)=\log{\frac{p(x,y)}{p(x)p(y)}}=\log{\frac{p(x|y)}{p(x)}}=\log{\frac{p(y|x)}{p(y)}}$$
如果x和y无关，则$p(x,y)=p(x)p(y)$；否则，若相关度越高，$\frac{p(x,y)}{p(x)p(y)}$越大。
故该值衡量了是x和y的相关度。  
共现矩阵是由元素$X_{ij}$构成的矩阵，$X_{ij}$为词语j出现在词语i的上下文中的次数（或词语i与上下文词语j共现的次数）。
PMI矩阵由共现矩阵得到：
$$PMI(i,j)=\log{\frac{\hat{P(i,j)}}{\hat{P(i)}\hat{P(j)}}}=\log{\frac{X_{ij}|D|}{X_iX_j}}$$
|D|是共现对总数。
而`PPMI(positive PMI)`为：$PPMI(i,j)=max(PMI(i,j),0)$
`PPMI`在语义相似性任务上的表现要好于`PMI`。但两者都有一个缺点：即对小概率事件的偏好。
## SVD(Singular Value Decomposition)
trancated SVD可对矩阵降维得到词向量。  
SVD将矩阵M分解为$U\cdot\Sigma\cdot V^T$，其中U、V正交，$\Sigma$是特征值对角阵，降序排列。通过只保留$\Sigma$的前d个元素可实现矩阵的降维。新矩阵$M_d=U_d\cdot\Sigma_d\cdot V^T$可保留原矩阵的大部分信息。  
$W=U_d\cdot\Sigma_d$可作为词向量的行，因为该矩阵行之间的点乘与$M_d$的行之间点乘相同。而$V_d$的行可作为上下文向量。
## SGNS(Skip-Grams with Negative Sampling)
可见Mikolov两篇论文
# 超参数的转移
本章描述了将SGNS和GloVe的一些机制抽象超参数，并运用于PMI上的方法。  
分为三种超参数：pre-processing hyperparameters; association metric hyperparameters; post-processing hyperparameters
## pre-processing hyperparameters
影响算法的输入数据
### Dynamic Context Window(dyn)
可按权重统计共现，权重随词对距离的增大而衰减。如GloVe按$\frac{1}{d}$衰减；word2vec按$\frac{L-d+1}{L}$衰减。（L为窗口长度，d为词距）
### Subsampling(sub)
降低频繁词的采样率。按$p=1-\sqrt{\frac{t}{f}}$移除词语。t是一个阈值（推荐取$10^{-5}$），f为词语在语料库中的频率。若词语的移除在统计共现对之前进行，若窗口只算没被删除的单词，称为dirty；若仍然算被删除的单词称为clean。两者表现差不多。
### Deleteing Rare Words(del)
删除稀少的词语。
## Association Metric Hyperparameters
定义词语-上下文关系的计算
### Shifted PMI(neg)
negative sampling的采样数k等价于移动SGNS的隐含优化目标：$PMI(w,c)-\log k$。故可得到$SPPMI(w,c)=max(PMI(w,c)-\log k,0)$  
negative sampling中的k起到两个作用：更好地估计负样本的分布，k越大意味着更多的数据和更好的估计；影响正样本的先验概率，k越大说明负样本的可能性越大。SPPMI只能起到第二个作用。
### Context Distribution Smoothing(cds)
通过取指数平滑context的概率分布：$\hat{P_\alpha(j)}=\frac{X_j^\alpha}{\sum_jX_j^\alpha}$  
平滑通过增大了稀少上下文的概率，减小了相应PMI，从而改善了PMI对稀少词的偏好问题。
## Post-processing Hyperparameters
修改最后得到的词向量
### Adding Context Vectors(w+c)
将同一个单词的词向量和上下文向量相加作为输出的词向量，可以改善表现。但不适合用于PPMI等方法。
### Eigenvalue Weighting(eig)
在前文中，SVD得到的词向量和上下文向量不对称，令其对称可改善语义性能：$W=U_d\cdot \sqrt{\Sigma_d}$, $C=V_d \cdot \sqrt{\Sigma_d}$。  
更一般地，可令$W=U_d\cdot \Sigma_d^p$。
### Vector Normalization(nrm)
对词向量标准化
# 超参数配置
SVD不可使用shifted PPMI。  
SVD的特征值指数不可取1，取0.5比较好。  
SVD、PPMI最好使用短窗口。  
SGNS最好使用较多的负样本。

# 其它要点
改变单个超参数的设置通常比换一个更好的算法或用更大的语料库训练对模型的提升更大。  
通过调整相似的超参数集合，没有某种算法会永远优于其它算法。 
调参至最优值的情况下，SGNS通常效果比GloVe好。  
 Nevertheless, Melamud et al. (2014) show that capturing joint contexts can indeed improve performance on word similarity tasks, and we believe it is a direction worth pursuing.  
# 疑问
文中说没有最优的算法，但根据实验来看，这只是对word similarity任务而言的，而对analogy任务，几乎总是SGNS要更好。而且在这些任务上，SGNS差的时候与最优算法也差不太多。总的来看，感觉还是SGNS要优秀一些。

