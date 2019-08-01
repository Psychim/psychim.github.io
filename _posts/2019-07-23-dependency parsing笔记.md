---
layout: post
title:  "Dependency Parsing笔记"
categories: "dependency parsing"
tags: dependency-parsing
---
# 语法结构
语法结构分为两种：constituency structures和dependency structures  
## Dependency Structure
某个单词(dependent)依赖于另一个唯一的单词(head)的依赖关系构成。所有依赖关系形成一个树结构，某个单词为根，或者只有一个假根(fake root)。
# Dependency Parsing
解析句子依赖结构的任务，输出一棵依赖树。Dependency Parsing分为两个子任务：
1. Learning: 通过训练集学习一个解析模型M
2. Parsing: 用一个解析模型M求出一个句子S的最优依赖图D

## Transition-Based Dependency Parsing
通过状态机的状态转移(transition)生成依赖树。其learning任务是学习一个可以预测下一步转移的模型。parsing任务则通过状态转移和转移模型生成依赖树。  
状态机由states和transitions两个要素组成。
### States
一个状态是一个三元组$c=(\sigma,\beta,A)$:  
1. $\sigma$是一个词语栈，包含了接下来要获得依赖关系的词语。  
2. $\beta$是一个缓冲区，包含了句子中还未被处理的词语。  
3. $A$是依赖关系集合，一个依赖关系表示为$(w_i,r,w_j)$，表示$w_i$依赖于$w_j$，$r$是依赖的种类。  

对于一个句子$S=w_0w_1\cdots w_n$，其初始状态$c_0=([w_0]_\sigma,[w_1,w_2,\cdots,w_n]_\beta,\empty)$，终止状态为$(\sigma,[]_\beta,A)$
### Transitions
有三种类型的转移：
1. SHIFT: 将$\beta$的第一个词语入栈$\sigma$。前提是$\beta$非空；
2. LEFT-ARC$_r$: 添加一个依赖关系$(w_j,r,w_i)$，$w_i$是栈顶第二个元素，$w_j$是栈顶元素，然后移除$w_i$。前提是$\sigma$中至少有2个元素且$w_i$不能为根；
3. RIGHT-ARC$_r$: 添加一个依赖关系$(w_i,r,w_j)$，$w_i$，$w_j$同样分别为栈顶第二、第一个元素，然后移除$w_j$。前提是$\sigma$中至少有2个元素。

