---
title: "IntelliCode Compose"
author: "Haobo Gu"
tags: [paper-note, code-completion]
date: 2021-02-10T17:28:08+08:00
draft: false
summary: IntelliCode Compose - Code Generation using Transformer
---

## Introduction

Main contributions:

- Introduce GPT-C, a multi-layer generative transformer model, which is a variant of the GPT-2
- Propose and deploy a novel end-to-end code sequence completion system, and an efficient client-side caching system
- Introduce MultiGPT-C – a multilingual version of GPT-C

## Dataset

1.2 billion lines of source code in Python, C#, JS and TS;

4.7 million files

## Approach

Model definition

![image-20210219175241630](https://raw.githubusercontent.com/HaoboGu/pics/master/uPic/image-20210219175241630.png)

where $W_e$ is the input token embedding matrix, $C$ is the context vector of tokens, $W_p$ is the position embedding matrix.

The input token embedding matrix is reused in output layout, which removes large fully connected layout and reduces the number of parameters by 25%

## Preprocessing

AST/CST/CFG is not used in this paper.

The input code is normalized using a custom tokenizer to overcome the style issues. Also, the token type is extracted in both training and inference procedure to help the normalization, extract subtoken vocabulary and encode the sequences.

### Vocabulary

Two tokenization methods:

1. BPE
2. casing convention, including camelCase, PascalCase and snake_case

Several special tokens are added to vocabulary: `<BOF>`, `<EOF>`, `<EOL>`, `<INDENT>`, `<DEDENT>`

### Sensitive Data Processing

To avoid the leaking of sensitive data, IntelliCode Compose uses `<NUM_LIT>`, `<STR_LIT>` and `<COMMENT>` to replace numeric literals, string literals and comments. 

Note that not all literals are replaced, there are some literals in each language are kept. For example, `<STR_LIT:__main__>` means this token is a string literal `"__main__"`.

## Model Training

In training, IntelliCode Compose uses a decreasing learning rate with several warm-up perioids. The training is scaled up using synchronous data-parallel distributed training algorithm with local gradient accumulation.

The authors use 5 Lambda V100 boxes, each having sixteen V100 GPUs with 32 GB HBM2 memory, eight 100 GB InfiniBand, and one 100 GB Ethernet connection, managed with Kubernetes

The best model has 24 layers, 16 heads and the bpe vocabulary size is 50000. 

The hyper parameters are set as the following table:

![image-20210220142120344](https://raw.githubusercontent.com/HaoboGu/pics/master/uPic/image-20210220142120344.png)

In training, base learning rate is set to 6.25*10^-5, batch size 128, learning rate decay of 0.98 per epoch. 

## Sequence Decoding

Beam search, ending with `<EOL>` or other language specific tokens

Batched inference is performed, the batch size is set to beam size k, which reduces L*k inference calls to L, where L is the sequence length.

## Client-Side Post-Processing

### Caching

The suggestion results are cached as a trie with score. The key is the preceding context. If the user continues typing, the trie is pruned and then the suggestion is calculated by traversing the trie using a greedy search.

The trie traversing will be terminated early if none of the child nodes has a score that is larger than the score of its parent multiplied by a ratio 𝑅:

![image-20210220143752196](https://raw.githubusercontent.com/HaoboGu/pics/master/uPic/image-20210220143752196.png)

Where L is the position in sequence. $\alpha$ And $k$ are two parameters, $\alpha$ controls the overall average length, and $k$ controls the speed than R decreases. In the practice, the authors select 𝛼 = 0.8 and 𝜅 = 10 to gain a balance between suggestion length and relevance.

### Suggestion Processing

The special tokens are removed or replaced case by case.

## Multi-Lingual Model

The authors tests four approaches:

1. Language-agnostic baseline

2. Language-type embedding

3. Language control code

4. Double heads multilingual model

   Add a programming language classification task during model pretraining

![image-20210220150046866](https://raw.githubusercontent.com/HaoboGu/pics/master/uPic/image-20210220150046866.png)

## Evaluation

![image-20210220150030532](https://raw.githubusercontent.com/HaoboGu/pics/master/uPic/image-20210220150030532.png)





