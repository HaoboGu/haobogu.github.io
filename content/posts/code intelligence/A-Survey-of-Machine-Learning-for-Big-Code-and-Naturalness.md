---
title: "A Survey of Machine Learning for Big Code and Naturalness"
author: "Haobo Gu"
tags: ["paper note", research]
toc: true
date: 2021-08-16T17:38:34.723004+08:00
---

<!--more-->
## Introduction

Software engineering(SE) and programming languages(PL) should make the transition just like NLP: from brittle rule-based expert systems to statistical methods and machine learning techniques. 

The paper is orginized as follows:

1. Difference between natural language and source code
2. Taxonomy of probabilistic models of source code
3. SE and PL applications 
4. Challenges and future directions

## The Naturalness Hypothesis

The Naturalness Hypothesis

> Software is a form of human communications; software corpora have similar statistical properties to natural language corpora; these properties can be exploited to build better software engineering tools

Insights:

1. Because coding is an act of communication, so the large code corpora is expected to have rich patterns, just like natural language corpora. 
   Example: models developed for natural language are effective for source code
2. Probabilistic model can be *learned* to describe how developers *naturally* writes and use code.
3. Naturalness means learnable and predictionable


## Text, Code and Machine Learning

Source code plays a role of connector between computers and the human mind. 

Hindle et al. shows that code is more predictable than texts using n-gram language model. 

Enumerate differences between code and texts is helpful for us to gain some insights on modifying NLP techniques to deal with code:

Code is executable and has formal syntax and semantics. We will discuss these aspects comparing code with text. 

### Executable

Code is more brittle, small changes may lead to huge differences. Texts are usually more robust to readers. It requires combination of probabilistic and formal methods. 

Programming languages can be translated between each other exactly, because all mainstream programming languages are Turing-complete.

### Formality

Programming languages are formal languages. Code's formality facilites reuse, Gabel et al. found that code is more pattern dense than text. 

Also, code has less ambiguity than text, but the problem of ambiguity still exists.

### Cross-Channel Interaction

Natural semantic units of code are **identifiers, statements, blocks and functions**. All of them have less semantic information than textual semantic units like words and sentences.

Almost 70% words are identifiers and neologisms(新词) are more common in code. 

 - How existing work handles neologism: introduce cache mechanisms or decompose identifiers at a subtoken level

Which semantic unit of code is more useful? It's an open question. 

## Probabilistic Models of Code

### Code-generating Models

General code-generating models can be formulated as:
$$
P_D(c|C(c))
$$
When context $C(c)$ is $\emptyset$, the probability distribution $P_D$ is a **language model** for code; when $C(c)$ is code, $P_D$ is a **transducer model**; when $C(c)$ is natural language, $P_D$ is a **code-generative multimodal model** of code.

In addition, by the way they generate code's structure, code-generating models can be divided into three types:

1. Token-level models: generate token sequences

   - Cannot always generate executable code
   - Approaches: 
		- N-gram, with smoothing and cache mechanism. 
  		- RNN

2. Syntactic models: generate tree structures

   - Generate syntactically correct code
   - Different from application in NLP, PCFG is not suitable for code generation. Because nearby tokens may be far away in the AST, PCFG cannot even capture close dependencies(as well as long-range dependencies, but this is not the point, n-gram do not capture this neither)
   -  Expensive computation cost than token-level model

3. Semantic models: generate graph structures

   - DIfficult, less resource on this topic

#### Types of Code Generating Models

##### 1. Language Models

LM models programming language itself without using any external content. 

Code LMs are evaluated using perplexity or cross-entropy just like normal LM in NLP

##### 2. Code Transducer Models

Code transducer models translate code from one format to another format. For example, pesudo code generator or code migration. 

Most of transducer models are phrase based model, which is not good at describing long-range dependency.

Transducer models can be evaluated using BLEU, etc. 

##### 3. Multimodal Models

Multimodal model exploits non-code modalities, like comments, search queries, etc. 

The key idea is to convert natural language to an intermediate representation, then parse it to code, which is similar with semantic parsing in NLP

### Representational Models of Code

Different from generative models, representational models learns embeddings from code. 

#### Distributed representations(Code embedding)

Tasks

- Predict method name
- Predict relevant API sequences(Seq2seq)
- Represent solution-problem mapping
- Source code input&output pairs to review student assignment
- Neural code-generative models 
- Generate other modalities like comments

#### Structured prediction

Predict interdependent variables, like POS prediction in NLP. Structured prediction and distributed representations are not exclusive.

Example:

- *Raychev et al*. represent code as a variable dependency network, each variable is a node, model pairwise interactions as a CRF -> predict types and names of variables

- *Gu et al.* predict the sequence of API calls

### Pattern Mining Models

Pattern mining models aims to discover **a finite set of human-interpretable patterns** from source code.

Example:

​	*Fowkes et al.* learn the latent variables of a graphical model to infer common API usage patterns

## Applications

### Recommender Systems

Modeling programmers' intent is a challenge.

The most prominent recommender system in code intelligence field is **<u>code completion</u>**.

