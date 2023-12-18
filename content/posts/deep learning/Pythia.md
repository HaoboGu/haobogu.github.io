---
title: "Pythia: AI Assisted Code Completion System"
author: "Haobo Gu"
tags: [paper-note, code-completion]
date: 2020-09-04T17:59:37+08:00
summary: As a part of Intellicode extension in Visual Studio Code, Pythia exploits large scale deep learning methods trained on extracted code context sequences from ASTs and achieves SOTA performance on Python code completion task
slug: "pythia-ai-assisted-code-completion-system"
---

Paper link: https://www.kdd.org/kdd2019/accepted-papers/view/pythia-ai-assisted-code-completion-system



## Abstract

This paper proposes an end-to-end code completion approach call Pythia, which has been deployed as part of Intellicode extension in Visual Studio Code. Pythia is trained on 2700+ Python open source software GitHub repositories and achieves SOTA performance. Pythia uses LSTM to learn the long distance dependencies in code context sequences, which is extracted using in-order depth-first AST traversal. The parameter tuning and deployment related issues are also discussed in the later parts of the paper.

## Dataset

The authors collected 2700 top-starred Github Python projects in various domains. The dataset contains over 15.8 million **method calls**. The following is the most method call occurrences:

![image-20190904104648313](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-09-04-024648.png)

The dataset is divided to development set and test set with a 70-30 ratio on the repo level. Then the development set is split into training and validation sets in the proportion 80-20.

The online model is trained using the entire dataset.

## Code Representations

The Pythia exploits the partial AST which is derived from code snippets containing method calls and member access expression. The ASTs are serialized to token sequence using **in-order depth-first traversal**. When extracting training sequence, for each method call, ***T*** preceding tokens are used, where T is a tunable parameter.

Further, the AST token sequence must be converted to numeric vector to be consumed by LSTM. Word2Vec is used to train the embedding of tokens. 

### Embedding  training

All tokens are mapped to integers from 1 to V, while the infrequent tokens are removed in order to reduce the vocabulary size. During training, OOVs are mapped to an integer greater than V. `.` is used as the EOS character. 

Method names are one-hot encoded as the labels. The task is to predict method names using given code snippets. All tokens are mapped to low-dimentsional vectors with semantic relationships preserved. 

### Tricks

1. The type of variables is inferred using static analysis methods
2. Import alias is ignored
3. Variable names are normalized to `var:<variable type>` format

## Nerual Code Completion Model

LSTM is used to predict completion. 

![image-20190904113911307](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-09-04-033911.png)

The input embedding matrix is reused as the output classification matrix to reduce the number of parameters. Hence, the fully connected layer in before the output is no longer needed. 

### Model Training

Parallel backpropagation through time algorithm with adam optimizer and mini-batch is used to train the LSTM. In the training, the sequences are split into three buckets by sequence lengths. In each bucket, the lengths of sequences are padded to the maximum sequence length in this bucket. 

![image-20190904143553065](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-09-04-063553.png)

As for the learning rate, the authors scale the learning rate up proportionally to the number of works during the first 4 epochs:

![image-20190904145128559](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-09-04-065128.png)

## Model Evaluation

![image-20190904145730931](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-09-04-065731.png)

![image-20190904145716624](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-09-04-065716.png)

## Model Deployment

To deploy the model to lightweight client devices, the **neural network quantization** method is used to reduce the number of stored weights. The original Pythia model is trained in IEEE 754 numeric format, which is based on 32-bit float. The numeric format of published model is quantized into 8-bit unsigned integer. For Pythia, the model size is reduced from 152MB to 38MB with quantization. The top-5 accuracy is reduced from 92% to 89%. 

