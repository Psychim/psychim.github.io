---
layout: post
title:  "Neural Machine Translation of Rare Words with Subword Units"
categories: paper
tags: paper bpe nmt mt
---
# 方法
## Byte Pair Encoding(BPE)
1. 词语结尾添加特殊符号`·`或`</w>`，以便BPE的句子可以还原回词语
2. 将词语拆分成符号
3. 统计相邻的符号对频数
4. 合并最大的一对
5. 重复3-4 n次

### Example Code
```python
import re, collections

def get_stats(vocab):
    pairs = collections.defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

vocab= {'l o w </w>' : 5, 'l o w e r </w>' :2, 
        'n e w e s t </w>':6, 'w i d e s t </w>':3}
num_merges = 10
for i in range(num_merges):
    pairs = get_stats(vocab)
    best = max(pairs, key=pairs.get)
    vocab=merge_vocab(best, vocab)
    print(best)
```
# 实验
## 训练
先训练一段时间，取最后4个checkpoint，再固定embedding层训练这4个model。
# 结论
- 子词表示可以让神经网络学习复合词和音译词的翻译
- 联合源语言和目标语言生成BPE效果更好
- BPE能改善rare words和unseen words的翻译效果