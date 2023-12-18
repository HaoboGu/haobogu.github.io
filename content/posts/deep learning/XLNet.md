---
title: "XLNet"
author: "Haobo Gu"
tags: ["xlnet", "paper note"]
date: 2021-06-11T14:15:10.659576+08:00
draft: false
summary: Introduction of XLNet
---
# XLNet

## Introduction

现在主流的预训练模型可以被分为两类：autoregressive(AR)和autoencoding(AE)

- AR：recurrent model

  单向模型

- AE：reconstruct the original data

  gap between pretraining and finetuning

  假设：被mask的词无关

XLNet尝试使用一个新的预训练任务来解决AR和AE预训练模型现存的一些问题。

Contribution：

- 预训练任务Permutation：不使用前向/后向的顺序，而是使用打乱后的单词顺序
- 使用Transformer-XL里面的segment recurrence mechainism和relative position encoding
- 调整Transformer-XL使得其结构适用于permutation-based LM

## Objective: Permutation Language Modeling

如果序列$\textbf{x} = [x_1, ...,x_T]$, 那么一共会有T!种排列方式，记为 $Z_T$ 。那么， $z_t$ 和 $\textbf{z}_{<t}$ 就是permutation$\textbf{z}\in Z_t$中的第t个元素，和前t-1个元素（在原始序列的的位置）

那么，建模的目标就可以设置为：

![image-20200608193906171](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-08-113906.png)

其中，$\theta$是所有的permutation所共享的

### Remark

训练的目标是把输入序列的顺序打乱了的，因此我们需要记住原始序列的顺序：positional encoding只使用原始序列中的位置

另外，在finetuning的时候，只使用原始序列

?是否仍然存在pretraining-finetuning gap?

### 对Transformer的修改

由于XLNet是permutation language modeling，因此不能使用传统的softmax作为输出层：

![image-20200608195203229](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-08-115203.png)

其中$h_\theta$是X_z<t的隐藏状态。这样的话，会导致对于相同前缀的permutation，其下一个token的概率永远是一样的：

[1,2,3 | 4,5] vs [1,2,3 | 5,4]

因此需要加入位置信息$z_t$，上式变为：

![image-20200608195451740](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-08-115451.png)

这里就是所谓的Two-Stream Self-Attention的来源：同时考虑$g_\theta$和$h_\theta$

### Two-Stream Self-Attention

这里带来了一个问题：预测$x_{z_t}$，就只能考虑位置信息$z_t$；预测$x_{z_j}, j > t$，那么就可以考虑整个t位置上面的context信息$x_{z_t}$

所以，XLNet提出同时维护两套隐状态：$h_\theta(x_{z_{<=t}})$和$g_\theta(x_{z_{<t}}, z_t)$，后面使用$h_{z_t}$和$g_{z_t}$表示，即content stream和query stream。

对于第一层，g0被设为一个可训练的向量，h0就直接使用第一个词的embedding。在训练的时候，两个stream共同使用一套参数$\theta$

![image-20200609110004280](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-09-030004.png)

![image-20200609110749492](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-09-030749.png)



![image-20200609110801520](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-09-030801.png)

整体模型的结构：

![image-20200609111723275](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-09-031723.png)

### Partial Prediction

由于实际训练时，permutation LM收敛很慢，因此在每一个permutation序列中只预测一部分词

### Incorporating Ideas from Transformer-XL

从Transformer-XL中借鉴了两个非常重要的机制：

- relative positional encoding

  用来记录原始序列中的顺序

- segment recurrence mechanism

  ![image-20200609114101816](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-06-09-034102.png)

### Modeling Multiple Segments

对于多segment的训练，XLNet会随机采样两个segment，然后把它们接起来 - [CLS, A, SEP, B, SEP]，作为PLM中的一个序列来训练

### Relative Segment Encoding

只考虑两个位置是否在同一个segment中，记为s_ij

对于两个位置i,j，Attention weight $a_{ij}=(q_i+b)^Ts_{ij}$ ，其中q_i是Transformer的query，b是bias



## Experiment

### Setup

- tokenizer: sentencepiece

- Pretraining sequence length: 512

- 512TPU v3, 500K steps
- optimizer: Adam weight decay optimizer
- Batch size: 8192
- pretraininig cost 5.5 days

### Performance

好于BERT/RoBERTa