Existing Research:

 -  n-gram and cached n-gram:

    n-gram LM is used to model source code, cache is used to catch local information

 -  AST-level LM

    Maddison and Tarlow

### Inferring Coding Conventions

Coding convention: syntactic constraints on code beyond grammar, like formatting, CamelCase naming, etc.

Learning of coding conventions from codebases helps teams to determine their coding conventions

Machine learning models for learning surface structure are well-suited for this task, while there exists a challenge: the sparsity of the code constructs. 

### Code Defects

Finding code defects is quite difficult beacuse of the rarity of defects and the extreme diversity of source code. 

Proliminary work shows that the lower probability of specific code may indicate code defects. Not all anomalous behavior is a bug(it may be just rare behavior), the vast diversity of code constructs entails sparsity, from which all anomaly detection methods suffer. 

### Code Translation, Copying, and Clones

#### Translation

The success of SMT has inspired researchers to use ML to translate code from one source language to another. Translation on similar languages are easier. But for languages which have different memory allocations, translation is difficult. 

#### Clones

Clones may indicate refactoring opportunities.

Embedding can be used to detect clones due to the similar snippets share the similar distributed vector representations. Use embedding allows us to learn a continuous similarity metric(like cosine distance) between code locations, rather than using edit distance.

### Code to Text and Text to Code

#### Code to Text

C2T has applications to code documentation and readability. 

Example: code to pseudocode, code to comments

#### Text to Code

This area is more related to semantic parsing in NLP.

> Semantic parsing is the task of converting a natural language utterance into a representation of its meaning, often database or logical queries. Those representations can be used in subsequentily question answering.

Based on current research, the target code can be Excel macros, Java expressions, shell commands, etc. 

For more information, please refer to Neubig's survey of code generation methods. 

### Documentation, Traceability and Information Retrieval

General C2T and T2C can be used here, but more specialized solutions are more effective on problems like documentation and code search. Information retrieval methods are widely used. 

Code search problems are evaluated using rank-based measures. 

### Program Synthesis

Program synthesis is concerned with generating full or partial programs from a specification. When the specification is natural language, this is a semantic parsing task(same as T2C). Besides natural language, a lot of other specifications can be used in program synthesis.

### Program Analysis

Program analysis aims to ***soundly*** extract semantic properties (like correctness) from programs. There are three streams which exploit probabilistic models to deal with code sound analyses problem.

1. A family of modes relaxes the soundness requirement, yielding probabilistic results instead

2. The second stream uses machine learning to create models that produce plausible hypotheses of formal verification statements that can be proved

   Example: Brockschmidt et al. proposed a model which generates separation logic expressions from the heap graph of a program, suitable for formally verifying program's correctness.

3. The third stream learns heuristics replacing hard-coded heuristics to speed-up the search for a formal automated proof

## Challenges and Future Directions

Deep learning is popular, but it is not necessary. In many cases, simple models can outperform advanced off-the-self deep learning methods. 

### Bridging Representations and Communities

In software engineering research, a well-defined and widely useful set of representations are used, such as AST, CFG, etc. In contrast, machine learning usually uses continuous representations. 

Introducing better representations that bridge the gap between machine learning and source code will allow the probabilistic models of code to reason about the rich structure and semantics of code. 

### Data Sparsity, Compositionality and Strong Generalization

**<u>Sparsity</u>**: it's hard to find multiple source code elements which perform **exactly** the same tasks. 

**<u>Compositionality</u>**: meaning of some element can be understood by composing the meaning of its constituent parts. 

- Example

  Methods -> Classes

  Word -> Variable names in camel case naming

- This area is still challenging for general machine learning fields, because capturing relations between objects especially across abstraction levels is difficult.

  > If suﬃcient progress is to be made, representing source code artifacts in machine learning will improve signiﬁcantly, positively aﬀecting other downstream tasks

**<u>Strong generalization</u>**: big machine model learned from big code is usually to large for developer's machine. Hence, generalization is a deployability problem. 

- Direction: transfer learning & one-shot learning
- Learning from multiple views of source code may help to deal with generalization problem and data sparsity

### Measures

High-quality benchmarks, shared tasks and evaluation metrics play an important role win great success in NLP and CV in recent years. 

- Some meansurements can be imprecise

- Some metrics that are widely used in NLP are not suitable for source code, such as BLEU. 

  Small changes in source code may lead to big difference; syntactically diverse answers may be semantically equivalent.

## New Domains

### Debugging

Trace buggy code during execution

### Traceability

Study links among software engineering artifacts

Example: fix commit -> bug reports

### Code Completion and Synthesis

We don't have a well-accepted benchmark in this area. 

When reasoning about incomplete code's semantics, modeling how code could be completed is helpful 

### Education

### Assistive Tools

## Conclusions

Almost every area of program analysis and software engineering would benefit from the development of probabilistic models of source code. 

> Probabilistic models of source code raise the exciting opportunity of learning from existing code, probabilistically reasoning about new source code artifacts and transferring knowledge between developers and projects
>
> Most of the research contained in this review was conducted within the past few years

