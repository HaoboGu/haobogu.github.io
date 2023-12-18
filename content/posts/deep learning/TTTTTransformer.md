---
title: "From Transformer to Transformer-XL"
author: "Haobo Gu"
tags: [“deep learning", "nlp", "transformer", "paper note"]
date: 2021-10-09T15:06:33+08:00
summary: "Note for learning transformer(in Chinese)"
---

## 背景

在Transformer之前，NLP领域的seq2seq模型主要基于RNN结构，如LSTM，GRU等。这种结构有几个难以克服的缺点:

1. 难以并行化
2. 速度慢
3. 长距离依赖记忆不够长

Google提出的基于self-attention的Transformer和Transformer-XL结构可以很好地解决RNN的缺点，所以自2017年以来，Transformer已经成为了NLP领域对语言建模的默认选项。 Bert、GPT、XLNET等模型的基础单位都是Transformer。

## Overview


论文里面的Transformer沿用了传统的encoder-decoder结构。其主要构造单元为：

1. Multi-Head Attention layer
2. Position-wise fully connected layer
3. Positional encoding
4. LayerNorm

需要注意的是，对于encoder和decoder，每个结构可以重叠N次，在论文里面N=6，即encoder叠了N层。这时，decoder的每一层拿到的K-V的输入都是encoder的最后一层的结果


## Attention

Multi-Head Attention是Transformer中最重要的一部分。

### Scaled Dot-Product Attention

Multi-head attention的基础是Scaled Dot-Product Attention, 这里，输入有三个，Q代表query，K、V代表一个Key-Value对。用公式表示为：

$$
Attention(Q, K, V) = softmax(\frac{QK^T}{\sqrt{d_k}})\times V
$$

所谓self-attention实际上就是Q、K、V三个是一样的。这里的Q、K、V都是多个单词embedding的矩阵。如果句子长度为128个token，embedding的长度$d\_{model}=512$，那么左边的softmax输出的实际上就是128个权重向量，和value embedding相乘得到加了self-attention的结果。

可以简单理解为，self-attention是通过Q和K算出注意力需要放到哪些维度上，即权重向量，然后乘V得到加上attention之后的输出。

### Multi-Head Attention

了解了Scaled Dot-Product Attention之后，Multi-Head Attention就简单了：在输入的时候，把V、K、Q都重复h次，这个h就是head的数目。当然这里不是简单的复制，而是对V、K、Q做线性变换h次。注意，在线性变换的时候，需要把输出的维度变为$\frac{d\_{model}}{h}$，这样的话在最终输出concat之后的结果的维度才正确。同时，这样也不会增加计算消耗。

在模型中，除了self-attention之外，还有一个encoder-decoder attention，区别就是在encoder-decoder attention中，Q和K变成了encoder的输出。V还是上一步decoder的输出。这里和传统RNN结构一样。

实际上，Transformer的并行性主要体现在训练上面，在做推理的时候，需要和传统RNN一样，一个词一个词地预测，并且在预测下一个词的时候需要当前的输出作为输入。

## Position-wise Feed-Forward Layer

这一层简称FFN，是对每一个位置的结果单独训练一个网络，位置和位置之间不共享参数，因此叫"Position-wise"。FFN的结构也很简单：两个线性变换，其中一个是ReLU：
$$
FFN(x) = max(0, xW_1+b_1)W_2+b_2
$$

中间层的宽度是超参数$d\_{ff}$，在论文中取值为2048

## Norm Layer

Transformer中，Attention层和FFN层后面都加了一个Normalization：

$$
LayerNorm(x+Sublayer(x))
$$

其中，LayerNorm的方法见Hinton的[论文](https://arxiv.org/pdf/1607.06450.pdf)。

## Positional Encoding

由于self-attention中并没有保存位置相关的信息，因此需要加Positional Encoding。

在Transformer论文中，作者提供了两种Positional Encoding：第一种是基于三角函数的，第二种是learned positional embedding。经过实验两者的结果差不多，所以作者使用的基于三角函数的Positional Encoding。这里，i是维度，pos是token的位置。所以，对于每个维度，位置编码的波长都是不同的。而对于相同的维度来说， $PE\_{pos+k}$ 总是可以表示为$PE\_{pos}$的线性函数。

## Transformer的问题

- 虽然在理论上，Transformer接受任意长的序列进行并行训练。但是，在实际训练中，由于GPU、TPU内存的限制，一般的做法是指定一个固定的上下文长度，然后把输入序列按照这个长度进行切分(segmentation)，然后把每个segment扔到Transformer里面进行训练。这种训练方式带来了两个缺点：第一个是无法捕捉跨segment的长距离依赖，第二个是固定长度切分会把同一个语义块（如一个句子）切分到不同的segment里面，这样的话在预测时，由于缺乏上文信息对前几个单词的预测效果很差。在训练时，也会降低收敛速度。

- 在预测时，Transformer在每一步只预测下一个单词，然后右移一位再预测。同样是只能拿到segment内的信息，且右移之后所有的预测步骤都要走一遍，速度慢。

为了解决这些问题，Google提出了Transformer-XL

## Transformer-XL

Transformer-XL主要有两个贡献：1. 解决fixed-length context问题。2.引入了一种新的位置编码方式

### Segment-Level Recurrence with State Reuse

Transformer-XL解决fixed-length context的方式是引入recurrent：对于每个segment，重用其之前的segment的状态开始训练而不是从头开始训练。

这里，$h\_{\tau}^n$表示第$\tau$个segment $s\_\tau$生成的第n个隐层状态序列。所以这里首先把前面的segment和当前的segment做一个concat，然后再用线性映射的方式生成新的Q、K、V扔进Transformer中。

在预测时也是类似，可以用到之前所有的segment的信息。由于所有的segment的隐状态都可以被缓存，所以相比Transformer，Transformer-XL的速度可以提升1800+倍。

### Relative Positional Encoding

在Transformer-XL中，由于要对不同的窗口进行处理，因此在原版Transformer中的基于三角函数的绝对位置编码就会出现问题：原来的编码只和维度i和token的位置pos有关系，那么在不同的segment中会出现一样的位置编码（即有相同的pos和i）。

原来的绝对位置编码公式中，$U\_{1:L}$ 是绝对位置编码，$E\_{s\_\tau}$是embedding，$h\_\tau$是隐状态

所以，Transformer-XL引入了相对位置矩阵：

$$
R\_i \in \mathbb{R}^{L_{max}\times d}
$$

这里，i是两个位置的相对距离，$L\_{max}$是整个输入序列的最长长度，在实际计算中，i可以是从0到M+L-1的任何数字，M是记忆的长度，L是segment长度。

在实际应用中，R是可以被提前计算出来的，使用vanilla transformer中的三角函数即可。

不同于vanilla Transformer中单纯地把位置编码和embedding加起来，Transformer-XL是在计算attention score的时候动态插入相对位置编码

这个公式猛一看不好理解，回想一下原始的attention score的计算方式：

$$
Attention(Q, K, V) = softmax(\frac{QK^T}{\sqrt{d_k}})\times V
$$

考虑进来原来的绝对位置编码U和线性变换矩阵W，有

$$
QK^T=(E+U)W_q\times ((K+U)W_k)^T
$$

那对于第i个query和第j个key来说，有

<img src="http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-10-09-081458.png" alt="image-2" style="zoom: 50%;" />

对比两个方案，发现区别有3点：

- 使用提前计算好的相对位置编码$R\_{i-j}$代替绝对位置编码$U\_j$

   这样就在计算attention score的时候动态引入了相对位置编码

- 使用可学习的参数$u,v$代替$W\_qU\_i$

   在考虑相对位置时，比较标准是位置i，所以这里的$U\_i$是固定的($R\_0$)，因此对于任意的i，这里都可以采用同样的向量表示。因此这两个参数就被归一化成了可学习的参数$u,v$

- 使用$W\_{k,E},W\_{k,R}$代替$W\_k$
之前由于是$(E+U)W\_k$，对E和U做的是同一个变换。这里把它分开，变成两个可学习的线性变换。其中$W\_{k,E}$对应的是对key的embedding的变换，而$W\_{k,R}$是对key的相对位置编码的变换。同理，使用两个参数$u,v$代替$W\_qU\_i$也是相同的理念

所以对于新的attention的公式，(a)对应的是content-based addressing，即query和key的内容(embedding)之间的关系；(b)对应的是content-dependent positional bias，即query内容相关的位置关系；(c)对应的是global content bias，即全局的内容影响（key的影响）；(d)对应的是global positional bias，全局的位置关系，即当前key的位置。

这种位置编码以前的论文也出现过，但是只有(a)和(b)，舍弃了(c)和(d)，即只看重和当前query相关的信息和位置编码。在Transformer-XL中，把和全局信息相关的东西也考虑进来了。

在实际实现的时候，通过平移可以减小bd的计算量，见Appendix B

相关代码为：

```python
def rel_multihead_attn(w, r, r_w_bias, r_r_bias, attn_mask, mems, d_model,
                       n_head, d_head, dropout, dropatt, is_training,
                       kernel_initializer, scope='rel_attn'):
  # w : token emb
	# r : 反向的绝对位置emb
	# r_w_bias ：公式中的u
	# r_r_bias : 公式中的v
  
  ...
  ...
  ...
    # 提取W_q + u和W_q + v
    rw_head_q = w_head_q + r_w_bias
    rr_head_q = w_head_q + r_r_bias

    # 计算(a)和(c)
    AC = tf.einsum('ibnd,jbnd->ijbn', rw_head_q, w_head_k)
    # 计算(b)和(d)，这里的计算用了一个trick，使得BD的O(N^3)的计算量降到了O(N)，见论文appendix B
    BD = tf.einsum('ibnd,jnd->ijbn', rr_head_q, r_head_k)
    BD = rel_shift(BD)

    # 对QK^T做scale
    attn_score = (AC + BD) * scale
    # Mask
    attn_mask_t = attn_mask[:, :, None, None]
    attn_score = attn_score * (1 - attn_mask_t) - 1e30 * attn_mask_t

    # Softmax + dropout
    attn_prob = tf.nn.softmax(attn_score, 1)
    attn_prob = tf.layers.dropout(attn_prob, dropatt, training=is_training)

    # attention向量乘以V得到最终的结果
    attn_vec = tf.einsum('ijbn,jbnd->ibnd', attn_prob, w_head_v)
  ...
  ...
```

在Segment-Level Recurrence和Relative Positional Encoding的基础上，一个N层的Transformer-XL的整体公式表示为：

<img src="http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-10-09-085118.png" alt="image-3" style="zoom:50%;" />

其中，n=1,...,N，$h\_\tau^0=E\_{s\_\tau}$，即第一层的h是embedding。

## Reference

1. Transformer: https://arxiv.org/abs/1706.03762 

2. Transformer-XL: https://arxiv.org/abs/1901.02860

3. The Annotated Transformer: https://nlp.seas.harvard.edu/2018/04/03/attention.html

4. The Illustrated Transformer: http://jalammar.github.io/illustrated-transformer/

   



