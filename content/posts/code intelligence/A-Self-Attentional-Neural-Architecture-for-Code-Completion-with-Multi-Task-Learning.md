---
title: "A Self-Attentional Neural Architecture for Code Completion with Multi-Task Learning"
author: "Haobo Gu"
tags: ["code completion", "paper note"]
date: 2020-07-16T15:16:28.899251+08:00
draft: false
summary: Paper note on A Self-Attentional Neural Architecture for Code Completion with Multi-Task Learning
---
# A Self-Attentional Neural Architecture for Code Completion with Multi-Task Learning

## Introduction

Current code completion algorithms sufferes from 3 major drawbacks:

1. Hierarchical structural information of the program is not fully utilized

2. In programs, the semantic relationships can be very long

3. Underuse of the information from related tasks

  In AST, node's type and value are two closely related attributes - but existing methods predict them separately

 

### Contribution

- Multi-task learning model for code completion
- Exploit the hierarchical structure information of program
- Introduce Transformer-XL to capture long-distance dependencies



## Motivating example

![image-20200706170542750](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-06-090543.png)

- Predicting `break`
- Predicting node's type & value

NOTE: borrows the idea from code2vec? 

## Background

### Language modeling

Probability of S:

![image-20200706170942064](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-06-090942.png)

N-gram:

![image-20200706170951403](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-06-090951.png)

RNN: capture longer dependencies than N-gram

LSTM/GRU: ease the vanishing gradient problem - uses about 200 context words on average

Transformer: fixed-length context

👍Transformer-XL: captures longer context



### Multi-task learning

MTL can improve generalization and reduce the risk of over-fitting

Method: parameter sharing of hidden layers

- soft parameter sharing: use the distance of parameters

- hard parameter sharing: hidden layers are shared among tasks, output layers are task-specific



## Proposed model

Task: predict the probability of next node in AST

Partial AST encoder: Transformer-XL

Path2root encoder: 

MTL: hard parameter sharing

output layers are specfic

![image-20200706172509025](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-06-092509.png)

### Program representation

program -> AST -> sequence of nodes

​	- in-order depth-first traversal

Node -> type[value] -> $x = [T;V]$

![image-20200708182320153](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-08-102320.png)

### Partial AST encoder

Transformer-XL is used to encode partial AST. It's a standard Transformer-XL.

![image-20200708200026435](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-08-120027.png)

### Path2Root encoder

The path from predicting node to the root node is extracted. In the above example, the extracted path is `{BinOp, Return, body, FunctionDef}` in order to predict `NameLoad[b]`

In this part, a bidirectional-LSTM is used as the encoder. The two hidden vectors are concatenated.

### Task-specific output layers

Two tasks: predict next node's type and value

Output layers: the output of partial AST encoder and path2root encoder are concatenated. Softmax is used to generate the prediction

### Training

#### loss

Because there are multiple tasks, the final loss is calculated using a weighted sum over the task-specific losses

## Experiments

### Dataset & Metrics

dataset: java/python/js

Metrics: accuracy

### Experimental setup

Vocab size: 50000

embedding size: type[300], value[1200], total[1500]

6-layer Transformer-XL, 6 heads, d_h = 64

Segment length = 50

### Performance

![image-20200708204448097](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-08-124448.png) 

### Ablation study

![image-20200708205259004](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-08-125259.png)

### Weights![image-20200708205456718](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2020-07-08-125457.png)

